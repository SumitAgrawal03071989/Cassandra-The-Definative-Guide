# Chapter 4. The Cassandra Query Language

## Cassandra’s Data Model
* Cassandra defines a table to be a logical division that associates similar data. For example, you might have a user table, a hotel table, an address book table, and so on. In this way, a Cassandra table is analogous to a table in the relational world.
* Cassandra uses a special type of primary key called a composite key (or compound key) to represent groups of related rows, also called partitions. The composite key consists of a partition key, plus an optional set of clustering columns.
* The partition key is used to determine the nodes on which rows are stored and can itself consist of multiple columns. The clustering columns are used to control how data is sorted for storage within a partition. Cassandra also supports an additional construct called a static column, which is for storing data that is not part of the primary key but is shared by every row in a partition.

## Clusters
* Cassandra database is specifically designed to be distributed over several machines operating together that appear as a single instance to the end user. So the outermost structure in Cassandra is the cluster, sometimes called the ring, because Cassandra assigns data to nodes in the cluster by arranging them in a ring.

## Keyspaces
A cluster is a container for keyspaces. A keyspace is the outermost container for data in Cassandra, corresponding closely to a database in the relational model.

## Tables
A table is a container for an ordered collection of rows, each of which is itself an ordered collection of columns. Rows are organized in partitions and assigned to nodes in a Cassandra cluster according to the column(s) designated as the partition key. The ordering of data within a partition is determined by the clustering columns.

```
ALTER TABLE user ADD middle_initial text;
DESCRIBE TABLE user;


INSERT INTO user (first_name, middle_initial, last_name,title) VALUES ('Bill', 'S', 'Nguyen', 'Mr.');
INSERT INTO user (first_name, middle_initial, last_name,title) VALUES ('Bill', 'R', 'Nguyen', 'Mr.');

SELECT * FROM user WHERE first_name='Bill' AND last_name='Nguyen';
```

## UPSERT
Because Cassandra uses an append model, there is no fundamental difference between the insert and update operations. If you insert a row that has the same primary key as an existing row, the row is replaced. If you update a row and the primary key does not exist, Cassandra creates it.
For this reason, it is often said that Cassandra supports upsert, meaning that inserts and updates are treated the same,


