# Container ModuleSync Configs

This repository contains default configuration for
[modulesync](http://github.com/puppetlabs/modulesync) for the container at Puppet community.
Changes to this repository should be synced with modulesync across all of the container repos.

A full description of ModuleSync can be found in
[ModuleSync's README](https://github.com/puppetlabs/modulesync).
This README describes how the templates are rendered in the Puppet Labs configuration.

## Configuring ModuleSync

**`modulesync.yml`**

A key-value store of arguments to pass to ModuleSync. Each key is the name of a
flag argument to the msync command. For example, `namespace: myusername`
represents passing `--namespace myusername` to msync. This file does not appear
in this repository because it only serves to override default configuration. To
override the default configuration, the file may look something like this:

```yaml
---
namespace: yourusername
branch: yourbranch
```

**`managed_modules.yml`**

A YAML array containing the names of the modules to manage.

## Run ModuleSync

To run modulesync, you need to have the modulesync gem installed. You also need a PAT (Personal Access Token) from GitHub to authenticate with the GitHub API. You can create a PAT by following the instructions [here](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token).

```shell
bundle config set --local path 'vendor/bundle'
bundle install

export GITHUB_TOKEN=your_github_pat
bundle exec msync update \
  -f container-test \
  --pr \
  --pr-labels modulesync \
  --pr-title Modulesync \
  --git-base=https://github.com/ \
  --noop
```

## Run ModuleSync for another Organization

```shell
export GITHUB_TOKEN=your_github_pat
bundle exec msync update \
  --pr \
  --pr-labels modulesync \
  --pr-title Modulesync \
  --git-base=https://github.com/ \
  --namespace openvoxproject \
  --managed-modules-conf managed_modules_openvox.yml \
  --noop
```

## Defining Module Files

**`config_defaults.yml`**

Each first-level key in this file is the name of a file in a module to manage.
These files only appear here if there are templates in the moduleroot/
directory that need to be rendered with some default values that might be
overridden. The files listed do not necessarily represent all the files that
will be managed. The files in moduleroot/ represent all the files that will be
managed, except for unmanaged and deleted files (see [#Special Options]).

**`.sync.yml`**

This file should appear in the module itself if there are any values to
override from the config_defaults.yml file or if there are any additional
values to assign. A description of what optional values can be defined in
.sync.yml follows in the description of each file in moduleroot/. .sync.yml
will have the same format as config_defaults.yml.

### Note

Each template is rendered in slightly different ways. Your templates do not
need to be identical to these, as long as your config_defaults.yml or .sync.yml
files contain as first-level keys the exact names of the files you are
managing and appropriately handle the data structures you use in your templates
(arrays versus hashes versus single values).

**`moduleroot/Gemfile`**

The Gemfile contains a list of gems, optionally with versions, to install in
the development and test groups. config_defaults.yml contains a list of
"required" gems to install, in the form of an array where each element contains
the names and versions of the gems. This section of config_defaults.yml might
look like

```yaml
Gemfile:
  required:
  - gem: rake
    version: '~>1.2'
  - gem: rspec-puppet
#...
```

The template also looks in .sync.yml for a group of optional gems to install,
and merges this list with the list found in config_defaults.yml. This section
of .sync.yml will look the same as the section of config_defaults.yml, but the
name will be "optional" rather than "required".

**`moduleroot/Rakefile`**

The Rakefile gets most of its tasks from the puppetlabs_spec_helper. The
variables in the template represent lint checks to disable. config_defaults.yml
contains an array of checks to pass in to PuppetLint.configuration.send. The
key for this array is called default_disabled_lint_checks. .sync.yml may
contain an additional array of checks to disable, with the key
extra_disabled_lint_checks.

**`moduleroot/.gitignore`**

Contains some standard files to ignore. You can pass in additional files as an
array with the key "paths" in your .gitignore section in .sync.yml.

You can add additional environments for a specific module to test by adding an
extras: section with the same format to the module's .sync.yml.

## Special Options

### Unmanaged Files

A file can be marked "unmanaged" in .sync.yml, in which case modulesync will
not try to modify it. This is useful if, for example, the module has special
Rake tasks in the Rakefile which is difficult to manage through a template.

To mark a file "unmanaged", list it in .sync.yml with the value `unmanaged:
true`. For example,

```yaml
---
spec/spec_helper.rb:
  unmanaged: true
```

### Deleted Files

Managing files may mean removing files. You can ensure a file is absent by
marking it "delete". This is useful for purging nodesets.

To mark a file deleted, list it in .sync.yml with the value `delete: true`. For
example,

```yaml
---
spec/acceptance/nodesets/sles-11sp1-x64.yml
  delete: true
```
