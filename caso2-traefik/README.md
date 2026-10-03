# Caso 2 — Traefik Reverse Proxy

Implementación de **Traefik como reverse proxy y balanceador de carga** utilizando Docker Compose, con segmentación de redes mediante IPAM virtual para controlar la comunicación y el aislamiento de los servicios.

## 📋 Descripción

Este caso corresponde al laboratorio práctico de **Traefik** del Parcial #1 de Redes.

El objetivo es implementar un punto de entrada centralizado para los servicios containerizados mediante Traefik, utilizando el proveedor de Docker para detectar automáticamente los contenedores y gestionar el enrutamiento del tráfico.

La infraestructura también utiliza diferentes redes Docker para separar los componentes y limitar la comunicación entre ellos.

## 🏗️ Arquitectura

```text
                         Cliente
                            │
                            │ HTTP :80
                            ▼
                    ┌───────────────┐
                    │    Traefik    │
                    │ Reverse Proxy │
                    │ Load Balancer │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
          ┌─────────────┐       ┌─────────────┐
          │    App 1    │       │    App 2    │
          │  Container  │       │  Container  │
          └─────────────┘       └─────────────┘
```

Traefik funciona como punto de entrada de la infraestructura y distribuye las solicitudes hacia los servicios disponibles.

## 🛠️ Tecnologías

* Traefik v3.6
* Docker
* Docker Compose
* Docker Networking
* IPAM virtual
* Reverse Proxy
* Load Balancing

## 📁 Estructura

```text
Caso-Traefik/
├── traefik/
│   └── traefik.yml
├── docker-compose.yml
└── README.md
```

## 🔀 Traefik

Traefik se utiliza como **reverse proxy**, permitiendo que los clientes accedan a los servicios a través de un único punto de entrada.

La configuración utiliza el proveedor de Docker, permitiendo que Traefik obtenga información de los contenedores y determine cómo enrutar las solicitudes.

Entre sus principales funciones en este laboratorio se encuentran:

* Recibir solicitudes HTTP.
* Enrutar tráfico hacia los servicios.
* Detectar servicios mediante Docker.
* Distribuir solicitudes entre diferentes instancias.
* Centralizar el acceso a los servicios.

## ⚙️ Configuración

El archivo:

```text
traefik/traefik.yml
```

contiene la configuración estática de Traefik.

Entre los elementos configurados se encuentran los **entrypoints**, el proveedor de Docker y el acceso al socket de Docker en modo de solo lectura.

El contenedor utiliza:

```yaml
- /var/run/docker.sock:/var/run/docker.sock:ro
```

El modo `ro` permite que Traefik consulte información de Docker sin otorgarle permisos de escritura sobre el socket.

## 🌐 Segmentación de red

El proyecto utiliza redes Docker independientes para separar los componentes de la infraestructura.

La segmentación mediante **IPAM virtual** permite definir rangos de direcciones específicos para cada red y controlar qué servicios pueden comunicarse entre sí.

La arquitectura busca evitar que todos los contenedores compartan una única red, reduciendo la comunicación innecesaria entre servicios.

### Principio de mínimo privilegio

Cada servicio se conecta únicamente a las redes que necesita para cumplir su función.

Esto permite:

* Aislar componentes internos.
* Reducir la superficie de exposición.
* Limitar la comunicación entre contenedores.
* Separar el tráfico externo del tráfico interno.

## 🐳 Docker Compose

El archivo `docker-compose.yml` define la infraestructura necesaria para ejecutar Traefik.

El servicio principal es:

```yaml
traefik:
  image: traefik:v3.6
```

También se configuran los puertos:

```yaml
ports:
  - "80:80"
  - "8080:8080"
```

* **Puerto 80:** punto de entrada HTTP.
* **Puerto 8080:** dashboard de Traefik durante el desarrollo.

## 🚀 Ejecución

### 1. Levantar la infraestructura

```bash
docker compose up -d
```

O para reconstruir los servicios:

```bash
docker compose up -d --build
```

### 2. Verificar los contenedores

```bash
docker compose ps
```

### 3. Ver los logs de Traefik

```bash
docker compose logs traefik
```

Para seguir los logs en tiempo real:

```bash
docker compose logs -f traefik
```

### 4. Verificar las redes

```bash
docker network ls
```

Para inspeccionar una red específica:

```bash
docker network inspect <nombre_de_la_red>
```

## 📊 Dashboard

Durante el desarrollo, Traefik expone su dashboard mediante el puerto `8080`.

```text
http://localhost:8080
```

El dashboard permite visualizar información sobre:

* Routers.
* Services.
* Middlewares.
* EntryPoints.
* Providers.

## ⚖️ Load Balancing

Traefik puede distribuir las solicitudes entre múltiples instancias de un servicio.

La arquitectura permite ejecutar varias réplicas del mismo servicio y utilizar Traefik como punto de distribución del tráfico.

Conceptualmente:

```text
                  ┌─────────────┐
                  │   Traefik   │
                  └──────┬──────┘
                         │
                ┌────────┼────────┐
                │        │        │
                ▼        ▼        ▼
              App 1    App 2    App 3
```

Esto permite demostrar el funcionamiento de un **balanceador de carga dentro de una infraestructura containerizada**.

## 🔐 Consideraciones de seguridad

La configuración aplica principios de aislamiento mediante redes Docker.

Traefik funciona como punto de entrada para el tráfico externo, mientras que los servicios internos permanecen en redes destinadas a su comunicación específica.

Además, el acceso al Docker socket se configura en modo de solo lectura:

```yaml
/var/run/docker.sock:/var/run/docker.sock:ro
```

Esto limita las operaciones que Traefik puede realizar sobre el socket.

## 🧪 Verificación

Para comprobar que Traefik está funcionando:

```bash
docker compose ps
```

Consultar los logs:

```bash
docker compose logs traefik
```

Verificar las redes:

```bash
docker network ls
```

Inspeccionar el direccionamiento IP:

```bash
docker network inspect <nombre_de_la_red>
```

Finalmente, acceder al dashboard:

```text
http://localhost:8080
```

## 📚 Conceptos demostrados

* Traefik.
* Reverse Proxy.
* Load Balancing.
* Docker Provider.
* Docker Compose.
* EntryPoints.
* Service Discovery.
* Docker Networking.
* IPAM virtual.
* Segmentación de redes.
* Aislamiento de servicios.
* Principio de mínimo privilegio.
* Infraestructura containerizada.

## 🎓 Contexto académico

**Universidad Tecnológica de Panamá**
**Facultad de Ingeniería de Sistemas Computacionales**

Proyecto correspondiente al **Parcial #1 — Despliegue e Infraestructura Containerizada**.

Este caso forma parte del laboratorio práctico de **Traefik**, cuyo objetivo es implementar un reverse proxy y balanceador de carga dentro de una infraestructura Docker segmentada mediante redes virtuales.
