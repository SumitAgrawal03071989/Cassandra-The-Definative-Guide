# Chapter 3. Installing Cassandra

## Starting the Server

Go to bin directory of cassandra.

```
cassandra -f
```

## Start CQLSH : Cassandra client.

Go to bin directory of cassandra.

```
cqlsh
```
```
CREATE TABLE user ( first_name text , last_name text, title text, PRIMARY KEY (last_name, first_name)) ;
DESCRIBE TABLE user;

INSERT INTO user (first_name, last_name, title) VALUES ('Bill', 'Nguyen', 'Mr.');
```


