# explore-kylo
explore-kylo

## Q&A

### What is the difference between Kylo and HortonWorks Data Flow


#### Why kylo over Nifi

* Better userbility
* More suitable for Business Analyst to use
* error handling
* cleanup, reuse template

#### Where is all the meta data Nifi stored

* FlowRepository: /opt/nifi/data/flowfile_repository
* ContentRepositorys: /opt/nifi/data/content_repository
* Provenance Reporitoy: /opt/nifi/data/provenance_repository
* Canvas data: /opt/nifi/data/conf/flow.xml.gz
* Databases? /opt/nifi/nifi-1.0.0/database_repository
* MessageQueue? /opt/activemq/current
* Log: /var/log/nifi

#### Why import template and register template are two steps?

#### Terminology

Job: One running feed

- Running: Started? 
- Completed: 
- Abandoned: 
- FAILED
    
Feed: One flow

Linearage/Provanence: 

SLA, Profile

#### behavior on multiple Standarize rule

#### Merge strategy performance

#### Kylo Tables

    mysql -p kylo  
    // password: hadoop

    spring.datasource.url=jdbc:mysql://localhost:3306/kylo
    spring.datasource.username=root
    spring.datasource.password=hadoop

table name is case sensitive

| Tables_in_kylo                 |
|:-------------------------------|
| AUDIT_LOG                      |
| BATCH_EXECUTION_CONTEXT_VALUES |
| BATCH_FEED_SUMMARY_COUNTS_VW   |
| BATCH_JOB_EXECUTION            |
| BATCH_JOB_EXECUTION_CTX_VALS   |
| BATCH_JOB_EXECUTION_PARAMS     |
| BATCH_JOB_EXECUTION_SEQ        |
| BATCH_JOB_INSTANCE             |
| BATCH_JOB_SEQ                  |
| BATCH_NIFI_JOB                 |
| BATCH_NIFI_STEP                |
| BATCH_STEP_EXECUTION           |
| BATCH_STEP_EXECUTION_CTX_VALS  |
| BATCH_STEP_EXECUTION_SEQ       |
| CHECK_DATA_TO_FEED_VW          |
| DATABASECHANGELOG              |
| DATABASECHANGELOGLOCK          |
| FEED                           |
| FEED_ACL_INDEX                 |
| FEED_CHECK_DATA_FEEDS          |
| FEED_HEALTH_VW                 |
| GENERATED_KEYS                 |
| KYLO_ALERT                     |
| KYLO_ALERT_CHANGE              |
| KYLO_VERSION                   |
| LATEST_FEED_JOB_END_TIME_VW    |
| LATEST_FEED_JOB_VW             |
| LATEST_FINISHED_FEED_JOB_VW    |
| MODESHAPE_REPOSITORY           |
| NIFI_EVENT                     |
| NIFI_FEED_PROCESSOR_STATS      |
| NIFI_RELATED_ROOT_FLOW_FILES   |
| SLA_ASSESSMENT                 |
| SLA_METRIC_ASSESSMENT          |
| SLA_OBLIGATION_ASSESSMENT      |
+--------------------------------+   

#### Hive Tables

    mysql -p hive  
    // password: hadoop

    hive.metastore.datasource.url=jdbc:mysql://localhost:3306/hive
    
    // cloudera using metastore
    // hive.metastore.datasource.url=jdbc:mysql://localhost:3306/metastore
    
    hive.metastore.datasource.username=root
    hive.metastore.datasource.password=hadoop



table name is case sensitive

