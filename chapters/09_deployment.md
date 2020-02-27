# Nano Project: Deploy Sysfoo on Tomcat

So far you have learnt to write cookbooks, recipes and how to create data driven cookbooks. Now that you have setup tomcat, you have been
tasked to write code to deploy a java web application to tomcat server. This application can be pulled from the CI server e.g. Jenkins, CircleCI etc. And the deployment should be automated.

### Steps :
  * Create a GitHub account
  * Clone sample java webapp repo from https://github.com/schoolofdevops/sample-webapp
  * Create a CircleCI Account https://circleci.com and integrate with your github sample-webapp project, setup the build
  * Write a deploy recipe with chef, add it to run list and test deploy.


## Deploy Recipe Specs:


Create a sysfoo::deploy recipe with the following specifications,
  * Your recipe should download the artifact (warfile) from circle CI
   (e.g https://5-94848332-gh.circle-artifacts.com/0/tmp/circle-artifacts.uqT2VIZ/sysfoo.war)
  * The above URL should be parameterized. Updating the URL using an attribute should deploy the new version.
  * The resource which downloads the artifact should be idempotent. It should not download the same file again if chef-client is run, unless its a new version of the artifact.  
  * The downloaded artifact  should be copied it to the webapps directory of tomcat as app.war.  In this example, it would be /usr/share/tomcat/webapps/sysfoo.war
  * The previous version of the application, if available, should be deleted (path: /usr/share/tomcat/webapps/sysfoo)
  * Tomcat needs to be restarted whenever a new version of artifact s deployed.
