rdo-vm-factory
==============

Provides automated setup of RDO with various deployment scenarios. The
deployment scenarios mainly focus on Keystone and integration with
external identity sources.

Automation typically covers creation of VMs created from cloud images,
installation and configuration of RDO, and installation and configuration
of other software that is needed for the particular scenario (such as
Active Directory or FreeIPA).

Global configuration is defined in global.conf. This mainly consists
of virtual network settings that will be used by the VMs that are
created by the individual scenarios.

Scenario specific configuration is defined in individual .conf files.
Each VM has it's own conf file, which defines things such as the OS
image and package repositories that should be used, packages to install,
and user and password information. The actual configuration steps that
are used to set up software after VM creation are in cloud-init user-data
scripts, which are prefixed with 'vm-post-cloud-init-'.

To deploy a particular scenario, you should just be able to run 'setup.sh'
in the scenario directory that you want to deploy. This needs to be run
as a user with passwordless sudo abilities. Once a VM is iniitally created,
you can ssh into it and follow the progress by tailing the following log
files:

  - /var/log/cloud-init.log - initial VM creation and package installtion

  - /var/log/vm-post-cloud-init-\*.sh.log - user-data script execution

When you are finished with the VMs, you can just run cleanup.sh in the
scenario directory to clean everything up.

## Setups
### rdo-kerberos-setup

This scenario sets up FreeIPA on one VM and RDO on a second. Deployment
details include:

  - Keystone running in httpd
  - v3 cloud sample policy is used for a multi-domain deployment
  - cloud_admin administers the overall cloud (admin_domain)
  - Service users live in the 'default' domain
  - FreeIPA set up as a separate Keystone domain using LDAP identity
  - FreeIPA 'admin' user administers the FreeIPA domain
  - Kerberos authentication is configured for Keystone at /krb

When the deployment is complete, you should be able to do the following
to test Kerberos authentication:

```
[rdouser@rdo]$ source ~/keystonerc_kerberos 
[rdouser@rdo ~(keystone_kerberos)]$ kinit admin
Password for admin@RDODOM.TEST: 
[rdouser@rdo ~(keystone_kerberos)]$ openstack role list
+----------------------------------+---------------+
| ID                               | Name          |
+----------------------------------+---------------+
| 22b79d81c9ed48dda625bf7de6df7645 | admin         |
| 45ec68348ee84917a992d4e401d253d8 | ResellerAdmin |
| 89ac3797a4ef46d89c0a9f69a1de6851 | SwiftOperator |
| 9fe2ff9ee4384b1894a90878d3e92bab | _member_      |
+----------------------------------+---------------+
[rdouser@rdo ~(keystone_kerberos)]$
```

Other keystonerc files are supplied to perform tasks as the cloud admin, or
for non-kerberized FreeIPA admin tasks.
