Publishing container's port via host's ports

```sh
# Compose file
...
  app:
    ports:
      - host_port:container_host
  
# Dockerfile
EXPOSE CONTAINER_HOST

# run command
$ docker run -p host_port:container_host image_name
```

