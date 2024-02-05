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
-e POSTGRES_USER="postgres" \
-e POSTGRES_PASSWORD="postgres" \
-e POSTGRES_DB="postgres" \
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
-e POSTGRES_USER="postgres" \
-e POSTGRES_PASSWORD="postgres" \
-e POSTGRES_DB="postgres" \
--network=airflow_default \
-p 5434:5434 \
-v pgbackrest_pg0002:/var/lib/postgresql/data \
-v pgbackrest_backup:/var/lib/postgresql/backup \
postgres:15.5
````


[Instalação do pgbackrest:](https://pgbackrest.org/user-guide.html#async-archiving)
````
mkdir -p /build
wget -q -O - \
       https://github.com/pgbackrest/pgbackrest/archive/release/2.49.tar.gz | \
       tar zx -C /build

sudo apt-get install make gcc libpq-dev libssl-dev libxml2-dev pkg-config \
       liblz4-dev libzstd-dev libbz2-dev libz-dev libyaml-dev libssh2-1-dev

cd /build/pgbackrest-release-2.49/src && ./configure && make

sudo apt-get install postgresql-client libxml2 libssh2-1

sudo scp build:/build/pgbackrest-release-2.49/src/pgbackrest /usr/bin

sudo chmod 755 /usr/bin/pgbackrest
sudo mkdir -p -m 770 /var/log/pgbackrest
sudo chown postgres:postgres /var/log/pgbackrest
sudo mkdir -p /etc/pgbackrest
sudo mkdir -p /etc/pgbackrest/conf.d
sudo touch /etc/pgbackrest/pgbackrest.conf
sudo chmod 640 /etc/pgbackrest/pgbackrest.conf
sudo chown postgres:postgres /etc/pgbackrest/pgbackrest.conf

sudo -u postgres pgbackrest
````



