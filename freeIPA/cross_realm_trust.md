# Kerberos Cross-Realm Authentication

If you use Cloudera Manager to enable Hadoop security on your cluster, the Cloudera Manager Server will create several principals (e.g. hdfs, hbase, etc.) and then generate keytabs for those principals. Cloudera Manager will then deploy the keytab files on every host in the cluster. As Cloudera recommends we use a dedicated local MIT Kerberos KDC and realm for the Hadoop cluster. Cloudera Manager automatically manages the creation and storage of the service principals for the Hadoop cluster.

After configuring the default Kerberos realm for the cluster you want Cloudera Manager to manage, you must set up **one-way cross-realm trust** between the cluster-dedicated KDC (i.e. MIT KDC) and our FreeIPA KDC. In this way, users belonging to a different realm can be authenticated by the Cloudera Services. In particular, we must add the same principal (i.e. krbtgt/MIT_REALM@FREEIPA_REALM) on both databases:
These principals (one for each database) should all have the **same passwords, key version numbers, and encryption types**. 

Run the following commands in the kadmin.local or kadmin shell on both KDCs:

1) On MIT Kerberos Server
> kadmin.local
 addprinc krbtgt/MIT_REALM@FREEIPA_REALM
 password: abc1234

2) On FreeIpa Server

> kadmin.local -x ipa-setup-override-restrictions
 addprinc krbtgt/MIT_REALM@FREEIPA_REALM
 password: abc1234

Finally **Add Domain to MIT KDC Configuration**:


```bash
[libdefaults]
 ...
[realms]

MIT_REALM = {
	kdc = mit_kdc
	admin_server = mit_kdc
}

FREEIPA_REALM = {
	kdc = freeipa_kdc
	admin_server = freeipa_kdc
}
```

Fore more details about Cross-Realm Trust setting, see [here](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/cm_sg_kdc_def_domain_s2.html).


## References
[1] [Cloudera Cross-Realm Trust tutorial](https://www.cloudera.com/documentation/enterprise/5-12-x/topics/cm_sg_kdc_def_domain_s2.html)

[2] [Kerberos Cross-Realm Authentication slides](http://www.dia.uniroma3.it/~afscon09/docs/angius.pdf)
