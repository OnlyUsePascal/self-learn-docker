## Basic
Create a file `Dockerfile` with the following

```Dockerfile
FROM node:20-alpine
WORKDIR /app

# copy source
COPY . . 
RUN yarn install --production

# default cmd
CMD ["node", "src/index.js"]
```

Common instructions for Dockerfile

```Dockerfile
FROM image

WORKDIR <img-path>

COPY <host-path> <img-path>

RUN <command>

ENV <key> <value> 

EXPOSE <port> 

# usually run "RUN useradd <user> first"
USER <user or uid>

CMD ["cmd", "parameters"]
```

Or, magically, we can use `docker init` to quickly boot up the files, including `docker-compose.yml`, `Dockerfile`, `.gitignore`

## Building An Image
```sh
$ docker build .

$ docker build -t jounpham/my-img .
```

With the previous command, the image will have no name, but the output will provide the ID of the image. Which is not very convenient for humans. That is why naming conventions are used

```
[HOST[:PORT_NUMBER]/]PATH[:TAG]
```
- `HOST`: docker registry. default = `docker.io`
- `PATH`: image path. The format is `[NAMESPACE/]REPOSITORY`. Default namespace is `library`
- `TAG`: identifier for tagging versions or variants. Default is `latest`

So for e.g.,
- `nginx` is equivalent to `docker.io/library/nginx:latest`
- `jounpham/my-img` ~ `docker.io/jounpham/my-img:latest`

## Build Cache

```Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "./src/index.js"]
```

When building an image, an instruction create a changing layer for the previous build. To minimize the total build time, **docker caching** checks if the previous build / state can be re-used. 

To avoid file conflict from using old builds, there are some rules for **cache invalidation**

- Any changes to the command of a `RUN` instruction invalidates that layer
- Any changes to files copied into the image with the `COPY` or `ADD` instructions
- Once one layer is invalidated, all following layers are also invalidated
- [More info](https://docs.docker.com/build/cache/invalidation/)

Turning back to the image build file
- The pipeline will be: "import node base image" -> "working directory located at `/app`" -> copy the whole source -> `yarn install` dependencies
- This means for every small change in source, build cache is invalidated, and initiated new layer + `yarn install` -> time-consuming.

```Dockerfile
FROM node:20-alpine
WORKDIR /app

# handle dependencies
COPY package.json yarn.lock ./
RUN yarn install --production 

# copy source
COPY . . 
EXPOSE 3000

# exec
CMD ["node", "src/index.js"]
```

- So one optimization is narrowing down the `COPY` instruction to only dependencies list `package.json`
- Except there are some new packages added or removed, the driver-based layer don't need to re-installed for every new build
- Oh also rmb to skip the `node_modules` dir using `.dockerignore`














