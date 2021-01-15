# Original instructions from here:

https://github.com/vousmeevoyez/kong-konga-example

With a little difference:
- The Kong Admin API ports are not opened to host, but only exposed to containers
- Setting Kong Admin API loop back is described here in readme
- Simplified and other little cleanups

# How-To

1. `git clone https://github.com/aldycool/KongDeploy.git`
2. Change to the directory: `cd KongDeploy`
3. Recommended to change KONGA_ENV in .env to new Guid value
4. To start installation: `sudo ./deploy.sh`
5. Setup the Kong Admin API Loopback:

```
# Add Kong Admin API as Service:
curl --location --request POST 'http://localhost:8001/services/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "admin-api",
    "host": "localhost",
    "port": 8001
}'

# Add Kong Admin API's Route:
curl --location --request POST 'http://localhost:8001/services/admin-api/routes' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paths": ["/admin-api"]
}'

# Test:
curl localhost:8000/admin-api/

# Enable Key Auth Plugin:
curl -X POST http://localhost:8001/services/admin-api/plugins \
    --data "name=key-auth"

# Add Konga as Consumer:
curl --location --request POST 'http://localhost:8001/consumers/' \
--form 'username=konga' \
--form 'custom_id=custom-id-konga'

# TAKE NOTE of the generated "id" from above, replace the {generated_id} below:
curl --location --request POST 'http://localhost:8001/consumers/{generated_id}/key-auth'

# TAKE NOTE of the "key" from above, this will be used in the Konga setup
```

6. Open in browser: `http://{kong-server-ip-address}:9000`, where `{kong-server-ip-address}` is the IP address of your host.

7. Follow the standard admin account creation procedure. For the Kong Admin API update connection screen, choose Key Auth tab
- Name: can be anything (ex. KongAdminAPILoopBack), 
- Loopback API Url: http://kong:8000/admin-api (the use of 'kong' here is supposed to be supported due to docker network)
- API Key: (the "key" from above)

8. don't forget to replace KONGA_ENV to production:
```
sudo docker-compose down
# Change KONGA_ENV=production, save the file.
sudo docker-compose up -d db kong konga
```

# Deployment Notes
- The `deploy.sh` is only executed once, and and all instruction above are expected to be executed completely.
- To shut down the containers, run `sudo docker-compose down`
- During the shut down, if needed, all the objects created (docker network, persistent volume) can be deleted to reset back to original state, run `sudo docker system prune --volumes`
- To activate the containers again, run `sudo docker-compose up -d db, kong, konga`. The containers are explicitly stated here, because these are the only active containers needed (the `kong-migrations` is just only run once)
- If you plan to deploy this in **Docker Swarm** please be aware that the current port exposures in the `kong` container **will expose the Kong Admin API Ports in Docker Swarm!!** even though it is already binded to loopback API (127.0.0.1:8001). To mitigate this, read about binding using mode: host in here: https://stackoverflow.com/questions/50621936/docker-service-exposed-publicly-though-made-to-expose-ports-to-localhost-only

  