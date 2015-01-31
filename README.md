# Vagrant Presentation

This is the accompanying text to the Vagrant presentation available
in this repository with comprehensive and structured information
featured on the slides.

*Note: This presentation and text talks primarily about using
Vagrant with libvirt provider provided by vagrant-libvirt
plugin on Fedora and derivates. A lot of the information is general
though.*

## Presenters

- Josef Stříbný
  - GitHub: [@strzibny](https://www.github.com/strzibny)
  - Twitter: [@strzibnyj](https://www.twitter.com/strzibnyj)
  - Blog: [Notes to self](https://nts.strzibny.name)
- Tomáš Hrčka
  - GitHub: [@humaton](https://www.github.com/humaton)

We are part of Red Hat Developer Experience team. Josef does mostly
Ruby and RubyGems packaging for Fedora and RHEL, and started to
maintain Vagrant in Fedora.

## Vagrant World

Vagrant is built around hosts, guests, providers and provisioners. Images are distributed in a provider-specific *.box* files
and project's configuration is saved within *Vagrantfile*.

### Definitions

#### Host

- Host operating system, its detection and capabilities (Windows, RHEL)

Many included by defaults, others by plugins.

#### Guest

- Guest operating system, its detection and capabilities (Ubuntu, OpenBSD, Fedora), **virtualized environment**

Many included by defaults, others by plugins.

#### Provider

- Mean of virtualisation, type of virtual machines used (**VirtualBox**, libvirt)

VirtualBox is upstream default. *libvirt* and others are available via plugins.

#### Provisioner

- Configuration management system to set up (provision) your machine (**Shell, Chef, Puppet**)

Shell, Chef and Puppet are included in Vagrant by default.

#### Vagrant box

- Package format including **provider-specific** image plus metadata

#### Vagrantfile

- Definition of configuration and provisioning for a project **on top of a given box**



## Creating a Vagrant Box

### From Scratch

#### Preparing the Image

To create a desired image for your Vagrant box you need to ensure that
**vagrant** user with sudo rights and without password protection exists.

Vagrant will use this *vagrant* user to access your virtual machine via
SSH which means that SSH has to be present in the virtual machine and
properly set up.

To do so on CentOS 6, one has to install *openssh-server* package, start
and enable the service first:

```shell
# yum install openssh-server
# service sshd start
# chkconfig sshd on
# netstat -tulpn | grep :22
```

For Fedora 21 Server is this already done *by default* so this step can
be skipped.

Vagrant uses SSH keys to access the virtual machine so the public
keys needs to be present on the VM for *vagrant* user.

You can set them easily with the following commands for *vagrant* user:

```shell
$ mkdir .ssh
$ curl -k https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub
> .ssh/authorized_keys
$ chmod 0700 .ssh
$ chmod 0600 .ssh/authorized_keys
```

Since Vagrant would do system provisioning and administration,
it needs to have sudo access on the machine. Using `visudo` command on Fedora
you can edit *sudoers* file and ensure that vagrant won't need a password
and have sudo rights with `%vagrant ALL=(ALL) NOPASSWD: ALL`.

When editing the *sudoers* file make sure that `requiretty` option is
not present (or it is commented out). That is the default on Fedora.
If you would require a real TTY then Vagrant commands sent to the VM
would fail.

Last option to set up when in *sudoers* file is to allow SSH agent
forwarding. You can set that up if you want to use the SSH key from
our local machine inside the Vagrant box.

```
# No password
%vagrant ALL=(ALL) NOPASSWD: ALL
...
# No requiretty, Fedora default
# Defaults    requiretty
...
# SSH agent forwarding
Defaults env_keep += "SSH_AUTH_SOCK"
```

A good practice is also setting up **root** user with password "vagrant",
although it is not necessary for Vagrant to work. Finally you can add
anything you like to the VM. If you would like to do provision with
Chef or Puppet for example, you would need to install them at this time.
But if you would like to have installed Ruby, you can decide whether to
include Ruby in the VM itself or include the installation later in the
Vagrantfile.

#### Packaging VM as Box


vagrant-libvirt provider and Fedora 21 server

- *qcow2* image file named **box.img**
- optional **Vagrantfile** with defaults
- **metadata.json**


```shell
$ cat metadata.json
{
  "provider" : "libvirt",
  "format" : "qcow2",
  "virtual_size" : 40
}
```


```shell
$ tar cvzf vagrant-fedora-21-server.box ./metadata.json vagrant-fedora-21-server.qcow2
```

### In an Automatic Fashion

```shell
$ gem install veewee
$ veewee kvm templates # list available templates for KVM
$ veewee kvm define centos7-x86_64 'CentOS-7.0-1406-x86_64-netinstall' -f
$ veewee kvm build 'centos7-x86_64' --workdir=/home/strzibny
```

Box already comes with <strong>Puppet</strong> and <strong>Chef</strong>.


```shell
$ ls veewee/templates/CentOS-7.0-1406-x86_64-netinstall/
base.sh
chef.sh
cleanup.sh
definition.rb
ks.cfg
puppet.sh
vagrant.sh
virtualbox.sh
vmfusion.sh
zerodisk.sh
```


## Vagrant in Fedora

### Vagrant in Fedora 22

Vagrant has been submitted as a Fedora 22 feature together with *vagrant-libvirt* plugin.
As Vagrant is already built for Fedora 22 in the official repositories it is enough to run:

```shell
# dnf install vagrant-libvirt
```

to install Vagrant with libvirt support. libvirt is the new default provider in Fedora builds.

We would like to introduce a new **@vagrant** group in Fedora that should install all packaged
plugins.

### Older Fedoras and RHEL

Following Copr repositories were created to install Vagrant on Fedora 20, 21 and as a software
collection on RHEL.

[jstribny/vagrant-f20](http://copr.fedoraproject.org/coprs/jstribny/vagrant-f20)
[jstribny/vagrant-f21](http://copr.fedoraproject.org/coprs/jstribny/vagrant-f21)
[jstribny/vagrant1 software collection](http://copr.fedoraproject.org/coprs/jstribny/vagrant1)


- how to use Vagrant as a developer
- where to find boxes
- demo
- attributions
