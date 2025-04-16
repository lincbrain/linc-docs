# LINC | WebKNOSSOS Deployment

This document is designed to help deploy a new version for LINC | WebKNOSSOS via AWS EC2

### Create an instance in AWS EC2 with at least 32GB of memory

Proceed to AWS and create an AWS Linux instance

• r5.2xlarge is suggested for instance type
• x86_64 architecture is suggested
• Ensure that ports 80 and 443 are available.
• Ensure that the instance is reachable via Public IP address

### Connect the instance to a Route 53 Domain Record

Proceed to Route 53 and create an A Record with the desired domain that is pointing to the Public IP address of the EC2 Instance

### Return to AWS EC2 and ssh onto the instance

Once the instance is running, SSH onto the instance. 

First, install the appropriate dependencies -- you'll need docker, docker-compose (and most likely git and vim for file management)

```shell
sudo yum install docker git vim -y

sudo service docker start

sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")')/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

Next, proceed to do the following commands (These steps are mostly inline with https://docs.webknossos.org/webknossos/installation.html)

```shell
sudo mkdir opt && sudo cd opt
sudo mkdir webknossos && sudo cd webknossos

sudo mkdir certs && sudo mkdir certs-data

sudo wget https://github.com/scalableminds/webknossos/raw/master/tools/hosting/docker-compose.yml

sudo mkdir binaryData

sudo chown -R 1000:1000 binaryData

sudo touch nginx.conf
```

Next, you'll need to issue an SSL certificate directly on the server -- `certbot` is used here:

```shell
sudo docker run --rm -p 80:80 -v $(pwd)/certs:/etc/letsencrypt -v $(pwd)/certs-data:/data/letsencrypt certbot/certbot certonly --standalone -d webknossos.lincbrain.org --email admin@lincbrain.org --agree-tos --non-interactive
```

You'll need to next populate the nginx.conf -- replace `webknossos.lincbrain.org` with whatever A name you used in Route 53

```shell
events {}

http {
    # Main server block for the webknossos application
    server {
        listen 80;
        server_name webknossos.lincbrain.org;

        location /.well-known/acme-challenge/ {
            root /data/letsencrypt;
        }

        location / {
            return 301 https://$host$request_uri;
        }
    }

    server {
        listen 443 ssl http2;
        server_name webknossos.lincbrain.org;

        ssl_certificate /etc/letsencrypt/live/webknossos.lincbrain.org/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/webknossos.lincbrain.org/privkey.pem;

        # webknossos-specific overrides
        client_max_body_size 0;
        proxy_read_timeout 3600s;

        location / {
            set $cors '';
            if ($http_origin ~* (https://staging--lincbrain-org\.netlify\.app|https://.*\.lincbrain\.org|https://lincbrain\.org)) {
                set $cors 'true';
            }

            if ($cors = 'true') {
                add_header 'Access-Control-Allow-Origin' "$http_origin" always;
                add_header 'Access-Control-Allow-Credentials' 'true' always;
                add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
                add_header 'Access-Control-Allow-Headers' 'Accept, Content-Type, X-Requested-With, Authorization, Cookie' always;
            }

            if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Allow-Origin' "$http_origin" always;
                add_header 'Access-Control-Allow-Credentials' 'true' always;
                add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
                add_header 'Access-Control-Allow-Headers' 'Accept, Content-Type, X-Requested-With, Authorization, Cookie' always;
                add_header 'Content-Length' 0 always;
                add_header 'Content-Type' 'text/plain' always;
                return 204;
            }

            proxy_pass http://webknossos-webknossos-1:9000;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Cookie $http_cookie;
            proxy_set_header Transfer-Encoding "";
            proxy_buffering off;

            proxy_hide_header Access-Control-Allow-Origin;
            proxy_hide_header Access-Control-Allow-Credentials;
            proxy_hide_header Access-Control-Allow-Methods;
            proxy_hide_header Access-Control-Allow-Headers;
        }
    }

    # Separate server block for serving the binaryData directory
    server {
        listen 8080;
        server_name webknossos.lincbrain.org;

        location /binaryData/ {
	    alias /home/ec2-user/opt/webknossos/binaryData/;
            autoindex on;
            autoindex_exact_size off;
            autoindex_localtime on;
            allow all;
        }
    }
}
```

You'll next want to alter the `docker-compose` pulled earlier via `wget`

Remove the `nginx-letsencrypt` service, and alter the `nginx` as such:

```
nginx-proxy:
  image: nginx:latest
  container_name: nginx-proxy
  ports:
    - "8080:8080"
    - "80:80"
    - "443:443"
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf:ro
    - ./certs:/etc/letsencrypt
    - /home/ec2-user/opt/webknossos/binaryData:/home/ec2-user/opt/webknossos/binaryData:ro
  depends_on:
    - webknossos
