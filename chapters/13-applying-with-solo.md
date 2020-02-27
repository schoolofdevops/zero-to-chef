# Applying code with chef-solo

Even though a typical installation of chef includes a chef server and nodes running chef-client and working in a pull model, there are times your organization may not have a server and uses **chef-solo** instead. 

In a severless model, there are three ways you could apply the code with following utilities     
  * chef-apply
  * chef-solo
  * chef-client --local-mode  | -z

Even though, officially Chef recommends using chef-client with local mode as it creates a in memory server,


file: solo.rb
```
cookbook_path "/home/devops/sysfoo/cookbooks"
role_path  "/home/devops/sysfoo/cookbooks/roles"
environment_path "/home/devops/sysfoo/cookbooks/environments"
environment "prod"

```

file: node.json
```
{ "run_list" : "role[app]"}
```


## Steps

  * Create solo.rb
  * Create node.json
  * Copy over the following
    * cookbooks
    * roles
    * environments


Generate dependencies and vendor those with the following command

```
    berks vendor /path

```
