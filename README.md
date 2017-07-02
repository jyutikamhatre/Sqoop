# Sqoop
Proof of concept - Hadoop - SQOOP




Project Description
Import/Exporting retail data to and/or from HDFS, Hive
Dataset Resource
Created own tables
Special operation
1. Import, Import-all-tables,Exclude-tables
2. Export tables
3. using target and warehouse dir options
4. specifying number of mappers
5. using eval insert data in the table
6. use incremental option appending data at last value
7. specifying null for string and non-string values
8. use fields and lines terminators
9. import to hive
Goal
Import/Exporting retail data to and/or from HDFS, Hive
MSSQL and Hive databases
Database : retail
tables : customers, order_products, product_stock, products
Input files
Customers
product_stock
Output file
/output/customers_sqoop

mysql> create table customers( id int not null,name varchar(30) not null,address varchar(30) not null, tel varchar(14) not null,email varchar(30) not null);
Query OK, 0 rows affected (0.06 sec)

mysql> describe customers;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| id      | int(11)     | NO   |     | NULL    |       |
| name    | varchar(30) | NO   |     | NULL    |       |
| address | varchar(30) | NO   |     | NULL    |       |
| tel     | varchar(14) | NO   |     | NULL    |       |
| email   | varchar(30) | NO   |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
5 rows in set (0.00 sec)

mysql> create table products(id int,name varchar(20),price double);
Query OK, 0 rows affected (0.00 sec)

mysql> describe products;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| price | double      | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.01 sec)

mysql> create table product_stock(product_id int not null,instock bigint not null);
Query OK, 0 rows affected (0.06 sec)

mysql> describe product_stock;
+------------+------------+------+-----+---------+-------+
| Field      | Type       | Null | Key | Default | Extra |
+------------+------------+------+-----+---------+-------+
| product_id | int(11)    | NO   |     | NULL    |       |
| instock    | bigint(20) | NO   |     | NULL    |       |
+------------+------------+------+-----+---------+-------+
2 rows in set (0.00 sec)


mysql> create table order_products(order_id int not null, product_id int not null, product_quantity int not null,customer_id int not null,datetime date not null);
Query OK, 0 rows affected (0.06 sec)

mysql> describe order_products;
+------------------+---------+------+-----+---------+-------+
| Field            | Type    | Null | Key | Default | Extra |
+------------------+---------+------+-----+---------+-------+
| order_id         | int(11) | NO   |     | NULL    |       |
| product_id       | int(11) | NO   |     | NULL    |       |
| product_quantity | int(11) | NO   |     | NULL    |       |
| customer_id      | int(11) | NO   |     | NULL    |       |
| datetime         | date    | NO   |     | NULL    |       |
+------------------+---------+------+-----+---------+-------+
5 rows in set (0.01 sec)

mysql> show tables;
+------------------+
| Tables_in_retail |
+------------------+
| customers        |
| order_products   |
| product_stock    |
| products         |
+------------------+
4 rows in set (0.00 sec)

Created locally and transferred customers file to HDFS which contains below data:

100,sinom corporation,malad west,9876543210,info@abc.com
200,micra inc,kandiwali east,9876543210,info@micra.com
300,aron solutions,dadar west,9876543210,info@aron.com
400,vio informatics,juhu,9876543210,info@vio.com

notroot@ubuntu:~/lab/data$ hdfs dfs -put customers /input/customers

notroot@ubuntu:~/lab/data$ sqoop export --connect jdbc:mysql://localhost/retail --driver com.mysql.jdbc.Driver --table customers --username root --password admin --export-dir /input/customers


mysql> select * from customers;
+-----+-------------------+----------------+------------+----------------+
| id  | name              | address        | tel        | email          |
+-----+-------------------+----------------+------------+----------------+
| 100 | sinom corporation | malad west     | 9876543210 | info@abc.com   |
| 400 | vio informatics   | juhu           | 9876543210 | info@vio.com   |
| 200 | micra inc         | kandiwali east | 9876543210 | info@micra.com |
| 300 | aron solutions    | dadar west     | 9876543210 | info@aron.com  |
+-----+-------------------+----------------+------------+----------------+
4 rows in set (0.00 sec)

