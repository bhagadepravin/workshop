# Kerberos Workshop

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
An authentication server that performs the initial authentication and issues ticket-granting tickets for users.
A ticket granting server that issues service tickets that are based on the initial ticket-granting tickets.
A principals database of secret keys for all the users and services that it maintains.

![kdc](https://user-images.githubusercontent.com/28974904/173366640-c2e90f0d-6cf0-4330-8c9b-49f30944c600.jpeg)



### Understanding Kerberos authentication:
![Understanding Kerberos authentication](https://user-images.githubusercontent.com/28974904/173363503-e9db7171-610e-4933-a7b1-7bfa089b2632.png)
