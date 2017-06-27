# How I got started with Puppet, Chef  and Devops ?

As Georg Buchner said:

> We are only puppets, our strings
> are being pulled by unknown forces.

My journey with Devops began with Puppet ( a tool similar to chef, in fact Chef has  from this) back in 2008, when as I was part of the Ops Team managing web scale infrastructure for a SaaS company. Our team consisted of ops engineers in  India and US working out of  their respective timezones, keeping lights on 24x7.  As part of day to day operations, all of us would  make ad hoc changes to the servers, and not always communicate back with the team on the other side of the globe. We did not have daily sync up meetings either. As a result of this ad hoc, inconsistent setup, regularly, issues would pop up with the product, in pre production/integration as well as in production environments. Being in charge of triaging, my team would  spend a lot of time flipping through logs, doing root cause analysis and  figuring out whether its a problem due to inconsistent configurations or a actual code related issue. Tired of this fire fighting, we started looking for solutions to help us efficiently manage this environment. And thats when we came across Puppet, which was popular, was already picked by likes of google, and had an active community around it. We started using it to setup the infrastructure as well as manage changes through a centralized console.

Results were immediate, and tangible. After implementing puppet based configuration management system,

* We now had a centralized tool which streamlined our process of managing configurations. This resulted in minimal ad hoc changes and **consistency** across the environments.

* **Visibility** was another important outcome. Since we started writing infrastructure as a Code, everyone including the developers had **visibility** into the way infrastructure was configured. All one had to do was look at the svn/git repository, the last commits to know what changes were made, my whom and when. What more, developers could even tweak the application properties in their own integration environments.

* **Error rates**, specially related to configurations dropped  significantly, giving us more time back in our day, to focus on scale and other important issues.

That was the beginning of my journey with devops automation tools. Even though Puppet was the first kid on the block, Chef, which was released in 2009 has evolved to be a very mature, powerful and flexible automation tool, with a great community built around it.

Before we start looking into what makes puppet a excellent choice for a automation tool, lets first understand what configuration management is about.


## Infrastructure Life Events and Chef

If you are the one  who is in the business of managing  more than a handful of systems, you should be familiar with the term "Configuration Management" (not to confuse with traditional Software Configuration Management or SCM). Be it physical servers, virtual machines or cloud based setup, infrastructure typically goes through the  following life events,

![infra_lifecycle](images/pictures/infra_lifecycle.png)

* **Provisioning**
  * Provision servers - physical, virtual or cloud. This is where the servers are brought into being.
  * Install Operating System either using manual/automated install process or using an image/template.

* **Configuration Management**
  * Base Systems Configurations:  Prepare the systems with the base configurations such as users,  packages, security configurations, network setup etc.
  * Tech Stack: install and configure services such as apache, tomcat, middle wares, setup cron jobs, install and configure databases etc.
  * Application: Deploy the application code on top of the tech stack configured. This is where the code written by your team gets deployed with relevant configurations.

* **Change Management**
  Configurations made during the initial setup do not remain  do not last for a life time. Infrastructure is in the constant state of flux  and evolves over time. Change management involves,
  * Updating configurations parameters across a class of servers eg. update port that web server is running on.
  * Deploying new versions of the application code, push security patches etc, install additional services.

Chef serves as a excellent tool for Configuration Management as well as Change Management. And along with tools such as AWS Opsworks, Cloud Plugins, Vagrant, Terraform it could also  automate provisioning of servers too. However chef as a tool by itself does have an ability to provision and comes in to play once the Operating System is installed and chef agent is setup.

If you are looking for a tool which could also provision servers, and do it effectively, you should  consider Ansible.

## Evolution of Configuration Management

The need for managing configurations and ongoing  changes had been a challenge which has seen various approaches. Lets have a look at the evolution of configuration management,

### Manual

As systems engineers, we almost always begin configuring systems by hand, in a ad hoc manner. This approach is the easiest, and useful when you have only a handful of systems to manage, simple configurations, and where you do not have repeatable tasks or updates. However, as you start growing, this crude approach quickly gets out of control. It also involves manual processes, which mandates a operator to be present, and is prone to errors or omissions.


### Scripts

Scripts allow one to take a sequence of commands to run and put it in a procedural program. Whenever there is need to repeat the process,  scripts come in very handy. Some of the popular scripting languages amongst the systems personnel include shell/bash, perl, python, ruby or powershell.  Scripts are almost always the first approach towards automating manual tasks. However, scripts are not scalable or flexible enough to manage a sophisticated infrastructure spanning across multitude of environments or the ones involving multiple different operating systems etc.

### Configuration Management and  Software Configuration Management

Software Configuration Management is referred to often with context of
Revision Control which is about tracking changes to the application code.
This is just one part of a larger field of Configuration Management.  


