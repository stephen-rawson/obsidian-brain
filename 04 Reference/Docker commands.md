## Container Management

- List containers: `sudo docker ps --format "table {{.ID}}\t{{.Names}}"`
- Remove containers (force stop): `sudo docker rm -f cont1 cont2`
- Restart containers: `sudo docker restart cont1 cont2`

## Debugging
- Live logs: `sudo docker logs cont1 -f`