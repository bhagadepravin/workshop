<p align="center">
  <a href="https://web.mit.edu/kerberos/">
    <img alt="Jamify" src="https://user-images.githubusercontent.com/28974904/173368978-5b51bde7-4660-4ed4-9b76-aeeb78ef85dc.jpeg" />
  </a>
</p>
<h1 align="center">
   Kerberos Workshop <br/>
</h1>

In this workshop we will configure and understand kerberos authentication to use with Hadoop environment. For this workshop configure HDP clusters in lab. We will use Ambari servers of both the clusters as KDC and one of the node as KDC client.

------------------------------------------------------------------------------------------------------------------------------

LAB : 
  - [Understanding Kerberos authentication:](https://github.com/bhagadepravin/workshop/blob/main/kerberos-workshop.md#why-kerbeors)
  - [Install and configure KDC.](https://github.com/bhagadepravin/workshop/blob/main/kerberos-workshop.md#install-mit-kdc)
  - [Enabling Kerberos Authentication Using Ambari](https://github.com/bhagadepravin/workshop/blob/main/kerberos-workshop.md#enable-kerberos-on-hdp-cluster-with-mit-kdc-kerberos)
  - [Understanding Spnego authentication](https://github.com/bhagadepravin/workshop/blob/main/kerberos-workshop.md#understanding-spnego-authentication)
    - [How to use curl cmd to access Spnego enabled UI's](https://github.com/bhagadepravin/workshop/blob/main/kerberos-workshop.md#1-access-a-spnego-enabled-ui-via-curl-cmd)
    - [How to configure Browser to access Spnego enabled UI's](https://github.com/bhagadepravin/workshop/blob/main/kerberos-workshop.md#2-access-spnego-enabled-ui-via-browser)


# LAB: 

## Why Kerbeors?
* Kerberos is a solution to your network security problems.
* It provides the tools of authentication and strong cryptography over the network to help you secure your information systems across your entire enterprise.

##  What is Kerbeors?
* Kerberos is a network authentication protocol. 
* It is designed to provide strong authentication for client/server applications by using secret-key cryptography. 
* Kerberos is a system for authenticating access to distributed services:


Kerberos runs as a third-party trusted server known as the **Key Distribution Center (KDC)**. Each user and service on the network is a principal.

The KDC has three main components:

* An **Authentication Server (AS)** that performs the initial authentication and issues ticket-granting tickets for users.
* A **Ticket Granting Server (TGS)** that issues service tickets that are based on the initial ticket-granting tickets.
* A principals **Database** of secret keys for all the users and services that it maintains.

![kdc](https://user-images.githubusercontent.com/28974904/173366640-c2e90f0d-6cf0-4330-8c9b-49f30944c600.jpeg)


**Step-1:-** 
User login and request services on the host. Thus user requests for ticket-granting service. 
 
**Step-2:-**
 Authentication Server verifies user’s access right using database and then gives ticket-granting-ticket and session key. Results are encrypted using the Password of the user. 
 
**Step-3:-**
 The decryption of the message is done using the password then send the ticket to Ticket Granting Server. The Ticket contains authenticators like user names and network addresses. 
 
**Step-4:-** 
 Ticket Granting Server decrypts the ticket sent by User and authenticator verifies the request then creates the ticket for requesting services from the Server. 
 
**Step-5:-**
 The user sends the Ticket and Authenticator to the Server. 
 
**Step-6:-**
 The server verifies the Ticket and authenticators then generate access to the service. After this User can access the services. 

### Understanding Kerberos authentication:
![Understanding Kerberos authentication](https://user-images.githubusercontent.com/28974904/173363503-e9db7171-610e-4933-a7b1-7bfa089b2632.png)

## Few Kerberos Terms:

## Kerbeors Principal:-
* A principal is an identity in the system; a person or a thing like the hadoop namenode which has been given an identity.
* In Hadoop, a different principal is usually created for each service and machine in the cluster, such as hdfs/node1, hdfs/node2, ... etc. These principals would then be used for all HDFS daemons running on node1, node2, etc.

## Kerberos Realm:-
* A Kerberos Realm is the security equivalent of a subnet: all principals live in a realm. 
Examples: **ENTERPRISE**, **HCLUSTER**
* This would allow a Hadoop cluster with its own KDC and realm to trust the ENTERPRISE realm, but for the enterprise realm to not trust the HCLUSTER realm, and hence all its principals. This would prevent a principal **hdfs/node1@HCLUSTER** from having access to the **ENTERPRISE** systems.

## Keytab:-
* A (binary) file containing the secrets needed to log in as a principal
* It contains all the information to log in as a principal, so is a sensitive file.

## Tickets:-
* A ticket is something which can be passed to a server to identify that the caller and to provide a secret key that can be used between the client an the server —for the duration of the ticket's lifetime. 
* It is all that a server needs to authenticate a client: there's no need for the server to talk to the KDC.

# Install MIT KDC

**Step 1.**
On one host install and configure KDC with realm name ADSRE.COM.

```bash
$ yum install krb5-server krb5-libs krb5-workstation
$ vi /etc/krb5.conf 
[...]
default_realm=ADSRE.COM

[realms]
ADSRE.COM {
  kdc=<Hostname>
  admin_server=<Hostname>
}

$ kdb5_util create -s
$ service krb5kdc start
$ service kadmin start
```

**Step 2.** Create a user principal and a admin principal.
```bash
$ kadmin.local
kadmin.local: listprincs
kadmin.local: addprinc user1@ADSRE.COM
kadmin.local: addprinc admin/admin@ADSRE.COM
```
**Step 3.** Review the kdc.conf file and configuration options.
```bash
cat /var/kerberos/krb5kdc/kdc.conf
cat /var/kerberos/krb5kdc/kadm5.acl
```
* https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/kdc_conf.html 
* https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/kadm5_acl.html

Configure `kdc.conf` so that keys can be created with different encryption types (rc4-hmac) like below:
```bash
[realms]
ADSRE.COM = {
master_key_type = aes256-cts
acl_file = /var/kerberos/krb5kdc/kadm5.acl
dict_file = /usr/share/dict/words
admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
}
```
Review the `kadm5.acl`, refer documentation for possible acl and the format.

https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/kadm5_acl.html

**Step 4.** On client install the krb5 client pkgs and configured krb5.conf.
```bash
#yum install krb5-libs krb5-workstation
(Update krb5.conf with REALM and kdc info) #vi /etc/krb5.conf [...] default_realm=ADSRE.COM [..]

[realms]

ADSRE.COM
{
 admin_server = <kadmin_server_hostname>
  kdc = <kdc_server_hostname>
}
```
**Step 5.** Verify if you can authenticate with the userprincipal user1 created on kdc in step 2.

```bash
$ kinit user1
```

### Optional, Use bash script to install MIT KDC 

# Hands On Lab:

#### Setup MIT KDC
- https://gist.github.com/bhagadepravin/f1f6a3d9c26e24eee76d8288de3f6f89

#### Enable kerberos on HDP cluster with MIT KDC Kerberos.
- https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.4/authentication-with-kerberos/content/enabling_kerberos_authentication_using_ambari.html


# Understanding Spnego authentication:

Kerberos is a network authentication protocol for client/server applications, and SPNEGO provides a mechanism for extending Kerberos to Web applications through the standard HTTP protocol.


**Example:** 

### 1. Access a Spnego Enabled UI via `curl` cmd

Currently we are trying to access SOLR WebUI, which is Spnego enabled, which means it require Kerberos authentication for access.
We will kinit with admin/solr principal.

with curl cmd we will pass `--negotiate -u:" along with service url.

when I use the --negotiate option, curl initially sends a request with no credentials, and then when it gets a 401,
it sends another request, this time with the Kerberos credentials. This is all a normal part of the HTTP Negotiate protocol.

In the end you will see HTTP principal in klist cmd, Once curl cmd was execute successfully, As Spnego uses HTTP protocol
```bash
[root@mstr3 keytabs]# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: infra-solr/mstr3.hdp265.centos7.adsre@ACCEL2HDP.COM

Valid starting     Expires            Service principal
06/14/22 19:40:00  06/15/22 19:40:00  krbtgt/ACCEL2HDP.COM@ACCEL2HDP.COM
	renew until 06/21/22 19:40:00
  
  
[root@mstr3 keytabs]# curl -ik --negotiate -u : "http://$(hostname -f):8886/solr/admin/collections?action=clusterstatus&wt=json&indent=true"
HTTP/1.1 401 Authentication required
WWW-Authenticate: Negotiate
Set-Cookie: hadoop.auth=; Path=/; Domain=mstr3.hdp265.centos7.adsre; Expires=Thu, 01-Jan-1970 00:00:00 GMT; HttpOnly
Content-Type: text/html; charset=ISO-8859-1
Cache-Control: must-revalidate,no-cache,no-store
Content-Length: 334

HTTP/1.1 200 OK
WWW-Authenticate: Negotiate YGoGCSqGSIb3EgECAgIAb1swWaADAgEFoQMCAQ+iTTBLoAMCARKiRARCnLdWCA6ZXewSPuUh6KfmjWmLcRIXJEWCJNfB2loGJhJnlM1sWWj5fJnZdlQUHUEQZ+iFQt9meEdJw7nBvw87Lqe/
Set-Cookie: hadoop.auth="u=infra-solr&p=infra-solr/mstr3.hdp265.centos7.adsre@ACCEL2HDP.COM&t=kerberos&e=1655251939613&s=bTPlg2RlXGulLmmjdwWK5inlzrM="; Path=/; Domain=mstr3.hdp265.centos7.adsre; Expires=Wed, 15-Jun-2022 00:12:19 GMT; HttpOnly
Content-Type: application/json; charset=UTF-8
Transfer-Encoding: chunked

{
  "responseHeader":{
    "status":0,
    "QTime":1376},
  "cluster":{
    "collections":{},
    "properties":{"urlScheme":"http"},
    "live_nodes":["mstr3.hdp265.centos7.adsre:8886_solr"]}}


[root@mstr3 keytabs]# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: infra-solr/mstr3.hdp265.centos7.adsre@ACCEL2HDP.COM

Valid starting     Expires            Service principal
06/14/22 19:40:00  06/15/22 19:40:00  krbtgt/ACCEL2HDP.COM@ACCEL2HDP.COM
	renew until 06/21/22 19:40:00
06/14/22 19:42:19  06/15/22 19:40:00  HTTP/mstr3.hdp265.centos7.adsre@
	renew until 06/21/22 19:40:00
06/14/22 19:42:19  06/15/22 19:40:00  HTTP/mstr3.hdp265.centos7.adsre@ACCEL2HDP.COM
	renew until 06/21/22 19:40:00
```

### 2. Access Spnego Enabled UI via Browser.

* For Windows Machine: https://community.cloudera.com/t5/Community-Articles/User-authentication-from-Windows-Workstation-to-HDP-Realm/ta-p/245957
* For Mac Machine : https://community.cloudera.com/t5/Community-Articles/Configure-Mac-and-Firefox-to-access-HDP-HDF-SPNEGO-UI/ta-p/249092
