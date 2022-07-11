# Ranger Workshop

LAB:
-
-
  -
  -
  
LAB:

## Why Apache Ranger?

* Centralized security administration to manage all security related tasks in a central UI or using REST APIs.
* Fine grained authorization to do a specific action and/or operation with Hadoop component/tool and managed through a central administration tool
* Standardize authorization method across all Hadoop components.
* Enhanced support for different authorization methods - Role based access control, attribute based access control etc.
* Centralize auditing of user access and administrative actions (security related) within all the components of Hadoop.


## What is Apache Ranger?

Apache Ranger is an Hadoop component used for Authorization and Auditing. Meaning, Rangeris used to decide whether to allow a user access to a Hadoop resources (HDFS file, Hive table or a YARN scheduler queue) or not. This access decision is taken based on a Ranger policy.

Ranger is also used for its auditing capabilities. It stores the access records from various Hadoop services to a central location (Solr and/or HDFS) where these audit logs can be leveraged further.

Ranger users are typically the organizational policy administrator who allow/deny access to Hadoop resources to their end users. End users usually don’t need to access to Ranger directly.For them, Ranger will always be a behind-the-scene component.

## What are the services that Ranger run?

● **Ranger Admin**

    * This service is the central main component to entire Ranger ecosystem. RangerAdmin provides administration capabilities to Ranger users. Ranger Admin
      manages policies and users/groups. It exposes a web UI (Ranger Admin UI), which runs at default port 6080, where user can perform CRUD operations           onpolicies, users/groups and can also access audit records.

● **Ranger UserSync**
    * This service is responsible for syncing users, groups and group membershipsfrom external sources like Active Directory or an LDAP server. By default,       it syncs the users/groups from the Unix files of the node where UserSync is running.

● **Ranger TagSync**
    * TagSync is introduced specially for syncing Atlas tags into Ranger.
    * It polls Atlas over REST and gets all the new tags defined in Atlas.
