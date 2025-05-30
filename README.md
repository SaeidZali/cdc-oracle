in wsl<br>
docker login container-registry.oracle.com<br>
docker pull container-registry.oracle.com/database/enterprise:21.3.0.0<br>
docker compose up --detach<br>
docker exec -it oracle /bin/bash<br>
