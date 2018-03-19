---
layout: post
title: "GitHub Issues - As a deployment gate in VSTS Release Management"
date: 2018-01-23
author: utkarsh
tags: ["DevOps", "VSTS", "GitHub", "ReleaseGates"]
categories:
- "DevOps"
description: "Do you want to stop deployment as long as there are outstanding issues in your GitHub repository? In this blog post we will see how we can leverage powerful VSTS deployment gate functionality to validate all the necessary prerequisites for your next big deployment."
permalink: /DevOps/github-issues-as-deployment-gate-in-vsts-rm
keywords: "DevOps, VSTS, VSTS Deployment Gate Azure Function, Azure Function, Azure Function, GitHub, GitHub Issues, Deployment Gate, Release Management, Visual Studio"
image: "/images/screenshots/utkarsh/github-issues-deployment-gate/deployment-gate-success.png"
published: true
---

Software teams monitor user feedback and issue reports when rolling releases across their infrastructure estate. In our (VisualStudioGeeks - Utkarsh & Tarun's) experience Twitter and GitHub combined are the quickest way to get updates on potential problems in your software. The process of monitoring GitHub issues as a deployment gate is a manual step today, in this blogpost we'll show you how to automate this by leveraging the power of Azure Functions and VSTS. 
<!--more-->

In this blogpost we'll explore, 
+ GitHub Issues API
+ Creating an Azure Function to inspect GitHub issues
+ Integrating Azure Function Deployment Gate into VSTS

## GitHub Issues API  ##

GitHub API allows us to search [many things](https://developer.github.com/v3/search/). But because our goal is to wait for deployment if we have any open issues in our repository, we will be using [search issues API](https://developer.github.com/v3/search/#search-issues).

GitHub also has a very powerful search qualifiers which allow you to fine grain your search and get you the exact results. You can read more about search qualifiers [here](https://help.github.com/articles/searching-issues-and-pull-requests/).

To illustrate, say we would like to search open issues in the [Export/Import Build definition extension repository](https://github.com/onlyutkarsh/ExportImportBuildDefinition). The search query for this would be...

```
https://api.github.com/search/issues?q=state:open+repo:onlyutkarsh/ExportImportBuildDefinition&sort=created&order=asc
```

## Azure Function to search GitHub issues ##

As Tarun showed in his previous [blog post](https://www.visualstudiogeeks.com/DevOps/IntegratingServiceNowWithVstsReleaseManagementUsingDeploymentGate), VSTS deployment gate has native support for Azure functions. 

Following on from that post, use the following login in your HTTP based Azure function to query GitHub using it's issue API. The function accepts a search query as an input and returns the search result as a JSON.

```csharp
[FunctionName("SearchGitHubIssues")]
public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequestMessage req, TraceWriter log)
{
    var query = QueryHelpers.ParseQuery(req.RequestUri.Query);
    var items = query.SelectMany(x => x.Value, (col, value) => new KeyValuePair<string, string>(col.Key, value)).ToList();

    var queryBuilder = new QueryBuilder(items);
    var searchQuery = queryBuilder.ToQueryString();

    var request = new HttpRequestMessage(HttpMethod.Get, $"https://api.github.com/search/issues{searchQuery}");
    request.Headers.Add("Accept", "application/vnd.github.v3+json");
    request.Headers.Add("User-Agent", "YOUR-APP-NAME");

    var response = await _httpClient.SendAsync(request).ConfigureAwait(false);
    string json = await response.Content.ReadAsStringAsync().ConfigureAwait(false);
    GitHubSearchResult jsonObject = JsonConvert.DeserializeObject<GitHubSearchResult>(json);
    if (response.StatusCode == HttpStatusCode.OK)
    {
        SearchResponse succesResponse = new SearchResponse();
        succesResponse.SearchTerm = searchQuery.ToString();
        succesResponse.TotalCount = jsonObject.TotalCount;
        succesResponse.SearchResult = jsonObject;
        log.Info("Response returned sucessfully.");
        return req.CreateResponse(HttpStatusCode.OK, succesResponse);
    }
}
```

We've deployed the Azure function, let's work through an example to see how easy is it to search for specific issues on GitHub. In the query below, we are searching for open issues under the demo repository `demoext`. 

```
https://vsgeeks-github-depg8-001.azurewebsites.net/api/SearchGitHubIssues?q=state:open+repo:onlyutkarsh/demoext&sort=created&order=asc
```

The result of the search query shown above will look like this.

![Postman]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/postman.png)

What's cool about this approach is that you have full control on the search query, let's get a little adventurous and search for open issues with label "bug" in VSTS Agent GitHub repository.

```
https://vsgeeks-github-depg8-001.azurewebsites.net/api/SearchGitHubIssues?q=state:open+repo:Microsoft/vsts-agent+label:bug&sort=created&order=asc
```

Notice the search expression - `?q=state:open+repo:Microsoft/vsts-agent+label:bug&sort=created&order=asc`

![Search Issues VSTS Agent Repo]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/search-issues-vstsagent-repo.png)

## Monitor your search queries ##

The integration of Azure Function and App Insights let's you track your search queries and how they are performing, see below, app insights records the call's and provides performance feedback and visibility...

![App Insights]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/app-insights.png)

## Integrating Azure function into VSTS as a Deployment Gate ##

Now that we have an Azure function in place to search GitHub, let's see how easy is it to integrate this with VSTS. To configure the Azure Function as a deployment gate in the release definition, in your release definition navigate to the "Gates" section and select "Azure Function" and configure as shown in the screen shot below.

![Pre Deployment Azure Function Gate]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/pre-deployment-azure-function-gate.png)

Please **note** that we are basing the evaluation on the `totalCount` of issues in the returned JSON. In this case accessing that the total count of issues is equal to zero with the below expression.

```
eq(root['totalCount'], 0)
```

In short, when the release is triggered, the GitHub issue search function will be evaluated. The result of the query will be evaluated and if the total count is 0, the gate will be considered passed. However, if the count is greater than 0, the gate will be considered failed. 

## Evaluating GitHub issues ##

Playing the example forward, let's say we have an open issue in the GitHub repo.

![GitHub Open Issue]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/github-open-issue.png)

