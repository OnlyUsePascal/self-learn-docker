Run docker with base image ubuntu in interactive mode

```sh
$ docker run --name=base-container -ti ubuntu
```

* [docker run vs docker start](https://stackoverflow.com/questions/34782678/difference-between-running-and-starting-a-docker-container)

Inside container 
```sh
# install node
$ apt update && apt install nodejs

# try node
$ node -e 'console.log("send nude")'
```

At new terminal, commit the changes as a new image named `node-img`
```sh
$ docker container commit -m "Add node" base-container node-img

# check history to see the change
$ docker image history node-img

# exec node to see if changes applied
# below run new img in new instance with command node ...
$ docker run node-img node -e 'console.log(123)'
```

We can level 1 more step with a small js file
- option `-t` [meaning](https://docs.docker.com/reference/cli/docker/container/run/#tty)

```sh
# run interactive mode + new container name + create js file
$ docker run --name=node-container -it node-img
$ echo 'console.log(`hello world`)' > app.js

# commit change in new img
# this time we add a default startup command 
$ docker commit -m 'add app.js' -c 'CMD node app.js' node-container app-img

# test run
$ docker run app-img
```
