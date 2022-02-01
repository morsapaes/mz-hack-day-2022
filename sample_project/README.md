# Taking off with Materialize, Redpanda and dbt

This is a sample project with enough plumbing to spin up an end-to-end analytics pipeline using Materialize, Redpanda and dbt to explore real-time data.

## Setup

<p align="center">
<img width="650" alt="demo_overview" src="https://user-images.githubusercontent.com/23521087/151333471-98ad518d-5ac5-444e-b065-83e3aaa42748.png">
</p>

The project uses [Docker Compose](https://docs.docker.com/get-started/08_using_compose/) to make it easier to bundle up all the services in the pipeline:

* **Data generator**

  Finding streaming data to play with isn't always a breeze, so we wired up a data generator that polls the [OpenSky Network API](https://openskynetwork.github.io/opensky-api/index.html) continuously to save you some time! For details like poll frequency and message schema, or just a template to create a message producer for a different source of data, check the [`data-generator` directory](/data-generator).

* **Redpanda**

* **Materialize**
  `mzcli`

* **dbt**

* **Metabase**

  One option to get data out of Materialize is to plug it into a BI tool. Materialize [Metabase]().

### Installation

To get started, make sure you have installed:

* [Docker](https://docs.docker.com/get-docker/)
* [Docker Compose](https://docs.docker.com/compose/install/)

We recommend running Docker with at least 2 CPUs and 8GB of memory, so double check your [resource preferences](https://docs.docker.com/desktop/mac/#preferences) before moving on!

### Getting the setup up and running

> :raised_hand_with_fingers_splayed: **Warning for M1 Mac users:** some of the tools used in this project don't provide multi-architecture Docker images just yet, so you need to run an extra initial step to set the variables that determine the right images to fetch for `arm64`. **Don't skip this step!**

If you're on a M1 Mac, first run:

```bash
export ARCH=linux/arm64 MIMG=iwalucas`
```

To get the setup up and running:

```bash
# Start the setup
docker-compose up -d

# Check if everything is up and running!
docker-compose ps
```

At any point, you can stop the data generator using:

```bash
docker stop data-generator
```

Once you're done playing with the setup, tear it down:

```bash
docker-compose down -v
```

## dbt

If this is your first time trying out dbt with Materialize, the ["dbt and Materialize"](https://materialize.com/docs/guides/dbt/#document-and-test-a-dbt-project) guide has everything you need to get started!

### Get in that shell!

```bash
docker exec -it dbt /bin/bash
```

From here, you can run your dbt commands as usual. For example, to check that the `dbt-materialize` plugin has been installed, and that everything is working:

```bash
dbt --version

dbt debug
```

### Build and run your models

```bash
dbt run
```

If you don't usually work with Docker, you might be asking yourself: how do I go about modifying existing models or adding new ones? The `/dbt` directory is mounted into the container when , so any changes you make in the `/dbt` folder will.

### Generate documentation

```bash
dbt docs generate

dbt docs serve
```

## Redpanda

### Check the source data

The data generator produces JSON-formatted events with flight information into the `flight_information` Redpanda topic. To check that the topic has been created:

```bash
docker-compose exec redpanda rpk topic list
```

and that there's data landing:

```bash
docker-compose exec redpanda rpk topic consume flight_information
```

To exit the consumer, press **Ctrl+C**.

## Materialize

[](https://materialize.com/docs/get-started/)

### Inspect the database

To connect to the running Materialize service, you can use any [compatible CLI](https://materialize.com/docs/connect/cli/). Let's roll with `mzcli`, which is included in the setup:

```bash
docker-compose run mzcli
```

```sql
SHOW SOURCES;

         name
-----------------------
 icao_mapping
 rp_flight_information
```

* icao_mapping

* rp_flight_information

```sql
SHOW VIEWS;

          name
------------------------
 stg_flight_information
 stg_icao_mapping
```

* stg_flight_information

* stg_icao_mapping

### Query the transformed data

explain that the existing views are non-materialized and you need to materialize something + what you really want to keep in-memory




As an example, you can create a dbt model with the following statement that enriches flight information with aircraft reference data:

```sql

```

and see the results :

```sql

```

If you re-run the `SELECT` statement at different points in time, you can see the updated results based on the latest data. And this is where Materialize shines! with materialized views that continuously update as the underlying data changes.

## Metabase

To visualize the results in [Metabase](https://www.metabase.com/):

**1.** In a browser, navigate to <localhost:3030>.

**2.** Click **Let's get started**.

**3.** Complete the first set of fields asking for your email address. This
    information isn't crucial for anything but has to be filled in.

**4.** On the **Add your data** page, specify the connection properties for the Materialize database:

Field             | Value
----------------- | ----------------
Database          | PostgreSQL
Name              | opensky
Host              | **materialized**
Port              | **6875**
Database name     | **materialize**
Database username | **materialize**
Database password | Leave empty

**5.** Click **Ask a question** -> **Native query**.

**6.** Under **Select a database**, choose **opensky**.