# Customizing Community Cookbooks

## Customizing Strategy

- Fork - Directly use the community cookbook by forking the upstream cookbook.
- Wrapper - Create a wrapper cookbook which calls the community cookbook and customize it, by creating a dependency.

## Creating myhaproxy cookbook

- Generate a cookbook called `myhaproxy` inside `sysfoo`.

```console
cd /workspace/chapter8/sysfoo/
chef generate cookbook cookbooks/myhaproxy
```

- Add dependency in `myhaproxy/metadata.rb`

```ruby
depends 'haproxy', '~> 1.6.6'
```

  - Here `~> 1.6.6` will be calling the community cookbook of haproxy one version level higher to this, i.e, `1.6.7`.

-  Now add the wrapper content to include and modify `haproxy` cookbook by adding the following in `myhaproxy/recipes/default.rb`

```ruby
node.default['haproxy']['members'] = [{
     "hostname" => "node1",
     "ipaddress" => "node1",
     "port" => 8080,
     "ssl_port" => 8080
   },
   {
    "hostname" => "node2",
    "ipaddress" => "node2",
    "port" => 8080,
    "ssl_port" => 8080
  }]


node.default['haproxy']['incoming_port'] = 8080


include_recipe 'haproxy::default'
```

## Resolve Dependency and Bootstrapping

- Use berkshelf to resolve dependency of `myhaproxy` cookbook.
- Upload the cookbook to server.
- Now bootstrap `node4` by naming it as `lb` and adding the following run_list
  - base
  - chef-client
  - myhaproxy

## Verifying Load Balancer

- After bootstrapping visit http://ip:8084 to check the load Balancer.
- It is 8084 because the host port 8080 is mapped to 8084.
