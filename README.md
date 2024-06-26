# sle_micro Test Env

This repo provides a framework for bringing up one of more sle_micro VMs, via
libvirt, in a repeatable fashion.


# Quick Start

Assuming you have an appropriate version of Ansible available, with
the relevant Ansible collections installed (see below for the Ansible
requirements) to quickly get started you can create an sle_micro test env
by running the `ansible/test_env_create.yml` playbook, like so:

```shell
$ ansible-playbook ansible/sle_micro_create.yml -e sle_micro_image_path=$IMAGEPATH
```

Doing so will automatically setup the `vms.yml` and `vnets.yml` files
under the `settings/` directory using the corresponding example files
and fail, asking you to re-run the playbook.

When you re-run the playbook, as requested, it should create, all going
well, a Libvirt managed sle_micro VM, called `sle_micro`, on the local system,
with the Ansible workload for sle_micro installed, that you can log in to using
the generated SSH config file, `ssh/config`, as follows:

```shell
$ ssh -F ssh/config sle_micro
```

Re-running the `ansible/test_env_create.yml` playbook will destroy and
recreate the configured VMs and associated networks.

If you want to completely cleanup all of these resources you can run
the `ansible/test_env_cleanup.yml` playbook as follows:

```shell
$ ansible-playbook ansible/sle_micro_cleanup.yml
```

See below for further details on requirements, using a remote Libvirt
host, and customising the test env that will be deployed.


# Requirements

## Ansible
It is recommended to use an Ansible Core >= 2.13, which corresponds to
version 6 or later of the Ansible meta package.

NOTE: The Ansible commands must have viable Python Libvirt support.

Tested with Ansible installed in a virtualenv (see below):
  * Ansible meta package version 6.7.0
  * Ansible core version 2.13.7

### Ansible Collection Requirements

Additional collections may be required, beyond those that are provided
by the ansible.builtin collection. The list of required collections are
specified in `ansible/requirements.yml` file, and can be installed by
running the `ansible-galaxy` command as follows:

```shell
$ ansible-galaxy collection install -r ansible/requirements.ym
```

## Libvirt host

The VMs will be created on the specified Libvirt host, defaulting to
the local system. Please ensure that the system has been appropriately
setup and is ready to use.

If using a remote Libvirt host, please ensure that SSH access to the
system works for the current user to connect to the required account
on the Libvirt host.


# Getting Started

See the example files in settings/ for available configuration settings.
Example configuration files are provided with comments explaining what
options are available.

It is highly recommended to at least create the `vnets.yml` and `vms.yml`
files under settings/.

NOTE: If you run the `sle_micro_create.yml` playbook without having created
the required settings files, the example versions will be used to create
these files automatically but those example settings could potentially
conflict with other network resources so please check they are correct.

## Remote Libvirt host

Setup the `libvirt_host.yml` if you want to use a remote Libvirt host.
The default is to use the local system as the Libvirt host.

## Configuring Workloads to be setup

To customise the workloads that will be installed, you can copy the
`settings/workloads.yml.example` to `settings/workloads.yml` and modify
the example list, which, by default, specifies the `ansible` workload
to be installed. These settings match the defaults that will be used if
no workloads are specified.

The configured workloads will automatically be installed when bringing
up the test env, but if you configure additional workloads after having
created a test-env, you can run the `ansible/setup_sle_micro_workloads.yml`
playbook to install them without having to recreate the entire test env.

### Support for runlabels

It is expected that any workload container images specified will have
labels defined that provide 'install' and 'uninstall' capabilities,
though the actual names used can be customised.

## Creating a Test Env

Run `ansible-playbook ansible/sle_micro_create.yml -e sle_micro_image_path=$IMAGEPATH ` to create the VMs,
and associated libvirt networking infrastructure, on the target Libvirt
host, as specified by the configured `settings/*.yml` files.

## Cleaning up a Test Env

Run `bin/ansible-playbook ansible/sle_micro_cleanup.yml` to cleanup any
previously created VMs and associated resources.


# SSH Access to Test Env VMs

As part of bringing up any test VMs for the first time an SSH key pair,
of type ed25519, will be created under the `ssh` directory.

This generated key's public value, along with any others that may have
been specified via *ssh_pub_keys* in the `settings/config.yml`, will be
added to the `authorized_keys` files of both the root and test env user
accounts, via th Ignition config settings, when bringing up the VMs, and
will be used by Ansible to access the VMs.

Additionally `ssh/config` will be generated, with appropriate config
settings, such as ProxyCommand directives when using a remote Libvirt
host, allowing direct access to the created VMs.

For example, assuming default VM config settings, you could SSH to the
*sle_micro* VM, running locally or on a remote Libvirt host as follows:

```
% ssh -F ssh/config sle_micro
Last login: Thu Feb 16 20:45:09 UTC 2023 from 192.168.187.1 on ssh
Have a lot of fun...
sle_micro@sle_micro:~>
```


# Contributing

The code should pass validation with `bin/ansible-lint` (version 6.14 as
of writing), noting that 'yaml[comments]' are explicitly ignored for
the `settings/*.yml` files that may have been created.

## Some rules explicitly ignored in code
Some `# noqa ...` comment markers exist in the code to explicitly ignore
certain rules, such as complaints that handlers should be used for actions
that run because something has changed.
