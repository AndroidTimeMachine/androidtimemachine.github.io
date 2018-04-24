---
layout: page
title: Usage Guide
---

## Installation of the graph database

The Docker image containing the graph database is based on the [official Neo4j image.](https://store.docker.com/images/neo4j)  The only difference is, that this Docker image contains the dataset and an [`EXTENSION_SCRIPT`](http://neo4j.com/docs/operations-manual/current/installation/docker/#docker-new-image) ([`load_db.sh`](https://github.com/androidtimemachine/neo4j_open_source_android_apps/blob/master/scripts/load_db.sh)) which preloads the data when starting the container.

 1. Pull this image: `docker pull androidtimemachine/neo4j_open_source_android_apps`
 2. Use it as decribed in [official documentation](http://neo4j.com/docs/operations-manual/current/installation/docker/)

For example:

    docker run --rm --detach=true \
        --publish=7474:7474 --publish=7687:7687 \
        androidtimemachine/neo4j_open_source_android_apps

This command starts the Docker image and exposes ports used by Neo4j. The `--rm` options tells Docker to remove any newly created data inside the container after it has stopped running.

Map volumes into the container in order to persist state between executions:

    docker run --rm --detach=true \
        --publish=7474:7474 --publish=7687:7687 \
        --volume=$HOME/neo4j/data:/data \
        --volume=$HOME/neo4j/logs:/logs \
        androidtimemachine/neo4j_open_source_android_apps


When running the container for the first time, data gets imported into the graph database. This can take several seconds. Subsequent starts with an existing database in a mapped volume skip the importing step.


## Usage

You can access the Neo4j web-interface at `http://localhost:7474` and connect _Gopher_ clients to `bolt://localhost:7687`.

Alternatively, a  _Cypher_ shell can be run with the `bin/cypher-shell` command once a container is running.  Use the container ID returned by the `docker run` command or find it out with `docker ps`.

    $ docker run <...>  # As above
    6455917a2532b0c9bc335f93568022bd66c6dd4208f16b29b7f8b14b9418238b
    $ docker exec --interactive --tty 6455917a2532b0c9bc335f93568022bd66c6dd4208f16b29b7f8b14b9418238b bin/cypher-shell

When logging in for the first time, a new password needs to be set. Log-in with username `neo4j` and password `neo4j` to set a new password. [This step can be skipped by setting a default password or disabling authentication.](http://neo4j.com/docs/operations-manual/current/installation/docker/#docker-overview).


![Neo4j Web Interface](https://github.com/af60f75b/neo4j_open_source_android_apps/raw/master/doc/img/neo4jwebinterface.png)


## Example Queries

Below we list a series of example queries that highlight how to explore data in the graph.

For some of these queries, the Neo4j plugin [APOC](https://guides.neo4j.com/apoc) is necessary. Install it by mapping it into the container as follows:

    $ mkdir plugins
    $ cd plugins
    $ wget https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/download/3.3.0.1/apoc-3.3.0.1-all.jar
    $ docker run --rm --detach=true \
        --publish=7474:7474 --publish=7687:7687 \
        --volume=$PWD:/plugins \
        androidtimemachine/neo4j_open_source_android_apps

### Example 1

Select apps belonging to the _Finance_ category with more than 10 commits in a given week.

    WITH apoc.date.parse('2017-01-01', 's', 'yyyy-MM-dd')
            as start,
        apoc.date.parse('2017-01-08', 's', 'yyyy-MM-dd')
            as end
    MATCH (p:GooglePlayPage)<-[:PUBLISHED_AT]-
        (a:App)-[:IMPLEMENTED_BY]->
        (:GitHubRepository)<-[:BELONGS_TO]-
        (:Commit)<-[c:COMMITS]-(:Contributor)
    WHERE 'Finance' in p.appCategory
        AND start <= c.timestamp < end
    WITH a.id as package, SIZE(COLLECT(DISTINCT c)) as commitCount
    WHERE commitCount > 10
    RETURN package, commitCount

### Example 2

Select contributors who worked on more than one app in a given month.

    WITH apoc.date.parse('2017-01-01', 's', 'yyyy-MM-dd')
            as start,
        apoc.date.parse('2017-08-01', 's', 'yyyy-MM-dd')
            as end
    MATCH (app1:App)-[:IMPLEMENTED_BY]->
        (:GitHubRepository)<-[:BELONGS_TO]-
        (:Commit)<-[c1:COMMITS|AUTHORS]-
        (c:Contributor)-[c2:COMMITS|AUTHORS]->
        (:Commit)-[:BELONGS_TO]->
        (:GitHubRepository)<-[:IMPLEMENTED_BY]-
        (app2:App)
    WHERE c.email <> 'noreply@github.com'
        AND app1.id <> app2.id
        AND start <= c1.timestamp < end
        AND start <= c2.timestamp < end
    RETURN DISTINCT c
    LIMIT 20

### Example 3

Providing our dataset in containerized form allows future research to easily augment the data and combine it for new insights. The following is a very simple example showcasing this possibility.  Assuming all commits have been tagged with self-reported activity of developers, select all commits in which the developer is fixing a performance-related bug.  For demonstration purposes, a very simple tagger is applied. Optimally, tagging is done with a more sophisticated model.

    MATCH (c:Commit)
    WHERE c.message CONTAINS 'performance'
    SET c :PerformanceFix

Also, given these additional labels, performance related fixes can then be easily used in any kind of query via the following snippet.

    MATCH (c:Commit:PerformanceFix) RETURN c LIMIT 20

### Example 4

Metadata from GitHub and Google Play can be combined and compared.  Both platforms have popularity measures such as star ratings.  The following query returns these metrics for further analysis.

    MATCH (r:GitHubRepository)<-[:IMPLEMENTED_BY]-
        (a:App)-[:PUBLISHED_AT]->(p:GooglePlayPage)
    RETURN a.id, p.starRating, r.forksCount,
        r.stargazersCount, r.subscribersCount,
        r.watchersCount, r.networkCount
    LIMIT 20

### Example 5

Does a higher number of contributors relates to more successful apps? The following query returns the average review rating on Google Play and the number of contributors to the source code.

    MATCH (c:Contributor)-[:AUTHORS|COMMITS]->
        (:Commit)-[:BELONGS_TO]->
        (:GitHubRepository)<-[:IMPLEMENTED_BY]-
        (a:App)-[:PUBLISHED_AT]->(p:GooglePlayPage)
    WITH p.starRating as rating, a.id as package,
        SIZE(COLLECT(DISTINCT c)) as contribCount
    RETURN package, rating, contribCount
    LIMIT 20
