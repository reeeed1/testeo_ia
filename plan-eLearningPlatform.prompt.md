# PLAN DE PROYECTO: Plataforma E-Learning (LMS) Full-Stack

## 1. VISIÓN GENERAL

### Objetivo del Producto
Desarrollar una plataforma de gestión de aprendizaje (LMS) robusta y moderna que permita a los instructores publicar cursos y a los estudiantes adquirirlos y consumirlos. El sistema debe ser autohospedable y gestionable localmente mediante contenedores.

### Tipos de Usuarios
1. **Administrador**: Gestión global del sistema, usuarios y finanzas.
2. **Instructor**: Creador de contenido (Nota: funcionalidad simplificada o unificada con Admin en MVP).
3. **Estudiante**: Consumidor de contenido.

### Casos de Uso Principales
- **Visitante**: Explorar catálogo, ver detalles de curso, registrarse/login.
- **Estudiante**: Comprar curso (Stripe), ver "Mis Cursos", reproducir lecciones, marcar progreso.
- **Admin**: Crear/Editar/Borrar cursos y lecciones, ver historial de órdenes, gestionar usuarios.

### Alcance
- **MVP (Fase 1)**: Catálogo público, Auth completa, Pagos básicos (Stripe), Consumo de video (enlaces o local), Gestión básica de cursos por Admin.
- **Mejoras (Fase 2)**: Dashboard de instructor separado, Exámenes/Quizzes, Certificados, Comentarios.

---

## 2. STACK TECNOLÓGICO

### Frontend
- **Framework**: **React 18**. Estándar de industria, gran ecosistema, rendimiento con features de React 18 (Concurrency, Transitions).
- **Lenguaje**: **TypeScript**. Seguridad de tipos indispensable para mantenibilidad a largo plazo.
- **Build Tool**: **Vite**. Velocidad de desarrollo superior y builds optimizados.
- **Estilos**: **Tailwind CSS**. Desarrollo rápido de UI sin salir del HTML, altamente personalizable.
- **UI Components**: **Headless UI**. Accesibilidad garantizada sin estilos opinionados, integrándose perfecto con Tailwind.
- **Routing**: **React Router v6**. Manejo de rutas declarativo y nested routes.
- **State**: **Redux Toolkit (RTK)**. Gestión robusta de estado global (auth, carrito).
- **Fetching**: **Axios + React Query (TanStack Query)**. Axios para peticiones HTTP tipadas; React Query para cache, revalidación y estado del servidor.
- **Auth**: Gestión de **JWT** (access token en memoria/corto plazo, refresh token en httpOnly cookie).
- **Testing**: **Vitest** (compatible con Vite, rápido) + **React Testing Library**.

### Backend
- **Runtime**: **Node.js 20** (LTS).
- **Framework**: **NestJS**. Arquitectura angular-like, inyección de dependencias, modularidad y escalabilidad empresarial.
- **Lenguaje**: **TypeScript**.
- **Arquitectura**: **Clean Architecture / Modular**. Separación de responsabilidades: Controladores -> Servicios -> Repositorios.
- **Auth**: **Passport** con estrategias JWT. Sistema de Refresh Tokens rotativos.
- **Validación**: **class-validator** y **class-transformer** para DTOs.
- **Doc**: **Swagger (OpenAPI)** generado automáticamente via decoradores.
- **Pagos**: **Stripe Node.js SDK**.
- **Testing**: **Jest** (nativo en NestJS) + **Supertest** para e2e.

### Base de Datos
- **Motor**: **PostgreSQL 15+**. Relacional, robusto, ideal para datos estructurados (usuarios, órdenes).
- **ORM**: **Prisma**. Type-safety extrema, migraciones declarativas y DX superior.

### Infraestructura Local
- **Docker**: Contenedorización de servicios.
- **Docker Compose**: Orquestación de servicios (App + DB + Tools).
- **Nginx**: Reverse proxy para servir frontend y backend bajo un mismo dominio local (opcional/recomendado para simular prod).

---

## 3. MODELO DE USUARIOS Y ROLES

### Roles (RBAC)
Los roles se gestionan mediante un Enum en la DB y Decoradores (`@Roles('ADMIN')`) en NestJS.

1. **Studen** (Default):
   - `GET /courses` (público)
   - `POST /orders` (comprar)
   - `GET /enrollments` (sus cursos)
   - `GET /courses/:id/content` (solo si compró)
2. **Admin**:
   - Acceso total a todos los endpoints.
   - `POST/PUT/DELETE /courses` y `/lessons`.
   - `GET /users`, `GET /admin/stats`.

