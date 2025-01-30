---
draft: false
date: 2022-06-27
categories:
  - Azure Devops
tags:
  - azure devops
---
# “Could not create SSL/TLS secure channel” error when using self-hosted Azure DevOps Agent
Recently the team I’m in has been getting into Microsoft’s new Bicep language. As part of a release pipeline, the infrastructure was being deployed – in this case an Azure App Service. Then the application code was being deployed using the standard “Azure App Service Deploy” task. At this particular task, it would error out:

`Error: Could not complete the request to remote agent URL 'https://<App Service Name>.scm.azurewebsites.net/msdeploy.axd?site=<App Service Name>'.
Error: The request was aborted: Could not create SSL/TLS secure channel.`

The pipeline was being run through our “Default” agent pool, which was a self-hosted agent.
<!-- more -->

## Troubleshooting
The first thing I tried was to recreate the scenario using the Microsoft-hosted agent. When running the process using their agent, it worked. The error message strongly hinted that the issue was related to TLS settings somewhere.

When looking at the Bicep code, I noticed when doing a “what-if”, it would set the TLS minimum version for the SCM interface from 1.0 to 1.2, even though this setting wasn’t actually defined in the template. When looking at the ARM templates generated from existing App Services, they all listed this setting at TLS 1.0 as well.

The next thing I did was go back to the Bicep template and explicitly set the minimum TLS version fot the SCM interface. First I tried 1.0 and the deployment succeeded. Set to 1.2 and it failed. So this established there was a TLS problem somewhere.

The agent VM was running Windows 2019 and thus, should support TLS 1.2 by default. The only registry settings controlling encrpyption settings were to disable SSL 3.0. Even after explicitly enabling the TLS 1.2 to enable in the registry, the deployment still failed.

Those who have dealt with TLS settings in Windows before will know there’s a setting for which TLS versions at supported at the OS level, and settings for instructing .NET to use strong encryption. By default, this strong encryption setting isn’t enabled, which causes .NET applications to use lower levels of encryption.

After enabling the strong encryption setting on the self-hosted agent, the deployment started working.

## Lessons Learned
A couple of items come to mind out of all this:

* Agent configuration – It’s important to keep your self-hosted agents’ configuration aligned with that of the Microsoft hosted agents
* Bicep behaviour – It’s worth running a “what-if” with a Bicep template after it’s initial deployment to see if any values are magically appearing or changing themselves outside of what you’ve defined in the template.
