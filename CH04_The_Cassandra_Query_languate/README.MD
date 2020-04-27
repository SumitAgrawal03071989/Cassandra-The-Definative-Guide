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


## Columns

### TIMESTAMPS
Each time you write data into Cassandra, a timestamp, in microseconds, is generated for each column value that is inserted or updated. Internally, Cassandra uses these timestamps for resolving any conflicting changes that are made to the same value, in what is frequently referred to as a last write wins approach.

```
SELECT first_name, last_name, title, writetime(title) FROM user;
```

### TIME TO LIVE (TTL)
One very powerful feature that Cassandra provides is the ability to expire data that is no longer needed. This expiration is very flexible and works at the level of individual column values.


## CQL Types
CQL supports a flexible set of data types, including simple character and numeric types, collections, and user-defined types. 
### Numeric Data Types

int
A 32-bit signed integer (as in Java)

bigint
A 64-bit signed long integer (equivalent to a Java long)

smallint
A 16-bit signed integer (equivalent to a Java short)

tinyint
An 8-bit signed integer (as in Java)

varint
A variable precision signed integer (equivalent to java.math.BigInteger)

float
A 32-bit IEEE-754 floating point (as in Java)

double
A 64-bit IEEE-754 floating point (as in Java)

decimal
A variable precision decimal (equivalent to java.math.BigDecimal)

### Textual Data Types
text, varchar

### Time and Identity Data Types
timestamp
date, time

uuid
A universally unique identifier (UUID) is a 128-bit value
The uuid type is often used as a surrogate key, either by itself or in combination with other values.
Because UUIDs are of a finite length, they are not absolutely guaranteed to be unique. However, most operating systems and programming languages provide utilities to generate IDs that provide adequate uniqueness. You can also obtain a Type 4 UUID value via the CQL uuid() function and use this value in an INSERT or UPDATE.


timeuuid
This is a Type 1 UUID, which is based on the MAC address of the computer, the system time, and a sequence number used to prevent duplicates. This type is frequently used as a conflict-free timestamp. CQL provides several convenience functions for interacting with the timeuuid type: now(), dateOf(), and unixTimestampOf().
The availability of these convenience functions is one reason why timeuuid tends to be used more frequently than uuid.

## PRIMARY KEYS ARE FOREVER
After you create a table, there is no way to modify the primary key, because this controls how data is distributed within the cluster, and even more importantly, how it is stored on disk.

## Other Simple Data Types
boolean
blob ( A binary large object (blob) )

inet
This type represents IPv4 or IPv6 internet addresses.

counter
The counter data type provides a 64-bit signed integer, whose value cannot be set directly, but only incremented or decremented. Cassandra is one of the few databases that provides race-free increments across data centers. Counters are frequently used for tracking statistics such as numbers of page views, tweets, log messages, and so on. The counter type has some special restrictions. It cannot be used as part of a primary key. If a counter is used, all of the columns other than primary key columns must be counters.

```
CREATE TABLE user_visits (
  user_id uuid PRIMARY KEY, visits counter);

UPDATE user_visits SET visits = visits + 1
  WHERE user_id=ebf87fee-b372-4104-8a22-00c1252e3e05;

SELECT visits from user_visits WHERE
  user_id=ebf87fee-b372-4104-8a22-00c1252e3e05;
```

### A WARNING ABOUT IDEMPOTENCE
The counter increment and decrement operators are not idempotent. An idempotent operation is one that will produce the same result when executed multiple times. Incrementing and decrementing are not idempotent because executing them multiple times could result in different results as the stored value is increased or decreased.

To see how this is possible, consider that Cassandra is a distributed system in which interactions over a network may fail when a node fails to respond to a request indicating success or failure. A typical client response to this request is to retry the operation. The result of retrying a nonidempotent operation such as incrementing a counter is not predictable. Since it is not known whether the first attempt succeeded, the value may have been incremented twice. This is not a fatal flaw, but something you’ll want to be aware of when using counters.

The only other CQL operation that is not idempotent besides incrementing or decrementing a counter is adding an item to a list, which we’ll discuss next.

# Collections

## SET

The set data type stores a collection of elements. The elements are unordered when stored, but are returned in sorted order. 

```
ALTER TABLE user ADD emails set<text>;

UPDATE user SET emails = { 'mary@example.com' } WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

SELECT emails FROM user WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

UPDATE user SET emails = emails + {'mary.rodriguez.AZ@gmail.com' } WHERE first_name = 'Mary' AND last_name = 'Rodriguez';
```

### OTHER SET OPERATIONS

You can also clear items from the set by using the subtraction operator: SET emails = emails - {'mary@example.com'}.
Alternatively, you could clear out the entire set by using the empty set notation: SET emails = {}.


## LIST

The list data type contains an ordered list of elements.

```
ALTER TABLE user ADD phone_numbers list<text>;

UPDATE user SET phone_numbers = ['1-800-999-9999' ] WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

SELECT phone_numbers FROM user WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

UPDATE user SET phone_numbers = phone_numbers + [ '480-111-1111' ]  WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

You can replace an individual item in the list when you reference it by its index:


UPDATE user SET phone_numbers[1] = '480-111-1111' WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

```

### EXPENSIVE LIST OPERATIONS
Because a list stores values according to position, there is the potential that updating or deleting a specific item in a list could require Cassandra to read the entire list, perform the requested operation, and write out the entire list again. This could be an expensive operation if you have a large number of values in the list. For this reason, many users prefer to use the set or map types, especially in cases where there is the potential to update the contents of the collection.


## MAP
The map data type contains a collection of key-value pairs. The keys and the values can be of any type except counter

```
ALTER TABLE user ADD login_sessions map<timeuuid, int>;

UPDATE user SET login_sessions =
  { now(): 13, now(): 18}
  WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

SELECT login_sessions FROM user
WHERE first_name = 'Mary' AND last_name = 'Rodriguez';
```

## Tuples

```
ALTER TABLE user ADD address tuple<text, text, text, int>;
UPDATE user SET address = ('7712 E. Broadway', 'Tucson', 'AZ', 85715 ) WHERE first_name = 'Mary' AND last_name = 'Rodriguez';
ALTER TABLE user DROP address;
```

## User-Defined Types

Cassandra gives you a way to define your own types to extend its data model. These user-defined types (UDTs) are easier to use than tuples since you can specify the values by name rather than position.

```
CREATE TYPE address (
  street text,
  city text,
  state text,
  zip_code int);

ALTER TABLE user ADD
addresses map<text, address>;

InvalidRequest: code=2200 [Invalid query] message="Non-frozen
  collections are not allowed inside collections: map<text,
  address>"
```

What is going on here? It turns out that a user-defined data type is considered a collection, as its implementation is similar to a set, list, or map. You’ve asked Cassandra to nest one collection inside another.

### FREEZING COLLECTIONS

Freezing is a concept that was introduced as a forward compatibility mechanism. For now, you can nest a collection within another collection by marking it as frozen, which means that Cassandra will store that value as a blob of binary data. In the future, when nested collections are fully supported, there will be a mechanism to “unfreeze” the nested collections, allowing the individual attributes to be accessed.

You can also use a collection as a primary key if it is frozen.


```
ALTER TABLE user ADD addresses map<text,
  frozen<address>>;

UPDATE user SET addresses = addresses +
  {'home': { street: '7712 E. Broadway', city: 'Tucson',
  state: 'AZ', zip_code: 85715 } }
  WHERE first_name = 'Mary' AND last_name = 'Rodriguez';

  
```