| Tables_in_Hive                 |
|:-------------------------------|
| AUX_TABLE                 |
| BUCKETING_COLS            |
| CDS                       |
| COLUMNS_V2                |
| COMPACTION_QUEUE          |
| COMPLETED_COMPACTIONS     |
| COMPLETED_TXN_COMPONENTS  |
| DATABASE_PARAMS           |
| DBS                       |
| DB_PRIVS                  |
| DELEGATION_TOKENS         |
| FUNCS                     |
| FUNC_RU                   |
| GLOBAL_PRIVS              |
| HIVE_LOCKS                |
| IDXS                      |
| INDEX_PARAMS              |
| KEY_CONSTRAINTS           |
| MASTER_KEYS               |
| NEXT_COMPACTION_QUEUE_ID  |
| NEXT_LOCK_ID              |
| NEXT_TXN_ID               |
| NOTIFICATION_LOG          |
| NOTIFICATION_SEQUENCE     |
| NUCLEUS_TABLES            |
| PARTITIONS                |
| PARTITION_EVENTS          |
| PARTITION_KEYS            |
| PARTITION_KEY_VALS        |
| PARTITION_PARAMS          |
| PART_COL_PRIVS            |
| PART_COL_STATS            |
| PART_PRIVS                |
| ROLES                     |
| ROLE_MAP                  |
| SDS                       |
| SD_PARAMS                 |
| SEQUENCE_TABLE            |
| SERDES                    |
| SERDE_PARAMS              |
| SKEWED_COL_NAMES          |
| SKEWED_COL_VALUE_LOC_MAP  |
| SKEWED_STRING_LIST        |
| SKEWED_STRING_LIST_VALUES |
| SKEWED_VALUES             |
| SORT_COLS                 |
| TABLE_PARAMS              |
| TAB_COL_STATS             |
| TBLS                      |
| TBL_COL_PRIVS             |
| TBL_PRIVS                 |
| TXNS                      |
| TXN_COMPONENTS            |
| TYPES                     |
| TYPE_FIELDS               |
| VERSION                   |
| WRITE_SET                 |


#### Security setup

tls-toolkit, /opt/kylo/setup/nifi/install-kylo-components.sh

http://apache.mirrors.spacedump.net/nifi/1.2.0/nifi-toolkit-1.2.0-bin.tar.gz

#### JDK version incompatible

export JAVA_HOME=/etc/alternatives/jre_1.8.0/

#### Where is extra meta data Kylo stored

#### How Nifi cluster members communicate with each other

Nifi using ZeeKeeper to communicate. The Primary node is also elected by ZeeKeeper

#### What is the difference between Data Linearage and Data Provenance?

#### Data Locality for HDFS?

#### Which web server nifi using?
Jetty. TOBE confirmed

#### How to delete in Nifi!!!

#### Why can not change scheduler after connected

#### why have Exception when start or stop nifi?
    /opt/nifi/current/bin/nifi.sh stop
    
    Exception in thread "main" java.lang.UnsupportedClassVersionError: org/apache/nifi/bootstrap/RunNiFi : Unsupported major.minor version 52.0
    	at java.lang.ClassLoader.defineClass1(Native Method)
    	at java.lang.ClassLoader.defineClass(ClassLoader.java:800)
    	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
    	at java.net.URLClassLoader.defineClass(URLClassLoader.java:449)
    	at java.net.URLClassLoader.access$100(URLClassLoader.java:71)
    	at java.net.URLClassLoader$1.run(URLClassLoader.java:361)
    	at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
    	at java.security.AccessController.doPrivileged(Native Method)
    	at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
    	at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
    	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
    	at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
    	at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:482)
    	
#### HTTPS connections setup in Nifi

https://community.hortonworks.com/questions/87154/how-to-setup-standardsslcontextservice.html

there might be some configuration in /opt/nifi/current/conf 

#### Kylo web got redirected to login page after installation

check security.jwt.key attribute in /opt/kylo/kylo-services/conf/application.properties and /opt/kylo/kylo-ui/conf/application.properties

they should have the same value, it is kind of encryption key between kylo ui and kylo backend services, if it is different, then ui can’t take with backend

this key is automatically generated during installation and should be the same, 


#### How to resume a failed processor in Nifi

There step, it might fail for some reason. 
While we don't want to start from begin again. 
Just want resume the failed one from the point.

#### Where is really the directory configured in Kylo in cluster setup?

having it in all nodes?

#### How replay works in Nifi

#### How to fix the wrong record in file import and to make it correct manually

#### Does Kylo supports incremental transformation?

When two 

#### Kylo airflow comparison

#### Sqoop does not included in kylo_docker

#### How can we handle further for the invalid data

The records which do not conform with the rule we set in the Ingest feed will goes to invalid record

#### How to fix wrong feed configuration?

Assume that after configured a feed and test some data import, found that there are some configuration in the feed was not correct. And I was fix it. 

While there are some records already imported

