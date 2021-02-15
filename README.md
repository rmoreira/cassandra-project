# Cassandra with Python (and Flask) in docker-composer:

This repository shows how to stand a three-node Cassandra cluster and Flask webserver to display the data from the database.
The scripts in the init-scripts folder creates a keyspace called "sampledata", sets replication to 2, and initializes the database with some data, in this case, the index number and name of the months.

## Video
if you want to follow along by wachting the video below:

[![demo video](http://img.youtube.com/vi/50wx7WEZB4U/0.jpg)](https://www.youtube.com/watch?v=50wx7WEZB4U "Cassandra with Python (and Flask) in docker-composer")

## Steps:

* ```docker-composer build```
    * This step will take a few minutes to build.


* ```docker-compose up```
    * This step will take a few seconds to come up because the Cassandra cluster has to initialize.

## Verify that the webapp is up:

* Go to <http://localhost:5555/>

* The output should look something like this:
```
{
  "1": "January",
  "2": "February",
  "3": "March",
  "4": "April",
  "5": "May",
  "6": "June",
  "7": "July",
  "8": "August",
  "9": "September",
  "10": "October",
  "11": "November",
  "12": "December"
}
```

## Verify that replication is working:

* Open another terminal window.
* See the running containers:
```
docker-compose ps
```
Output:
```
               Name                              Command               State           Ports
-----------------------------------------------------------------------------------------------------
cassandra-project_cassandra-node1_1   /opt/bitnami/scripts/cassa ...   Up      7000/tcp, 9042/tcp
cassandra-project_cassandra-node2_1   /opt/bitnami/scripts/cassa ...   Up      7000/tcp, 9042/tcp
cassandra-project_cassandra-node3_1   /opt/bitnami/scripts/cassa ...   Up      7000/tcp, 9042/tcp
cassandra-project_web-app_1           python /app/app.py               Up      0.0.0.0:5555->5555/tcp

```

Stop one container:
```
docker stop cassandra-project_cassandra-node3_1
```
Once it stops, go to to <http://localhost:5555/> again.
With [replication](./init-scripts/0-create-keyspace.cql) set to 2, Cassandra's QUORUM allows the database to successfully return the request since all nodes have two thirds of the data.

Stop another container:
```
docker stop cassandra-project_cassandra-node3_1
```
And go to to <http://localhost:5555/> again.  The application will thrown an exception.

## Increasing replication from 2 to 3:
By increasing replication from 2 to 3, reads will be faster, but writes will be slower.
Press Control+C to stop the running cluster, and clean up the existing containers:
```
docker-compose rm -f
```
Then start the cluster again:
```
docker-compose up
```
Wait a few seconds and go to <http://localhost:5555/> again.
You should get a successful response.

Now try to stop 2 Cassandra containers:
```
docker stop cassandra-project_cassandra-node2_1
docker stop cassandra-project_cassandra-node3_1
```
Wait a few seconds and go to <http://localhost:5555/> again.
You should get a successful response because replication was set to 3, and all nodes have all the data.


## References:
* https://github.com/bitnami/bitnami-docker-cassandra
* https://towardsdatascience.com/getting-started-with-apache-cassandra-and-python-81e00ccf17c9
* http://blog.adnansiddiqi.me/getting-started-with-apache-cassandra-and-python/
* https://github.com/kadnan/PythonCassandraTutorial
* http://beadooper.com/?p=182
* https://myadventuresincoding.wordpress.com/2019/01/29/cassandra-switching-from-simplestrategy-to-networktopologystrategy/
