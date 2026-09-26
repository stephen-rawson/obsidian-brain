services:
  couchdb:
    image: couchdb:3
    container_name: couchdb
    networks:
      synobridge:
        ipv4_address: 172.20.1.41
    environment:
      COUCHDB_USER: obsidian
      COUCHDB_PASSWORD: Juniper-Dandelions-Aardvark64
    volumes:
      - /volume1/docker/couchdb/data:/opt/couchdb/data
      - /volume1/docker/couchdb/etc:/opt/couchdb/etc/local.d
    logging:
      driver: json-file
      options: { max-size: "10m", max-file: "3" }
    restart: unless-stopped

networks:
  synobridge:
    external: true