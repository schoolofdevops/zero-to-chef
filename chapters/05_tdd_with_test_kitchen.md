# TDD with Test Kitchen

## App test vs Cookbook Test

* Test configuration can be defined in two different places.
* One is inside app directory `myapp/test/recipes/default_test.rb`
* Another one is inside the cookbooks directory `myapp/cookbooks/mycookbook/test/recipes/default_test.rb`

![App test vs Cookbook Test](images/pictures/05_1.png)

## Inspec vs Serverspec

* Inspec [resources](https://www.inspec.io/docs/reference/resources/)
* Serverspec [resources](http://serverspec.org/resource_types.html)

## Creating test file

### User Resource

* Add **inspec user resource** to check for `tomcat user`

```ruby
unless os.windows?
  describe user('tomcat') do
    it { should exist }
  end
end
```

### Port Resource

* Add **inspec port resource** to verify listening of port `8080`
* Include protocols for `tcp6`

```ruby
describe port(8080) do
  it { should be_listening }
  its('protocols') { should include 'tcp6' }
end
```

### Service Resource

* Add **inspec service resource** to verify `tomcat service`
* Include `installed`, `enabled` and `running` conditions

```ruby
describe service('tomcat') do
  it { should be_installed }
  it { should be_enabled }
  it { should be_running }
end
```

Complete file of `/workspace/myapp/test/recipes/default_test.rb` is as follows

```ruby
unless os.windows?
  describe user('tomcat') do
    it { should exist }
  end
end

describe port(8080) do
  it { should be_listening }
  its('protocols') { should include 'tcp6' }
end

describe service('tomcat') do
  it { should be_installed }
  it { should be_enabled }
  it { should be_running }
end
```

---
[Previous Module](04_cookbooks.md) ------ [Next Module](06_multi_node_cluster_setup.md)
