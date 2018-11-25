#### What is Cassandra ?
- NoSQL distributed database technology.  
- A big data technology with massive scalability. - Based on AWS Dynamodb and Google Big Table.  
- Commonly used to create database that is spread across nodes in more than one data center for high availability, fault tolerant.  
- Highly performant, proven, fault tolerant, no single point of failure, durable, elastic and supported for real time applications, IOT sensor data, messaging etc.  
- Netflix, EBay is a popular user.  
- cassandra.apache.org  


#### System requirements  
- 64-bit Macintosh, Windows, Linux  
- 8 GB Ram  
- 30 GB of free hard drive space  

#### ARCHITECTURE  
- You can have multiple nodes like 4 in one AZ or Region and replica or it in another region.  
- Datastax provides enterprice version of cassandra with support and documentation.  
- www.datastax.com  

#### SNITCH  
- Snitch is how the nodes in a cluster know about the topology in a masterless topology.  
- Snitching options.  
```
Dynamic snitching
Simple Snitch
RackInferringSnitch
PropertyFileSnitch
GossipingPropertyFileSnitch
EC2Snitch
EC2MultiRegionSnitch
```
- For each snitch there is a configuration file like.
`PropertyFileSnitch`
```
130.77.100.147 = DC1 : RAC1
130.77.100.148 = DC1 : RAC1
130.77.100.165 = DC1 : RAC1
130.77.100.109 = DC1 : RAC2
130.77.100.110 = DC1 : RAC2
130.77.100.111 = DC1 : RAC2

130.77.100.128 = DC2 : RAC1
130.77.100.129 = DC2 : RAC1
130.77.100.107 = DC2 : RAC2
130.77.100.108 = DC2 : RAC2
```
- This is a `PropertyFileSnitch`, for this configuration, you need to put this file to each node to know the topology nodes, dcs and racs. If you have a big cluster, it's hard to put it to all machines, so you can use `GossipPropertyFileSnitch`.  

#### GOSSIP
- Gossip is how the nodes in a cluster communicate with each other.  
- Every one second, one node exchanges information with another 3 nodes to exchange information about itself and other nodes that known about.  It's internal communication method.  
- For external communication `CQL` or `Thrift` are used.  

#### DATA DISTRIBUTION
- It's done through consistent hashing to strive even distribution of data across the nodes in a cluster.  
- A table is spread across the nodes in the cluster.  
- Example in a table id is hashed and a partitioner used to decide which rows are kept in which node depending the values which nodes are responsible for. 
- Default partitioner is `Murmur3`.  
- `Murmur3` takes the id and creates a hash between -2^63 and 2^63 and each node has an endpoint value assigned to it.  

```
H01033638 - > -7456322128261119046
H01545551 - > -2487391024765843411
H00999943 - > 6394005945182357732
```
- Calculate the token ranges. You need to replace number 4 to number of nodes in your cluster and find, which raw in which node.  
```
python -c 'print [str(((2**64 / 4 ) - 2**63) for i in range(4)]' ['-9234566.....' , '- 461757...', '0', '461666....' ]
```
- Every node has a range and to which range the hash is falling, raw is kept there. Each cassandra cluster is responsible for keeping values between his endpoint token and previous nodes token. This is the range.  

#### REPLICATION FACTOR  
- Specifies, how many replica of the database will be. 1 means no. It's common 2 or 3.  So, if a node goes down, it can leave. Number of nodes that can fail.  

#### VIRTUAL NODES
- An alternative way to assign token ranges to nodes and are now the default in Cassandra.  
- Virtual nodes provides many small token ranges like 128, 256, 512 slices instead of having one large token range. Higher values for high performance servers, lower values for low performance servers and the cluster will be balanced.  
- When a new node is added, it receives many small token range slices from the existing nodes.   
- In the old way, it was common to double the number of nodes, could be a value half of the value of the existing end-points.  

#### DOWNLOADING CASSANDRA  
Download Options;  
Community verstion from datastax  
- https://planetcassandra.org is a good starting point.  
- https://cassandra.apache.org  >  https://academy.datastax.com/planet-cassandra/
- https://datastax.com - Enterprise with some additional tools like HADOOP, SOLAR.  
- Create an Ubuntu instance.  
```
wget http://apache.spd.co.il/cassandra/2.2.13/apache-cassandra-2.2.13-bin.tar.gz
```

#### INSTALLATION
- Install Oracle JRE or JDK, not open JDK/JRE. 
- tar -xvzf unpackage both.   

#### MAIN CONFIGURATION FILES
- cassandra.yaml  
- See the settings  
  - cluster_name
  - number_of_tokens
  - data_file_directories  
  - endpoint_snitch 

```
# So our user can write to those directories  
sudo mkdir /var/log/cassandra
sudo mkdir /var/lib/cassandra
chown -R ${USER}:${GROUP} /var/lib/cassandra
chown -R ${USER}:${GROUP} /var/log/cassandra
``` 

#### STARTING CASSANDRA
```
# Add to .bashrc or profile 
export PATH="/home/ubuntu/cassandra/jre1.8.0_191/bin/:${PATH}"
bin/cassandra
bin/cassandra -f # Foreground
# Node Tool
bin/nodetool status
```
Output:
```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  127.0.0.1  97.16 KB   256          100.0%            33603a9c-3ae0-4a63-b93c-f1a45e06b1a4  rack1
```
- UN means U:Up N: State Normal
- You will see IP Address, Load is data size, virtual tokens,Owns effective how muc of the data of the cluster it holds, host id and rac, also upside you see the datacenter.  
```
ps aux |grep "cassandra"
bin/nodetool info
```
Output:  
```
ID                     : 33603a9c-3ae0-4a63-b93c-f1a45e06b1a4
Gossip active          : true
Thrift active          : false
Native Transport active: true
Load                   : 97.16 KB
Generation No          : 1542285728
Uptime (seconds)       : 1415
Heap Memory (MB)       : 72.68 / 1974.00
Off Heap Memory (MB)   : 0.00
Data Center            : datacenter1
Rack                   : rack1
Exceptions             : 0
Key Cache              : entries 10, size 816 bytes, capacity 98 MB, 23 hits, 36 requests, 0.639 recent hit rate, 14400 save period in seconds
Row Cache              : entries 0, size 0 bytes, capacity 0 bytes, 0 hits, 0 requests, NaN recent hit rate, 0 save period in seconds
Counter Cache          : entries 0, size 0 bytes, capacity 49 MB, 0 hits, 0 requests, NaN recent hit rate, 7200 save period in seconds
Token                  : (invoke with -T/--tokens to see all 256 tokens)
```
```
# This will print out all the tokens.  
bin/nodetool ring
```

