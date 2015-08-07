---
title: "Hive Storage Plugin"
parent: "Storage Plugin Configuration"
---
Drill 1.0 supports Hive 0.13. Drill 1.1 supports Hive 1.0. To access Hive tables
using custom SerDes or InputFormat/OutputFormat, all nodes running Drillbits
must have the SerDes or InputFormat/OutputFormat `JAR` files in the 
`<drill_installation_directory>/jars/3rdparty` folder.

## Hive Remote Metastore Configuration

The Hive metastore configuration runs as a separate service outside
of Hive. Drill communicates with the Hive metastore through Thrift. The
metastore service communicates with the Hive database over JDBC. Point Drill
to the Hive metastore service address, and provide the connection parameters
in a Hive storage plugin configuration to configure a connection to Drill.

{% include startnote.html %}Verify that the Hive metastore service is running before you register the Hive metastore.{% include endnote.html %}  

To register a remote Hive metastore with Drill:

1. Issue the following command to start the Hive metastore service on the system specified in the `hive.metastore.uris`:
   `hive --service metastore`
2. In the [Drill Web UI]({{ site.baseurl }}/docs/plugin-configuration-basics/#using-the-drill-web-ui), select the **Storage** tab.
3. In the list of disabled storage plugins in the Drill Web UI, click **Update** next to the `hive` instance. For example:

        {
          "type": "hive",
          "enabled": false,
          "configProps": {
            "hive.metastore.uris": "",
            "javax.jdo.option.ConnectionURL": "jdbc:derby:;databaseName=../sample-data/drill_hive_db;create=true",
            "hive.metastore.warehouse.dir": "/tmp/drill_hive_wh",
            "fs.default.name": "file:///",
            "hive.metastore.sasl.enabled": "false"
          }
        }
4. In the configuration window, add the `Thrift URI` and port to `hive.metastore.uris`. For example:

          ...
             "configProps": {
             "hive.metastore.uris": "thrift://<host>:<port>",
          ...
5. Change the default location of files to suit your environment; for example, change `"fs.default.name"` property from `"file:///"` to one of these locations:
   * `hdfs://`
   * `hdfs://<hostname>:<port>`
6. If you are running Drill and Hive in a secure MapR cluster, remove the following line from the configuration:  
   `"hive.metastore.sasl.enabled" : "false"`
7. Click **Enable**.  
8. If you are running Drill and Hive in a secure MapR cluster, add the following line to `<DRILL_HOME>/conf/drill-env.sh` on each Drill node and then [restart the Drillbit service]({{site.baseurl}}/docs/starting-drill-in-distributed-mode/):  
   `export DRILL_JAVA_OPTS="$DRILL_JAVA_OPTS -Dmapr_sec_enabled=true -Dhadoop.login=maprsasl -Dzookeeper.saslprovider=com.mapr.security.maprsasl.MaprSaslProvider -Dmapr.library.flatclass"`

After configuring a Hive storage plugin, you can [query Hive tables]({{ site.baseurl }}/docs/querying-hive/).

## Hive Embedded Metastore Configuration

The Hive metastore configuration is embedded within the Drill process. Configure an embedded metastore only in a cluster that runs a single Drillbit and only for testing purposes. Do not embed the Hive metastore in production systems.

Provide the metastore database configuration settings in the Drill Web UI. Before you configure an embedded Hive metastore, verify that the driver you use to connect to the Hive metastore is in the Drill classpath located in `/<drill installation directory>/lib/.` If the driver is not there, copy the driver to `/<drill
installation directory>/lib` on the Drill node. For more information about storage types and configurations, refer to ["Hive Metastore Administration"](https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin).

To configure an embedded Hive metastore, complete the following
steps:

1. In the [Drill Web UI]({{ site.baseurl }}/docs/plugin-configuration-basics/#using-the-drill-web-ui), and select the **Storage** tab.
2. In the disabled storage plugins section, click **Update** next to `hive` instance.
3. In the configuration window, add the database configuration settings.

    **Example**

          {
            "type": "hive",
            "enabled": false,
            "configProps": {
              "hive.metastore.uris": "",
              "javax.jdo.option.ConnectionURL": "jdbc:<database>://<host:port>/<metastore database>",
              "hive.metastore.warehouse.dir": "/tmp/drill_hive_wh",
              "fs.default.name": "file:///",
              "hive.metastore.sasl.enabled": "false"
            }
          }
5. Change the `"fs.default.name"` attribute to specify the default location of files. The value needs to be a URI that is available and capable of handling file system requests. For example, change the local file system URI `"file:///"` to the HDFS URI: `hdfs://`, or to the path on HDFS with a namenode: `hdfs://<authority>:<port>`
6. Click **Enable**.
  