### Flujo de Autenticación
1. **Login**: `POST /auth/login` -> Recibe `accessToken` (body) y `refreshToken` (httpOnly cookie).
2. **Uso**: Cliente envía `Authorization: Bearer <token>` en headers.
3. **Expiración**: Interceptor en axios detecta 401 -> llama a `POST /auth/refresh` (envía cookie) -> recibe nuevos tokens -> reintenta petición original.
4. **Logout**: `POST /auth/logout` -> Invalida refresh token en DB y limpia cookie.

---

## 4. MODELO DE DATOS (DATABASE DESIGN)

### Schema Resumido (Prisma)

```prisma
model User {
  id           String       @id @default(uuid())
  email        String       @unique
  passwordHash String
  fullName     String?
  roles        Role[]       @default([STUDENT])
  createdAt    DateTime     @default(now())
  
  enrollments  Enrollment[]
  orders       Order[]
  refreshTokens RefreshToken[]
}

enum Role {
  STUDENT
  ADMIN
  INSTRUCTOR
}

model Course {
  id          String   @id @default(uuid())
  title       String
  slug        String   @unique
  description String   @db.Text
  price       Decimal  @db.Decimal(10, 2)
  isPublished Boolean  @default(false)
  thumbnailUrl String?
  
  modules     Module[]
  enrollments Enrollment[]
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

model Module {
  id       String   @id @default(uuid())
  title    String
  order    Int
  courseId String
  course   Course   @relation(fields: [courseId], references: [id])
  lessons  Lesson[]
}

model Lesson {
  id          String  @id @default(uuid())
  title       String
  videoUrl    String?
  content     String? @db.Text // Markdown
  isFree      Boolean @default(false)
  moduleId    String
  module      Module  @relation(fields: [moduleId], references: [id])
  durationSec Int     @default(0)
}

model Enrollment {
  id        String   @id @default(uuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  courseId  String
  course    Course   @relation(fields: [courseId], references: [id])
  enrolledAt DateTime @default(now())
  
  @@unique([userId, courseId])
}

model Order {
  id            String      @id @default(uuid())
  userId        String
  user          User        @relation(fields: [userId], references: [id])
  totalAmount   Decimal
  status        OrderStatus @default(PENDING)
  stripeSessionId String?   @unique
  createdAt     DateTime    @default(now())
  
  items         OrderItem[]
}

enum OrderStatus {
  PENDING
  COMPLETED
  FAILED
  REFUNDED
}

// ... Tablas auxiliares: OrderItem, PaymentLog, etc.
```

---

## 5. BACKEND – API DESIGN

### Estructura Global
Prefijo: `/api/v1`

### Módulo Auth
- `POST /auth/register`: `{ email, password, fullName }`
- `POST /auth/login`: `{ email, password }` -> `{ accessToken, user }`
- `POST /auth/refresh`: (Usa Cookie) -> `{ accessToken }`
- `POST /auth/logout`: Revoca sesión.

### Módulo Users
- `GET /users/me`: Perfil propio + Enrollments resumen.
- `PATCH /users/me`: Actualizar perfil.

### Módulo Courses
- `GET /courses`: Listado paginado y filtrable (público).
- `GET /courses/:slug`: Info pública del curso (sin videos privados).
- `GET /courses/:id/learn`: Estructura completa con videos (Guard: IsEnrolled).
- `POST /courses` (Admin): Crear curso.
- `PATCH /courses/:id` (Admin): Actualizar.
- `POST /courses/:id/modules` (Admin): Agregar módulo.

### Módulo Payments
- `POST /payments/checkout`: `{ courseIds: [] }` -> Crea Stripe Session -> Retorna `{ url }`.
- `POST /payments/webhook`: Recibe eventos Stripe (`checkout.session.completed`) -> Crea `Order` y `Enrollment`.

### Manejo de Errores
- Filtro de Excepciones Global (`HttpExceptionFilter`): Estandariza respuestas JSON (`statusCode`, `message`, `timestamp`).

---

## 6. FRONTEND – PÁGINAS Y ESTRUCTURA

### Layouts
- **MainLayout**: Navbar (Logo, menú, carrito, avatar usuario), Footer.
- **AuthLayout**: Centrado, diseño limpio para Login/Register.
- **LearnLayout**: Sidebar con lista de lecciones (acordeón módulos), Player de video central, navegación anterior/siguiente.
- **AdminLayout**: Sidebar de gestión (Dashboard, Cursos, Usuarios, Ventas).

