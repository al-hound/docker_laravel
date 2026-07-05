# Docker enviroment for Laravel

This enviroment expects the Laravel project to be in `src`

## Setup the Laravel project 

In order to setup a Laravel project we have 3 options:

### Use Laravel installer

In order to use the Laravel installer use this command `docker compose run --rm app laravel new project`, however the project will be created inside `src` giving you a nested folder, that must be changed. So in order to fix it take the project out of the `src` folder and placed on root and then replace the `src` folder with the `project` directory, as seen below. This happens because of how the installer works, and I coudln't find a better solution, if a better way is found, then this part will be updated.

```bash
# IMPORTANT! If you want to create the project with the installer, the created project must be manually put in the root directory as src
docker compose run --rm app laravel new project
# Replacing the project (you may need to change ownership)
mv src/project ./ 
rm -R src
mv project src
```

### Create a basic Laravel project

If you don't need to use the installer, and only need a basic project with no modules instaled, then you can use the basic command, it will create a basic project with no modules, but you don'need to do anything else, as the installation will be done directly into the src folder. 

```bash
# If you don't need the installer and want to create a fresh project, then only run this command and it will work correctly 
docker compose run --rm app composer create-project laravel/laravel .
```

## Use an existing Laravel project

Copy the entire project to the `src` directory and then modify the `.env` file according to your needs and then run:

```bash
docker compose up -d --build
docker compose exec app composer install
```

## Deploy the enviroment 

Once the Laravel project has been placed on the `src` folder, then you need to modify a couple of things in order to make it work.

First of all, modify the `.env` inside `src`, which is the Laravel project, by default Laravel uses sqlite, so you need to adapt the file according to the Variables declared on the docker `.env` file, so it should be something like this:

```.env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=admin
DB_PASSWORD=secret
```
You also need to change the vite config file for the assets to correctly load, so in order to do that, add the following to `vite.config.js`:

```js
    server: {
        host: '0.0.0.0',
        hmr: {
            host: 'localhost',
        },
        watch: {
            usePolling: true, // Crucial for detecting changes in shared folder on Docker
            //....
        },
    },
```

Once all the configuration has been modified, then you must build the containers, change permissions and install the npm modules

```bash
# Build containers
docker compose up -d --build
docker compose exec app php artisan key:generate
docker compose exec app php artisan migrate
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
docker compose run --rm vite npm install
```

The application should be available on:

- Laravel: `http://localhost:8080`
- Vite: `http://localhost:5173`
- Mailpit: `http://localhost:8025`

Unless the `.env` has been changed, in which case adapt the ports

## Multiple instances of the same compose

If you are going to have more than one project running at the same time, then you need to change the ports in order to avoid conflicts, so modify the `.env` file, you have a comment that indicates which variables need to be changed in order to solve any potencial conflict

## Useful commands

Use `docker compose exec app` if you want to execute commands on the laravel app.

```bash
docker compose exec app php artisan migrate
docker compose exec app php artisan tinker
docker compose exec app composer require vendor/package
docker compose run --rm node npm install
docker compose down
docker compose down -v
```

`docker compose down -v` deletes also all volumes from MySQL and Redis.