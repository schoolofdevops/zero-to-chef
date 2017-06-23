# Cookbook and Run_list

In the previous chapter we gone through resources and recipes, now let us go through Cookbook and Run_list in this chapter.

## What is Cookbook?

- Cookbook contains configuration and policy which are created using Ruby as its reference language.
- Cookbook is created with desired scenario in mind and contains required components to support the final scenario of a system.
- Cookbook contains collection of various components as follows,

|Components|Description|
|:---|:---|
|Recipes|Collection of resources in a single file is called recipes.<br/>Recipes are stored in a cookbook.<br/>Recipe must be added to a run_list before it can be used by the chef-client.<br/>Is always executed in the same order as listed in a run_list.
|Attributes|Attributes are used to override the default settings of the node.<br/>For each cookbook, attributes in the `default.rb` file are loaded first, and then additional attribute files (if present) are loaded in lexical sort order.
|Files|Distributing any type of files to nodes, these are static files|
|Templates|A template is a file written in markup language that uses Ruby statements to solve complex configuration scenarios, and to update dynamic values.|
|Cookbook Versions|A cookbook version is used to maintain a various version of same file with different functionality.<br/> A version may exist for many reasons, such as ensuring the correct use of a third-party component, updating a bug fix, or adding an improvement.|

### Community Cookbooks

**Chef Maintained Cookbooks**

Chef maintains a collection of cookbooks that are widely used by the community.

**Community Cookbooks**

The community has authored thousands of cookbooks, ranging from niche cookbooks that are used by only a few organizations to cookbooks that are some of the most popular and are used by nearly everyone.

## Chef Generate

- `chef generate` is a part of chef-dk and this command is used to generate a set of file system. Which can be used as a template.
- To know more about _chef generate_ you can check with the man page,

```console
chef generate
Usage: chef generate GENERATOR [options]

Available generators:
app             Generate an application repo
cookbook        Generate a single cookbook
recipe          Generate a new recipe
attribute       Generate an attributes file
template        Generate a file template
file            Generate a cookbook file
lwrp            Generate a lightweight resource/provider
repo            Generate a Chef code repository
policyfile      Generate a Policyfile for use with the install/push commands
generator       Copy ChefDK's generator cookbook so you can customize
build-cookbook  Generate a build cookbook for use with Delivery
```

### Chef Generate -  App vs Cookbook

There are two methods to create code. Traditional approach has been to generate one repository per cookbook. The new approach is to generate one repository per Application, with cookbooks for each component e.g. a repo for web-app with cookbooks for tomcat, java, mysql, deployment.  

This method also uses a common test kitchen environment etc.

