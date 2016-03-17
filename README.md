# Cassandra Keys and Clustering Columns
This exercise is for anyone new to Cassandra and is confused by the terms Primary Key, Partition Key, Compound Keys, and Clustering Columns

Definitions: 
>The **Partition Key** is responsible for data distribution accross your nodes.

>The **Clustering Key** is responsible for data sorting within the partition.

>The **Primary Key** is equivalent to the Partition Key in a single-field-key table.

>The **Composite/Compund Key** is just a multiple-columns Primary key

Why it matters: We think of Cassandra nodes as being organized in a ring. Each node in the ring is responsible for a portion of the data. But how do we know which data goes on which node? It's simple. Each node is responsible for a range of tokens. When you input data you designate the primary/partition key. Cassandra hashes that key. The hash or "token" tells us the physical address of the data. We send the data to the node responsible for that token (address) range. 

Got it? Good! So it follows that to get that data back, you MUST know the phsycial address or Token of that data. You must query with the Key. The system hashes the key and retrieves the data with that token. BAM

I'm going to provide you the format here and you can refer back to it through the exercise as you learn about each piece:
```
PRIMARY KEY        (Key1, cluster1, cluster2) 
-------------------------+-------------------+
             Partion Key | Clustering Columns  


PRIMARY KEY        ((Key1, Key2) cluster1, cluster2) 
--------------------------------+-------------------+
 		            Partion Key | Clustering Columns  
----------------------------------------------------+
                   Compound/Composite Key
```


Time to dig in!

This exercise assumes that you have at least one Cassandra or DSE node up and running. You may need to replace **127.0.0.1** with whatever your node's IP Address is.

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
    episode int,
    na_gross double,
    non_na_gross double,
    worldwide_gross double,
    inflammation_adj double,
    imdb_rating double, 
    rotten_rating int,
    PRIMARY KEY (date)
);
```
Let's insert some data

```
INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1977, 'A New Hope', 4, 460998007, 314400000, 775398007, 131485043, 8.7, 94);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1980,'The Empire Strikes Back',5,290475067, 247900000 , 538375067 ,751204635,8.8,94);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1983, 'Return of the Jedi',6,309306177, 165800000 , 475106177 ,724064338,8.4,80);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (2002,'Attack of the Clones',2,310676740, 338721588 , 649398328 ,414858818,6.7,66);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (2005,'Revenge of the Sith',3,380270577, 468484191 , 848754768 ,460743580,7.7,79);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, rotten_rating) VALUES (2015,'The Force Awakens',7,931067821,1122000000,2053067821,931067821,94);
```

Episode 1 was intentionally left out of this dataset because it makes sense to pretend that it does not exist. If you strongly disagree you can take this insert statement and shove it (into your table)

```
INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1999,'The Phantom Menace',1,474544677, 552500000 , 1027044677 ,699066761,6.5,56);
```

With a Simple Primary/Partition Key you MUST specify the primary key column to query. Let's test it:

```
SELECT * FROM starwars WHERE date=1977;
```

you should see something like 
```
 date | episode | imdb_rating | inflammation_adj | na_gross | name       | non_na_gross | rotten_rating | worldwide_gross
------+---------+-------------+------------------+----------+------------+--------------+---------------+-----------------
 1977 |       4 |         8.7 |       1.3149e+08 | 4.61e+08 | A New Hope |    3.144e+08 |            94 |       7.754e+08

```

Now we have to admit that a simple Star Wars table isn't the best fit for this exercise, however, I thought it would be fun so lets go with it. Lets pretend that we didn't have to wait an average of 6.333333333333333333333333333333 years for each film. What if Episode 5 came out in 1977 as well? 

Let's see what happens if you insert a second film into 1977

```
INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1977,'The Empire Strikes Back',5,290475067, 247900000 , 538375067 ,751204635,8.8,94);
```

Now, we will query using 1977 and see what happens

```
SELECT * FROM starwars WHERE date=1977;
```

And you see something like this

```
 date | episode | imdb_rating | inflammation_adj | na_gross   | name                    | non_na_gross | rotten_rating | worldwide_gross
------+---------+-------------+------------------+------------+-------------------------+--------------+---------------+-----------------
 1977 |       5 |         8.8 |        7.512e+08 | 2.9048e+08 | The Empire Strikes Back |    2.479e+08 |            94 |      5.3838e+08
```

What happened? Your Primary Key wasn't unique and you overwrote the best film in the entire series. How could you! 

How could we designate a more unique primary key such that this wouldn't happen?  It makes sense that date + episode would provide a unique key in all scenarios. Thus, we're going to have TWO things in our primary key and you must specify BOTH to get any piece of data.

```
drop table pkfun.starwars;

