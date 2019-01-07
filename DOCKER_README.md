# See main README.md for background

This README covers running this as part of a docker image

## Overview
In this tutorial, the StreamSets Data Collector is used to create streaming data pipelines to write to/from Kafka and ultimately landing in DSE.  The high-level architecture is shown in the following diagram:

![StreamsConsumer](README.photos/StreamSetsBoth.png)

## Requirements

1. Docker engine (Free CE version works fine https://www.docker.com/community-edition)
## Getting Started
1. Pull this github repo into `datastaxStreamsetsDocker` directory  
    ```bash
    git clone https://github.com/jphaugla/datastaxStreamsetsDocker.git
    ```
2. `cd datastaxStreamsetsDocker`
3. Start docker containers with `docker-compose up -d`
4. Wait 1 minute for containers to startup
5. Verify DataStax is up-and-running (again, see previous step, this may take a minute or two for Docker images):
```bash
docker exec dse cqlsh dse -u cassandra -p cassandra -e "desc keyspaces";
```
6. Create keyspace and table in DSE:
```bash
docker cp schema/cassandra_schema.cql dse:/opt/dse;
docker exec dse cqlsh dse -f /opt/dse/cassandra_schema.cql
```
7. Verify table exists:
```bash
docker exec dse cqlsh dse -e "desc killrmovies.movies"
```

## Set up mysql

1. create mysql streamsets username
```bash
docker cp schema/create_mysql_un.sql mysql:/;
docker exec mysql  mysql -u root --password="mysql" -e "source create_mysql_un.sql"
```
2. create killrmovies mysql schema
```bash
docker cp schema/killrmovies_mysql_schema.sql mysql:/;
docker exec mysql  mysql -u ss --password="ss" -e "source killrmovies_mysql_schema.sql"
```
2. load mysql data
```bash
docker cp data mysql:/moviedata;
docker exec mysql  mysql killrmovies -u ss --password="ss" -e "source /moviedata/movies.sql"
docker exec mysql  mysql killrmovies -u ss --password="ss" -e "source /moviedata/ratings.sql"
docker exec mysql  mysql killrmovies -u ss --password="ss" -e "source /moviedata/tags.sql"
docker exec mysql  mysql killrmovies -u ss --password="ss" -e "source /moviedata/users.sql"
docker exec mysql  mysql killrmovies -u ss --password="ss" -e "source /moviedata/_movie_uuid.sql"
docker exec mysql  mysql killrmovies -u ss --password="ss" -e "source /moviedata/_user_uuid.sql"
```

## Add Additional Libraries to Stage Libraries section

The package manager makes stage libraries available to pipelines.  The following three stage libraries need to be added for the subsequent pipelines.

1. Login to StreamSets Data Collector at http://localhost:18630 with username: admin password: admin
2. Click on *Package Manager* icon ![Package Manager](README.photos/icon_PackageManager.png)
3. Select Cassandra Java Driver
4. click the ellipsis buttons and select install
5. On Install Stage Library, click cancel
6. Using the same technique, ensure the following packages are installed:  JDBC and finally, MySQL BinLog
7. restart streamsets container to make the drivers usable 
```bash
docker restart streamdc
```


## Running and Creating StreamSets Pipelines

Import pre-configured pipelines and run.  

The steps shown in the above screencast (starting at "Get Started" blue screen)

1. Login to StreamSets Data Collector at http://localhost:18630 with username: admin password: admin
2. Import the `exports/RDBMS_CDC_to_Cassandra.json` pipeline and start
3. Import the `exports/RDBMS_to_Cassandra_Bulk_Load.json` pipeline and start
4. Verify data has landed in DSE from terminal with `docker exec dse cqlsh dse -e "select * from avro.cctest limit 10"`

That's it.  Easy button.

Note: if you start/stop pipelines in the future.  You will likely need to "Reset Origin" of both pipelines.  Please consult StreamSets documentation for more information around resetting pipeline origins.


#### Preview the Data
* Feel free to hit the Preview icon to examine the data before executing the pipeline.

#### Execute the Pipeline
* Hit the Start icon.  Watching the metrics, 10,000 rows should go to the producer.  Stop the pipeline using the red stop rectangle.  To rerun at a later time, make sure to **reset origin** (under the ellipsis) to restart the load from the beginning.


## Further Reference

* DataStax Docker Images
[https://github.com/datastax/docker-images/#datastax-platform-overview](https://github.com/datastax/docker-images/#datastax-platform-overview).

* [StreamSets Data Collector](https://streamsets.com/)

* [StreamSets Docker Images](https://github.com/streamsets/datacollector-docker.git)

* Test data came from a StreamSets tutorial github but I have copied the data into this github.
 [https://github.com/streamsets/tutorials/blob/master/sample_data/](https://github.com/streamsets/tutorials/blob/master/sample_data/)
