Actúa como un **arquitecto de software senior, tech lead full-stack y DevOps engineer**.
Genera un **PLAN DE PROYECTO EXTREMADAMENTE DETALLADO** para desarrollar una **plataforma web de cursos online (e-learning)** inspirada en <https://logicanacademy.com/>.

El objetivo es construir una aplicación **full-stack**, modular, escalable y desplegable **100% en local usando Docker Compose**, con frontend, backend, base de datos y servicios auxiliares.

========================

1. VISIÓN GENERAL
========================
Describe:

- Objetivo del producto
- Tipo de usuarios
- Casos de uso principales
- Alcance MVP vs futuras mejoras

La plataforma debe permitir:

- Mostrar un **catálogo de cursos**
- Registro e inicio de sesión de usuarios
- Compra de cursos mediante pagos online
- Gestión de cursos por administradores
- Acceso a cursos comprados por los usuarios

========================
2. STACK TECNOLÓGICO (OBLIGATORIO)
========================

Selecciona y justifica explícitamente las siguientes tecnologías:

### Frontend

- Framework: **React 18**
- Lenguaje: **TypeScript**
- Build tool: **Vite**
- Estilos: **Tailwind CSS**
- UI Components: **Headless UI**
- Routing: **React Router**
- State management: **Redux Toolkit**
- Fetching: **Axios + React Query**
- Autenticación: manejo de JWT
- Testing: **Vitest + React Testing Library**

### Backend

- Runtime: **Node.js 20**
- Framework: **NestJS**
- Lenguaje: **TypeScript**
- Arquitectura: **Clean Architecture / Modular**
- Autenticación: **JWT + Refresh Tokens**
- Autorización: **RBAC (Admin, Instructor, Student)**
- Validación: **class-validator**
- Documentación API: **Swagger**
- Pagos: **Stripe**
- Webhooks de pago
- Testing: **Jest + Supertest**

### Base de Datos

- Motor: **PostgreSQL**
- ORM: **Prisma**
- Migraciones
- Seeders de datos iniciales

### Infraestructura / DevOps

- Contenedores: **Docker**
- Orquestación local: **Docker Compose**
- Reverse proxy (opcional): **Nginx**
- Variables de entorno: `.env`
- Volúmenes persistentes para DB

========================
3. MODELO DE USUARIOS Y ROLES
========================

Define roles y permisos:

- Admin
  - CRUD de cursos
  - Gestión de usuarios
  - Ver pagos
- Instructor (opcional para futura fase)
  - Crear y actualizar cursos
- Student
  - Comprar cursos
  - Acceder a cursos comprados

Incluye:

- Flujos de autenticación
- Refresh tokens
- Protección de rutas frontend y backend

========================
4. MODELO DE DATOS (DATABASE DESIGN)
========================

Define tablas y relaciones:

- User
- Role
- Course
- Category
- Lesson
- Enrollment
- Payment
- Order
- AuditLog (opcional)

Incluye:

- Campos principales
- Tipos
- Relaciones (1:N, N:M)
- Índices importantes
- Constraints
- Ejemplo de schema Prisma

========================
5. BACKEND – API DESIGN
========================

Diseña la API REST con ejemplos de endpoints:

### Auth

- POST /auth/register
- POST /auth/login
- POST /auth/refresh
- POST /auth/logout

### Users

- GET /users/me
- GET /users (admin)
- PUT /users/:id

### Courses

- GET /courses
- GET /courses/:id
- POST /courses (admin)
- PUT /courses/:id
- DELETE /courses/:id

### Payments

- POST /payments/checkout
- POST /payments/webhook (Stripe)
- GET /payments/history

Incluye:

- DTOs
- Guards
- Interceptors
- Manejo de errores

========================
6. FRONTEND – PÁGINAS Y COMPONENTES
========================

Define páginas principales:

- Home
- Catálogo de cursos
- Detalle de curso
- Login / Register
- Dashboard de usuario
- Mis cursos
- Checkout
- Panel de administración

Define:

- Componentes reutilizables
- Layouts
- Protección de rutas
- Manejo de estados de carga y error

========================
7. PAGOS ONLINE
========================

Implementa Stripe con:

- Checkout Session
- Webhooks
- Estados de pago
- Asociación usuario-curso tras pago exitoso
- Modo sandbox

========================
8. DOCKER Y DOCKER COMPOSE
========================

Define:

### Servicios

- frontend
- backend
- postgres
- nginx (opcional)

Incluye:

- Dockerfile para frontend
- Dockerfile para backend
- docker-compose.yml
- Variables de entorno
- Volúmenes persistentes
- Puertos expuestos

========================
9. ESTRUCTURA DE CARPETAS
========================

Define estructura clara para:

- frontend/
- backend/
- infra/
- docs/

========================
10. TESTING Y CALIDAD
========================

Incluye:

- Tests unitarios
- Tests de integración
- Validación de endpoints críticos
- Linting y formato

========================
11. DOCUMENTACIÓN
========================

Genera contenido para:

- README.md
- Setup local paso a paso
- Comandos Docker
- Variables de entorno
- Flujo de desarrollo

========================
12. ROADMAP Y TAREAS
========================

Divide el proyecto en:

- Fase 1: MVP
- Fase 2: Mejoras
- Fase 3: Escalabilidad

Incluye:

- Lista de tareas tipo Jira/GitHub Issues
- Prioridad
- Estimación de tiempo
- Dependencias

========================
FORMATO DE SALIDA
========================

- Usa títulos numerados
- Usa listas claras
- Incluye ejemplos de código SOLO cuando sea necesario
- Mantén enfoque práctico y ejecutable
- Todo debe ser coherente y listo para implementar

Fin del prompt.