#### LOGS  
```
cat /var/logs/system.log
```
- Log settings
```
cat conf/log4j-server.properties
# log4j.appender.R.File=/var/log/cassandra/system.log
```

#### COMMUNICATE WITH CASSANDRA
- CQL: SQL like query language.  
- Thrift: is a low-level API, still supported bu may be phased out. It doesn't have the exact sql commands, like `join`.   
- For administrative activities, such as monitoring, management tasks, tools built on JMX(Java Management Extensions) are commonly used.  
- CQL basic data types text, varchar and int.  
- Other data types: https://docs.datastax.com/en/cql/3.3/cql/cql_reference/cql_data_types_c.html  

- CQL example:  
```
SELECT home_id, datetime, event, code_used FROM activity;
```
- https://docs.datastax.com/en/dse/6.0/cql/cql/cql_reference/cqlsh_commands/cqlsh.html  
- https://docs.huihoo.com/apache/cassandra/datastax/CQL-3.1-for-Cassandra-2.0-and-2.1.pdf  
- cqlsh requires python.  
```
cqlsh
```
```
DESCRIBE CLUSTER
HELP
HELP CREATE_KEYSPACE
```

#### CREATING A DATABASE
- `keyspace` is a database in cassandra where you can define tables.  
```
DESCRIBE KEYSPACES
# system system_traces
DESCRIBE KEYSPACE system
```
- Create a Database
```
CREATE KEYSPACE vehicle_tracker WITH REPLICATION = { 'class' : 'NetworkTologyStrategy', 'dc1' : 3, 'dc2' : 2};
# We don't have a replicated cluster, so we will use.
CREATE KEYSPACE vehicle_tracker WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 1};
DESCRIBE KEYSPACE vechicle_tracker
- Description of a keyspace is the exact CREATE COMMAND. 
```
- Delete a KEYSPACE
```
DROP KEYSPACE vehicle_tracker
```

#### CREATING A TABLE
- First create a database to use
```
CREATE KEYSPACE home_security WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 1};
```
- Create Table  
```
USE home_security;
CREATE TABLE activity ( home_id text, datetime timestamp, event text, code_used text, PRIMARY KEY (home_id, datetime)) WITH CLUSTERING ORDER BY (datetime DESC);
DESCRIBE TABLE activity;
```
- Delete Table  
```
DROP TABLE activity;
```

#### DEFINING A PRIMARY KEY
```
CREATE TABLE home ( home_id text, address text, city text, state text, zip text, owner text, phone text, alt_phone text, email text, phone_password text, main_code text, PRIMARY KEY(home_id) );
#OR  
CREATE TABLE home ( home_id text PRIMARY KEY, address text, city text, state text, zip text, owner text, phone text, alt_phone text, email text, phone_password text, main_code text, guest_code text );
```
#### RECOGNIZING A PARTITION KEY
- Partition key is hashed value of `home_id` in `home` table. `PartitionKey` and `RowKey` are same.  
- If a primary key has more than 2 columns, it's called `compound primary key`.  
- In a `compund primary key`, first column contains the partition key.  
- There is the Thrift raw and CQL raw. Need to ask the developer what are they talking about.  

#### Speciffying A Descending Order
- If you don't specify, it will be ascending. That means rows will be added at the end.  Descending means row will be added in the beginning of the table. Descending takes a little longer to write but improves read performance.  
```
CREATE TABLE activity ( home_id text, datetime timestamp, event text, code_used text, PRIMARY KEY (home_id, datetime)) WITH CLUSTERING ORDER BY (datetime DESC);
```
- You cannot alter and change order of a table. It's not an option.  
- Adding a column
```
ALTER TABLE activity ADD status text;  
```

#### EXCERSIZE
- Create a table called `home` under `home_security` keystore with columns, `home_id`, `address`, `city`, `state`, `zip`, `contact_name`, `phone`, `alt_phone`, `phone_password`, `email`, `main_code` and `guest_code` all with a data type `text` and `home_id` will be primary key.  
```
USE home_security;
CREATE TABLE home (home_id text PRIMARY KEY,address text, state text, zip text, contact_name text, phone text, alt_phone text, phone_password text, email text, main_code text, guest_code text);
```

#### WAYS TO WRITE DATA
- INSERT INFO  command  
- COPY command  (CSV format)  
- sstableloader tool bulk data input using sstable files, sstable file storage unit is how cassandra stores data.  

#### INSERT INTO command AND SELECT FROM
```
USE home_security;
INSERT INTO activity (home_id, datetime, code_used, event) VALUES ('H01474777', '2014-05-21 07:32:16', 'alarm set', '5599');
SELECT * FROM activity;
SELECT home_id, datetime, event from activity;
```

#### COPY command
- For importing or exporting data from CSV.  
- Create a file activity.csv .  
File: activity.csv  
```
home_id|datetime|event|code_used
H02257222|2014-05-21 05:29:47|alarm set|1566
H01474777|2014-05-21 07:32:16|alarm set|5599
H01033638|2014-05-21 07:50:22|alarm set|2121
H01033638|2014-05-21 05:50:43|alarm turned off|2121
H01033638|2014-05-21 07:55:58|alarm set|2121
```
- We have header, so WITH header = true and | is our delimiter.  
```
USE home_security;
COPY activity (home_id, datetime, event, code_used) FROM '/home/ubuntu/events.csv' WITH header = true AND delimiter = '|';
SELECT * FROM activity;
```
#### HOW DATA IS STORED IN CASSANDRA

