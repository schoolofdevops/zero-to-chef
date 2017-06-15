# Data Driven Cookbooks

## Code vs Data

![Code vs Data](images/pictures/06_1.png)

* once the data is taken out of the code then cookbook become generic.
* Generic cookbook can be used with any OS and environments.
* Reusability helps in reducing tonnes and tonnes of time.

## Node Object

- It is the representation of each node on chef server in `JSON` format.
- If we run a `chef-client` it generates a node object in chef server in case of nodes or in workstation (`/nodes/*.json`) in case of running locally in workstation.

## Attributes

It is used to specify a detail about node.
- Attributes are used by the chef-client to understand;
 - The current state of the node
 - What the state of the node was at the end of the previous chef-client run
 - What the state of the node should be at the end of the current chef-client run
- `Attributes` are taken in precedence, according to the place where we specify them, as follows;

### System Defined Attributes

#### OHAI

- It installs along with `chef-client`.
- It helps in getting all the information about nodes
- Finding information on node with ohai

```console
ohai
ohai ipaddress
ohai hostname
ohai memory
ohai memory/total
ohai cpu/model_name
```

- Referring attributes

```ruby
node['ipaddress']
node['hostname']
node['memory']['total']
node['cpu']['model_name']
```

### User Defined Attributes

- User defined attributes are located in four places as follows
  - Attributes file
  - Recipes
  - Roles
  - Environments

## Attribute File

- How the attribute file looks like?

![Attributes file](images/pictures/06_2.png)

- Precedence is defined
- Attribute and its value is defined as per the above format.

- Now we can access these attributes in `recipe` or `template` as shown below.

![Accessing Attributes](images/pictures/06_3.png)

**Example**

```ruby
template node['tomcat']['config'] do
  source 'tomcat.conf.erb'
  owner node['tomcat']['user']
  group node['tomcat']['group']
  mode '0644'
  action :create
  notifies :restart, "service[#{node['tomcat']['service']}]", :delayed
end
```

## Parameterizing Tomcat Configurations

- Now let us start creating attribute file
- Use `chef generate` to create a file in tomcat cookbook

```console
chef generate attribute cookbooks/tomcat default
```

- It will create a file as follows `cookbooks/tomcat/attributes/default.rb`
- Now we add the attributes and its value in `default.rb`(cookbooks/tomcat/attributes/default.rb)

```ruby
default['tomcat']['user'] = 'tomcat'
default['tomcat']['group'] = 'tomcat'
default['tomcat']['config'] = '/etc/tomcat/tomcat.conf'
default['tomcat']['packages'] = [ 'tomcat', 'tomcat-webapps' ]
default['tomcat']['service'] = 'tomcat'
```

- Now call these attributes in the following recipes
- **tomcat::config** will be changed as follows

```ruby
cookbook_file node['tomcat']['config'] do
  source 'tomcat.conf'
  owner node['tomcat']['user']
  group node['tomcat']['group']
  mode '0644'
  action :create
  notifies :restart, "service[#{node['tomcat']['service']}]", :delayed
end
```

- **tomcat::install** will be changed as follows

```ruby
package node['tomcat']['packages']
```

- **tomcat::service** will be changed as follows

```ruby
service node['tomcat']['service'] do
   action [ :start, :enable]
end
```

- Now apply it using `kitchen converge` and then verify using `kitchen verify`.

## Platform Specific cookbooks

- Let us convert our first created `base.rb` recipe into a cookbook.
- Create a cookbook using `chef generate`

```console
chef generate cookbook cookbooks/base
```

- Now move the `base.rb` to `default.rb` of base cookbook.

```console
mv /workspace/base.rb cookbooks/base/recipes/default.rb
```

- Add `default` recipe to `.kitchen.yml` runlist

```ruby
suites:
  - name: default
    run_list:
      - recipe[tomcat]
      - recipe[base::default]
```

- Apply using kitchen

```console
kitchen converge
```

- **It fails** because the service name for rhel/centos is `ntpd` and **not ntp** as like debian.

![centos vs ubuntu](images/pictures/06_4.png)

- To overcome this we use attribute for platform specific.
- We use system defined attribute `platform_family` to do this.

**Example** for `ohai platform_family` is

```console
root@ws:/workspace/myapp# ohai platform_family
[
  "debian"
]
root@ws:/workspace/myapp# kitchen login
Last login: Thu May  4 05:50:20 2017 from 172.17.0.1
[kitchen@9af3a004e202 ~]$ ohai platform_family
[2017-05-04T05:58:19+00:00] INFO: The plugin path /etc/chef/ohai/plugins does not exist. Skipping...
[
  "rhel"
]
[kitchen@9af3a004e202 ~]$ logout
Connection to localhost closed.
root@ws:/workspace/myapp#
```

- Now let us create a create a *default attribute* file for base cookbook using `chef generate`

```console
chef generate attribute cookbooks/base default
```

- Now add the following content to ` cookbooks/base/attributes/default.rb` attribute file

```ruby
case node['platform_family']
when 'rhel'
  default['ntp']['service'] = 'ntpd'
else
  default['ntp']['service'] = 'ntp'
end
```

- Call the attributes in `cookbooks/base/recipes/default.rb` recipe

```ruby
service node['ntp']['service'] do
  action [ :start, :enable ]
end
```

- Now apply again using kitchen

```
kitchen converge
```

- It is successful now and service is started based on platform.

**Nano Project** - Add some tests for recipes in base cookbook for different platform.

## Templates