CREATE TABLE pkfun.starwars (
    date int,
    name text,
    episode int,
    na_gross double,
    non_na_gross double,
    worldwide_gross double,
    inflammation_adj double,
    imdb_rating double, 
    rotten_rating int,
    PRIMARY KEY ((date,episode))
);
```
Why the double parentheses?  We're going to talk about clustering columns soon. But for now lets just go with it. If you want two things in your partition key... if you want Cassandra to has TWO columns to output a token, it MUST be within a set of parentheses within your primary key statement.

We're going to move on to a more complex example soon. So lets keep pretending that the Empire struck back in 1977. To do that we need to insert some data. 

```
INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1977, 'A New Hope', 4, 460998007, 314400000, 775398007, 131485043, 8.7, 94);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1977,'The Empire Strikes Back',5,290475067, 247900000 , 538375067 ,751204635,8.8,94);
```

What would happen if you just specified 1977 in your query? Let's see

```
SELECT * FROM starwars WHERE date=1977;
```
The system is going to tell you, that's right, it's smart enough to tell you that you MUST specify the episode as well. Remember how hashes work. They are guaranteed to be unique such that any string of any length outputs a unique number of a fixed length. Secondly, hash algorithms guarantee that for the same input you will always get the same outputted hash (token). Thus you need both pieces of the partition key to find the correct token. Otherwise you get something like this:

```
InvalidRequest: code=2200 [Invalid query] message="Partition key part episode must be restricted since preceding part is"
```
Let's give the system what it wants:

```
SELECT * FROM starwars WHERE date=1977 AND episode=5;
```

And you will get a specific answer back

```
 date | episode | imdb_rating | inflammation_adj | na_gross   | name                    | non_na_gross | rotten_rating | worldwide_gross
------+---------+-------------+------------------+------------+-------------------------+--------------+---------------+-----------------
 1977 |       5 |         8.8 |        7.512e+08 | 2.9048e+08 | The Empire Strikes Back |    2.479e+08 |            94 |      5.3838e+08
```

We all agree that this works. But what your application really needs is to get all data back for 1977. Right now you can't do that. 

Now it's time to talk about **clustering columns** and **wide rows**. 

Specifying a **clustering column** gives you an optional queryable field as well as something you can sort and filter on.

Time to try it out:
```
drop table starwars ;

CREATE TABLE pkfun.starwars (
    date int,
    name text,
    episode int,
    na_gross double,
    non_na_gross double,
    worldwide_gross double,
    inflammation_adj double,
    imdb_rating double, 
    rotten_rating int,
    PRIMARY KEY (date,episode)
);
```
And we'll need some data: 

```
INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1977, 'A New Hope', 4, 460998007, 314400000, 775398007, 131485043, 8.7, 94);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1977,'The Empire Strikes Back',5,290475067, 247900000 , 538375067 ,751204635,8.8,94);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (1983, 'Return of the Jedi',6,309306177, 165800000 , 475106177 ,724064338,8.4,80);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (2002,'Attack of the Clones',2,310676740, 338721588 , 649398328 ,414858818,6.7,66);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, imdb_rating, rotten_rating) VALUES (2005,'Revenge of the Sith',3,380270577, 468484191 , 848754768 ,460743580,7.7,79);

INSERT INTO pkfun.starwars (date, name, episode, na_gross, non_na_gross, worldwide_gross, inflammation_adj, rotten_rating) VALUES (2015,'The Force Awakens',7,931067821,1122000000,2053067821,931067821,94);
```

Now we should be able to grab everything from 1977: 
```
SELECT * FROM starwars WHERE date=1977;
```
Aw Yah: 

```
 date | episode | imdb_rating | inflammation_adj | na_gross   | name                    | non_na_gross | rotten_rating | worldwide_gross
------+---------+-------------+------------------+------------+-------------------------+--------------+---------------+-----------------
 1977 |       4 |         8.7 |       1.3149e+08 |   4.61e+08 |              A New Hope |    3.144e+08 |            94 |       7.754e+08
 1977 |       5 |         8.8 |        7.512e+08 | 2.9048e+08 | The Empire Strikes Back |    2.479e+08 |            94 |      5.3838e+08
```

That's cool, but it's hard to think about how that is actually stored on disk.
![Data on Disk](http://i.imgur.com/gMxR31f.png)

In our last example we realize that we actually do want to get just the first film from 1977. Thank goodness we specified a clustering column. Remeber that you MUST specify the Partition/Primary key but you CAN specify the clustering column.

```
SELECT * FROM starwars WHERE date=1977 AND episode=4;
```

Yields

```
 date | episode | imdb_rating | inflammation_adj | na_gross | name       | non_na_gross | rotten_rating | worldwide_gross
------+---------+-------------+------------------+----------+------------+--------------+---------------+-----------------
 1977 |       4 |         8.7 |       1.3149e+08 | 4.61e+08 | A New Hope |    3.144e+08 |            94 |       7.754e+08
```

There may be situations, however, where you have a lot of data. Think about a sensor in which your sensor is the primary key and it's readings are stored as wide rows. All your looking for is a range of readings from that specific sensor: 

```
SELECT * FROM starwars WHERE date=1977 and episode > 4;
```

Yelds

```
 date | episode | imdb_rating | inflammation_adj | na_gross   | name                    | non_na_gross | rotten_rating | worldwide_gross
------+---------+-------------+------------------+------------+-------------------------+--------------+---------------+-----------------
 1977 |       5 |         8.8 |        7.512e+08 | 2.9048e+08 | The Empire Strikes Back |    2.479e+08 |            94 |      5.3838e+08
```
That's it. You now know enough to be dangerous.

If you found this interesting, you should try out a more real world use case: https://academy.datastax.com/demos/getting-started-time-series-data-modeling

If you want to learn a lot more about data modeling with Cassandra you should do this free course: https://academy.datastax.com/courses/ds220-data-modeling

If you want the most useful Cassandra Modeling resource on the internet you should look here: http://www.sestevez.com/sestevez/CassandraDataModeler/

If you think you've found your calling, you should reach out to me. We're always looking for new talent kathryn.erickson@datastax.com

Built from a simple explanation put together by Carlo Bertuccini
http://stackoverflow.com/questions/24949676/difference-between-partition-key-composite-key-and-clustering-key-in-cassandra