- `cassandra-cli` is a tool to see how data is stored in Cassandra at a low level.  
```
bin/cassandra-cli
USE home_security;
LIST activity;
```
- Output
```
[default@home_security] LIST activity;
Using default limit of 100
Using default cell limit of 100
-------------------
RowKey: H01474777
=> (name=2014-05-21 07\:32\:16+0000:, value=, timestamp=1542563481381000)
=> (name=2014-05-21 07\:32\:16+0000:code_used, value=35353939, timestamp=1542563481381000)
=> (name=2014-05-21 07\:32\:16+0000:event, value=616c61726d20736574, timestamp=1542563481381000)
```
- One row is stored like this to be able to distribute to nodes.  


#### HOW DATA IS STORED ON DISK
- By default `/var/lib/cassandra/data` is the data location and can be changed at `config/cassandra.yaml`.  
- Each write goes to `memcache` and `commit log` in case of a failure to replay. When memcache is full it's flushed to disk as `sstable`. For each table on each node, there is a memcache.  
- Flush to disk manually;  
```
nodetool flush home_security
```
```
ls /var/lib/cassandra/data
```
Output:  
```
commitlog  data  saved_caches
```
- See table files.
```
ls /var/lib/cassandra/data/home_security/activity
```
Output (Files of a table):
```
home_security-activity-jb-1-CompressionInfo.db
home_security-activity-jb-1-Data.db
home_security-activity-jb-1-Filter.db
home_security-activity-jb-1-Index.db
home_security-activity-jb-1-Statistics.db
home_security-activity-jb-1-Summary.db
home_security-activity-jb-1-TOC.txt
```
- You can print out in json format how a table is stored, but not a usable information.
```
sstable2json /var/lib/cassandra/data/home_security/activity/home_security-activity-jb-1-Data.db  
```
- It will print out something like LIST activity, the table.  
```
sstable2json /var/lib/cassandra/data/home_security/activity/home_security-activity-jb-1-Data.db
```

#### DATA MODELING
- Need to understand implication of working with a distributed database, cassandra do not have join like a relational database, so instead of table joins which impacts the performance, in cassandra need to use a data modelling like all the data for a query is available in one table.  

#### USING A WHERE CLAUSE
- If you do a WHERE clause to a non-indexed column, query will not work.  
Where clause examples:  
```
select * from activity WHERE home_id='H01033638';
select * from activity WHERE home_id='H01033638' AND datetime>'2014-05-21 05:50:43+0000';
```

#### HOW TO CREATE SECONDARY INDEXCES FOR OTHER COLUMNS
- Creating secondary indexes does not increase the speed but profive refence for where clauses.  
- For each secondary index, Cassandra creates a hidden table on each node in the cluster.  
- For increasing the speed, you can create a table specifically for the query. The problem for this you are responsible to update this table as well.  
```
CREATE TABLE homes_by_state ( state text, zip text, home_id text, address text, PRIMARY KEY (state, zip, home_id) );
```
- Creating a secondary index.  
```
CREATE INDEX code_used_index ON activity (code_used);
```
- Then the where clause will work.
```
select * from activity WHERE event='alarm set';
```

#### COMPOSITE PARTITION KEY
- Composite partition key is where a partition is made up of more than one column.  
```
USE vehicle_tracker;
CREATE TABLE location ( vehicle_id text, date   text, time timestamp, latitude double,   longtitude double, PRIMARY KEY ((vehicle_id,   date), time) );
```
- vechicle_id and date together is composite partition key.  
- Number of cells that can be stored in a partition is 2.000.000.000 and a partition can grow endlessly, using composite partition keys helps to optimize this problem, as an example using only vechicle_id as partition key which the same vechicle_id will be used a lot of times in the same partition, you can use composite (vechicle_id and date) for a better partitioning.  
- Insert data, create a file called coordinates.csv  
File:  
```
vehicle_id|date|time|latitude|longitude
CA6AFL218|2014-05-19|2014-05-19 08:00:00|35.691656|-115.368103
LAKRM489|2014-05-19|2014-05-19 08:00:00|30.187493|-93.537481
WA063JXD|2014-05-19|2014-05-19 08:00:00|47.675471|-117.236187
ME100AAS|2014-05-19|2014-05-19 08:00:00|44.619094|-67.846205
FLN78197|2014-05-19|2014-05-19 08:00:00|24.702700|-81.143500
CA6AFL218|2014-05-19|2014-05-19 08:10:00|35.763006|-115.335144
LAKRM489|2014-05-19|2014-05-19 08:10:00|30.168498|-93.595159
WA063JXD|2014-05-19|2014-05-19 08:10:00|47.670928|-117.109236
ME100AAS|2014-05-19|2014-05-19 08:10:00|44.699649|-67.470432
FLN78197|2014-05-19|2014-05-19 08:10:00|24.706443|-81.127020
CA6AFL218|2014-05-19|2014-05-19 08:20:00|35.883264|-115.225281
LAKRM489|2014-05-19|2014-05-19 08:20:00|30.151875|-93.636358
WA063JXD|2014-05-19|2014-05-19 08:20:00|47.680175|-117.089323
ME100AAS|2014-05-19|2014-05-19 08:20:00|44.727950|-67.401767
FLN78197|2014-05-19|2014-05-19 08:20:00|24.691471|-81.190191
CA6AFL218|2014-05-19|2014-05-19 08:30:00|35.911527|-115.206312
LAKRM489|2014-05-19|2014-05-19 08:30:00|30.139406|-93.662064
WA063JXD|2014-05-19|2014-05-19 08:30:00|47.687109|-117.070097
ME100AAS|2014-05-19|2014-05-19 08:30:00|44.742584|-67.342716
FLN78197|2014-05-19|2014-05-19 08:30:00|24.660274|-81.273962
CA6AFL218|2014-05-19|2014-05-19 08:40:00|36.044227|-115.181124
LAKRM489|2014-05-19|2014-05-19 08:40:00|30.132280|-93.687170
WA063JXD|2014-05-19|2014-05-19 08:40:00|47.695891|-117.041258
ME100AAS|2014-05-19|2014-05-19 08:40:00|44.746485|-67.264438
FLN78197|2014-05-19|2014-05-19 08:40:00|24.647793|-81.327521
CA6AFL218|2014-05-19|2014-05-19 08:50:00|36.119593|-115.172584
LAKRM489|2014-05-19|2014-05-19 08:50:00|30.126341|-93.702276
WA063JXD|2014-05-19|2014-05-19 08:50:00|47.701436|-117.017912
ME100AAS|2014-05-19|2014-05-19 08:50:00|44.749411|-67.250705
FLN78197|2014-05-19|2014-05-19 08:50:00|24.670258|-81.356360
```
Insert data from coordinates.csv file.  
```
COPY location (vehicle_id,date,time,latitude,longtitude ) FROM 'coordinates.csv' WITH header = true AND delimiter = '|';
select * from location;
# Only 3 rows from the top
select * from location LIMIT = 3;
```
- See the partitions, called as internal rows.  
```
bin/nodetool
USE vehicle_tracer;
LIST location;
```
- You will see 5 raws, 5 rawkeys which means 5 partitions created. So for each vechicle_id and date together each day a new partition will be created.  
- In new cassandra 3.0 `cassandra-cli` will be deprecated and you will use regular cql. This may be used.  
```
sstable2json /var/lib/cassandra/data/vehicle_tracker/location/vehicle_tracker-location-jb-1-Data.db
sstablekeys /var/lib/cassandra/data/vehicle_tracker/location/vehicle_tracker-location-jb-1-Data.db
```

