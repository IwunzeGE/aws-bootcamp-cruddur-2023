# Week 1 — App Containerization

## Building and Testing out the Backend Locally
```
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

A 404 error was gotten at first because the ENV_VAR werent specified

![404 not found](https://user-images.githubusercontent.com/110903886/221421754-19b135f2-d5e6-4642-b86c-4d723425488a.png)

Export the ENV values and unlock your port as shown below:

![unlock port](https://user-images.githubusercontent.com/110903886/221421714-707745a6-7541-4b1a-a045-9e67921c7a55.png)

![export](https://user-images.githubusercontent.com/110903886/221421708-77ed45a3-8520-4e71-9af2-55951bbff994.png)

It now returns a json when the url is appended to `/api/activities/home`

![json output](https://user-images.githubusercontent.com/110903886/221421895-3c3a3f55-2f49-4892-9e5f-e7a0d86a3364.png)

## Creating Dockerfiles

### For the backend

- Create a file named Dockerfile in the `/backend-flask` dir
- Input the code below

![docker backend](https://user-images.githubusercontent.com/110903886/221422489-39ee3ce4-a12c-4649-8eea-ac8fec8d7de2.png)

```
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

Build Container

`docker build -t  backend-flask ./backend-flask`

Run Container with specified ENV

```
docker run --rm -p 4567:4567 -it backend-flask

docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask 
```

### For the Frontend

We have to run NPM Install before building the container since it needs to copy the contents of node_modules

```
cd frontend-react-js
npm i
```
- Create DockerFile in frontend-react-js
- Input the code below

```
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
Build Container
docker build -t frontend-react-js ./frontend-react-js
Run Container
docker run -p 3000:3000 -d frontend-react-js
```

### Docker-compose file

- Create docker-compose.yml at the root of your project.
- Paste the code below

```
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

Spin up the containers with `docker compose up`and check the urls


![docker compose up](https://user-images.githubusercontent.com/110903886/221422821-49413c8a-a15a-4012-90d6-4fc0ad8a4053.png)

![docker compose ports](https://user-images.githubusercontent.com/110903886/221422799-131545f5-7125-45f6-bca6-292c7071ba90.png)

You should get these responses

![json container output](https://user-images.githubusercontent.com/110903886/221422721-8a6109a9-5e88-41be-914a-4425d2df361e.png)

![3000](https://user-images.githubusercontent.com/110903886/221422734-37d58f01-a9fa-42d6-b437-b43475784556.png)




## Getting an image to Docker Hub

1. Log in on https://hub.docker.com/
2. Click on Create Repository.
3. Choose a name (e.g. verse_gapminder) and a description for your repository and click Create.
4. Log into the Docker Hub from the command line

`docker login`

Just with your own user name that you used for the account. Enter your password when prompted. If everything worked you will get a message similar to

WARNING: login credentials saved in /home/username/.docker/config.json

Login Succeeded![docker login](https://user-images.githubusercontent.com/110903886/221750961-e94f8006-89ee-4010-b0bc-5460d0d53e7a.png)

Check the image ID using

`docker images`

![docker images](https://user-images.githubusercontent.com/110903886/221751677-94d24066-b8b4-44e1-a180-5d5b54294823.png)

Tag your image

`docker tag 90dcc3be0d43 rockchip/aws_bootcamp2023:first-try`

Push the image

`docker pish rockchip/aws_bootcamp2023:first-try`

**IMAGE IN THE DOCKER HUB**

![docker hub](https://user-images.githubusercontent.com/110903886/221753490-e8af0462-8975-4fe0-b6e8-e4f6ac035920.png)

