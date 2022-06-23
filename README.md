# Rails 7 docker template with postgres and rspec

## Generate new rails app

To generate a new rails app with postgres and without minitest, run:

```shell
docker-compose run --no-deps web rails new . --force --database=postgresql -T
```

If you work with minitest, you can remove the `-T` flag from the command.

This is what happens when you run this command:

1. It builds an image based on the Dockerfile
2. It creates a new container based on that image
3. Runs `rails new` inside the container

`rails new` updates the Gemfile, so we will need to build the image again:

```shell
docker-compose build
```

## Adjust the `config/database.yml`

Adjust the `config/database.yml` to docker and the postgres image. It should
look like this:

```
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password: password
  pool: 5

development:
  <<: *default
  database: myapp_development


test:
  <<: *default
  database: myapp_test
```

Create the database:

```shell
docker-compose run web rake db:create
```

## Install rspec

To install rspec, add the 'rspec-rails' gem to your Gemfile:

```shell
group :development, :test do
  gem 'rspec-rails', '~> 5.0.0'
end
```

Followed by the install commands:

```shell
docker-compose run web bundle install
docker-compose run web rails generate rspec:install
```

Finally, rebuild your docker image based on the Gemfile changes:

```shell
docker-compose build
```

To start the tdd container:

```shell
docker-compose -f docker-compose.tdd.yml up
```

To run it in detach mode add the `--detach` flag.

To run the tests:

```shell
docker-compose -f docker-compose.tdd.yml run tdd rspec spec
```

## Install Tailwind CSS:

```shell
docker-compose run web bundle add tailwindcss-rails
docker-compose run web rails tailwindcss:install
```

## Boot the app

```shell
docker-compose up
```

This will:

1. Create a network to connect the app (web) container with the db (postgres)
   container
2. Create the postgres container
3. Create the web container
4. Start both containers

References: [Docker's sample rails and postgresql application](https://docs.docker.com/samples/rails/)