#### CASSANDRA DRIVERS
- http://cassandra.apache.org/doc/latest/getting_started/drivers.html  
http://www.datastax.com/download
- Drivers with * are not updated for a time.  
- Including as dependency in POM.xml for maven.  
```
<dependency>
      <groupID>com.datastax.cassandra</groupID>
      <artifactID>cassandra-driver-core</artifactId>
</dependency>
```
- You do this by ecliple add a dependency, so maven can download the dependent driver and include in compilation.  
- In order to use DataStax Java driver, the startup_native_transport : true should be enabled in `conf/cassandra.yml`  
Config: conf/cassandra.yml  
```
start_native_transport : true
```
- The second way is downloading the drivers and put inside our project the lib files.  
- You can download drivers from apache or datastacks the enterprise one.   

#### Cassandra Cluster Class
- For you java application to talk with Cassandra, you need to use `Cluster class` which allows your java application to talk with Cassandra.  
```
Cluster cluster = 
Cluster.builder().addContactPoints("127.0.0.1","127.0.0.2").build();
```
- Listing more than one contact point is recommended so if one contact point is down, another contact point can be used.  

#### Executing A Query From Our Java Application
```
Cluster cluster = 
Cluster.builder().addContactPoints("localhost").build();
Session session = cluster.connect();
String vehicleId = "CA6AFL218";
String trackDate = "2014-05-19";
String queryString = "SELECT time, latitue, longtitude FROM vehicle_tracker.location WHERE vehicle_id = '" + vehicleId + "' AND date = '" + trackDate + "';
ResultSet result = session.execute(queryString);
```

#### UPDATING DATA

```
UPDATE home_security.home SET phone = '310-883-7197' WHERE home_id = 'H01474777';
# More than 1 column
UPDATE home_security.home SET phone = '310-883-7197', contact_name = 'Mr. Drysdale' WHERE home_id = 'H01474777';
```

#### UNDERSTANDING HOW UPDATING WORKS
- There is no disk seek in Cassandra like RDBMS, update the data and save it. It's just an additional write with timestamp to memcache and flush to disk when memcache is full, also where latest timestamp is the read target for queries.   
- When you update a data like this.  
```
UPDATE home_security.home SET phone = '310-883-7197', contact_name = 'Mr. Drysdale' WHERE home_id = 'H01474777';
```
- Flash memcache and ls to data dir.  
```
nodetool flush home_security home
ls /var/lib/cassandra/data/home_security/home/
```
Output:  
```
home_security-home-jb-1-CompressionInfo.db  home_security-home-jb-1-Summary.db          home_security-home-jb-2-Index.db
home_security-home-jb-1-Data.db             home_security-home-jb-1-TOC.txt             home_security-home-jb-2-Statistics.db
home_security-home-jb-1-Filter.db           home_security-home-jb-2-CompressionInfo.db  home_security-home-jb-2-Summary.db
home_security-home-jb-1-Index.db            home_security-home-jb-2-Data.db             home_security-home-jb-2-TOC.txt
home_security-home-jb-1-Statistics.db       home_security-home-jb-2-Filter.db
```
- You will see that second set of sstable files are created for the updated data.  
```
sstable2json /var/lib/cassandra/data/home_security/home/home_security-home-jb-2-Data.db
```
- As you see in the newly created file, you will only see the update records and in the first sstable fileset the rest of the data.  
```
[{"key": "483031343734373737","columns": [["phone","310-883-7197",1542618617197000]]}
```

