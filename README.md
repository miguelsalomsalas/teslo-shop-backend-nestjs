<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

<h1 align="center">Teslo Shop · API REST</h1>

<p align="center">
  API REST para un e-commerce tipo <strong>Tesla Store</strong>: catálogo de productos, autenticación con JWT y roles, carga de imágenes, notificaciones en tiempo real con WebSockets y documentación interactiva con Swagger.
</p>

<p align="center">
  <img alt="NestJS"     src="https://img.shields.io/badge/NestJS-10-E0234E?logo=nestjs&logoColor=white">
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white">
  <img alt="PostgreSQL" src="https://img.shields.io/badge/PostgreSQL-14-4169E1?logo=postgresql&logoColor=white">
  <img alt="TypeORM"    src="https://img.shields.io/badge/TypeORM-0.3-FE0902">
  <img alt="JWT"        src="https://img.shields.io/badge/JWT-Auth-000000?logo=jsonwebtokens&logoColor=white">
  <img alt="Socket.IO"  src="https://img.shields.io/badge/Socket.IO-4-010101?logo=socketdotio&logoColor=white">
  <img alt="Swagger"    src="https://img.shields.io/badge/Swagger-OpenAPI-85EA2D?logo=swagger&logoColor=black">
  <img alt="Docker"     src="https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white">
</p>

---

## 📌 Descripción del proyecto

**Teslo Shop API** es el backend de una tienda online. Expone una API REST versionada bajo el prefijo `/api` que resuelve las necesidades típicas de un e-commerce real:

- Gestión completa del **catálogo de productos** (CRUD) con imágenes asociadas.
- **Búsqueda, paginación y filtrado** por género, rango de precio, tallas y texto libre.
- **Registro e inicio de sesión** de usuarios con contraseñas cifradas (`bcrypt`) y **JSON Web Tokens**.
- **Autorización basada en roles** (`user`, `admin`, `super-user`) mediante guards y decoradores personalizados.
- **Subida y servido de imágenes** de producto con validación de tipo de archivo y nombres únicos.
- **Chat / notificaciones en tiempo real** con WebSockets (Socket.IO) autenticados por JWT.
- **SEED** para poblar la base de datos con datos de ejemplo con una sola petición.
- **Documentación OpenAPI** autogenerada y navegable con Swagger UI.

> El proyecto está construido siguiendo la arquitectura modular de NestJS, con separación por dominios (módulos), DTOs validados, entidades TypeORM y patrón repositorio.

---

## 🖥️ Frontend que consume esta API

Esta API es consumida por una aplicación **frontend construida con React**:

