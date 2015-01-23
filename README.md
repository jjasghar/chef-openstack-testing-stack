# OpenStack cluster with chef-provisioning

This is the testing framework for OpenStack and Chef. We leverage this to test against our changes to our [cookbooks](https://wiki.openstack.org/wiki/Chef/GettingStarted) to make sure
that you can still build a cluster from the ground up with any changes we push up. This will eventually be tied into the gerrit workflow
and become a stackforge project.

This framework also gives us an opportunity to show different Reference Architectures and a sane example on how to start with OpenStack and Chef.

With the `master` branch of the cookbooks, which is currently tied to the base OpenStack Juno release, this supports deploying to an Ubuntu 14 platform.  Support for CentOS 7 with Juno could happen at a later time.

Support for CentOS 6.5 and Ubuntu 12 with Icehouse is available with the stable/icehouse branch of this project.

## Prereqs

- [ChefDK](https://downloads.chef.io/chef-dk/) 0.3.6 or later
- [Vagrant](https://www.vagrantup.com/downloads.html) 1.7.2 or later with [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or some other provider

## Initial Setup Steps

```shell
$ git clone https://github.com/jjasghar/chef-openstack-testing-stack.git testing-stack
$ cd testing-stack
$ vi vagrant_linux.rb # change the 'vm.box' to the box you'd like to run.
$ chef exec berks vendor cookbooks
$ chef exec ruby -e "require 'openssl'; puts OpenSSL::PKey::RSA.new(2048).to_pem" > .chef/validator.pem
$ export CHEF_DRIVER=vagrant
```

The stackforge OpenStack cookbooks by default use databags for configuring passwords.  There are four databags : *user_passwords*, *db_passwords*, *service_passwords*, *secrets*. I have a already created
the `data_bags/` directory, so you shouldn't need to make them, if you do something's broken.

You may also need to change the networking options around the `aio-nova.rb`, `aio-neutron.rb`, `multi-nova.rb` or `multi-neutron.rb`
files. I wrote this on my MacBook Pro with an `en0` you're mileage may vary.

**NOTE**: If you are running Ubuntu 14.04 LTS and as your base compute machine, you should note that the shipped kernel `3.13.0-24-generic` has networking issues, and the best way to resolve this is via: `apt-get install linux-image-generic-lts-utopic`. This will install at least `3.16.0`
from the Utopic hardware enablement.

## Rake Deploy Commands

These commands will spin up various OpenStack cluster configurations, the simplest being the all-in-one controller with Nova networking.

```bash
$ chef exec rake aio_nova       # All-in-One Nova-networking Controller
$ chef exec rake aio_neutron    # All-in-One Neutron Controller
$ chef exec rake multi_neutron  # Multi-Neutron Controller and 3 Compute nodes
$ chef exec rake multi_nova     # Multi-Nova-networking Controller and 3 Compute nodes
```

If you spin up one of the multi-node builds, you'll have four machines `controller`,`compute1`,`compute2`, and `compute3`. They all live on the
`192.168.100.x` network so keep that in mind. If you'd like to take this and change it around, whatever you decide your controller
node to be change anything that has the `192.168.100.60` address to that.

NOTE: We also have plans to split out the `multi-neutron-network-node` cluster also so the network node is it's own machine.
This is also `still not complete`.

### Access the Controller

```bash
$ cd vms
$ vagrant ssh controller
$ sudo su -
```

### Testing the Contoller

```bash
# Access the controller as noted above
$ source openrc
$ nova service-list && nova hypervisor-list
$ glance image-list
$ keystone user-list
$ nova list
```

#### Booting up an image on the Controller

```bash
# Access the controller as noted above
$ nova boot test --image cirros --flavor 1
```

#### Accessing the OpenStack Dashboard 

If you would like to use the OpenStack dashboard you should go to https://localhost:9443 and the username and password is `admin/mypass`.

## Cleanup

To cleanup all the nodes and start over again with a different environment or different environment attribute overrides, using the following rake command.

```bash
$chef exec rake clean          # blow everything away
```

## Known Issues and Workarounds

### RabbitMQ

The rabbitmq cookbook has attribute `version` = 3.4.3 and `use_distro_version` = false.  The stackforge cookbooks override the 'use_distro_version' to true.  Since the version shipped with ubuntu 14 is 3.2.4-1, it fails trying to install reabbitmq-server.  This testing environment have been patched to change the 'use_distro_version' back to false.

### Windows Platform

When using this on a Windows platform, here are some tweaks to make this work.

- After creating the .chef/validator.pem, you will need to convert the .pem file to linux EOL and ANSI character set.
- In order to get ssh to work, you will need an ssl client installed.  I used the one that came with [Git for Windows](git-scm.com/download).  I needed to append the `C:\Program Files (x86)\Git\bin;` to the system PATH.
 
## TODOs

- Support for CentOS 7 with Juno
- Easier debugging. Maybe a script to pull the logs from the controller.
- More automated verification testing.  Tie into some amount of [tempest](https://github.com/openstack/tempest) or [refstack](https://wiki.openstack.org/wiki/RefStack)? for basic cluster testing
