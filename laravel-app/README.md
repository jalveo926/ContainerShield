# Caso 1 — Laravel Containerizado

Implementación de una aplicación **Laravel containerizada con Docker Compose**, incluyendo la configuración de la aplicación, base de datos MySQL y segmentación de redes mediante IPAM virtual.

## 📋 Descripción

Este caso corresponde al laboratorio práctico de **Laravel** del Parcial #1 de Redes.

El objetivo es desplegar una aplicación Laravel dentro de un entorno completamente containerizado, separando la aplicación y la base de datos en servicios independientes y utilizando redes Docker para controlar su comunicación.

La infraestructura busca aplicar principios de **aislamiento y mínimo privilegio**, evitando exponer directamente servicios internos que no necesitan acceso desde el exterior.

## 🏗️ Arquitectura

```text
             Cliente
                │
                ▼
        ┌───────────────┐
        │    Laravel    │
        │      App      │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │     MySQL     │
        │   Database    │
        └───────────────┘
```

La aplicación Laravel y la base de datos se ejecutan en contenedores independientes y se comunican mediante una red Docker interna.

## 🛠️ Tecnologías

* Laravel
* PHP 8.4
* Docker
* Docker Compose
* MySQL
* Docker Networking
* IPAM virtual

## 📁 Estructura

```text
Caso-Laravel/
├── app/
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── composer.json
└── README.md
```

> La estructura puede variar dependiendo de los archivos adicionales utilizados por la aplicación.

## 🐳 Dockerfile

El `Dockerfile` define la imagen utilizada para ejecutar Laravel.

Entre las principales configuraciones se encuentran:

* Imagen base `php:8.4-cli`.
* Instalación de extensiones PHP necesarias como `pdo_mysql` y `zip`.
* Instalación de Composer.
* Instalación de las dependencias del proyecto mediante `composer install`.
* Exposición del puerto `8000`.
* Configuración del comando necesario para ejecutar la aplicación.

## 🌐 Red e IPAM

Docker Compose utiliza una red virtual configurada mediante **IPAM (IP Address Management)**.

La utilización de una red definida permite controlar el direccionamiento de los contenedores y establecer una comunicación aislada entre los servicios que forman parte de la aplicación.

La base de datos permanece dentro de la infraestructura interna y no necesita ser expuesta directamente al host.

## ⚙️ Docker Compose

El archivo `docker-compose.yml` define los servicios necesarios para ejecutar el laboratorio.

Los principales componentes son:

* **app:** contenedor encargado de ejecutar Laravel.
* **db:** contenedor encargado de ejecutar MySQL.
* **network:** red virtual utilizada para la comunicación entre los servicios.

## 🚀 Ejecución

### 1. Construir y levantar los servicios

```bash
docker compose up -d --build
```

### 2. Verificar los contenedores

```bash
docker compose ps
```

### 3. Ejecutar las migraciones

```bash
docker compose exec app php artisan migrate
```

Este comando ejecuta las migraciones de Laravel **dentro del contenedor `app`**, utilizando la configuración de conexión definida para comunicarse con el contenedor de MySQL.

### 4. Ver los logs

```bash
docker compose logs
```

Para consultar únicamente los logs de Laravel:

```bash
docker compose logs app
```

## 🧪 Verificación

Para comprobar que los servicios están funcionando correctamente:

```bash
docker compose ps
```

Los contenedores definidos por el proyecto deberían aparecer en estado `running`.

También