### Golden Images/ Templates

Golden images, or templates, or simply os images, are probably the quickest way to deploy servers complete with configurations, specially in virtual or cloud environments. Images are nothing but pre baked templates with operations system files, applications, and configurations. Take any cloud provider, and one of the first components to choose while you provision a server is the images. A lot of organizations these days package their products in the form of images such as ova, vmdk or even vagrant's box format. However images have one major problem i.e. change management. Every time there is an update, even a single line change,  one needs to build a new image. Not only this complete system image needs to be distributed but also existing systems need to be replaced with the new image. Imagine doing that in a dynamic environment involving frequent updates across  hundreds of servers. That could get too cumbersome. And thats where the need to come up with a new approach.  

### Infrastructure as a Code

"Infrastructure as a Code" or "Programmable Infrastructure" is where today's generation tools such as Puppet, Chef, Ansible, Salt fit in. These tools essentially allow one to write the state of the infrastructure using a higher level descriptive  language and store it as a code. Since this is a code, one could bring in the best practices that developers have been following for years e.g. using revision control systems, use of sophisticated editors, test driven development, peer programming etc. You could  even build the complete infrastructure from scratch in case of a disaster, as long as you have the code repository, compute resources and data backups in place. Since this code is written in a simple declarative syntax, it is self documenting and offers visibility to all stakeholders into the way infrastructure is built and configured.

![config_mgmt_approaches](images/pictures/config_mgmt_approaches.png)


| Approach     | Advantages     | Disadvantages
| :------------- | :------------- | :------------- |
| Manual | simple    | ad hoc, error prone, inconsistent, not repeatable |
| Scripts | repeatable, automated     | procedural, not scalable, inflexible |
| Images | repeatable, automated   | size, change management is not easy |
| Infrastructure as a Code | repeatable, flexible, scalable, automated, consistent     | agent based, learning curve  |


## Why to use chef ?

Now that we have started discussion on   Infrastructure as a Code, specifically chef, lets discuss about the specific features of chef that make it a useful tool for configuration and change management.

## Declarative vs Procedural Approach

Scripts take a procedural approach towards automation. With scripts, we focus on the **how** part. e.g. how to install a package, how to create a user, how to modify it later, how to do it on a specific platform. And if you would want to add a support for another platform, you may have to add additional procedure and write a conditional to check for the platform and call the relevant code. This involves a lot of efforts.

On the other hand, chef takes a declarative approach towards automation. With chef, our focus changes from   **how** part to **what**. Instead of writing procedures, we start using a simple declarative syntax to define the desired state of infrastructure.  Let me explain this with an analogy.

Lets say we want to build a house. When we  set out to do so, we hire a contractor, who in turns has a team of construction workers who know how to build it brick by brick. They do all the hard work to bring our house into a reality. Thats the *how* part.

On the other hand, you have a Architect. Lets take a moment and think about what he or she does. An Architect creates a plan, a blue print, which is nothing but a description of how your house should look like. The architect envisions the end state of your house, thinks about *what* should it consists of, breaks it down into components,  and starts creating  specification for each. He would then stitch it together to create a blueprint of our house, and hands it to the contractor to go and build it. This is the how part.

Thats exactly what we would start doing with chef, only difference being, we write the specification for desired state of IT infrastructure.

## Resource Abstraction Layer

We just learnt about chef allowing us to write the desired state of the infrastructure using a descriptive language. Now  this concept is brought into a reality with the its Domain Specific Language(DSL) which consists of a Resource Abstraction Layer (RAL). Lets describe how this works,

*TODO*: Create an image for each step below.

* chef looks at  infrastructure as a collection of entities to manage e.g. package, service, user, network interface etc.
* It then takes the  procedures, the actual logic to manage these entities and  bundles it into something called as **Providers**. Providers are platform specific. That way, each entity may have multiple providers e.g. managing user on linux vs osx vs windows.
* On top of these procedures, chef creates an abstraction layer. Instead of defining the procedures, it offers the users a simple declarative syntax to define the state of the entity and its properties, in the form of **Resources**.
* chef, then does the translation between Resources and Providers, and calls the procedure to put the entity in to the desired state just as described by the user.

This behavior allows the users of the chef to create policies stating what which entities should be present or absent, with what properties.

## Convergence and Idempotence

chef reads the resources, calls the relevant providers for the platform it runs on, and ensures the desired state of the resource is achieved. While it does so, it may need to make changes to the system. But what if the resource has already achieved the desired state and needs no further updates?  chef has the  built in intelligence to know what is the current state of the resource is. Instead of making changes blindly, it first compares the current state of the resource with the desired state, and the makes a decision whether it requires any changes, and if yes, what changes to make.  This process is called as convergence.

