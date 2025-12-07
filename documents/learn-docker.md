# Docker Learning Guide for Beginners

## Table of Contents
1. [What is Docker?](#what-is-docker)
2. [Why Use Docker?](#why-use-docker)
3. [Key Docker Concepts](#key-docker-concepts)
4. [Docker Installation](#docker-installation)
5. [Understanding Dockerfile](#understanding-dockerfile)
6. [Understanding docker-compose.yml](#understanding-docker-composeyml)
7. [Basic Docker Commands](#basic-docker-commands)
8. [Working with Laravel and Docker](#working-with-laravel-and-docker)
9. [Common Tasks](#common-tasks)
10. [Troubleshooting](#troubleshooting)

---

## What is Docker?

**Docker** is a platform that allows you to package your application and all its dependencies into a **container**. Think of a container like a lightweight, portable box that contains everything your application needs to run.

### Real-World Analogy
Imagine you're moving to a new house. Instead of packing each item separately and hoping everything works in the new place, Docker is like packing everything into a container that you know will work exactly the same way anywhere you move it.

---

## Why Use Docker?

### Problems Docker Solves:
1. **"It works on my machine"** - Docker ensures your app runs the same way on any computer
2. **Environment Setup** - No need to manually install PHP, MySQL, Redis, etc.
3. **Isolation** - Each application runs in its own container, separate from others
4. **Easy Deployment** - Deploy the same container to development, staging, and production
5. **Team Collaboration** - Everyone uses the same environment

### Benefits:
- ✅ Consistent development environment
- ✅ Easy to share and deploy
- ✅ Isolated from your main system
- ✅ Can run multiple versions of software simultaneously

---

## Key Docker Concepts

### 1. **Image** 📦
An **image** is a read-only template used to create containers. It's like a blueprint or recipe.

**Example:** `php:8.2-cli` is an image that contains PHP 8.2

### 2. **Container** 🚀
A **container** is a running instance of an image. It's the actual running application.

**Analogy:** 
- **Image** = Recipe (blueprint)
- **Container** = The actual cake (running application)

### 3. **Dockerfile** 📝
A text file with instructions on how to build an image. It's like a recipe for creating your container.

### 4. **docker-compose.yml** 🎼
A file that defines and runs multiple containers together. It's like a conductor's score for an orchestra.

### 5. **Volume** 💾
A way to share files between your computer and the container. Changes on your computer are reflected in the container.

---

## Docker Installation

### For macOS:
```bash
# Install Docker Desktop
# Download from: https://www.docker.com/products/docker-desktop
```

### Verify Installation:
```bash
docker --version
docker-compose --version
```

---

## Understanding Dockerfile

Let's break down the Dockerfile in this project:

```dockerfile
FROM php:8.2-cli
```
**What it does:** Starts with a base image that has PHP 8.2 installed.
- `FROM` = "Start with this base image"
- `php:8.2-cli` = Official PHP 8.2 image

```dockerfile
RUN apt-get update && apt-get install -y \
    git zip unzip curl \
    libzip-dev libpng-dev libonig-dev libxml2-dev \
    && docker-php-ext-install pdo_mysql mbstring zip exif pcntl
```
**What it does:** Installs system packages and PHP extensions needed for Laravel.
- `RUN` = "Execute this command"
- Installs tools like git, zip, curl
- Installs PHP extensions (pdo_mysql, mbstring, etc.)

```dockerfile
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
```
**What it does:** Copies Composer (PHP package manager) into the container.
- `COPY --from` = "Copy from another image"
- Gets Composer from the official Composer image

```dockerfile
WORKDIR /app
```
**What it does:** Sets `/app` as the working directory (where commands run).
- All subsequent commands will run in `/app` folder
- This is why we use `/app` in docker-compose.yml

```dockerfile
COPY . /app
```
**What it does:** Copies all project files into the container.
- `.` = Current directory (your project)
- `/app` = Destination in container

```dockerfile
RUN composer install --no-interaction --prefer-dist --optimize-autoloader
```
**What it does:** Installs PHP dependencies using Composer.

```dockerfile
EXPOSE 8000
```
**What it does:** Tells Docker that the container will listen on port 8000.
- Doesn't actually open the port (that's done in docker-compose.yml)

```dockerfile
CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
```
**What it does:** Command that runs when container starts.
- Starts Laravel development server
- `0.0.0.0` = Listen on all network interfaces (accessible from outside container)

---

## Understanding docker-compose.yml

Let's break down the docker-compose.yml file:

```yaml
services:
  laravel:
```
**What it does:** Defines a service named "laravel".
- `services` = List of containers to run
- `laravel` = Name of our service

```yaml
build:
  context: .
  dockerfile: Dockerfile
```
**What it does:** Tells Docker how to build the image.
- `context: .` = Use current directory as build context
- `dockerfile: Dockerfile` = Use the Dockerfile in current directory

```yaml
container_name: my-laravel-app
```
**What it does:** Gives the container a specific name.
- Makes it easier to reference: `docker exec my-laravel-app ...`

```yaml
ports:
  - "8000:8000"
```
**What it does:** Maps port 8000 from container to port 8000 on your computer.
- Format: `"host:container"`
- Access your app at `http://localhost:8000`

```yaml
volumes:
  - .:/app
  - /app/vendor
  - /app/storage
```
**What it does:** Syncs files between your computer and container.

- `.:/app` = Sync current directory (`.`) with `/app` in container
  - **Auto-reload:** Changes on your computer instantly appear in container!
  
- `/app/vendor` = Anonymous volume for Composer packages
  - Keeps vendor folder in container (faster, avoids permission issues)
  
- `/app/storage` = Anonymous volume for Laravel storage
  - Keeps storage files in container

```yaml
environment:
  - APP_ENV=local
  - APP_DEBUG=true
```
**What it does:** Sets environment variables for Laravel.
- `APP_ENV=local` = Development environment
- `APP_DEBUG=true` = Show detailed error messages

---

## Basic Docker Commands

### Building and Running

```bash
# Build the image
docker-compose build

# Start containers
docker-compose up

# Start in background (detached mode)
docker-compose up -d

# Build and start
docker-compose up --build

# Stop containers
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

### Viewing Information

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View logs
docker-compose logs

# Follow logs (like tail -f)
docker-compose logs -f

# View logs for specific service
docker-compose logs laravel
```

### Executing Commands

```bash
# Run command in container
docker-compose exec laravel php artisan migrate

# Open shell in container
docker-compose exec laravel bash

# Run one-time command
docker-compose run laravel php artisan make:controller UserController
```

### Managing Images and Containers

```bash
# List images
docker images

# Remove unused images
docker image prune

# Remove stopped containers
docker container prune

# View container resource usage
docker stats
```

---

## Working with Laravel and Docker

### Initial Setup

1. **Start the container:**
   ```bash
   docker-compose up --build
   ```

2. **Install dependencies (if needed):**
   ```bash
   docker-compose exec laravel composer install
   ```

3. **Copy environment file:**
   ```bash
   cp .env.example .env
   ```

4. **Generate application key:**
   ```bash
   docker-compose exec laravel php artisan key:generate
   ```

5. **Run migrations:**
   ```bash
   docker-compose exec laravel php artisan migrate
   ```

### Development Workflow

1. **Make changes to your code** (PHP, Blade templates, etc.)
2. **Changes automatically sync** to container (thanks to volume mount)
3. **Refresh browser** - changes are live!

**No need to restart the container!** 🎉

### Common Laravel Commands

```bash
# Create controller
docker-compose exec laravel php artisan make:controller PostController

# Create model
docker-compose exec laravel php artisan make:model Post

# Run migrations
docker-compose exec laravel php artisan migrate

# Rollback migration
docker-compose exec laravel php artisan migrate:rollback

# Clear cache
docker-compose exec laravel php artisan cache:clear
docker-compose exec laravel php artisan config:clear
docker-compose exec laravel php artisan view:clear

# Run tests
docker-compose exec laravel php artisan test
```

---

## Common Tasks

### Task 1: First Time Setup
```bash
# 1. Build and start
docker-compose up --build -d

# 2. Install dependencies
docker-compose exec laravel composer install

# 3. Setup environment
cp .env.example .env
docker-compose exec laravel php artisan key:generate

# 4. Run migrations
docker-compose exec laravel php artisan migrate
```

### Task 2: View Application Logs
```bash
# View all logs
docker-compose logs

# Follow logs in real-time
docker-compose logs -f laravel

# View last 100 lines
docker-compose logs --tail=100 laravel
```

### Task 3: Access Container Shell
```bash
# Open bash shell
docker-compose exec laravel bash

# Now you're inside the container!
# You can run: ls, cd, php artisan, etc.
```

### Task 4: Install New PHP Package
```bash
# Install package
docker-compose exec laravel composer require package/name

# Update autoloader
docker-compose exec laravel composer dump-autoload
```

### Task 5: Database Access
If you add a MySQL service to docker-compose.yml:
```bash
# Access MySQL
docker-compose exec mysql mysql -u root -p

# Or use Laravel's tinker
docker-compose exec laravel php artisan tinker
```

---

## Troubleshooting

### Problem: Container won't start
```bash
# Check logs
docker-compose logs laravel

# Rebuild from scratch
docker-compose down -v
docker-compose up --build
```

### Problem: Port 8000 already in use
```bash
# Change port in docker-compose.yml
ports:
  - "8001:8000"  # Use 8001 on your computer
```

### Problem: Permission denied errors
```bash
# Fix permissions
docker-compose exec laravel chmod -R 777 storage bootstrap/cache
```

### Problem: Changes not reflecting
```bash
# Restart container
docker-compose restart laravel

# Or rebuild
docker-compose up --build
```

### Problem: Container keeps stopping
```bash
# Check what's wrong
docker-compose logs laravel

# Run container interactively to see errors
docker-compose run laravel bash
```

### Problem: Out of disk space
```bash
# Clean up unused Docker resources
docker system prune -a

# Remove specific image
docker rmi image-name
```

---

## Key Takeaways

1. **Dockerfile** = Recipe for building your image
2. **docker-compose.yml** = Configuration for running containers
3. **Volumes** = Sync files between computer and container (auto-reload!)
4. **Ports** = Map container ports to your computer
5. **Environment Variables** = Configure your application

### Best Practices:
- ✅ Use volumes for code (auto-reload)
- ✅ Use anonymous volumes for dependencies (vendor, node_modules)
- ✅ Keep Dockerfile instructions in logical order
- ✅ Use `.dockerignore` to exclude unnecessary files
- ✅ Document your Docker setup

---

## Next Steps

1. **Practice:** Try modifying the Dockerfile and see what happens
2. **Experiment:** Add a MySQL service to docker-compose.yml
3. **Learn More:** 
   - Docker official docs: https://docs.docker.com/
   - Laravel Docker: https://laravel.com/docs/docker

---

## Quick Reference Card

```bash
# Start
docker-compose up -d

# Stop
docker-compose down

# Logs
docker-compose logs -f

# Execute command
docker-compose exec laravel php artisan [command]

# Shell access
docker-compose exec laravel bash

# Rebuild
docker-compose up --build
```

---

**Happy Docker Learning! 🐳**