To find more information on comparison between App and Cookbook approach, refer to [this blog](http://devopsguru.tumblr.com/post/147717124737/chef-generate-app-vs-chef-generate-cookbook-vs)[^devopsguru_blog].

Example of generating a app is

```console
chef generate app <app_repo_name>
```

Example of generating a single cookbook  is

```console
chef generate cookbook <cookbook_name>
```

## Generating App and Cookbooks to setup Webapp

Let us generate a repo for our application named "myapp".

* Starts with creating a `sysfoo` repo for our application.

```console
chef generate app sysfoo
```

* Let us create a first cookbook for our tomcat application.
* Inside app repo create a cookbooks for `tomcat` along with its prerequisite cookbook `java`

```console
cd sysfoo

chef generate cookbook cookbooks/java

chef generate cookbook cookbooks/tomcat
```

* App repo and cookbooks are created.

### Java Cookbook

Repo for java cookbook is generated now add resources to our cookbook.

* create a recipe to install `epel-release` and `java-1.7.0-openjdk`.
* In `myapp/cookbooks/java/recipes/default.rb` create a default recipe with the following content.

```ruby
package 'epel-release' do
  action :install
end

package 'java-1.7.0-openjdk' do
  action :install
end
```

## LAB Exercise - Tomcat Cookbook

* Cookbook for `tomcat` is already been generated .
* Add one recipe `install.rb` to **install** `tomcat` and `tomcat-webapps`.
* Add another recipe `service.rb` to **start** a `tomcat service`.

## Creating Local Environment to Test the Code

We have created cookbooks for Java and Tomcat and written reipes to install and configure tomcat. Before we uploade this code to the Chef Server and apply it at scale, its important that we test these recipes locally. Every subsequent change to the recipes need to be tested as well.

We may also need to test our code on multiple different platforms. Test kitchen offers us a way to create local test environments, apply the code and also execute automated tests to validate that our code works.

It could be used for
* Local development to create portable, use and throw test environments.
* Functional and Integration tests which could be triggered automatically as part of Continuous Integration environments.

###  Creating Test Kitchen Configuration

Test Kitchen comes with a configuration file .kitchen.yml . We have one file for each App and cookbooks. We would edit our top level .kitchen.yml file  available at sysfoo/.kitchen.yml

Make sure it matches the following config

```ruby
---
---
driver:
  name: docker

provisioner:
  name: chef_zero
  always_update_cookbooks: true

verifier:
  name: inspec

platforms:
  - name: centos-6.8
    driver_config:
      image: codespaces/chef-node-centos-6
      forward:
        - 8080:8080

suites:
  - name: default
    run_list:
      - recipe[sysfoo::default]
    verifier:
      inspec_tests:
        - test/recipes
    attributes:

```

Once created, check the current status of the environment by running ,

```console
cd /workspace/sysfoo/
kitchen list
```

TIP: Use yamllint.com to validate .kitchen.yaml if you get an error after running ``` kitchen list ```



### Create a Local Environment with Docker

Local environment for testing is created using docker, we will be uing ".kitchen.yml" for creating a test environment.

* Once the **.kitchen.yml** is updated we can create kitchen using `kitchen create` command, from the directory where *.kitchen.yml* file exists.

```console
kitchen create
```

* It creates a docker container with `centos-6.8`.

```console
kitchen list
```

* To list the available instances and their information.

### Adding recipe to run_list

Once the test environment created we need to add recipes to the run_list for testing it.

* Add both java and tomcat recipes to run_list in `.kitchen.yml`.
  * java::install
  * tomcat::install
  * tomcat::service

```ruby
suites:
  - name: default
    run_list:
      - recipe[java::install]
      - recipe[tomcat::install]
      - recipe[tomcat::service]
```


### Apply Chef Cookbooks

* Once the instance is created it can be converged along with the `run_list` specified in `.kitchen.yml`

```console
kitchen converge
```

* It will `install chef` and then will apply run_list to the instances.
* To verify, Login and check for java installation and version.

```console
kitchen login
```

* Logins to docker instance created by kitchen.

```console
which java
java -version
which tomcat
service tomcat status
```

### Destroying and Converging

* If you wish to re create the environment from scratch, use kitchen destroy followed by kitchen converge

```console
kitchen destroy
kitchen list
kitchen converge
```

* Run converge command directly to `create` a instance and then `converge` it and apply `run_list` to it.

**Verify** the converge by visiting `http://ipaddress:8080` for tomcat homepage.

![tomcat-homepage](images/pictures/04_2.png)

## Simplifying Run_list

* Lets make recipes simplified.
* Call all other recipes from `./myapp/cookbooks/tomcat/recipes/default.rb`

```ruby
#
# Cookbook Name:: tomcat
# Recipe:: default
#
# Copyright (c) 2017 The Authors, All Rights Reserved.

include_recipe 'java'
# we can also call the above recipe as
# include_recipe 'java::default'

include_recipe 'tomcat::install'
include_recipe 'tomcat::service'
```

* Now change the run_list in `./sysfoo/test/.kitchen.yml` and add only `tomcat::default`

```ruby
suites:
  - name: default
    run_list:
      - recipe[tomcat]
```

* Once added, now converge again.
* You could get error because of java cookbook not found, but tomcat includes java in run_list.
* Now to add `depended java` add entry in `./myapp/cookbooks/tomcat/metadata.rb` for the java dependency.

```ruby
name 'tomcat'
maintainer 'The Authors'
maintainer_email 'you@example.com'
license 'all_rights'
description 'Installs/Configures tomcat'
long_description 'Installs/Configures tomcat'
version '0.1.0'

depends 'java'
```

## Managing Files

* We will need to manage configurations eg. tomcat.conf
* since chef is a centralized configuration management system, we will keep the files centrally in cookbooks, which will then be copied to all managed nodes

### Generating Files

* Create `tomcat.conf` to manage the configuration of tomcat in all nodes.
* Use `chef generate file` in tomcat cookbook directory.

```console
chef generate file cookbooks/tomcat tomcat.conf
```

* Now `./sysfoo/cookbooks/tomcat/files/default/tomcat.conf` is generated using chef.
* Update `tomcat.conf` with the following content to manage tomcat.

```ruby
TOMCAT_CFG_LOADED="1"

JAVA_HOME="/usr/lib/jvm/jre"
JAVA_OPTS="-Xms64m -Xmx128m -XX:MaxPermSize=128M -Djava.security.egd=file:/dev/./urandom"

CATALINA_BASE="/usr/share/tomcat"
CATALINA_HOME="/usr/share/tomcat"
JASPER_HOME="/usr/share/tomcat"
CATALINA_TMPDIR="/var/cache/tomcat/temp"

TOMCAT_USER="tomcat"

SECURITY_MANAGER="false"

SHUTDOWN_WAIT="30"

SHUTDOWN_VERBOSE="false"

CATALINA_PID="/var/run/tomcat.pid"
```

{todo}
TIP: files/default 

### Recipe to manage cookbook files

* Now to manage `tomcat.conf` create a recipe called `config.rb` in tomcat cookbook.
* Use `chef generate recipe` to generate recipe intomcat cookbook.

```console
chef generate recipe cookbooks/tomcat config
```

* Now `./myapp/cookbooks/tomcat/recipes/config.rb` is generated.
* Add the following content into `config.rb`

```ruby
cookbook_file '/etc/tomcat/tomcat.conf' do
  source 'tomcat.conf'
  owner 'tomcat'
  group 'tomcat'
  mode  0644
  action :create
end
```

### Refreshing Services

* We can use  `notifies` with `timers` to trigger another resources in a recipes.
* Let us add an entry to *notifies tomcat service* in `config.rb`

```ruby
cookbook_file '/etc/tomcat/tomcat.conf' do
  source 'tomcat.conf'
  owner 'tomcat'
  group 'tomcat'
  mode  0644
  action :create
  notifies :restart, 'service[tomcat]', :delayed
end
```

* In the above `service tomcat` will be restarted if there is any change in `cookbook_file` resource.
* once `config.rb` recipe is created add an entry for run_list in `default.rb` of tomcat cookbook for applying it to nodes, as follows.

```ruby
include_recipe 'tomcat::config'
```

* Now `kitchen converge` and check logs to see `service tomcat` is being **restarted** for every change made in `tomcat.conf` file.
* Now verify by logging into docker instance using `kitchen login`.

[^devopsguru_blog]: Devopsguru Blog - http://devopsguru.tumblr.com/post/147717124737/chef-generate-app-vs-chef-generate-cookbook-vs