*TODO*: Draw diagrams, maybe just a flowchart

e.g.

Lets assume we have written a resource to create  a user  with a password.

```
user 'xyz' do
  uid   '501'
  gid   '501'
  home  '/home/xyz'
  password  '$1$foYKL0zO$elXbUOb/JjHqS4aI8O25i.'
end
```

*  First time chef applies this resource on the system, the user may not exist. chef detects the current state, compares it with the desired state which mentions it should exist, and finds the configuration drift. It then creates the user with the password to bring the current state to desired state.

* Lets assume we run chef again to apply the same resource. This time too it will compare the desired state with current state. Since the user is already present with the password provided, it deems no changes necessary. Instead of attempting to create the user again, it will skip the resource and move on to the next one.

* Lets assume we updated the password information for the user in our code description of the resource, and excute chef. This time, while comparing the current state with desired state, it detects that the user is present, but the password is updated. It does not attempt to create a user, but only changes one property i.e. password.

Idempotence is a useful property to  ensures that chef maintains the state of the infrastructure such that its always  in the policy. This also makes change management easier as one could keep running chef as a service which invokes itself at regular intervals, pulls the changes, and apply only what is changed.

## Centralized Configuration Management System

A typical installation of chef involves  a chef Master, which is a centralized management console and Agents runnings on every node managed.  Any changes to nodes have to go through the chef Master. This streamlines the process of pushing updated.  Instead of iterating over a list of hosts using a for loop or other such methods, or logging into each system to make changes, all you need to do is push updates to chef Server, from where those are automatically propogated to the nodes.

*TODO* ![centralized_management]()

### Pull Approach

While chef offers a centralized management approach, it works unlike most client server schemes. Instead of pushing updates to the nodes from master, its the duty of chef agent to go the master,  pull changes, and apply. Pull method offers more flexibility and scalability for the following reasons,

* Each node could decide how frequently or infrequently it should update. Some systems need frequent updates, where as others need not be updated for weeks. This could be controlled at  per client level.

* No need to manage inventory and connection details on the master. Nodes could come and go, and could be configured based on dynamic rules to classify them based on certain property e.g. host names, environments, or even hardware addresses.

* With push based approach, master must have a ability reach out and connect to each node managed in order to configure it. A lot of times you may have a master on a cloud or in a separate data center than nodes being managed, which could be behind a firewall or NAT device in a private network. In case of pull method, as long as the master is available on a well known address, and is reachable from the nodes, configurations can be pulled and applied.

## Code vs Data

A lot of applications that we configure on our servers are generic eg. apache, tomcat, mysql, mongo. These applications have a install base of hundreds of thousands of servers, used by organizations across the globe. Ever wondered how these become specific to your environment ? Even though you use the same apache web server used by many others, its how you configure it makes the difference. Even in single organization, you may have a apache server which behaves differently in different environment based on the configuration profile created. The process of setting up an application involves,


* Installing the generic application either from a source, package or a repository.
* Adding data. This where you set configuration parameters  e.g. port, user, max connections, webroot for apache.
* Start, enable the service.

chef allows separation of  code and data.

* Using chef's DSL, we write infrastructure code to install packages start services etc. Code is generic.
* chef's attributes templates, along with dat bags and attribute precedence rules provide a way to create different configuration profiles specific to different nodes, environments, platforms etc.

## Shared Library of Infrastructure Code

With chef's ability to separate code and configuration data, the declarative code that you write in the form of modules with  chef becomes generic enough to be shared and be reused. And since this code is in the version control, hosted services such as github offer the perfect means to publish this code. As you start writing code with chef, you would discover about chef forge, a library of community written modules. At the time of writing, there are more than 4000 modules available on chef forge. Similar to a lot of open source code, you need not re invent the wheel. For most open source applications, you would find very sophisticated modules which you could use without even single line change. Download the modules, install it on your own chef master, add configuration profile, and off you go.

## Cloud Integrations

With emergence of cloud, computing is moving towards the utility model. More and more organizations have already migrated or contemplating to migrate a partial or whole of their computing setup to cloud. And with that happening, which ever automation tool that you would consider, needs to have a close integration with the cloud platform that you plan to use. Provisioning of infrastructure components before chef comes in and starts configuring  There are two ways chef provides a way for this,


1. Through library of custom resources/libraries  which allow one to write provisioning of cloud components in the form of resources.

2. Cloud specific tools e.g. Cloudformation on AWS or Heat on Openstack are some of the tools which help provision components which are specific to that cloud and then call chef to do the configurations.

3. Third party tools such as Vagrant, Terraform have in turn ability to talk to multiple cloud providers. These tools typically provision servers on the cloud and then hand it off to chef for configurations.

## Iterative Approach to Automation

