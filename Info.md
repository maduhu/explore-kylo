## Ports

|port|function|
|:---|:-------|
|2222|SSH|
|3306|MySQL
|8079|Nifi|
|8088|Yarn|
|8400|Kylo|
|8420|
|9200|Elastic Search
|9300|Elastic Search
|10000|Hive
|61616|Elastic JMS

## Directories

|directory|description|
|:--------|:----------|
|/opt/nifi/current
|/opt/kylo/kylo-services/
|/opt/kylo/kylo-ui/
|
|/usr/local/hadoop/         | /usr/share/hdp/current | Hadoop Home
|/usr/local/hadoop/logs 
|
|/usr/local/hive/
|
|/opt/activemq/apache-activemq-5.13.3/
|/opt/activemq/apache-activemq-5.13.3//data
|
|/usr/share/elasticsearch | 
|/var/log/elasticsearch     | Elastic log
|/var/lib/elasticsearch     | Elastic Data
|/etc/elasticsearch         | Elastic Configuration
|
|/archive/${category}/${feed}/${archiveId} | hdfs dfs -ls /archive
|/etl | hdfs ingest root
|/model.db | hive ingest/profile root
|/app/warehouse | hive 
| /etl/${category}/${feed}/${feedts} | feedts=${now():toNumber()}, it will not be deleted while feed deleted, feedts is also a partition in ${feed}_feed table


### Initialize Script

	def flowFile = session.get()
    if(!flowFile) return
    def json = flowFile.getAttribute("metadata.table.fieldPoliciesJson");
    def inputFolder = flowFile.getAttribute("spark.input_folder")
    def feed = flowFile.getAttribute("feed")
    def category = flowFile.getAttribute("category")
    def feedts = flowFile.getAttribute("feedts")
    def folder = new File(inputFolder + "/"+category+"/"+feed+"/"+feedts)
    // If it doesn't exist
    if( !folder.exists() ) {
	    // Create all folders
	    folder.mkdirs()
    }
    def jsonFile = new File(folder,feed+"_field_policy.json")
    jsonFile.write(json)
    flowFile = session.putAttribute(flowFile,"table_field_policy_json_file",jsonFile.getCanonicalPath())
    session.transfer(flowFile, REL_SUCCESS)


    /tmp/kylo-nifi/spark/website/ingest/1496780055599/ingest_field_policy.json

    this file will be deleted after job executed


### Spark Validate And Split Records

Implemented at `com.thinkbiganalytics.spark.datavalidator.Validator`

- Load Hive table
- Go through records to check if it's valid, it will persist
- Write valid records to valid table
- Write invalid records to invalid table

The persist level can be passed through command line

`--storageLevel` [storage_level](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.storage.StorageLevel$)


### Merge Table

Implemented at `com.thinkbiganalytics.nifi.v2.ingest.MergeTable`

From hive table `${feed}_valid` to hive table `${feed}`

Merge strategy

- Dedupe
- Sync

### Water mark

Implemented at `com.thinkbiganalytics.nifi.v2.core.watermark.ReleaseHighWaterMark`


### Transform script

    // Get flow file
    def flowFile = session.get()
    if (!flowFile) return
    // Create output directory
    def inputFolder = flowFile.getAttribute("spark.input_folder")
    def category = flowFile.getAttribute("category")
    def feed = flowFile.getAttribute("feed")
    def feedts = flowFile.getAttribute("feedts")
    def folder = new File(inputFolder + "/" + category + "/" + feed + "/" + feedts)
    if (!folder.exists()) folder.mkdirs()
    
    // Write script
    def script = "sqlContext.setConf(\"hive.exec.dynamic.partition\", \"true\")\n"
    script = script + "sqlContext.setConf(\"hive.exec.dynamic.partition.mode\", \"nonstrict\")\n"
    script = script + flowFile.getAttribute("metadata.dataTransformation.dataTransformScript")
    script = script + ".withColumn(\"processing_dttm\", org.apache.spark.sql.functions.lit(\"" + feedts + "\"))"
    script = script + ".write.mode(SaveMode.Overwrite)"
    def isPreFeed = (flowFile.getAttribute("transform_script") != null)
    def sparkVersion = flowFile.getAttribute("spark.version")
    if (!isPreFeed && (sparkVersion == null || sparkVersion == "1")) {
      script = script + ".partitionBy(\"processing_dttm\")"
    }
    script = script + ".insertInto(\"" + category + "." + feed + (isPreFeed ? "" : "_feed") + "\")"
    def scalaFile = new File(folder, "transform.scala")
    scalaFile.write(script)
    // Write field policies
    def json = flowFile.getAttribute("metadata.table.fieldPoliciesJson")
    if (json != null) {
      def jsonFile = new File(folder, feed + "_field_policy.json")
      jsonFile.write(json)
      flowFile = session.putAttribute(flowFile, "table_field_policy_json_file", jsonFile.getCanonicalPath())
    }
    // Output file path
    flowFile = session.putAttribute(flowFile, "transform_script_file", scalaFile.getCanonicalPath())
    session.transfer(flowFile, REL_SUCCESS)
    
Script Sample

    sqlContext.setConf("hive.exec.dynamic.partition", "true")
    sqlContext.setConf("hive.exec.dynamic.partition.mode", "nonstrict")
    
    import org.apache.spark.sql._
    import org.apache.spark.sql.functions._
    
    var df = sqlContext.sql(
        """SELECT e.`eventname`, v.`venuename`, v.`venuecity`, v.`venuestate`, v.`venueseats` 
            FROM `concerts`.`events` e 
            INNER JOIN `concerts`.`venues` v ON v.`venueid` = e.`venueid`"
        ).select($"eventname", $"venuename", upper($"venuecity").as("venuecity"), $"venuestate", $"venueseats"
        ).select($"*", concat($"venuestate", lit(","), $"venuecity").as("city_state")
        ).groupBy($"city_state").count()
        
    df.withColumn("processing_dttm", lit("1496871337338")).
        write.mode(SaveMode.Overwrite).
        partitionBy("processing_dttm").
        insertInto("concerts.events_stats_feed")

#### Schema Index

The table is in MySQL database

    SELECT d.NAME DATABASE_NAME, d.OWNER_NAME OWNER, t.CREATE_TIME, t.TBL_NAME, t.TBL_TYPE,
        c.COLUMN_NAME, c.TYPE_NAME
        FROM hive.COLUMNS_V2 c
        JOIN hive.SDS s on s.CD_ID = c.CD_ID
        JOIN  hive.TBLS t ON s.SD_ID=t.SD_ID
        JOIN  hive.DBS d on d.DB_ID = t.DB_ID
        where d.name = '${category}'and t.tbl_name = '${feed}';
    
    