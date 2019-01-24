# Getting started

We can create the containers manually using the procedure below or we can run the docker-compose.yml.  

1. Create docker network for kong 
2. Create docker Postgres database for kong
3. Start an ephimeral container to run kong database migrations 
4. Create docker container with kong 
3. Start an ephimeral container to run konga database migrations 
5. Create docker container with konga

``` bash
# Create docker network for kong
docker network create kong-net

# Create docker Postgres database for kong
docker run -d --name kong-database --network=kong-net -p 5432:5432 -e "POSTGRES_USER=kong" -e "POSTGRES_DB=kong" postgres:9.6

# Start an ephimeral container to run database migrations
docker run --rm --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" kong:latest kong migrations bootstrap

# Create docker container with kong 
docker run -d --name kong --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" -e "KONG_PROXY_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" -p 8000:8000 -p 8443:8443 -p 8001:8001 -p 8444:8444 kong:latest

# Start an ephimeral container to run konga database migrations 
docker run --rm --network=kong-net pantsel/konga -c prepare -a postgres -u postgresql://kong@kong-database:5432/konga_db

# Create docker container with konga
docker run -d -p 1337:1337 --network=kong-net -e "DB_ADAPTER=postgres" -e "DB_HOST=kong-database" -e "DB_USER=kong" -e "DB_DATABASE=konga_db" -e "KONGA_HOOK_TIMEOUT=120000" -e "NODE_ENV=production" --name konga pantsel/konga:next
```


## OAuth2 get token of client credentials type 

curl --insecure -X POST --data "grant_type=client_credentials" --data "client_id=UwdMAVIXfo8dIyLkbPpngqTMG8Qeuolb" --data "client_secret=YoFKHCNDtYw71TqKoxHN6ZmFkJ5UtyVF" -H "Host: jsonplaceholder.typicode.com" https://localhost:8443/oauth2/token