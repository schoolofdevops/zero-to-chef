# Roles

- To create a roles in chef DSL we need to create a folder named `roles` inside the repo directory `(myapp/roles)`.
- A sample role file consist of the following,
  - Name
  - Description
  - Run_list
  - Attributes

- A sample role file is as follows `roles/sample.rb`

```ruby
name "starter"
description "An example Chef role"
run_list "recipe[starter]"
override_attributes({
  "starter_name" => "starter",
})
```

## Creating Roles for myapp

- Now create a roles for application and load_balancer.
  - myapp/roles/app.rb
  - myapp/roles/lb.rb

- Add the following content to app.rb

```ruby
name "app"
description "Tomcat Application Server"
run_list "recipe[base]", "recipe[tomcat]", "recipe[chef-client]", "recipe[myapp::deploy]"
override_attributes({
  "chef_client" => { "interval" => 120,
                     "splay" => 30
                   }
})
```

- Add the following content to lb.rb

```ruby
name "lb"
description "Load Balancer"
run_list "recipe[base]", "recipe[myhaproxy]", "recipe[chef-client]"
override_attributes({
  "chef_client" => { "interval" => 60,
                     "splay" => 20
                   }
})
```

## Uploading Roles to Chef Server

- From the `myapp` directory using knife command upload the roles from file **app.rb** and **lb.rb**

```console
knife role from file app.rb lb.rb
```

## Applying Roles to Run_list

- Now replace the existing run_list of nodes with roles.
- Add run_list to node1

```console
knife node run_list set app1 "role[app]"
```

- Add run_list to node2

```console
knife node run_list set app2 "role[app]"
```

- Add run_list to node4

```console
knife node run_list set lb "role[lb]"
```

## Run chef-client on all nodes

- Now we need to run `chef-client` on all nodes.
- We can do this by passing a `sudo chef-client` command to all nodes using knife as follows

```console
knife ssh "*:*" -x devops -a ipaddress "sudo chef-client"
```

- Verify the changes using `ps aux | grep chef-client` on all nodes to find the time interval.

```console
knife ssh "*:*" -x devops -a ipaddress "ps aux | grep chef-client"
```
