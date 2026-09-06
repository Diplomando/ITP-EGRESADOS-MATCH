# ITP-EgresadosMatch

## 1. Descripción del proyecto

Observatorio Laboral del Putumayo y Bolsa de Empleo con *matching* semántico de hojas de vida (CV) frente a ofertas laborales, orientado a egresados del ITP. Combina un módulo de intermediación laboral con indicadores del Observatorio Laboral y Empresarial (OLE).

## 2. Stack tecnológico

| Capa | Tecnología |
|---|---|
| Frontend | React + Vite, TypeScript, Tailwind CSS, shadcn/ui |
| Backend | NestJS (TypeScript) |
| Microservicio de matching | Python + FastAPI |
| Persistencia | Base de datos relacional vía TypeORM |
| Autenticación | JWT + Passport.js |
| CI/CD | GitHub Actions |

## 3. Arquitectura y estructura de carpetas

```
ITP-EgresadosMatch/
├── frontend/                  # Aplicación React
│   └── src/
│       ├── pages/              # Vistas (Login, Dashboard, HojaDeVida, Ofertas, etc.)
│       ├── components/ui/      # Primitivos base (shadcn/ui)
│       ├── features/           # Lógica por dominio (egresados, empresas, ofertas)
│       ├── hooks/               # Custom hooks
│       ├── services/            # Clientes HTTP / consumo de la API
│       ├── context/              # Contextos globales (Auth, Theme)
│       └── router/               # Configuración de rutas
├── backend/                    # API NestJS
│   └── src/
│       ├── modules/
│       │   ├── auth/             # Autenticación y guards por rol
│       │   ├── egresados/        # CRUD egresados y hojas de vida
│       │   ├── empresas/         # CRUD empresas y ofertas
│       │   ├── matching/         # Integración con microservicio NLP
│       │   └── observatorio/     # Indicadores y estadísticas OLE
│       ├── common/               # Filtros, pipes, decoradores compartidos
│       └── config/                # Configuración de entorno y base de datos
├── matching-service/           # Microservicio de matching semántico (Python/FastAPI)
│   ├── app/
│   └── requirements.txt
├── docs/                        # Documentación, diagramas UML, anteproyecto
├── .github/workflows/           # Pipelines de CI/CD
└── README.md
```

## 4. Variables de entorno

Cada subproyecto maneja su propio `.env` (no versionado; usar `.env.example` como plantilla):

