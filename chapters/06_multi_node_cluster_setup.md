# Multi Node Cluster

In this session  we going to simulate a realistic environment with multiple nodes being managed with Chef Server, similar to most of the real world implementations.

This would include 3 components,

* Chef Workstation (Could be your personal PC/Laptop)  
* Chef Server (Hosted or On-Premises )  
* Chef Clients ('n' number of machines; could be VM, AWS EC2, etc..)  

![Multi-Node](images/pictures/M2_1.png)

## Hosted Chef Server

### Creating an Account

- Now let us create a account in [chef.io](https://manage.chef.io/login) to manage our own hosted chef server.  
- Once account is created, verify and login.  
- Now create a organization of your own and then download chef-starter kit.  

### Setting Up Workstation

- Upload  starter kit to the workspace. Its the **chef-starter.zip** file that you downloaded from chef server.  
- Upload the file to workspace and extract it. You should see `chef-repo` directory created after being extracted.  
- Change into chef-repo directory created as the outcome of above command and validate workstation by running  

```
knife client list
```

The above command will return the **orgname-validator** client. If it does,you have successufully validated all of following,  

* Workstation software (knife) is installed  
* You have required configurations, authentication keys/credentials to talk to Chef Server  
* Chef Server is setup and ready
* Workstation is able to communicate with Chef Server. There are no network issues etc.  
* Workstaiton is able to authenticate with Chef Server  
* Workstation has made the API request and displays  the results returned by Chef Server  

![Tree](images/pictures/07_1.png)

### Moving knife configs to workspace

- Copy `.chef` directory from **chef-repo** to **workspace**.

```console
cd /workspace
mv chef-repo/.chef .
ls -al
```


Edit /workspace/.chef/knife.rb  
and update cookbook_path from

```
cookbook_path  ["#{current_dir}/../cookbooks"]

```
to

```
cookbook_path       ["cookbooks"]

```

Change into chapter6/sysfoo and validate

```
cd /workspace/chapter6/sysfoo

knife client list
```

From here on all  `knife` commands work from any subdirectory of /workspace.

### Bootstrapping a Managed Node

- Now run `knife client list` to get the list of client associated with the hosted chef-server.
- In our case we don't have any client as of now, so lets start adding nodes by using `knife bootstrap` command.
- From Workstation we need to bootstrap client.
- In codespaces environment we have pre-built nodes associated with the following IP.

|Node|Name|IP|Port Mapping|
|:---:|:---:|:---:|:---:|
|node1|app1|177.0.101.10|8081:8080|
|node2|app2|177.0.101.11|8082:8080|
|node3|app3|177.0.101.12|8083:8080|
|node4|lb|177.0.101.13|8084:8080|

- All nodes are accessible using ssh without password.

- Lets bootstrap node1 using the following command

```console
knife bootstrap node1 --ssh-user devops --sudo -N app1
```

  - Here `-N` is used to define the name of the node that we bootstrap.
  - `--ssh-user` is used to provide the name of the user in that particular node.
  - Also using `--sudo` to connect is to provide root previlages for running all the command.
  - `app1` is bootstrapped successfully.

- Now check for the available nodes

```console
knife node list
```

- Check for the existing status and specified runlist for node1(app1)

```console
knife node show app1
```

### Providing configurations to the Node
Change into chef-sysfoo directory on workstation.

Upload initial cookbooks

```
knife cookbook upload java tomcat

```

#### Defining Run List for the Node

```
knife node show app1
knife node run_list add app1 "recipe[tomcat]"
```

To apply, login to node1 and run chef-client

```
ssh devops@node1
sudo chef-client
```

### Managing Chef Client as a Service  

Lets now start managing chef-client and its configurations through chef cookbooks.  We have a special purpose cookbook by name **chef-client** which allows us to do so.  It will,
  * decide how to run chef-client, eg. cronjob, service etc.
  * does support   various types of service managers e.g. runit, bluepill, supervisord etc.
  * manages configurations for chef client .eg. how frequently chef-client runs

Lets upload chef-client cookbook which is already present if you are using the code repository provides.


```
knife cookbook upload chef-client

```

Did chef-client cookbook get uploaded successfully?   If not, observe the error and try to deduct the root cause.


### Berkshelf

Berkshelf is a dependency manager for Chef cookbooks. With it, you can easily depend on community cookbooks and have them safely included in your workflow. You can also ensure that your CI systems reproducibly select the same cookbook versions, and can upload and bundle cookbook dependencies without needing a locally maintained copy. Berkshelf is included in the Chef Development Kit.

- Now look into the downloaded `chef-client` cookbook.
- It has `berksfile` where the dependent cookbook are mentioned.
- To run `berks` command we need to be in the directory where berksfile is located, that is `workspace/sysfoo/cookbooks/chef-client`
- Now run install and upload command.

```console
cd cookbooks/chef-client

berks install; berks upload

cd ../..

```

Lets now add **chef-client** recipe to the runlist of node1.

```console
knife node run_list add app1 "recipe[chef-client]"
```



#### Providing Run List at the Bootstrap time

```console
knife bootstrap node2 -x devops --sudo -N app2 -r "recipe[tomcat],recipe[chef-client]"
```


- `--run-list` is used to specify run-list and by applying recipes to the node at the time of bootstrap.
- Now verify by visiting host ip with port mapping of 8080 to 8081 for node1 http://ip:8081 and port mapping of 8080 to 8082 for node2 http://ip:8082, where the tomcat application is installed and service is up and running.
