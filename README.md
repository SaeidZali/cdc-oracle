arrangement by Hossein Einabadi,Majid Peykani and Saeed Zali<br>
git clone https://github.com/SaeidZali/kafka-connect-oracle.git<br>
in wsl<br>
docker login container-registry.oracle.com<br>
docker pull container-registry.oracle.com/database/enterprise:21.3.0.0<br>
docker compose up --detach<br>
docker exec -it oracle /bin/bash<br>
cd /opt/oracle/oradata<br>
mkdir -p recovery_area<br>
curl https://raw.githubusercontent.com/SaeidZali/kafka-connect-oracle/refs/heads/main/01_logminer-setup.sh | sh<br>
curl https://raw.githubusercontent.com/SaeidZali/kafka-connect-oracle/refs/heads/main/inventory.sql | sqlplus debezium/dbz@//localhost:1521/orclpdb1<br>
exit<br>
docker exec -it oracle bash -c 'sleep 1; sqlplus Debezium/dbz@localhost:1521/orclpdb1'<br>
SELECT * FROM CUSTOMERS;<br>
exit<br>
exit<br>
docker exec -it connect /bin/bash<br>
curl https://maven.xwiki.org/externals/com/oracle/jdbc/ojdbc8/12.2.0.1/ojdbc8-12.2.0.1.jar -o ojdbc8-12.2.0.1.jar<br>
exit<br>
docker restart connect<br>
curl -X POST -H "Content-Type: application/json" -d '{ "name": "oracle-customer-source-connector-00", "config": { "connector.class": "io.debezium.connector.oracle.OracleConnector", "database.hostname": "oracle", "database.port": "1521", "database.user": "c##dbzuser", "database.password": "dbz", "database.server.name": "test", "database.history.kafka.topic": "history", "database.dbname": "ORCLCDB", "database.connection.adapter": "LogMiner", "database.history.kafka.bootstrap.servers": "kafka:9092", "table.include.list": "DEBEZIUM.CUSTOMERS", "database.schema": "DEBEZIUM", "database.pdb.name": "ORCLPDB1", "snapshot.mode": "schema_only", "include.schema.changes": "true", "key.converter": "io.confluent.connect.avro.AvroConverter", "value.converter": "io.confluent.connect.avro.AvroConverter", "key.converter.schema.registry.url": "http://schema-registry:8081", "value.converter.schema.registry.url": "http://schema-registry:8081" } }' http://localhost:8083/connectors<br>
curl -s "http://localhost:8083/connectors?expand=info&expand=status" | \
jq '. | to_entries[] | [ .value.info.type, .key, .value.status.connector.state,.value.status.tasks[].state,.value.info.config."connector.class"]|join(":|:")' | \
column -s : -t| sed 's/\"//g'| sort<br>
