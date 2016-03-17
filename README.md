# CassandraPrimaryKeys
This exercise is for anyone new to Cassandra and is confused by the terms Primary Key, Partition Key, Compound Keys, and Clustering Columns

This exercise assumes that you have at least one Cassandra or DSE node up and running. You may need to replace *127.0.0.1* with whatever your node's IP Address is.

Lets start by creating a keyspace 

```
cqlsh 127.0.0.1

CREATE KEYSPACE pkfun WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1 };

USE pkfun ;
```

The primary key is a general concept to indicate one or more columns used to retrieve data from a Table.


The primary key may be SIMPLE meaning that a single column makes up the primary or "Partition" key

```
CREATE TABLE pkfun.starwars (
    date int,
    name text,
    episode text,
    na_gross double,
    non_na_gross double,
    worldwide_gross double,
    inflammation_adj double,
    imdb_rating int, 
    rotten_rating int,
    PRIMARY KEY (date)
);
```
Let's insert some data

```
INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1977, 'A New Hope', 4, 460998007, 314400000, 775398007, 131485043, 8.7, 94);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1980,'The Empire Strikes Back',5,290475067, 247900000 , 538375067 ,751204635,8.8,94);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1983, 'Return of the Jedi',6,309306177, 165800000 , 475106177 ,724064338,8.4,80);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1999,'The Phantom Menace',1,474544677, 552500000 , 1027044677 ,699066761,6.5,56);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (2002,'Attack of the Clones',2,310676740, 338721588 , 649398328 ,414858818,6.7,66);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (2005,'Revenge of the Sith',3,380270577, 468484191 , 848754768 ,460743580,7.7,79);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, rotten_rating) VALUES (2015,'The Force Awakens',7,931067821,1122000000,2053067821,931067821,94);
```





Built from a simple explanation put together by Carlo Bertuccini
http://stackoverflow.com/questions/24949676/difference-between-partition-key-composite-key-and-clustering-key-in-cassandra
