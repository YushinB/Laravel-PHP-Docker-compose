# The target setup
this project need alot of setup step to accomplish, that the reason this project is good example of using docker to hander complicated setup.
The below image show the structure of our Laravel setup, it need totaly 6 containers, 3 apps container and 3 utility containers. <br>
![image](https://user-images.githubusercontent.com/34083808/184526111-47ab8a52-face-4057-8e15-ac8206e62dda.png)

```yaml
version: '3.8'

services:
  server:
    # image: 'nginx:stable-alpine'
    build:
      context: .
      dockerfile: dockerfiles/nginx.dockerfile
    ports:
      - '8000:80'
    volumes:
      - ./src:/var/www/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - php
      - mysql
  php:
    build:
      context: .
      dockerfile: dockerfiles/php.dockerfile
    volumes:
      - ./src:/var/www/html:delegated
  mysql:
    image: mysql:5.7
    env_file:
      - ./env/mysql.env
  composer:
    build:
      context: ./dockerfiles
      dockerfile: composer.dockerfile
    volumes:
      - ./src:/var/www/html
  artisan:
    build:
      context: .
      dockerfile: dockerfiles/php.dockerfile
    volumes:
      - ./src:/var/www/html
    entrypoint: ['php', '/var/www/html/artisan']
  npm:
    image: node:14
    working_dir: /var/www/html
    entrypoint: ['npm']
    volumes:
      - ./src:/var/www/html

```

at first, we need to create our laravel source code by using composer utility container.
```
docker-compose run --rm composer create-project --prefer-dist laravel/laravel .
```

after that, we need to change the eviroment variable of the laravel source code in ```./src/.env``` to our database container.
```
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

next step we will start out application by docker compose. 
```
docker-compose up --rm -d server php mysql
```
> if there is error related to permision of vendor folder, please delete this folder and run the composer update command ``` docker-compose run --rm composer update```

We can also just run only the server container by adding the dependency in docker compose file. 
```
    depends_on:
      - php
      - mysql
```
and then run the command ```docker-compose up --rm -d server```

## Run docker utility artisan to migrate the database. 
Please check the mysql password and username in .env file and also make sure to run ather 3 app container. 
```
docker-compose run --rm artisan migrate
```
> if there are any problems with migration you can you below command to refresh.
```
docker-compose run --rm artisan config:cache
docker-compose run --rm artisan config:clear
docker-compose run --rm artisan cache:clear
```

after finish all setup step, you shoud see the below page. <br>
![image](https://user-images.githubusercontent.com/34083808/184610364-b71ea362-00cb-4e6f-972b-6b97616e2b67.png)

