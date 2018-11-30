---
layout: post
title:  "Forget not hive"
categories: hive 
date:   2018-11-24 16:55:05 +0800
---
The motivation for this post is to put what i have learnt about Hive into practice.
And to keep a record of a workflow of useful Hive commands.

The resources I learnt Hive from are this [video series][hive-video].
And the book *Programming Hive* [book link][book-link].

Before I start with the example I would like to recall what all this Hive thingy is.

Hive is essentially a tool in the Hadoop ecosystem that is used to query data stored in HDFS.
HDFS stands for Hadoop Distributed Filesystem.
The use of Hadoop is to store large amounts of data that would be expensive and inefficient on traditional databases.
Hadoop is able to do that because the data can be stored on multiple machines. Thus the word distributed in HDFS.
Lastly Hadoop stores data in the form of files. 
To query a traditional database, we use a language called SQL.
We can query data in the HDFS using this tool called Hive that has a familiar syntax to SQL.<br>

**Hands on**<br>

**Set up HDFS and Hive**<br>
I tried downloading the hortonworks sandbox but my laptop did not have enough RAM to run it properly.
Then I found that I could try Hive by using the free trial version on Microsoft azure portal.
We have to create a hadoop cluster using HDInsight [hdinsight link][hdinsight-link]
Then we can run Hive queries on the cluster dashboard or if we have gitbash in windows, we can go to 
*HDInsight resource group > Settings > SSH + Cluster login*. And copy paste into gitbash the ssh command under *Connect to cluster using secure shell (SSH)*

Connect through ssh: ``ssh sshuser@exercisebook-ssh.azurehdinsight.net``<br>
Get the current local directory: ``pwd`` <br>
See what is in local directory: ``ls`` <br>
See what is in Hadoop directory: ``hadoop fs -ls /user ``<br>
Start Hive: ``beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'``<br>
Quit Hive: ``!quit``

**Get data**<br>
I went to this [data-link][data-link] provided by the youtube video's github, and saved it in my local as orders.csv.
Then I connected using Winscp to the azure platform using hostname as found in
*HDInsight resource group > Settings > SSH + Cluster login > Hostname*, username as sshuser and my password.
After connection, I transferred the orders.csv file to the azure platform server.
When I issue *ls* in gitbash, I now see the orders.csv file.<br>


See the first 10 records: ``head orders.csv``<br>
See number of rows in file: ``wc -l orders.csv``<br>

**Transfer data to HDFS**<br>
I said above that Hive is a tool to enable us to query files stored in HDFS.
Thus first we start off by putting the orders file into HDFS.

Make directory in HDFS: ``hadoop fs -mkdir -p /user/root/staging/financial``<br>
Move local file to HDFS: ``hadoop fs -moveFromLocal orders.csv /user/root/staging/financial``<br>
(The file will no longer exist in local) <br>
See last 1KB of file: ``hadoop fs -tail /user/root/staging/financial/orders.csv``<br>
See contents of file: ``hadoop fs -cat /user/root/staging/financial/orders.csv``<br>
Get no of rows: `` hadoop fs -cat /user/root/staging/financial/orders.csv |wc -l`` <br><br>


**Hive external table**<br>
Now that we have the data in HDFS, we can create a Hive external table.
The external part means Hive points a Hive table to that data, and will not have ownership of it.
When we drop the Hive table, the data will still remain.<br>

First we create a new database called *financials* in Hive.<br>
This will create a directory in HDFS ``/hive/warehouse/financials.db`` <br>
We can get the location of that directory from the describe database command below.

{% highlight hive %}
show databases;
create database financials;
use financials;

show tables in financials;
describe database financials;
{% endhighlight %}

If we go to that directory, we see it is empty as no table has been loaded.<br>
``hadoop fs -ls /hive/warehouse/financials.db``

Next we specify the schema of this table by issuing a create table statement.
The schema has to match the data in HDFS or else we will get nulls.
Hive is schema on read, so when we load the data, even though the schema does not match, the load will work.
However when we read/query it, we will get nulls as the schema does not match.

Now we create the external table orders_external. We have to specify the directory of the file in HDFS in *location* keyword.
Also, if there are more than 1 file in that directory, all of them have to have matching schema to the create statement in Hive.
The external table will contain data from all the files in that directory.
<br>

