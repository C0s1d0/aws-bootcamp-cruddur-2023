# Week 1 — App Containerization

## Backend Container

### Create Dockerfile for backend-flask

FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

#set environment variable
ENV FLASK_ENV=development

EXPOSE ${PORT}
#python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

### Build Container 
docker build -t  backend-flask ./backend-flask

### Run Container
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask


## Frontend Container

### Install Contents for node_modules

cd frontend-react-js;npm i

### Create docker file
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]

### Build Container
docker build -t frontend-react-js ./frontend-react-js

### Run Container
docker run -p 3000:3000 -d frontend-react-js

## Create docker-compose file
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

#### the name flag is a hack to change the default prepend folder
#### name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur

