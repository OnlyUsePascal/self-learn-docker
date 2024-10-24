# Persisting Container Data
The point is when we remove the containers, the data is still on local disk

Managing Volumes

```sh
$ docker volume create <name>

$ docker volume rm <name or id>

# remove all unused volumes
$ docker volume prune 
```
Use container with volumes

```sh
$ docker run -v (or --volume) <volume>:<container path> ...
```

Example for postgres

```sh
$ docker volume create postgres_data


# init db + bind postgres data to volume
$ docker run -d -name=postgres_example \
            -e POSTGRES_PASSWORD=123 \
            -v postgres_data:/var/lib/postgresql/data \
            postgres
            

# log in db shell 
$ docker exec -it postgres_example psql -U postgres


# & create sample data
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    description VARCHAR(100)
);
CREATE TABLE

INSERT INTO tasks (description) VALUES ('Finish work'), ('Have fun');


# verify
select * from tasks;
 id | description 
----+-------------
  1 | Finish work
  2 | Have fun
(2 rows)


# now stop & remove container, then create another
$ docker container stop postgres_example  

$ docker container rm postgres_example  

$ docker run --name=postgres_another -d -v postgres_data:/var/lib/postgresql/data postgres


# verify if the data is still there
$ docker exec -it postgres_another psql -U postgres

postgres=# select * from tasks;
 id | description 
----+-------------
  1 | Finish work
  2 | Have fun
```
- Notice when you run another container `postgres_another` u don't need to use password env, as the db is already there. 
- Oh and since it's already there, **password cannot be override unless configured inside db shell**


# Sharing Local Files

Two main ways for file sharing: *volumes and bind mounts*

volume `-v (--volume)` 
  - if you want the data remains after running or removing the container. /
  - More convenient for basic volume

Bind mounts `--mount`
  - If you have specific files or directories on your host system that you want to directly share with your container (e.g., config, compiled binaries )
  - Offer more advanced features & granular controls. More recommended on production.

```sh
$ docker run -v HOST-DIRECTORY:/CONTAINER-DIRECTORY:rw nginx

$ docker run --mount type=bind,source=/HOST/PATH,target=/CONTAINER/PATH,readonly nginx
```

A nice example
```sh
# start a container using httpd image
$ docker run -d -p 8080:80 --name my_site httpd:2.4


# try ping
$ curl localhost:8080/ 
# it works!!!


# Now try to insert another html file
$ makedir public_html

$ cd public_html

$ touch index.html
...


# Bind html file to container path
# using volumes
$ docker run -d --name my_site_volume -p 8080:80 -v .:/usr/local/apache2/htdocs/ httpd:2.4

# using bind mount
$ docker run -d --name my_site -p 8080:80 --mount type=bind,source=./,target=/usr/local/apache2/htdocs/ httpd:2.4


# Profit ! ! !
```












