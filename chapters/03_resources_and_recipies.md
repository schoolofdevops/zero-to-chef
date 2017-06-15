# Resources and Recipes

## Resources

A resource is a statement of configuration policy that describes the desired state of a node.

* Block of code, declares the desired state of a node.
* Resource type—such as package, template, or service.

Where a resource represents a piece of the system and the steps that are needed to bring that piece of the system from its current state into the desired state.

## Recipes

* Collection of resources in a single file is called recipes.
* Recipes must be stored in a cookbook.
* One recipe may be included in another recipe.
* May use the results of a search query and read the contents of a data bag (including an encrypted data bag)
* Recipe must be added to a run-list before it can be used by the chef-client
* Is always executed in the same order as listed in a run-list

### Writing our First recipe

Lets start creating a recipe file named as `base.rb`, with the following resource specification
  - Creating `user` - deploy
  - Removing `user` - dojo
  - Installing `packages` - tree, git, ntp, wget, unzip
  - Add content to motd `file`
  - Start `ntp` service

Before Writing a recipe lets have a look of available resources [here](https://docs.chef.io/resource.html) to build of recipe.

- Our recipe start with creating a `user` resources, for creating a user called `deploy`

```ruby
user 'deploy' do
  action :create
end
```

- The above resource will create only user `deploy` to add more information to user resources, [click here](https://docs.chef.io/resource_user.html)
- Let us create password for our user `deploy`, which need to be encrypted.
- In terminal use this command and enter paswword twice to create a encrypted password `openssl passwd -l`

```ruby
user 'deploy' do
  uid 5001
  home '/home/deploy'
  action :create
  password '$1$Ze1eJK3R$j5I0NRP5WxbZAaeXcfYW7/'
end
```

### Syntax Check

* To check the syntax of ruby file we use `ruby -c <file_name.rb>` where **-c** is used to "check syntax only".

```console
ruby -c base.rb
```

### Execute Recipe with Chef Client in Local Mode

* We will be using _Chef-client_ command with `--local-mode` or `-z` to run in localhost(chef workstation).

```console
chef-client --local-mode --log_level info <file_name.rb>
or
chef-client -z -l <file_name.rb>
```

- In the above we user `--log_level info` info or `-l` to get the detailed information on `chef-client` run.

### Using Why (Dry) Run

* Before applying recipe directly we will be using dry run mode to check what this recipe **would** do
* To use dry run and check what this recipe will do, we could use `--why-run` or `-w` with __chef-client__ command.

```console
chef-client --local-mode --log_level info --why-run base.rb
or
chef-client -z -l -w base.rb
```

- It tells that `user deploy` **would** be created, and not really created, because of why run mode.

### Running our first recipe

* After checking with dry run, we can apply our recipe to the local node.

```console
chef-client --local-mode --log_level info base.rb
or
chef-apply base.rb
```
**What it does?**

  * It creates user `deploy` with specified UID and home directory in local machine.

### Idempotence

Lets run chef-client again to reapply with same options

```console
chef-client --local-mode --log_level info base.rb
```

What happened when we run again the chef-client command?

* It checks for the presence of user `deploy` and hence it **Skipped** executing `user` resources.
* This is how, **idempotent** of **chef** works.

## LAB Exercises

We created `user deploy` now let us add the remaining resources to our `base.rb` recipe.
  - Removing `user` - dojo
  - Installing `packages` - tree, git, ntp, wget, unzip
  - Add content to motd `file`
  - Start `ntp` service

## Common Functionality

Resources can have some common functionality like sharing properties, conditionals and relative actions.

### Actions

**action :nothing**

The above common functionality can be used with any resource block to do nothing until notified by another resource to take action.

### Notifications

This property is used to listens to other resources in the resource collection and then takes actions based on the notification type (`notifies` or `subscribes`).

The syntax for notifies is:

```ruby
notifies :action, 'resource[name]', :timer
```

### Timers

A timer specifies the point during the chef-client run at which a notification is run.

**:before**

Specifies that the action on a notified resource should be run before processing the resource block in which the notification is located.

**:delayed**

Default. Specifies that a notification should be queued up, and then executed at the very end of the chef-client run.

**:immediate, :immediately**

Specifies that a notification should be run immediately, per resource notified.

### Resource with Guards

* To prevent the execute command from running again and again and providing idempotent to it, we use `gaurds` one of the common functionality along with the resource.

**Gaurds**

The following properties can be used to define a guard that is evaluated during the execution phase of the chef-client run:

* not_if - Prevent a resource from executing when the condition returns true.

* only_if - Allow a resource to execute only if the condition returns true.


[Click here](https://gist.github.com/initcron/eff10a8e5bde59b356a485539579d634) for an example recipe of common functionality, `deploy_facebooc.rb` file

---
[Next Module](04_cookbooks.md)