With deployment gate configured, the deployment gate will periodically trigger (as defined in the sampling interval) to check the response from the Azure function against defined GitHub search expression to see if it evaluates to true. 

Sampling result is shown in the 'Recent gate sampling` section as shown below.

![Recent Sampling Result]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/recent-sampling-result.png)

 VSTS makes it incredibly easy to work with Gates, full set of the logs to see the full results of the deployment gates are available for download. The logs include API calls and the response from the API.

![Log Result]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/log-result.png)

Playing the example forward, once you close all reported issues (based on your search query) as closed, VSTS will proceed with the deployment. And once the release succeeds, you will see the summary like below.

![Deployment Gate Success]({{site.url}}/images/screenshots/utkarsh/github-issues-deployment-gate/deployment-gate-success.png)

Notice, `Recent gate sampling` section is showing us that our second sampling evaluated the expression to true. Hence the deployment went ahead and successfully deployed our application.

## Use our Azure function to search GitHub repo for free ##

We love Azure functions and VSTS! To help you get started with deployment gates we're hosting the GitHub search Azure function that you can use for free to test this functionality out in your release pipeline. Don't forget to give us a shout out on Twitter if you find this useful.

Function URL: `https://vsgeeks-github-depg8-001.azurewebsites.net/api/SearchGitHubIssues`

Just ensure that your search query is as per [GitHub issues API](https://developer.github.com/v3/search/#search-issues).

## Summary ##

VSTS Release Management's deployment gate is a powerful feature, which enables you to seamlessly integrate third-party systems with your release pipeline. We have already many examples like pausing your deployment based on twitter sentiment [more here](https://blogs.msdn.microsoft.com/bharry/2017/12/15/twitter-sentiment-as-a-release-gate/) and Service now integration in your release pipeline in a [post](https://www.visualstudiogeeks.com/DevOps/IntegratingServiceNowWithVstsReleaseManagementUsingDeploymentGate) from Tarun. In this blog post we saw an additional scenario of pausing your deployment while you wait on issues to be closed in your GitHub repo. Let me know your thoughts/suggestions.