#### DELETING DATA
- Commands: DELETE, TRUNCATE, DROP  
- Create a file create.sql  
File: create.cql  
```
USE playground;

CREATE TABLE messages_by_user (
sender text, 
sent timestamp,
recip text,
body text,
message_id uuid,
PRIMARY KEY (sender, sent)
) WITH CLUSTERING ORDER BY (sent DESC);

COPY messages_by_user (sender, sent, recip, body, message_id) FROM 'messages.csv' WITH header = true AND delimiter = '|';
```
- Create a CSV file as data;  
File: messages.csv  
```
sender|sent|recip|body|message_id

annie|2013-07-21 15:31:23|juju|will be great to see you guys tonight!|f594a470-bffc-11e3-8a33-0800200c9a66

juju|2013-07-21 15:32:16|bobby|please pick up snacks|f81d4fae-7dec-11d0-a765-00a0c91e6bf6

juju|2013-07-21 15:32:58|bobby|and mixer!|c764e971-bffc-11e3-8a33-0800200c9a66

bobby|2013-07-21 15:34:01|juju|np, will do|dd963780-bffc-11e3-8a33-0800200c9a66

jonesy|2013-07-21 15:34:03|axel|meet up today?|c764e974-bffc-11e3-8a33-0800200c9a66

axel|2013-07-21 15:34:55|jonesy|for sure! 6:00 at our spot :)|e74fb030-bffc-11e3-8a33-0800200c9a66
```
- Connect to cqlsh and run the SOURCE query to create the tables and insert data.  
```
cqlsh
SOURCE 'create.cql';
```
- Delete a cell. body is the column name which will be deleted.  
```
USE playground;
DELETE body FROM messages_by_user WHERE sender='juju' AND sent = '2013-07-21 15:32:16';
```
- Delete a raw.  
`DELETE FROM messages_by_user WHERE sender='juju' AND sent = '2013-07-21 15:32:16';``
```
- Delete a table contend but leave the table.  
```
TRUNCATE playground.messages_by_user ;
```
- Delete the table;
```
DROP TABLE playground.messages_by_user ;
```

#### TOMBSTONES
- When a delete is written, a tombstone is create marking the data for deletion. The data is not immediately deleted is so that there is time for all of the nodes with a replica of the data (like any down nodes) to learn about the mark for deletion.  
- If a replication node is down, it replicates the delete when it comes up.  
- `gc_grace_seconds` default `864000` which is `10 days`, amount of time to wait.  
- After `gc_grace_seconds`, `compaction` needs to happen automatically. Compaction is when SSTables are combined, to improve performance of reads due to the fewer number of SSTables to reclaim disk space from deleted data.  
- Compaction can be done also manually;
```
nodetool compact
```
- `gc_grace_seconds` is seen like this;  
```
USE home_security;
DESCRIBE TABLE home;
```
- Example it's 10 days, your nodes should not down more than 10 days, accourding to this set it.  


- Delete a row, do a flush table and check tombstone;  
```
USE home;
DELETE home WHERE home_id = 'H00999943';
EXIT
nodetool flush home_security home
```
```
sstable2json /var/lib/cassandra/data/home_security/home/home_security-home-jb-3-Data.db
```
- Below is the deletion tumbstone. A new SSTable file set with number 3 is created.    
Output:  
```
[
{"key": "483030393939393433","metadata": {"deletionInfo": {"markedForDeleteAt":1542627493556000,"localDeletionTime":1542627493}},"columns": []}
]
```  
- Compaction manually;  
```
nodetool compact home_security home
```
- Compaction will merge all files to one file and create a new file increased by the last number and delete all the older files, it will also include inside the tumbstone.  
- Changing gc_grace_seconds
```
use home_security;
alter table home with GC_GRACE_SECONDS = 3600;
```
- A deletion is a combination of `tumbstones`, `gc_grace_seconds` , `compaction`

#### TTLs
- TTL is setted time in seconds for a data to be expired and deleted.  
```
INSERT INTO location (vechicle_id, date, time, latitude, longtitude) VALUES ('AZWT3RSKI', '2014-05-20', '2014-05-20 11:23:55', 34.872689, -111.757373) USING TTL 30;
```
- After 30 seconds, you will not see the data.  
```
USE vehicle_tracker;
select * from location;
EXIT
nodetool flush vehicle_tracker location
```
- Check the data file.  
```
sstable2json /var/lib/cassandra/data/vehicle_tracker/location/vehicle_tracker-location-jb-2-Data.db
```
- You will see the "d" deletion demarker tumbstore.  
```
[
{"key": "0009415a57543352534b4900000a323031342d30352d323000","columns": [["2014-05-20 11\\:23\\:55+0000:","5bf2a630",1542628912730000,"d"], ["2014-05-20 11\\:23\\:55+0000:latitude","5bf2a630",1542628912730000,"d"], ["2014-05-20 11\\:23\\:55+0000:longtitude","5bf2a630",1542628912730000,"d"]]}
]
```
- After `gc_grace_seconds` passes and compaction runs. Row will be deleted completely.  

#### CHANGING TTLs
- TTLs can ben updated like this. In this example position and TTL changes.  
```
UPDATE location USING TTL 7776000 SET latitude = 34.872689, longitude = -111.757373 WHERE vehicle_id = 'AZWT3RSKI' AND date = '2014-05-20' AND time='2014-05-20 11:23:55';
```

#### SELECTING HARDWARE FOR CASSANDRA
- Using at least minimum requirements is good.  
- As much as CPU Cores and Memory will, Cassandra's performance will increase.  
- The minimum number of cores is 4, for production 8 or more cores are common.  
- The minimum memory 8 GB, 32 GB and up is common for production usage.  

#### SELECTING STORAGE FOR CASSANDRA
- SSD III recommended.  
- Shared storage disk usage is not recommended by for cassandra and highly recommended. SSD is recommended.  
- Having more nodes in the cluster and less data on them is better than having just a few nodes with tons of data on them.   
- Recommended amount of data per node varies with type of disk being used but it's generally from 500GB to 1TB.  
- For maximum write performance. Having a separate drive for the commit log is recomended. So disk head does not need to move and continuously be used to write commit log entries.  
- Reference Architecture. https://www.datastax.com/wp-content/uploads/2014/01/WP-DataStax-Enterprise-Reference-Architecture.pdf  

#### NETWORKING
- Recommended is 1 Gbit/s and more preferably 10, 25 Gbit/s at AWS or cloud.  

#### PORTS USED
- 7000 : Cassandra intra-node communication (For gossip).  
- 9042 : Cassandra native binary protocol client(java client, CQL).  
- 9160 : Thrift client(oder port).  
- 7199 : JMX monitoring(for monitoring).  