### Páginas Clave
1. **Home**: Hero section, Cursos destacados, Testimonios.
2. **Catalog**: Grid de cursos con filtros (Categoría, precio).
3. **CourseDetail**: Landing page del curso (Qué aprenderás, temario, instructor, botón Comprar).
4. **Checkout**: Resumen de orden, confirmación de redirección a Stripe.
5. **MyCourses**: Grid de cursos comprados con barra de progreso.
6. **Classroom (Player)**: Vista de consumo de contenido.
7. **Admin/CourseEditor**: Formulario complejo (drag & drop lecciones, subida de video, edición rich text).

### Componentes UI (Headless UI + Tailwind)
- `Button`, `Input`, `Modal` (Dialog), `Dropdown` (Menu), `Accordion` (Disclosure).

---

## 7. PAGOS ONLINE

### Flujo Stripe
1. Usuario añade cursos al "Carrito" (estado local o backend).
2. Clic en "Pagar". Backend genera `Stripe Checkout Session` con `metadata: { userId, courseIds }`.
3. Frontend redirige a `session.url`.
4. Usuario paga en Stripe.
5. Stripe redirige a `/checkout/success` en Frontend.
6. Asíncronamente, Stripe llama al Webhook `/api/v1/payments/webhook`.
7. Backend verifica firma, valida evento, cambia estado `Order` a `COMPLETED` y crea registros en `Enrollment`.

---

## 8. DOCKER Y DOCKER COMPOSE

### `docker-compose.yml`
```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASS}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build: 
      context: ./backend
      dockerfile: Dockerfile.dev
    volumes:
      - ./backend:/app
      - /app/node_modules
    ports:
      - "3001:3000"
    environment:
      DATABASE_URL: postgres://${DB_USER}:${DB_PASS}@postgres:5432/${DB_NAME}
      # ... otras env vars
    depends_on:
      - postgres

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:5173" # Vite default
    environment:
      VITE_API_URL: http://localhost:3001/api/v1

volumes:
  postgres_data:
```

---

## 9. ESTRUCTURA DE CARPETAS

```text
/
├── backend/
│   ├── src/
│   │   ├── modules/
│   │   │   ├── auth/
│   │   │   ├── users/
│   │   │   ├── courses/
│   │   │   └── payments/
│   │   ├── common/
│   │   │   ├── decorators/
│   │   │   ├── guards/
│   │   │   └── filters/
│   │   ├── config/
│   │   └── main.ts
│   ├── prisma/
│   ├── test/
│   ├── Dockerfile
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   │   ├── common/
│   │   │   └── layout/
│   │   ├── features/ (slicing por dominio: auth, courses, cart)
│   │   ├── hooks/
│   │   ├── pages/
│   │   ├── store/
│   │   └── App.tsx
│   ├── Dockerfile
│   └── package.json
├── infra/
│   └── nginx.conf (opcional)
├── .env.example
├── docker-compose.yml
└── README.md
```

---

## 10. TESTING Y CALIDAD

- **Backend**:
  - Unit Tests (`.spec.ts`) para Services y lógica de negocio.
  - E2E Tests (`/test`) para flujos completos de API (ej: registro -> login -> crear curso).
  - Linting: `eslint-config-prettier`.
- **Frontend**:
  - Component Testing con `Vitest` + `RTL` para componentes UI complejos.
  - Integración para flujos críticos (ej: Checkout).

---

## 11. DOCUMENTACIÓN PRELIMINAR

### Setup Local
1. Clonar repositorio.
2. Copiar `.env.example` a `.env` y configurar credenciales.
3. Ejecutar `docker-compose up -d`.
4. Ejecutar migraciones: `docker-compose exec backend npx prisma migrate dev`.
5. Seed database: `docker-compose exec backend npm run seed`.
6. Acceder a `http://localhost:3000`.

---

## 12. ROADMAP

### Fase 1: Core MVP (Semanas 1-4)
- **S1**: Setup Monorepo/Docker, Configuración NestJS+Prisma, Auth Module.
- **S2**: DB Schema Cursos, API CRUD Cursos, Frontend Home + Catálogo.
- **S3**: Carrito de compras, Integración Stripe (Backend+Frontend).
- **S4**: Webhooks, Enrollments, Player de video básico, Despliegue local final.

### Fase 2: Robustez (Semanas 5-6)
- Upload de archivos reales (S3/MinIO local).
- Dashboard de Estudiante (progreso).
- Refinamiento de UI/UX (Headless UI animaciones).

### Fase 3: Escalabilidad
- Exámenes y Certificados.
- Roles de Instructor separados con dashboard propio.
- Transcodificación de video.
