+++
title = "Accessing mutiple Hadoop amespaces with native HDFS client"
date = "2020-02-05T22:26:04-05:00"
description = ""
tags = [
    "hadoop",
]
categories = [
    "bigdata",
]
+++

In hadoop we can have multiple namespaces in `hdfs-site.xml` for HDFS clients. This is achieved by a HDFS feature called: [HDFS Federation](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/Federation.html )



### Why do you need multiple namespaces

This feature is particularly useful when you need to distcp data between multiple highly available remote hadoop clusters. For HA clusters we don't always know the address of active namenode and it's not recommanded to hardcode the active namenode address in the distcp command in case of namenode fail over. 



### How to add additional namespaces

Cloudera has a detailed [document](https://docs.cloudera.com/documentation/enterprise/5-14-x/topics/cdh_admin_distcp_data_cluster_migrate.html#distcp_ha) on how to setup distcp with HA remote clusters. In essence, you need to edit `dfs.nameservices` property and add the following properties to `hdfs-site.xml`. (I have prepared a [script](https://gist.github.com/ryanym/eb3618efd2572d0b97c962b6be57d6bf) to generate below content, it assumes all services are in their default ports)

Edit:

Note the order of the nameservices matters, the first nameservice will be the default hdfs namespace that you can access without the `hdfs://` URI

```xml
<property>
 <name>dfs.nameservices</name>
 <value>nameservice1,pdvnameservice</value>
</property>
```

Add:

```xml
  <property>
    <name>dfs.ha.namenodes.pdvnameservice</name>
    <value>namenode1,namenode2</value>
  </property>
  <property>
    <name>dfs.client.failover.proxy.provider.pdvnameservice</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
  <property>
    <name>dfs.ha.automatic-failover.enabled.pdvnameservice</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:8020</value>
  </property>
  <property>
    <name>dfs.namenode.servicerpc-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:8022</value>
  </property>
  <property>
    <name>dfs.namenode.servicerpc-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:8022</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:50070</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:50070</value>
  </property>
  <property>
    <name>dfs.namenode.https-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:50470</value>
  </property>
  <property>
    <name>dfs.namenode.https-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:50470</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:8020</value>
  </property>
  <property>
    <name>dfs.namenode.servicerpc-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:8022</value>
  </property>
  <property>
    <name>dfs.namenode.servicerpc-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:8022</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:50070</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:50070</value>
  </property>
  <property>
    <name>dfs.namenode.https-address.pdvnameservice.namenode1</name>
    <value>master1.example.com:50470</value>
  </property>
  <property>
    <name>dfs.namenode.https-address.pdvnameservice.namenode2</name>
    <value>master2.example.com:50470</value>
  </property>
```

### Test the added namespace

```bash
hdfs --config ~/hadoop-conf-editted dfs -ls hdfs://addednameservice/                   
Found 13 items
drwx------+  - hbase        hbase                 0 2020-02-04 15:25 hdfs://addednameservice/hbase
drwxrwxr-x+  - hdfs         supergroup            0 2019-05-02 14:29 hdfs://addednameservice/share
drwxrwxrwt+  - hdfs         supergroup            0 2019-08-22 20:42 hdfs://addednameservice/tmp
drwxrwxr-x+  - hdfs         supergroup            0 2020-02-04 16:32 hdfs://addednameservice/user

```



### Pushing changes to all clients from Cloudera Manager

`HDFS` -> `Configuration` -> `Scope: Gateway` ->`HDFS Client Advanced Configuration Snippet (Safety Valve) for hdfs-site.xml`

Choose `View as XML` and copy the above properties. Note this will trigger a cluster restart and client configuration deploy.

Once changes are pushed to clients, we can now access the added namespace without the customised `hdfs-site.xml`

```shell
hdfs dfs -ls hdfs://addednameservice/                   
Found 13 items
drwx------+  - hbase        hbase                 0 2020-02-04 15:25 hdfs://addednameservice/hbase
drwxrwxr-x+  - hdfs         supergroup            0 2019-05-02 14:29 hdfs://addednameservice/share
drwxrwxrwt+  - hdfs         supergroup            0 2019-08-22 20:42 hdfs://addednameservice/tmp
drwxrwxr-x+  - hdfs         supergroup            0 2020-02-04 16:32 hdfs://addednameservice/user
```

### Going to HDFS Federation

If you are using Cloudera distribution of hadoop this can be done in Cloudera Manager using [Add Nameservice](https://docs.cloudera.com/documentation/enterprise/5-14-x/topics/cdh_admin_distcp_data_cluster_migrate.html#distcp_ha)

Note that HDFS Federation is **not** supported in CDH 6 or CDP 