#### DEPLOYING IN CLOUD
- AWS is very popular for cassandra installation.  
- If you are installing in AWS, i2.2xlarge, 60 GB memory and 1 TB HDD recommended.  
- Reference Architecture. https://www.datastax.com/wp-content/uploads/2014/01/WP-DataStax-Enterprise-Reference-Architecture.pdf  
- When you check in AWS site, you will see storage optimized I2 instances which in use cases explained that suitabe for NoSQL and cassandra.  
- Also in GCP, you can choose n1-standad-8, n1-highmem-16.  


#### ADDING A NODE TO CASSANDRA CLUSTER
- As each node in a Cassandra cluster has the same functionality as the other, it is fairly easy to add a new node.  

To add a new node, it needs:
- It has to have the same cluster name as the existing nodes in the cluster.  
- It needs IP address and network address to at least one of the nodes in the existing cluster.  
- IP address should be static for each node.  
```
172.31.16.10 - cass1
172.31.16.11 - cass2
```
- Set unique hostname for each node as above.  
- Also add each of your vms /etc/hosts .  
```
172.31.16.10   cass1
172.31.16.11   cass2
```
- Reboot the machines.  
###### LISTENER
- There are 2 listeners which you need to configure in conf/cassandra.yaml . Configure this for each nodes.  
```
listen_address: 172.31.16.10 - For node configuration.  
rpc_address: 172.31.16.10 - For client communication.  
```
###### SPECIFY SEED NODES
- Seed nodes are regular nodes that via IP address, provide a way for new nodes to join the cluster.  
- Don't add all the nodes as seed nodes. In common usage 2 or 3 nodes are specified as seed nodes. So, If one node is down, still it's possible to join the cluster.  
Conf: conf/cassandra.yaml  
```
seeds: "172.31.16.10,172.31.16.11"
```
- In our case, we have one instance and we will add the second, so to each instance we will set `172.31.16.10` and restart cassandra.  
Conf: conf/cassandra.yaml  
```
seeds: "172.31.16.10"
```
###### BOOTSTRAPING (ADDING A NODE TO CLUSTER) 
- Before starting cassandra on second node;  
- Ensure that is has the same `cluster_name`.  
- Enssue that it has the same `num_tokens`, so the load will be balanced.  
- We are using virtual node method and not initial tokens.  
```
172.31.16.11 - cass2
```
- `auto_bootstrap = true` is by default enabled but you will not find in the configuration file. In earlier versions of cassandra, it was an existing configuration.  
- When you start the second node, if `cluster_name`, `num_tokens`, `seeds` configured properly, start the cassandra on seconnd node and watch in the first node that joining the cluster.  
```
watch -n 2 nodetool status
```
Output:  
```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address       Load       Owns   Host ID                               Token
                Rack
# NEW NODE THAT JOINED.  
UN  172.31.16.11  94.65 KB   51.3%  18f82ae9-1d2a-45e0-9ff2-143a9143e9df  -9199022843233318307
                rack1
UN  172.31.16.10  153.72 KB  48.7%  5f5ed4b3-8602-48d2-8ec5-275c422d9e40  -9151020401596128422
                rack1
```

###### CLEANING UP A NODE
- Cleaning up a node means, deleting the data that the node is not responsible from. As you enlarge your cluster, data is spread out to the nodes. Another example after a failure if a node comes up and a data that it's not responsible for.  
```
nodetool -h 172.31.16.10 cleanup
```
###### USING CASSANDRA STRESS TOOL
- toos/bin/cassandra-stress
- Is a handy stress testing tool for cassandra.  
- As an example writing 100000 rows stress test.  
- By default a keyspace `Keyspace1` with tables `Standard1`, `Super1`, `SuperCounter1`, `Counter1`, `Counter3` are generated depending on the cassandra-stress options used.  
- Even you are stressing on node, the data will be distributes to the cluster.  
- The option can be customized.  
```
tools/bin/cassandra-stress -d 172.31.16.10 -n 100000
# HELP
tools/bin/cassandra-stress -h 
```
###### HOW APP TALKS WITH MULTIPLE NODES
- As an example Java Driver. See below, added 2 nodes inside a code.  
- You don't need to put all of the nodes. Java driver will get the topology from 1 of the 2 nodes and will be redundancy if one node fails.  
```
Cluster cluster = 
Cluster.builder().addContactPoints("172.31.16.10,172.31.16.11").build();
Session session = cluster.connect();
String vehicleId = "CA6AFL218";
String trackDate = "2014-05-19";
String queryString = "SELECT time, latitue, longtitude FROM vehicle_tracker.location WHERE vehicle_id = '" + vehicleId + "' AND date = '" + trackDate + "';
ResultSet result = session.execute(queryString);
```

#### MONITORING CASSANDRA  
- Tools for monitoring.  
```
- nodetool
- Jconsole
- OpsCenter
```
- All of this tools communicates with cassandra through `JMX` (Java Management Extensions).  
- Cassandra exposes many metrics and commands through `JMX` which can be used to monitor and manage a Cassandra cluster.  
###### nodetool
```
# Status of a cluster 
nodetool status
# Information of a node
nodetool info
nodetool -h 172.31.16.10 info
# Tokenrings list to nodes assignment 
nodetool ring
# Table stats for keyspaces
nodetool cfstats
# For read and write latency
nodetool cfhistograms
# Compaction information
nodetool compactionstats
# To see all nodetool commands
nodetool
```
###### JConsole
- JConsole comes with `JDK`  
```
bin/jconsole
# Choose Remote Process
localhost:7199
#OR
172.31.16.10:7199
```
- You will see a console like task manager.  
- You go to `MBEANS` tab scroll down `org.apache.cassandra.metrics > ColumnFamily > home_security > ReadLatency` as an example.  
###### OPSCENTER
- `OpsCenter` is a GUI web application for monitoring and managing a Cassandra cluster.  
- It has 2 versions `community` from `apache.org` and `Enterprise` from `datastax.com`.  
- You can download `Enterprise` if you open an account.  
- In `Enterprise Version` `Alerts`, `Services`, `Data Backups` features are enabled.  
- You need to install datastax agent on each node.  
- https://docs.datastax.com/en/archived/opscenter/5.0/opsc/about_c.html  
- https://docs.datastax.com/en/opscenter/6.1/  
- You can access the ops center like this.  
```
http://172.31.16.200:8888/
```

