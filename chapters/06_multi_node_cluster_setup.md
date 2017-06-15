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

- Extract the starter kit in your PC
- Upload the extracted `chef-repo` folder of started kit(chef-starter.zip) to our Workstation running in Akurath.
- All **knife** command need to be executed inside this `chef-repo` directory, where **knife configuration file**(knife.rb) and **client_key**(username.pem) exist in `chef-repo/.chef` directory.
- `Knife` command is used to contact chef-server from workstation.

![Tree](images/pictures/07_1.png)

- We have `myapp` directory for our application, so copy `.chef` directory from **chef-repo** to **myapp**.
- From myapp directory run the following command

```console
cp -R ../chef-repo/.chef .
```

- Similarly concatenate the content from `chef-repo/.gitignore` to `myapp/.gitignore`.
- From myapp directory run the following command

```console
cat ../chef-repo/.gitignore >> .gitignore
```

- Now `knife` commands work inside myapp directory.

## Uploading Cookbooks to Chef Server

- Clone the following cookbook from [github repo](https://github.com/schoolofdevops/chef-myapp).
  - chef-client
- Now copy this `chef-client` cookbook to `myapp` folder (workspace/myapp/cookbooks)
- Upload the all the cookbook from `myapp` application to chef server using knife command.
  - java
  - tomcat
  - base
  - chef-client
- Upload single cookbook at a time.

```console
knife cookbook upload java
```

- Upload multiple cookbook in a single command

```console
knife cookbook upload tomcat
knife cookbook upload base
```

- Uploading `chef-client` will give an error since it is dependent on multiple other cookbooks, here is where we use berksfile.

### Berkshelf

Berkshelf is a dependency manager for Chef cookbooks. With it, you can easily depend on community cookbooks and have them safely included in your workflow. You can also ensure that your CI systems reproducibly select the same cookbook versions, and can upload and bundle cookbook dependencies without needing a locally maintained copy. Berkshelf is included in the Chef Development Kit.

- Now look into the downloaded `chef-client` cookbook.
- It has `berksfile` where the dependent cookbook are mentioned.
- To run `berks` command we need to be in the directory where berksfile is located, that is `workspace/myapp/cookbooks/chef-client`
- Now run install and upload command.

```console
berks install; berks upload
```

### Bootstrapping nodes

- Now run `knife client list` to get the list of client associated with the hosted chef-server.
- In our case we don't have any client as of now, so lets start adding nodes by using `knife bootstrap` command.
- From Workstation we need to bootstrap client.
- In Akurath we have pre-built nodes associated with the following IP.

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

- Now add runlist to node1.

```console
knife node run_list add app1 "recipe[tomcat],recipe[base],recipe[chef-client]"
```

- Now bootstrap node2 along with run-list.

```console
knife bootstrap node2 --ssh-user devops --sudo -N app2 --run-list "recipe[tomcat],recipe[base],recipe[chef-client]"
or
knife bootstrap node2 -x devops --sudo -N app2 -r "recipe[tomcat],recipe[base],recipe[chef-client]"
```

- `--run-list` is used to specify run-list and by applying recipes to the node at the time of bootstrap.
- Now verify by visiting host ip with port mapping of 8080 to 8081 for node1 http://ip:8081 and port mapping of 8080 to 8082 for node2 http://ip:8082, where the tomcat application is installed and service is up and running.

---
[Previous Module](05_tdd_with_test_kitchen.md) ------ [Next Module](07_data_driven_cookbooks_attr_templates.md)