- A cookbook template is an Embedded Ruby (ERB) template that is used to dynamically generate static text files.
- Templates are great way to manage configuration files for different environment.
- ERB or ERUBIS
  - Contains text with dynamic ruby code
  - Uses tags to mark dynamic code
- ERB Tags
  - Code wrapped in `<% %>` or `<% -%>` is a statement that is evaluated.
  - Code wrapped in `<%= %>` is code that is evaluated and the result is placed into the file.
  - Harcoded strings dont have to be wrapped in erb tags if they are constant, but Ruby code must be wrapped in erb tags if you want the result of that code to go into your file.

### Templatize MOTD

- Generate `motd` template for *base cookbook*.

```console
chef generate template cookbooks/base motd
```

- Add content to the template file `cookbooks/base/templates/default/motd.erb`

```ruby
This server is a property of <%= node['org']['name'] %>

      SYSTEM INFO:

       HOSTNAME   : <%= node['hostname'] %>
       IP ADDRESS : <%= node['ipaddress'] %>
       MEMORY     : <%= node['memory']['total'] %>
```

- Now add organization name as a user defined attribute in `cookbooks/base/attributes/default.rb` attribute file.

```ruby
default['org']['name'] = "XYZ Inc."
```

- Complete attribute `default.rb` of `base cookbook` is as follows

```ruby
case node['platform_family']
when 'rhel'
  default['ntp']['service'] = 'ntpd'
else
  default['ntp']['service'] = 'ntp'
end

default['org']['name'] = "XYZ Inc."
```

- Now add template resource to the recipe `cookbooks/base/recipes/default.rb` file.

```ruby
template '/etc/motd' do
  source 'motd.erb'
  owner 'root'
  group 'root'
  mode  0644
end
```

- Apply using `kitchen converge`.

### Attributes Precedence

- Declaring attributes in various places takes precedence one over another.

![Attributes precedence](images/pictures/M4_1.png)

- Let us take an example of declaring port as follows

![Attributes precedence example](images/pictures/06_5.png)

- Now let us define organization name in recipe `cookbooks/base/recipes/default.rb` file, before calling attribute in any resource of a recipe.

```ruby
node.default['org']['name'] = "School of Devops"
```

- Complete recipe `default.rb` of `base cookbook` is as follows

```ruby
user 'deploy' do
  uid 5001
  home '/home/deploy'
  action :create
  password '$1$Ze1eJK3R$j5I0NRP5WxbZAaeXcfYW7/'
end

user 'dojo' do
  action :remove
end

package 'ntp' do
  action :install
end

package ['tree', 'unzip', 'wget'] do
  action :install
end

package 'git'

service node['ntp']['service'] do
  action [ :start, :enable ]
end

node.default['org']['name'] = "School of Devops Inc"

template '/etc/motd' do
  source 'motd.erb'
  owner 'root'
  group 'root'
  mode  0644
end
```

- Now apply using `kitchen converge`

## Templatizing Tomcat Cookbook (LAB Exercise)

- Generate a `tomcat.conf.erb` template for tomcat cookbook.

```console
chef generate template cookbooks/tomcat tomcat.conf
```

- Generate a `default.rb` attribute for tomcat cookbook.

```console
chef generate attribute cookbooks/tomcat default
```

- Add the following content to attribute `/workspace/myapp/cookbooks/tomcat/attributes/default.rb`

```ruby
default['tomcat']['user'] = 'tomcat'
default['tomcat']['group'] = 'tomcat'
default['tomcat']['config'] = '/etc/tomcat/tomcat.conf'
default['tomcat']['packages'] = [ 'tomcat', 'tomcat-webapps' ]
default['tomcat']['service'] = 'tomcat'
default['tomcat']['user'] = 'tomcat'
default['tomcat']['group'] = 'tomcat'
default['tomcat']['java_home'] = '/usr/lib/jvm/jre'
default['tomcat']['catalina_home'] = '/usr/share/tomcat'
default['tomcat']['java_opts'] = '-Xms32m -Xmx64m -XX:MaxPermSize=64M -Djava.security.egd=file:/dev/./urandom'
```

- Add the below content to `tomcat.conf.erb` template in `/workspace/myapp/cookbooks/tomcat/templates`

```ruby
TOMCAT_CFG_LOADED="1"

JAVA_HOME="<%= node['tomcat']['java_home'] %>"
JAVA_OPTS="<%= node['tomcat']['java_opts'] %>"

CATALINA_BASE="/usr/share/tomcat"
CATALINA_HOME="<%= node['tomcat']['catalina_home'] %>"
JASPER_HOME="/usr/share/tomcat"
CATALINA_TMPDIR="/var/cache/tomcat/temp"

TOMCAT_USER="<%= node['tomcat']['user'] %>"

SECURITY_MANAGER="false"

SHUTDOWN_WAIT="30"

SHUTDOWN_VERBOSE="false"

CATALINA_PID="/var/run/tomcat.pid"
```

- Now update `tomcat::config` recipe `/workspace/myapp/cookbooks/tomcat/recipes/config.rb` as mentioned below

```ruby
template node['tomcat']['config'] do
  source 'tomcat.conf.erb'
  owner node['tomcat']['user']
  group node['tomcat']['group']
  mode '0644'
  action :create
  notifies :restart, "service[#{node['tomcat']['service']}]", :delayed
end
```

- Apply using `kitchen converge`

- Now change the values of `JAVA_OPTS` in attribute file and then apply again for verifying the changes using template and attribute.

- Also verify using `kitchen login` and run `ps auwwx` command.

---
[Previous Module](06_multi_node_cluster_setup.md) ------ [Next Module](08_customizing_community_cookbooks.md)