```

`nginx` should now be able to be called appropriately via HTTPS once `webknossos` API is running

Lastly, you'll want to start the API and supporting containers:

```shell
DOCKER_TAG=xx.yy.z PUBLIC_HOST=webknossos.example.com LETSENCRYPT_EMAIL=admin@example.com \
docker compose up -d webknossos nginx
```

You can check the health of the containers via:

```
docker ps

# or

docker logs -f <container-id>
```

## Backups

### FossilDB

FossilDB is a scalableminds database that extends from the open-source RocksDB.

Temp steps / commands for FossilDB backup:

1. Exec into EC2 instance
2. Grab `fossildb-client` via `docker pull scalableminds/fossildb-client:master`
3. Determine the appropriate internal network that the `fossildb` instance is running in within the Dockerized setup on EC2: `docker inspect -f '{{range .NetworkSettings.Networks}}{{.NetworkID}} {{end}}' webknossos-fossildb-1`
4. `docker run --network <network-id> scalableminds/fossildb-client:master webknossos-fossildb-1 backup` should create the backup
5. The backup will be stored via `/home/ec2-user/opt/webknossos/persistent/fossildb/backup`

## Creating a new WebKNOSSOS with pre-existing backups

There are three different components that must be taken into account for a WebKNOSSOS clone:

• mounted Docker volumes -- represented by the `binaryData` and `persistent` directories in the WebKNOSSOS file structure
  - exported to AWS S3 via the `docker_volumes_backup.sh` cronjob script
• FossilDB data (manged via `fossildb-client restore` commands)
  - exported to AWS S3 via the `fossil_db_backup.sh` cronjob script
• PostgresDB data (managed via `pg_dump` and `pg_restore` commands)
  - exported to AWS S3 via the `postgres_backup.sh` cronjob script

When setting up a new clone, first follow the standard deployment steps above, **however** do not create the `binraryData` folder

You'll first want to restore the Docker volumes -- contained in the `webknosos_backups/` S3 subdirectory for wherever your cron jobs send the compressed backups

Copy the appropriate assets from S3 to the EC2 instance via the `aws cp <backup-bucket> <current destination>`

For example:

```
aws s3 cp s3://linc-brain-mit-staging-us-east-2/fossildb_backups/backup_2024-08-20_02-00-02.tar.gz ./backup_2024-08-20_02-00-02.tar.gz
```

Once you decompress (can use a tool like `gunzip`) and then extract the files -- (e.g. `tar -cvzf /home/ec2-user/opt/webknossos/webknossos_backup.tar.gz .`)
you are ready to proceed; however, ensure that `persistent` and `binaryData` folders from the extracted files are in the same directory as your `docker-compose.yml` file

Next, you want to restore the `fossildb` instance -- this can simply be done via the `docker-compose run fossil-db-restore` command

Almost there! You'll next want to bring up the remainder of the WebKNOSSOS API (along with the nginx-proxy, postgres, etc.) via `docker-compose --env-file env.txt webknossos nginx-proxy`

Notably, this will bring up the `postgres` container (however, we've yet to restore the container!). Thus you'll want to:
  - Mount the decompressed, unpacked backup (should be something like `<backup_timestamp>.sql`). The mount command should be something similar to: `docker cp /local/path/to/postgres_backup.sql <container_id>:/tmp/postgres_backup.sql`
  - Exec into the `postgres` container and open a `psql` shell via `psql -U postgres`
  - Next, drop the `webknossos` database -- e.g. `DROP DATABASE webknossos`
  - Create the database `webknossos` -- e.g. `CREATE DATABASE webknossos`
  - Restore the database's state via psql -- e.g. `psql -U postgres -d webknossos -f /tmp/webknossos_backup.sql`

Your clone should be all set now!







