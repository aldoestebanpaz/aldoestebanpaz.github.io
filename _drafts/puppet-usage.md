# Puppet usage

## Concepts

- Resources: A resource describes something about the state of the system, such as a certain user or file should exist, or a package should be installed.
- Facts: Puppet collects system information, called facts, by using the Facter tool. The facts are assigned as values to variables that you can use anywhere in your manifests. Puppet also sets some additional special variables, called built-in variables, which behave a lot like facts.
- Templates: Templates are documents that combine code, data, and literal text to produce a final rendered output. The goal of a template is to manage a complicated piece of text with simple inputs.
- Manifests: Puppet programs are called manifests and their filenames use the .pp extension
- Classes: Classes are code blocks that can be called elsewhere. Using classes allows the reuse of Puppet code, and can make reading manifests easier.
- Modules: A module is a collection of manifests and data (such as facts, files, and templates), and they have a specific directory structure. Modules are useful for organizing Puppet code, because they allows to to split the code into multiple manifests. It is considered best practice to use modules to organize almost all Puppet manifests.
- Catalog: When a node is configured, puppet agent uses a document that is termed as the Catalog and it can be downloaded from the Puppet Master. It has the state details of each resource that will be managed in a specific order. The data stored in Puppet Catalog is driven by three facts: Data provided by the puppet agent, External data details and Details related to Puppet manifests.
- Hiera: Hiera is a powerful way to store (class parameter) data outside of .pp files. Hiera also stores this data in a efficient hierarchial structure so to minimize code duplication. This is especially useful when you want to declare a class that requires a lot of class parameters.

## Environments

### The environment.conf file

Any environment can contain an environment.conf file. This file can override several settings whenever the Puppet master is serving nodes assigned to that environment.

Location
Each environment.conf file should be stored in an environment. It should be at the top level of its home environment, next to the manifests and modules directories.

For example, if your environments are in the default directory ($codedir/environments), the test environment’s config file should be located at $codedir/environments/test/environment.conf.



## Defaults




