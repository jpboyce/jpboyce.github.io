---
draft: false
date: 2023-12-04
categories:
  - Azure
tags:
  - azure front door  
---

# Rendering Issues with Nodejs/NextJS and Azure Front Door

Recently at my workplace, a new application using Node JS and NextJS was implemented. As with all our public facing websites, it was placed behind Azure’s Front Door service, to provide web application firewall (WAF) and caching functionality.

During testing, it was discovered that the site would sometimes not render properly. However it wasn’t a 100% failure rate.

## An Early Theory – Geography
Early on a common theme was noticed. If the user device was located in Brisbane, regardless of OS, browser or ISP used, the site would fail to render. If the device was in or close to Sydney, it would render properly. Trying a few other geographical points using VPNs showed similar behaviours.
<!-- more -->
## The Mysteries of Front Door
It turns out that Front Door uses a mix of Windows and Linux based systems for its infrastructure. There are subtle differences in behaviour between these systems. Forunately Microsoft have made it somewhat easy to figure out what you’re getting by looking at the X-Azure-Ref header. One will start with a datetimestamp style string and the other will appear to be completely random.

|   |   |
|---|---|
|Header Type 1 | ![Header1](../media/header1.png)|
|Header Type 2 | |

In the case of this issue, the header for systems in Sydney had one type and the systems in Brisbane (and other locations where the site wouldn’t render) had the other type. However there was clearly something in the new app that caused this behaviour as our other applications, written in another language, didn’t have these issues.

## The Application’s Contribution
It turns out the application has a particular behaviour that was causing one type of Front Door node to freak out and be unable to serve content properly. When a request is made, the application would return a content range header, indicating the size of the returned data. However this would often not match the actual size of the data returned. In some situations, this is normal behaviour and would cause a HTTP 206 (Partial Content) response code, with the remaining content requested in further responses.

In this particular situation, one type of the Front Door node couldn’t cope with this and wouldn’t return the data to the client at all. This would cause the site to not render properly.

## The Fix
There were two options presented to fix this. Firstly, update the application code to cause the content range header to match the actual content size. This was deemed very difficult, if not impossible to achieve. The second option was to add a rule on Front Door to strip the Accept-Encoding headers from requests. Since being implemented, this rule has prevented the issue from happening again.
