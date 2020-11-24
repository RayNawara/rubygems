# README

`docker run --rm -it -v ${PWD}:/usr/src -w /usr/src ruby:2.7 sh -c 'gem install rails:"~> 6.0.3" && rails new --skip-test Rails6_blueprint'`

Copy Dockerfile, docker-compose.yml and docker-entrypoint.sh

```
# Dockerfile

FROM ruby:2.7

# Install nodejs
RUN apt-get update -qq && apt-get install -y nodejs

# Add Yarn repository
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

# Update
RUN apt-get update -y

# Install Yarn
RUN apt-get install yarn -y

ADD . /usr/src/app
WORKDIR /usr/src/app

# Install & run bundler
RUN gem install bundler:'~> 2.1.4'

RUN bundle

CMD ./docker-entrypoint.sh
```

```
# docker-compose.yml

version: '3.2'

volumes:
  dbdata:
    driver: local

services:
  db:
    image: postgres:11
    environment:
      - PGDATA=/var/lib/postgresql/data/pgdata
      - POSTGRES_USER=rails
      - POSTGRES_PASSWORD=secret123
    volumes:
      - dbdata:/var/lib/postgresql/data/pgdata

  web:
    build: .
    ports:
      - '3000:3000'
    environment:
      - RAILS_ENV=development
      - RACK_ENV=development
      - POSTGRES_USER=rails
      - POSTGRES_PASSWORD=secret123
    volumes:
      - .:/usr/src/app
    depends_on:
      - db
```

docker-entrypoint.sh

```
#!/bin/sh

rm -f tmp/pids/server.pid
bin/rails server -b 0.0.0.0
```

Then get ownership
```
sudo chown -R $USER:$USER .

chmod u+x docker-entrypoint.sh

docker build -t rails6 .

docker-compose build

docker-compose run --rm web bundle exec rake webpacker:install
```

...And finally to check that everything has worked as intended:

```
docker-compose up -d db
docker-compose up web
```

web_1  | => Booting Puma
...
web_1  | Use Ctrl-C to stop

Now update your docker-compose.yml for auto-reloading

```
# docker-compose.yml

version: '3.2'

volumes:
  dbdata:
    driver: local

services:
  db:
    image: postgres:11
    environment:
      - PGDATA=/var/lib/postgresql/data/pgdata
      - POSTGRES_USER=rails
      - POSTGRES_PASSWORD=secret123
    volumes:
      - dbdata:/var/lib/postgresql/data/pgdata

  web:
    build: .
    ports:
      - '3000:3000'
    environment:
      RAILS_ENV: development
      RACK_ENV: development
      POSTGRES_USER: rails
      POSTGRES_PASSWORD: secret123
      WEBPACKER_DEV_SERVER_HOST: webpack_dev_server
    volumes:
      - .:/usr/src/app
    depends_on:
      - db
      - webpack

  webpack:
    build: .
    command: ./bin/webpack-dev-server
    volumes:
      - .:/usr/src/app
    ports:
      - '3035:3035'
    environment:
      NODE_ENV: development
      RAILS_ENV: development
      WEBPACKER_DEV_SERVER_HOST: 0.0.0.0
```

You should be good! Check localhost:3000. Everytime I run a scaffold or something it is set to root so I have to 

```
sudo chown -R $USER:$USER .
```

To start a new project clone this repo

To enter the container - 

```
docker exec -it rubygems_web_1 /bin/bash
```

update node and add npm 

```
curl -sL https://deb.nodesource.com/setup_14.x | bash

sudo apt-get install -y nodejs
```
