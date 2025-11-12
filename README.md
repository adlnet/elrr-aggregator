# ELRR Aggregator
ELRR service which aggregates incoming data to perform state changes to Learner Profiles.

There are database and kafka dependencies, but there's a [repo with a docker-compose](https://github.com/adlnet/elrr-dockercompose) that resolves them locally.

## Dev Requirements
- [Java JDK 17](https://www.oracle.com/java/technologies/downloads/) or later
- [Maven](https://maven.apache.org/)
- [PostgreSQL Database](https://www.postgresql.org/) (can install, or use [docker container](https://hub.docker.com/_/postgres/tags) version)

## Tools
- Database Client (can use postgres CLI, pgadmin, or a 3rd party client like [DBeaver](https://dbeaver.io/))
- [Docker](https://www.docker.com/products/docker-desktop/) if working with containers

## Running the Application

### 1. Build the application
`mvn clean install`

This will test, compile, and create a jar for the app in `target/`

### 2. Start and Configure the Database

You will need a running PostgreSQL database containing the schema in [Service Entities](https://github.com/adlnet/elrr-services-entities/blob/main/dev-resources/schema.sql).

You will also need to configure the app properties/ENV to point to that database (See **Properties and Environment Variables** below)

One option is to use the ELRR [Local Development Docker Compose](https://github.com/adlnet/elrr-dockercompose) which runs all of the appropriate dependencies with the connection details already in `application-local.properties`, but it does not seed the database with schema, so you will still need to use a DB client to run Service Entities' `schema.sql` against the `service-db` container's database.

### 3a. Running the application using the Spring Boot Maven plugin: 
This is the recommended and easiest way to run a local version of the application

- `mvn spring-boot:run -D spring-boot.run.profiles=local -e`  (Linux/MacOS)
or
`mvn spring-boot:run -D"spring-boot.run.profiles"=local -e` (Windows)

Note that profile is being set to `local`, this tells spring to leverage `src/main/resources/application-local.properties` which allows you to easily change system settings for your local run. See **Properties and Environment Variables** for details.

### 3b. Running the application using the Jar file
This is closer to how the application will run in a Docker Container or in production.

- `cd target/`
- `java -jar elrraggregator-_.jar` (you must fill in the version number that matches the current target build)

To configure launch for this method you will set ENV variables instead of tweaking `application-local` as the jar will default to `application.properties` which accepts ENV overrides.

## Properties and Environment Variables
Configuration variables for running the application

| Property | ENV Variable | Default | Description |
| -------- | -------- | -------- | -------- |
| spring.datasource.url (partly, jdbc url)  | PGHOST | - | PostgreSQL Server Host
| spring.datasource.url (partly, jdbc url)  | PGPORT | - | PostgreSQL Server Port
| spring.datasource.url (partly, jdbc url)  | PG_DATABASE | - | PostgreSQL Database Name
| spring.datasource.username  | PG_RW_USER | - | PostgreSQL Username
| spring.datasource.password  | PG_RW_PASSWORD | - | PostgreSQL Password
| spring.jpa.properties.hibernate.default_schema  | ELRR_DB_SCHEMA | datasync_schema | Default PostgreSQL Schema
| brokerUrl (partly, combined with port)  | ELRR_AGG_BROKER_HOST | elrr-kafka | Kafka Broker Host
| brokerUrl (partly, combined with host)  | ELRR_AGG_BROKER_PORT | 9092 | Kafka Broker Port
| kafka.topic  | ELRR_AGG_BROKER_TOPIC | datasync-statements | Kafka Topic
| kafka.dead.letter.topic  | ELRR_AGG_BROKER_DLQ | datasync-statements-dlq | Kafka Dead Letter Queue
| kafka.groupId  | ELRR_AGG_BROKER_GROUPID | elrr-consumer-group | Kafka Group ID
| kafka.groupIdConfig  | ELRR_AGG_BROKER_GROUPID_CONFIG | elrr-consumer-group | Kafka Group ID Config

## Running Locally With Dependencies and Testing

### System Dependencies

#### Kafka

Aggregator depends on a Kafka Topic to receive new xAPI data. You can create a local Kafka, use docker, or cloning and running the [ELRR Local Docker Compose](https://github.com/adlnet/elrr-dockercompose) will provide a database and kafka already matching the current defaults in `application-local.properties`. If you start up this compose, and then run using Spring Boot as detailed above, you should have a running version.

#### Datasync and End-to-End Testing

[ELRR Datasync](https://github.com/adlnet/elrr-datasync) is the component which pushes statements to the Kafka topic for Aggregator to consume. If you would like to do an end to end test you will likely have to setup a running Datasync and External Services (referenced in Datasync docs) to test feeding data from an LRS. You can verify the results of the test by interrogating Aggregator's Database to see that records are updated successfully.
