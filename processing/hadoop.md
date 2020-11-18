# Hadoop

## Distributed Computing
- Lots of cheap hardware
    - HDFS
- Replication and Fault Tolerance
    - YARN
- Distributed Computing
    - MapReduce

## HDFS
- GCS is used on GCP.
    - Don’t use HDFS as you would have to pay for a VM on Compute Engine.
- Suited for batch processing.
    - Data access has high throughput rather than low latency.
### Architecture
- **Name Node**
    - 1 master node
    - Contains YARN resource manager
    - Manages overall file system
    - Stores
        - The directory structure
        - Metadata on the files
- **Data Nodes**
    - Physically stores the data in the files.
### Storing Data
- Break data into blocks of equal size
    - Different length files are treated the same way
    - Storage is simplified
    - Unit for replication and fault tolerance
- Blocks are of size 128 MB
    - Larger -> Reduces parallelism
    - Smaller -> Increases overhead (more metadata)
- Stores the blocks across the data nodes
    - Each node contains a partition or a split of data
    - How do we know where the splits of a particular file are?
        - Name Node (File 1 | Block 1 | Data Node)
### High Availability
- Can have multiple name nodes.
- Kept in sync with Zookeeper
### Default Replication Strategy
- Maximize Redundancy
    - 1st location chosen at random
    - 2nd has to be on a different rack (if possible)
    - 3rd will be on same rack as the second, but on a different node.
        - Reduces inter-rack traffic and improves write performance.
    - Read operations are sent to the rack closest to the client.
- Minimize Write Bandwidth
    - Data is forwarded from first data node to the next replica location.
    - Forwarded further to the next replica location.
    - Forwarding requires a large amount of bandwidth.
        - Increases cost of writes.

## MapReduce
### Map
- An operation performed in parallel, on small portions of dataset.
- Outputs KV pairs
### Reduce
- Mapper outputs become one final output.

SQL interface over MapReduce = Hive
- Data analysts understand SQL but not Java code.
1. What {key, value} pairs should be emitted in the map step?
2. How should values with the same key be combined?

## YARN (Yet Another Resource Negotiator)
- Coordinate tasks running on the cluster.
- Assign new nodes in case of failure.
### Architecture
#### Resource Manager
- Runs on a single master node
- Schedules tasks across nodes
- Starts Application Master within containers.
#### Node Manager
- Run on all other nodes
- Manages tasks on the individual node.
- Can have multiple containers.
- Can request containers for mappers and reducers.
#### Application Master
- If additional resources are required, Application Master makes the request.
- 1 instance per application.
- Client communicates directly to get status, progress updates via an application-specific protocol.
#### Container
- All processes are run within a container in a Node Manager.
- Package of resources including RAM, CPU, Network, HDD etc on a single node.
- Executes the application code.
- Can communicate with Application Master itself.
### Location Constraint
- Assign a process to the same node where the data to be processed lives.
- If CPU/Memory not available, WAIT!
### Scheduling Policies
- FIFO Scheduler
    - Queue
- Capacity Scheduler
    - Priority Queue
- Fair Scheduler
    - Jobs assigned equal share of all resources

## HBase
- Database management system on top of Hadoop.
- Integrates with your application just like a traditional database.
### Columnar Store
- Advantages
    - Sparse Tables
        - No wastage of space when storing data.
    - Dynamic Attributes
        - Update attributes dynamically without changing storage structure.
        - Do not need to change schema.
### Denormalized Storage
- Column names repeat across rows.
- Normalization Reduces data duplication => Optimizes storage.
    - Storage is cheap in a distributed file system.
    - Optimize number of disk seeks instead by denormalization.
        - Don’t have to join tables.
- Read a single record to get all details about an employee in one read operation.
### Only CRUD operations
- No comparisons/sorting/inequality checks across multiple rows
    - No joins
    - No group by
    - No order by
- No operations involving multiple tables
- No indexes on tables
- No constraints
### ACID at ROW level
- Updates to a single row are atomic
    - All columns are updated, or none are.
- Updates to multiple rows are not atomic
    - Even if update is on the same column in multiple rows.

## Hive
- Provides a SQL interface to Hadoop.
- Bridge to Hadoop for people without OOP exposure.
- Not suitable for very low latency apps due to HDFS.
- HiveQL ~= SQL
- Wrapper on top of MapReduce
### Metastore
- HCatalog
- Bridge between HDFS and Hive
- Stores metadata for all tables in Hive
- Maps the files and directories in Hive to tables
- Holds the definitions and the schema for each table
- Any database with a JDBC driver can be used as a metastore.
- Development
    - Use built-in Derby database
    - Embedded metastore
    - Only one session can connect.
- Production
    - Local metastores
        - Allow multiple sessions to connect to Hive
        - DB is a separate process and can be on separate host.
    - Remote metastores
        - Separate processes for Hive and the metastore
        - Metastore runs in its own JVM process.
        - Processes communicate with Metastore using Thrift network API (hive.metastore.uris property)
        - Does not require admin to share JDBC login info for the metastore db with each Hive user.
- Hive vs. RDBMS
    - Large vs. Small datasets
    - Parallel vs. serial computations
    - High vs. low latency
    - Read vs. Read/write operations
    - Not ACID compliant vs. ACID compliant
- HiveQL vs. SQL
    - High latency
        - Records not indexed.
        - Fetching a row runs a MapReduce which may take minutes.
        - Not owner of the data.
            - It exists in HDFS
        - Schema-on-read
    - Not ACID compliant
        - Data can be dumped into Hive tables from any source
    - Row level updates, deletes as a special case
    - Many more built in functions
    - Only equi-joins allowed
- OLAP in Hive
    - Partitioning
        - State specific queries will run only on data in one directory.
        - Splits NOT of the same size.
    - Bucketing	
        - Size of each split should be the same.
            - Hash of a column value
        - Each bucket is a separate file
        - Makes sampling and joining data more efficient
            - Reduces search space
    - Join Optimizations
        - Join operations are Map Reduce jobs under the hood
            - Optimize joins by reducing the amount of data held in memory
        - Reducing data held in memory
            - On a join, one table is held in memory while the other is read from disk
                - Hold smaller in memory
        - Structuring Joins as Map-Only Operation
            - Filter queries (only these rows)
                - Mapper needs to use null as key
    - Windowing in Hive
        - A suite of functions which are syntactic sugar for complex queries.
        - e.x. What revenue percentile did this supplier fall into this quarter?
            - Window = 1 quarter
            - Operation = Percentile on revenue

## Pig
- ETL
- A data manipulation language
- Transforms unstructured data into a structured format
- Query this structured data using interfaces like Hive.
- Raw Data -> Pig -> Warehouse -> HiveQL -> Analytics
- Pig Latin
    - A procedural, data flow language to extract, transform and load.
        - Procedural
            - Uses a series of well-defined steps to perform operations.
            - No if statements or for loops.
            - Specifies exactly how data is to be modified at each step.
        - Data Flow
            - Focused on transformations applied to the data.
            - Written with a series of data operations in mind.
            - Nodes in a DAG
    - Data from one or more sources can be read, processed and stored in parallel.
    - Cleans data, precomputes common aggregates before storing in a data warehouse.
- Pig on Hadoop
    - Optimizes operations before MapReduce jobs are run, to speed operations up.
- Works better with Apache Tez and Spark.

## Oozie
- A tool used to schedule workflows on all the Hadoop ecosystem technologies.
