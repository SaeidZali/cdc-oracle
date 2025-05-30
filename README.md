arrangement by Hossein Einabadi,Majid Peykani and Saeed Zali<br>
1.in wsl<br>
2.git clone https://github.com/SaeidZali/kafka-connect-oracle.git<br>
3.docker login container-registry.oracle.com<br>
4.docker pull container-registry.oracle.com/database/enterprise:21.3.0.0<br>
5.docker compose up --detach<br>
6.docker exec -it oracle /bin/bash<br>
7.cd /opt/oracle/oradata<br>
8.mkdir -p recovery_area<br>
9.curl https://raw.githubusercontent.com/SaeidZali/kafka-connect-oracle/refs/heads/main/01_logminer-setup.sh | sh<br>
10.curl https://raw.githubusercontent.com/SaeidZali/kafka-connect-oracle/refs/heads/main/inventory.sql | sqlplus debezium/dbz@//localhost:1521/orclpdb1<br>
11.exit<br>
12.docker exec -it oracle bash -c 'sleep 1; sqlplus Debezium/dbz@localhost:1521/orclpdb1'<br>
13.SELECT * FROM CUSTOMERS;<br>
14.exit<br>
15.exit<br>
16.docker exec -it connect /bin/bash<br>
17.curl https://maven.xwiki.org/externals/com/oracle/jdbc/ojdbc8/12.2.0.1/ojdbc8-12.2.0.1.jar -o ojdbc8-12.2.0.1.jar<br>
18.exit<br>
19.docker restart connect<br>
20.curl -X POST -H "Content-Type: application/json" -d '{ "name": "oracle-customer-source-connector-00", "config": { "connector.class": "io.debezium.connector.oracle.OracleConnector", "database.hostname": "oracle", "database.port": "1521", "database.user": "c##dbzuser", "database.password": "dbz", "database.server.name": "test", "database.history.kafka.topic": "history", "database.dbname": "ORCLCDB", "database.connection.adapter": "LogMiner", "database.history.kafka.bootstrap.servers": "kafka:9092", "table.include.list": "DEBEZIUM.CUSTOMERS", "database.schema": "DEBEZIUM", "database.pdb.name": "ORCLPDB1", "snapshot.mode": "schema_only", "include.schema.changes": "true", "key.converter": "io.confluent.connect.avro.AvroConverter", "value.converter": "io.confluent.connect.avro.AvroConverter", "key.converter.schema.registry.url": "http://schema-registry:8081", "value.converter.schema.registry.url": "http://schema-registry:8081" } }' http://localhost:8083/connectors<br>
21.curl -s "http://localhost:8083/connectors?expand=info&expand=status" | \
jq '. | to_entries[] | [ .value.info.type, .key, .value.status.connector.state,.value.status.tasks[].state,.value.info.config."connector.class"]|join(":|:")' | \
column -s : -t| sed 's/\"//g'| sort<br>
22.you should see<br>
source | oracle-customer-source-connector-00 | RUNNING | RUNNING | io.debezium.connector.oracle.OracleConnector<br>
