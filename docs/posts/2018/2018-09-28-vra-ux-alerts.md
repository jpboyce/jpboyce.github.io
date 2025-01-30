---
draft: false
date: 2018-09-28
categories:
  - vRealize Orchestration
tags:
  - vrealize orchestration
---
# Improving the vRA Admin Experience – Reservation Alerts to Slack
The Reservation system in vRealize Automation (vRA) provides a bucket of resources to a team or business unit via Business Group.  A risk with Reservations comes about with how I think VMware intended them to be used vs how some organisations may use them.  I suspect VMware’s intention was that Reservations should be self-managed by the Business Group associated with it.  This makes sense if each individual team has a Business Group as the scope of what’s in the Reservation is “their stuff”.  It would mean if a Reservation reached capacity, it would be up to that team to manage the situation.

What if the Business Group was being used differently, where it covers multiple teams?  In the event of the Reservation becoming full, the scope is larger than one team.  In this situation, it might be good to get a heads up on when Reservations are running low on resources.  Email alerts can be setup and yes, sent through to Slack, the formatting in Slack is less than desirable.  So I decided to look at a way of doing it better.
<!-- more -->
## Re-using The Wheel
In the [entry](https://blog.jpboyce.org/2018/09/19/vra-customer-experience-send-chef-errors-slack/) about sending Chef errors to Slack, I had written a [generic vRealize Orchestrator (vRO) workflow](https://github.com/jpboyce/vro-resources/tree/master/send-slack-message) that can send a message to Slack.  Since that part of things exist, all I need to do is get the Reservation information out of vRA.

## Getting Reservation Data – Approach Options
There’s a few ways to possibly extract the information about Reservations in vRA.  When looking at the API Explorer inside vRO, there is references to Reservation objects.  When I looked at using this approach, I found that it wasn’t exposing the information in a easy-to-use fashion.  There was hopping from one object type to another, with the actual information stored in a format that required a little too much effort to deconstruct.  The vRA API exposes Reservation actions directly.  Specifically, by using the [/api/reservations/info](https://code.vmware.com/apis/387/vra-reservation#!/default/get_api_reservations_info) action, it will return summary information about Reservations (using [/api/reservations](https://code.vmware.com/apis/387/vra-reservation#!/default/get_api_reservations) returns a lot more detailed information than I needed).

## Introducing the “vCAC” REST Client 
Using vRA’s REST API would imply all the hoop-jumping required, like getting your bearer token constructed and so on.  Fortunately, vRO can leverage an internal REST client of sorts.  By using the vCACCAFEHost object of your vRA server, it’s possible to leverage a createRestClient method, which allows a REST client to be easily created.
``` javascript
// Find the VCACHost by its ID
var vcacCafeHost = Server.findForType("vCACCAFE:VCACHost",vcacHostId);
// Define the endpoint
var endpoint = 'com.vmware.vcac.core.cafe.reservation.api';
// Create a REST client with this endpoint
var restClient = vcacCafeHost.createRestClient(endpoint);
```

Once this is done, it’s just a matter of feeding in the sort of things we normally would for a REST call.  This process is easier than a regular REST API call as we don’t have to worry about JSON payload formatting or similar issues.
``` javascript
// Define the resource URL
var resItemsUrl = "reservations/info";
// Perform a GET request using the resource URL
var resItems = restClient.get(resItemsUrl);
// resItems is a vCACCAFEServiceResponse object type, with a .getBodyAsString method that gets the response body as a string response
var response = resItems.getBodyAsString();
// Parse the response into a JSON object
var json = JSON.parse(response);
```
Once the response has been returned, it’s a matter of iterating through it and checking values against a threshold.  If the threshold is exceeded, an alert message text is constructed and sent to Slack via the Slack Message workflow.
``` javascript
// High memory condition
System.log("Too much memory allocated, sending slack message!");
// Setup Slack items
var slackwfMemAlloc = Server.getWorkflowWithId(slackWorkflowId);
var slackwfMemAllocProperties = new Properties();
var slackMessageMemAlloc = ":rotating_light: WARNING: *" + tmpResMemAlloc + "%* memory allocation on reservation `" + x.name + "`! :rotating_light:";
slackwfMemAllocProperties.put("slackMessage",slackMessageMemAlloc);
var slackwfMemAllocToken = slackwfMemAlloc.execute(slackwfMemAllocProperties);
```
The resulting output is shown in the screenshot below.

![Image](../media/2018-09-28-001.png)

Since the Slack message format allows a reasonable amount of formatting in a syntax that doesn’t break vRO’s javascript, you can customise the message output to your requirements without worrying about things breaking.
