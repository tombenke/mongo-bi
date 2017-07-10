mongo-bi
========

## About

This is a docker-compose project which helps to analyze and monitor the business data of an application that is stored in MongoDb.

Its components are docker images, that work together as a pipeline to deliver the end results.

You can use in one of the two modes:
- on-line, or
- off-line.

The drawing below shows how the on-line pipeline looks like:

    +---------+     +-------------+     +---------------+     +--------+     +---------+
    | mongodb |---->| transporter |---->| elasticsearch |---->| kibana |---->| browser |
    +---------+     +-------------+     +---------------+     +--------+     +---------+

In thi case the live `mongodb` database is synchronized to the `elasticsearch` server by the `transporter` utility.
The `elasticsearch` is indexing the incoming documents, stores and runs queries, and provides search results in JSON format through its REST API,
that is consumed by `kibana`, which helps to interactively analyze and visualize the result of queries.
The user can see the final results on its web `browser`.

The off-line mode is similar, but the data is fed elasticsearch not directly, but from a local backup of the live database,
as the next drawing shows:

    +--------+     +-----------+     +---------------+     +-------------+     +---------------+     +--------+     +---------+
    |mongodb |---->| db-backup |---->| local-mongodb |---->| transporter |---->| elasticsearch |---->| kibana |---->| browser |
    +--------+     +-----------+     +---------------+     +-------------+     +---------------+     +--------+     +---------+

So, first a backup has to be created into the local disk. The `local-mongodb` is configured to see directly this backup like if it were
its own database, then, in the followings, it feeds the pipeline like it happens in case of the on-line mode.


## Configuration

Before run the setup, you have to define the following environment variables:

- `MONGO_BACKUP_DIR`: THe path to the folder you stored your mongodb backup.
- `DB_NAME`: The name of the database you want to analyze. This will the also the name of the index on elasticsearch,
   into which the `transporter` will upload the documents.
- `MONGODB_URI`: The full URI of the mongodb server. Either the local one or the on-line one, depending on which mode you want to use.
- `ELASTICSEARCH_URI`: The full URI of the elasticserach server.

You should create a dedicated env file for each setup, you are willing to analyze.

This is an example of an environment file, that you can use for off-line mode:

    export MONGO_BACKUP_DIR="/home/tombenke/topics/myproject/myproject-db/2017-06-25"
    export DB_NAME="myproject"
    export MONGODB_URI="mongodb://mongo:27017"
    export ELASTICSEARCH_URI="http://elasticsearch:9200"

In both on-line and off-line modes, the docker-compose file is the same, but you will not use the `mongodb`

## References

- [Environment variables in Compose](https://docs.docker.com/compose/environment-variables/)
- [From inside of a Docker container, how do I connect to the localhost of the machine?](https://stackoverflow.com/questions/24319662/from-inside-of-a-docker-container-how-do-i-connect-to-the-localhost-of-the-mach)
- [Controlling startup order in Compose](https://docs.docker.com/compose/startup-order/)
- [Wait for another service to become available](https://github.com/Eficode/wait-for/blob/master/wait-for)
- [X-Pack](https://www.elastic.co/products/x-pack)
- [danielguerra/elasticsearch-x-pack - Elasticsearch with x-pack installed](https://github.com/danielguerra69/elasticsearch-x-pack)
- [danielguerra/kibana-x-pack - Kibana with x-pack installed](https://github.com/danielguerra69/kibana-x-pack)
- [Data visualization with Elasticsearch aggregations and D3](https://www.elastic.co/blog/data-visualization-elasticsearch-aggregations)
