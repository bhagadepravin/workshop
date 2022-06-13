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

LAB 1: 
  - Understanding Kerberos authentication:
  - Install and configure KDC.


# LAB 1: 

## Why Kerbeors?
Kerberos is a solution to your network security problems. 
It provides the tools of authentication and strong cryptography over the network to help you secure your information systems across your entire enterprise.

##  What is Kerbeors?
Kerberos is a network authentication protocol. 
It is designed to provide strong authentication for client/server applications by using secret-key cryptography. 
Kerberos is a system for authenticating access to distributed services:


Kerberos runs as a third-party trusted server known as the Key Distribution Center (KDC). Each user and service on the network is a principal.

The KDC has three main components:

* An **authentication server (AS)** that performs the initial authentication and issues ticket-granting tickets for users.
* A **ticket granting server (TGS)** that issues service tickets that are based on the initial ticket-granting tickets.
* A principals **database** of secret keys for all the users and services that it maintains.

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

* Kerbeors Principal:- 
A principal is an identity in the system; a person or a thing like the hadoop namenode which has been given an identity.
In Hadoop, a different principal is usually created for each service and machine in the cluster, such as hdfs/node1, hdfs/node2, ... etc. These principals would then be used for all HDFS daemons running on node1, node2, etc.

* Kerberos Realm:-

A Kerberos Realm is the security equivalent of a subnet: all principals live in a realm. 

Examples: ENTERPRISE, HCLUSTER
This would allow a Hadoop cluster with its own KDC and realm to trust the ENTERPRISE realm, but for the enterprise realm to not trust the HCLUSTER realm, and hence all its principals. This would prevent a principal hdfs/node1@HCLUSTER from having access to the ENTERPRISE systems.

* Keytab:-
A (binary) file containing the secrets needed to log in as a principal
It contains all the information to log in as a principal, so is a sensitive file.

* Tickets:-
A ticket is something which can be passed to a server to identify that the caller and to provide a secret key that can be used between the client an the server —for the duration of the ticket's lifetime. It is all that a server needs to authenticate a client: there's no need for the server to talk to the KDC.