- **backend/**: `DATABASE_URL`, `JWT_SECRET`, `JWT_EXPIRES_IN`, `MATCHING_SERVICE_URL`, `PORT`.
- **matching-service/**: `MODEL_NAME`, `PORT`.
- **frontend/**: `VITE_API_URL`.

Nunca commitear secretos ni credenciales reales.

## 5. Comandos de desarrollo

### Frontend / Backend (npm)

```bash
npm install                # instalación de dependencias
npm run dev                # frontend: hot reload (Vite)
npm run start:dev          # backend: hot reload (NestJS)
npm run build               # build de producción
npm run preview             # previsualizar build (frontend)
npm run lint                # linter
npm run lint:fix
npm run format
npm run test                 # unitarios
npm run test:watch
npm run test:cov             # cobertura
npm run test:e2e             # e2e (backend)
npx tsc --noEmit             # verificación de tipos
```

### Microservicio de matching (Python)

```bash
python -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows

pip install -r requirements.txt
uvicorn app.main:app --reload

flake8 app/
black app/

pytest
pytest --cov=app
```

## 6. Checklist previo a un commit

Antes de dar por terminada una tarea, confirmar que pasan:

1. `npm run lint` (frontend y backend)
2. `npm run test` (frontend y backend)
3. `npx tsc --noEmit`
4. `npm run build`
5. `pytest` (si se tocó `matching-service/`)

## 7. Convenciones de Git y flujo de trabajo

- **Ramas**: `feature/<nombre>`, `fix/<nombre>`, `docs/<nombre>`, `chore/<nombre>`.
- **Commits**: Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`).
- **Pull Requests**: obligatorios para integrar a `main`/`develop`; requieren al menos una revisión y CI en verde.
- **Issues**: usar plantillas para bugs y features; vincular PRs a su issue correspondiente.
- No hacer *rebase* ni *force-push* sobre ramas compartidas sin avisar al equipo.

## 8. Estándares del frontend

- Componentes funcionales con Hooks; un componente por archivo, PascalCase.
- Estado local (`useState`/`useReducer`) vs. estado de servidor (TanStack Query) — nunca mezclar. Context API solo para auth/tema.
- Separación entre componentes "de datos" (containers, consumen hooks/servicios) y de presentación (dumb components, solo props).
- TypeScript estricto; `any` prohibido salvo justificación explícita; tipos compartidos con backend en `types/`/`shared/` cuando sea posible.
- Tailwind CSS como estándar; evitar CSS custom salvo casos puntuales; shadcn/ui sin lógica de negocio.
- Formularios: React Hook Form + Zod, esquemas reutilizados con el backend cuando este valide con Zod/class-validator.
- Assets optimizados (WebP/SVG), lazy loading, code-splitting por ruta (`React.lazy`/`Suspense`).
- Nomenclatura: hooks con prefijo `use`, servicios en `*.service.ts`, tipos en `*.types.ts`.

## 9. Estándares del backend y la API

- Un módulo NestJS por dominio (`auth`, `egresados`, `empresas`, `matching`, `observatorio`), cada uno con `controller`, `service`, `module` y `dto/`.
- DTOs con `class-validator` + `class-transformer` en todos los endpoints; nunca exponer entidades de BD directamente (usar DTOs de salida).
- Capas: Controller (HTTP) → Service (lógica de negocio) → Repository/ORM (persistencia). No acceder a la BD desde el controller.
- Rutas REST: `/api/v1/{recurso}`, verbos HTTP semánticos, respuestas consistentes (`{ data, message }` o `{ error }`).
- Documentación con Swagger/OpenAPI (`@nestjs/swagger`), disponible en `/api/docs`.
- Testing: unitarios por servicio (mockeando repositorios) y e2e por módulo con BD de test.

## 10. Base de datos y ORM

- Migraciones versionadas con TypeORM; **nunca** `synchronize: true` en producción.
- Nombres de tablas y columnas en `snake_case`.
- El microservicio de matching no accede directamente a la BD principal; se comunica vía API con el backend.

## 11. Seguridad y autenticación

- JWT con Passport.js; Guards por rol (`@Roles('egresado' | 'empresa' | 'admin')`).
- Nunca exponer contraseñas ni tokens en logs.
- Variables sensibles solo por entorno (`.env`), nunca hardcodeadas.

## 12. Manejo de errores

- Filtros globales de excepciones (`HttpExceptionFilter`) que devuelvan códigos HTTP correctos y mensajes uniformes.
- El microservicio de matching debe responder con códigos de error estándar (4xx/5xx) y payload consistente ante fallos del modelo o de entrada.

## 13. CI/CD

- Pipelines en `.github/workflows/` para frontend, backend y microservicio.
- Cada pipeline ejecuta: instalación de dependencias, lint, tests y build antes de permitir merge a `main`.
- Despliegue automático (o manual gatillado) solo desde `main` tras pasar CI.

## 14. Restricciones del agente

- No crear archivos, carpetas o dependencias no solicitadas explícitamente, salvo estrictamente necesarias (y en ese caso, mencionarlo).
- No modificar archivos fuera del alcance de la tarea pedida.
- No renombrar ni mover archivos existentes sin confirmación previa.
- No eliminar código, tests o configuración existente sin preguntar, aunque parezca "no usado".
- No reescribir un módulo completo cuando se pidió un cambio puntual.
- No hacer comentarios en el código.