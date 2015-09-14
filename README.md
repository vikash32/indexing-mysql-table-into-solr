# indexing-mysql-table-into-solr
indexing mysql table into solr

Here we will go through step by step process.
To index mysql table into solr we require these technology.
* [Solr5.0.0](http://archive.apache.org/dist/lucene/solr/5.0.0/solr-5.0.0.zip)
* MySql Database
* [Mysql connector](http://cdn.mysql.com/archives/mysql-connector-java-5.1/mysql-connector-java-5.1.32.tar.gz)

Let's Begin the party with MySql.
Install mysql, mysql-server first, using yum or whichever your favourite method.
```
yum install mysql mysql-server
```
Now, First we will create database named test and inside this DB a table EMPLOYEE and in this table we will put first_name, last_name, age, sex, income all are in varchar for now.
```
su - #swith to root user
mysqladmin -u root -p create test #provide your root passwd when it requires.
mysql -u root -p test #provide root passwd if required.
mysql> create table EMPLOYEE(
    -> first_name varchar(30),
    -> last_name varchar(30),
    -> age varchar(6),
    -> sex varchar(1),
    -> income varchar(10)
    -> );
mysql> insert into EMPLOYEE(first_name, last_name, age, sex, income)
    -> values
    -> ('vikash', 'singh', '25', 'M', '12345');
mysql> exit;
```
Now we are done with Mysql, Let's move ahead to the solr.

Download Solr from apache [SOLR5.0.0](http://archive.apache.org/dist/lucene/solr/5.0.0/solr-5.0.0.zip)
Extracts solr5.0.0.zip folder into Downloads directory 
```
unzip solr-5.0.0.zip
```
After having solr5.0.0 and mysql connector download and extract and MySql installed. , follow mwntioned steps.

Place [mysql-connector-jar](http://cdn.mysql.com/archives/mysql-connector-java-5.1/mysql-connector-java-5.1.32.tar.gz) file into "Downloads/solr-5.0.0/contrib/dataimporthandler/lib". 
Create subdirectory "lib" if it is not present.

Now start solr.
```
cd $HOME/Downloads/solr5.0.0/ ; bin/solr start     # By default solr will start on 8983 port, http://localhost:8983/solr
```
create Core
```
bin/solr create_core -c employees -d basic_configs
```
Now open "solrconfig.xml" file, 
```
vi $HOME/Downloads/solr5.0.0/server/solr/employees/conf/solrconfig.xml
```
And add following line within config tag. here one thing to notice that in lib tag of below code the dir i nothing but the path of lib directory of jar file, i have used root so it is showing me the root/Downloads/solr5.0.0/contrib/dataimporthandler/lib, in your case it maye be different path.
``` 
<lib dir="/root/Downloads/solr-5.0.0/contrib/dataimporthandler/lib/" regex=".*\.jar" />
<lib dir="/root/Downloads/solr-5.0.0/dist/" regex="solr-dataimporthandler-\d.*\.jar" /> 
    <requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
     <lst name="defaults"> 
       <str name="config">db-data-config.xml</str> 
      </lst> 
    </requestHandler>
```
Now open schema.xml in the same directory i.e $HOME/Downloads/solr5.0.0/server/solr/employeed/schema.xml 
```
vi $HOME/Downloads/solr5.0.0/server/solr/employeed/schema.xml
```
And add following lines.here i am telling solr what to index from table of mysql.
```
<dynamicField name="*_name" type="text_general" multiValued="false" indexed="true" stored="true" />
   <dynamicField name="*age" type="int" multiValued="false" indexed="true" stored="true" />
   <dynamicField name="*sex" type="text_general" multiValued="false" indexed="true" stored="true" />
   <dynamicField name="*income" type="int" multiValued="false" indexed="true" stored="true" />
```
now create db-data-config.xml file.
```
vi db-data-config.xml
```
And now put your mysql db related info here, and from this file solr knows what to show on the search.
```
<dataConfig>
<dataSource type="JdbcDataSource"
driver="com.mysql.jdbc.Driver"
url="jdbc:mysql://localhost:3306/test"   # Connecting Database test
user="root"
password="secret" />
<document>
<entity name="id" query="select first_name as 'id' , first_name, last_name, age, sex, income from EMPLOYEE ;" />
</document>
</dataConfig>
```
Now restart the solr server.
```
cd $HOME/Downloads/solr5.0.0/; bin/solr restart
```
Now you can indexed all your table data by using example. open browser and call below URL. it will show you how many documets it has indexed. The first URL indexed all of the table data whether there is changes happened on DB or not, however 2nd URL picked only the changes which happen in DB , it's upto you which URL you can use, for learning and developing purpose got for 1st one.
```
http://localhost:8983/solr/employees/dataimport?command=full-import
http://localhost:8983/solr/employees/dataimport?command=delta-import
```

Now you can see the results as a numdoc count.also you can query on solr.
```
http://localhost:8983/solr/employees/select?q=*&wt=json&qf=first_name%20last_name&defType=edismax&indent=true
```

That's All

For any confusion on above understanding, please feel free to mail me at sincerevikash@gmail.com i would be glad to help.