However convinced you are with chef, scrapping  your existing automation tool  overnight for a shiny new tool may have  unknown risks attached. More over, it may could  be challenging  to get a buy in from your management to invest in time, money and resources to implement a new solution without showing them the value.

With chef, you could take a iterative approach towards automation. You could start with a single application, or even a single entity such as a system user to manage. Once you see the value, show it to all stake holders, get a buy in and move to the next application, one at a time.

## Device Support

Managing configurations on systems running common operating systems such as windows, linux, os x, where chef agent application  can be installed is easy.  Its a completely different beast when it comes to   managing configurations on  devices running their own specialized version of operating systems/firmwares. Examples are configuring CISCO's network switches routers, EMC's storage array.

Along with programmable infrastructures, new concepts such as Software Defined Networking(SDN) and Software Defined Storage(SDS) are taking root and changing the way devices are being managed. chef supports managing devices in two ways,

1. Devices that are based on linux and have chef agent ported to run on those can be configured the same way as other generic systems.
2. Sub set of devices that do not have chef agent can still be configured with chef's device support over ssh/telnet. For such devices, chef uses push instead of pull approach.

## Audit and Compliance

With ability to define the state of the  infrastructure components as a code and then  converge, one could easily codify the infrastructure policies and have them enforced. chefs ability to test, log changes, and reporting mechanisms  help keeping a trail of the state of the systems and its components over time and track who made changes, when, if there are any nodes which have fallen out of policy etc.

## When to Use chef ?

You should consider using chef for the following purposes,

* **Configuration Management** + Change Management: You have many nodes to deploy with changes happening often. You need to update the nodes and applications running on those often.
* **Compliance and Audit**: When your organization has to comply to policies and you need an ability to convert those policies into a code which would auto correct and bring the nodes into the policy in case of configuration drifts. You also need to audit the systems regularly and prepare reports to find out which nodes have drifted away from the policy etc. as well as mitigate such issues.
* **Software Delivery** : If you are in business of building software and delivering as ova images or similar, chef is better approach to deliver the product and push updates to it.

## Who is it for?

* Systems Administrators/Engineers who are managing systems at scale and  need to install, configure, patch, monitor and maintain systems and prepared reports for.
* Application Operations Engineers  who are responsible for installing, configuring, integrating, monitoring, maintaining application infrastructures.
* Build and release engineers who are in charge of setting up environments for CI/CD cycles, as well as deploy/release applications to production environments.
* Network Engineers who configure and maintain networking devices such as CISCO Routers and Switches at scale.
* Storage Administrators who configure and maintain storage devices.
* Developers who are building application delivery to their customers. Also as users of the develops tools, developers may need to change application properties for  the environments that they create/use.  

## What chef is not?

* Graphical Management Tool (e.g. SCCM):
  If you are looking for a tool which would allow you to manage everything through a graphical interface without writing any code or without ever having to use an editor, well chef is not the tool for you.  I have come across many engineers, who say "well, the capabilities of chef sounds great, but can I do all of this using a GUI where I can just click click and get things done.... ? " Well, its infrastructure as a code is what we are talking about. Even though enterprise chef chef offers a nice GUI, its mostly for reporting and classifications. Since I started using chef and similar tools, I have been using text editors more often.

  * Automated Testing Tool ( e.g. selenium ):
  Its not a silver bullet. Its not one solutions to all. I meet a lot of QA folks who have heard that chef would automate everything and you could use it for testing too.  Well, there is always a special purpose tool for each task and the application testing is not chef's ball game.  Sure chef could help in testing by letting you automate the process of building and configuring a fresh environment to run your tests inside, and give you ability to do it repeatedly. It also gives you a way to test infrastrcuture code. However, its not a test automation tool.

  * Pure Application Deployment and Orchestration Tool:
  Even though chef has been talking about application orchestration, if you are looking for a tool purely for application deployments, rolling updates, canary releases, orchestrated deployments over multiple hosts, you have tools which do it better. A few  tools I could suggest you for application deployment and orchestration are Asible, Capistrano, Code Deploy  which are push based, and work better in such scenarios.

  * Agent less Management System:
  Except for a sub set of network devices, chef mandates running agents on each node being managed. In fact its designed to be heavy on the agent side which is responsible to initiate communication with the master, pull policies,  enforce and report back.  If you need a agent less management system, chef is not the one. Again, I would suggest using Ansible in such cases, which works over ssh and is agent less.

  * Software Configuration Management(SCM) Tool
  SCM is a part of the larger Configuration Management and typically refers to the practice of revision/version control. chef is not the tool which does the version control, however it can be used to replicate and manage software configurations across a cluster of nodes.

  * A one stop  Devops solutions

  * chef Use Cases/ Customer Stories

## Chef vs Puppet

## chef vs Ansible

## chef vs Docker
