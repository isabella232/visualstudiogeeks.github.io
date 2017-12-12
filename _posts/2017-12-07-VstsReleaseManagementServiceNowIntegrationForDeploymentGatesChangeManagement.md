---
layout: post
title: "Integrating VSTS Release Management with ServiceNow using Deployment Gate for Change Management"
date: 2017-12-07
author: tarun
tags: ["DevOps", "VSTS", "ServiceNow", "ReleaseGates"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/Dec17/VstsServiceNow-int.jpg"
description: "Do you want to integrate VSTS build & release management with ServiceNow for change management? In this blogpost we'll see how easy it is to build integration between these two systems to break the silos between development and operations teams. We'll be leveraging the deployment gates functionality to build the integration with Service Now by using Azure Functions."
permalink: /DevOps/IntegratingServiceNowWithVstsReleaseManagementUsingDeploymentGate
published: true
keywords: "ServiceNow, DevOps, VSTS, ServiceNow VSTS Integration, ServiceNow VSTS PlugIn, ServiceNow VSTS Release Management, ServiceNow Change Management VSTS, Vsts Deployment Gate ServiceNow, Vsts Deployment Gate Azure Function, Azure Function, Azure Function Service Now, ServiceNow API Azure Function, ServiceNow Json Azure Function, Deployment Gate, Release Management, Visual Studio"
---
Are you using ServiceNow for change management and VSTS for Release Management. Are you wondering **how you could automate the release approval once a change has been approved for implementation in service now...?** Look no further, in this blogpost i'll show you how to leverage the *Deployment Gates* feature in VSTS to integrate with service now and hopefully help you remove a manual step from your release approval workflow...

![ServiceNow Integration with VSTS](/images/screenshots/tarun/Dec17/SurprisedHappyVstsDeploymentSignal.gif)

<!--more-->
VSTS allows you to create Deployment Gates... Read this official post for a low down on what [Deployment Gates](https://docs.microsoft.com/en-us/vsts/build-release/concepts/definitions/release/approvals/gates) do... To save you the trouble, "Deployment Gates" do exactly what they say...

> *Gate the deployment process*...  

Gates enable integration with a bunch of external services and API's that allow you to plug in anything from your 'live site monitoring tool' to your 'issue management tool' to your 'change management tool'... 

> #### Gates also allow **native** integration with _Azure functions_, which is what we'll use to build out our integration with Service Now.

# Service Now - API 
Now before you can integrate with ServiceNow, you need to know how to get data out of serviceNow. ServiceNow supports a proper [REST API](http://wiki.servicenow.com/index.php?title=REST_API). However, you need to be set up to use the REST API, this can be challenging in various enterprise organizations. Simply that or if you are stuck with a ServiceNow instance that's really old, you'll find it doesn't support the REST API.

I'll show you a little way to cheat, you don't really need to use the ServiceNow REST API to get data out of ServiceNow... ServiceNow stores everything in either purpose build tables or custom tables, which you can directly navigate to... Let's see this in action, for example - changes are stored in 'changes tables', you can see this reflect in the URL... 

![ServiceNow Identify Table Name](/images/screenshots/tarun/Dec17/SnowCrNo.jpg)

Now that we know the name of the table, let's see how easy is to build out a direct URL to it... 

`https://xxx.service-now.com/change_request.do?sysparm_action=getRecords&sysparm_query=number=CHG00557943&displayvalue=true`

Last but not the least, now append `&JSON` to the URL to get the response back in JSON. You can optionally use `csv` if you want to get the response back in a comma separated format.

> You may find that more up to date instances of ServiceNow would accept `&JSON2` instead of `&JSON`... More on this available on ServiceNow docs [here](https://docs.servicenow.com/bundle/jakarta-servicenow-platform/page/integrate/inbound-other-web-services/concept/c_JSONv2WebService.html?title=JSONv2_Web_Service)

`https://xxx.service-now.com/change_request.do?sysparm_action=getRecords&sysparm_query=number=CHG00557943&displayvalue=true&JSON`

![ServiceNow Change Request in JSON](/images/screenshots/tarun/Dec17/SnowCrJson.jpg)

Now that we have this figured out let's see how we can now leverage the deployment gates to query service now to identify whether a change is approved... 

# Deployment Gates - Azure Function
At the time of writing the blogpost, VSTS supports the following deployment gates... As you will notice all of them come with great flexibility... 
+ Azure Monitor
+ Invoke Azure Function
+ Invoke REST API
+ Query Work Items

![ServiceNow Change Request in JSON](/images/screenshots/tarun/Dec17/VstsDeploymentGate.jpg)

We'll be leveraging the `Invoke Azure Function` deployment gate... You can get to the Deployment Gate by navigating into the release definition and clicking the event icon on the environment... You can have one or many deployment gates associated to one environment, just don't let this get to your head, it's easy to end up making the release process overly complicated... 

# Azure Function to integrate with ServiceNow

We'll create an Azure function that'll accept a change request number, invoke the ServiceNow API with the change number and parse the JSON response to get the status of the change request. Let's get started...

Launch Visual Studio, create a new project using the Azure Function project template... Visual Studio 15.3.5 now has out of box support for Azure Function project template... Name the project `ChangeStatusFuncApp`...

![Azure Function Project](/images/screenshots/tarun/Dec17/Vs15_FuncApp.jpg)

In the project add a new item of type `Azure Function`, select the type as `Http trigger` and the access right as `Function`...

![Azure Function Project](/images/screenshots/tarun/Dec17/AzureFunForServiceNow.jpg)

Now in the local.settings.json file add two settings, one to store the username and the other two store the password of the account you'll use to access serviceNow. 

![Azure Function App Setting](/images/screenshots/tarun/Dec17/AzureFuncAppSettings.jpg)

Alright, let's add some code to access the ServiceNow API and parse the json...

``` cs

using System.Linq;
using System.Text;
using System.Net;
using Newtonsoft.Json;
using System.Net.Http;
using System.Configuration;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System;
using System.Net.Http.Formatting;

namespace ChangeStatusFuncApp
{
    public static class SnowChangeStatusTrigger
    {
        [FunctionName("SnowChangeStatusTrigger")]
        public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]HttpRequestMessage req, TraceWriter log)
        {
            log.Info("C# HTTP trigger function processed a request.");

            // parse [cr] query parameter
            string cr = req.GetQueryNameValuePairs()
                .FirstOrDefault(q => string.Compare(q.Key, "cr", true) == 0)
                .Value;

            // Dynamically generate the URL for ServiceNow API 
            var restUrl = "change_request.do?sysparm_action=getRecords&sysparm_query=number=" + cr + "&displayvalue=true&JSON";

            var svc_usr = ConfigurationManager.AppSettings["ServiceNowUsername"];
            var svc_pwd = ConfigurationManager.AppSettings["ServiceNowPassword"];

            var client = new HttpClient();
            var byteArray = Encoding.ASCII.GetBytes(string.Format("{0}:{1}", svc_usr, svc_pwd));
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", Convert.ToBase64String(byteArray));
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.BaseAddress = new System.Uri(@"https://xxx.service-now.com");
            client.MaxResponseContentBufferSize = 25600000;

            var response = await client.GetAsync(restUrl);
            var json = response.Content.ReadAsStringAsync().Result;
            var obj = JsonConvert.DeserializeObject<Rootobject>(json);

            return obj == null 
                ? req.CreateErrorResponse(HttpStatusCode.BadRequest, string.Format("No data found with input '{0}'", cr)) 
                : req.CreateResponse(HttpStatusCode.OK, obj.records.First(), JsonMediaTypeFormatter.DefaultMediaType);
        }
    }


    public class Rootobject
    {
        public Record[] records { get; set; }
    }

    public class Record
    {
        public string sys_id { get; set; }

        public string u_change_status { get; set; }

        public string description { get; set; }
    }
}

```

Test this locally to ensure it's working as expected... Run the function in debug mode and pass the parameter `cr=CHG00557943`...

![Azure Function Debug locally](/images/screenshots/tarun/Dec17/AzureFuncDebugLocally.jpg)

Publish the function to a function app and be sure to add the two application settings namely `ServiceNowUsername` and `ServiceNowPassword` as application settings in the Function App. You can test the function app by running the same test along with the Function key...

# Integrating ServiceNow Azure Function Deployment Gate into VSTS
Launch Visual Studio Team services in a browser and navigate to the release pipeline, click the event icon on the environment to bring up the release settings for that environment. Navigate down to gates and click to add new gate, select Azure function. Specify the Azure Function URL `https://azsu-af-snow-devtest-001.azurewebsites.net/api/SnowChangeStatusTrigger` and specify the Azure function key, you can store this securely in a release level variable and encrypt it, specify the query parameter as `cr=CHG00557943`... 

![Deployment Gate Options](/images/screenshots/tarun/Dec17/DeploymentGateVsts.jpg)

Next set the completion option... Set the completion based on `ApiResponse` and the success criteria as `eq(root['u_change_status'], 'Approved for implementation')`... This will only let the deployment gate go green when the service now change request status is equal to Approved for implementation, in short the deployment won't start till the change is fully approved for implementation. 

![Deployment Gate Options](/images/screenshots/tarun/Dec17/AzureFuncVstsChangeEvaluate.jpg)

Be sure to set the timeout and the sampling interval as per your requirements... 

![Deployment Gate Options](/images/screenshots/tarun/Dec17/DeploymentGateOptions.jpg)

With the deployment gate successfully configured let's kick off the release for this environment to see the service now integration to check the status of the change request ahead of starting the deployment... 

![Deployment Gate Options](/images/screenshots/tarun/Dec17/deploymentgatesummary.jpg)

You can download the logs to see the full results of the deployment gates, including API calls and the results back from the API...

![Deployment Gate Options](/images/screenshots/tarun/Dec17/VstsAzureDeploymentGateLogs.jpg)

And finally as the release succeeds, this is what the final release summary looks like... 

![Deployment Gate Summary](/images/screenshots/tarun/Dec17/ReleaseGatesSummary.jpg)

# Summary
Deployment Gate gives you access to Azure Functions that allow you to integrate with other systems and services within your organization to automate the workflows that are manual in your deployment process... In this blogpost we saw how easy it is to integrate Service Now with VSTS to validate the change request approval ahead of rolling out the deployment in a specific environment. Would you like to see deeper integration between VSTS and ServiceNow, if so, what else?

 






















 