Import table content to HDFS file /output/customers_sqoop:

notroot@ubuntu:~/lab/data$ sqoop import --connect jdbc:mysql://localhost/retail --username root --password admin --driver com.mysql.jdbc.Driver --table customers --target-dir /output/customers_sqoop --m 1

notroot@ubuntu:~/lab/data$ hdfs dfs -ls /output/customers_sqoop
Found 2 items
-rw-r--r--   1 notroot supergroup          0 2017-06-26 05:28 /output/customers_sqoop/_SUCCESS
-rw-r--r--   1 notroot supergroup        216 2017-06-26 05:28 /output/customers_sqoop/part-m-00000
notroot@ubuntu:~/lab/data$ hdfs dfs -cat /output/customers_sqoop/part*
100,sinom corporation,malad west,9876543210,info@abc.com
400,vio informatics,juhu,9876543210,info@vio.com
200,micra inc,kandiwali east,9876543210,info@micra.com
300,aron solutions,dadar west,9876543210,info@aron.com

Insert data to MYSQL table manually:

mysql>insert into products('111111','mouse','2000');
Query OK, 1 row affected (0.01 sec)

mysql>insert into products('222222','keyboard','1000');
Query OK, 1 row affected (0.01 sec)

mysql>insert into products('333333','monitor','5000');
Query OK, 1 row affected (0.01 sec)

mysql>insert into products('444444','CPU','25000');
Query OK, 1 row affected (0.01 sec)

mysql> use retail;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from products;
+--------+----------+-------+
| id     | name     | price |
+--------+----------+-------+
| 111111 | mouse    |  2000 |
| 222222 | keyboard |  1000 |
| 333333 | monitor  |  5000 |
| 444444 | CPU      | 25000 |
+--------+----------+-------+
4 rows in set (0.00 sec)

Inserted below data to MYSQL table manually:

mysql> select * from order_products;
+----------+------------+------------------+-------------+------------+
| order_id | product_id | product_quantity | customer_id | datetime   |
+----------+------------+------------------+-------------+------------+
|        1 |     111111 |               10 |         100 | 2017-03-01 |
|        2 |     222222 |               10 |         100 | 2017-04-01 |
|        3 |     333333 |               20 |         200 | 2017-05-01 |
|        4 |     444444 |               10 |         300 | 2017-06-01 |
|+----------+------------+------------------+-------------+------------+
6 rows in set (0.01 sec)

Created file locally product_stock with comma separated values and transferred to HDFS:

notroot@ubuntu:~/lab/data$ hdfs dfs -put product_stock /input/product_stock

Fill MYSQL Table with export command:

notroot@ubuntu:~/lab/data$ sqoop export --connect jdbc:mysql://localhost/retail --driver com.mysql.jdbc.Driver --table product_stock --username root --password admin --export-dir /input/product_stock --m 1

mysql> select * from product_stock;
+------------+---------+
| product_id | instock |
+------------+---------+
|     111111 |     300 |
|     222222 |     400 |
|     333333 |     600 |
|     444444 |     100 |
+------------+---------+
4 rows in set (0.00 sec)

Import-all-tables by excluding few tables:

notroot@ubuntu:~/lab/data$ sqoop import-all-tables --connect jdbc:mysql://localhost/retail --username root --password admin --driver com.mysql.jdbc.Driver --exclude-tables prod                     uct_stock,products --warehouse-dir fewtables --m 1


Display and Insert data to MYSQL table using eval command:

notroot@ubuntu:~/lab/data$ sqoop eval --connect jdbc:mysql://localhost/retail --driver com.mysql.jdbc.Driver --username root --password admin --query "select * from order_products"
Warning: /home/notroot/lab/software/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/notroot/lab/software/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
Warning: /home/notroot/lab/software/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/../zookeeper does not exist! Accumulo imports will fail.
Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
17/06/29 14:59:13 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
17/06/29 14:59:13 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
17/06/29 14:59:13 WARN sqoop.ConnFactory: Parameter --driver is set to an explicit driver however appropriate connection manager is not being set (via --connection-manager). Sqoop is going to fall back to org.apache.sqoop.manager.GenericJdbcManager. Please specify explicitly which connection manager should be used next time.
17/06/29 14:59:13 INFO manager.SqlManager: Using default fetchSize of 1000
----------------------------------------------------------------------
| order_id    | product_id  | product_quantity | customer_id | datetime   |
----------------------------------------------------------------------
| 1           | 111111      | 10          | 100         | 2017-03-01 |
| 2           | 222222      | 10          | 100         | 2017-04-01 |
| 3           | 333333      | 20          | 200         | 2017-05-01 |
| 4           | 444444      | 10          | 300         | 2017-06-01 |

