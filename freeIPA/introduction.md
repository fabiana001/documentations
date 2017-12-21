# FreeIPA
FreeIPA is an Identity management system sponsored by Red Hat. It aims to provide an easily managed Identity, Policy, and Audit (IPA) [[2]](https://www.freeipa.org/page/IPAv3_Architecture) [[3]](https://www.freeipa.org/page/IPA_and_AD).  The IPA project was started to provide a solution for using Window AD on UNIX/Linux environment, representing thus an AD surrogate for Windows clients in the UNIX/Linux world.

## FreeIpa Components

1.  FreeIPA Directory Service. It is built on the [389 DS](http://directory.fedoraproject.org/) LDAP server;
2. MIT's Kerberos 5 for authentication and single sign-on;
4. Management framework based on  Apache HTTP Server and Python;
5. [Web UI](https://www.freeipa.org/page/Web_UI);
6. [Dog Tag for integrated certification authority (CA)](https://www.freeipa.org/page/PKI) ;
7.  BIND with a custom plugin for the integrated [DNS](https://www.freeipa.org/page/DNS);

## Authentication, Authorization and Accounting

 Authentication, authorization, and accounting (AAA) is a term for a framework for intelligently controlling access to computer resources, enforcing policies, auditing usage, and providing the information necessary to bill for services. These combined processes are considered important for effective network management and security.

**Authentication** is the process in which a user is identified and verified to a system. It requires presenting some sort of identity and credentials, such as a user name and password. The system then compares the credentials against the configured authentication service. If the credentials match and the user account is active, then the user is authenticated. In general the authentication process is composed by two distinct phases [[4](http://www.infosectoday.com/Articles/AU5219_C01.pdf)]:

1. Identification which provides user identity to the security system (generally given in form of user ID). 
2. Authentication which is the process of validating use identity (through credentials).

Once a user is authenticated, the information is passed to the access control service to determine what the user is permitted to do though the authorization process.

**Authorization** refers to rules that determine who is allowed to do what (e.g. Is user X authorized to access resource R? Is user X authorized to perform operation P? Is user X authorized to perform operation P on resource R?).

**Accounting**  is the process of maintaining an audit trail for user actions on the system. This is sometimes referred to as auditing. Accounting may be useful from a security perspective to determine authorized or unauthorized actions; it may also provide information for successful and unsuccessful authentication to the system.


### Authentication with LDAP
The Lightweight Directory Access Protocol (LDAP) is an open, vendor-neutral, industry standard application protocol for accessing and maintaining distributed directory information services over an Internet Protocol (IP) network.

A common use of LDAP is to provide a central place to store usernames, passwords and other users' attributes. This allows many different applications and services to connect to the LDAP server to validate users.

Benefits [[5](http://www.informit.com/store/ldap-directories-explained-an-introduction-and-analysis-9780201787924)]:

1. Make network administration easier:
	- Central management of people information;
	- Central management of machine and computer configuration;
	- Central management of user accounts;
	- Reduced support costs from centralized management.
2. Unify access to network resources
	- Uniform naming convention;
	- Potential for single login to network resources;
3. Provide single destination for users to search for information:
	- Contact information;
	- Central location of network resources;
	- Potential as a catalog of any data.    
4. Improve data management:
	- Improve the consistency of data which is wodely used;
	- Provide centrally managed security for business-critical data;
	- Organize data in a logical structure;
5. Provide repository and lookup for application and service data.

#### LDAP Directory Structure
The protocol provides an interface with directories that follow the 1993 edition of the X.500 model:

- An entry consists of a set of attributes.
- An attribute has a name (an attribute type or attribute description) and one or more values. The attributes are defined in a schema (see below).
Each entry has a unique identifier: its Distinguished Name (DN). This consists of its Relative Distinguished Name (RDN), constructed from some attribute(s) in the entry, followed by the parent entry's DN. Think of the DN as the full file path and the RDN as its relative filename in its parent folder (e.g. if /foo/bar/myfile.txt were the DN, then myfile.txt would be the RDN).

In the following there is an example of a LDAP entry (in LDIF format):
```
 dn: cn=John Doe,dc=example,dc=com
 cn: John Doe
 givenName: John
 sn: Doe
 telephoneNumber: +1 888 555 6789
 telephoneNumber: +1 888 555 1232
 mail: john@example.com
 manager: cn=Barbara Doe,dc=example,dc=com
 objectClass: inetOrgPerson
 objectClass: organizationalPerson
 objectClass: person
 objectClass: top
```


![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")
*Example of LDAP directory tree *

#### Authentication
To start an LDAP session, the LDAP client first must authenticate itself to the service. That is, it must tell the LDAP server who is going to be accessing the data so that the server can decide what the client is allowed to see and do. If the client authenticates successfully to the LDAP server, then when the server subsequently receives a request from the client, it will check whether the client is allowed to perform the request. This process is called access control.

In LDAP, authentication is supplied in the "bind" operation. Ldapv3 supports three types of authentication:

 - anonymous, does not require any type of access info. It is used for testing scope;
	 
 > ldapsearch -x -H ldap://127.0.0.1:389/ -b "dc=example,dc=it" -s sub
	 
 - simple,  consists of sending the LDAP server the fully qualified DN of the client (user) and the client's clear-text password
 
  > ldapsearch -x -H ldap://localhost:389/ -b "dc=example,dc=it" -D "cn=manager,dc=example,dc=it" -w secret 
 
 - SASL authentication.

### Authentication with Kerberos

Kerberos is a network authentication protocol. It is designed to provide strong authentication for client/server applications by using secret-key cryptography (on the basis of tickets). A free implementation of this protocol is available from the Massachusetts Institute of Technology. Kerberos is available in many commercial products as well.

Its designers aimed it primarily at a client–server model and it provides mutual authentication—both the user and the server verify each other's identity. Kerberos protocol messages are protected against eavesdropping and replay attacks. In particular, Kerberos builds on symmetric key cryptography and requires a trusted third party, and optionally may use public-key cryptography during certain phases of authentication. Kerberos uses UDP port 88 by default.

For more details about how the Kerberos Protocol works, see [here](https://it.wikipedia.org/wiki/Protocollo_Kerberos) (section *Operazioni di Kerberos*)

[Benefits](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/authconfig-kerberos):

1. It uses a security layer for communication while still allowing connections over standard ports.
2. It automatically uses credentials caching with SSSD, which allows offline logins.

[Drawbacks](https://en.wikipedia.org/wiki/Kerberos_(protocol)):

1. Single point of failure: It requires continuous availability of a central server. When the Kerberos server is down, new users cannot log in. This can be mitigated by using multiple Kerberos servers and fallback authentication mechanisms.
2. Kerberos has strict time requirements, which means the clocks of the involved hosts **must be synchronized** within configured limits. Network Time Protocol daemons are usually used to keep the host clocks synchronized. 
3. In case of symmetric cryptography adoption (Kerberos can work using symmetric or asymmetric (public-key) cryptography), since all authentications are controlled by a centralized key distribution center (KDC), compromise of this authentication infrastructure will allow an attacker to impersonate any user.
4. Each network service which requires a different host name will need its own set of Kerberos keys. This complicates virtual hosting and clusters.

## FreeIpa Support
FreeIPA uses standard components and protocols so any LDAP/Kerberos (and even NIS) client can interoperate with FreeIPA Directory Server for basic authentication and user/group enumeration. However additional management functionality can be provided.


### FreeIpa Backup and Restore

There are two classes of the disaster scenarios:
1. Server Loss: The FreeIPA deployment loses one, several, or all servers due to a disaster (fire, earthquake, hardware malfunction, etc.) and needs to get back online as soon as possible.
2. Data Loss: FreeIPA data was accidentally deleted, either by a user or by a software bug, and the deletion was propagated to all servers, given that FreeIPA is a multi-master solution.

For more details, see [here](https://www.freeipa.org/page/Backup_and_Restore)


### One-Time Password
One of the best ways to increase authentication security is to require two factor authentication (2FA). One of the more popular options is to use one-time passwords (OTP). 

One-time password (OTP) is a password that is valid for only one authentication session; it becomes invalid after use. Unlike traditional static passwords that stay the same for a longer period of time, OTPs keep changing. OTPs are used as part of two-factor authentication: the first step requires the user to authenticate with a traditional static password, and the second step prompts for an OTP issued by a recognized authentication token.
Authentication using an OTP combined with a static password is considered safer than authentication using a static password alone. Because an OTP can only be used for successful authentication once, even if a potential intruder intercepts the OTP during login, the intercepted OTP will already be invalid by that point.

###  System Security Services Daemon (SSSD)
The System Security Services Daemon (SSSD) is a system service to access remote directories and authentication mechanisms. It connects a local system (an SSSD client) to an external back-end system (a provider). This provides the SSSD client with access to identity and authentication remote services using an SSSD provider. 
For example, these remote services include: an LDAP directory, an Identity Management (IdM) or Active Directory (AD) domain, or a Kerberos realm.

For this purpose, SSSD:
1. Connects the client to an identity store to retrieve authentication information.
2. Uses the obtained authentication information to create a local cache of users and credentials on the client.

Users on the local system are then able to authenticate using the user accounts stored in the external back-end system.
**NOTE** that SSSD **does not create user accounts on the local system**. Instead, it uses the identities from the external data store and lets the users access the local system.

Benefits:
1. **Reduced load on identity and authentication servers**. SSSD has an own identities and credentials cache ( these cache files are stored in the /var/lib/sss/db/ directory). Therefore, SSSD contacts the servers only if the information is not available in the cache.
2. **Offline authentication**. If and when the connection to the central server is lost an application can continue to operate if it has a cache; SSSD provides such cache.
3. **A single user account: improved consistency of the authentication process**.  Everything is managed in one place, including policies. With SSSD, it is not necessary to maintain both a central account and a local user account for offline authentication.

#### Using FreeIPA as Identity Provider for SSSD
SSSD provides access to identity and authentication services and is primarily aimed at the operating system level. SSSD can be configured with FreeIPA in the way to have fine-grained control over a user's access to the system. In particular, SSSD can use FreeIPA for all of its backend functions: : authentication, identity lookups, access, and password changes. 

FreeIPA Client integrates with many Linux native services so that administrator can conveniently configure them centrally, on FreeIPA server. The services mostly use SSSD so that they can also benefit from caching and be available when the client is offline. Example of those service are: SUDO integration (server can provide centralized sudoers to all clients), SELinux user mapping (through HBAC policies), and more. SSSD is configured by default.

For more details see [here](https://www.freeipa.org/images/c/cc/FreeIPA33-sssd-access-control.pdf) and [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/sssd-ad)



## References
[1] [CONFIGURING KERBEROS (WITH LDAP OR NIS) USING AUTHCONFIG, Red Hat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/authconfig-kerberos)

[2] [IPAv3 Architecture](https://www.freeipa.org/page/IPAv3_Architecture)

[3] [IPA and AD](https://www.freeipa.org/page/IPA_and_AD)

[4] [Mechanics of User Identification and Authentication: Fundamentals of Identity Management](http://www.infosectoday.com/Articles/AU5219_C01.pdf)

[5] [ LDAP Directories Explained: An Introduction and Analysis](http://www.informit.com/store/ldap-directories-explained-an-introduction-and-analysis-9780201787924) 
