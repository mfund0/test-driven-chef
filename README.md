## Test-Driven Chef Cookbook Development

*Cherry-picked facts around tools to write chef cookbooks with tests
included. Motivation: This stuff can be tedious to find and I have not
found a good, up to date book that covers it all.*

## Static Analysis

### [Benefits](https://docs.chef.io/workstation/cookstyle/)

-   Enforcing style conventions and best practices.

-   Helping every member of a team author similarly structured code.

-   Maintaining uniformity in the source code.

-   Setting expectations for fellow (and future) project contributors.

-   Detecting deprecated code that creates errors after upgrading to a newer Chef Infra Client release.

-   Detecting common Chef Infra mistakes that cause code to fail or behave incorrectly.

### Toolchain

We can use Cooksyte \[2\], which uses RuboCop. Checks performed on the
code are called cops. Place custom rules in a ​​.rubocop.yml file at the
root of the repository. Example commands to run linting tests:

  |         |           |
| ------------- | ------------- |
| `cookstyle .`      | Runs the linter recursively. |
| `cookstyle --auto-gen-config`     | Generates a .rubocop_todo.yaml in directory .. to later rename to .rubocop.yaml |
| `cookstyle --init` | Generate a .rubocop.yml file in the  current directory. |

Example Syntax for .rubocop.yml:

```yaml
AllCops:
 TargetChefVersion: 14.0
 
Layout/DotPosition:
 EnforcedStyle: trailing
 
Layout/IndentArray:
 EnforcedStyle: consistent
```

More Cookstyle Cops can be found here:\
[[https://docs.chef.io/workstation/cookstyle/cops/]{.ul}](https://docs.chef.io/workstation/cookstyle/cops/)

## Unit Testing

### Benefits

### Toolchain

#### ChefSpec

ChefSpec is a unit testing Framework ( an extension of ruby's RSpec). It
uses Chef Solo to run tests (named: examples) locally.

Benefits:

-   Runs tests locally fast, gets feedback fast without needing a VM

-   Your tests can vary node attributes, operating systems, and other system data to assert behaviour under varying conditions

ChefSpec are usually stored in a path like spec/something_spec.rb

Example commands:
|               |           |
| ------------- | ------------- |
| `chef exec rspec .` | Runs tests. |

Example Syntax

Given a resource:

  -----------------------------------------------------------------------
  package \'foo\'
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

A test for it may look as follows:

```ruby
require 'chefspec'
 
describe 'example::default' do
 let(:chef_run) { ChefSpec::Runner.new.converge(described_recipe) }
 
 it 'installs foo' do
   expect(chef_run).to install_package('foo')
 end
end
```

Find more examples here:
[https://github.com/chefspec/chefspec/blob/main/README.md](https://github.com/chefspec/chefspec/blob/main/README.md)

### When to write ChefSpec Unit Tests

*ChefSpec excels at testing complex logic in a cookbook, but can\'t
actually tell you if a cookbook is doing the right thing. Integration
testing is provided by the* *[Test Kitchen](https://kitchen.ci/)
project, and for most simple cookbooks without much logic in them we
recommend you start off with integration tests and only return to
ChefSpec and unit tests as your code gets more complicated.*
-- https://github.com/chefspec/chefspec#when-to-use-chefspec

## Integration Testing

### Toolchain

#### Test Kitchen

A framework for running integration tests in one or more platforms in
isolation. It uses a file called kitchen.yml to describe a testing
configuration. On a high level, the configurations include:

> **driver**: What runs the tests suites
>
> **provisioner**: How a Chef Client will be simulated
>
> **platforms**: a collection of platform operating systems to use to
> run the test suites
>
> **verifier:** What framework to use to run the tests
>
> **suites**: A collection of test suites that define what aspect of the
> cookbook to tests

A sample of a kitchen.yaml file :

```yaml
driver:
 name: vagrant
 
provisioner:
  name: chef_solo
  environments_path: test/integration/environments/test
 
platforms:
 - name: ubuntu-20.04
 - name: ubuntu/focal64
 
suites:
 - name: default
   run_list:
     - recipe[cookbook_b::default]
   data_bags_path: "test/integration/data_bags"
   provisioner:
     client_rb:
       environment: envxyz
   attributes: {
     'some_attr':{}}
```

A typical test folder structure for everything test kitchen needs is as
follows:

|               |               |
| ------------- | ------------- |
| `test/integration/default_tests.rb`     | default inspec test |
| `test/fixtures`       | Fixture cookbooks to mock out   |
| `test/data_bags` | Where test kitchen will find data bags. <br /> *Hint: You can create local encrypted data bags with:* <br /> `knife data bag from file my_data_bag\` <br /> ` /path/to/data_bag_item.json -z --secret-file /path/to/secret` |
| `test/integration/environment`        | where to find environment files   |

Common commands:

|               |               |
| ------------- | ------------- |
| `kitchen init` | Adds minimal tests configuration to your cookbook. Test files also generated when one runs thecommand generate a cookbook from scratch: chef generate cookbook |
| `kitchen create` | Creates an instance (VM or Container) |
| `kitchen converge` | Uses a provisioner to configure one or more instances |
| `kitchen verify` | Runs tests on one or more instances |
| `kitchen login` | Logs in to an instance |
| `kitchen test` | Runs all stages of the test (destroy, create, converge, setup, verify and destroy) |

Kitchen uses either Berkshelf or Policyfiles to resolve cookbook
dependencies

##### Using Berkshelf

A Berksfile is used to specify where test kitchen will find cookbooks
and fixtures. Example:

```ruby
source chef_repo: "path/to/all/cookbooks"
 
group :test do
 cookbook 'resolved', path: 'test/integration/fixtures/some_cookbook'
end
```

##### Using Policyfiles \[WIP\]

A Policyfile is used to specify where test kitchen will find cookbooks
and fixtures. Example:

```ruby

```

##### Data Bags \[TODO\]

##### Environments \[TODO\]

##### Fixture Cookbooks \[TODO\]

#### InSpec \[WIP\]

Chef InSpec is an open-source framework for testing and auditing your
applications and infrastructure.

[https://github.com/inspec/inspec/tree/main/docs-chef-io/content/inspec/resources](https://github.com/inspec/inspec/tree/main/docs-chef-io/content/inspec/resources)

#### Kitchen-vagrant \[TODO\]

A Test Kitchen Driver for Vagrant

#### Kitchen-dokken \[TODO\]

A Test Kitchen Driver for Vagrant

#### Kitchen-Terraform \[TODO\]

## Continuous Integration \[TODO\]

### Toolchain

#### Jenkins

#### Github Actions
