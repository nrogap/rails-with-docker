# Rails with Docker
**rails** and **postgres** on **docker**

# Preparing
In case you don't clone this repository, and prefer to start everything by yourself. Please follow these step below.
## Create Rails Project
`rails new project-name --database=postgresql`

## Create .env file
```
POSTGRES_USER=nrogap
POSTGRES_PASSWORD=abcd1234
POSTGRES_HOST=db
```

## Edit database.yml file
```yml
# config/database.yml

default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: <%= ENV["POSTGRES_USER"] %>
  password: <%= ENV["POSTGRES_PASSWORD"] %>
  # For details on connection pooling, see rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: blog_development

test:
  <<: *default
  database: blog_test

production:
  <<: *default
  host: <%= ENV["POSTGRES_HOST"] %>
  database: blog_production
```

## Create Dockerfile
```Dockerfile
FROM ruby:2.5.1

RUN apt-get update -yqq \
  && apt-get install -yqq --no-install-recommends \
    nodejs \
    postgresql-client \
    && rm -rf /var/lib/apt/lists

WORKDIR /usr/src/app
COPY Gemfile* ./
RUN bundle install
COPY . .

EXPOSE 3000
CMD rails server -b 0.0.0.0
```

## Create docker-compose.yml
```yml
version: "3"

services:
  db:
    image: postgres:10
    env_file: .env

  app:
    build: .
    env_file: .env
    volumes:
      - .:/usr/src/app
    ports:
      - "3000:3000"
    depends_on:
      - db
```

# Setup Docker Container
1. Create and start `database` container in background mode `-d`
```
docker-compose up -d db 
```
2. Build `app` container
```
docker-compose build app
```
3. Run `db:create` and `db:migrate` on container `app`, and remove container after run `--rm`
```
docker-compose run --rm app rake db:create db:migrate
```
4. Create and start `app` container
```
docker-compose up app
```

# Knowledge Source
- [Deploying a Rails Application with Docker](https://www.youtube.com/watch?v=jlVrYgVEl6M)