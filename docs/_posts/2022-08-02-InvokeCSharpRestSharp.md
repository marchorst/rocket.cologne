---
layout: post
title:  Invoke C# with RestSharp
tags: rpa invoke invokec# c#
---
Example on how to use RestSharp with Invoke C# Activity
```
#r "Path\to\RestSharp.dll"
using RestSharp;
using System;

var client = new RestClient(inUrl);
client.Timeout = -1;
var request = new RestRequest(Method.POST);
request.AddHeader("Content-Type", "application/json");
request.AddBody(inBody);
IRestResponse response = client.Execute(request);
Console.WriteLine(response.Content);
```
