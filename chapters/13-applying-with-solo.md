# Applying code with chef-solo

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
