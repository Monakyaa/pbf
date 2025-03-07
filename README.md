# DevOps - Struktur Proyek



## 🚀 Setup Docker dan Docker Compose

### 1. Instalasi Docker & Docker Compose

Pastikan Docker dan Docker Compose sudah terinstal:

#### **Ubuntu/Debian**:

```sh
sudo apt update && sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker
```

#### **Windows (WSL2) & macOS**:

Unduh dan instal dari [Docker Official Website](https://www.docker.com/get-started)

---

## 📂 Struktur Folder

Buat struktur berikut di dalam project utama:

```
project-docker/
│── backend/   # CodeIgniter Project
│── frontend/  # Laravel Project
│── docker/
│   ├── Dockerfile-php
│   ├── Dockerfile-nginx
│   ├── my.cnf
│── .env
│── docker-compose.yml
```

---

## 🛠️ Setup Docker untuk Backend (CodeIgniter) & Frontend (Laravel)

### **1️⃣ Dockerfile-php untuk PHP 8.3**

Buat file `docker/Dockerfile-php`:

```dockerfile
FROM php:8.3-fpm
RUN apt-get update && apt-get install -y \
    libpng-dev libjpeg-dev libfreetype6-dev zip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd pdo pdo_mysql
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
WORKDIR /var/www
```

### **2️⃣ Dockerfile-nginx untuk Nginx**

Buat file `docker/Dockerfile-nginx`:

```dockerfile
FROM nginx:latest
COPY default.conf /etc/nginx/conf.d/default.conf
```

### **3️⃣ Konfigurasi Nginx**

Buat file `docker/default.conf`:

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/public;
    index index.php index.html index.htm;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

---

## 📜 Konfigurasi `docker-compose.yml`

Buat file `docker-compose.yml` di dalam `project-docker/`:

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8
    container_name: mysql_container
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: mydatabase
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  backend:
    build:
      context: .
      dockerfile: docker/Dockerfile-php
    container_name: backend_container
    restart: always
    working_dir: /var/www
    volumes:
      - ./backend:/var/www
    depends_on:
      - mysql
    ports:
      - "9000:9000"

  frontend:
    build:
      context: .
      dockerfile: docker/Dockerfile-php
    container_name: frontend_container
    restart: always
    working_dir: /var/www
    volumes:
      - ./frontend:/var/www
    depends_on:
      - backend
    ports:
      - "5173:5173"

  nginx:
    build:
      context: .
      dockerfile: docker/Dockerfile-nginx
    container_name: nginx_container
    restart: always
    ports:
      - "8080:80"
    depends_on:
      - backend
      - frontend
    volumes:
      - ./backend:/var/www
      - ./docker/default.conf:/etc/nginx/conf.d/default.conf

volumes:
  mysql_data:
```

---

## 🔽 Clone Backend (CodeIgniter) & Frontend (Laravel)

### **Backend (CodeIgniter)**

```sh
git clone https://github.com/username/codeigniter-backend.git backend
cd backend
composer install
cp .env.example .env
php artisan key:generate
```

### **Frontend (Laravel)**

```sh
git clone https://github.com/username/laravel-frontend.git frontend
cd frontend
composer install
npm install
cp .env.example .env
php artisan key:generate
```

---

## 🚀 Menjalankan Docker Compose

Jalankan semua container dengan perintah:

```sh
docker-compose up -d --build
```

---

## 🔎 Cek Status Container

Pastikan semua container berjalan dengan:

```sh
docker ps
```

Akses **Backend**: [http://localhost:8080](http://localhost:8080)\
Akses **Frontend**: [http://localhost:5173](http://localhost:5173)

---

## 📤 Upload ke GitHub

Buat file `.gitignore` di `project-docker/` untuk menghindari file yang tidak perlu:

```
mysql_data/
backend/vendor/
frontend/node_modules/
frontend/vendor/
```

Lalu jalankan:

```sh
git init
git add .
git commit -m "Initial Docker setup"
git branch -M main
git remote add origin https://github.com/username/project-docker.git
git push -u origin main
```

---

## 🔄 Cara Menjalankan Setelah Update

Jika ada perubahan di lokal dan ingin update ke VPS:

```sh
git pull origin main
docker-compose up -d --build
```

Periksa log jika ada error:

```sh
docker logs backend_container
docker logs frontend_container
```

Restart container:

```sh
docker-compose restart
```

---


