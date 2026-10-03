# Redes y Despliegue Containerizado 

Implementación y documentación de laboratorios prácticos de **Laravel** y **Traefik** utilizando Docker y Docker Compose, con énfasis en **segmentación de redes mediante IPAM, aislamiento de servicios, proxy inverso y balanceo de carga**.

## Tecnologías

* Docker
* Docker Compose
* Laravel
* PHP
* MySQL
* Traefik
* Docker Bridge Networks
* IPAM
* Reverse Proxy
* Load Balancing

## Estructura del proyecto

```text
parcial1-redes/
├── caso1-laravel/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── ...
│
├── caso2-traefik/
│   ├── docker-compose.yml
│   ├── traefik/
│   │   └── traefik.yml
│   └── ...
│
├── .gitignore
└── README.md
```

---

# Caso 1 — Laravel

Implementación de una aplicación Laravel containerizada junto con su base de datos mediante Docker Compose.

La arquitectura utiliza segmentación de red para separar la aplicación de la base de datos:

```text
                 frontend-net
                172.20.0.0/24
                     │
                     ▼
               Laravel / App
                     │
                     │
                backend-net
                172.20.1.0/24
                     │
                     ▼
                   MySQL
```

La aplicación se conecta a la base de datos mediante la red interna, evitando exponer directamente el servicio de base de datos hacia el exterior.

### Componentes principales

* `Dockerfile` para construir el contenedor de Laravel.
* `docker-compose.yml` para orquestar la aplicación y la base de datos.
* Red `frontend-net` para la aplicación.
* Red `backend-net` para la comunicación interna con MySQL.
* IPAM personalizado para definir las subredes utilizadas.

---

# Caso 2 — Traefik

Implementación de **Traefik como proxy inverso y balanceador de carga** utilizando Docker Compose.

La infraestructura se divide en dos redes:

```text
                    proxy
              172.21.0.0/24
                     │
                     ▼
                ┌─────────┐
                │ Traefik │
                └────┬────┘
                     │
                   backend
              172.22.0.0/24
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       whoami-1   whoami-2   whoami-3
```

### Segmentación

| Red       | Subred          | Gateway      | Función                      |
| --------- | --------------- | ------------ | ---------------------------- |
| `proxy`   | `172.21.0.0/24` | `172.21.0.1` | Red de entrada para Traefik  |
| `backend` | `172.22.0.0/24` | `172.22.0.1` | Red interna de los servicios |

Traefik está conectado a ambas redes, mientras que los servicios `whoami` solamente pertenecen a `backend`.

Esto permite que Traefik actúe como punto de entrada entre el tráfico externo y los servicios internos, evitando que los backends sean accesibles directamente desde el host.

## Reverse Proxy

Traefik recibe las solicitudes HTTP mediante el puerto `80` y utiliza las reglas configuradas mediante Docker labels para determinar qué servicio debe procesar cada solicitud.

El servicio `whoami` utiliza la regla:

```text
Host(`whoami.localhost`)
```

Una solicitud como:

```bash
curl -H "Host: whoami.localhost" http://localhost
```

es recibida por Traefik y posteriormente reenviada hacia una de las instancias internas de `whoami`.

## Load Balancing

Se ejecutan tres instancias del servicio:

```text
whoami-1
whoami-2
whoami-3
```

Las tres son descubiertas automáticamente por Traefik mediante el Docker provider.

Una prueba con múltiples solicitudes permitió observar respuestas provenientes de diferentes contenedores:

```text
Hostname: 25440253b9b7
Hostname: ee762cc2b28a
Hostname: 157b39e169c0
Hostname: ee762cc2b28a
Hostname: 157b39e169c0
Hostname: 25440253b9b7
```

Esto demuestra que las solicitudes son distribuidas entre las diferentes instancias disponibles.

## Aislamiento de los servicios

Los contenedores `whoami` no publican puertos directamente hacia el host. Solamente están conectados a la red `backend`.

Traefik, por otro lado, posee conectividad con ambas redes:

```text
Traefik
├── proxy
└── backend
```

De esta forma, los clientes acceden a los servicios mediante Traefik y no directamente mediante las direcciones IP internas de los contenedores.

## Principio de mínimo privilegio

La arquitectura aplica el principio de mínimo privilegio al limitar la conectividad de cada componente a las redes necesarias para cumplir su función.

Los servicios backend permanecen aislados en una red interna, mientras que Traefik funciona como punto de entrada controlado. Esta separación reduce la superficie de exposición y evita que los servicios internos sean publicados directamente.

## Ejecución

### Caso 1

Desde el directorio correspondiente:

```bash
docker compose up -d
```

### Caso 2

Para iniciar Traefik y tres instancias del backend:

```bash
docker compose up -d --scale whoami=3
```

Verificar los contenedores:

```bash
docker compose ps
```

### Dashboard de Traefik

Con los contenedores ejecutándose, el Dashboard está disponible en:

```text
http://localhost:8080/dashboard/
```

## Verificación de redes

Las redes pueden inspeccionarse mediante:

```bash
docker network inspect caso2-traefik_proxy caso2-traefik_backend
```

También es posible verificar las redes de un contenedor específico:

```bash
docker inspect caso2-traefik-whoami-1 --format '{{json .NetworkSettings.Networks}}'
```

## Objetivos demostrados

* [x] Containerización mediante Docker.
* [x] Orquestación mediante Docker Compose.
* [x] Configuración de redes virtuales.
* [x] Segmentación mediante IPAM.
* [x] Aislamiento de servicios internos.
* [x] Implementación de Traefik como reverse proxy.
* [x] Descubrimiento automático de servicios mediante Docker.
* [x] Balanceo de carga entre múltiples instancias.
* [x] Uso del Dashboard de Traefik.
* [x] Aplicación del principio de mínimo privilegio.

Proyecto desarrollado como parte del laboratorio de Redes de la Universidad Tecnológica de Panamá.
