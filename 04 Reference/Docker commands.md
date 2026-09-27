## Container Management

- List containers: `sudo docker ps --format "table {{.ID}}\t{{.Names}}"`
- Remove containers (force stop): `sudo docker rm -f cont1 cont2`