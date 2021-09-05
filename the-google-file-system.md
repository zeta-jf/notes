#1. Introduction:

component failures are the norm rather than the exception.

core feature: constant monitoring, error detection, fault tolerance and automatic recovery.

#2. Design Overview:
##2.1 Assumptions

files are huge, and not optimize small files
two types of read: sequencial read and random read (ofter sort random read according to offset)

num of write operations < num of read operations

use producer-consumer model to handle concurrent write/read

High sustained bandwidth > latency

##2.3 Architecture
cluster -> single master and multiple chunckservers

replication: 3

master maintains file system metadata

client implements the file system API and commnunicates with the master and chunkservers to read or write data.

##2.6 MetaData
file and chunk namespaces
the mapping from files to chunks
the locations of each chunk's replicas

###2.6.2 Chunk Locations
master use HeartBeat messages to controls all chunk placement and monitors chunkserver status.

###2.6.3 Operation log
GFS use operation log to record the critical change of metadata

# 3. System Interactions

## 3.1 Leases and Mutation Order

master would pick a chunk as the *primary*, and let primary to determine the order of mutation.

the lease would time out after 1 min, but the master could use *heart beat* message to extend the lease.

control workflow:
- client {request} -> master
  master{primary, replica locations} > client (if no primary, master grants one to replica it chooses)

- client will cache the response (since if it needs mutations in the future, it could reuse this information)

- client pushes the data to all replicas, chunkserver will store the data in an internal LRU

- once all the replicas have acknowledged receiving the data, the primary decided the mutation order(serialization number) and send the order to all chunckservers.

- replica confirmed that they have completed the operation.

- primary replies to the client.

## 3.2 Data flow

each machine forwards the data to the "closest" machine

## 3.3 Atomic record appends

this operation is atomic, at least once
write request does not contains offset, it just contains the data, master chooses where to append the data and return the offset.

## 3.4 Snapshot

- revoke leases on the chunks in the files.

- log the operation and copy the metadata in master (new created snapshot files point to the same chunks as the source file)

- master recevied a request which want to write data in *C*, then master asks all chunkserver which has a current replica of C to create a new chunck *C'*.

# 4. Master operation

## 4.1 Namespace management and locking

GFS does not have traditional path, it only has *path mapping table*
e.g. path: /a/b/c/../z/file.txt locks: /a /a/b ... /a/b/..z/ /a/b/.../z/file.txt

read lock prevent the directory from being deleted, renamed or snapshotted.
write locks serializes attempts to create a file with the same name twice.

## 4.5 Stale replica detection

use version number

# Fault tolerance and diagnosis

## 5.1 High availability

- fact recovery: master and chunkserver could recover from the termination
- chunk replication
- master replication: using operation log(multiple machines) to recover (restart), if master failed, choose another master.

shadow master: read-only

## 5.2 Data Integrity

detect corruption: 1 chunk(64MB) -> 1000 blocks(64kb, each has 32bit checksum), checksum is kept in memory and persistently stored with logging.

when read a chunck -> if checksum is not valid -> return error -> master replace it with a valid replica 


