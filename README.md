# Docker enviroment for Laravel

This enviroment expects the Laravel project to be in `src`

## Create a new laravel project 

In order to create a new Laravel project we can use this command `docker compose run --rm app laravel new project`, however the project will be created inside `src` giving you a nested folder, that must be changed. So in order to fix it take the project out of the `src` folder and placed on root and then replace the `src` folder with the `project` directory, as seen below.

```bash
# IMPORTANT! the created project must be manually put in the root directory as src
docker compose run --rm app laravel new project
# Replacing the project (you may need to change ownership)
mv src/project ./ 
rm -R src
mv project src
# Build containers
docker compose up -d --build
docker compose exec app php artisan key:generate
docker compose exec app php artisan migrate
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
```

The application should be available on:

- Laravel: `http://localhost:8080`
- Vite: `http://localhost:5173`
- Mailpit: `http://localhost:8025`

Unless the `.env` has been changed, in which case adapt the ports

You also need to change the vite config file for the assets to correctly load, so in order to do that, add the following to `vite.config.js`:

```js
    server: {
            host: '0.0.0.0',
            hmr: {
                host: 'localhost',
            },
            watch: {
                usePolling: true, // Crucial for detecting changes in shared folder on Docker
            },
        },
    });
```

## Use a project with an existent Laravel project

Copy the entire project to the `src` directory and then modify the `.env` file according to your needs and then run:

```bash
docker compose up -d --build
docker compose exec app composer install
docker compose exec app php artisan key:generate
docker compose exec app php artisan migrate
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
```

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