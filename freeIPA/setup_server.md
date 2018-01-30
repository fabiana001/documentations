# Setup freeIpa server on Centos 7

## Install FreeIpa
### Preparing IPA Server

You can either set the hostname when you create the server or set it from the command line after the server is created, using the hostname command:

> hostname idm.staging.daf.teamdigitale.it

update the package repository.
>  yum update

Disable firewalld.
> systemctl disable firewalld
> systemctl stop firewalld
> systemctl status firewalld

### Setting Up DNS

You need to verify that the DNS names resolve properly. You can use the dig command for this. Install the bind-utils package to get dig and other DNS testing utilities.

> yum install bind-utils

Running :
> dig +short idm.staging.daf.teamdigitale.it A

you should see the ip address (e.g. 192.168.20.13). The following command do the vice versa:

> dig +short -x 192.168.20.13

Finally edit */etc/hosts* for changing the host file to point the server's hostname to its external IP address. The file should look as following:

    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    #
    192.168.20.13 idm.staging.daf.teamdigitale.it idm.staging.daf.teamdigitale.it


### Configuring the Random Number Generator

Install and enable rngd:

> yum install rng-tools
> systemctl start rngd
> systemctl enable rngd
> systemctl status rngd

The output should include active (running) in green.

### Installing the FreeIPA Server

Install freeipa server:

> yum install ipa-server
> ipa-server-install

In addition to authentication, FreeIPA has the ability to manage DNS records for hosts. This can make provisioning and managing hosts easier. In this tutorial we will not be using FreeIPA's integrated DNS. It is not needed for a basic setup.

    WARNING: conflicting time&date synchronization service 'chronyd' will be disabled in favor of ntpd
    Do you want to configure integrated DNS (BIND)? [no]: no

**Diagnostic Steps**
It is highly recommended to watch the messages that are logged to the journal while IPA server starts to check for any potentially significant errors:

> journalctl -f

## Verifying the FreeIPA Server Functions

First, verify that the Kerberos realm installed correctly by attempting to initialize a Kerberos token for the admin user.  Then, verify that the IPA server is functioning properly.

> kinit admin

> ipa user-find admin

It should return something like the following:

    --------------
    1 user matched
    --------------
    User login: admin
    Last name: Administrator
    Home directory: /home/admin
    Login shell: /bin/bash
    Principal alias: admin@STAGING.DAF.TEAMDIGITALE.IT
    UID: 1305200000
    GID: 1305200000
    Account disabled: False
    ----------------------------
    Number of entries returned 1
    ----------------------------

Now, we should also be able to access the web UI at https://idm.staging.daf.teamdigitale.it/ipa/ui/

## References

[1] [How To Set Up Centralized Linux Authentication with FreeIPA on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-centralized-linux-authentication-with-freeipa-on-centos-7#step-4-%E2%80%94-installing-the-freeipa-server)

[2] [How to manually start IdM/IPA 4.0-4.4 server services for troubleshooting purposes in RHEL 7.0-7.3](https://access.redhat.com/solutions/1605213)

