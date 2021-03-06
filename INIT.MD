How to initialize the docker containers:

Make sure to create all of the folders that will be used by the volumes first. Here, we are using `/splash/data/mongo` and `/splash/data/elastic`

Create a .env file and add the following (replacing with your folder names)

```
MONGO_DB_LOC=/splash/data/mongo

ELASTIC_DATA_LOC=/splash/data/elastic
```

In your .env file create the environment variable 
`MONSTACHE_NAMESPACE_REGEX=^db_name\.run_start` where `db_name` is the name of the database with the run start documents. This will ensure that Monstache only copies over the `run_start` collection from the appropriate database.

Then in the `init/python/config.py` change the `“index_name”` property of the `elastic` dict to `“db_name.run_start”`

Change the `“alias”` property to what you would like to refer to the index using the REST endpoint of elastic. 


On Linux hosts run  `sysctl -w vm.max_map_count=262144`  and then `sysctl --system`

See: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-prod-cluster-composefile

Change your working directory to the top level directory

These commands initialize the replica set for mongo:

`docker-compose up -d db`

Find the name of your mongo container with 

`docker ps` The following line assumes "splash_db_1", but that might be different on your system.


`docker exec -i splash_db_1 /bin/bash < ./init/init_mongo.sh`

Run `docker-compose up`

Wait until you see 
```

monstache        | INFO 2020/05/16 00:16:52 Successfully connected to Elasticsearch version 7.5.0

monstache        | INFO 2020/05/16 00:16:52 Listening for events

monstache        | INFO 2020/05/16 00:16:52 Watching changes on the deployment
```

Open another terminal and change your working directory to `./init/python`

Run:

`python3 -m venv env`

`source ./env/bin/activate`

`pip install -r requirements.txt`

Additionally if in development run: `pip install -r requirements-dev.txt`

Finally to set up the Elastic mappings run: `python3 setup.py`


Note: For some slow computers elastic will take too long to start up and so monstache will timeout when attempting to obtain a health check from elastic. If this happens then you will need to open `./monstache/monstache.config.toml` and make some changes.

Within this file set `elasticsearch-healthcheck-timeout` and `elasticsearch-healthcheck-timeout-startup` to larger values. Here is a sample config file:
```
elasticsearch-healthcheck-timeout-startup=60
elasticsearch-healthcheck-timeout=10
```

Adjust the values as needed.