Insert 2 records by repeating below command( content different ):

notroot@ubuntu:~/lab/data$ sqoop eval --connect jdbc:mysql://localhost/retail --driver com.mysql.jdbc.Driver --username root --password admin -e "insert into order_products values(5,333333,10,300,'2017-06-25')"
Warning: /home/notroot/lab/software/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/notroot/lab/software/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
Warning: /home/notroot/lab/software/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/../zookeeper does not exist! Accumulo imports will fail.
Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
17/06/29 15:04:49 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
17/06/29 15:04:49 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
17/06/29 15:04:49 WARN sqoop.ConnFactory: Parameter --driver is set to an explicit driver however appropriate connection manager is not being set (via --connection-manager). Sqoop is going to fall back to org.apache.sqoop.manager.GenericJdbcManager. Please specify explicitly which connection manager should be used next time.
17/06/29 15:04:49 INFO manager.SqlManager: Using default fetchSize of 1000
17/06/29 15:04:50 INFO tool.EvalSqlTool: 1 row(s) updated.
notroot@ubuntu:~/lab/data$


----------------------------------------------------------------------
| order_id    | product_id  | product_quantity | customer_id | datetime   |
----------------------------------------------------------------------
| 1           | 111111      | 10          | 100         | 2017-03-01 |
| 2           | 222222      | 10          | 100         | 2017-04-01 |
| 3           | 333333      | 20          | 200         | 2017-05-01 |
| 4           | 444444      | 10          | 300         | 2017-06-01 |
| 5           | 333333      | 10          | 300         | 2017-06-25 |
| 6           | 333333      | 10          | 300         | 2017-06-26 |

Use incremental option for appending the data:

notroot@ubuntu:~/lab/data$ sqoop import --connect jdbc:mysql://localhost/retail --driver com.mysql.jdbc.Driver --username root --password admin --table order_products --warehouse-dir fewtables --m 1 --incremental append --check-column order_id --last-value 4

.......................
                Bytes Written=54
17/06/29 15:13:25 INFO mapreduce.ImportJobBase: Transferred 54 bytes in 23.4747 seconds (2.3004 bytes/sec)
17/06/29 15:13:25 INFO mapreduce.ImportJobBase: Retrieved 2 records.
17/06/29 15:13:25 INFO util.AppendUtils: Appending to directory order_products
17/06/29 15:13:25 INFO util.AppendUtils: Using found partition 1
17/06/29 15:13:25 INFO tool.ImportTool: Incremental import complete! To run another incremental import of all data following this import, supply the following arguments:
17/06/29 15:13:25 INFO tool.ImportTool:  --incremental append
17/06/29 15:13:25 INFO tool.ImportTool:   --check-column order_id
17/06/29 15:13:25 INFO tool.ImportTool:   --last-value 6
17/06/29 15:13:25 INFO tool.ImportTool: (Consider saving this with 'sqoop job –create')



Import all tables to hive directory:

notroot@ubuntu:~/lab/data$ sqoop import-all-tables --connect jdbc:mysql://localhost/retail --driver com.mysql.jdbc.Driver --username root --password admin --warehouse-dir /user/hive/warehouse/newretail.db --fields-terminated-by ',' --lines-terminated-by '\n' --null-string 'NA' --null-non-string 'NA' --m 1


Import using hive-import option:

notroot@ubuntu:~/lab/data$ sqoop import --connect jdbc:mysql://localhost/retail --table customers --driver com.mysql.jdbc.Driver --username root --password admin --hive-import --m 1



