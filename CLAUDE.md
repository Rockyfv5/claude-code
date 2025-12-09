# Platziflix - Proyecto Multi-plataforma

## 📋 Índice
1. [Arquitectura del Sistema](#arquitectura-del-sistema)
2. [Stack Tecnológico](#stack-tecnológico)
3. [Estructura del Proyecto](#estructura-del-proyecto)
4. [Modelo de Datos](#modelo-de-datos)
5. [API Endpoints](#api-endpoints)
6. [Patrones Arquitectónicos](#patrones-arquitectónicos)
7. [Comunicación Inter-Plataforma](#comunicación-inter-plataforma)
8. [Comandos de Desarrollo](#comandos-de-desarrollo)
9. [Consideraciones de Desarrollo](#consideraciones-de-desarrollo)

---

## Arquitectura del Sistema

Platziflix es una plataforma de cursos online con **arquitectura multi-plataforma** que funciona como un ecosistema unificado a través de una API REST centralizada:

```
┌─────────────────────────────────────────────────────────────┐
│                    PLATZIFLIX ECOSYSTEM                      │
└─────────────────────────────────────────────────────────────┘
                               │
                  ┌────────────┴────────────┐
                  │   REST API (FastAPI)    │
                  │  PostgreSQL Database    │
                  │      Port: 8000         │
                  └────────────┬────────────┘
                               │
      ┌────────────────────────┼────────────────────────┐
      │                        │                        │
┌─────▼─────┐         ┌───────▼──────┐        ┌───────▼──────┐
│ Frontend  │         │   Android    │        │     iOS      │
│Next.js 15 │         │    Kotlin    │        │    Swift     │
│Port: 3000 │         │   Compose    │        │   SwiftUI    │
└───────────┘         └──────────────┘        └──────────────┘
```

### Componentes Principales:
- **Backend**: API REST con FastAPI + PostgreSQL (fuente única de verdad)
- **Frontend**: Aplicación web con Next.js 15
- **Mobile**: Apps nativas Android (Kotlin) + iOS (Swift)

---

## Stack Tecnológico

### Backend (FastAPI/Python)
- **Framework**: FastAPI
- **Base de datos**: PostgreSQL 15
- **ORM**: SQLAlchemy 2.0
- **Migraciones**: Alembic
- **Container**: Docker + Docker Compose
- **Gestión dependencias**: UV (moderno y rápido)
- **Puerto**: 8000
- **Testing**: pytest + httpx

### Frontend (Next.js)
- **Framework**: Next.js 15 (App Router con React Server Components)
- **React**: 19.0
- **Lenguaje**: TypeScript (strict mode)
- **Estilos**: SCSS + CSS Modules (auto-import de variables)
- **Testing**: Vitest + React Testing Library
- **Fonts**: Geist Sans & Geist Mono
- **Puerto**: 3000

### Mobile

#### Android
- **Lenguaje**: Kotlin
- **UI**: Jetpack Compose (UI declarativa)
- **Arquitectura**: Clean Architecture + MVI
- **Networking**: Retrofit 2 + OkHttp 3
- **Async**: Kotlin Coroutines
- **DI**: Manual (AppModule)

#### iOS
- **Lenguaje**: Swift
- **UI**: SwiftUI (UI declarativa)
- **Arquitectura**: Clean Architecture + MVVM
- **Networking**: URLSession nativo
- **Async**: Swift Concurrency (async/await)
- **Reactive**: Combine Framework

---

## Estructura del Proyecto

### Árbol Completo
```
claude-code/
├── Backend/                              # API FastAPI + PostgreSQL
│   ├── app/
│   │   ├── alembic/                      # Migraciones de DB
│   │   │   └── versions/                 # Archivos de migración
│   │   ├── core/                         # Configuración
│   │   │   └── config.py                 # Settings (Pydantic)
│   │   ├── db/                           # Database layer
│   │   │   ├── base.py                   # Engine & session
│   │   │   └── seed.py                   # Data seeding
│   │   ├── models/                       # SQLAlchemy ORM models
│   │   │   ├── base.py                   # BaseModel (soft delete)
│   │   │   ├── course.py                 # Course model
│   │   │   ├── teacher.py                # Teacher model
│   │   │   ├── lesson.py                 # Lesson model
│   │   │   ├── class_.py                 # Class model
│   │   │   ├── course_teacher.py         # M:N association
│   │   │   └── course_rating.py          # Rating model
│   │   ├── schemas/                      # Pydantic schemas
│   │   │   └── rating.py                 # Rating DTOs
│   │   ├── services/                     # Business logic
│   │   │   └── course_service.py         # CourseService
│   │   ├── tests/                        # Unit & integration tests
│   │   ├── main.py                       # FastAPI app entry
│   │   └── alembic.ini                   # Alembic config
│   ├── docker-compose.yml                # Docker services
│   ├── Dockerfile                        # API container
│   ├── Makefile                          # Dev commands
│   └── pyproject.toml                    # Dependencies (UV)
│
├── Frontend/                             # Next.js 15 App
│   ├── src/
│   │   ├── app/                          # App Router
│   │   │   ├── page.tsx                  # Home (course grid)
│   │   │   ├── layout.tsx                # Root layout
│   │   │   ├── course/[slug]/            # Course detail route
│   │   │   │   ├── page.tsx              # Server Component
│   │   │   │   ├── error.tsx             # Error boundary
│   │   │   │   ├── loading.tsx           # Loading state
│   │   │   │   └── not-found.tsx         # 404 handler
│   │   │   └── classes/[class_id]/       # Class player route
│   │   │       └── page.tsx
│   │   ├── components/                   # React components
│   │   │   ├── Course/                   # Course card
│   │   │   ├── CourseDetail/             # Course detail
│   │   │   ├── StarRating/               # Rating display
│   │   │   └── VideoPlayer/              # Video player
│   │   ├── services/
│   │   │   └── ratingsApi.ts             # API client
│   │   ├── types/                        # TypeScript types
│   │   │   ├── index.ts                  # Course types
│   │   │   └── rating.ts                 # Rating types
│   │   └── styles/
│   │       ├── reset.scss                # CSS reset
│   │       └── vars.scss                 # SCSS variables
│   ├── package.json
│   ├── next.config.ts                    # Next.js config
│   ├── tsconfig.json                     # TypeScript config
│   └── vitest.config.ts                  # Test runner
│
└── Mobile/
    ├── PlatziFlixAndroid/                # Kotlin App
    │   └── app/src/main/
    │       └── java/com/espaciotiago/platziflixandroid/
    │           ├── data/                 # Data layer
    │           │   ├── entities/         # DTOs
    │           │   ├── mappers/          # DTO → Domain
    │           │   ├── network/          # Retrofit config
    │           │   └── repositories/     # Repository impl
    │           ├── domain/               # Domain layer
    │           │   ├── models/           # Domain models
    │           │   └── repositories/     # Repository interfaces
    │           ├── presentation/         # Presentation layer
    │           │   └── courses/
    │           │       ├── components/   # Compose UI
    │           │       ├── screen/       # Screens
    │           │       ├── state/        # UI State
    │           │       └── viewmodel/    # ViewModels (MVI)
    │           ├── di/                   # Dependency injection
    │           └── ui/theme/             # Material Theme
    │
    └── PlatziFlixiOS/                    # Swift App
        └── PlatziFlixiOS/
            ├── Data/                     # Data layer
            │   ├── Entities/             # DTOs
            │   ├── Mapper/               # DTO → Domain
            │   └── Repositories/         # Repository impl
            ├── Domain/                   # Domain layer
            │   ├── Models/               # Domain models
            │   └── Repositories/         # Repository protocols
            ├── Presentation/             # Presentation layer
            │   ├── ViewModels/           # ViewModels (MVVM)
            │   └── Views/                # SwiftUI views
            ├── Services/                 # Network layer
            │   ├── NetworkService.swift  # Protocol
            │   ├── NetworkManager.swift  # Implementation
            │   └── NetworkError.swift    # Error types
            └── PlatziFlixiOSApp.swift    # App entry
```

---

## Modelo de Datos

### Entidades y Relaciones

```
┌─────────────┐         ┌──────────────────┐         ┌─────────────┐
│   Teacher   │◄───────►│ course_teachers  │◄───────►│   Course    │
│             │  M:N    │  (association)   │  M:N    │             │
└─────────────┘         └──────────────────┘         └──────┬──────┘
                                                             │ 1:N
                                                             ├──────► Lesson
                                                             │
                                                             │ 1:N
                                                             └──────► CourseRating
```

### Detalles de Entidades

#### BaseModel (Herencia común)
```python
- id: Integer (PK)
- created_at: DateTime (auto)
- updated_at: DateTime (auto)
- deleted_at: DateTime (soft delete)
```

#### Course
```python
- name: String(255) NOT NULL
- description: Text NOT NULL
- thumbnail: String(500) NOT NULL
- slug: String(255) UNIQUE NOT NULL [INDEXED]
- teachers: Many-to-Many → Teacher
- lessons: One-to-Many → Lesson
- ratings: One-to-Many → CourseRating
- @property average_rating: Float (calculado)
- @property total_ratings: Integer (calculado)
```

#### Teacher
```python
- name: String(255) NOT NULL
- email: String(255) UNIQUE NOT NULL [INDEXED]
- courses: Many-to-Many → Course
```

#### Lesson
```python
- course_id: Integer [FK → courses.id, INDEXED]
- name: String(255) NOT NULL
- description: Text NOT NULL
- slug: String(255) [INDEXED]
- video_url: String(500) NOT NULL
- course: Many-to-One → Course
```

#### CourseRating
```python
- course_id: Integer [FK → courses.id, INDEXED]
- user_id: Integer [INDEXED]
- rating: Integer CHECK (rating >= 1 AND rating <= 5)
- course: Many-to-One → Course
- UNIQUE(course_id, user_id, deleted_at)  # Una rating activa por usuario
```

---

## API Endpoints

### Base URL
- **Development**: `http://localhost:8000`
- **Documentación**: `http://localhost:8000/docs` (Swagger UI)

### Endpoints Disponibles

#### General
| Método | Endpoint | Descripción | Respuesta |
|--------|----------|-------------|-----------|
| `GET` | `/` | Mensaje de bienvenida | `{"message": "Welcome to Platziflix API"}` |
| `GET` | `/health` | Health check + DB status | `{"status": "ok", "database": "connected"}` |

#### Courses
| Método | Endpoint | Descripción | Respuesta |
|--------|----------|-------------|-----------|
| `GET` | `/courses` | Lista todos los cursos con ratings | `Course[]` con `average_rating`, `total_ratings` |
| `GET` | `/courses/{slug}` | Detalle de curso por slug | `CourseDetail` con profesores, lecciones |
| `GET` | `/classes/{class_id}` | Detalle de lección/clase | `ClassDetail` |

#### Ratings
| Método | Endpoint | Descripción | Respuesta |
|--------|----------|-------------|-----------|
| `POST` | `/courses/{course_id}/ratings` | Crear/actualizar rating (idempotente) | `201 Created` → `CourseRating` |
| `GET` | `/courses/{course_id}/ratings` | Lista todos los ratings del curso | `CourseRating[]` |
| `GET` | `/courses/{course_id}/ratings/stats` | Estadísticas agregadas | `RatingStats` |
| `GET` | `/courses/{course_id}/ratings/user/{user_id}` | Rating del usuario | `CourseRating` o `404` |
| `PUT` | `/courses/{course_id}/ratings/{user_id}` | Actualizar rating existente | `200 OK` → `CourseRating` |
| `DELETE` | `/courses/{course_id}/ratings/{user_id}` | Soft delete rating | `204 No Content` |

### Ejemplos de Respuestas

#### GET /courses
```json
[
  {
    "id": 4,
    "name": "Curso de React.js",
    "description": "Aprende React desde cero...",
    "thumbnail": "https://example.com/react.jpg",
    "slug": "curso-de-react",
    "average_rating": 4.5,
    "total_ratings": 142,
    "created_at": "2025-01-15T10:00:00",
    "updated_at": "2025-01-15T10:00:00",
    "deleted_at": null
  }
]
```

#### GET /courses/{slug}
```json
{
  "id": 4,
  "name": "Curso de React.js",
  "slug": "curso-de-react",
  "teacher_id": [1, 2],
  "classes": [
    {
      "id": 101,
      "name": "Introducción a React",
      "description": "Conceptos básicos de React",
      "slug": "intro-react"
    }
  ],
  "average_rating": 4.5,
  "total_ratings": 142
}
```

#### POST /courses/{course_id}/ratings
```json
// Request Body
{
  "user_id": 42,
  "rating": 5
}

// Response (201 Created)
{
  "id": 123,
  "course_id": 4,
  "user_id": 42,
  "rating": 5,
  "created_at": "2025-10-14T10:30:00",
  "updated_at": "2025-10-14T10:30:00",
  "deleted_at": null
}
```

---

## Patrones Arquitectónicos

### 1. Backend - Service Layer Pattern

```
┌─────────────────────────────────────────────────────┐
│            HTTP Layer (FastAPI)                      │
│  - Endpoints: @app.get(), @app.post()               │
│  - Request/Response validation (Pydantic)           │
└──────────────────┬──────────────────────────────────┘
                   │ Dependency Injection
┌──────────────────▼──────────────────────────────────┐
│            Service Layer                            │
│  - CourseService (Business Logic)                   │
│  - get_all_courses(), add_course_rating()           │
│  - Transacciones y validaciones                     │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│            Data Layer                               │
│  - SQLAlchemy Models (ORM)                          │
│  - PostgreSQL Database                              │
│  - Joins, agregaciones SQL                          │
└─────────────────────────────────────────────────────┘
```

**Archivo clave**: `Backend/app/services/course_service.py`

**Ventajas**:
- ✅ Lógica de negocio separada del HTTP layer
- ✅ Testeable sin inicializar FastAPI
- ✅ Reutilizable desde múltiples endpoints
- ✅ Transacciones centralizadas

**Ejemplo**:
```python
class CourseService:
    def __init__(self, db: Session):
        self.db = db

    def get_all_courses(self) -> List[Course]:
        """Obtiene todos los cursos con stats de rating"""
        # Lógica de negocio compleja
        # Joins, agregaciones SQL
        return courses
```

### 2. Frontend - React Server Components (RSC)

```
┌─────────────────────────────────────────────────────┐
│        Next.js App Router (File-based)               │
│  - app/page.tsx: Server Component (default)         │
│  - app/course/[slug]/page.tsx: Dynamic route        │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│        Server Components (RSC)                      │
│  - Data fetching en servidor                        │
│  - SEO-friendly, no JS en cliente                   │
│  - Cache: "no-store" para datos frescos             │
└──────────────────┬──────────────────────────────────┘
                   │ Props
┌──────────────────▼──────────────────────────────────┐
│        Client Components (cuando necesario)         │
│  - "use client" directive                           │
│  - Interactividad: forms, video player              │
│  - useState, useEffect, event handlers              │
└─────────────────────────────────────────────────────┘
```

**Archivos clave**:
- `Frontend/src/app/course/[slug]/page.tsx` (Server Component)
- `Frontend/src/components/VideoPlayer/VideoPlayer.tsx` (Client Component)

**Ventajas**:
- ✅ SEO optimizado (HTML renderizado en servidor)
- ✅ Menor JS enviado al cliente
- ✅ Data fetching más rápido
- ✅ Streaming de componentes

### 3. Android - Clean Architecture + MVI

```
┌─────────────────────────────────────────────────────┐
│        Presentation Layer (MVI)                     │
│  - ViewModel: CourseListViewModel                   │
│  - UiState: Single source of truth (StateFlow)      │
│  - Jetpack Compose UI (declarativo)                 │
└──────────────────┬──────────────────────────────────┘
                   │ Interfaces
┌──────────────────▼──────────────────────────────────┐
│        Domain Layer                                 │
│  - Models: Course (pure Kotlin, no Android deps)    │
│  - Repository Interfaces (abstracciones)            │
└──────────────────┬──────────────────────────────────┘
                   │ Implementación
┌──────────────────▼──────────────────────────────────┐
│        Data Layer                                   │
│  - RemoteCourseRepository (implementación)          │
│  - Retrofit + OkHttp (networking)                   │
│  - Mappers: CourseDTO → Course                      │
└─────────────────────────────────────────────────────┘
```

**Archivos clave**:
- `data/repositories/RemoteCourseRepository.kt`
- `presentation/courses/viewmodel/CourseListViewModel.kt`
- `presentation/courses/state/CourseListUiState.kt`

**MVI Pattern**:
```kotlin
// UI State (Single source of truth)
data class CourseListUiState(
    val courses: List<Course> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

// ViewModel expone StateFlow
class CourseListViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(CourseListUiState())
    val uiState: StateFlow<CourseListUiState> = _uiState.asStateFlow()
}
```

**Ventajas**:
- ✅ Unidirectional data flow (predecible)
- ✅ Testeable (cada capa independiente)
- ✅ Separation of concerns
- ✅ No dependencias Android en Domain layer

### 4. iOS - Clean Architecture + MVVM

```
┌─────────────────────────────────────────────────────┐
│        Presentation Layer (MVVM)                    │
│  - ViewModel: @Published properties (Observable)    │
│  - SwiftUI Views (declarativo)                      │
│  - Combine: Reactive programming                    │
└──────────────────┬──────────────────────────────────┘
                   │ Protocols
┌──────────────────▼──────────────────────────────────┐
│        Domain Layer                                 │
│  - Models: Course, Class, Teacher                   │
│  - Repository Protocols (abstracciones)             │
└──────────────────┬──────────────────────────────────┘
                   │ Implementación
┌──────────────────▼──────────────────────────────────┐
│        Data Layer                                   │
│  - RemoteCourseRepository (implementación)          │
│  - NetworkManager + URLSession                      │
│  - Mappers: CourseDTO → Course                      │
└─────────────────────────────────────────────────────┘
```

**Archivos clave**:
- `Data/Repositories/RemoteCourseRepository.swift`
- `Presentation/ViewModels/CourseListViewModel.swift`
- `Services/NetworkManager.swift`

**MVVM Pattern**:
```swift
@MainActor
class CourseListViewModel: ObservableObject {
    // Published properties (observable)
    @Published var courses: [Course] = []
    @Published var isLoading: Bool = false
    @Published var errorMessage: String? = nil

    // Computed properties
    var filteredCourses: [Course] {
        courses.filter { /* filter logic */ }
    }
}
```

**Ventajas**:
- ✅ SwiftUI-friendly (@Published + @ObservedObject)
- ✅ Reactive con Combine (debounce, publishers)
- ✅ Protocol-oriented design
- ✅ Swift Concurrency (async/await nativo)

---

## Comunicación Inter-Plataforma

### Contrato API Unificado

Todas las plataformas (Frontend, Android, iOS) consumen el **mismo contrato REST**:

| Plataforma | Base URL | Notas |
|------------|----------|-------|
| **Backend** | `0.0.0.0:8000` | Bind a todas las interfaces |
| **Frontend** | `http://localhost:8000` | Acceso directo a localhost |
| **Android** | `http://10.0.2.2:8000` | Emulator mapeo a host localhost |
| **iOS** | `http://localhost:8000` | Simulator acceso directo |

### Estrategia de Manejo de Errores

Todos los clientes implementan manejo consistente:

**HTTP Status Codes**:
- `200 OK` - Éxito
- `201 Created` - Recurso creado
- `204 No Content` - Éxito sin body
- `400 Bad Request` - Error de validación
- `404 Not Found` - Recurso no encontrado
- `500 Internal Server Error` - Error del servidor

**Formato de Error Backend**:
```json
{
  "detail": "Course with id 999 not found",
  "error_code": "COURSE_NOT_FOUND"
}
```

**Client-side Error Handling**:
- **Frontend**: Clase `ApiError` con `status`, `code`, `details`
- **Android**: Tipo `Result<T>` con `.onSuccess` / `.onFailure`
- **iOS**: `NetworkError` enum con casos específicos

### TypeScript Types (Frontend)

**Archivo**: `Frontend/src/types/index.ts`

```typescript
interface Course {
  id: number;
  name: string;
  description: string;
  thumbnail: string;
  slug: string;
  average_rating?: number;   // 0-5
  total_ratings?: number;
}

interface CourseDetail extends Course {
  classes: Class[];
  teacher_id: number[];
}

interface Class {
  id: number;
  title: string;
  description: string;
  video: string;
  slug: string;
}
```

**Archivo**: `Frontend/src/types/rating.ts`

```typescript
interface CourseRating {
  id: number;
  course_id: number;
  user_id: number;
  rating: number;           // 1-5
  created_at: string;       // ISO 8601
  updated_at: string;
}

interface RatingRequest {
  user_id: number;
  rating: number;           // 1-5
}

interface RatingStats {
  average_rating: number;   // 0-5
  total_ratings: number;
}
```

---

## Comandos de Desarrollo

### Backend

**IMPORTANTE**: Todos los comandos deben ejecutarse dentro del contenedor Docker API. Antes de ejecutar cualquier comando, verifica que el contenedor esté funcionando y revisa el `Makefile` con los comandos disponibles.

```bash
cd Backend

# Iniciar servicios Docker (DB + API)
make start

# Detener servicios
make stop

# Ver estado de containers
make ps

# Ver logs en tiempo real
make logs

# Ejecutar migraciones
make migrate

# Crear nueva migración
make create-migration MSG="descripción del cambio"

# Poblar datos de prueba
make seed

# Reset completo de datos (drop + seed)
make seed-fresh

# Ejecutar tests
make test

# Entrar al shell del contenedor API
make shell

# Ejecutar comando dentro del contenedor
make exec CMD="python app/db/seed.py"
```

### Frontend

```bash
cd Frontend

# Servidor de desarrollo (localhost:3000)
yarn dev

# Build de producción
yarn build

# Iniciar servidor de producción
yarn start

# Ejecutar tests (Vitest)
yarn test

# Tests en modo watch
yarn test:watch

# Coverage
yarn test:coverage

# Linter
yarn lint

# Fix lint errors
yarn lint:fix

# Type checking
yarn type-check
```

### Mobile

#### Android
```bash
cd Mobile/PlatziFlixAndroid

# Build debug APK
./gradlew assembleDebug

# Install en dispositivo/emulador
./gradlew installDebug

# Ejecutar tests unitarios
./gradlew test

# Ejecutar tests instrumentados
./gradlew connectedAndroidTest

# Clean build
./gradlew clean
```

#### iOS
```bash
cd Mobile/PlatziFlixiOS

# Build con xcodebuild
xcodebuild -scheme PlatziFlixiOS -configuration Debug

# Ejecutar tests
xcodebuild test -scheme PlatziFlixiOS

# O abrir en Xcode
open PlatziFlixiOS.xcodeproj

# Ejecutar desde Xcode: Cmd + R
# Tests desde Xcode: Cmd + U
```

---

## URLs del Sistema

- **Backend API**: http://localhost:8000
- **Frontend Web**: http://localhost:3000
- **API Docs (Swagger)**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

---

## Base de Datos

### Configuración Docker
```yaml
Database: platziflix_db
User: platziflix_user
Password: platziflix_password
Port: 5432
Host: localhost (o db desde dentro del container)
```

### Conexión desde host
```bash
psql -h localhost -p 5432 -U platziflix_user -d platziflix_db
# Password: platziflix_password
```

### Migraciones Alembic
- **Ubicación**: `Backend/app/alembic/versions/`
- **Crear migración**: `make create-migration MSG="add new field"`
- **Aplicar migraciones**: `make migrate`
- **Rollback**: `cd Backend && docker-compose exec api alembic downgrade -1`

### Schema Actual

**Tablas**:
- `teachers` - Profesores
- `courses` - Cursos
- `lessons` - Lecciones de un curso
- `course_teachers` - Asociación M:N entre cursos y profesores
- `course_ratings` - Ratings de cursos (1-5 estrellas)

**Indexes**:
- `courses.slug` (UNIQUE, BTREE)
- `teachers.email` (UNIQUE, BTREE)
- `lessons.course_id` (BTREE)
- `lessons.slug` (BTREE)
- `course_ratings.course_id` (BTREE)
- `course_ratings.user_id` (BTREE)

**Constraints**:
- `course_ratings.rating` CHECK (rating >= 1 AND rating <= 5)
- `course_ratings.UNIQUE(course_id, user_id, deleted_at)` - Una rating activa por usuario

---

## Funcionalidades Implementadas

- ✅ **Catálogo de cursos** con grid estilo Netflix
- ✅ **Detalle de cursos** (profesores, lecciones, clases)
- ✅ **Navegación por slug** SEO-friendly
- ✅ **Reproductor de video** integrado
- ✅ **Sistema de ratings** (1-5 estrellas)
  - Crear/actualizar rating
  - Ver estadísticas agregadas
  - Rating promedio y total de ratings
- ✅ **Health checks** de API y DB
- ✅ **Apps móviles nativas**
  - Android (Kotlin + Jetpack Compose)
  - iOS (Swift + SwiftUI)
- ✅ **Testing** en todas las capas
  - Backend: pytest
  - Frontend: Vitest + React Testing Library
  - Mobile: Unit tests

---

## Consideraciones de Desarrollo

### Generales

1. **Docker obligatorio** para el backend (DB + API)
2. **API REST** como única fuente de verdad para Frontend/Mobile
3. **Convenciones de naming**:
   - Python (Backend): `snake_case`
   - TypeScript (Frontend): `camelCase`
   - Kotlin (Android): `camelCase`
   - Swift (iOS): `camelCase`
   - Clases/Componentes: `PascalCase` (todos)

### Backend

4. **Service Layer Pattern**: Toda la lógica de negocio en `CourseService`
5. **Soft Deletes**: Usar `deleted_at` para preservar historial
6. **Migraciones obligatorias**: Todo cambio de schema requiere migración Alembic
7. **Testing requerido**: Tests para nuevos endpoints y servicios
8. **Validación Pydantic**: Schemas para request/response
9. **SQL Aggregations**: Cálculos de ratings en DB (performance)

### Frontend

10. **TypeScript strict mode**: Activado en `tsconfig.json`
11. **Server Components primero**: Usar RSC para data fetching
12. **Client Components solo cuando necesario**: Interactividad, forms, video
13. **CSS Modules**: Estilos con scope local
14. **Testing requerido**: Tests para nuevos componentes

### Mobile

15. **Clean Architecture**: Separación estricta de capas
16. **Repository Pattern**: Abstracción de data sources
17. **Mapper Pattern**: DTOs → Domain models
18. **Android**:
    - MVI para state management
    - Jetpack Compose para UI
    - Retrofit para networking
19. **iOS**:
    - MVVM con Combine
    - SwiftUI para UI
    - URLSession nativo

### Seguridad

20. **Nunca commitear**: `.env`, `credentials.json`, secrets
21. **Validación de entrada**: En API y frontend
22. **SQL Injection**: Protección con SQLAlchemy ORM
23. **CORS**: Configurado en FastAPI para desarrollo

---

## Roadmap / Features Pendientes

- ⬜ Autenticación de usuarios (JWT)
- ⬜ Autorización por roles (admin, estudiante)
- ⬜ Progreso de cursos por usuario
- ⬜ Comentarios en lecciones
- ⬜ Sistema de búsqueda avanzada
- ⬜ Filtros por categoría/nivel
- ⬜ Recomendaciones personalizadas
- ⬜ Notificaciones push (mobile)
- ⬜ Modo offline (mobile)
- ⬜ Cache de videos (mobile)

---

## Comandos Útiles Rápidos

```bash
# 🚀 Iniciar desarrollo completo
cd Backend && make start          # Terminal 1: Backend
cd Frontend && yarn dev           # Terminal 2: Frontend

# 🔄 Reset completo de datos
cd Backend && make seed-fresh

# 📋 Ver logs
cd Backend && make logs

# 🧪 Ejecutar todos los tests
cd Backend && make test           # Backend tests
cd Frontend && yarn test          # Frontend tests

# 🛠️ Crear migración
cd Backend && make create-migration MSG="add new field"

# 🐛 Debug backend
cd Backend && make shell          # Entrar al container
python app/db/seed.py            # Ejecutar script

# 📦 Build producción
cd Frontend && yarn build         # Next.js build
cd Mobile/PlatziFlixAndroid && ./gradlew assembleRelease
```

---

## Troubleshooting

### Backend no inicia
```bash
# Verificar containers
cd Backend && docker-compose ps

# Ver logs
cd Backend && docker-compose logs api

# Recrear containers
cd Backend && docker-compose down -v
cd Backend && make start
```

### Migraciones fallan
```bash
# Ver estado actual
cd Backend && docker-compose exec api alembic current

# Ver historial
cd Backend && docker-compose exec api alembic history

# Rollback una versión
cd Backend && docker-compose exec api alembic downgrade -1
```

### Frontend no conecta al backend
- Verificar que Backend esté corriendo en `localhost:8000`
- Revisar CORS en `Backend/app/main.py`
- Verificar URL en `Frontend/src/services/ratingsApi.ts`

### Android no conecta al backend
- Usar `http://10.0.2.2:8000` (no `localhost`)
- Verificar permiso INTERNET en `AndroidManifest.xml`
- Revisar logs en Logcat

### iOS no conecta al backend
- Verificar que Simulator y backend estén en mismo host
- Revisar `Info.plist` para App Transport Security
- Usar `http://localhost:8000`

---

## Recursos Adicionales

- **FastAPI Docs**: https://fastapi.tiangolo.com
- **Next.js Docs**: https://nextjs.org/docs
- **Jetpack Compose**: https://developer.android.com/jetpack/compose
- **SwiftUI**: https://developer.apple.com/xcode/swiftui

---

**Esta memoria contiene toda la información necesaria para continuar el desarrollo del proyecto Platziflix. Mantén este documento actualizado con cada cambio significativo en la arquitectura.**