{% highlight hive %}
CREATE EXTERNAL TABLE orders_external (
order_id int,
order_date string,
order_customer_id int,
order_status string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/user/root/staging/financial';
{% endhighlight %}

In conclusion, we are now able to query data stored in HDFS using Hive.  We can also see the create table statement that was used to create a Hive table
using the show create table command.
{% highlight hive %}
select * from orders_external limit 5;
describe formatted financials.orders_external;
show create table financials.orders_external;
{% endhighlight %}


If we now add the same file to that HDFS directory */user/root/staging/financial*, when we select count of the Hive external table *orders_external*
, we will have twice the number of rows.<br>
This is because the Hive external table is pointing to that directory.<br>
Make copy of file: 
{% highlight hive %}
hadoop fs -cp /user/root/staging/financial/orders.csv /user/root/staging/financial/orders_copy.csv
{% endhighlight %}
If we go to the Hive warehouse directory, we see it is still empty as the Hive external table is pointing to the HDFS directory<br>
``hadoop fs -ls /hive/warehouse/financials.db``<br><br>



**Hive managed table**<br>
If the data is not in HDFS, we can also create a Hive table and load data into it directly. Or if the data is in HDFS and we want to create a Hive table 
that will have ownership of this data, we can create a Hive table. These tables are called managed tables. If we don't specify external keyword 
in the create table command, it would a be Hive managed table.
When we drop Hive managed table, both metadata and data would be gone.

I used this [data2 link][data2-link] for the data souce categories.csv.<br>
First we create the Hive managed table schema for categories.<br>
We can see in the location output of the describe command below that the table is stored in */hive/warehouse/financials.db/categories*

{% highlight hive %}
CREATE TABLE financials.categories (
category_id int,
category_type int,
category_name string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

describe formatted financials.categories;
{% endhighlight %}

If we go to ``hadoop fs -ls /hive/warehouse/financials.db/categories``, nothing is returned as no data is loaded.
Now we load data into the table categories from local. We can also load from another HDFS directory by dropping the *local* keyword. But this will move not copy the file.
(File in original HDFS directory will be gone)
 
{% highlight hive %}
LOAD DATA LOCAL INPATH '/home/sshuser/categories.csv'
into table financial.categories;
{% endhighlight %}

Now when we go to ``hadoop fs -ls /hive/warehouse/financials.db/categories``, we see a file categories.csv as data is loaded.<br>
We can issue `` hadoop fs -tail /hive/warehouse/financials.db/categories/categories.csv`` to see last 1KB records.<br>
<br>

**Hive partitioned managed table**<br>
We can create tables that are partitioned. For example we can partition a table by its column country.
Then subdirectories, each containing a country will be created. This allows for fast queries. For example, when we want to find
results related to a certain specific country, we can just scan the contents of one directory containing that country. 

First we explore how to create a partitioned managed table using the orders dataset.<br>
We need to include the keyword *PARTITIONED BY*

*Create schema of Hive table with partition*<br>
{% highlight hive %}
CREATE TABLE orders_part (
order_id int,
order_date string,
order_customer_id int,
order_status string
)
PARTITIONED BY (order_month string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
{% endhighlight %}
<br>

One way of creating a particular partition is:
{% highlight hive %}
alter table orders_part add partition(order_month='2014-02');
{% endhighlight %}

If we go to ``hadoop fs -ls /hive/warehouse/financials.db/orders_part``, we can see a subdirectory *order_month=2014-02*.
It is currently empty.
We can add data by querying the existing orders_external table.
We can use *overwrite* keyword instead of *into* to overwrite table.
{% highlight hive %}
INSERT into TABLE orders_part
partition(order_month='2014-02')
select * from  financials.orders_external where substr(order_date,1,7) ='2014-02';
{% endhighlight %}

We can also add data from local:
{% highlight hive %}
LOAD DATA LOCAL INPATH '/home/sshuser/orders_201402.csv'
into table financial.orders_part
partition(order_month='2014-02');
{% endhighlight %}
<br>

Another way is to create partitions dynamically.
This means all the distinct values of the partition column will automatically be translated into different subdirectories.
The last column in the select statement will be used as the partition column. (We used a different name order_mth to test)
{% highlight hive %}
insert overwrite table orders_part partition (order_month)
select order_id, order_date, order_customer_id, order_status,substr(order_date,1,7) order_mth from financials.orders_external; 

show partitions financials.orders_part;

{% endhighlight %}
If we go to ``hadoop fs -ls /hive/warehouse/financials.db/orders_part``, we can see multiple subdirectories *order_month=2013-07*
to *order_month=2014-07*. The partitions have been created dynamically.
When we query the table orders_part, it will behave as 1 table even though the data are in different subdirectories.
<br>
<br>

**Hive partitioned external table**<br>
When a non partitioned external table is created, location must be specified.
However it need not be when the external table is partitioned.

{% highlight hive %}
CREATE EXTERNAL TABLE orders_part (
order_id int,
order_date string,
order_customer_id int,
order_status string
)
PARTITIONED BY (order_month string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
{% endhighlight %}


To add the partition we issue:
{% highlight hive %}
ALTER TABLE orders_part ADD PARTITION(month='2014-07')
location '/user/root/staging/financial/orders/2014/07';
{% endhighlight %}<br>

For dynamic partitioning to work, we need to ensure hive.exec.dynamic.partition=true and
hive.exec.dynamic.partition.mode=nonstrict.
In conclusion, we have seen how Hive can be used to query tables in Hadoop, the differences between external and managed tables
and partitioned tables. Next we will explore how Hive can be used in data transformations, joins and how it is used in the 
data infrastructure of various companies.

[hive-video]: https://www.youtube.com/watch?v=5Hrhr1H4IQ0&list=PLf0swTFhTI8o6PhuwMO6rx4-m3CnZ7uBp&index=8
[book-link]: http://shop.oreilly.com/product/0636920023555.do
[hdinsight-link]: https://docs.microsoft.com/en-us/azure/hdinsight/hadoop/apache-hadoop-linux-create-cluster-get-started-portal
[full-azure-workflow]: https://www.youtube.com/watch?v=A3WAC4i_YFA&index=2&list=PLwAyfvNbx0t8tjs8Hr2ugFbyHbNXu3_Do&t=745s
[data-link]: https://raw.githubusercontent.com/dgadiraju/data/master/retail_db/orders/part-00000
[data2-link]: https://raw.githubusercontent.com/dgadiraju/data/master/retail_db/categories/part-00000