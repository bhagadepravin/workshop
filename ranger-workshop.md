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

* **Ranger Admin**
    * This service is the central main component to entire Ranger ecosystem. RangerAdmin provides administration capabilities to Ranger users. Ranger Admin
      manages policies and users/groups. It exposes a web UI (Ranger Admin UI), which runs at default port 6080, where user can perform CRUD operations           onpolicies, users/groups and can also access audit records.

*  **Ranger UserSync**
    * This service is responsible for syncing users, groups and group membershipsfrom external sources like Active Directory or an LDAP server. By default,       it syncs the users/groups from the Unix files of the node where UserSync is running.

*  **Ranger TagSync**
    * TagSync is introduced specially for syncing Atlas tags into Ranger.
    * It polls Atlas over REST and gets all the new tags defined in Atlas.

## What are the major Ranger components?

* **Ranger Policy**
     * Ranger policies are the basic building blocks of Ranger authorization. A policy defines three main authorization entities - **which user** 
      (or       group) can perform **what operations** over **which Hadoop resources**.
     * By default, Ranger policies can have only "Allow" conditions. 
     * From HDP 2.6 (and Ranger 0.7.0) onwards, Ranger Policy can also have a "Deny" conditions. By default, the deny conditions are disabled. 
       Users need to enable this by adding `ranger.servicedef.enableDenyAndExceptionsInPolicies=true` in `custom ranger-admin-site` via **Ambari**.
     * In the case, when a policy has both allow and deny conditions, the deny conditions will be given preference over allow conditions.

* **Ranger Service Repositories**
     * Ranger Service Repositories are the service specific containers under which, all the policies that belongs to a service are stored.
     * When a Ranger Plugin is enabled for a host service (like HDFS, Hive or YARN), a default repository for that service is created by Ambari.
     * Service Repositories are named with cluster name. E.g. “MyProdCluster_hadoop” for HDFS repo, “MyProdCluster_hive” for Hive repo and so on.
     * In Ranger Admin UI, the Ranger policies are organised by Service Repositories.

* **Ranger Plugins**
     * Ranger Plugins are in-memory dynamic hook that run inside any Ranger-enabled host service (like Namenode, ResourceManager etc.). Each service loads the Ranger plugin during startup and based on authorization configuration, the plugin is activated.
     * Ranger Plugins keep polling the Ranger Admin over REST for the latest policy data every 30 seconds (configurable). The policy data is stored in JSON format in a on-disk local file, called PolicyCache.
     * PolicyCache file will NOT get updated if there is no change in policy from the last downloaded version (A policy version at service repo level is maintained in Ranger database).
     * Ranger Plugins consult the Ranger policy and decide whether to allow/deny access to end-user.
     * Enabling Ranger Plugins : https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.6.5/bk_security/content/ch_enable_ranger_plugins.html

* **Ranger Audit**
     * Ranger Audit component is responsible for logging and record keeping of all the access attempts (both failed and passed) made by end-user.
     * Ranger Audit records can be stored in HDFS (for archival) and/or SOLR (for faster retrieval). SOLR store can be either Ambari Infra (one node SolrCloud) or an enterprise SolrCloud (multi-node SolrCloud).
     * **Note:** Ambari Infra Solr is, by default, a single node SolrCloud setup. In bigger production setup, it can also be configured to be a multi-node SolrCloud.
     * Ranger Audit records are generated and pushed to store by Ranger Plugins.

