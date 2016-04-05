# [Bigtable: A Distributed Storage System for Structured Data](http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)

## One-line Summary

Bigtable is a distributed storage system for managing structured data that is designed to scale to a very large size: petabytes of data across thousands of commodity servers.

## Introduction

* In many ways, Bigtable resembles a database: it shares many implementation strategies with databases.
* Bigtable provides clients with a simple data model that supports dynamic control over data layout and format, and allows clients to reason about the locality properties of the data represented in the underlying storage.
  * Data is indexed using row and column names that can be arbitrary strings.
* Finally, Bigtable schema parameters let clients dynamically control whether to serve data out of memory or from disk.

## Data Model

First there is an example of a _Webtable_:

![Webtable Example](https://www.usenix.org/legacy/events/osdi06/tech/chang/chang_html/img5.png) 

The row name is a reversed URL. The contents column family contains the page contents, and the anchor column family contains the text of any anchors that reference the page. CNN’s home page is referenced by both the Sports Illustrated and the MY-look home pages, so the row contains columns named anchor:cnnsi.com and anchor:my.look.ca. Each anchor cell has one version; the contents column has three versions, at timestamps t3, t5, and t6.

### Rows

* Row key is 64KB in size, but typical 10-100 bytes.
* Every read/write for a single row key is atomic.
* Data sorted in lexicographically by row key.
* Each _tablet_ contains all data associated with a row range by dynamically partition, this is the unit of distribution and load balancing.
* Reversed URL: com.cnn.www, com.google.maps/index.html, etc.

### Column Families

* Column keys are grouped into sets called column families, which form the basic unit of access control.
* A column key is named using the following syntax: `family:qualifier`.
* An example of column family is the **anchor** in the above figure.
* Access control and both disk and memory accounting are performed at the column-family level.

### Timestamps

* Each cell in a Bigtable can contain multiple versions of the same data; these versions are indexed by timestamp.
* Can be either assgined by Bigtable or by client applications.
* Garbage collection:
  * Specify the number of version to be kept
  * Specify the time period a version to be kept
  
## APIs

The Bigtable API provides functions for creating and deleting tables and column families. It also provides functions for changing cluster, table, and column family metadata, such as access control rights.

Two examples below:

![Write and Read](http://image.slidesharecdn.com/iraklispsaroudakis-key-valuestores-120306055234-phpapp01/95/bigtable-and-dynamo-8-728.jpg?cb=1331014359)

## Building Blocks

* Bigtable uses the distributed _Google File System (GFS)_ to store log and data files.
* The Google _SSTable_ file format is used internally to store Bigtable data.
  * A persistent, ordered immutable map from keys to values.
  * Contains a sequence of blocks and the block index stored at the end of SSTable.
  * Block index is loaded into memory when the SSTable is opened.
  * Optionally, an SSTable can be completely mapped into memory.
*Bigtable relies on a highly-available and persistent distributed lock service called Chubby.

## Implementation ##

A Bigtable cluster stores a number of tables. Each table consists of a set of tablets, and each tablet contains all data associated with a row range. Initially, each table consists of just one tablet. As a table grows, it is automatically split into multiple tablets.

The Bigtable implementation has three major components: a library that is linked into every client, one master server, and many tablet servers.

* Master: Responsible for assigning tablets to tablet servers, detecting the addition and expiration of tablet servers, balancing tablet-server load, and garbage collection of files in GFS.
* Tablet: Each tablet server manages a set of tablets, the tablet server handles read and write requests to the tablets that it has loaded, and also splits tablets that have grown too large.
* Client: Clients communicate directly with tablet servers for reads and writes.

### Tablet Location ###

See the below image:

![Three-level hierarchy](http://zhangjunhd.github.io/assets/2013-03-10-bigtable/2.png) 

The first level is a file stored in Chubby that contains the location of the _root tablet_. The root tablet contains the location of all tablets in a special **METADATA** table. Each **METADATA** tablet contains the location of a set of user tablets.

The **METADATA** table stores the location of a tablet under a row key that is an encoding of the tablet’s table identifier and its end row.

The client library caches tablet locations. If the client does not know the location of a tablet, or if it discovers that cached location information is incorrect, then it recursively moves up the tablet location hierarchy.




### Tablet Assignment ###

### Tablet Serving ###

### Compactions ###














