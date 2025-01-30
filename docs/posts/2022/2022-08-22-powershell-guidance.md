---
draft: false
date: 2022-08-22
categories:
  - Powershell
  - Security
tags:
  - powershell
  - powershell security
---

# Security Agencies Issue PowerShell Guidance

PowerShell has historically presented some challenges with security. This can lead to the temptation of disabling it outright, removing an avenue of attack opportunities. Recently, [a number of government security agencies issued recommendations](https://www.itnews.com.au/news/dont-remove-powershell-us-uk-and-nz-security-agencies-581763) relating to PowerShell.
<!-- more -->
## A Two-Part Approach
The recommendations from the agencies can be split into two groups – methods to prevent PowerShell being used and abused by malicious actors, and methods for increasing visibility of abuse attempts.

Some of the options in the first group include network controls around PowerShell Remoting (such as appropriate rules in Windows Firewall). Another option proposed is to make PowerShell run in Constrained Language Mode (CLM). CLM forces a number of restrictions on the use of PowerShell, including restricting access to only approved .NET types.

The second group of options involve increasing the logging of PowerShell activities. These items actually surprised me. In the past when working on Windows hardening, the CIS Benchmarks have recommended against this sort of logging. When reviewing the latest versions of the Benchmarks, it seems they’re aligned with the advice from the government agencies, with some logging to be enabled.

Overall, these recommendations are a good start in securing PowerShell in an organisation.
