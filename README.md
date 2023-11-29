# SnowPro Core: Notes 

<details>
  <summary>Access Control</summary>

  ## Access Control
[Ref](https://docs.snowflake.com/en/user-guide/security-access-control-overview)

### Framework
* **Discretionary Access Control (DAC):** Each object has an owner, who can in turn grant access to that object.
* **Role-based Access Control (RBAC):** Access privileges are assigned to roles, which are in turn assigned to users.

The key concepts to understanding access control in Snowflake are:

**A. Securable object**

  * An entity to which access can be granted.
    * Unless allowed by a grant, access is denied.
    
    **Hierarchy of objects and containers**


![RoleHierarchy](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/RoleHierarchy.png)

**B. Role**
  * An entity to which privileges can be granted.
  * Roles are in turn assigned to users
  * *Note that roles can also be assigned to other roles, creating a role hierarchy.

 In Snowflake there are these System-Defined Roles
* **ORGADMIN** (aka Organization Administrator)
  * Role that manages operations at the organization level. More specifically, this role:
    * Can create accounts in the organization.
    * Can view all accounts in the organization (using SHOW ORGANIZATION ACCOUNTS) as well as all regions enabled for the organization (using SHOW REGIONS).
    * Can view usage information across the organization.

* **ACCOUNTADMIN**  (aka Account Administrator)
  * Role that encapsulates the SYSADMIN and SECURITYADMIN system-defined roles.
    * It is the top-level role in the system and should be granted only to a limited/controlled number of users in your account.

* **SECURITYADMIN** (aka Security Administrator)
  * Role that can manage any object grant globally, as well as create, monitor, and manage users and roles. More specifically, this role:
    * Is granted the MANAGE GRANTS security privilege to be able to modify any grant, including revoking it.
    * Inherits the privileges of the USERADMIN role via the system role hierarchy (i.e. USERADMIN role is granted to SECURITYADMIN).

* **USERADMIN** (aka User and Role Administrator)
  * Role that is dedicated to user and role management only. More specifically, this role:
    * Is granted the CREATE USER and CREATE ROLE security privileges.
    * Can create users and roles in the account.
    * This role can also manage users and roles that it owns. Only the role with the OWNERSHIP privilege on an object (i.e. user or role), or a higher role, can modify the object properties.

* **SYSADMIN** (aka System Administrator)
  * Role that has privileges to create warehouses and databases (and other objects) in an account.
    * If, as recommended, you create a role hierarchy that ultimately assigns all custom roles to the SYSADMIN role, this role also has the ability to grant privileges on warehouses, databases, and other objects to other roles.

* **PUBLIC**
  * Pseudo-role that is automatically granted to every user and every role in your account.
  * The PUBLIC role can own securable objects, just like any other role; however, the objects owned by the role are, by definition, available to every other user and role in your account.
  * This role is typically used in cases where explicit access control is not needed and all users are viewed as equal with regard to their access rights.


**C. Privilege**
  * A defined level of access to an object.
  * Multiple distinct privileges may be used to control the granularity of access granted.
  * [All privileges can be found here](https://docs.snowflake.com/en/user-guide/security-access-control-privileges)

**D.User**
   * A user identity recognized by Snowflake, whether associated with a person or program.

* ***In addition***
  * each securable object has an owner that can grant access to other roles.
    * This model is different from a user-based access control model in which rights and privileges are assigned to each user or groups of users.
    * The Snowflake model is designed to provide a significant amount of both control and flexibility.

![GrantingRoles](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/GrantingRoles.png)

</details>

<details>
  <summary>Data Loading</summary>

## Data Loading

### Supported File Locations

**External Stages**
* Amazon S3
* Google Cloud Storage
* Microsoft Azure

#### Listing files in stage

The LIST command Returns a list of files that have been staged (i.e. uploaded from a local file system or unloaded from a table) in one of the following Snowflake stages:

* Named internal stage.
* Named external stage.
* Stage for a specified table.
* Stage for the current user.

LIST can be abbreviated to LS.

Syntax for the LIST command:
```
LIST { internalStage | externalStage } [ PATTERN = '<regex_pattern>' ]
```

**Usage notes**
* In contrast to named stages, table and user stages are not first-class database objects; rather, they are implicit stages associated with the table/user. As such, they have no grantable privileges of their own:

  * You can always list files in your user stage (i.e. no privileges are required).

  * To list files in a table stage, you must use a role that has the OWNERSHIP privilege on the table.


**OUTPUT**
|Column |Data Type |Description|
|--| -- | -- |
|name | VARCHAR|Name of the staged file.|
|size |NUMBER |	Size of the file compressed (in bytes).|
|md5 |VARCHAR |	The MD5 column stores an MD5 hash of the contents of the staged data file, which can be used to verify the file was staged (uploaded) correctly. Note that Amazon S3 stages report the value via the S3 eTag field, which may not be an MD5 hash of the file contents.|
|last_modified | last_modified|Timestamp when the file was last updated in the stage.|


#### Uploading files external stages
Note

* PUT does not support uploading files to external stages. To upload files to external stages, use the utilities provided by the cloud service.

* The ODBC driver supports PUT with Snowflake accounts hosted on the following platforms:

  * Amazon Web Services 

  * Google Cloud Platform 

  *  Microsoft Azure 

#### Rewmoving files in stage
REMOVE command Removes files from either an external (external cloud storage) or internal (i.e. Snowflake) stage.

For internal stages, the following stage types are supported:

* Named internal stage

* Stage for a specified table

* Stage for the current user

REMOVE can be abbreviated to RM.
Syntax for the REMOVE command:
```
REMOVE { internalStage | externalStage } [ PATTERN = '<regex_pattern>' ]
```

**Internal Stages**
* **User**
  * A user stage is allocated to each user for storing files.
  * This stage type is designed to store files that are staged and managed by a single user but can be loaded into multiple tables.
  * User stages cannot be altered or dropped.
* **Table**
  * A table stage is available for each table created in Snowflake. 
  * This stage type is designed to store files that are staged and managed by one or more users but only loaded into a single table. 
  * Table stages cannot be altered or dropped.
  * *Note that a table stage is not a separate database object; rather, it is an implicit stage tied to the table itself.*
    *  A table stage has no grantable privileges of its own. 
    * To stage files to a table stage, list the files, query them on the stage, or drop them, you must be the table owner (have the role with the OWNERSHIP privilege on the table)
* **Named**
  * A named internal stage is a database object created in a schema. 
  * This stage type can store files that are staged and managed by one or more users and loaded into one or more tables.
  * Because named stages are database objects, the ability to create, modify, use, or drop them can be controlled using security access control privileges. 
#### Uploading files internal stages
* Upload files to any of the internal stage types from your local file system using the PUT command.

Syntax for the PUT command:
```
PUT file://<path_to_file>/<filename> internalStage
    [ PARALLEL = <integer> ]
    [ AUTO_COMPRESS = TRUE | FALSE ]
    [ SOURCE_COMPRESSION = AUTO_DETECT | GZIP | BZ2 | BROTLI | ZSTD | DEFLATE | RAW_DEFLATE | NONE ]
    [ OVERWRITE = TRUE | FALSE ]
```
**Required Parameters - that are nice to know for the exam**

**``` internalStage ```**

| | |
|--| -- |
|```@[namespace.]int_stage_name[/path] ```| Files are uploaded to the specified **named** internal stage. |
|```  @[namespace.]%table_name[/path] ``` | Files are uploaded to the stage for the specified **table**. |
|``` @~[/path]  ``` | Files are uploaded to the stage for the current **user.** |

**Note**

*If the stage name or path includes spaces or special characters, it must be enclosed in single quotes (e.g. '@"my stage"' for a stage named "my stage").*

**Optional Parameters - that are nice to know for the exam**

**``` PARALLEL = integer ```**
* Specifies the number of threads to use for uploading files.
* The upload process separate batches of data files by size:
  * **Small files** (< 64 MB compressed or uncompressed) are staged in parallel as individual files.

  * **Larger files** are automatically split into chunks, staged concurrently, and reassembled in the target stage. A single thread can upload multiple chunks.

Supported values: 1-99

Default: 4

**``` AUTO_COMPRESS = TRUE | FALSE ```**
* TRUE: Files are compressed (if they are not already compressed).
* FALSE: Files are not compressed (i.e. files are uploaded as-is).

Default: TRUE

**``` SOURCE_COMPRESSION = AUTO_DETECT | GZIP | BZ2 | BROTLI | ZSTD | DEFLATE | RAW_DEFLATE | NONE ```**

* Default: AUTO_DETECT

**``` OVERWRITE = TRUE | FALSE ```**
* TRUE: An existing file with the same name is overwritten.
* FALSE: An existing file with the same name is not overwritten.

Default: FALSE

**Usage Notes**
* The command cannot be executed from Snowsight; instead, use the SnowSQL client or Drivers to upload data files
* File-globbing patterns (i.e. wildcards) are supported.
* The command does not create or rename files.
* **All files stored on internal stages for data loading and unloading operations are automatically encrypted** using AES-256 strong encryption on the server side. By default, Snowflake provides additional client-side encryption with a 128-bit key (with the option to configure a 256-bit key). For more information, see encryption types for internal stages.
* **The command ignores any duplicate files you attempt to upload to the same stage.** A duplicate file is an unmodified file with the same name as an already-staged file.
  * To overwrite an already-staged file, you must modify the file you are uploading so that its contents are different from the staged file, which results in a new checksum for the newly-staged file.

#### Downloading files internal stages
The GET command Downloads data files from one of the following Snowflake stages to a local directory/folder on a client machine:

* Named internal stage.
* Internal stage for a specified table.
* Internal stage for the current user.

**Note**

* GET does not support downloading files from external stages. To download files from external stages, use the utilities provided by the cloud service.

* The ODBC driver supports GET with Snowflake accounts hosted on the following platforms:

  * Amazon Web Services
  * Google Cloud Platform 
  * Microsoft Azure 


Syntax for the GET command:
```
GET internalStage file://<local_directory_path>
    [ PARALLEL = <integer> ]
    [ PATTERN = '<regex_pattern>'' ]
```

**Required Parameters - that are nice to know for the exam**
**``` internalStage ```**

| | |
|--| -- |
|```@[namespace.]int_stage_name[/path] ```| Files are uploaded to the specified **named** internal stage. |
|```  @[namespace.]%table_name[/path] ``` | Files are uploaded to the stage for the specified **table**. |
|``` @~[/path]  ``` | Files are uploaded to the stage for the current **user.** |

**Note**

*If the stage name or path includes spaces or special characters, it must be enclosed in single quotes (e.g. '@"my stage"' for a stage named "my stage").*


**Optional Parameters - that are nice to know for the exam**

**``` PARALLEL = integer ```**
* Specifies the number of threads to use for downloading the files. The granularity unit for downloading is one file.
* 

Supported values: 1-99

Default: 10

**``` PATTERN = 'regex_pattern' ```**
* Specifies a regular expression pattern for filtering files to download. 
* The command lists all files in the specified path and applies the regular expression pattern on each of the files found.

Default: No value (all files in the specified stage are downloaded)

**Usage Notes**
* The command cannot be executed from Snowsight; instead, use the SnowSQL client or Drivers to upload data files
* The command does not rename files.
* Downloaded files are automatically decrypted using the same key that was used to encrypt the file when it was either uploaded (using PUT) or unloaded from a table (using COPY INTO <location>).

### File formats

**Format type options**

```STRIP_OUTER_ARRAY = TRUE | FALSE```
Use: Data loading and external tables
Definition: Boolean that instructs the JSON parser to remove outer brackets (i.e. [ ]).
Default:FALSE

```STRIP_NULL_VALUES = TRUE | FALSE```
Use:Data loading and external tables

Definition: Boolean that instructs the JSON parser to remove object fields or array elements containing null values.
Default:FALSE


</details>

<details>
  <summary>Editions</summary>

## Editions

![Editions](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/Editions.png)

</details>

<details>
  <summary>Micro-partitions & Data Clustering</summary>

## Micro-partitions & Data Clustering
### What are Micro-partitions?
* All data in Snowflake tables is automatically divided into micro-partitions, which are **contiguous** units of storage.
* Each micro-partition contains between 50 MB and 500 MB of uncompressed data (note that the actual size in Snowflake is smaller because data is always stored compressed).
* Groups of rows in tables are mapped into individual micro-partitions, organized in a columnar fashion.
* This size and structure allows for extremely granular pruning of very large tables, which can be comprised of millions, or even hundreds of millions, of micro-partitions.

Snowflake stores metadata about all rows stored in a micro-partition, including:
* The range of values for each of the columns in the micro-partition.
* The number of distinct values.
* Additional properties used for both optimization and efficient query processing.

**Note:**
*Micro-partitioning is automatically performed on all Snowflake tables. Tables are transparently partitioned using the ordering of the data as it is inserted/loaded.*

### Benefits of Micro-partitioning [Ref](https://docs.snowflake.com/en/user-guide/tables-clustering-micropartitions#benefits-of-micro-partitioning)
The benefits of Snowflake’s approach to partitioning table data include:
* Snowflake micro-partitions are derived automatically; they don’t need to be explicitly defined up-front or maintained by users.
* Micro-partitions are small in size (50 to 500 MB, before compression), which enables extremely efficient DML and fine-grained pruning for faster queries.
* Micro-partitions can overlap in their range of values, which, combined with their uniformly small size, helps prevent skew. 
* Columns are stored independently within micro-partitions, often referred to as columnar storage. This enables efficient scanning of individual columns; only the columns referenced by a query are scanned.
* Columns are also compressed individually within micro-partitions. Snowflake automatically determines the most efficient compression algorithm for the columns in each micro-partition.


### What is Data Clustering? [Ref](https://docs.snowflake.com/en/user-guide/tables-clustering-micropartitions#what-is-data-clustering)

* In Snowflake, as data is inserted/loaded into a table, clustering metadata is collected and recorded for each micro-partition created during the process.
* Snowflake then leverages this clustering information to avoid unnecessary scanning of micro-partitions during querying, significantly accelerating the performance of queries that reference these columns.

The following diagram illustrates a Snowflake table, t1, with four columns sorted by date:

![MicroPartitions](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/MicroPartitions.png)


The table consists of 24 rows stored across 4 micro-partitions, with the rows divided equally between each micro-partition. Within each micro-partition, the data is sorted and stored by column, which enables Snowflake to perform the following actions for queries on the table:

1. First, prune micro-partitions that are not needed for the query.
1. Then, prune by column within the remaining micro-partitions.

### Clustering Information Maintained for Micro-partitions

Snowflake maintains clustering metadata for the micro-partitions in a table, including:

* The total number of micro-partitions that comprise the table.

* The number of micro-partitions containing values that overlap with each other (in a specified subset of table columns).

* The depth of the overlapping micro-partitions.

#### Clustering Depth

* The clustering depth for a populated table measures the average depth (1 or greater) of the overlapping micro-partitions for specified columns in a table. 
  * The smaller the average depth, the better clustered the table is with regards to the specified columns.
* Clustering depth can be used for a variety of purposes, including:
  * Monitoring the clustering “health” of a large table, particularly over time as DML is performed on the table.
  * Determining whether a large table would benefit from explicitly defining a clustering key.

**Clustering Depth Illustrated**
The following diagram provides a conceptual example of a table consisting of five micro-partitions with values ranging from A to Z, and illustrates how overlap affects clustering depth:

![ClusteringDepth](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/ClusteringDepth.png)

As this diagram illustrates:

1. At the beginning, the range of values in all the micro-partitions overlap.

1. As the number of overlapping micro-partitions decreases, the overlap depth decreases.

1. When there is no overlap in the range of values across all micro-partitions, the micro-partitions are considered to be in a constant state (i.e. they cannot be improved by clustering).

### What is a Clustering Key?
*A clustering key is a subset of columns in a table (or expressions on a table) that are explicitly designated to co-locate the data in the table in the same micro-partitions.*
</details>

<details>
  <summary>Multi-cluster Warehouses</summary>

## Multi-cluster Warehouses
[Ref](https://docs.snowflake.com/en/user-guide/warehouses-multicluster)
*When deciding whether to use multi-cluster warehouses and the number of clusters to use per multi-cluster warehouse, consider the following:*

* If you are using **Snowflake Enterprise Edition** (or a higher edition), all your warehouses should be configured as multi-cluster warehouses.

* Unless you have a specific requirement for running in Maximized mode, multi-cluster warehouses should be configured to run in Auto-scale mode, which enables Snowflake to automatically start and stop clusters as needed.

When choosing the ***minimum*** and ***maximum*** number of clusters for a multi-cluster warehouse:
| | |
|--| -- |
| Minimum | Keep the default value of 1; this ensures that additional clusters are only started as needed. However, if high-availability of the warehouse is a concern, set the value higher than 1. This helps ensure multi-cluster warehouse availability and continuity in the unlikely event that a cluster fails.  |
| Maximum       |  Set this value as large as possible, while being mindful of the warehouse size and corresponding credit costs. For example, an X-Large multi-cluster warehouse with maximum clusters = 10 will consume 160 credits in an hour if all 10 clusters run continuously for the hour.  |


### Maximized vs. Auto-scale
You can choose to run a multi-cluster warehouse in either of the following modes:
* **Maximized**
  * This mode is enabled by specifying the same value for both maximum and minimum number of clusters (note that the specified value must be larger than 1). 
  * When the warehouse is started, Snowflake starts all the clusters so that maximum resources are available while the warehouse is running.

* Auto-scale 
  * This mode is enabled by specifying different values for maximum and minimum number of clusters. In this mode, Snowflake starts and stops clusters as needed to dynamically manage the load on the warehouse:
   * As the number of concurrent user sessions and/or queries for the warehouse increases, and queries start to queue due to insufficient resources, Snowflake automatically starts additional clusters, up to the maximum number defined for the warehouse.
   * Similarly, as the load on the warehouse decreases, Snowflake automatically shuts down clusters to reduce the number of running clusters and, correspondingly, the number of credits used by the warehouse.

#### SCALING_POLICY
To help control the usage of credits in Auto-scale mode, Snowflake provides a property, SCALING_POLICY, that determines the scaling policy to use when automatically starting or shutting down additional clusters.

| Policy | Description | Warehouse Starts… | Warehouse Shuts Down… |
|--| -- |-- |-- |
| Standard (default) | Prevents/minimizes queuing by favoring starting additional clusters over conserving credits. |The first cluster starts immediately when either a query is queued or the system detects that there’s one more query than the currently-running clusters can execute. <br> <br> Each successive cluster waits to start 20 seconds after the prior one has started.  | After 2 to 3 consecutive successful checks (performed at 1 minute intervals), which determine whether the load on the least-loaded cluster could be redistributed to the other clusters without spinning up the cluster again. |
|Economy | 	Conserves credits by favoring keeping running clusters fully-loaded rather than starting additional clusters, which may result in queries being queued and taking longer to complete. | Only if the system estimates there’s enough query load to keep the cluster busy for at least 6 minutes. | After 5 to 6 consecutive successful checks (performed at 1 minute intervals), which determine whether the load on the least-loaded cluster could be redistributed to the other clusters without spinning up the cluster again. |

</details>

<details>
  <summary>Organizations</summary>

## Organizations

* Organizations simplify account management and billing, Replication and Failover/Failback, Snowflake Secure Data Sharing, and other account administration tasks.

* This feature allows organization administrators to view, create, and manage all of your accounts across different regions and cloud platforms.

### Benefits

* A central view of all accounts within your organization. For more information, refer to Viewing Accounts in Your Organization.
* Self-service account creation.
* Data availability and durability by leveraging data replication and failover. 
* Seamless data sharing with Snowflake consumers across regions.
* Ability to monitor and understand usage across all accounts in the organization.


### ORGADMIN Role
A user with the ORGADMIN role can perform the following actions:
1. Create an account in the organization.
1. View/show all accounts within the organization.
1. View/show a list of regions enabled for the organization. 
1. View usage information for all accounts in the organization.
1. Enable database replication for an account in the organization.


**Note**

*Once an account is created, ORGADMIN can view the account properties but does not have access to the account data.*

</details>

<details>
  <summary>Role Hierarchy and Privilege Inheritance</summary>

## Role Hierarchy and Privilege Inheritance

The following diagram illustrates the hierarchy for the system-defined roles, as well as the recommended structure for additional, user-defined account roles and database roles. The highest-level database role in the example hierarchy is granted to a custom (i.e. user-defined) account role. In turn, this role is granted to another custom role in a recommended structure that allows the system-defined SYSADMIN role to inherit the privileges of custom account roles and database roles:

![RoleHierarchy2](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/RoleHierarchy2.png)

For a more specific example of role hierarchy and privilege inheritance, consider the following scenario:

* Role 3 has been granted to Role 2.

* Role 2 has been granted to Role 1.

* Role 1 has been granted to User 1.

![PrivilegeInheritance](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/PrivilegeInheritance.png)

</details>

<details>
  <summary>Sharing </summary>

## Sharing 

You can share the following Snowflake database objects:
* External tables.
* Dynamic tables.
* Secure views.
* Secure materialized views.
* Secure UDFs.
* Tables.

|Data Sharing Mechanism | Share With Whom? | Auto-fulfill Across Clouds? | Optionally Charge for Data? |Optionally Offer Data Publicly? |  Get Consumer Usage Metrics? |
|---|---|---|---|---|---|
|Listing|	One or more accounts in any region|Yes|Yes|Yes|Yes|
|Direct share|	One or more accounts in any region|No|No|No|No|

* If you want to manage a group of accounts, and control who can publish and consume listings in that group, consider using a Data Exchange.

</details>

<details>
  <summary>Snowflake.account_ussage vs. Snowflake.information_schema </summary>

## Snowflake.account_ussage vs. Snowflake.information_schema
* The INFORMATION_SCHEMA views and table functions display data in real-time, whereas the ACCOUNT_USAGE views have some built-in latency, due to the process of extracting the usage data from Snowflake's internal metadata store.

|Difference   | Account Usage  |  	Information Schema |   
|---|---|---|
| Includes dropped objects  | yes  | No  |   
| Latency of data  | From 45 minutes to 3 hours (varies by view)  | None  |  
|  Retention of historical data | 1 Year  | From 7 days to 6 months (varies by view/table function)  |   
[Reference](https://docs.snowflake.com/en/sql-reference/account-usage#differences-between-account-usage-and-information-schema)

* [Enabling the SNOWFLAKE Database Usage for Other Roles](https://docs.snowflake.com/en/sql-reference/account-usage#enabling-the-snowflake-database-usage-for-other-roles)
  * By default, the SNOWFLAKE database is available only to the **ACCOUNTADMIN role**.

    To enable other roles to access the database and schemas, and query the views, a user with the ACCOUNTADMIN role must grant the following data sharing privilege to the desired roles:

        IMPORTED PRIVILEGES

    ```
    USE ROLE ACCOUNTADMIN;

    GRANT IMPORTED PRIVILEGES ON DATABASE snowflake TO ROLE SYSADMIN;
    GRANT IMPORTED PRIVILEGES ON DATABASE snowflake TO ROLE customrole1;

    USE ROLE customrole1;

    SELECT database_name, database_owner FROM snowflake.account_usage.databases;
    ```

| Role              | Purpose and Description                                                         |
|-------------------|---------------------------------------------------------------------------------|
| OBJECT_VIEWER     | The OBJECT_VIEWER role provides visibility into object metadata.                |
| USAGE_VIEWER      | The USAGE_VIEWER role provides visibility into historical usage information.    |
| GOVERNANCE_VIEWER | The GOVERNANCE_VIEWER role provides visibility into policy related information. |
| SECURITY_VIEWER   | The SECURITY_VIEWER role provides visibility into security based information.   |  

</details>

<details>
  <summary>Snowpipe</summary>

## Snowpipe
[Ref](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro)
* Snowpipe enables loading data from files as soon as they’re available in a stage.
* The data is loaded according to the COPY statement defined in a referenced pipe.
* All data types are supported, including semi-structured data types such as JSON and Avro.

### How Is Snowpipe Different from Bulk Data Loading?

**Authentication**  
| What?          | Description |
|----------------|-------------| 
| Bulk data load |    Relies on the security options supported by the client for authenticating and initiating a user session.         |
| Snowpipe       |   When calling the REST endpoints: Requires key pair authentication with JSON Web Token (JWT). JWTs are signed using a public/private key pair with RSA encryption.          |

**Load History**  
| What?          | Description |
|----------------|-------------| 
| Bulk data load |    Stored in the metadata of the target table for 64 days. Available upon completion of the COPY statement as the statement output.         |
| Snowpipe       |  Stored in the metadata of the pipe for 14 days. Must be requested from Snowflake via a REST endpoint, SQL table function, or ACCOUNT_USAGE view.   |

**Transactions**  
| What?          | Description |
|----------------|-------------| 
| Bulk data load |    Loads are always performed in a single transaction. Data is inserted into table alongside any other SQL statements submitted manually by users.         |
| Snowpipe       |  Stored in the metadata of the pipe for 14 days. Must be requested from Snowflake via a REST endpoint, SQL table function, or ACCOUNT_USAGE view.   |

**Compute Resources**  
| What?          | Description |
|----------------|-------------| 
| Bulk data load |   Requires a user-specified warehouse to execute COPY statements.       |
| Snowpipe       |  Uses Snowflake-supplied compute resources. |

**Cost**  
| What?          | Description |
|----------------|-------------| 
| Bulk data load |   Billed for the amount of time each virtual warehouse is active. |
| Snowpipe       | Billed according to the compute resources used in the Snowpipe warehouse while loading the files.  |


### General File Sizing Recommendations
The number of load operations that run in parallel cannot exceed the number of data files to be loaded. To optimize the number of parallel operations for a load, we recommend aiming to produce data files roughly 100-250 MB (or larger) in size compressed.

</details>

<details>
  <summary>Preparing to Load Data</summary>

## Preparing to Load Data
[Ref](https://docs.snowflake.com/en/user-guide/data-load-prepare)

### Data File Compression
* Snowflake recommend that you compress your data files when you are loading large data sets. 
* When loading compressed data, Snowflake will automatically determine the file and codec compression method for your data files. 

### Supported File Formats
The following file formats are supported:

| Structured/Semi-structured | Type | Notes |
|----------------------------|------| ------| 
| Structured |  Delimited (CSV, TSV, etc.)  |Any valid singlebyte delimiter is supported; default is comma (i.e. CSV). |
| Semi-structured |   JSON | |
|  |   Avro | 	Includes automatic detection and processing of compressed Avro files. |
|  |  ORC  | Includes automatic detection and processing of compressed ORC files.|
|  |  Parquet  | Includes automatic detection and processing of compressed Parquet files. Currently, Snowflake supports the schema of Parquet files produced using the Parquet writer v1. Files produced using v2 of the writer are not supported. |
|  |   XML | Supported as a preview feature. |

### Semi-structured File Formats
* Snowflake natively supports semi-structured data, which means semi-structured data can be loaded into relational tables without requiring the definition of a schema in advance.
* Snowflake supports loading semi-structured data directly into columns of type VARIANT

NB: Snowflake has these three Semistructured datatypes:
* VARIANT
* OBJECT
* ARRAY

</details>

<details>
  <summary>Query Profile</summary>

## Query Profile
  [ref](https://docs.snowflake.com/en/user-guide/ui-query-profile)



### Common Query Problems Identified by Query Profile
[ref](https://docs.snowflake.com/en/user-guide/ui-query-profile#common-query-problems-identified-by-query-profile)

#### “Exploding” Joins

One of the common mistakes SQL users make is joining tables without providing a join condition (resulting in a “Cartesian product”), or providing a condition where records from one table match multiple records from another table.

his can be observed by looking at the number of records produced by a Join operator, and typically is also reflected in Join operator consuming a lot of time.

![ExplodingJoins](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/ExplodingJoins.png)

#### UNION Without ALL

In SQL, it is possible to combine two sets of data with either UNION or UNION ALL constructs. The difference between them is that UNION ALL simply concatenates inputs, while UNION does the same, but also performs duplicate elimination.

A common mistake is to use UNION when the UNION ALL semantics are sufficient. These queries show in Query Profile as a UnionAll operator with an extra Aggregate operator on top (which performs duplicate elimination).

#### Queries Too Large to Fit in Memory

For some operations (e.g. duplicate elimination for a huge data set), the amount of memory available for the compute resources used to execute the operation might not be sufficient to hold intermediate results. <br/>
As a result, the query processing engine will start spilling the data to local disk. If the local disk space is not sufficient, the spilled data is then saved to remote disks.

This spilling can have a profound effect on query performance (especially if remote disk is used for spilling). To alleviate this, we recommend:

* Using a larger warehouse (effectively increasing the available memory/local disk space for the operation), and/or

* Processing data in smaller batches.

#### Inefficient Pruning

Snowflake collects rich statistics on data allowing it not to read unnecessary parts of a table based on the query filters. However, for this to have an effect, the data storage order needs to be correlated with the query filter attributes.

The efficiency of pruning can be observed by comparing Partitions scanned and Partitions total statistics in the TableScan operators. If the former is a small fraction of the latter, pruning is efficient. If not, the pruning did not have an effect.

Of course, pruning can only help for queries that actually filter out a significant amount of data. If the pruning statistics do not show data reduction, but there is a Filter operator above TableScan which filters out a number of records, this might signal that a different data organization might be beneficial for this query.

<br/>
<br/>
______________________________________________________________________________________________
<br/>
<br/>

|  |  | 
|----------------------------|------|
| **Steps** | If the query was processed in multiple steps, you can toggle between each step. | 
| **Operator tree** | The middle pane displays a graphical representation of all the operator nodes for the selected step, including the relationships between each operator node. | 
| **Node list** | The middle pane includes a collapsible list of operator nodes by execution time. | 
| **Overview** | The right pane displays an overview of the query profile. The display changes to operator details when an operator node is selected. | 


### Profile Overview / Operator Details

**Execution Time**

Execution time provides information about “where the time was spent” during the processing of a query. Time spent can be broken down into the following categories, displayed in the following order:

* Processing — time spent on data processing by the CPU.

* Local Disk IO — time when the processing was blocked by local disk access.

* Remote Disk IO — time when the processing was blocked by remote disk access.

* Network Communication — time when the processing was waiting for the network data transfer.

* Synchronization — various synchronization activities between participating processes.

* Initialization — time spent setting up the query processing.

**Statistics**
A major source of information provided in the detail pane is the various statistics, grouped in the following sections:

* IO — information about the input-output operations performed during the query:
  * Scan progress — the percentage of data scanned for a given table so far.

  * Bytes scanned — the number of bytes scanned so far.

  * Percentage scanned from cache — the percentage of data scanned from the local disk cache.

  * Bytes written — bytes written (e.g. when loading into a table).

  * Bytes written to result — bytes written to the result object. For example, select * from . . . would produce a set of results in tabular format representing each field in the selection. In general, the results object represents whatever is produced as a result of the query, and Bytes written to result represents the size of the returned result.

  * Bytes read from result — bytes read from the result object.

  * External bytes scanned — bytes read from an external object, e.g. a stage.

* DML — statistics for Data Manipulation Language (DML) queries:
  * Number of rows inserted — number of rows inserted into a table (or tables).

  * Number of rows updated — number of rows updated in a table.

  * Number of rows deleted — number of rows deleted from a table.

  * Number of rows unloaded — number of rows unloaded during data export.  

* Pruning — information on the effects of table pruning:
  * Partitions scanned — number of partitions scanned so far.

  * Partitions total — total number of partitions in a given table.
* Spilling — information about disk usage for operations where intermediate results do not fit in memory:
  * Bytes spilled to local storage — volume of data spilled to local disk.

  * Bytes spilled to remote storage — volume of data spilled to remote disk.

* Network — network communication:

  * Bytes sent over the network — amount of data sent over the network.

* External Functions — information about calls to external functions


**Attributes**

</details>
<details>
  <summary>SAMPLE / TABLESAMPLE</summary>

## SAMPLE / TABLESAMPLE
[Ref](https://docs.snowflake.com/en/sql-reference/constructs/sample)

Syntax
```
SELECT ...
FROM ...
  { SAMPLE | TABLESAMPLE } [ samplingMethod ] ( { <probability> | <num> ROWS } ) [ { REPEATABLE | SEED } ( <seed> ) ]
[ ... ]
```
**Specifies the sampling method to use:**
* ```BERNOULLI | ROW``` 
  * Includes each row with a ```<probability>``` of p/100. Similar to flipping a weighted coin for each row. 
*  ```SYSTEM | BLOCK```
  *  Includes each block of rows with a ```<probability>``` of p/100. Similar to flipping a weighted coin for each block of rows. This method does not support fixed-size sampling.

**Specifies whether to sample based on a fraction of the table or a fixed number of rows in the table, where:**

* ```<probability>``` or ```<num> ROWS```
  * ```<probability>``` specifies the percentage probability to use for selecting the sample.
    * Can be any decimal number between 0 (no rows selected) and 100 (all rows selected) inclusive.
  * ```<num>``` specifies the number of rows (up to 1,000,000) to sample from the table. 
    * Can be any integer between 0 (no rows selected) and 1000000 inclusive.


**Deterministic sampling**
* ```REPEATABLE | SEED ( <seed> )```
  * Specifies a seed value to make the sampling deterministic. 
    * Can be any integer between 0 and 2147483647 inclusive.

</details>

<details>
  <summary>Streams</summary>

[ref](https://docs.snowflake.com/en/user-guide/streams-intro)
## Streams
* When created, a stream logically takes an initial snapshot of every row in the source object (e.g. table, external table, or the underlying tables for a view) by initializing a point in time (called an offset) as the current transactional version of the object. 
* Note that a stream itself does not contain any table data.
  * A stream only stores an offset for the source object and returns CDC records by leveraging the versioning history for the source object. 
  * When the first stream for a table is created, several hidden columns are added to the source table and begin storing change tracking metadata.
    * These columns consume a small amount of storage. 

### Stream Columns
A stream stores an offset for the source object and not any actual table columns or data. When queried, a stream accesses and returns the historic data in the same shape as the source object (i.e. the same column names and ordering) with the following additional columns:

|           |        |
|-----------------------|-------------|
| METADATA$ACTION | Indicates the DML operation (INSERT, DELETE) recorded. |
| METADATA$ISUPDATE    |   Indicates whether the operation was part of an UPDATE statement. <br> Updates to rows in the source object are represented as a pair of DELETE and INSERT records in the stream with a metadata column METADATA$ISUPDATE values set to TRUE.  <br> <br> Note that streams record the differences between two offsets. If a row is added and then updated in the current offset, the delta change is a new row. The METADATA$ISUPDATE row records a FALSE value.        |
|  METADATA$ROW_ID      |  Specifies the unique and immutable ID for the row, which can be used to track changes to specific rows over time.       |


### Types of Streams
* Standard
  * Supported for streams on tables, directory tables, or views.
  * A standard (i.e. delta) stream tracks all DML changes to the source object, including inserts, updates, and deletes (including table truncates).
* Append-only
  * Supported for streams on standard tables, directory tables, or views.
  * An append-only stream tracks row inserts only. Update and delete operations (including table truncates) are not recorded.
* Insert-only
  * Supported for streams on external tables only.
  * An insert-only stream tracks row inserts only; they do not record delete operations that remove rows from an inserted set (i.e. no-ops).

</details>

<details>
  <summary>SQL syntax</summary>

## SQL syntax

### File Functions
* GET_PRESIGNED_URL and BUILD_SCOPED_FILE_URL are non-deterministic functions; the others are deterministic.

</details>

<details>
  <summary>Tasks</summary>

## Tasks
[ref](https://docs.snowflake.com/en/user-guide/tasks-intro)

A task can execute any one of the following types of SQL code:

* Single SQL statement

* Call to a stored procedure

* Procedural logic using Snowflake Scripting


### DAG of Tasks

* Each task (except the root task) can have multiple predecessor tasks (dependencies); likewise, each task can have multiple subsequent (child) tasks that depend on it.
  * A task runs only after all of its predecessor tasks have run successfully to completion.
* You can specify the predecessor tasks when creating a new task (using CREATE TASK … AFTER) or later (using ALTER TASK … ADD AFTER).
  * A DAG is limited to a ***maximum of 1000 tasks total (including the root task).*** 
  * A single task can have a ***maximum of 100 predecessor tasks and 100 child tasks.***

### Link Severed Between Predecessor and Child Tasks
[ref](https://docs.snowflake.com/en/user-guide/tasks-intro#link-severed-between-predecessor-and-child-tasks)
Dependencies among tasks in a DAG can be severed as a result of any of the following actions:

* Remove predecessors for a child task using ALTER TASK … REMOVE AFTER.

* Drop predecessors for a child task using DROP TASK.

* Transfer ownership of a child task to a different role using GRANT OWNERSHIP.

If any combination of the above actions severs the relationship between the child task and all predecessors, **then the former child task becomes either a standalone task or a root task**, depending on whether other tasks identify the task as their predecessor.


</details>

<details>
  <summary>Time stored</summary>

## Time stored

| What?                 | time        |
|-----------------------|-------------|
| Snowpipe load history | 14 days     |
|                       |             |
|                       |             |
|                       |             |

</details>

<details>
  <summary>Time travel</summary>

![TimeTravel](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/TimeTravel.png)
### Data Retention Period

A key component of Snowflake Time Travel is the data retention period.

The standard retention period is 1 day (24 hours) and is automatically enabled for all Snowflake accounts:
* For Snowflake Standard Edition, the retention period can be set to 0 (or unset back to the default of 1 day) at the account and object level (i.e. databases, schemas, and tables).
* For Snowflake Enterprise Edition (and higher):

  * For transient databases, schemas, and tables, the retention period can be set to 0 (or unset back to the default of 1 day). The same is also true for temporary tables.

  * For permanent databases, schemas, and tables, the retention period can be set to any value from 0 up to 90 days.

###  Snowflake Fail-safe
When the retention period ends for an object, the historical data is moved into Snowflake Fail-safe

![FailSafe](https://github.com/Niklasingwersen/SnowPro-Core-notes/blob/main/Images/FailSafe.png)

Fail-safe provides a (non-configurable) 7-day period during which historical data may be recoverable by Snowflake. 

</details>
