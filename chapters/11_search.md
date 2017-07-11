# Search

- Search is used to run a query on chef-server.
- To search for nodes, roles, environment and data bags.
- Search patterns include exact, wildcards and range.
- Search can be included within recipe or using knife CLI.

## Replacing myhaproxy recipe

- Instead of adding the additional web servers manually to the list of haproxy, we can use search inside recipe to perform that.
- Now we can change the default recipe as follows
- Path: _myapp/cookbooks/myhaproxy/recipes/default.rb_

```ruby
all_web_nodes = search("node", "role:app")

members = [ ]

all_web_nodes.each do | web_node |

     member = {
        'hostname' => web_node['hostname'],
        'ipaddress' => web_node['ipaddress'],
        'port' => 8080,
        'ssl_port' => 8080
    }

  members.push(member)
end

node.default['haproxy']['members'] = members

node.default['haproxy']['incoming_port'] = 8080

include_recipe 'haproxy::default'
```

- Now before uploading this to chef server we will change the version of cookbook by adding the version in metadata.
- Change the version in `myapp/cookbooks/myhaproxy/metadata.rb` from **0.1.0** to **0.2.0**.
- Upload the myhaproxy cookbook.

## Verifying Search Functionality

- Now bootstrap node3 as web server using the role app.
- Before bootstrap watch the haproxy.cfg file in `node4(Load Balancer)`, which will be getting updated for every 60 seconds defined by chef-client process.

```console
watch -n 1 tail /etc/haproxy/haproxy.cfg
```

- Now bootstrap node3 from workstation,

```console
knife bootstrap node3 -x devops --sudo -N "app3" -r "role[app]"
```

- After a minute we can see the entry of new web-server `node3` added to the load balancer configuration file.
