Original instructions from here:
https://github.com/vousmeevoyez/kong-konga-example
With a little difference:
- The Kong Admin API ports are not opened to host, but only exposed to containers
- Setting Kong Admin API loop back is described here in readme
- Simplified and other little cleanups

1. git clone https://github.com/aldycool/KongDeploy.git
2. Change to the directory: cd KongDeploy
3. Recommended to change KONGA_ENV in .env to new Guid value
4. To start installation, run: sudo ./start.sh
5. Setup the Kong Admin API Loopback:

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
# Open in browser: http://{kong-server-ip-address}:9000
# Follow the standard admin account creation procedure. For the Kong Admin API update connection screen, choose Key Auth tab, enter Name: can be anything (ex. KongAdminAPILoopBack), Loopback API Url: http://kong:8000/admin-api (the use of 'kong' here is supposed to be supported due to docker network), API Key: (the "key" from above)
 # don't forget to replace KONGA_ENV to production:
sudo docker-compose down
# Change KONGA_ENV=production, save the file.
sudo docker-compose up -d db kong konga
  