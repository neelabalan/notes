using [dataset](https://github.com/neelabalan/mongodb-sample-dataset)

- [Basics](#basics)
  - [commands](#commands)
    - [for import and export](#for-import-and-export)
    - [for commandline querying](#for-commandline-querying)
    - [commands for inserting](#commands-for-inserting)
    - [insertion without order](#insertion-without-order)
    - [updating documents commands](#updating-documents-commands)
    - [deletion commands mql](#deletion-commands-mql)
    - [delete a collection](#delete-a-collection)
    - [comparison operators](#comparison-operators)
    - [unary operators](#unary-operators)
    - [expressive query operator](#expressive-query-operator)
    - [array operators](#array-operators)
      - [projection](#projection)
    - [querying arrays and subdocuments](#querying-arrays-and-subdocuments)
    - [aggregation](#aggregation)
      - [sort and limit methods](#sort-and-limit-methods)
    - [index](#index)
    - [upsert](#upsert)
- [Cluster Management](#cluster-management)
        - [file structure](#file-structure)
        - [replication commands](#replication-commands)
        - [reference commands](#reference-commands)
        - [logging](#logging)
            - [log verbosity levels](#log-verbosity-levels)
            - [log severity levels](#log-severity-levels)
        - [what is replication?](#what-is-replication)
        - [replica set](#replica-set)
        - [database profiling](#database-profiling)
            - [commands](#commands-1)
        - [built-in roles](#built-in-roles)
        - [security](#security)
            - [commands](#commands-2)
        - [replication commands](#replication-commands-1)
  - [sharding](#sharding)
    - [what is sharding](#what-is-sharding)
    - [shard key](#shard-key)
    - [picking a good shard key](#picking-a-good-shard-key)
    - [hashed shard key](#hashed-shard-key)
    - [chunks](#chunks)
    - [jumbo chunks](#jumbo-chunks)
    - [balancing](#balancing)
    - [queries in sharded cluster](#queries-in-sharded-cluster)
    - [targeted queries vs scatter gather](#targeted-queries-vs-scatter-gather)
    - [read preference](#read-preference)
    - [write concerns](#write-concerns)
    - [read concerns](#read-concerns)
- [Aggregation](#aggregation-1)
        - [Introduction and Aggregation concepts](#introduction-and-aggregation-concepts)
        - [Basic aggregation `$match` and `$project`](#basic-aggregation-match-and-project)
        - [basic aggregation utilities](#basic-aggregation-utilities)
  
  - [core aggregation combining information](#core-aggregation-combining-information)
  - [facets](#facets)

# Basics

## commands

### for import and export

```bash
mongodump --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies"
mongoexport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --collection=sales --out=sales.json
mongorestore --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies"  --drop dump
mongoimport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --drop sales.json
```

### for commandline querying

```js
show dbs
use sample_training
show collections
db.zips.find({"state": "NY"}).count()
db.zips.find({"state": "NY", "city": "ALBANY"}).pretty()
```

### commands for inserting

```js
db.inspections.findOne();
db.inspections.insert({
    "_id" : ObjectId("56d61033a378eccde8a8354f"),
    "id" : "10021-2015-ENFO",
    "certificate_number" : 9278806,
    "business_name" : "ATLIXCO DELI GROCERY INC.",
    "date" : "Feb 20 2015",
    "result" : "No Violation Issued",
    "sector" : "Cigarette Retail Dealer - 127",
    "address" : {
            "city" : "RIDGEWOOD",
            "zip" : 11385,
            "street" : "MENAHAN ST",
            "number" : 1712
    }
})
```

### insertion without order

```js
db.inspections.insert([
    { "_id": 1, "test": 1 },
    { "_id": 1, "test": 2 },
    { "_id": 4, "test": 3 }
])

db.inspections.insert(
    [
        { "_id": 1, "test": 1 },
        { "_id": 1, "test": 2 },
        { "_id": 3, "test": 3 }
    ],
    { "ordered": false }
)
```

### updating documents commands

```js
use sample_training
db.zips.updateMany({ "city": "HUDSON" }, { "$inc": { "pop": 10 } })
db.zips.updateOne({ "zip": "12534" }, { "$set": { "pop": 17630 } })
db.grades.updateOne(
    { 
        "student_id": 250, 
        "class_id": 399 
    },
    { 
        "$push": { 
            "scores": { 
                "type": "extra credit",
                "score": 100 
            }
        }
    }
)
```

### deletion commands mql

```js
db.inspections.deleteMany({ "test": 1 })
db.inspections.deleteOne({ "test": 3 })
```

### delete a collection

` db.<collection>.drop() `

### comparison operators

```js
db.trips.find({ 
    "tripduration": { "$lte" : 70 },
    "usertype": { "$ne": "Subscriber" } 
}).pretty()

db.trips.find({ 
    "tripduration": { "$lte" : 70 },
    "usertype": { "$eq": "Customer" }
}).pretty()

db.trips.find({ 
    "tripduration": { "$lte" : 70 },
    "usertype": "Customer" 
}).pretty()
```

### unary operators

```js
db.routes.find({ 
    "$and": [
        { 
            "$or" :[ 
                { "dst_airport": "KZN" },
                { "src_airport": "KZN" }
            ]
        },
        { 
            "$or" :[ 
                { "airplane": "CR2" },
                { "airplane": "A81" } 
            ] 
        }
    ]
}).pretty()

db.inspections.find({ 
    "$or": [ 
        { "date": "Feb 20 2015" },
        { "date": "Feb 21 2015" } 
    ],
    "sector": { "$ne": "Cigarette Retail Dealer - 127" }
}).pretty()
```

### expressive query operator

```js
db.trips.find({ 
    "$expr": { 
        "$eq": [ "$end station id", "$start station id"] 
    }
}).count()

db.trips.find({ 
    "$expr": { 
        "$and": [ 
            { "$gt": [ "$tripduration", 1200 ]},
            { "$eq": [ "$end station id", "$start station id" ]}
        ]
    }
}).count()
```

### array operators

```js
db.listingsAndReviews.find({ 
    "amenities": {
        "$size": 20,
        "$all": [ 
            "Internet", "Wifi",  "Kitchen",
            "Heating", "Family/kid friendly",
            "Washer", "Dryer", "Essentials",
            "Shampoo", "Hangers",
            "Hair dryer", "Iron",
            "Laptop friendly workspace" 
        ]
    }
}).pretty()
```

#### projection

```js
db.listingsAndReviews.find(
    { 
        "amenities": { 
            "$size": 20, 
            "$all": [ 
                "Internet", "Wifi",  "Kitchen", "Heating",
                "Family/kid friendly", "Washer", "Dryer",
                "Essentials", "Shampoo", "Hangers",
                "Hair dryer", "Iron",
                "Laptop friendly workspace" 
            ] 
        } 
    }, 
    {
        "price": 1, 
        "address": 1
    }
).pretty()
```

```js
db.grades.find(
    { "class_id": 431 },
    { 
        "scores": { 
            "$elemMatch": { "score": { "$gt": 85 } } 
        }
    }
).pretty()
```

### querying arrays and subdocuments

```js
use sample_training
db.companies.find(
    { "relationships.0.person.last_name": "Zuckerberg" },
    { "name": 1 }
).pretty()

db.companies.find(
    { 
        "relationships.0.person.first_name": "Mark",
        "relationships.0.title": { "$regex": "CEO" } 
    },
    { "name": 1 }
).count()

db.companies.find(
    { 
        "relationships": { 
            "$elemMatch": { 
                "is_past": true,
                "person.first_name": "Mark" 
             } 
         } 
    },
    { "name": 1 }
).pretty()
```

### aggregation

```js
db.listingsAndReviews.find(
    { "amenities": "Wifi" },
    { "price": 1, "address": 1, "_id": 0 }
).pretty()

db.listingsAndReviews.aggregate([
    { "$match": { "amenities": "Wifi" } },
    { 
        "$project": { 
            "price": 1,
            "address": 1,
            "_id": 0 
         }
    }
]).pretty()

db.listingsAndReviews.aggregate([ 
    { 
        "$project": { 
            "address": 1, 
            "_id": 0 
        }
    },
    { 
        "$group": { 
            "_id": "$address.country" 
        }
    }
])

db.listingsAndReviews.aggregate([
    { "$project": { "address": 1, "_id": 0 }},
    { 
        "$group": { 
            "_id": "$address.country",
            "count": { "$sum": 1 } 
         } 
    }
])
```

#### sort and limit methods

```js
use sample_training
db.zips.find().sort({ "pop": 1 }).limit(1)
db.zips.find().sort({ "pop": 1, "city": -1 })

db.companies.find( 
    {"founded_year":{"$ne":null}},
    {"name":1, "founded_year":1}
).sort({"founded_year":1}).limit(5)
```

### index

```js
db.trips.createIndex({ "birth year": 1 })
db.trips.createIndex({ "start station id": 476, "birth year": 1 })
```

### upsert

> if document present update else insert

```js
db.iot.updateOne(
    { 
        "sensor": r.sensor, 
        "date": r.date,
        "valcount": { "$lt": 48 } 
    },
    { 
        "$push": { 
            "readings": { 
                "v": r.value, 
                "t": r.time 
            } 
        },
        "$inc": { "valcount": 1, "total": r.value } 
     },
     { "upsert": true }
)
```

# Cluster Management

### file structure

- A daemon is a program or process that’s meant to be run but not interacted with directly.
- mongod is the main daemon process for MongoDB
  - it is the core server of the database, handling connections, requests, and most importantly, persisting your data
  - mongod contains all of the configuration options to make the db secure, distributed, and consistent
  - when mongod is launched, it essentially means starting up a new database
  - how to communicate with mongod
    - a database client that is programmed to communicate with mongod.
    - issue database commands, like document inserts or updates, to the client, and the client takes care of communicating with mongod to execute those commands
  - default configuration for mongod
    - port: 27017
    - dbpath: /data/db
      - This is where the data files representing databases, collections, and indexes are stored so that your data persists after mongod stops running.
      - The dbpath also stores journaling information so that your data remains consistent in the case of an unexpected crash.
    - bind_ip: localhost
    - auth: disabled

```bash
# dbpath
ls -l /data/db

# diagnostics data
ls -l /data/db/diagnostic.data

# list journal directory
ls -l /data/db/journal

# list socket file
ls /tmp/mongodb-27017.sock

# default log file
tail /var/log/mongodb/mongod.log
```

### replication commands

- `rs.status()`
  - reports general health of each node
  - gets data from hearbeat
  - stats
    - uptime
    - optime (last time operation)
- `rs.isMaster()`
  - describes a node’s role in replica set
  - shorter output than `status`
- `db.serverStatus()['repl']`
  - section of `db.serverStatus()` output
  - similar to `rs.isMaster()`
  - `rbid` is included here
- `rs.printReplicationInfo()`
  - only returns oplog data relative to current node
  - contains timestamps for first and last oplog events
  - oplog is periodically flushed to make new data

### reference commands

```javascript
db.createUser()
db.dropUser()

// collection management
db.<collection>.renameCollection()
db.<collection>.createIndex()
db.<collection>.drop()

// database management commands
db.dropDatabase()
db.createCollection()

// db status
db.serverStatus()
```

### logging

two logging facilities provided by mongodb for tracking activities

- process logs displays activity on the mongodb instance
- the process log collects activity into one of the following components
  - ACCESS
    - access control, such as authentication
  - COMMAND
    - database commands
  - CONTROL
    - control activities, such as initialization
  - FTDC
    - the diagnostic data collection mechanism
  - GEO
    - the parsing of geospatial shapes
  - INDEX
    - index related
  - NETWORK
    - network activities related
  - QUERY
    - queries including planner activities
  - REPL
    - replica sets
  - REPL_HB
    - replica set hearbeats
  - ROLLBACK
    - rollback operations
  - SHARDING
    - sharding related
  - STORAGE
    - storage actvities
  - JOURNAL
    - journaling activities
  - WRITE
    - write operations and update commands

#### log verbosity levels

> -1: inherit from parent
> 0: default verbosity, to include informational messages
> 1-5: increases verbosity level to include debug messages

#### log severity levels

- F - Fatal
- E - Error
- W - Warning
- I - Informational
- D - Debug

```js
// get logging componenets
db.getLogComponents()
// change logging level
db.setLogLevel(0, "index")
// view logs 
db.adminCommand({ "getLog": "global" })j
```

`tail -f /data/db/mongod.log`

### what is replication?

- replication is a concept of maintaining multiple copies of the data
- the main reason why replication is necessary is because one can never assume that all of your servers will always be available
- it can be due to
  - maintenance
  - disaster wipes
  - servers experiencing downtimes
- so the point of replication is to make sure that in the event the server goes down the access to data is not interupted. This concept is called availability
- mongodb uses asynchronous, statement based replication because it’s platform independent and allows more flixibility within a replica set

### replica set

- in mongodb a group of nodes that each have copies of the same data is called replica set
- in a replica set all data is handled by default in one of the nodes and it’s up to the remaining nodes in the set to sync up with it and replicate the new data that has been written throught asynchronous mechanism
- the node where data is sent is called primary node, all others secondary
- the goal here is that all nodes need to stay consistent with each other
- failover - one of the secondary node take up in case the primary node goes down
- the nodes decide specifically which secondary will become primary though an election
- once the node comes back online and it’s sure that it can catch up and sync on the latest copy of the data, it will re-join the repilca set automatically.
- replication
  - binary replication
    - they examine the exact byte that changed in the data files and recording those changes in a binary log. The secondary nodes then receive a copy of the binary log and write the specified data that changed to the exact byte locations that are specified on the binary log
    - this goes pretty easy on the secondaries because they get specific instruction on what bytes to chagne and what to chnage them to
    - the one asumption is that the OS is consistent across replica sets
    - the binary approach requires less data to be stored in the binary log which implicitly means this approach is much faster due to less lag in transfer of instructions to secondaries
  - statement based replication
    - after a write is completed on the primary node, the write statement itself is stored in the oplog, and the secondaries then sync their oplogs with the primary’s oplog and replay any new statements on their own data
    - the right commands actually undergo a small transformation before they’re stored in the oplog
    - the idea here is that the statement can be applied indefinte number of times while still resulting in the same data state. This property is called **idempotency

### database profiling

- events captured by Mongodb profiler
  - CRUD
  - administrative operations
  - configuration operations 

| Level | Description                                                                         |
| ----- | ----------------------------------------------------------------------------------- |
| 0     | The profiler is off and does not collect any data. This is default profiler level   |
| 1     | The profiler collects data for operations that take longer than the value of slowms |
| 2     | The profiler collects data for all operations                                       |

#### commands

```js
db.runCommand({listCollections: 1})
```

```js
db.getProfilingLevel()
db.getCollectionNames()
db.setProfilingLevel( 1, { slowms: 0 } )
```

### built-in roles

- role based access control (RBAC)
- database users are granted roles
- custom roles
- built-in roles
  - database user
    - read, readWrite
  - database admin
    - dbAdmin
      - collStats
      - dbHash
      - dbStats
      - killCursors
      - listIndexer
      - listCollections
      - bypassDocumentValidation
      - collMod
      - collStats
      - compact
      - convertToCapped
    - userAdmin
      - changeCustomData
      - changePassword
      - createRole
      - createUser
      - dropRole
      - dropUser
      - grantRole
      - revokeRole
      - setAuthenticationRestriction
      - viewRole
      - viewUser
      - …more
    - dbOwner
  - clutser admin
    - clusterAdmin
    - clusterManager
    - clusterMonitor
    - hostManager
  - backup/restore
    - backup and restore
  - super user
    - root
- role structure
  - privileges
  - action + resource
- resource
  - database
  - collection
  - set of collections
  - cluster
    - replica set
    - shard cluster
- role structure
  - role is composed of
    - set of privileges
      - actions -> resources
    - network authentication restrictions
      - clientSource
      - serverAddress

### security

- authentication client
  
  - SCRAM (salted challenge response authentication mechanism)
  - X.509 certificate for authentication
  - enterpise only
    - LDAP (light weight directory access protocol)
    - kerberos

- cluster authentication
  
  - intercluster auth

- authorization
  
  - role based access control
    - each user has one or more roles
    - each role has one more privileges
    - a privilege represents a group of Actions and Resource those action apply to

- always create a user with admin privileges first

#### commands

```js
use admin
db.createUser({
  user: "root",
  pwd: "root123",
  roles : [ "root" ]
})

// mongo --username root --password root123 --authenticationDatabase admin

db.stats()
use admin
db.shutdownServer()
```

```javascript
mongo admin -u root -p root123

db.createUser(
  { user: "security_officer",
    pwd: "h3ll0th3r3",
    roles: [ { db: "admin", role: "userAdmin" } ]
  }
)

db.createUser(
  { user: "dba",
    pwd: "c1lynd3rs",
    roles: [ { db: "admin", role: "dbAdmin" } ]
  }
)

db.grantRolesToUser( "dba",  [ { db: "playground", role: "dbOwner"  } ] )

db.runCommand( { rolesInfo: { role: "dbOwner", db: "playground" }, showPrivileges: true} )
```

```js
// get replica set status
rs.status()

// adding other members to replica set
rs.add("repl:27012")
rs.add("repl:27013")

// getting an overview of the replica set topology
rs.isMaster()

// stepping down the current primary
rs.stepDown()

// checking replica set overview after election
rs.isMaster()
```

### replication commands

- `rs.status()` 
  - reports general health of each node
  - gets data from hearbeat
  - stats
    - uptime
    - optime (last time operation)
- `rs.isMaster()` 
  - describes a node's role in replica set
  - shorter output than `status`
- `db.serverStatus()['repl']`
  - section of `db.serverStatus()` output
  - similar to `rs.isMaster()`
  - `rbid` is included here
- `rs.printReplicationInfo()`
  - only returns oplog data relative to current node
  - contains timestamps for first and last oplog events
  - oplog is periodically flushed to make new data

```js
use local
show collections

// query the oplog after connected to replica set
use local
db.oplog.rs.find()

// store oplog stats as variable called stats
var stats = db.oplog.rs.stats()
var stats = db.oplog.rs.stats(1024*1024)

// verify that this collection is capped
stats.capped

// get current size of oplog
stats.size

// get size limit of oplog
stats.maxSize

// get current oplog data
rs.printReplicationInfo()
```

## sharding

### what is sharding

- Sharding is the process of breaking up large tables into smaller chunks called shards that are spread across multiple servers.
- A shard is essentially a horizontal data partition that contains a subset of the total data set, and hence is responsible for serving a portion of the overall workload
- In between a sharded cluster and it’s clients we set up a kind of router process that accepts queries from clients and then figures out which shard should receive that query. This router process is called mongos
- mongos uses the metadata about which data is contained in which shard but the data is stored in the config servers.
- But the data is very often used by mongos. So it has to made sure that the data stays highly available. So a replication mechaism is used to replicate the data on config servers
- config server replica set is deployed for this purpose

### shard key

- This is the indexed field or fields that mongodb uses to partition data in a sharded collection, and distribute it across the shards in your cluster
- mongodb uses the shard key to divide up these documents into logical groupings that mongodb then distributes across our cluster. Mongodb then referes to these groupings as chunks
- the value fo the field or fields we choose as our shard key help to define the inclusive lower and the exclusive upper bound of each chunk. The shard key is used to define chunk boundaries, it also defines which chunk a given document is going to belong to
- every time a new document is written to a collection the mongo s router checks which shard contains the appropriate chunk for that documents key value and routes the document to that shard only
- shard key field must be present in every document in the collection

### picking a good shard key

- what makes a good shard key?
  - cardinality
    - many possible unique shard key values (high boundary)
    - higher cardanality means more chunks - no restriction on growth
  - frequency (need to be low freq)
    - high frequency = low repetition of a given unique shard key value
  - monotonic change (should be non-monotonic)
    - id fields which increase monotonically (need to be non linear)
- sharding is a permanent operation
  - you cannot unshard a collection once sharded
  - you cannot update the shard key of a sharded collection
  - you cannot update the values of the shard key for any document in the sharded collection
  - test your shard keys in a staging env first before sharding in prod

### hashed shard key

- underlying index is hashed
- can be used in monotonically changing id fields
  - example can be time stamped value
- queries on ranges of shard key values are more likely to be scatter-gather
- cannot support geographically isolated read operations using zoned sharding
- hashed index must be on a single non-array field
- hashed indexes dont’ support fast sorting
- steps
  - `sh.enableSharding("<database>")`
  - `db.collection.createIndex({"<field>": "hashed"})` to create index for your shard key fields
  - `sh.shardCollection("db.collection", {<shard key field> : "hashed" })`to shard the collection

### chunks

- chunks size, shard key cardinality/freq will determine the number of chunks
- chunks can only live at one designated shard at a time #### commands

### jumbo chunks

- larger than the defined chunk size
- cannot move jumbo chunks
  - once marked as jumbo the balancer skips these chunks and avoids trying to move them
  - in some cases these will not be able to split

### balancing

- good shard key should contribute to how evenly mongodb can distribute the data across the shards in your cluster
- the balancer process runs on the primary member of the config server replica set
- the balancer process checks the chunked distribution of data across the sharded cluster and looks for certain mingration threshold
- the balancer process makes sure that the cluster is evenly distributed as possible
- the balancer is an automatic process and requires minimal user input

### queries in sharded cluster

- whether we’re doing targeted or scatter gather queries, the mongos opens a cursor against each of the targeted shards.
- each cursor executes the query predicate, and returns any data returned by the query for that shard.
- the mongos now has the results from each targeted shard
- the mongos merges all of tha results together to form the total set of documents that fullfulss this query, and returns that set of documents to the client application.

### targeted queries vs scatter gather

- targeted queries require the shard key in the query for fast performance
- ranged queries on the shard key may still require targeting every shard in the cluster
- without the shard key, the mongos must perform a scatter-gather query

### read preference

- with read preference, we can direct the application to route it’s query to a secondary member of the replica set, instead of the primary.
- supported modes
  - primary
  - primaryPreferred (used during fail over)
  - secondary
  - secondaryPreferred (if not available goes to primary)
  - nearest (geo near)

### write concerns

- Write concern describes the level of acknowledgment requested from MongoDB for write operations to a standalone mongod or to replica sets or to sharded clusters. In sharded clusters, mongos instances will pass the write concern on to the shards.
- write concern levels
  - 0, don’t wait for acknowledgement
  - 1, wait for acknowledgement from the primary only
  - =2, wait for acknowledgement from the primary and one or more secondaries
  - “majority”, wait for ack from a majority of replica set members
- write concern options
  - wtimeout:
    - the time to wait for the requestd write concern before marking the operation as failed
  - j: <true|false> requires the node to commit the write operation to the journal before returning an ack
- write concern commands
  - insert
  - update
  - delete
  - findAndModify

### read concerns

- The readConcern option allows to control the consistency and isolation properties of the data read from replica sets and replica set shards
- read concern levels
  - fast and latest (only for primary read concerns)
    - local
    - available (sharded clusters)
  - fast and safe
    - majority
  - latest and safe
    - linearizable

# Aggregation

### Introduction and Aggregation concepts

- pipelines are compostion of stages

- stages are configured to produce desirable transformations

- stages can be arranged in multiple ways

- typical pipeline `$match -> $project -> $group`

- aggregation structure and sytax
  
  - `$match` example of aggregation operator
  
  - `$gte` example of query operator
  
  - `db.collection.aggregate( [ { <stage> }, ... ], {options} )`
  
  - example
    
    ```js
    db.solarSystem.aggregate(
      [
          {
              $match: {
                  atmosphericComposition: { $in: [/02/] },
                  meanTemperature: { $gte: -40, $lte: 40 },
              },
              $project: {
                  _id: 0,
                  name: 1,
                  hasMoons: { $gt: ["$numberOfMoons", 0] },
              },
          },
      ], 
      { allowDiskUse: true }
    );
    ```
  
  - field path expression `$fieldName` {`$numberOfMoons`}
  
  - system variable `$$UPPERCASE` {`$$CURRENT`}
  
  - user variable `$$foo`

### Basic aggregation `$match` and `$project`

- `$match`
  
  - used like filter
  
  - uses standard mongodb query operator
  
  - we can perform matches based on comparison, logic, arrays and much more
  
  - the only limitation is that we cannot use the `$where` operator
  
  - and if we want to use the `$test` operator, the `$match` stage must be the first stage in the pipeline
  
  - if `$match` is the first stage, it can take advantage of indexes which increase the speed of queries
    
    ```js
    var pipeline = [{
        $match: {
            $and: [
                { $or: [{ rated: "PG" }, { rated: "G" }] },
                { languages: { $all: ["English", "Japanese"] } },
                { genres: { $nin: ["Crime", "Horror"] } },
                { "imdb.rating": { $gte: 7 } },
            ],
        },
    }];
    ```

- `$project`
  
  - Passes along the documents with the requested fields to the next stage in the pipeline.
  
  - The specified fields can be existing fields from the input documents or newly computed fields.
  
  - once we specify one field to retain, we must specify all fields we want to retain.
  
  - The `_id` field is the only exception to this
  
  - `$project` can be used as many times as required within aggregation pipeline
  
  - `$project` can be used to reassign values to existing field name and to derive entirely new fields
  
  - example:
    
    ```js
    db.solarSystem.aggregate([{    
        $project: { 
            _id:0, 
            name:1 
        }
    }])
    db.solarSystem.aggregate([{    
        $project: {
            _id:0, 
            name:1, 
            "gravity.value":1 
        }
    }])
    earthdb.solarSystem.aggregate([
        {    
            $project: {        
                _id:0,         
                name: 1,        
                myweight: {             
                    $multiply: [                
                        { $divide: ["$gravity.value", 9.8 ] },                
                        86            
                    ]        
                }    
            }
        }
    ])
    
    var pipeline = [ 
        { 
            $match: { 
                $and: [ 
                    { $or: [ {rated: “PG”}, {rated: “G”} ] }, 
                    { languages: { $all: [“English”, “Japanese”] } }, 
                    { genres: { $nin: [“Crime”, “Horror”] } }, 
                    { "imdb.rating" : { $gte: 7 } }, 
                ] 
            } 
        }, 
        { 
            $project: { \_id: 0, title:1, rated:1 } 
        } 
    ]
    
    db.movies.aggregate([ 
        { 
            $project: { 
                _id:0, 
                titlesize: { 
                    $size: {$split: [“$title”, " "]} 
                } 
            }, 
        }, 
        { $match: { titlesize : 1 } } 
    ]).itcount()
    ```

### basic aggregation utilities

- `$addFields`
  
  - very similar to`$project` with one key difference - `$addFields`adds fields to a document 
  
  - while with`$project`, we can selectively remove and retain fields 
  
  - `$addFields`only allows to modify the incoming pipeline documents with new computer fields or to modify existing fields 
  
  - only value transformation is performed and other fields so as such without explicity calling them like done in`$project`
    
    ```js
      db.solarSystem.aggregate([ 
          { 
              $addFields: { 
                  gravity: "$gravity.value", 
                  mass:"$mass.value", 
                  radius: "$radius.value", 
                  sma:"sma.value" 
              } 
          } 
      ])
    
    // combining addFields and project
    
      db.solarSystem.aggregate([ 
          { 
              $project: { 
                  _id:0, 
                  name:1, 
                  gravity: 1, 
                  mass: 1, 
                  radius: 1, 
                  sma:1 } 
              }, 
          },
          { 
              $addFields: {  
                  gravity: "$gravity.value", 
                  mass:"$mass.value",  
                  radius: "$radius.value", 
                  sma:"sma.value" 
              }
          }
      )]
    ```

- `$geoNear`
  
  - aggregation framework solution to performing geoqueries within the aggregation pipeline
  
  - this must be the first stage in a pipeline
  
  - cannot use `$near` predicate in the query
  
  - can be used in charted collections, whereas `$near` cannot be used
  
  - this needs one and only one geoindex in the collection
  
  - summary points
    
    - the collection can have one and only one 2dsphere index
    
    - if using 2dsphere, the distance is returned in meters. If using logacy coordinates, the distance is returned in radians.
    
    - required arguments
    
    - near
      
      - it is the point where we like to search near
      - `near: { type: "point", coordinates: [ -73.456, 40.5678]}`
    
    - distanceField
    
    - spherical
      
      ```js
      db.nycFacilities.aggregate([
        {    
            $geoNear: {        
                near: {            
                    type: "Point",            
                    coordinates: [-73.9876543, 40.7575432]        
                },        
                distanceField: "distanceFromMongoDB",        
                spherical: true    
            }
        }
      ]).pretty()
      ```

- cursor like stages

```js
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).pretty()
// count the number of documents
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).count()
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).skip(5).pretty();
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).limit(5).pretty()
db.solarSystem.find({}, { "_id": 0, "name": 1, "numberOfMoons": 1 }).sort( {"numberOfMoons": -1 } ).pretty();
db.solarSystem.aggregate([
    {  
        "$project": {    
            "_id": 0,    
            "name": 1,    
            "numberOfMoons": 1  
            }
        },
        { "$limit": 5  }
]).pretty();

db.solarSystem.aggregate([
    {  
        "$project": {    
            "_id": 0,    
            "name": 1,    
            "numberOfMoons": 1  
        }
    }, 
    {  "$skip": 1}
]).pretty()
db.solarSystem.aggregate([        
    {            
        "$match": {                
            "type": "Terrestrial planet"            
        }        
    },         
    {            
        "$project": {                
            "_id": 0,                
            "name": 1,                
            "numberOfMoons": 1            
        }        
    },         
    {            
        "$count": "terrestrial planets"        
    }    
]).pretty()

db.solarSystem.aggregate([
    {  
        "$match": {    
            "type": "Terrestrial planet"  
        }
    }, 
    {  "$count": "terrestrial planets"}
]).pretty()

db.solarSystem.aggregate([
    {  
        "$project": {    
            "_id": 0,    
            "name": 1,    
            "numberOfMoons": 1  
        }
    }, 
    {  
        "$sort": { 
            "numberOfMoons": -1 
        }
    }
]).pretty();

db.solarSystem.aggregate([
    {  
        "$project": {    
            "_id": 0,    
            "name": 1,    
            "hasMagneticField": 1,    
            "numberOfMoons": 1  
        }
    }, 
    {  
        "$sort": { 
            "hasMagneticField": -1, 
            "numberOfMoons": -1 
        }
    }
]).pretty();

db.solarSystem.aggregate([
    {  
        "$project": {    
            "_id": 0,    
            "name": 1,    
            "hasMagneticField": 1,    
            "numberOfMoons": 1  
        }
    }, 
    {  
        "$sort": { 
            "hasMagneticField": -1, 
            "numberOfMoons": -1 
        }
    }], 
    { "allowDiskUse": true }
).pretty();
```

- `$sample`
  - it will select a set of random documents from a collection in one of two ways
    - `{ $sample: {size: < N, how many documents }}`
      - when N<=5% of number of documents in source collection AND
      - source collection has >=100 documents AND
      - `$sample` is the first stage
    - if any of the above conditions do not apply `{sample: {size: < N}}`

```js
var pipeline = [    
    {        
        $match: {            
            $and: [                
                { countries: { $all: ["USA"] } },                
                { "tomatoes.viewer.rating":{ $gte: 3}},                
                { cast: {$exists: true} }            
            ]        
        }    
    },    
    {        
        $project: {            
            num_favs: {                 
                $size:  {                     
                    $setIntersection: [                         
                        ["Sandra Bullock", "Tom Hanks", "Julia Roberts", "Kevin Spacey", "George Clooney"],                         
                        "$cast"                    
                    ]                 }
            },            
            title: 1,            
            "tomatoes.viewer.rating":1,            
            _id:0                    
        }    
    },    
    {
        "$sort": { 
            num_favs: -1, 
            "tomatoes.viewer.rating": -1, 
            title: -1 
        } 
    },    
    {
        "$limit": 25
    }
]

var pipeline = [    
    {        
        $match: {            
            $and: [                    
                { languages: { $all: ["English"] } },                    
                { "imdb.rating": {$gte: 1}},                    
                { "imdb.votes": {$gte: 1}},                    
                { year: {$gte:1990}}            
            ]        
        }    
    },    
    {        
        $project: {            
            scaled_votes: {                
                $add: [                    
                    1,                    
                    {                        
                        $multiply: [                            
                            9,                            
                            {                                
                                $divide: [                                    
                                    { $subtract: ["$imdb.votes", 5] },                                    
                                    { $subtract: [1521105, 5] }                                
                                ]                            
                            }                        
                        ]
                    }                
                ]            
            },            
            normalized_rating: { 
                $avg: ["$scaled_votes", "$imdb.rating"]
            },            
            title:1,            
            _id:0        
        }    
    },    
    {        
        $sort:{ normalized_rating:1}    
    }
]
```

## core aggregation combining information

- `$group`
  - summary
    - `_id` is where to speicify what incoming docuemnts should be grouped on
    - can use all accumulator expressions within `$group`
    - `$group` can be used multiple times within a pipeline
    - it may be necessary to sanitize incoming data

```js
db.movies.aggregate([    
    {        
        $group: {            
            _id: "$year",            
            num_films_in_year: { $sum: 1}        
        }    
    }
])

db.movies.aggregate([    
    {        
        $group: {            
            _id: {                
                numDirectors: {                    
                    $cond: [
                        { $isArray: "$directors"}, 
                        {$size: "$directors"}, 
                        0
                    ]                
                }            
            },            
            numFilms: { $sum: 1},            
            averageMetacritic: { $avg: "$metacritic" }        
        }    
    },    
    {        
        $sort: {"_id.numDirectors": -1}    
    }
])

db.movies.aggregate([    
    {        
        $match: {metacritic: {$gte: 0}}    
    },    
    {        
        $group: {            
            _id: null,            
            averageMetacritic: {$avg: "$metacritic"},        
        }    
    }
])
```

- accumulator stages with `$project`
  
  - summary
    
    - available accumulator expressions `$sum $avg $max $min $stdDevPop $stdDevSam`
    
    - expression have no memory between documents
    
    - may still have to use `$reduce` or `$map` for more complex calculations 
      
    ```js
    db.icecream_data.aggregate([{ 
        $project: { 
            _id:0, 
            max_high: { 
                $reduce: { 
                    input: "$trends", 
                    initialValue: -Infinity, 
                    in: { 
                        $cond: [ 
                            { 
                                $gt: ["$$this.avg_high_tmp", "$$value"]
                            }, 
                            "$$this.avg_high_tmp", "$$value" 
                        ] 
                    } 
                } 
            } 
        } 
    }]) 
    db.icecream_data.aggregate([{ 
        $project: { 
            _id:0, 
            max_high: { $max: "$trends.avg_high_tmp"}
        }
    }])
    db.icecream_data.aggregate([ 
        { 
            $project: { 
                _id:0, 
                average_cpi: { 
                    $avg: "$trends.icecream_cpi"
                }, 
                cpi_deviation: {
                    $stdDevPop: "$trends.icecream_cpi"
                } 
            } 
        } 
    ]) 
    db.movies.aggregate([ 
        { 
            $addFields: { 
                result: { 
                    $regexMatch: { 
                        input: "$awards", 
                        regex: /Won?/ 
                    } 
                } 
            } 
        }, 
        { $match: { result:true}}, 
        { 
            $group: { 
                _id: null, 
                highest_rating: {$max:"$imdb.rating"}, 
                lowest_rating: {$min: "$imdb.rating"}, 
                average_rating: {$avg:"$imdb.rating"}, 
                deviation: {$stdDevSamp: "$imdb.rating"} 
            } 
        } 
    ]) 

```
- `$unwind`
    - summary
        - `$unwind` only works on array values
        - there are two forms for unwind. long and short forms
        - using unwind on large collections with big documents may lead to performance issues
    ```js
    {    
        "title": "start trek",    
        "genre": [        
            "adventure",        
            "action"    
        ]
    }
    // above not equal to below works on pure equivalence
    {    
        "title": "star trek",    
        "genre": [        
            "action",        
            "adventure"    
        ]
    }

    db.movies.aggregate([    
        {        
            $match: {            
                "imdb.rating": { $gt: 0},            
                year: { $gte: 2010, $lte: 2015},            
                runtime: {$gte: 90}        
            }    
        },    
        {        
            $unwind: "$genres"    
        },    
        {        
            $group: {            
                _id: {                
                    year: "$year",                
                    genre: "$genres"            
                },            
                average_rating: { $avg: "$imdb.rating" }        
            }    
        }    
        {        
            $sort: { "_id.year": -1, average_rating: -1 }    
        }
    ])
    db.movies.aggregate([    
        {        
            $match: { 
                languages: { 
                    $all: ["English"] }
                },    
            },    
        },
        {        
            $unwind: "$cast"    
        },    
        {        
            $group:{            
                _id: "$cast",            
                numFilms: { $sum: 1},            
                average:{ $avg: "$imdb.rating"}        
            }    
        },    
        {        
            $sort:{ numFilms:-1}    
        }
    ])
```

- `$lookup`
  
  - format
    
    ```js
    $lookup: {
      from: <collection to join>,
      localField: <field from the input documents>,
      foreignField: <field from the docuemnts of the "from" collection>
      as: <output array field>
    }
    ```
  
  - summary
    
    - the from collection cannot be sharded
    
    - the from collection must be in the same database
    
    - the values in localField and foreignField are matched on equality
    
    - as can be any name, but if it exists in the working document that field will be overwritten
      
    ```js
    db.air_alliances.aggregate([{        
        $lookup: {            
            from: "air_airlines",            
            localField: "airlines",            
            foreignField: "name",            
            as: "airlines"        
        }    
    }]).pretty()

    db.air_routes.aggregate([    
        {        
            $match: {            
                airplane: /747|380/        
            }    
        },    
        {        
            $lookup: {            
                from: "air_alliances",            
                foreignField: "airlines",            
                localField: "airline.name",            
                as: "alliance"        
            }    
        },    
        {        
            $unwind: "$alliance"    
        },    
        {        
            $group: {        
                _id: "$alliance.name",        
                count: { $sum: 1 }        
            }    
        },    
        {        
            $sort: { count: -1 }    
        }
    ])

    db.air_routes.aggregate([    
        {         
        
            $addFields: {             
                result: {                 
                    $or:[                    
                        {$regexMatch: { input: "$airplane", regex: /747/ } },                    
                        {$regexMatch: { input: "$airplane", regex: /380/ } }                
                    ]            
                }        
            }    
        
        },    
        {        
        
            $match: {            
                result: true        
            }    
        
        },    
        {        
        
            $project: {            
                _id: 0,            
                airplane: 1,            
                "airline.name": 1        
            }    
        
        },    
        {        
        
            $group:{            
                _id: "$airline.name",            
                count: {$sum:1}        }
            }    
        
        }
    ])
    
    ```
    
- `$graphLookup`
  
  - transitive closure implementation
  
  - template
    
    ```js
    {
      $graphLookup: {    
          from: 'lookup table',    
          startWith: 'expression for value to start from',    
          connectFromField: 'field name to connect from',    
          connectToField: 'field name to connect to '    
          as: 'filed name for result array'    
          maxDepth: 'number of iter to perform'    
          depthField: 'field name for number of iter to reach this node'    
          restrictSearchWithMatch: 'match condition to apply to lookup'
      }
    }
    db.parent_referene.aggregate([         
      { $match: {name: 'Eliot'}},        
      {            
          $graphLookup: {                
              from: 'parent_reference',                
              startWith: '$_id',                
              connectFromField: '_id',                
              connectToField: 'reports_to',                
              as: 'all_reports'            
          }        
      }    
    ])
    
    db.parent_reference.aggregate([    
      {$match: {name:'shannon'}},    
      {        
          $graphLookup: {            
              from: 'parent_reference',            
              startWith: '$reports_to',            
              connectFromField: 'reports_to',            
              connectToField: '_id',            
              as: 'bosses'        
          }    
      }
    ])
    
    db.child_reference.aggregate([        
      {$match: {name: 'Dev'}},        
      {            
          $graphLookup: {                
              from: 'child_reference',                
              startWith: '$direct_reports',                
              connectFromField: 'direct_reports',                
              connectToFiled: 'name',                
              as: 'till_2_level_reports',                
              maxDepth: 1           
          }        
      }    
    ])
    
    db.airlines.aggregate([    
      { $match: {name: "TAP Portugal"}},    
      {        
          $graphLookup: {            
              from: 'routes',            
              as: 'chain',            
              startWith: '$base',            
              connectFromField: 'dst_airport',            
              connectToField: 'src_airport',            
              maxDepth: 1        
          }    
      }
    ]).pretty()
    
    db.airlines.aggregate([    
      { $match: {name: "TAP Portugal"}},    
      {        
          $graphLookup: {            
              from: 'routes',            
              as: 'chain',            
              startWith: '$base',            
              connectFromField: 'dst_airport',            
              connectToField: 'src_airport',            
              maxDepth: 1,            
              restrictSearchWithMatch: { 'airline.name': 'TAP Portugal'}        
          }    
      }
    ]).pretty()
    
    db.air_alliances.aggregate([    
      {        
          $match: {name: "OneWorld"}    
      },    
      {        
          $graphLookup: {            
              startWith: "$airlines",            
              from: "air_airlines",            
              connectFromField: "name",            
              connectToField: "name",            
              as: "airlines",            
              maxDepth: 0,            
              restrictSearchWithMatch: {                
                  country: {$in: ["Germany", "Spain", "Canada"]}            
              }        
          }    
      },    
      {        
          $graphLookup: {            
              startWith: "$airlines.base",            
              from: "air_routes",            
              connectFromField: "dst_airport",            
              connectToField: "src_airport",            
              as: "connections",            
              maxDepth: 1        
          }    
      },    
      {        
          $project: {            
              validAirlines: "$airlines.name",            
              "connections.dst_airport": 1,            
              "connections.airline.name":1         
          }    
      },    
      {        
          $unwind: "$connections" ,    
      },    
      {        
          $project: {            
              isValid: {$in: ["$connections.airline.name", "$validAirlines"]},            
              "connections.dst_airport":1        
          }    
      },    
      {$match: {isValid: true}},    
      {$group:{_id: "$connections.dst_airport"}}
    ]).toArray().length
    ```

## facets

- summary
  
  - `$bucket`
    - must always specify at least 2 values to boundaries
    - boundaries must all be of the same general type
    - count is inserted by default with no output, but removed when output is specified
  - `$bucketAuto`
    - cardinality of groupBy expression may impact even distribution and number of buckets
    - specifying a granularity requires the expression to groupBy to resolve to a numeric value
    - `$sortByCount`
      - is equivalent to a group stage to count occurence, and the sorting in descending order

- `$sortByCount`
  
```js
db.companies.aggregate([    
    { 
        '$match': { 
            '$text': {'$search': 'network'}
        }
    },    
    { '$sortByCount': '$category_code' }
])

db.companies.aggregate([    
    {
        '$match': {'$text': { '$search': 'network'}}
    },    
    {'$unwind': '$offices'},    
    {'$match': {'offices.city': {'$ne': ''}}},    
    {'$sortByCount': '$offices.city'}
])

db.movies.aggregate([ 
    {'$sortByCount': '$imdb.rating'}
]).pretty()

```

- `$bucket`

```js
db.companies.aggregate([    
    {         
        '$match': {            
            'founded_year': { '$gt': 1980},            
            'number_of_employees': { '$ne': null}        
        }    
    },    
    {        
        '$bucket': {            
            'groupBy': '$number_of_employees',            
            'boundaries': [ 0, 20, 50, 100, 500, 1000, Infinity],            
            'default': 'Other'        
        }    
    }
])

db.companies.aggregate([    
    {         
        '$match': {            
            'founded_year': { '$gt': 1980},        
        }    
    },    
    {        
        '$bucket': {            
            'groupBy': '$number_of_employees',            
            'boundaries': [ 0, 20, 50, 100, 500, 1000, Infinity],            
            'default': 'Other',            
            'output': {                
                'total': {'$sum': 1},                
                'average': {'$avg': '$number_of_employees'},                
                'categories': {'$addToSet': '$category_code'}            
            }        
        }    
    }
])
```

- `$bucketAuto`

```js
db.companies.aggregate([    
    {        
        '$match': {            
            'offices.city': 'New York'        
        }    
    },    
    {        
        '$bucketAuto': {            
            'groupBy': '$founded_year',            
            'buckets': 5        
        }    
    }
])

db.companies.aggregate([    
    {        
        '$match': {            
            'offices.city': 'New York'        
        }    
    },    
    {        
        '$bucketAuto': {            
            'groupBy': '$founded_year',            
            'buckets': 5,            
            'output': {                
                'total': {'$sum':1},                
                'average': {'$avg': '$number_of_employees'}            
            }        
        }    
    }
])
```

- `$facet`

```js
db.companies.aggregate([    
    {        
        '$match': {            
            '$text': { '$search': 'Databases'}        
        }    
    },    
    {        
        '$facet': {            
            'Categories': [                
                {'$sortByCount': '$category_code'}            
            ],            
            'Employees': [                
                { '$match': {'founded_year': {'$gt': 1980}}},                
                {                     
                    '$bucket': {                        
                        'groupBy': '$number_of_employees',                        
                        'boundaries': [0, 20, 50, 100, 500, 1000, Infinity],                        
                        'default': 'Other'                    
                    }                
                }            
            ],            
            'Founded': [                
                {
                    '$match': {'offices.city': 'New York'}
                },                
                {                    
                    '$bucketAuto': {                        
                        'groupBy': '$founded_year',                        
                        'buckets': 5                    
                    }                
                }            
            ]
        }    
    }
])

db.movies.aggregate([    
    {'$match': {'metacritic': 100}},    
    { '$project': {        
        '_id': 0,        
        'imdb.rating': 1,        
        'metacritic': 1,        
        'title': 1    
    }}
]).pretty()

db.movies.aggregate([    
    {        
        '$match': {'imdb.rating': {'$ne': ""}}    
    },    
    {        
        '$sort':  { 'imdb.rating': -1 }     
    },    
    {        
        '$limit': 10    
    },    
    {        
        '$project': {            
            _id: 0,            
            'imdb.rating':1,            
            'title':1        
        }    
    }
]).pretty()

db.movies.aggregate([    
    {        
        '$match': {'metacritic': {'$gte': 99}}    
    },    
    {        
        '$sort':  { 'metacritic': -1 }     
    },    
    {        
        '$project': {            
            _id: 0,            
            // 'metacritic':1,            
            'title':1        
        }    
    }]
).pretty()

db.movies.aggregate([    
    {        
        '$match':  { 
            'imdb.rating': {'$gt': 8}
        },    
    },    
    {        
        '$facet': {            
            'IMDB': [                
                {                    
                    '$bucketAuto': {                        
                        'groupBy': '$imdb.rating',                        
                        'buckets': 1,                        
                        'output': {'$title'}                    
                    }                
                }            
            ],           
            'METACRITIC': [                
                {                    
                    '$bucketAuto': {                        
                        'groupBy': '$metacritic',                        
                        'buckets': 10,                        
                        'output': {'title': {'$addToSet': '$title'} }                    }
                }   
            ]       
        }    
    },    
    {        
        '$setIntersection': [ 
            '$IMDB', '$METACRITIC'
        ]    
    },    
    {        
        '$project': {            
            _id: 0,            
            title: 1        
        }    
    }
]).pretty()

db.movies.aggregate([  
    {    
        $match: {      
            metacritic: { $gte: 0 },      
            "imdb.rating": { $gte: 0 }    
        }  
    },  
    {    
        $project: {      
            _id: 0,      
            metacritic: 1,      
            imdb: 1,      
            title: 1    
        }  
    },  
    {    
        $facet: {      
            top_metacritic: [        
                {          
                    $sort: {            
                        metacritic: -1,            
                        title: 1          
                    }        
                },        
                {          
                    $limit: 10        
                },        
                {          
                    $project: {            
                        title: 1          
                    }        
                }      
            ],      
            top_imdb: [        
                {          
                    $sort: {            
                        "imdb.rating": -1,            
                        title: 1          
                    }        
                },        
                {          
                    $limit: 10        
                },        
                {          
                    $project: {            
                        title: 1          
                    }        
                }      
            ]    
        }  
    },  
    {    
        $project: {      
            movies_in_both: {        
                $setIntersection: ["$top_metacritic", "$top_imdb"]      
            }    
        }  
    }
])
```



- `$redact`
    - `$$KEEP` and `$$PRUNE` automatically apply to all levels below the evalutated levels
    - `$$DESCEND` retains the current level and evaluates the next level down
    - `$redact` is not for restricting access to a collection

- `$out`
    - must be last stage in the pipeline
    - summary
        - will create a new collection or overwrite an existing collection if specified
        - honors indexes on existing collections
        - will not create or overwrite data if pipeline errors
        - create collections in the same database as the source collection
        ```js
        db.collection.aggregate([
            { $group: { _id: null, values: {$push: "$some_field"}}},
            { $unwind: "$some_field"},
            { $out: "new_collection"}
        ])
        ```

- `$merge`
    - allows to output results of an aggregation pipeline right into another collection 
        - gives options to specify whether 
            - merge
            - insert
            - merge the results with the existing documents
            allowing to output to an existing sharder or unsharded collection and to different db
    
```js
{ 
    $merge: {
        into: <collection> -or- { db: <db>, coll: <collection> },
        on: <identifier field> -or- [ <identifier field1>, ...],  // Optional
        let: <variables>,                                         // Optional
        whenMatched: <replace|keepExisting|merge|fail|pipeline>,  // Optional
        whenNotMatched: <insert|discard|fail>                     // Optional
    } 
}

// custom pipeline
{
    $merge: {
        into: <target>,
        whenNotMatched: [
            {
                $addFields: {
                    totoal: {$sum: ["$total", "$$new.total"]}
                }
            }
        ]
    }
}
```

- `$views`
    - summary
        - views contain no data themselves. They are created on demand and reflect the data in the source collection
        - views are ready only. Write opeartions to vews will error
        - views have some restrictions. They must abide by the rules of the aggregation framework.
        - horizontal slicing is performed with the `$match` stage, reducing the number of documents that are resumed
        - vertical slicing is performed a `$project` or other shaping stage, modifying individual documents
    - no write operations
    - no index operations
    - no renaming
    - collation restrictions
    - find() operations with projection operators are not permitted
    - no `$text`
    - none of below operation permitted
    ```
    $, $elemMatch, $slice, $meta, $text, $geoNear 
    ```
    
