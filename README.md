Display docker information from a vagrant machine, similar to what boot2docker provides

Based in part on the work done by Michael Kuzmin for vagrant-guestip at https://github.com/mkuzmin/vagrant-guestip

#Quick Start

- Export following in a `Vagranfile`

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

#Vagrant file for libvirt/kvm and virtualbox provider

# check if plugin is installed on system
unless Vagrant.has_plugin?("vagrant-adbinfo")
  raise "vagrant-adbinfo plugin is not installed, run `vagrant plugin install vagrant-adbinfo` to install the plugin."
end

Vagrant.configure(2) do |config|
  config.vm.box = "atomicapp/dev"

  # forward the guest port 2376 of `docker` daemon in guest to host port 2379 for client side tooling to connect
  config.vm.network "forwarded_port", guest: 2376, host: 2379, auto_correct: true

  config.vm.provider "libvirt" do |libvirt, override|
    libvirt.driver = "kvm"
    libvirt.memory = 1024
    libvirt.cpus = 2
  end

  config.vm.provider "virtualbox" do |vbox, override|
    vbox.memory = 1024
    vbox.cpus = 2

    # Enable use of more than one virtual CPU in a virtual machine.
    vbox.customize ["modifyvm", :id, "--ioapic", "on"]
  end

end
```

- ```vagrant up```

- ```vagrant adbinfo```


#How to Develop/Test

1 - Install VirtualBox (http://www.if-not-true-then-false.com/2010/install-virtualbox-with-yum-on-fedora-centos-red-hat-rhel/)

2 - Get a box to test

3 - Build a vagrant file

4 - bundle install

5 - bundle exec vagrant up # starts the vagrant box

6 - bundle exec vagrant adbinfo # tests the command

7 - rake build # puts the gemfile in pkg/

8 - get bex to do a `rake release` after bumping the version number, etc.


# Potential Plans/Ideas (in Priority Order)

## VirtualBox Support (Priority 1a - implemented)

For the VirtualBox provider, the user is encouraged to use a forwarded_port directive (with auto_correct) in their Vagrantfile to expose the docker port.

The plugin will use the :forwarded_ports capability to find the forwarded port number and provide it with localhost as the IP address.

Why? By default Virtualbox creates a box that has no inbound network access.  The pattern is to always forward ports, something that a daemon takes care.

## TLS Certificate Location Support, part 2 (Priority 1b - implemented)

The TLS certificate will always land in a known spot, so we need to add that

## libvirt Support (Priority 2 - not started)

For the libvirt provider, the user will be encouraged to just start the box with no special directives in the Vagrantfile.

The plugin will use the standard-docker port combined with the private IP address of the vagrant box. 

Why? vagrant-libvirt has two challenges:

1. There is currently no implementation of the :forwarded_ports capability to list all forwarded ports.  Additionally the codebase does not keep track of these directly, so it is a non-trivial fix.  (Though the fix may still be easy/medium).
2. Vagrant-libvirt doesn't actually support forwarded ports directly.  Instead a private network is always started and when a forwarded port is requested and instance of `ssh` is spawned to take care of port forwarding.  Since this has non-trivial overhead, there is no reason to use it when we can just get to the box directly

## Anti-Pattern Support (priority 3 - not started)

- Support inbound network and no port forward for VirtualBox.
- Support a port forward on libvirt.
  - note: port collision is not supported in libvirt, yet see:
    https://github.com/pradels/vagrant-libvirt/issues/321

## Other Ideas (not started)

- Support `vagrant adbinfo <target>` (multiple targets?)


## Contact us
For queries regarding the plugin, feel free to put a note on container-tools@redhat.com mailing list.
