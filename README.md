# Docker Challenge
Tansima Kamal Fiana's final docker challenge

## Introduction
This repository is storing my work for the Docker challenges which is a part of our course on Operating Systems and Cloud Computing. The aim is to gain practical knowledge in Docker and container while implementing real-world applications that are closely to the content covered in the coursework.

# Challenge 3

## Installations
 - Install [VsCode](https://code.visualstudio.com/) and set it up.
 - Install [MySQL](https://www.mysql.com/) and set it up.
   - Following the default installation steps, you should end up with the MySQL username and password being 'root' and 'password'. For this project, it is set up as 'root' and 'fiona'. This will need to be changed when cloning the project.
 - Install [Docker Desktop Client](https://www.docker.com/products/docker-desktop/) and set it up.
 - Install [Node.js](https://nodejs.org/en) and set it up.
 - Install [Github Desktop Client](https://desktop.github.com/).

Launch all of the programs you installed to make sure they are initialized on your desktop before proceeding.

## Cloning the Base Repository
- To clone the base template repository, first create a new repository in your github.com account. After successfully creating it and retrieving the link to clone, navigate here and clone the files (either by downloading the .zip or cloning with the URL) into your new repository.
 - Make sure to include a README.md file in the project root folder that gives descriptive notes about what your project is about. (Like this one!)
 - Populate the student.cfg file with your student information.

 - Make sure that after each major (or minor) changes, you commit and push to your repository using the Github Desktop Client.

## Structuring Filepaths and Environment Variables
 - Navigate to docker-challenge-#/db/Dockerfile and make sure the first argument in the COPY command points to the .sql file in ./init/init.sql.
 - Navigate to docker-challenge-#/api/Dockerfile and make sure the first argiment in the COPY commands point to:
   - ./package.json
   - ./package-lock.json
   - ./
 - Create a .env file in the docker-challenge-3/ and populate it as followed:
 > [!NOTE]
 > The username and password may be different depending on how MySQL was set up initially.
 > Moreover, both segments (DB and MYSQL) should have matching fields.

```env
DB_ROOT_PASSWORD=password
DB_DATABASE=db
DB_USERNAME=root
DB_PASSWORD=fiona
DB_HOST=db

MYSQL_ROOT_PASSWORD=password
MYSQL_DATABASE=db
MYSQL_USER=root
MYSQL_PASSWORD=password
MYSQL_HOST=db
```

 - Navigate to nginx/nginx.conf and add this at the top of the file:
```conf
 + events {
 +   worker_connections 1024;
 + }

upstream loadbalancer {
...
```

 - Create a new file called `docker-compose.yml` in the project root and populate it with the following code
```yml
version: '3.8'
services:
  db:
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_HOST: ${MYSQL_HOST}
    volumes:
      - ./db/init/:/docker-entrypoint-initdb.d
      - db_data:/var/lib/mysql
    networks:
      - backend

  node-service:
    build: ./api
    restart: always
    environment:
      DB_HOST: ${DB_HOST}
      DB_USERNAME: ${DB_USERNAME}
      DB_PASSWORD: ${DB_PASSWORD}
      DB_DATABASE: ${DB_DATABASE}
      DB_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    depends_on:
      - db
    networks:
      - backend

  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - node-service
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    networks:
      - backend

volumes:
  db_data:

networks:
  backend:
```

## Populating the Database
 - In VsCode, open the terminal by either navigating at the top of the toolbar to `View > Terminal` or by hititng the shortcut `Ctrl + \``
 - Run the command `docker-compose up -d` and wait for the process to finish.
 - Run the command `docker-container ls`
 - Copy the CONTAINER ID of the `mariadb` IMAGE
 - Run the command `docker exec -it <CONTAINER_ID> /bin/bash` (eg. `docker exec -it 7grew9erwr9 /bin/bash`)
 - Run the command `mariadb -ppassword` (In my case, it's `mariadb -pfiona`)
 - Run the command `use db`
 - Run these SQL commands (found in db/init/init.sql):

```SQL
CREATE TABLE books (
 id INT AUTO_INCREMENT PRIMARY KEY,
 title VARCHAR(100) NOT NULL,
 author VARCHAR(255) NOT NULL
);

INSERT INTO books (id, title, author) VALUES (1, 'To Kill a Mockingbird', 'Harper Lee');

INSERT INTO books (id, title, author) VALUES (2, '1984', 'George Orwell');

INSERT INTO books (id, title, author) VALUES (3, 'Pride and Prejudice', 'Jane Austen');

INSERT INTO books (id, title, author) VALUES (4, 'The Great Gatsby', 'F. Scott Fitzgerald');
```

## Viewing the Database in Your Browser
 - In the VsCode Terminal run the commands `docker-compose down --rmi all` and after the process is finished, run `docker-compose up -d` and wait for the process to be done.
 - Open your browser and navigate to the URL `localhost:8080/api/status`, the output should be similar to this:
```json
{
 "status": "success",
 "contents": {
  "MemFree": 1234567,
  "MemAvaliable": 1234567
 },
 "pid": 1,
 "hostname": "1a2b3c4d5e"
 "counter": 0
}
```

 - Navigate to the URL `localhost:8080/api/books`, the output should be this:
```json
[
 {
  "id": 1,
  "title": "To Kill a Mockingbird",
  "author": "Harper Lee"
 },
 {
  "id": 2,
  "title": "1984",
  "author": "George Orwell"
 },
 {
  "id": 3,
  "title": "Pride and Prejudice",
  "author": "Jane Austen"
 },
 {
  "id": 4,
  "title": "The Great Gatsby",
  "author": "F. Scott Fitzgerald"
 }
]
```

 - Navigate to the URL `localhost:8080/api/books/#`, where # is the ID of a book (eg. `localhost:8080/api/books/1`). The output is the individual books matching with their ID.

# Challenge 4
In this challenge, the node service needs to be scaled up to three instances. Before do this, let's understand what happens with just one instance.

## Before Scaling Up
 - Navigate to `localhost:8080/api/stats` and view the hostname. Refresh the page and see what happens
  - The hostname remains the same
 - Take down the docker and rebuild it by opening the VsCode Terminal and running `docker-compose down --rmi all` and then `docker-compose up -d`.
 - Navigate back to `localhost:8080/api/stats` and view the hostname. The hostname should have changed.

This shows that the user is using only one instance, which is the same instance. This may be an issue if the service goes down and users need to access the services. For this, the services should be scaled up to allow users to automatically be redirected to other instances.

## After Scaling Up
 - Decompose the docker container with `docker-compose down --rmi all` and wait for the process to finish.
 - Run the command `docker-compose up --build --scale node-service=3` and view the output.
  - You should notice the console outputting logs from three individual node services.
 - Check the Docker Desktop Client and view the instances in the container.
 - Navigate to `localhost:8080/api/stats` and view the output.
  - Refresh the page and view the output. Do this two more times.
  - You should notice that the hostnames cycle through all 3 instances in a cycle.

If one of the instances would be down, we would still be able to access the other two instances, allowing us to resume our work.
