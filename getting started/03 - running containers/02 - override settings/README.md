The `run` command can be the last layer to cast modification on the image.

```sh
# I'mma just scamp on the possible variable here, does not really matter

$ docker run \ 
    --name CONTAINER_NAME \ 
# port
    -p HOST_PORT:CONTAINER_PORT \
# or use the env file like this
    -e ENV_KEY1:VALUE1 \
    -e ENV_KEY2:VALUE2 \
    --env-file .env \     
# specs
    --memory="512m" \ 
    --cpus="0.5" \
    images
```

One interesting aspect to override - custom network
- By default, containers automatically connect to a special network called a *bridge network* when you run them. 
- This allows containers on the same host to communicate with each other while keeping them isolated from the outside world and other hosts. 
- There will be time u don't want a container to be accessible by others. That's where an isolated, custom network comes in.

```sh
# inspect my network
$ docker network ls

# by default, every container is connected to bridge network
# create my own network
$ docker network create my_network

# warm up with 2 separate containers
$ docker run --name postgres-module1 -e POSTGRES_PASSWORD=123 -d -p 8080:5432 --network my_network postgres

$ docker run --name postgres-module2 -e POSTGRES_PASSWORD=456 -d -p 8081:5432 --network my_network postgres

# inspect the custom network, u will find two containers with their own addresseses
 ...
"Containers": {
          "787d8ee9958f20fbd1d12fc63ca422449ca8bcc6d5ee4c2b140c7795cc9979e8": {
              "Name": "postgres-module2",
              "EndpointID": "95e0a39274438f237612ad9a4f1179fe5bee51549efff22f90027821cc16c30c",
              "MacAddress": "02:42:ac:12:00:03",
              "IPv4Address": "172.18.0.3/16",
              "IPv6Address": ""
          },
          "de909c504df4c2e6599e8b17c39e3e0085c8f1c3e09442704fc2545047d5f045": {
              "Name": "postgres-module1",
              "EndpointID": "49ef00a800893d87da5e83c2d435798671960eb5c976587b520c9229e1cd422c",
              "MacAddress": "02:42:ac:12:00:02",
              "IPv4Address": "172.18.0.2/16",
              "IPv6Address": ""
          }
    },
    ...
  
# Good part is, we can connect from one container to another
$ docker exec -it postgres-module1 bash
$ psql -h 172.18.0.3 -p 5432 -U postgres
< enter module2 password >
```

Or you can override using compose file
```docker
services:
  psql-custom:
    image: postgres
    entrypoint: ["docker-entrypoint.sh", "postgres"]
    command: ["-h", "localhost", "-p", "5432"]
    environment:
      - POSTGRES_PASSWORD=123
```


Reference
- https://docs.docker.com/engine/network/tutorials/standalone/


