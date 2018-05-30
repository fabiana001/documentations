## Before to start

### Enable NTP (Network Time Protocol)
Kerberos is a time sensitive protocol because its authentication is based partly on the timestamps of the tickets. We need to enable NTP (Network Time Protocol), otherwise clients attempting to authenticate from a machine with an inaccurate clock will be failed by the KDC in authentication attempts due to the time difference. So on **all machines** (both clients and server) we do the following:

> $ yum install ntp

> $ service ntpd start

> $ chkconfig ntpd on

Make sure that on all machines the date/time is set correctly:

> date

If it's not you might need to synchronize the hardware clock (see [here](http://docs.slackware.com/howtos:hardware:syncing_hardware_clock_and_system_local_time) for more details) and/or change your timezone:

> timedatectl set-timezone Europe/Rome

> date



## Create freeIpa client

Install ipa-client package:

> yum install ipa-client

Add freeipa server into /etc/hosts:

> 192.168.0.29    idm.daf.gov.it                  idm

Run the command *ipa-client-install* with the following configurations:
- IPA server name: idm.daf.gov.it
- DNS discovery? [no]: yes
- User authorized to enroll computers: admin
- Password for admin@DAF.GOV.IT: <password admin freeipa>

The output should look like:

```
WARNING: ntpd time&date synchronization service will not be configured as
conflicting service (chronyd) is enabled
Use --force-ntpd option to disable it and force configuration of ntpd

Using existing certificate '/etc/ipa/ca.crt'.
DNS discovery failed to determine your DNS domain
Provide the domain name of your IPA server (ex: example.com): daf.gov.it
Provide your IPA server name (ex: ipa.example.com): idm.daf.gov.it
The failure to use DNS to find your IPA server indicates that your resolv.conf file is not properly configured.
Autodiscovery of servers for failover cannot work with this configuration.
If you proceed with the installation, services will be configured to always access the discovered server for all operations and will not fail over to other servers in case of failure.
Proceed with fixed values and no DNS discovery? [no]: yes
Client hostname: test-sssd-fabiana.daf.gov.it
Realm: DAF.GOV.IT
DNS Domain: daf.gov.it
IPA Server: idm.daf.gov.it
BaseDN: dc=daf,dc=gov,dc=it

Continue to configure the system with these values? [no]: yes
Skipping synchronizing time with NTP server.
User authorized to enroll computers: admin
Password for admin@DAF.GOV.IT:
Enrolled in IPA realm DAF.GOV.IT
Created /etc/ipa/default.conf
Domain daf.gov.it is already configured in existing SSSD config, creating a new one.
The old /etc/sssd/sssd.conf is backed up and will be restored during uninstall.
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm DAF.GOV.IT
trying https://idm.daf.gov.it/ipa/json
[try 1]: Forwarding 'schema' to json server 'https://idm.daf.gov.it/ipa/json'
trying https://idm.daf.gov.it/ipa/session/json
[try 1]: Forwarding 'ping' to json server 'https://idm.daf.gov.it/ipa/session/json'
[try 1]: Forwarding 'ca_is_enabled' to json server 'https://idm.daf.gov.it/ipa/session/json'
Systemwide CA database updated.
Hostname (test-sssd-fabiana.daf.gov.it) does not have A/AAAA record.
Failed to update DNS records.
Missing A/AAAA record(s) for host test-sssd-fabiana.daf.gov.it: 192.168.0.41.
Missing reverse record(s) for address(es): 192.168.0.41.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
[try 1]: Forwarding 'host_mod' to json server 'https://idm.daf.gov.it/ipa/session/json'
Could not update DNS SSHFP records.
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring daf.gov.it as NIS domain.
Client configuration complete.
The ipa-client-install command was successful

```

Test that the client can connect successfully to the IPA domain and can perform basic tasks. For example, check that the IPA tools can be used to get user and group information:


```
$ id
$ getent passwd userID
$ getent group ipausers
```

To create the home dir of a user:

> authconfig --enablemkhomedir --update

For more details see [here](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/users.html#homedir-pammod)

In the following there are few usefull commands:
> ipa user-find fabiana --timelimit=120

> ipa group-find staff --timelimit=120

> ipa group-find --user=fabiana

> ipa group-find --no-user=fabiana //cerca i gruppi a cui fabiana non appartiene