#### REPAIRING NODES  
###### UNDERSTANDING REPAIRS  
- Repairing is for updating a node's data to be current.  
- This comes into play when you are using a replication factor higher than 1.  
Example Reasons:  
- Node outdated because it was down.  
- Replication factor for a keyspace increased.  
- Replication factor for a keyspace decreased (cleanup necessary.)  
- Token range for a node have changed.  
- Repair should be run on each node at least once `within gc_grace_seconds` period of time. So data  marked as deleted on one node does not come back through another node, because the other node was unaware of the delete.  
- As an example if grace period is 10, running repair at least once a week is recommended.  
- `Example Changing Replication Factor And Repairing`  
- Repair can put heavy load on the cluster, it is best to run repair during low-usage hours.  
- If using `OpsCenter Enterprise`, there is an option to have `OpsCenter` automatically run a continuous repair service in the background with minimal impact.  
```
# CHANGE REPLICATION FACTOR
cqlsh
ALTER KEYSPACE home_security WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor'= 2}
exit
# REPAIR A KEYSPACE
nodetool -h 172.31.16.10 repair home_security
# REPAIR ALL KEYSPACES
nodetool -h 172.31.16.10 repair
```
- It will take a long time because, it will check all the `merkle trees`, it will be fixed on all nodes.  

#### CONSISTENCY
- Accuracy of data when there is more than one replica of the data.  
- A consistency level is specified per read or write request, with a default consistency level set by the client driver.  
- The default is `1`.  
`READ CONSISTENCY LEVELS`
```
ALL - Returns the data if all replicas up and responding. Read will fail if a replica does not respont.  
EACH_QUORUM - Returns the record with the most recent timestamp once a quorum of replicas in each datacenter responded.  
LOCAL_SERIAL - Same as serial but confined to data center.
LOCAL_QUORUM - Same like quorum but confined to data center. Avoids latency of inter-data center communication.  
LOCAL_ONE - Same like ONE but only if the replica is in the local data center.  
ONE - Returns a response from the closest replica, as determined by snitch. By default, a read repair runs in the background to make the other replicas consistent.  
QUORUM - Returns the record with the most recent timestamp after a quorum of replicas has responded regardless of data center.  
SERIAL - Allow reading the current and possibly uncommitted state of data without proposing a new addition or update. If a SERIAL read finds an uncommited transaction in progress, it will commit the transaction as part of the read.
TWO - Returns the most recent data from two of the closest replicas.  
THREE - Returns the most recent data from three of the closest replicas.  
```
- Consistency is set during the query.  
```
cqlsh
# Show Consistency;
CONSISTENCY;
# Set Consistency;
COSISTENCY TWO;
# Set Consistency;
COSISTENCY THREE;
# Set Consistency;
COSISTENCY QUORUM;
```
- https://docs.datastax.com/en/archived/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html  
###### WRITE CONSISTENCY
- It works the same way and writes according to the consistency levels that you choosed.  

#### HINTED HANDOFF
- Hinted handoff, a coordinator node takes the write temporarily when a node is down. By default, temporary store the write for 3 hours on behalf of down node.  
- Once the down node comes back, the coordinator node holding the write forwards the write.  
- A hinted handoff write does not count toward the ONE, QUORUM, or ALL consistency levels.  
- If the consistency level of ANY is specified with a write, the hinted handoff write satisfies the consistency level for the write, even if the write never eventually makes it to the node that owns it.  
- Consistency level of ANY should only be used if it is not critical for the write to be written.  
Settings: conf/cassandra.yaml
```
hinted_handoff_enabled : true
max_hint_windows_in_ms : default is 10800000 (3 hours)
```
- https://docs.datastax.com/en/cassandra/3.0/cassandra/operations/opsRepairNodesHintedHandoff.html  

#### READ REPAIR
- Read repair is an option to have a repair happen as part of a read request.  
- Read repair compares the replicas for requested data and updates outdated replicas.  
- By Default `read_repair_chance=0.100000` which meand `% 10`, each 10 read requests read repair will be done.  
- Read repair happens in the background unless a consistency of ALL is used, in which case the replicas are repaired before the response to the client is sent, which means that the data from the closest replica is sent and then the read repair happens.  
- It can changed by `ALTER TABLE COMMAND`  


#### REMOVING A NODE FROM CLUSTER  
- You may need to remove a node by some reason like;  
- You don't need the capacity.  
- HW maintenance.  
- HW failure.  
- It's done by the following commands.  
- Cassandra is FT and it can be done gracefully.  
```
nodetool decommision  
nodetool removenode  # If there is a failure and you don't have no choice.  
nodetool -h 192.168.159.103 -p 7199 decommission
```
- On node that you decommissioned.  
```
ps aux |grep cass
kill 2079  # PID number of cassandra.  
rm -rf commitlog data saved_caches
```
- Decommissioning assigns the token ranges to other nodes and  streams the data begin decommissioned.  

#### PUTTING A NODE BACK INTO SERVICE
- Data is not removed from decommissioned node. So, when you put the node to the service, it's best to clear the data and start fresh.  
- To completely delete the data;  
```
cd /var/lib/cassandra/
rm -rf commitlog data saved_caches
cassandra  # Just start the cassandra if it's configured, it will join as fresh.
# You can run a cleanup on other nodes also repait.  
```

