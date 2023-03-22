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


### Adding DynamoDB Local and Postgres

We are going to use Postgres and DynamoDB local in future labs We can bring them in as containers and reference them externally

Lets integrate the following into our existing docker compose file:


**Postgres**

```
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
```

```
volumes:
  db:
    driver: local
```

- Install the postgres client into Gitpod in the `gitpod.yml` file.

```
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

**DynamoDB Local**

```
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

## Homework Challenges

### Getting an image to Docker Hub

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

`docker push rockchip/aws_bootcamp2023:first-try`

**IMAGE IN THE DOCKER HUB**

![docker hub](https://user-images.githubusercontent.com/110903886/221753490-e8af0462-8975-4fe0-b6e8-e4f6ac035920.png)



## Running Docker on an EC2 Instance

- Launch an EC2 instance - Since I'm on Free tier, I'd be using a free tier supported AMI

![new instance](https://user-images.githubusercontent.com/110903886/221769774-40811017-1ecf-4b8d-91ff-6f3a6b3d04f7.png)

- Install the Docker Engine 

**Set up the repository**

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:

```
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
````

2. Add Docker’s official GPG key:

```
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

3. Use the following command to set up the repository:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. Update the apt package index:

`sudo apt-get update`

5. Install Docker Engine, containerd, and Docker Compose.

`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

- Pull an image from docker hub registry and run the container

`docker pull rockchip/aws_bootcamp2023:first-try`

![ec2 1](https://user-images.githubusercontent.com/110903886/221769979-8cd95524-6c2a-44c1-adf0-12149ae99a72.png)

Check if it runs on the Ec2 public IP

![ec2](https://user-images.githubusercontent.com/110903886/221770082-818e4026-ee8a-43b7-8c35-8861a801668f.png)


## Creating the Backend & Front Notification Feature! - [https://youtu.be/k-_o0cCpksk]


Add a new path to the openapi.yml file and edit as instructed in the video above

![addnewpath](https://user-images.githubusercontent.com/110903886/222161837-893bb90c-b8e0-4db6-8764-cb3d1723d00b.png)

```
  /api/activities/notifications:
    get:
      description: 'Return a feed of activity for all the people i follow'
      tags:
        - activities
        
      parameters: []
      responses:
        '200':
          description: Returns an array of activities
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Activity'
  ```

Create new file called notificattion_activities in the backend-flask/services dir

```
from datetime import datetime, timedelta, timezone
class NotificationsActivities:
  def run():
    now = datetime.now(timezone.utc).astimezone()
    results = [{
      'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
      'handle':  'Guess it worked',
      'message': 'I am a DevOps Engineer',
      'created_at': (now - timedelta(days=2)).isoformat(),
      'expires_at': (now + timedelta(days=5)).isoformat(),
      'likes_count': 5,
      'replies_count': 1,
      'reposts_count': 0,
      'replies': [{
        'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
        'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  'Worf',
        'message': 'This post has no honor!',
        'likes_count': 0,
        'replies_count': 0,
        'reposts_count': 0,
        'created_at': (now - timedelta(days=2)).isoformat()
      }],
    },
   
    ]
    return results
  ```


Edit the codes from app.py and 
Test it out if it works

![notiffication json](https://user-images.githubusercontent.com/110903886/221773350-64260f13-a518-4037-a4aa-cd4a4564c6db.png)
