# Configurando pgbackrest em dois servidores em docker.

O projeto será configura e executar backup no servidor __pg0001__ e restaura a instância no __pg0002__, as duas instância de banco estaram rodando no docker.


Criando os volumes:
````
docker volume create --driver local --opt type=none --opt device=/data/pgbackrest/backup --opt o=bind pgbackrest_backup
docker volume create --driver local --opt type=none --opt device=/data/pgbackrest/pg0001 --opt o=bind pgbackrest_pg0001
docker volume create --driver local --opt type=none --opt device=/data/pgbackrest/pg0002 --opt o=bind pgbackrest_pg0002
````

Criando os containers:
__pg0001__:
````
docker run --name pg0001 -t \
-e DB_SERVER_HOST="pg0001" \
-e POSTGRES_USER="Sentinel" \
-e POSTGRES_PASSWORD="Sentinel" \
-e POSTGRES_DB="SentinelDataSuite" \
--network=airflow_default \
-p 5433:5433 \
-v pgbackrest_pg0001:/var/lib/postgresql/data \
-v pgbackrest_backup:/var/lib/postgresql/backup \
postgres:15.5
````
__pg0002__:
````
docker run --name pg0002 -t \
-e DB_SERVER_HOST="pg0002" \
-e POSTGRES_USER="Sentinel" \
-e POSTGRES_PASSWORD="Sentinel" \
-e POSTGRES_DB="SentinelDataSuite" \
--network=airflow_default \
-p 5433:5433 \
-v pgbackrest_pg0002:/var/lib/postgresql/data \
-v pgbackrest_backup:/var/lib/postgresql/backup \
postgres:15.5
````



