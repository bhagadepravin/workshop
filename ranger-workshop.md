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


![image](https://user-images.githubusercontent.com/28974904/178187675-eca13dec-b7f4-481c-a32f-12e0dd7a772d.png)
![image](https://user-images.githubusercontent.com/28974904/178187718-2ff8a911-9a4e-48aa-a493-148c8ccaf9d4.png)


*  **How does user get authorized in Ranger?**
     * Ranger Authorization takes place inside the host services (like HDFS NameNode, YARN ResourceManager and HiveServer2 etc.)

*  **How does Ranger audit work?**
     * Ranger plugins send their audit event (whether access was granted or not and based on which policy) directly to the configured sink for audits, which can be HDFS, Solr or both.

<img width="824" alt="image" src="https://user-images.githubusercontent.com/28974904/178188017-bfbdc8a1-88aa-4909-afe9-1e3b1fd2b81b.png">

1. When the plugin is enabled AND no specific policy is in place for access to some object, the plugin will fall back to enforcing the standard component level Access Control Lists (ACL’s). For HDFS that would be the user : rwx / group : rwx / other : rwx ACL’s on folders and files.
2. Once this defaulting to component ACL’s happens the audit events show a ‘ - ‘ in the ‘Policy ID’ column instead of a policy number. If a Ranger policy was in control of allowing/denying access, the policy number is shown in the first column.


## Ranger Troubleshooting

### Ranger Policy / Plugins
<img width="943" alt="image" src="https://user-images.githubusercontent.com/28974904/178188599-f6c3901f-cdf2-404d-8e55-f507b57454cc.png">

* **1.1.0 Check if Ranger Plugin is enabled**
     * To check if Ranger Plugin is enabled or not, one can use Ambari Dashboard. If user wants to use Ranger for a service (say HDFS) and Ranger Plugin for that host service is not enabled, then we need to enable the Plugin first.
     * Please navigate to  `Ambari > Ranger > Configs > Ranger Plugin` to check if a Ranger Plugin is enabled.
<img width="907" alt="image" src="https://user-images.githubusercontent.com/28974904/178188696-ed5bf8be-c01f-44b6-9735-8978398810db.png">


* **1.1.1 Enable Plugin in Ranger**
     * As shown above, enable Ranger plugin via `Ambari > Ranger > Configs > Ranger Plugin` option.
     * As we can see, one can enable & disable Ranger Plugins for various host services from this page. Once a plugin is enabled or disabled, Ambari will notify to restart the respective host service.
     * This is the recommended & most common way to enable / disable any Ranger Plugin.

* **1.2.0 Check if Ranger plugin is loaded**
     * Next, one should check if Ranger plugin gets loaded by the host service correctly. 
     * The quickest way to check this is to see the host service process ( on the node and make sure that it has ranger JARs loaded. Please use this sequence of commands:
```bash
# Find the Namenode PID and use in the next command
$ ps -ef | grep namenode

# In the output below, ranger JARs should be listed if Ranger plugin is correctly loaded by HDFS NameNode process
$ lsof -p <nn-pid> | grep ranger-plugin
```
<img width="830" alt="image" src="https://user-images.githubusercontent.com/28974904/178188942-3f1668e8-8368-4a34-ae72-0ac5ba7eb6af.png">

* Similar commands can be used for other host services like HiveServer2, YARN ResourceManager, Ranger KMS, HBASE Master, Knox etc. This is the most accurate and safest way to confirm that a Ranger Plugin is indeed loaded.
* Another way to check this - log in to Ranger Admin UI and confirm whether the Default Ranger service repository has been created with default policy.

<img width="833" alt="image" src="https://user-images.githubusercontent.com/28974904/178188998-fcad289e-d25a-4030-9da7-5c2c3ba42387.png">
<img width="837" alt="image" src="https://user-images.githubusercontent.com/28974904/178189008-8df592d2-63a1-4714-a86e-c6a89adb666f.png">

* **1.2.1 Enable Plugin in Hadoop Service**
     * Sometimes, Ranger Plugin may not be enabled correctly by Ambari. In those cases, one should check Ranger configuration at host service level in Ambari.
     * One can also enable / disable Ranger Plugin directly from host service configuration in Ambari. These options can be accessed from `Ambari > Host Service (HDFS) > Configs > Advanced ranger-hdfs-plugin-properties` :
<img width="819" alt="image" src="https://user-images.githubusercontent.com/28974904/178189084-c96f6469-edc1-4a15-85df-366ce67a0545.png">

* Along with the above checkbox, one needs to fill necessary details (default values would work for most properties).

* **1.3.0 Check if PolicyCache is updated**
     * A good Ranger Plugin should connect to Ranger Admin over REST and download the latest policy information in a on-disk PolicyCache file on the host service node

     * To check if PolicyCache is updated:
        * Check the last modified date/time stamp of the json file - should be a recent timestamp
        * Make some changes (add some user/resource) in any Ranger policy (& wait for 30 seconds) and see if Ranger plugin updates the same in the PolicyCache file.
<img width="829" alt="image" src="https://user-images.githubusercontent.com/28974904/178189273-95b3e79a-f062-400d-a608-ea80869721bb.png">
Or check Plugin Status Tab

* **1.3.1 Make sure PolicyCache is getting updated**
- If PolicyCache file is not getting updated, then here are a couple of things that we can do:
     * Check the file permission of PolicyCache file - should be owned by a user who is running the host service (hdfs for NameNode, yarn for YARN RM, hive for HiveServer2 etc.)
     * Try restarting the host service (Get customer’s permission to do this, if this is a production environment)
     * Check for errors in host service logs, usually there could be connection issue between host service and Ranger Admin.
     * You may need to enable debug for Ranger plugin and restart the host service again to get the root cause of the error.

* **1.4.0 Check if a Ranger Policy actually defined**
-- If PolicyCache is updated correctly and still authorization is not working, then check if there is a policy defined which allows access for the combination of these:
     * user (or group that the user is part of)
     * Resource (that is being accessed)
     * operation (that is being performed by the said user)

* **1.4.1 Create a Ranger Policy**
     * If there is no policy defined, then create a policy via Ranger Admin UI under the right Service Repo to allow access to this user/group + operation + resource combination. For example, here’s how to create a policy for `hr1` user to allow him to perform `read` operation on `/data/office` location `in HDFS`:

<img width="832" alt="image" src="https://user-images.githubusercontent.com/28974904/178189545-94ca0365-110f-4057-9be5-772e4dee6446.png">
Once the new policy is saved, wait for 30 seconds for plugins to download the new policy information. This plugin operation can be verified by Audit tab in Ranger Admin UI, like this:

<img width="830" alt="image" src="https://user-images.githubusercontent.com/28974904/178189582-fa47041e-bd5a-4d6b-a506-76e75da4b9a1.png">

For your service name (bahubali_hadoop in this example), if ‘Export Date’ shows a recent date/time and ‘HTTP Response Code’ shows 200, then new policy is downloaded correctly.

* **1.5.0 Check Ranger Audit logs for Access Enforcer**
     * If access is still denied, then check Ranger Audit logs in Ranger Admin UI and search for the user name who is trying to access the resource.
     * Once you get to the latest `"Denied"` record for this user, then look for value `“Access Enforcer”` tab.
        * If this value is `“hadoop-acl”`, that means the permission denied is due to POSIX ACLs in HDFS filesystem.
        * If this is value is `“ranger-acl”`, that means that permission denied is due a Ranger policy, whose ID can be found in 'Policy ID' column.
<img width="831" alt="image" src="https://user-images.githubusercontent.com/28974904/178189828-9c6e1d52-3dba-4ece-80c2-d76d48a0ce5d.png">

For example, since we have created policy for ‘hr1’ user to read on /data/office and when /data/office is owned by ‘hdfs’ user, then any read operation will be allowed due to ‘ranger-acl’ but any write operation will be denied due to ‘hadoop-acl’.
<img width="832" alt="image" src="https://user-images.githubusercontent.com/28974904/178189875-e4bdcc26-7d64-4004-8c98-d394c46d466e.png">

Once you have found the Access Enforcer, then modify either Ranger Policy or HDFS ACLs to allow / deny access to user as per customer’s problem statement.

* **1.6.0 Check Host Service logs**
If user authorization is still not working as expected, then we should look into the host service logs. Enabling debug would always help to understand the real reason for the error. So,
     * Enable debug in host service (e.g. HDFS NameNode)
     * Ask customer to perform the same operation which was failing
     * Look for any error / stack trace in the debug log
     * Work further based on the error message.

* **1.7.0 Collect Configs + Logs**

### Ranger Troubleshooting Data Collection

    * **Configuration files**
        * a. Ranger Admin: /etc/ranger/admin/conf/*
        * b. Ranger UserSync: /etc/ranger/usersync/conf/*
        * c. Ranger TagSync: /etc/ranger/tagsync/conf/*

    * **Log files**
        * a. Ranger Admin: From /var/log/ranger/admin/ directory, collect xa_portal.log, access_log.* (latest one) and catalina.out
        * b. Ranger UserSync: From /var/log/ranger/usersync directory, collect usersync.log
        * c. Ranger TagSync: From /var/log/ranger/tagsync directory, collect tagsync.log and tagsync.out
        * d. Host Service logs: For individual services, debug logs with debug enabled for “org.apache.ranger.*” classes would help a lot. For example:
              * i. HDFS - NameNode logs (DataNode logs NOT required)
              * ii. Hive - HiveServer2 / Hive Metastore logs
              * iii. HBase-HBaseMasterlogs
              * iv. Atlas - Atlas server logs

    * **PolicyCache files**
        * a. In HDP 3.1 and below, the PolicyCache file can be found in /etc/ranger/<Service-Repo-name>/policycache directory on the node where respective Hadoop service is running. For example,
            * i. HDFS - /etc/ranger/bahubali_hadoop/policycache/hdfs_bahubali_hadoop.json
            * ii. Hive - /etc/ranger/bahubali_hive/policycache/hiveServer2_bahubali_hive.json
            * iii. KMS-/etc/ranger/bahubali_kms/policycache/kms_bahubali_kms.json
