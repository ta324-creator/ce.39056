# ConectandoEnsenada.org — Plan Maestro de Arquitectura
**Versión 1.0 | Documento Confidencial**
*Preparado por: Arquitecto Principal de Software*

---

## Índice

1. [Análisis del Proyecto](#1-análisis-del-proyecto)
2. [Arquitectura General del Sistema](#2-arquitectura-general-del-sistema)
3. [Estructura del Proyecto](#3-estructura-del-proyecto)
4. [Stack Tecnológico Recomendado](#4-stack-tecnológico-recomendado)
5. [Base de Datos — Diseño de Alto Nivel](#5-base-de-datos--diseño-de-alto-nivel)
6. [Módulos Principales](#6-módulos-principales)
7. [Roadmap por Fases](#7-roadmap-por-fases)
8. [Riesgos Técnicos](#8-riesgos-técnicos)
9. [Decisiones Arquitectónicas Importantes](#9-decisiones-arquitectónicas-importantes)
10. [Preguntas Críticas Antes de Comenzar](#10-preguntas-críticas-antes-de-comenzar)

---

## 1. Análisis del Proyecto

### 1.1 ¿Qué estamos construyendo realmente?

ConectandoEnsenada.org no es una aplicación web. Es una **plataforma urbana digital** — un concepto relativamente nuevo que combina lo que en la industria se conoce como *City OS* (sistema operativo de ciudad) con un *super-app* de escala local.

Los referentes más cercanos a nivel global son:
- **Nextdoor** (comunidad hiperlocal)
- **Yelp + LinkedIn + Eventbrite + Airbnb + MercadoLibre** — todo bajo un mismo techo
- **Pero ninguno está localizado para Ensenada**, lo cual es precisamente nuestra ventaja competitiva.

### 1.2 La complejidad real del proyecto

Este proyecto tiene una complejidad técnica equivalente a 6-8 startups independientes que comparten identidad, usuarios y datos. Cada módulo tiene sus propios patrones:

| Módulo | Complejidad | Razón |
|---|---|---|
| Directorio de Negocios | Media | Datos semi-estáticos, SEO intensivo |
| Empleos | Media | Ciclo de vida de publicaciones, aplicaciones |
| Marketplace | Alta | Pagos, transacciones, disputas, seguridad |
| Eventos | Media | Fechas, capacidad, ticketing |
| Noticias / CMS | Media | Flujo editorial, autores, categorías |
| Turismo | Media | Contenido curado, itinerarios, SEO turístico |
| Restaurantes | Media-Alta | Menús, reseñas, reservaciones futuras |
| IA Integrada | Muy Alta | RAG, embeddings, fine-tuning, contexto local |
| Dashboard Negocios | Alta | Multi-tenancy, analytics, pagos SaaS |
| Panel Administrativo | Alta | Moderación, roles, permisos granulares |
| Publicidad | Alta | Impresiones, clics, facturación, targeting |
| API Pública | Alta | Rate limiting, versioning, autenticación |

### 1.3 Usuarios del sistema

Identificamos **5 tipos de usuarios** con necesidades distintas:

```
USUARIOS ANÓNIMOS
  └── Turistas (principalmente EE.UU. / Canadá)
  └── Visitantes ocasionales

USUARIOS REGISTRADOS
  └── Residentes locales
  └── Profesionistas buscando empleo

DUEÑOS DE NEGOCIO / ORGANIZACIONES
  └── Comercios locales (plan gratuito / premium)
  └── Organizaciones sin fines de lucro

MODERADORES / EDITORES
  └── Equipo editorial de noticias
  └── Moderadores de contenido

ADMINISTRADORES
  └── Super Admin (acceso total)
  └── Admin (gestión operativa)
```

---

## 2. Arquitectura General del Sistema

### 2.1 Patrón Arquitectónico: Modular Monolith → Preparado para Microservicios

**Mi recomendación: comenzar con un Monolito Modular.**

Muchos equipos cometen el error de empezar con microservicios desde el día uno. Esto es una trampa. Los microservicios son una solución a problemas de escala que aún no tienes. Agregan:
- Latencia entre servicios
- Complejidad operacional (Kubernetes, service mesh, etc.)
- Necesidad de un equipo de DevOps dedicado
- Dificultad para iterar rápido

La arquitectura correcta para ConectandoEnsenada es:

```
FASE 1-3: Monolito Modular
  ┌─────────────────────────────────────────────┐
  │              Next.js Application             │
  │                                             │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
  │  │ Módulo   │  │ Módulo   │  │ Módulo   │  │
  │  │Negocios  │  │ Empleos  │  │ Events   │  │
  │  └──────────┘  └──────────┘  └──────────┘  │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
  │  │ Módulo   │  │ Módulo   │  │ Módulo   │  │
  │  │  News    │  │   AI     │  │  Market  │  │
  │  └──────────┘  └──────────┘  └──────────┘  │
  └─────────────────────────────────────────────┘
         │                │               │
    Supabase         Meilisearch       OpenAI
    (DB + Auth       (Búsqueda)        (IA)
    + Storage)

FASE 4+: Extraer a Microservicios según demanda
  (Solo cuando el módulo lo justifique por tráfico o equipo)
```

Los módulos están **bien definidos internamente** desde el inicio. Si en el futuro un módulo (como Marketplace) necesita escalar independientemente, podemos extraerlo sin reescribir desde cero. Esto es arquitectura inteligente: **diseñar para el futuro sin pagar su costo hoy**.

### 2.2 Diagrama de Alto Nivel

```
                    ┌─────────────────────────────────┐
                    │         CLIENTES / USUARIOS      │
                    │  Browser  │  Mobile  │   API     │
                    └───────────┴──────────┴───────────┘
                                     │
                    ┌────────────────▼────────────────┐
                    │           VERCEL CDN              │
                    │     (Edge Network Global)         │
                    └────────────────┬────────────────┘
                                     │
                    ┌────────────────▼────────────────┐
                    │        NEXT.JS APP (Web)          │
                    │                                  │
                    │  ┌─────────────────────────────┐ │
                    │  │    App Router (RSC + SSR)    │ │
                    │  │   Layouts / Pages / API      │ │
                    │  └─────────────────────────────┘ │
                    │  ┌──────────┬──────┬──────────┐  │
                    │  │ Auth     │ SEO  │ Analytics│  │
                    │  │ (Supa)   │Layer │  Layer   │  │
                    │  └──────────┴──────┴──────────┘  │
                    └────┬──────┬──────┬──────┬───────┘
                         │      │      │      │
              ┌──────────▼─┐ ┌──▼───┐ ┌▼────┐ ┌▼──────────┐
              │  Supabase  │ │ Mei- │ │Open │ │  Stripe   │
              │            │ │ lise-│ │ AI  │ │           │
              │ PostgreSQL │ │ arch │ │     │ │ Payments  │
              │ Auth       │ │      │ │ GPT │ │ + Webhooks│
              │ Storage    │ │Search│ │ 4o  │ │           │
              │ Realtime   │ │      │ │     │ │           │
              │ Edge Func. │ └──────┘ └─────┘ └───────────┘
              └────────────┘
```

### 2.3 Estrategia de Rendering (Crítico para SEO)

Esta es una de las decisiones más importantes del proyecto. Con Next.js App Router tenemos múltiples estrategias:

| Estrategia | Uso en este proyecto | Razón |
|---|---|---|
| **SSG** (Static Site Generation) | Páginas de turismo, landing pages | Contenido estático, máximo SEO y velocidad |
| **ISR** (Incremental Static Regen.) | Negocios, restaurantes, noticias | Cambia ocasionalmente, SEO crítico |
| **SSR** (Server-Side Rendering) | Búsquedas, resultados filtrados | Dinámico, indexable por Google |
| **CSR** (Client-Side Rendering) | Dashboards, admin panel | No necesita SEO, altamente interactivo |
| **Streaming / Suspense** | Páginas complejas | UX perceptivamente más rápida |

**Regla general que aplicaremos:**
> *"Si Google lo necesita indexar, lo renderizamos en el servidor. Si solo el usuario lo ve, lo renderizamos en el cliente."*

---

## 3. Estructura del Proyecto

### 3.1 Monorepo con Turborepo

Recomiendo usar un **monorepo** desde el primer día. La razón principal: cuando llegues a la aplicación móvil (Expo/React Native), podrás compartir tipos, lógica de negocio y componentes sin duplicar código.

```
conectando-ensenada/                    ← Raíz del monorepo
│
├── apps/
│   ├── web/                            ← Aplicación Next.js principal
│   │   ├── app/
│   │   │   ├── (public)/               ← Rutas públicas (sin auth)
│   │   │   │   ├── page.tsx            ← Home / Landing
│   │   │   │   ├── negocios/
│   │   │   │   │   ├── page.tsx        ← Directorio de negocios
│   │   │   │   │   └── [slug]/
│   │   │   │   │       └── page.tsx    ← Perfil de negocio
│   │   │   │   ├── empleos/
│   │   │   │   ├── marketplace/
│   │   │   │   ├── eventos/
│   │   │   │   ├── noticias/
│   │   │   │   ├── turismo/
│   │   │   │   ├── restaurantes/
│   │   │   │   └── organizaciones/
│   │   │   │
│   │   │   ├── (auth)/                 ← Rutas de autenticación
│   │   │   │   ├── login/
│   │   │   │   ├── registro/
│   │   │   │   └── callback/
│   │   │   │
│   │   │   ├── (dashboard)/            ← Portal de negocios (auth requerida)
│   │   │   │   └── dashboard/
│   │   │   │       ├── page.tsx        ← Overview del negocio
│   │   │   │       ├── perfil/
│   │   │   │       ├── empleos/
│   │   │   │       ├── analytics/
│   │   │   │       ├── publicidad/
│   │   │   │       └── suscripcion/
│   │   │   │
│   │   │   ├── (admin)/                ← Panel administrativo
│   │   │   │   └── admin/
│   │   │   │       ├── page.tsx
│   │   │   │       ├── negocios/
│   │   │   │       ├── usuarios/
│   │   │   │       ├── contenido/
│   │   │   │       ├── moderacion/
│   │   │   │       └── analytics/
│   │   │   │
│   │   │   └── api/                    ← API Routes (Next.js)
│   │   │       ├── ai/
│   │   │       │   └── chat/
│   │   │       ├── search/
│   │   │       ├── webhooks/
│   │   │       │   ├── stripe/
│   │   │       │   └── supabase/
│   │   │       └── v1/                 ← API Pública (futura)
│   │   │
│   │   ├── components/
│   │   │   ├── ui/                     ← Componentes base (botones, inputs, etc.)
│   │   │   ├── layout/                 ← Header, Footer, Sidebar
│   │   │   ├── seo/                    ← JsonLD, MetaTags, Sitemap helpers
│   │   │   └── modules/                ← Componentes específicos por módulo
│   │   │       ├── businesses/
│   │   │       ├── jobs/
│   │   │       ├── events/
│   │   │       └── ai/
│   │   │
│   │   ├── lib/                        ← Utilidades y clientes de servicios
│   │   │   ├── supabase/
│   │   │   │   ├── client.ts           ← Cliente del browser
│   │   │   │   ├── server.ts           ← Cliente del servidor (SSR)
│   │   │   │   └── middleware.ts
│   │   │   ├── openai/
│   │   │   ├── stripe/
│   │   │   ├── search/                 ← Meilisearch client
│   │   │   └── utils/
│   │   │
│   │   └── modules/                    ← Lógica de negocio por módulo
│   │       ├── businesses/
│   │       │   ├── actions.ts          ← Server Actions
│   │       │   ├── queries.ts          ← Queries a la DB
│   │       │   └── types.ts
│   │       ├── jobs/
│   │       ├── marketplace/
│   │       ├── events/
│   │       ├── news/
│   │       ├── tourism/
│   │       └── ai/
│   │           ├── assistant.ts        ← Lógica del asistente de IA
│   │           ├── embeddings.ts       ← Generación de embeddings
│   │           └── rag.ts              ← Retrieval Augmented Generation
│   │
│   └── mobile/                         ← (Fase 5) React Native / Expo
│       └── README.md                   ← Placeholder para el futuro
│
├── packages/
│   ├── types/                          ← Tipos TypeScript compartidos
│   │   ├── database.ts                 ← Tipos generados por Supabase
│   │   ├── api.ts
│   │   └── index.ts
│   │
│   ├── ui/                             ← Componentes UI compartidos (web + mobile)
│   │   └── src/
│   │
│   └── config/                         ← Configuraciones compartidas
│       ├── eslint/
│       ├── typescript/
│       └── tailwind/
│
├── infrastructure/
│   ├── supabase/
│   │   ├── migrations/                 ← Migraciones SQL versionadas
│   │   ├── functions/                  ← Supabase Edge Functions
│   │   │   ├── send-notification/
│   │   │   ├── generate-embeddings/
│   │   │   └── moderate-content/
│   │   └── seed/                       ← Datos iniciales
│   │
│   └── meilisearch/
│       └── config/                     ← Índices y configuración
│
├── turbo.json                          ← Configuración Turborepo
├── package.json
└── README.md
```

---

## 4. Stack Tecnológico Recomendado

### 4.1 Validación del Stack Propuesto

Tu stack propuesto es sólido. Tengo algunas observaciones importantes:

#### ✅ Lo que apruebo completamente

| Tecnología | Veredicto | Razón |
|---|---|---|
| **Next.js 15 + App Router** | ✅ Perfecto | RSC, SEO, performance, ecosystem |
| **TypeScript** | ✅ Obligatorio | La escala del proyecto lo exige |
| **Tailwind CSS** | ✅ Excelente | Velocidad de desarrollo, consistencia |
| **Supabase** | ✅ Muy buena elección | Auth + DB + Storage + Realtime en uno |
| **PostgreSQL** | ✅ Ideal | Relacional + JSON + pgvector para IA |
| **Vercel** | ✅ Correcto | La mejor opción para Next.js |
| **OpenAI** | ✅ Para IA generativa | GPT-4o + Embeddings |

#### ⚠️ Observaciones importantes que debes considerar

**1. Stripe solo → Stripe + MercadoPago**

> Ensenada es México. Una porción significativa de tus usuarios no tendrá tarjetas internacionales o preferirán MercadoPago. Si ofreces Marketplace o planes premium para negocios sin MercadoPago, perderás conversiones.

Recomendación: **Stripe para usuarios con tarjetas internacionales + MercadoPago para el mercado local mexicano.** Ambos pueden coexistir. Evalúa el costo de implementar los dos al inicio vs. el costo de perder ventas.

**2. Meilisearch: gran elección, pero con matices**

Meilisearch es excelente. Pero tienes una decisión importante:

| Opción | Costo | Control | Complejidad |
|---|---|---|---|
| **Meilisearch Cloud** | ~$30-100/mes | Bajo | Mínima |
| **Meilisearch Self-hosted** | ~$10-20/mes (VPS) | Total | Media |
| **Supabase FTS (pg_trgm)** | $0 (incluido) | Total | Mínima |
| **Typesense Cloud** | Similar | Bajo | Mínima |

**Mi recomendación estratégica:** Usar **Supabase Full-Text Search** en Fase 1 (MVP). Es gratis, está integrado, y es suficientemente bueno para comenzar. Migrar a **Meilisearch Cloud** en Fase 2 cuando el volumen de datos justifique el costo y la complejidad adicional. Esto nos ahorra un servidor de infraestructura en las etapas iniciales.

**3. Google OAuth → Google OAuth + autenticación por email/teléfono**

No todos los usuarios locales tienen o usan Google. Supabase incluye autenticación por email/magic link y teléfono (OTP). Recomiendo habilitarlos desde el inicio.

#### 🆕 Adiciones que recomiendo

| Tecnología | Propósito | Justificación |
|---|---|---|
| **Turborepo** | Monorepo management | Preparación para mobile, packages compartidos |
| **Supabase pgvector** | Embeddings para IA | Ya incluido en PostgreSQL de Supabase |
| **Zod** | Validación de datos | TypeScript runtime validation en forms y API |
| **React Hook Form** | Formularios | Los mejores forms de React, sin re-renders |
| **Radix UI** | Componentes accesibles | Base accesible (WAI-ARIA) para el design system |
| **next-intl** | Internacionalización | Español + Inglés desde el inicio (turismo) |
| **Sentry** | Error monitoring | Captura errores en producción |
| **Resend** | Emails transaccionales | Mejor alternativa a SendGrid para Next.js |
| **Upstash Redis** | Rate limiting + cache | Sin servidor, por petición, ideal para Vercel |

### 4.2 Stack Final Recomendado

```
FRONTEND
  ├── Next.js 15 (App Router)
  ├── React 19
  ├── TypeScript 5
  ├── Tailwind CSS 4
  └── Radix UI + shadcn/ui

BACKEND / BaaS
  ├── Supabase (PostgreSQL + Auth + Storage + Realtime)
  ├── Next.js API Routes + Server Actions
  └── Supabase Edge Functions (para lógica serverless pesada)

BÚSQUEDA
  ├── Fase 1: Supabase FTS (pg_trgm + tsvector)
  └── Fase 2+: Meilisearch Cloud

PAGOS
  ├── Stripe (tarjetas internacionales)
  └── MercadoPago (mercado local MX) — evaluar en Fase 2

IA
  ├── OpenAI GPT-4o (chat + generación)
  ├── OpenAI text-embedding-3-small (embeddings)
  └── pgvector en Supabase (vector store)

INFRAESTRUCTURA
  ├── Vercel (hosting + CDN + Edge)
  ├── Upstash Redis (rate limiting, cache de sesión)
  └── Sentry (monitoring de errores)

COMUNICACIÓN
  ├── Resend (emails transaccionales)
  └── Supabase Realtime (notificaciones en tiempo real)

HERRAMIENTAS DE DESARROLLO
  ├── Turborepo (monorepo)
  ├── Zod (validación)
  ├── React Hook Form
  └── next-intl (ES/EN)
```

---

## 5. Base de Datos — Diseño de Alto Nivel

### 5.1 Estrategia de Esquemas PostgreSQL

Organizaremos la base de datos en **esquemas separados** por dominio. Esto mantiene todo organizado y facilita la seguridad con Row Level Security (RLS).

```sql
Esquema PUBLIC      -- Tablas compartidas (usuarios, categorías)
Esquema BUSINESSES  -- Directorio de negocios
Esquema JOBS        -- Bolsa de trabajo
Esquema MARKETPLACE -- Compra/venta
Esquema EVENTS      -- Eventos
Esquema CMS         -- Noticias y contenido editorial
Esquema TOURISM     -- Turismo y atractivos
Esquema ADS         -- Sistema de publicidad
Esquema ANALYTICS   -- Eventos de analítica
Esquema AI          -- Embeddings y contexto para IA
```

### 5.2 Entidades Principales

```
┌─────────────────────────────────────────────────────────┐
│                        USUARIOS                          │
│  users (auth.users de Supabase)                         │
│    ├── profiles (datos públicos del usuario)            │
│    └── user_roles (admin, moderator, business_owner...) │
└────────────────────────┬────────────────────────────────┘
                         │
         ┌───────────────┼────────────────┐
         │               │                │
┌────────▼───────┐ ┌─────▼──────┐ ┌──────▼──────┐
│   NEGOCIOS     │ │  EMPLEOS   │ │  MARKETPLACE│
│ businesses     │ │ jobs       │ │ listings    │
│ categories     │ │ categories │ │ categories  │
│ hours          │ │ applications│ │ transactions│
│ media          │ │            │ │ messages    │
│ reviews        │ └────────────┘ └─────────────┘
│ plans          │
└────────────────┘

┌─────────────┐  ┌───────────┐  ┌──────────────┐
│   EVENTOS   │  │  NOTICIAS │  │   TURISMO    │
│ events      │  │ articles  │  │ attractions  │
│ categories  │  │ authors   │  │ itineraries  │
│ attendees   │  │ categories│  │ categories   │
└─────────────┘  │ tags      │  └──────────────┘
                 └───────────┘

┌──────────────┐  ┌───────────┐  ┌──────────────┐
│  PUBLICIDAD  │  │ ANALÍTICA │  │    IA        │
│ ads          │  │ page_views│  │ embeddings   │
│ campaigns    │  │ events    │  │ chat_sessions│
│ impressions  │  │ sessions  │  │ knowledge_base│
└──────────────┘  └───────────┘  └──────────────┘
```

### 5.3 Tablas Clave — Descripción

**`profiles`** — Perfil público de cada usuario
```
id (uuid, FK → auth.users)
display_name, avatar_url, bio
phone, location
preferred_language (es|en)
created_at, updated_at
```

**`businesses`** — El corazón del directorio
```
id (uuid)
owner_id (FK → profiles)
name, slug (único, para URLs)
description, short_description
category_id, subcategory_id
address, city, state, zip
latitude, longitude          ← PostGIS para búsqueda geoespacial
phone, email, website
social_links (jsonb)         ← Flexible para cualquier red social
hours (jsonb)                ← Horarios estructurados
verified (boolean)
featured (boolean)
plan_type (free|starter|pro|enterprise)
status (active|pending|suspended|rejected)
avg_rating, review_count
created_at, updated_at
```

**`jobs`** — Bolsa de trabajo
```
id (uuid)
business_id (FK)
title, slug
description (richtext)
type (full_time|part_time|freelance|internship)
category_id
salary_min, salary_max, salary_currency
remote (boolean)
location
requirements (jsonb)
expires_at
status (active|paused|closed|expired)
application_count
created_at, updated_at
```

**`marketplace_listings`** — Marketplace
```
id (uuid)
seller_id (FK → profiles)
title, slug, description
price, currency
category_id
condition (new|like_new|good|fair)
images (jsonb array)
location
status (active|sold|reserved|removed)
view_count, contact_count
created_at, updated_at
```

**`articles`** — Sistema de noticias/CMS
```
id (uuid)
author_id (FK)
title, slug
excerpt, content (richtext)
featured_image_url
category_id
tags (text array)
status (draft|review|published|archived)
published_at
view_count
seo_title, seo_description  ← SEO específico por artículo
created_at, updated_at
```

**`embeddings`** — Para el sistema de IA
```
id (uuid)
entity_type (business|job|event|article|tourism_spot)
entity_id (uuid)
content (text)               ← Texto indexado
embedding (vector(1536))     ← pgvector
metadata (jsonb)
created_at, updated_at
```

### 5.4 Estrategia de Row Level Security (RLS)

Supabase RLS es nuestra capa de seguridad a nivel de base de datos. Ejemplos de políticas:

```
businesses:
  SELECT → todos (solo businesses.status = 'active')
  INSERT → solo usuarios autenticados
  UPDATE → solo el owner_id del negocio
  DELETE → solo admins

jobs:
  SELECT → todos
  INSERT → solo business_owners con plan activo
  UPDATE → solo el business_owner correspondiente

marketplace_listings:
  SELECT → todos
  INSERT → usuarios autenticados
  UPDATE → solo seller_id

articles:
  SELECT → todos (donde status = 'published')
  INSERT/UPDATE → editors y admins
```

---

## 6. Módulos Principales

### Módulo 1: Identidad y Autenticación
**Responsabilidad:** Gestionar quién es cada usuario y qué puede hacer.
- Registro / Login con Google OAuth, Email/Magic Link
- Perfiles de usuario con preferencias
- Sistema de roles granular (RBAC)
- Protección de rutas por rol

### Módulo 2: Directorio de Negocios
**Responsabilidad:** El módulo más crítico para SEO y retención.
- Perfiles de negocio completos con geolocalización
- Sistema de categorías jerárquico
- Horarios de atención
- Galería de fotos
- Sistema de reseñas y calificaciones
- Planes de membresía (gratuito, starter, pro)
- Verificación de negocios

### Módulo 3: Bolsa de Trabajo
**Responsabilidad:** Conectar empleadores con candidatos locales.
- Publicación de vacantes por negocios verificados
- Filtros por categoría, tipo y salario
- Sistema de aplicaciones (formulario + CV)
- Alertas de empleo por email
- Moderación de publicaciones

### Módulo 4: Marketplace
**Responsabilidad:** Compra y venta entre particulares y negocios.
- Listados de productos con imágenes
- Sistema de mensajería entre vendedor/comprador
- Integración con Stripe (y MercadoPago futuro)
- Sistema de reportes de fraude
- Categorías: inmuebles, vehículos, electrónica, servicios, etc.

### Módulo 5: Eventos
**Responsabilidad:** Agenda cultural y social de Ensenada.
- Publicación de eventos con fechas, lugar, precio
- Calendario visual de eventos
- Categorías: cultural, deportivo, gastronómico, musical
- Integración con mapas para ubicación
- Recordatorios por email

### Módulo 6: Noticias y CMS
**Responsabilidad:** Portal de noticias local con flujo editorial.
- Sistema de categorías y tags
- Editor de rich text (TipTap o similar)
- Flujo editorial: borrador → revisión → publicación
- SEO por artículo (meta tags personalizados)
- Galería de imágenes por artículo
- Comentarios (moderados)

### Módulo 7: Turismo
**Responsabilidad:** Guía turística digital de Ensenada.
- Directorio de atractivos turísticos
- Itinerarios sugeridos
- Categorías: playas, vinos, gastronomía, historia, aventura
- Orientado a turistas de EE.UU. (inglés prioritario aquí)
- Integración con Google Maps

### Módulo 8: Asistente de IA — "Asistente Ensenada"
**Responsabilidad:** El gran diferenciador de la plataforma.
- Chat con IA que conoce Ensenada en profundidad
- RAG (Retrieval-Augmented Generation) sobre toda la base de datos
- Responde: "¿Dónde comer mariscos con vista al mar?" → busca en nuestra DB
- Sugerencias personalizadas de negocios, eventos, rutas
- Responde en español e inglés automáticamente
- Embeddings de todo el contenido de la plataforma en pgvector

**Arquitectura de la IA:**
```
Usuario pregunta
      │
      ▼
Embed la pregunta (OpenAI)
      │
      ▼
Búsqueda de similitud en pgvector
      │
      ▼
Recuperar contexto relevante (negocios, eventos, artículos)
      │
      ▼
Construir prompt con contexto
      │
      ▼
GPT-4o genera respuesta fundamentada en datos reales
      │
      ▼
Respuesta al usuario con referencias a la plataforma
```

### Módulo 9: Dashboard de Negocios
**Responsabilidad:** Portal de autogestión para dueños de negocio.
- Edición completa del perfil
- Gestión de empleos publicados
- Analytics: vistas, clics, llamadas, mensajes
- Gestión de suscripción y facturación
- Responder reseñas
- Campañas de publicidad básica

### Módulo 10: Panel Administrativo
**Responsabilidad:** Control total de la plataforma.
- Dashboard con métricas globales
- Gestión de usuarios y roles
- Moderación de contenido con cola de revisión
- Gestión de planes y pagos
- Configuración global del sistema
- Logs de actividad

### Módulo 11: Sistema de Publicidad
**Responsabilidad:** Monetización a través de espacios publicitarios.
- Banner rotativo en homepage
- Listados destacados en directorios
- Patrocinios de secciones
- Seguimiento de impresiones y clics
- Facturación directa (integrada con Stripe)

### Módulo 12: API Pública
**Responsabilidad:** (Fase 5) Permitir integraciones externas.
- REST API versionada (/api/v1/)
- Autenticación por API key
- Rate limiting por plan
- Documentación con OpenAPI/Swagger
- Webhooks para eventos importantes

---

## 7. Roadmap por Fases

### Fase 1 — Fundamentos e MVP (Meses 1-4)
**Objetivo:** Tener algo real, usable y valioso en producción.

```
INFRAESTRUCTURA (Semanas 1-2)
  ├── Configurar monorepo con Turborepo
  ├── Next.js con TypeScript + Tailwind
  ├── Supabase: auth, DB, storage
  ├── Vercel: deploy, variables de entorno
  ├── CI/CD básico (GitHub Actions)
  └── Sentry para error monitoring

IDENTIDAD (Semanas 2-3)
  ├── Autenticación Google OAuth + Email
  ├── Perfiles de usuario
  └── Sistema de roles básico

DIRECTORIO DE NEGOCIOS (Semanas 3-8)
  ├── Schema de negocios en PostgreSQL
  ├── CRUD de negocios (solo admins al inicio)
  ├── Páginas SEO por negocio (/negocios/[slug])
  ├── Búsqueda básica con Supabase FTS
  ├── Filtros por categoría y ubicación
  ├── Mapa integrado
  └── Sistema de reseñas básico

SEO FOUNDATION (Paralelo)
  ├── Metadata dinámica por página
  ├── JSON-LD para negocios (LocalBusiness schema)
  ├── Sitemap dinámico
  ├── Open Graph para compartir
  └── robots.txt

LANDING PAGE Y DISEÑO (Semanas 6-8)
  ├── Design system base
  ├── Homepage con buscador prominente
  ├── Navegación principal
  └── Footer con links importantes

LANZAMIENTO FASE 1
  └── Beta privada con primeros 50 negocios de Ensenada
```

### Fase 2 — Contenido y Comunidad (Meses 4-7)
**Objetivo:** Expandir la oferta de contenido y atraer usuarios regulares.

```
├── Módulo de Eventos
├── Módulo de Noticias (CMS editorial)
├── Módulo de Empleos (publicación + aplicaciones)
├── Módulo de Restaurantes (extensión del directorio)
├── Dashboard básico para dueños de negocio (autogestión)
├── Migración a Meilisearch para búsqueda avanzada
├── Sistema de notificaciones por email (Resend)
└── Internacionalización ES/EN (next-intl)
```

### Fase 3 — IA y Turismo (Meses 7-10)
**Objetivo:** Diferenciación radical a través de IA y turismo.

```
├── Asistente de IA "Ensenada AI"
│   ├── Sistema de embeddings con pgvector
│   ├── Pipeline RAG completo
│   └── Chat UI integrado en toda la plataforma
├── Módulo de Turismo
├── Itinerarios sugeridos por IA
├── Marketplace (versión básica, sin pagos)
└── App móvil web progresiva (PWA)
```

### Fase 4 — Monetización y Analytics (Meses 10-14)
**Objetivo:** Hacer la plataforma financieramente sostenible.

```
├── Planes de membresía para negocios (Stripe)
├── MercadoPago integration
├── Sistema de publicidad gestionado
├── Marketplace con pagos (Stripe)
├── Dashboard de analytics avanzado para negocios
├── Panel administrativo completo
├── Sistema de moderación automatizado (IA + humanos)
└── Analytics interno de la plataforma
```

### Fase 5 — Escala y Ecosistema (Mes 14+)
**Objetivo:** Convertir la plataforma en el ecosistema definitivo.

```
├── Aplicaciones móviles nativas (React Native / Expo)
├── API Pública documentada
├── SDK para desarrolladores
├── Programa de partners
├── Organizaciones y OSC (tercer sector)
├── Recursos de gobierno local (integración)
└── Evaluación de arquitectura: ¿qué módulos extraer a microservicios?
```

---

## 8. Riesgos Técnicos

### Riesgo 1: Scope Creep (CRÍTICO)
**Descripción:** El proyecto tiene un alcance enorme. El peligro más grande es intentar construir todo a la vez.

**Mitigación:**
- Fase 1 limitada estrictamente al directorio de negocios
- Cada nueva feature debe pasar el filtro: *"¿Esto mejora la plataforma para Ensenada?"*
- Regla: **ningún módulo nuevo inicia hasta que el anterior está estable**

### Riesgo 2: SEO Competitivo
**Descripción:** Google Maps, Yelp, TripAdvisor ya indexan negocios de Ensenada. Competir con ellos en SEO tomará tiempo.

**Mitigación:**
- Enfocarse en palabras clave locales de long-tail ("mariachis ensenada para fiestas", "dentista bilingüe ensenada")
- Contenido único que los grandes no tienen (noticias locales, eventos, contexto cultural)
- La IA puede generar contenido diferenciado a escala

### Riesgo 3: Adopción por Parte de Negocios Locales
**Descripción:** Los negocios pequeños de Ensenada pueden tener baja alfabetización digital.

**Mitigación:**
- Onboarding ultra simplificado (menos de 5 pasos)
- Considerar "setup asistido" (un operador crea el perfil por ellos)
- Versión en español primero, siempre

### Riesgo 4: Costos de Infraestructura
**Descripción:** Supabase + Vercel + OpenAI + Meilisearch pueden escalar en costo rápidamente.

**Mitigación:**
- Supabase Free Tier es suficiente para Fase 1 (hasta 500MB DB, 1GB storage)
- OpenAI: implementar caché de respuestas comunes en Redis (Upstash)
- Meilisearch: empezar con Supabase FTS (gratis) hasta Fase 2
- Vercel Free Tier es suficiente para Fase 1

*Presupuesto estimado de infraestructura:*

| Fase | Costo/mes estimado |
|---|---|
| Fase 1 (MVP) | ~$0-20 |
| Fase 2 | ~$50-150 |
| Fase 3 | ~$150-400 |
| Fase 4 | ~$300-800 |

### Riesgo 5: Calidad del Contenido y Moderación
**Descripción:** Una plataforma de esta escala puede ser target de spam, negocios falsos, empleos fraudulentos.

**Mitigación:**
- Sistema de moderación desde Fase 1 (todo necesita aprobación)
- OpenAI Moderation API para contenido generado por usuarios
- Sistema de reportes desde el inicio
- Los negocios requieren verificación antes de aparecer en el directorio

### Riesgo 6: Dependencia de Terceros
**Descripción:** Dependemos de Supabase, Vercel, OpenAI. Cambios en sus precios o TOS podrían impactar.

**Mitigación:**
- Abstraer todos los clientes detrás de interfaces propias (`lib/supabase`, `lib/openai`)
- Si un proveedor cambia, solo modificamos la capa de abstracción
- PostgreSQL es estándar; migrar de Supabase a otro PostgreSQL es factible

### Riesgo 7: Bilingüismo y Localización
**Descripción:** Ensenada tiene turismo principalmente americano. Contenido solo en español limita el alcance turístico.

**Mitigación:**
- next-intl desde Fase 1 (aunque solo español al inicio)
- Datos de negocios estructurados permiten traducción futura
- La IA puede asistir con traducciones contextuales

---

## 9. Decisiones Arquitectónicas Importantes

### ADR-001: Monolito Modular vs Microservicios
**Decisión:** Monolito Modular
**Razón:** Velocidad de desarrollo, menor costo operacional, equipo pequeño. Los módulos están bien delimitados para extraerse cuando sea necesario.
**Revisión:** En Fase 5, evaluar qué módulos justifican extracción.

### ADR-002: App Router de Next.js (no Pages Router)
**Decisión:** App Router
**Razón:** React Server Components reducen el JS enviado al cliente (mejor performance), mejor soporte para streaming/Suspense, es el futuro del framework.
**Costo:** Curva de aprendizaje moderada. La documentación de Supabase con App Router ya está madura.

### ADR-003: Server Actions para mutaciones
**Decisión:** Usar Server Actions de Next.js para formularios y mutaciones de datos
**Razón:** Elimina la necesidad de crear API Routes para cada operación. El código del servidor y cliente está colocado. Más seguro (los datos no se exponen al cliente).
**Alternativa rechazada:** API Routes para todo (más verbose, más boilerplate).

### ADR-004: Supabase como Backend primario
**Decisión:** Supabase maneja auth, DB, storage y realtime
**Razón:** Un único proveedor para 4 servicios críticos reduce complejidad de integración. La capa RLS es seguridad de DB nativa.
**Riesgo aceptado:** Dependencia de un proveedor. Mitigado por abstracción en `lib/`.

### ADR-005: pgvector para IA (no Pinecone ni Weaviate)
**Decisión:** pgvector extensión en Supabase PostgreSQL
**Razón:** Los embeddings viven junto a los datos. Búsqueda vectorial + SQL en la misma query. Cero costo adicional.
**Alternativa rechazada:** Pinecone (costo adicional, complejidad de sincronización de datos).

### ADR-006: Búsqueda en 2 etapas
**Decisión:** Supabase FTS en Fase 1, Meilisearch en Fase 2+
**Razón:** Evitar costo y complejidad de Meilisearch en etapas tempranas. La búsqueda de Supabase es suficientemente buena para el MVP.
**Señal de migración:** Cuando los usuarios reporten resultados de búsqueda insatisfactorios, o cuando la DB supere 10,000 registros.

### ADR-007: Slugs únicos para SEO
**Decisión:** Cada negocio, evento, artículo tendrá un slug único legible por humanos
**Razón:** URLs amigables para SEO (`/negocios/bodega-de-santo-tomas`) vs IDs (`/negocios/f4a3b2c1`)
**Implementación:** Generación automática + validación de unicidad en DB

### ADR-008: JSON-LD para datos estructurados
**Decisión:** Implementar Schema.org JSON-LD en todas las páginas clave
**Razón:** Google usa datos estructurados para rich snippets, Google Maps, Google Events
**Schemas prioritarios:** LocalBusiness, Restaurant, Event, JobPosting, Article, FAQPage

### ADR-009: Idioma primario
**Decisión:** Español primario, Inglés secundario
**Razón:** La mayoría de residentes locales son hispanohablantes. Turistas aprecian el inglés pero no es crítico para el MVP.
**Implementación:** next-intl con español como fallback, inglés en secciones de turismo.

### ADR-010: Mobile-First en diseño, Mobile App en Fase 5
**Decisión:** El diseño web es mobile-first desde el día 1. App nativa en Fase 5.
**Razón:** La mayoría del tráfico vendrá de móviles. PWA puede cubrir necesidades básicas hasta Fase 5.

---

## 10. Preguntas Críticas Antes de Comenzar

Estas preguntas no son opcionales. Las respuestas cambiarán decisiones de arquitectura.

---

### Sobre el Negocio y Modelo de Ingresos

**P1. ¿Cuál es el modelo de monetización principal desde el inicio?**
- ¿Los negocios pagan desde el primer día?
- ¿Hay un período de lanzamiento gratuito para atraer negocios?
- ¿Los planes de pago son por características o por visibilidad?

*Por qué importa: determina la prioridad del sistema de pagos en el roadmap.*

**P2. ¿Tienes acuerdos o alianzas previas con el gobierno municipal, cámaras de comercio o grupos empresariales de Ensenada?**

*Por qué importa: si hay alianzas institucionales, podemos estructurar el onboarding de negocios de forma diferente (datos pre-cargados, validación institucional).*

**P3. ¿Quién validará/aprobará los negocios en el directorio?**
- ¿Un equipo interno?
- ¿Proceso automatizado con validación posterior?
- ¿Autogestión con reporte de usuarios?

*Por qué importa: define el flujo completo de moderación y cómo lo construimos.*

---

### Sobre el Equipo y Recursos

**P4. ¿Cuál es el equipo de desarrollo disponible?**
- ¿Solo tú y yo (cofundador + arquitecto)?
- ¿Hay presupuesto para contratar desarrolladores adicionales?
- ¿Existe equipo editorial para el módulo de noticias?

*Por qué importa: la capacidad del equipo dicta el ritmo del roadmap.*

**P5. ¿Cuál es el presupuesto mensual para infraestructura en Fase 1?**

*Por qué importa: decide si pagamos por Meilisearch desde el inicio o usamos FTS gratuito.*

---

### Sobre los Usuarios

**P6. ¿El marketplace es C2C (persona a persona), B2C (negocio a consumidor) o ambos?**

*Por qué importa: cambia completamente el diseño del esquema, los flujos de onboarding y los requerimientos legales.*

**P7. ¿El módulo de empleos es solo para negocios registrados en la plataforma, o cualquier empleador puede publicar?**

*Por qué importa: si cualquiera puede publicar, necesitamos mayor moderación y verificación.*

**P8. ¿Los turistas son un usuario primario o secundario en el MVP?**

*Por qué importa: si los turistas son primarios, el módulo de turismo e inglés suben en prioridad desde Fase 1.*

---

### Sobre el Contenido

**P9. ¿El módulo de noticias es un portal de noticias propio o agrega noticias de otros medios locales?**

*Por qué importa: si es propio, necesitamos un equipo editorial. Si agrega, necesitamos RSS/scraping y permisos.*

**P10. ¿Tienes ya una base de datos de negocios de Ensenada (CSV, spreadsheet) para cargar al inicio?**

*Por qué importa: si tenemos datos iniciales, el directorio puede lanzar con contenido real. Si no, el directorio estaría vacío al inicio.*

---

### Sobre Decisiones Técnicas Específicas

**P11. ¿Tienes dominio y hosting de Ensenada.org ya adquirido? ¿Qué pasa con ConectandoEnsenada.mx?**

*Por qué importa: el dominio .mx puede ser importante para SEO local en México.*

**P12. ¿Cuál es tu nivel de comodidad técnica con el stack propuesto?**
- ¿Has trabajado antes con Next.js App Router?
- ¿Tienes experiencia con Supabase?
- ¿TypeScript es familiar?

*Por qué importa: si hay curvas de aprendizaje, ajustamos el ritmo de las Fases 1 y 2.*

**P13. ¿Hay alguna función que debería estar disponible en el lanzamiento público que yo no haya mencionado?**

*Por qué importa: quiero asegurarme de no haber omitido nada que consideres esencial.*

---

## Resumen Ejecutivo

| Aspecto | Decisión |
|---|---|
| **Patrón** | Monolito Modular (preparado para microservicios) |
| **Monorepo** | Turborepo (web + packages + mobile futuro) |
| **Framework** | Next.js 15 App Router + React 19 |
| **Base de datos** | Supabase PostgreSQL con RLS por dominio |
| **Búsqueda** | FTS de Supabase → Meilisearch en Fase 2 |
| **IA** | OpenAI GPT-4o + pgvector + RAG |
| **Pagos** | Stripe + MercadoPago (Fase 2) |
| **Rendering** | ISR para SEO, CSR para dashboards |
| **MVP** | Directorio de negocios + Auth + Búsqueda |
| **Timeline MVP** | 3-4 meses |
| **Costo inicial** | ~$0-20/mes |

---

*Este documento es un punto de partida. Cada decisión aquí es revisable antes de comenzar a construir. Tu retroalimentación y las respuestas a las preguntas de la Sección 10 son el siguiente paso.*

---

**Próximos pasos sugeridos:**
1. Revisas este documento y me das feedback
2. Respondemos las 13 preguntas críticas
3. Definimos el alcance exacto de la Fase 1
4. Diseñamos el schema de base de datos detallado
5. Primera línea de código
