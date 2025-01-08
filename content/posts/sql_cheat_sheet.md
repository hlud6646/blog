+++
title = 'SQL Cheat Sheet'
date = 2024-10-01T17:52:32+10:00
draft = true
math = true
+++

This page contains a small collection of useful references relating to SQL databases.
Some are to do with administration (creating databases etc) and some to do with complicated
queries (e.g. finding the median value in a result set).

## New Project
For a new project, consider creating a new user.


### Postgresql
- Step into `psql` as the postgres user: `$ sudo -u  postgres psql`
- `# create user madonna with encrypted password bigsecret;`
- `# create database popsongs;`
- `# grant all privileges on database popsongs to madonna;`
- `# alter database popsongs owner to madonna;`

### MariaDB
- Check that mariadb is installed and running: `$ mariadb --version` and `systemctl status mariadb`
- Step into `mariadb` console as root: `$ sudo mysql -u root;`
- `# create user 'madonna'@localhost identified by 'bigsecret';`
- `# create database popsongs;`
- `# grant all privileges on popsongs.* to madonna@localhost;`
- `# flush privileges;`
- `# show grants for madonna;`


## Inspections
Some common inspections on a db server.

### Mariadb
- To see all users on a database query one of the system tables: `# select user from mysql.user;`
- To see all the databases use the simple `# show databases;`


### Misc

#### Median
This has been added to mariadb, but probably isn't available in all other dialects. 
Seems odd, since the median is a common and useful summary statistic.

Anyway suppose you have table of weather stations identified by latitude and longitude:

```sql
create or replace table station
(
    id  int auto_increment primary key,
    lat float,
    lon float
);

insert into station (lat, lon)
values (40.7128, -74.0060),
       (41.8781, -87.6298),
       (39.9523, -75.1631),
       (37.7749, -122.4194),
       (33.2951, -96.9836),
       (33.5443, 151.3069);
```
and you are looking for the median latitude.

Firstly count the rows.
```sql
select count(*) as `value`
from station;
```
'value' might seem a bad name for this quantity, but this calculation will 
be used in a subquery, and the whole subquery aliased as 'total_rows'.

Next you need to query the table for latitude, in ascending order, 
with an extra column that records the position in the sequence.
```sql
select @row_number := @row_number + 1 as `row_number`, lat
from station,
     (select @row_number := 0) as r
order by lat;
```
returns
| row\_number | lat |
| :--- | :--- |
| 1 | 33.2951 |
| 2 | 33.5443 |
| 3 | 37.7749 |
| 4 | 39.9523 |
| 5 | 40.7128 |
| 6 | 41.8781 |


If you have an even number of rows, you need the middle two. 
If you have an odd number of rows, you just need the middle one.
Both cases are captures by the predicates
$$rowNumber \geq totalRows / 2$$ and
$$rowNumber \leq totalRows/2 + 1$$.

Joining these pieces together you get the following query:
```sql
select all_rows.row_number, all_rows.lat
from (select @row_number := @row_number + 1 as `row_number`, station.*
      from station,
           (select @row_number := 0) as r
      order by lat) as all_rows,
     (select count(*) as `value`
      from station) as total_rows
where all_rows.row_number >= total_rows.value / 2
  and all_rows.row_number <= total_rows.value / 2 + 1
;
```
| row\_number | lat |
| :--- | :--- |
| 3 | 37.7749 |
| 4 | 39.9523 |

and you only need to apply `avg` in the top line to compute the actual median.
If the table had an odd number of rows, the output above would be a single line
and `avg` would leave the value unchanged.