👉 **[teslo-shop-frontend-react](https://github.com/miguelsalomsalas/teslo-shop-frontend-react)**

La app en React consume estos endpoints para renderizar el catálogo, gestionar el carrito, autenticar usuarios, mostrar el detalle de cada producto y consumir las imágenes servidas por este backend. Juntos forman un proyecto **full-stack** (React + NestJS + PostgreSQL).

---

## 🧰 Herramientas y tecnologías utilizadas

| Área                  | Herramientas                                                                                           |
| --------------------- | ------------------------------------------------------------------------------------------------------ |
| **Framework backend** | [NestJS 10](https://nestjs.com/) (arquitectura modular, inyección de dependencias)                     |
| **Lenguaje**          | TypeScript 5                                                                                           |
| **Base de datos**     | PostgreSQL 14                                                                                          |
| **ORM**               | TypeORM 0.3 (entidades, relaciones, migraciones vía `synchronize`, query builder y transacciones)      |
| **Autenticación**     | Passport + `passport-jwt`, `@nestjs/jwt`, `bcrypt` para el hash de contraseñas                         |
| **Autorización**      | Guards, `custom decorators` y metadata de roles (`@Auth()`, `@GetUser()`, `@RoleProtected()`)          |
| **Validación**        | `class-validator` y `class-transformer` con `ValidationPipe` global (whitelist + forbidNonWhitelisted) |
| **Archivos**          | `Multer` (`@nestjs/platform-express`) + `@nestjs/serve-static` para servir contenido estático          |
| **Tiempo real**       | `@nestjs/websockets` + `Socket.IO`                                                                     |
| **Documentación**     | `@nestjs/swagger` (OpenAPI 3 / Swagger UI)                                                             |
| **Infraestructura**   | Docker Compose (PostgreSQL)                                                                            |
| **Configuración**     | `@nestjs/config` con variables de entorno                                                              |
| **Calidad de código** | ESLint + Prettier                                                                                      |
| **Testing**           | Jest (unit + e2e con Supertest)                                                                        |

---

## 🗂️ Módulos de la aplicación

| Módulo        | Responsabilidad                                                                        |
| ------------- | -------------------------------------------------------------------------------------- |
| `auth`        | Registro, login, verificación de estado, emisión y validación de JWT, guards de roles. |
| `products`    | CRUD de productos, relación uno-a-muchos con imágenes, filtros y paginación.           |
| `files`       | Subida de imágenes de producto y endpoint para servirlas.                              |
| `seed`        | Reinicia y puebla la base de datos con usuarios y productos de ejemplo.                |
| `messages-ws` | Gateway de WebSockets con autenticación por token y difusión de mensajes.              |
| `common`      | DTOs y utilidades compartidas (p. ej. `PaginationDto`).                                |

---

## 🔌 Resumen de endpoints

Prefijo global: **`/api`**

### Auth

| Método | Ruta                     | Descripción                              |
| ------ | ------------------------ | ---------------------------------------- |
| `POST` | `/api/auth/register`     | Crea un usuario y devuelve su JWT        |
| `POST` | `/api/auth/login`        | Inicia sesión y devuelve su JWT          |
| `GET`  | `/api/auth/check-status` | Renueva el token del usuario autenticado |

### Products

| Método   | Ruta                  | Descripción                                                                                                  | Protección        |
| -------- | --------------------- | ------------------------------------------------------------------------------------------------------------ | ----------------- |
| `GET`    | `/api/products`       | Lista productos con paginación y filtros (`limit`, `offset`, `gender`, `minPrice`, `maxPrice`, `sizes`, `q`) | Público           |
| `GET`    | `/api/products/:term` | Producto por `id`, `slug` o `title`                                                                          | Público           |
| `POST`   | `/api/products`       | Crea un producto                                                                                             | JWT               |
| `PATCH`  | `/api/products/:id`   | Actualiza un producto                                                                                        | JWT + rol `admin` |
| `DELETE` | `/api/products/:id`   | Elimina un producto                                                                                          | JWT + rol `admin` |

### Files

| Método | Ruta                            | Descripción                                    |
| ------ | ------------------------------- | ---------------------------------------------- |
| `POST` | `/api/files/product`            | Sube una imagen de producto (valida extensión) |
| `GET`  | `/api/files/product/:imageName` | Devuelve la imagen solicitada                  |

### Seed

| Método | Ruta        | Descripción                                             |
| ------ | ----------- | ------------------------------------------------------- |
| `GET`  | `/api/seed` | Destruye y recrea la base de datos con datos de ejemplo |

### WebSockets

- Namespace por defecto de Socket.IO, autenticado con el header `authentication` (JWT).
- Eventos: `message-from-client` → `message-from-server`, `clients-updated`.

---

## 📖 Documentación de la API (Swagger)

Con el proyecto levantado, la documentación interactiva está disponible en:

```
http://localhost:3000/api
```

---

## 🚀 Puesta en marcha (desarrollo)

1. Clonar el proyecto.
2. Instalar dependencias:
   ```bash
   yarn install
   ```
3. Clonar el archivo `.env.template` y renombrarlo a `.env`.
4. Configurar las variables de entorno (ver más abajo).
5. Levantar la base de datos:
   ```bash
   docker-compose up -d
   ```
6. Levantar el servidor en modo desarrollo:
   ```bash
   yarn start:dev
   ```
7. Ejecutar el SEED para poblar la base de datos:
   ```
   http://localhost:3000/api/seed
   ```

---

## 🔑 Variables de entorno

| Variable      | Descripción                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------ |
| `PORT`        | Puerto en el que corre la API (p. ej. `3000`)                                                          |
| `HOST_API`    | URL base de la API, usada para construir las URLs de las imágenes (p. ej. `http://localhost:3000/api`) |
| `STAGE`       | Entorno de ejecución (`dev` / `prod`); en `prod` habilita SSL en la conexión a la BBDD                 |
| `DB_HOST`     | Host de PostgreSQL                                                                                     |
| `DB_PORT`     | Puerto de PostgreSQL (`5432`)                                                                          |
| `DB_NAME`     | Nombre de la base de datos                                                                             |
| `DB_USERNAME` | Usuario de la base de datos                                                                            |
| `DB_PASSWORD` | Contraseña de la base de datos                                                                         |
| `JWT_SECRET`  | Clave secreta para firmar los JSON Web Tokens                                                          |

---

## 🧪 Tests

```bash
yarn test        # unit tests
yarn test:e2e    # end-to-end
yarn test:cov    # cobertura
```

---

## 🏗️ Notas de producción

1. Crear el archivo `.env.prod`.
2. Definir las variables de entorno de producción.
3. Construir la imagen:
   ```bash
   docker-compose -f docker-compose.prod.yaml --env-file .env.prod up --build
   ```

---

## 👤 Contacto

**Miguel Salom**

- GitHub: [@miguelsalomsalas](https://github.com/miguelsalomsalas)
- Frontend del proyecto (React): [teslo-shop-frontend-react](https://github.com/miguelsalomsalas/teslo-shop-frontend-react)
  </parameter>
  </invoke>
