# Docker Challenge
Tansima Kamal Fiana's final docker challenge

## Introduction
This repository is storing my work for the Docker challenges which is a part of our course on Operating Systems and Cloud Computing. The aim is to gain practical knowledge in Docker and container while implementing real-world applications that are closely to the content covered in the coursework.

# Challenge 3

## Installations
 - Install VsCode and set it up.
 - Install MySQL and set it up.
   - Following the default installation steps, you should end up with the MySQL username and password being 'root' and 'password'. For this project, it is set up as 'root' and 'fiona'. This will need to be changed when cloning the project.
 - Install Docker Desktop Client and set it up.
 - Install Node.js and set it up.
 - Install Github Desktop Client.

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

 - In VsCode, open the terminal by either navigating at the top of the toolbar to `View > Terminal` or by hititng the shortcut `Ctrl + \``
 - Run the commands