#### REMOVING A DEAD NODE  
- Removing dead nodes means, reassign the token ranges and the data that the dead node was responsible for.  `nodetool removenode` is used. ```
nodetool status  # get host id of dead node.  
nodetool removenode 5edef963-10aa-4683-8476-75e7a8fdb6fd
```
- You need to set replication factor for your keystores else you loose data.  

#### REDEFINING A CLUSTER FOR MULTIPLE DATACENTERS  
- Snitch should be changed from `SimpleSnitch`.  
- DC and Rack Information should be specified.  
- Replication strategy for `keyspaces` should be changed to `NetworkTopologyStrategy` which allows number of replicas in each data center for a keyspace.  
```
- Create a new node 172.31.16.13 - cass4
- Install cassandra as before
- Set hostname
- Set /etc/hosts for all servers
- conf/cassandra.yaml set listener
- conf/cassandra.yaml set seeds
- conf/cassandra.yaml set same cluster name
```
###### CHANGING SNITCH TYPE
- Below are the snitch types, all the nodes in the cluster should be in the same snitch type.  
```
 Out of the box, Cassandra provides
  - SimpleSnitch:
    Treats Strategy order as proximity. This can improve cache
    locality when disabling read repair.  Only appropriate for
    single-datacenter deployments.
  - GossipingPropertyFileSnitch
    This should be your go-to snitch for production use.  The rack
    and datacenter for the local node are defined in
    cassandra-rackdc.properties and propagated to other nodes via
    gossip.  If cassandra-topology.properties exists, it is used as a
    fallback, allowing migration from the PropertyFileSnitch.
  - PropertyFileSnitch:
    Proximity is determined by rack and data center, which are
    explicitly configured in cassandra-topology.properties.
  - Ec2Snitch:
    Appropriate for EC2 deployments in a single Region. Loads Region
    and Availability Zone information from the EC2 API. The Region is
    treated as the datacenter, and the Availability Zone as the rack.
    Only private IPs are used, so this will not work across multiple
    Regions.
  - Ec2MultiRegionSnitch:
    Uses public IPs as broadcast_address to allow cross-region
    connectivity.  (Thus, you should set seed addresses to the public
    IP as well.) You will need to open the storage_port or
    ssl_storage_port on the public IP firewall.  (For intra-Region
    traffic, Cassandra will switch to the private IP after
    establishing a connection.)
  - RackInferringSnitch:
    Proximity is determined by rack and data center, which are
    assumed to correspond to the 3rd and 2nd octet of each node's IP
    address, respectively.  Unless this happens to match your
    deployment conventions, this is best used as an example of
    writing a custom Snitch class and is provided in that spirit.
```
- We will use `GossipingPropertyFileSnitch` which is alternative for `PropertyFileSnitch`, so will not add all the information to each node in the cluster, each node will keep it's `cassandra-rackdc.properties file` about itself. And it's very popular.     
- When snitch type is changed. Rolling restart necessary to all cluster nodes.  
Config: `config/cassandra.yaml`
```
endpoint_snitch: GossipingPropertyFileSnitc
```
- For nodes cass1, cass2
###### MODIFY CASSANDRA RACKDC.PROPERTIES
Config: cass/conf/cassandra-rackdc.properties  
```
dc=DC1
rack=RAC1
```
- For nodes cass3, cass4
Config: cass/conf/cassandra-rackdc.properties  
```
dc=DC2
rack=RAC1
```
```
nodetool status
```
Output:  
```
Note: Ownership information does not include topology; for complete information, specify a keyspace
Datacenter: DC2
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address       Load       Owns   Host ID                               Token                                    Rack
DN  172.31.16.13  49.81 KB   26.1%  0eabc4d5-7083-40af-b519-c9b85430bd72  -9166395768483610552                     RAC1
UN  172.31.16.12  2.13 MB    24.7%  2490f754-1ea8-4d5d-a8c2-0c472765ed0e  -9210479190555184443                     RAC1
Datacenter: DC1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address       Load       Owns   Host ID                               Token                                    Rack
UN  172.31.16.11  2.12 MB    25.6%  18f82ae9-1d2a-45e0-9ff2-143a9143e9df  -9199022843233318307                     RAC1
UN  172.31.16.10  1.91 MB    23.6%  5f5ed4b3-8602-48d2-8ec5-275c422d9e40  -9151020401596128422                     RAC1
```
###### CHANGING REPLICATION STRATEGY FOR KEYSPACES
```
ALTER KEYSPACE vehicle_tracker WITH replication = { 'class': 'NetworkTopologyStrategy', 'DC1': 2, 'DC2': 2 };
```
- When you change repplication strategy, you need to run repair.  
```
nodetool -h 172.31.16.11 repair vehicle_tracker
```

#### RESOURCES FOR FURTHER LEARNING
LINKS  
- https://docs.datastax.com/docs  
- Core PDF, CQL Guide.  
- https://academy.datastax.com/developer-blog  
- http://cassandra.apache.org/blog/  
- planetcassandra.org/blog/  
BOOKS
- Apache Cassandra Hands-On Training Level One  
- Cassandra High Performance Cookbook  
VIDEOS
- www.youtube.com/user/PlanetCassandra (Patrick McFadin is with huge knowled about cassandra)  
POSTING QUESTIONS
- stackoverflow.com/questions/tagged/cassandra  

EVENTS
- cassandra.meetup.com  




- QUESTIONS
- How to change indexing, is it possible.
- How to delete a column in cassandra ?
- Which file system to use for cassandra and tuning.  
- How to set default gc_grace period.
- How to set default TTL for a table.
- Does changing seed requires restart of cassandra service.
- How to plan token numbers.  
- How to know replication status
- How to know number of tokens, number of token ranges.
- How to change replication status.  
- nodetool status, what does it mean `Note: Ownership information does not include topology; for complete information, specify a keyspace`
- Cassandra how application finds the closer end point of cassanda  
- Point in time backup
- replication strategies
- Learn opscenter
- Learn gossip and things like that. 
- Authentication and backup.  
- Note: Ownership information does not include topology; for complete information, specify a keyspace (nodetool status). What is it ?  




