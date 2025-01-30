---
draft: false
date: 2018-12-11
categories:
  - vRealize Orchestration
tags:
  - vrealize orchestration
---
# Creating Service Accounts with vRealize Orchestrator

vRealize Orchestrator (vRO) has a lot of plugins that allow it to integrate with other systems and services.  One of such plugin is for Active Directory.  This plugin allows you to perform a number of standard AD activities, like creating users.  vRO already has built in workflows to create and manipulate users.  In this post, I’m going to run through what you might end up implementing if you wanted to be able to create Service Accounts via vRO.
<!-- more -->
## The Scenario
Some questions may come up as to why you would need to create a workflow to do this.  One question might be why we would want to create Service Accounts via vRO.  There may be times when you need a Service Account for a new system or service (such as SQL Server) being deployed via vRO.  Rather than use the same shared Service Account, your organisation may prefer to use individual ones.

The second question that may come up is why not use the existing Workflows.  These Workflows may achieve the core goal of creating the Service Account in Active Directory but they don’t achieve the nuance of what might be required without some extra code.  Using some of the organisations I’ve worked at as an example, some of these extra bits may be:

* The password needs to be significantly longer
* The password needs to be stored in some sort of password or secret management solution so other administrators can reference it
* Service Accounts have to be created in certain Organisational Units

Based on these sort of factors, the out-of-the-box workflows can’t fully achieve what’s required.  Also, you may want to take some control or work out of the process (like removing the choice of target Organisational Unit or the generation of the password).  Lastly, you may want to put some extra protections or controls in place, like checking if the requested Service Account name exists already.

Based on these factors, it would be quite reasonable to have a request or user story that outlines the following:

* The Workflow allows the Service Account Name to be specified
* The Workflow checks to ensure the specified name isn’t already in use
* The password for the Service Account is randomly generated and meets organisational requirements
* The password for the Service Account is saved to our password/secret management system
* The account is created in a predetermined Organisational Unit

## Design and Building Blocks
Even if the existing Workflows aren’t suitable for our purpose, there are pieces we can take from them.  The Workflow “Create a user with a password in an organizational unit” is very close to the functionality we want and the core of this Workflow is the Action called “createUserWithPassword“.  The parameters for it appear to cover the bare essentials for what would be required.

![Image](../media/2018-12-11-001.png)

When viewing the script that makes up the Action, we can see it uses a .createUserWithPassword method.  There’s a comparable method called createUserWithDetails that adds parameters for First Name and Last Name.  With this as the core of the Workflow, we can start thinking about how the overall Workflow might look.

![Image](../media/2018-12-11-002.png)

With a flowchart like this, we can see there’s two main pieces that need building.  Generating the random password and adding the password to the password/secret management system.

## Generating A Random Password
Since vRealize Orchestrator uses JavaScript for its internal scripting, an easy approach for this piece could be to take an existing JavaScript password generator and adapt it.  I’ve done this and wrapped it up as an Action, with passLength as an input.
``` javascript
// Generates a random password of a specified length and returns it
// Heavily inspired by the password generator at Sololearn.com by Marcel Feenstra (https://code.sololearn.com/WMLm7mRuVroc)

// charSet is the characters to be used in generating the password.  If these characters aren't suitable for your needs, change it
var charSet = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!\"#$%&'()*+,-./:;<=>?@[\\]^_`{|}~";
var passWord = "";
var charPos = "";

for (i = 0; i < passLength; i++) {
	charPos = Math.floor(Math.random() * charSet.length);
	var passChar = charSet.charAt(charPos);
	passWord += passChar;
}

return passWord;
// END
```

## Adding to Password/Secrets Management System
A lot of password management systems have API support these days and this allows the Workflow to pass the generated password and other details over to the management system to store.  In the case of my sample workflow, I’m sending it to Password Manager Pro as it’s a system I’ve used in the past.  The only real difference between one system to the next will most likely be the exact format you need to send the data in the API call.

## Adding Some Safety Features
One of the issues that comes into play when publishing a workflow like this is it will be consumed by a broader audience than the group of people who were performing this task manually previously (this is really part of the original reason for making this workflow in the first place).  A consequence of this is the need for “safety” – putting safeguards in place to ensure the workflow doesn’t break anything.  One such safeguard we might put in is to check that the service account specified doesn’t already exist.

To achieve this, we might consider a simple bit of code that returns a true/false on the search and act accordingly.  Fortunately, vRO already has the core bit of code for this via the Action “getUserFromContainer“.  The Action will either return the user as an AD:User object or a null.  Assuming all your service accounts are living in the same Organisational Unit structure, this Action is a good option for performing this check.  An example of how this could behave if implemented is shown below.

![Image](../media/2018-12-11-003.png)

Another safety check to include may relate to the service account name itself.  Your organisation may have a particular format for the account names (using a prefix like svc is common).  This could either be performed as a regular expression check on the workflow’s presentation elements or as part of the workflow.

## Bringing It All Together & User Experience
With all these building blocks, we can now put them together into a Workflow.  The example Workflow I created looks like this:

![Image](../media/2018-12-11-004.png)

When executed, the requester is prompted for the username and a display name.  Below is the user experience in the vRealize Automation (vRA) Catalog.

![Image](../media/2018-12-11-005.png)

Directly exposing a vRO workflow as a vRA Catalog item doesn’t always carry across Presentation behaviours exactly.  For example, the user name checking I created and associated in the vRO Workflow doesn’t work in “real time” with the vRA Catalog Item.  However, it will execute when the request is submitted and behave correctly (it will block the request if the user exists) as shown below.

![Image](../media/2018-12-11-006.png)

The end result of the Service Account object in Active Directory and the entry in Password Manager Pro can be seen below:

![Image](../media/2018-12-11-007.png)

By using this sort of approach, it’s possible to delegate the ability to perform a task like creating Service Accounts, while also maintaining some degree of control and create an audit trail of the requests.
