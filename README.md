# CDC-streaming-to-Snowflake-in-Azure
Streaming Framework that will stream change data capture data to Snowflake in near Real Time by Kafka and Event Hub

Change Data Capture (CDC) is a technique used to track row-level changes in database tables in response to create, update and delete operations. Different databases use different techniques to expose these change data events - for example, logical decoding in PostgreSQL, MySQL binary log (binlog) etc. This is a powerful capability, but useful only if there is a way to tap into these event logs and make it available to other services which depend on that information.

This tutorial walks you through how to set up a change data capture based system on Azure using Event Hubs (for Kafka), Azure DB for PostgreSQL and Debezium. It will use the Debezium PostgreSQL connector to stream database modifications from PostgreSQL to Snowflake tables via Kafka topics in Event Hubs.

Debezium is a distributed platform that builds on top of Change Data Capture features available in different databases. It provides a set of Kafka Connect connectors which tap into row-level changes (using CDC) in database table(s) and convert them into event streams. These event streams are sent to Apache Kafka which is a scalable event streaming platform - a perfect fit! Once the change log events are in Kafka, they will be available to all the downstream applications.


Prerequisite:

Create an Event Hubs namespace - After the setup, please ensure that you keep the Connection String handy since you will need it to configure Kafka Connect.
Setup and configure Azure Database for PostgreSQL - Please ensure that you keep the following PostgreSQL related information handy since you will need them to configure the Debezium Connector in the subsequent sections - database hostname (and port), username, password. please refer https://github.com/percy2112/CDC-streaming-to-Snowflake-in-Azure/blob/main/postgres_prerequisite_setup.txt
Snowflake Setup: please refer instructions from file https://github.com/percy2112/CDC-streaming-to-Snowflake-in-Azure/blob/main/snowflake_prerequisite_setup.txt

Configure PostgreSQL:

Before installing the kafka and PostgreSQL connector, we need to:

Ensure that the PostgreSQL instance is accessible from your Kafka Connect cluster
Ensure that the PostgreSQL replication setting is set to “Logical”
Create a table which you can use to try out the change data capture feature

If you're using Azure DB for PostgreSQL, create a firewall rule using az postgres server firewall-rule create command to whitelist your Kafka Connect host.

To change the replication mode for Azure DB for PostgreSQL, you can use the az postgres server configuration command:

az postgres server configuration set --resource-group <name of resource group> --server-name <name of server> --name azure.replication_support --value logical



or use the Replication menu of your PostgreSQL instance in the Azure Portal.

Setup Kafka Connect with Azure Event Hub and Debezium Postgres Connector and Snowflake Kafka Connector 

To run Kafka Connect , Here I am using Azure Virtual Machine to Install Kafka Connect and perform all setup activity.

1. Initial Setup and installing java environment:

This will help to install the java JDK.
• sudo apt update 
• sudo apt install default-jre 
• sudo apt install default-jdk

2. Downloading and Installing Kafka 

These commands will help us to install Apache Kafka with ease.
• sudo mkdir /home/kafka 
• cd /home/kafka
• sudo wget -c https://dlcdn.apache.org/kafka/3.2.0/kafka_2.12-3.2.0.tgz -O - | sudo tar -xz -C /home/kafka

3. Downloading and extracting Debezium plugin 

These commands will download the Debezium plugin for Postgres, it will help to communicate Postgres database to Kafka Connect.
• sudo wget -c https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/1.2.0.Final/debezium-connector-postgres-1.2.0.Final-plugin.tar.gz -O - | sudo tar -xz -C /home/kafka
• sudo cp debezium-connector-postgres/*  /home/kafka/kafka_2.12-3.2.0/libs 

4. Downloading and Extracting Snowflake plugin

These commands will download the snowflake jar which will help to consume events from Event Hub by connecting to  Kafka Connect.
• sudo wget -c https://repo1.maven.org/maven2/com/snowflake/snowflake-kafka-connector/1.8.0/snowflake-kafka-connector-1.8.0.jar
• sudo wget -c https://repo1.maven.org/maven2/org/bouncycastle/bc-fips/1.0.1/bc-fips-1.0.1.jar 
• sudo wget -c https://repo1.maven.org/maven2/org/bouncycastle/bcpkix-fips/1.0.3/bcpkix-fips-1.0.3.jar
• #############  Copy all the three jar file to kafka_2.12-3.2.0/libs/       ##############

5. Creating configuration properties for kafka

Creating config file connect-event.properties will setup Event Hub as Broker for Kafka Connect to manage/create topics in event hub.
• sudo mkdir config
• Save connect-event.properties in config folder ( please refer connect-event.properties file )

6. Start Kafka connect-

Starting kafka connect to run the process.
• cd kafka_2.12-3.2.0/bin
• sudo ../bin/connect-distributed.sh /home/kafka/config/connect-event.properties

7. Create postgres and Snowflake connector json files  in config folder

Here we will create connector files for source(Postgres) and Sink(Snowflake) .

Download the file from link and make the necessary changes as per your system details.

Open new terminal and run below command and paste the configuration and save the file.
• sudo nano pg-source-connector.json
• sudo nano sf-sink-connector.json
• #########    Please refer connector files on my GitHub Repo to Paste respective configuration settings on above file   #################

8. Register the connectors in Kafka Connect

By Using REST API call, we will register Postgres and Snowflake Connector  as Producer and Consumer respectively to Kafka Connect.

• curl -X POST -H "Content-Type: application/json" --data @pg-source-connector.json http://localhost:8083/connectors 
• curl -X POST -H "Content-Type: application/json" --data @sf-sink-connector.json http://localhost:8083/connectors 

9. Connector operations:
• To check status of connector: 
curl -s http://localhost:8083/connectors/sf-demo-sink-connector/status
• To Delete the registered connector:
curl -X DELETE http://localhost:8083/connectors/sf-demo-sink-connector

Conclusion:

Now we have setup every things and by doing some changes into Postgres Table we can see the Payload in Kafka log and in no time it will load the events into snowflake tables.


Change Data Capture is a powerful technique which can help "unlock the database" by providing near real-time access to it's changes.

Ref: 

Github Link: 

https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-kafka-connect-debezium

https://debezium.io/documentation/reference/1.9/connectors/postgresql.html

https://docs.snowflake.com/en/user-guide/kafka-connector.html
