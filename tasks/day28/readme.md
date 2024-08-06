# [Day 28](https://www.youtube.com/watch?v=ZAPX21TMkkQ) - Docker Volume Explained - Docker Bind Mount - Docker Persistent Storage

Day 28 was preparing for the Day 29 on how to persist data in Docker.
The Video focussed mostly on [volumes](https://docs.docker.com/storage/volumes/) and how to persist data with it and where it is stored.
With these the data can also be shared between multiple containers

Create a Test-Image (example of day02)
```
git clone https://github.com/docker/getting-started-app.git
cd getting-started-app/
touch Dockerfile
```

Content of the Dockerfile
```
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```
Create it
```
docker build -t day02-todo .
```

Create a Volume in docker
```
docker volume create data_vol
```

See if it's created
```
docker volume ls
```

Also visible in
```
sudo ls /var/lib/docker/volumes
```

Run a Docker Container with a volume 
```
docker run -v data_vol:/app -dp 3000:3000 --name=todo day02-todo
```

## Tasks
The Task for Day 28 is to follow the steps in the video, like it's shown abount and also trying to use not only volumes, but also mounting a local folder into the Cotnainer

The command [mounting](https://docs.docker.com/storage/bind-mounts/) a folder into the container on startup is 
```
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```
