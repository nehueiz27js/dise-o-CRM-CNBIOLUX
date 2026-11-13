# CRM de Clientes · CN BIOLUX — Guía paso a paso

Guía completa para programar el panel **CRM de Clientes** (el de la imagen de referencia) dentro de la web de CN BIOLUX.

**Stack:** Next.js (App Router) · React · TypeScript · Tailwind CSS · Prisma · Cloudinary · Vercel

> El CRM es una **sección nueva dentro del panel admin existente** (`/admin/crm`). El Sidebar, el header y la autenticación ya los tenés: acá solo construís el contenido de la pantalla.

---

## Índice

1. [El diseño descripto en texto](#1-el-diseño-descripto-en-texto)
2. [Orden de trabajo (fases)](#2-orden-de-trabajo-fases)
3. [Estructura de carpetas](#3-estructura-de-carpetas)
4. [Fase 1 · Tokens de diseño y componentes base](#4-fase-1--tokens-de-diseño-y-componentes-base)
5. [Fase 2 · Modelo de datos (Prisma)](#5-fase-2--modelo-de-datos-prisma)
6. [Fases 3 y 4 · Pantalla principal conectada a la base](#6-fases-3-y-4--pantalla-principal-conectada-a-la-base)
7. [Fase 5 · Búsqueda, filtros y paginación](#7-fase-5--búsqueda-filtros-y-paginación)
8. [Fase 6 · Server Actions (crear, editar, estado, notas)](#8-fase-6--server-actions)
9. [Fase 7 · Tabs de la ficha](#9-fase-7--tabs-de-la-ficha)
10. [Fase 8 · Importar y exportar Excel](#10-fase-8--importar-y-exportar-excel)
11. [Fase 9 · Permisos, validaciones y responsive](#11-fase-9--permisos-validaciones-y-responsive)
12. [Fase 10 · Deploy en Vercel](#12-fase-10--deploy-en-vercel)
13. [Checklist final](#13-checklist-final)

---

## 1. El diseño descripto en texto

### Layout general

Grid de 2 columnas: **Sidebar fija (~245 px)** + **contenido (resto)**.

**Sidebar** (ya existe en tu admin, solo agregá el ítem):
- Logo CN BIOLUX
- Dashboard, Productos, Categorías, Comunicaciones (badge rojo con contador), Proyectos, **CRM de Clientes** (activo, resaltado en verde), Configuración, Secciones de la Web, Usuarios y Permisos
- Abajo: tarjeta "Conectado como" (avatar, nombre, rol) y "Cerrar Sesión"

**Contenido**, de arriba hacia abajo:

1. **Header**
   - Botón hamburguesa
   - Título "CRM de Clientes" con barra verde vertical a la izquierda y subtítulo "Gestioná tus clientes, consultas e instalaciones en un solo lugar"
   - A la derecha: indicador "● Sistema Online", campana con contador y avatar

2. **4 tarjetas KPI** (cada una: ícono de color, número grande, etiqueta y variación con flecha verde)
   - 124 Clientes Registrados (+12%)
   - 48 Consultas Totales (+8%)
   - 37 Instalaciones (+15%)
   - 92% Clientes Activos

3. **Zona principal en 2 columnas** (`350px | 1fr`)
   - **Izquierda — lista de clientes**
     - Buscador + botón "Filtros"
     - Botones "＋ Nuevo Cliente" (verde) e "Importar Excel" (outline)
     - Lista: avatar con iniciales, "Apellido Nombre", teléfono, badge de estado y flecha `›`. El seleccionado tiene borde verde.
     - Paginación (1, 2, 3, ›)
   - **Derecha — ficha del cliente**
     - Cabecera: avatar grande, nombre, badge de estado, teléfono, email, localidad, botones "Editar Cliente" y "Más ▾"
     - Tabs: **Resumen**, Consultas (3), Instalaciones (1), Historial, Documentos
     - Grilla de 4 tarjetas:
       - **Información del Cliente**: nombre, teléfono/WhatsApp (con ícono y botón copiar), correo, localidad, dirección, tipo de cliente (badge)
       - **Estado y Seguimiento**: select de estado actual, select de responsable, fecha de última actividad, próxima acción (botón naranja)
       - **Actividad Reciente**: línea de tiempo con puntos de color + "Ver todo →"
       - **Notas**: texto de la nota + textarea "Agregar nota…" + botón "Guardar Nota"

4. **Barra inferior de acciones**
   - Registrar Consulta (verde) · Agregar Instalación (azul) · Importar Excel (violeta) · Exportar Reporte (outline)
   - A la derecha: "● Última actualización: fecha – hora" y botón refrescar

### Paleta (tema oscuro, valores aproximados)

| Uso | Color |
|---|---|
| Fondo | `#0B1217` |
| Paneles / tarjetas | `#111B22` |
| Bordes | `#1E2A33` |
| Verde marca | `#22C55E` |
| Azul | `#3B82F6` |
| Violeta | `#7C3AED` |
| Naranja | `#F59E0B` |
| Texto | `#E6EDF3` |
| Texto apagado | `#8B98A5` |

### Colores de estado

| Estado | Color |
|---|---|
| Activo | Verde |
| En gestión | Naranja |
| Presupuesto | Azul |
| Instalación | Violeta |
| Seguimiento | Turquesa |

---

## 2. Orden de trabajo (fases)

| Fase | Qué hacés | Resultado |
|---|---|---|
| 1 | Tokens de diseño + componentes base | Botones, badges y tarjetas reutilizables |
| 2 | Modelo de datos en Prisma + seed | Base con clientes de prueba |
| 3 | Layout del CRM con datos falsos | Pantalla idéntica a la imagen, estática |
| 4 | Conectar lista y ficha a la base | Cliente seleccionable por URL |
| 5 | Buscar, filtrar, paginar | Lista funcional |
| 6 | Server Actions | CRUD con historial automático |
| 7 | Tabs: Consultas, Instalaciones, Historial, Documentos | Ficha completa |
| 8 | Importar / exportar Excel | Carga masiva y reportes |
| 9 | Permisos, validaciones, responsive | Listo para producción |
| 10 | Deploy | Funcionando en Vercel con base en la nube |

**Regla de oro:** primero que se vea, después que funcione. No pases a la fase siguiente sin probar la actual.

---

## 3. Estructura de carpetas

```
src/
├─ app/admin/crm/
│  ├─ page.tsx                  ← pantalla principal (Server Component)
│  ├─ actions.ts                ← Server Actions
│  ├─ export/route.ts           ← descarga del reporte Excel
│  └─ _components/
│     ├─ Header.tsx
│     ├─ StatCard.tsx
│     ├─ ClientList.tsx
│     ├─ ClientListItem.tsx
│     ├─ ClientHeader.tsx
│     ├─ ClientTabs.tsx
│     ├─ InfoCard.tsx
│     ├─ StatusCard.tsx
│     ├─ ActivityCard.tsx
│     ├─ NotesCard.tsx
│     ├─ StatusBadge.tsx
│     ├─ Pagination.tsx
│     └─ BottomActions.tsx
├─ components/ui/               ← Button, Card, Avatar, Input (compartidos)
└─ lib/
   ├─ prisma.ts
   ├─ validations.ts            ← esquemas Zod
   ├─ crm.ts                    ← consultas (KPIs, listado)
   └─ excel.ts                  ← importar / exportar
```

Dependencias nuevas:

```bash
npm install zod exceljs lucide-react
```

---

## 4. Fase 1 · Tokens de diseño y componentes base

### Colores (Tailwind v4, en `globals.css`)

```css
@theme {
  --color-bg: #0B1217;
  --color-panel: #111B22;
  --color-line: #1E2A33;
  --color-brand: #22C55E;
  --color-info: #3B82F6;
  --color-violet: #7C3AED;
  --color-warn: #F59E0B;
  --color-text: #E6EDF3;
  --color-muted: #8B98A5;
}
```

> Si tu proyecto usa Tailwind v3, estos mismos valores van en `theme.extend.colors` de `tailwind.config.ts`.

### Card base

```tsx
// src/components/ui/Card.tsx
export function Card({ title, icon, action, children, className = "" }: {
  title?: string; icon?: React.ReactNode; action?: React.ReactNode;
  children: React.ReactNode; className?: string;
}) {
  return (
    <section className={`rounded-xl border border-line bg-panel p-4 ${className}`}>
      {title && (
        <header className="mb-3 flex items-center justify-between">
          <h3 className="flex items-center gap-2 font-semibold">{icon}{title}</h3>
          {action}
        </header>
      )}
      {children}
    </section>
  );
}
```

### Badge de estado

```tsx
// src/app/admin/crm/_components/StatusBadge.tsx
const styles = {
  ACTIVO:      "bg-brand/20 text-brand",
  EN_GESTION:  "bg-warn/20 text-warn",
  PRESUPUESTO: "bg-info/20 text-info",
  INSTALACION: "bg-violet/20 text-violet",
  SEGUIMIENTO: "bg-teal-500/20 text-teal-400",
} as const;

export const statusLabels = {
  ACTIVO: "Activo", EN_GESTION: "En gestión", PRESUPUESTO: "Presupuesto",
  INSTALACION: "Instalación", SEGUIMIENTO: "Seguimiento",
} as const;

export type Status = keyof typeof styles;

export function StatusBadge({ status }: { status: Status }) {
  return (
    <span className={`rounded-full px-3 py-1 text-xs font-semibold ${styles[status]}`}>
      {statusLabels[status]}
    </span>
  );
}
```

### Avatar con iniciales

```tsx
// src/components/ui/Avatar.tsx
export function Avatar({ first, last, size = 40 }: { first: string; last: string; size?: number }) {
  return (
    <div
      style={{ width: size, height: size }}
      className="grid shrink-0 place-items-center rounded-full bg-info font-bold text-white"
    >
      {(first[0] ?? "") + (last[0] ?? "")}
    </div>
  );
}
```

### Tarjeta KPI

```tsx
// src/app/admin/crm/_components/StatCard.tsx
import { ArrowUp } from "lucide-react";

export function StatCard({ icon, value, label, trend, color }: {
  icon: React.ReactNode; value: string | number; label: string;
  trend?: number; color: string;
}) {
  return (
    <div className="flex items-center gap-4 rounded-xl border border-line bg-panel p-5">
      <div className={`text-4xl ${color}`}>{icon}</div>
      <div className="flex-1">
        <p className="text-3xl font-bold">{value}</p>
        <p className="text-sm text-muted">{label}</p>
      </div>
      {trend !== undefined && (
        <span className="flex items-center text-sm text-brand">
          <ArrowUp size={14} />{trend}%
        </span>
      )}
    </div>
  );
}
```

---

## 5. Fase 2 · Modelo de datos (Prisma)

```prisma
enum ClientStatus { ACTIVO EN_GESTION PRESUPUESTO INSTALACION SEGUIMIENTO }
enum ClientType   { RESIDENCIAL COMERCIAL INDUSTRIAL RURAL }

model Client {
  id             String       @id @default(cuid())
  firstName      String
  lastName       String
  phone          String
  email          String?
  city           String?
  province       String?
  address        String?
  type           ClientType   @default(RESIDENCIAL)
  status         ClientStatus @default(ACTIVO)
  responsibleId  String?
  responsible    User?        @relation(fields: [responsibleId], references: [id])
  nextAction     String?
  lastActivityAt DateTime     @default(now())
  createdAt      DateTime     @default(now())

  inquiries      Inquiry[]
  installations  Installation[]
  activities     Activity[]
  notes          Note[]
  documents      Document[]

  @@index([lastName])
  @@index([status])
}

// Ya tenés "Consultas": solo agregale la relación opcional con Client
model Inquiry {
  id        String   @id @default(cuid())
  clientId  String?
  client    Client?  @relation(fields: [clientId], references: [id])
  channel   String   // WEB | WHATSAPP | TELEFONO
  message   String
  createdAt DateTime @default(now())
}

model Installation {
  id          String    @id @default(cuid())
  clientId    String
  client      Client    @relation(fields: [clientId], references: [id], onDelete: Cascade)
  kind        String    // on grid | off grid
  powerKw     Float?
  scheduledAt DateTime?
  status      String    @default("PENDIENTE")
}

// Alimenta "Actividad Reciente" y la tab "Historial"
model Activity {
  id        String   @id @default(cuid())
  clientId  String
  client    Client   @relation(fields: [clientId], references: [id], onDelete: Cascade)
  kind      String   // CONSULTA | ESTADO | PRESUPUESTO | NOTA | INSTALACION
  text      String
  createdAt DateTime @default(now())
  @@index([clientId, createdAt])
}

model Note {
  id        String   @id @default(cuid())
  clientId  String
  client    Client   @relation(fields: [clientId], references: [id], onDelete: Cascade)
  body      String
  createdAt DateTime @default(now())
}

model Document {
  id        String   @id @default(cuid())
  clientId  String
  client    Client   @relation(fields: [clientId], references: [id], onDelete: Cascade)
  name      String
  url       String   // Cloudinary
  createdAt DateTime @default(now())
}
```

Agregá en tu modelo `User` existente el lado inverso: `clients Client[]`.

```bash
npx prisma migrate dev -n crm
```

### Seed con los clientes de la imagen

```ts
// prisma/seed.ts
import { prisma } from "../src/lib/prisma";

const data = [
  { firstName: "Gabriel", lastName: "Martínez",  phone: "223 555 1234", status: "ACTIVO" },
  { firstName: "María",   lastName: "González",  phone: "223 567 8901", status: "EN_GESTION" },
  { firstName: "Diego",   lastName: "Rodríguez", phone: "223 111 2222", status: "PRESUPUESTO" },
  { firstName: "Laura",   lastName: "Fernández", phone: "223 333 4444", status: "INSTALACION" },
  { firstName: "Carlos",  lastName: "López",     phone: "223 666 7777", status: "SEGUIMIENTO" },
  { firstName: "Ana",     lastName: "Díaz",      phone: "223 888 9999", status: "ACTIVO" },
] as const;

async function main() {
  for (const c of data) await prisma.client.create({ data: { ...c, city: "Mar del Plata", province: "Bs.As." } });
}
main().finally(() => prisma.$disconnect());
```

En `package.json`: `"prisma": { "seed": "tsx prisma/seed.ts" }` y corré `npx prisma db seed`.

> **Ojo con el deploy:** SQLite sirve en local, pero en Vercel el sistema de archivos no persiste. Antes de publicar el CRM con datos reales, pasá a **PostgreSQL** (Neon, Supabase o Vercel Postgres). Con Prisma alcanza con cambiar `provider` y `DATABASE_URL`.

---

## 6. Fases 3 y 4 · Pantalla principal conectada a la base

### Consultas de apoyo

```ts
// src/lib/crm.ts
import { prisma } from "@/lib/prisma";

const DAY = 86_400_000;
const pct = (now: number, prev: number) => (prev === 0 ? 0 : Math.round(((now - prev) / prev) * 100));

export async function getKpis() {
  const d30 = new Date(Date.now() - 30 * DAY);
  const d60 = new Date(Date.now() - 60 * DAY);

  const [clients, clientsNow, clientsPrev,
         inquiries, inqNow, inqPrev,
         installs, insNow, insPrev, active] = await Promise.all([
    prisma.client.count(),
    prisma.client.count({ where: { createdAt: { gte: d30 } } }),
    prisma.client.count({ where: { createdAt: { gte: d60, lt: d30 } } }),
    prisma.inquiry.count(),
    prisma.inquiry.count({ where: { createdAt: { gte: d30 } } }),
    prisma.inquiry.count({ where: { createdAt: { gte: d60, lt: d30 } } }),
    prisma.installation.count(),
    prisma.installation.count({ where: { scheduledAt: { gte: d30 } } }),
    prisma.installation.count({ where: { scheduledAt: { gte: d60, lt: d30 } } }),
    prisma.client.count({ where: { status: "ACTIVO" } }),
  ]);

  return {
    clients, clientsTrend: pct(clientsNow, clientsPrev),
    inquiries, inquiriesTrend: pct(inqNow, inqPrev),
    installs, installsTrend: pct(insNow, insPrev),
    activePct: clients ? Math.round((active / clients) * 100) : 0,
  };
}
```

### La página

En Next.js reciente `searchParams` es una **Promise**: hay que hacerle `await`.

La URL guarda todo el estado (`?client=ID&tab=resumen&q=gonz&status=ACTIVO&page=2`). Así los enlaces se pueden compartir y no necesitás estado global.

```tsx
// src/app/admin/crm/page.tsx
import { prisma } from "@/lib/prisma";
import { getKpis } from "@/lib/crm";
import { Users, MessageCircle, Wrench, CheckCircle2 } from "lucide-react";
import { StatCard } from "./_components/StatCard";
import { ClientList } from "./_components/ClientList";
import { ClientHeader } from "./_components/ClientHeader";
import { ClientTabs } from "./_components/ClientTabs";
import { InfoCard } from "./_components/InfoCard";
import { StatusCard } from "./_components/StatusCard";
import { ActivityCard } from "./_components/ActivityCard";
import { NotesCard } from "./_components/NotesCard";
import { BottomActions } from "./_components/BottomActions";

const PAGE_SIZE = 5;

type SP = { client?: string; tab?: string; q?: string; status?: string; page?: string };

export default async function CrmPage({ searchParams }: { searchParams: Promise<SP> }) {
  const sp = await searchParams;
  const page = Math.max(1, Number(sp.page ?? 1));

  const where = {
    ...(sp.status && { status: sp.status as any }),
    ...(sp.q && {
      OR: [
        { firstName: { contains: sp.q } },
        { lastName:  { contains: sp.q } },
        { phone:     { contains: sp.q } },
      ],
    }),
  };

  const [clients, total, kpis, users] = await Promise.all([
    prisma.client.findMany({
      where, orderBy: { lastActivityAt: "desc" },
      skip: (page - 1) * PAGE_SIZE, take: PAGE_SIZE,
    }),
    prisma.client.count({ where }),
    getKpis(),
    prisma.user.findMany({ select: { id: true, name: true } }),
  ]);

  const selectedId = sp.client ?? clients[0]?.id;
  const selected = selectedId
    ? await prisma.client.findUnique({
        where: { id: selectedId },
        include: {
          responsible: true,
          notes: { orderBy: { createdAt: "desc" }, take: 1 },
          activities: { orderBy: { createdAt: "desc" }, take: 4 },
          _count: { select: { inquiries: true, installations: true } },
        },
      })
    : null;

  const tab = sp.tab ?? "resumen";

  return (
    <div className="flex flex-col gap-4 p-6">
      {/* KPIs */}
      <section className="grid grid-cols-1 gap-4 sm:grid-cols-2 xl:grid-cols-4">
        <StatCard icon={<Users />}         value={kpis.clients}    label="Clientes Registrados" trend={kpis.clientsTrend}   color="text-info" />
        <StatCard icon={<MessageCircle />} value={kpis.inquiries}  label="Consultas Totales"    trend={kpis.inquiriesTrend} color="text-brand" />
        <StatCard icon={<Wrench />}        value={kpis.installs}   label="Instalaciones"        trend={kpis.installsTrend}  color="text-warn" />
        <StatCard icon={<CheckCircle2 />}  value={`${kpis.activePct}%`} label="Clientes Activos" color="text-brand" />
      </section>

      {/* Lista + ficha */}
      <section className="grid gap-4 lg:grid-cols-[350px_1fr]">
        <ClientList clients={clients} total={total} page={page} pageSize={PAGE_SIZE}
                    selectedId={selectedId} sp={sp} />

        {selected ? (
          <div className="flex flex-col gap-4 rounded-xl border border-line bg-panel p-4">
            <ClientHeader client={selected} />
            <ClientTabs
              active={tab} clientId={selected.id}
              counts={{ inquiries: selected._count.inquiries, installations: selected._count.installations }}
            />
            {tab === "resumen" && (
              <div className="grid gap-4 xl:grid-cols-2">
                <InfoCard client={selected} />
                <StatusCard client={selected} users={users} />
                <ActivityCard activities={selected.activities} clientId={selected.id} />
                <NotesCard clientId={selected.id} lastNote={selected.notes[0]?.body} />
              </div>
            )}
            {/* las otras tabs se agregan en la Fase 7 */}
          </div>
        ) : (
          <p className="grid place-items-center text-muted">No hay clientes para mostrar.</p>
        )}
      </section>

      <BottomActions clientId={selected?.id} />
    </div>
  );
}
```

> Como el Sidebar y el header ya existen en tu `admin/layout.tsx`, esta página solo devuelve el contenido.

### Lista de clientes

```tsx
// src/app/admin/crm/_components/ClientListItem.tsx
import Link from "next/link";
import { ChevronRight, Phone } from "lucide-react";
import { Avatar } from "@/components/ui/Avatar";
import { StatusBadge } from "./StatusBadge";

export function ClientListItem({ c, selected, href }: { c: any; selected: boolean; href: string }) {
  return (
    <Link
      href={href}
      className={`flex items-center gap-3 rounded-lg border p-3 transition hover:bg-white/5
        ${selected ? "border-brand bg-brand/5" : "border-line"}`}
    >
      <Avatar first={c.firstName} last={c.lastName} />
      <div className="min-w-0 flex-1">
        <p className="truncate font-semibold">{c.lastName} {c.firstName}</p>
        <p className="flex items-center gap-1 text-xs text-muted"><Phone size={12} />{c.phone}</p>
      </div>
      <StatusBadge status={c.status} />
      <ChevronRight size={16} className="text-muted" />
    </Link>
  );
}
```

```tsx
// src/app/admin/crm/_components/ClientList.tsx
import { ClientListItem } from "./ClientListItem";
import { Pagination } from "./Pagination";
import { SearchBox } from "./SearchBox"; // ver Fase 5

export function ClientList({ clients, total, page, pageSize, selectedId, sp }: any) {
  const buildHref = (id: string) => {
    const p = new URLSearchParams({ ...sp, client: id });
    p.delete("tab");
    return `/admin/crm?${p}`;
  };

  return (
    <aside className="flex flex-col gap-3 rounded-xl border border-line bg-panel p-3">
      <SearchBox defaultValue={sp.q} status={sp.status} />
      <div className="flex gap-2">
        {/* "Nuevo Cliente" abre un modal con el formulario (Fase 6) */}
        <button className="flex-1 rounded-lg bg-brand px-3 py-2 font-semibold text-black">＋ Nuevo Cliente</button>
        <a href="#importar" className="rounded-lg border border-line px-3 py-2">Importar Excel</a>
      </div>
      <div className="flex flex-col gap-2">
        {clients.map((c: any) => (
          <ClientListItem key={c.id} c={c} selected={c.id === selectedId} href={buildHref(c.id)} />
        ))}
      </div>
      <Pagination page={page} total={total} pageSize={pageSize} sp={sp} />
    </aside>
  );
}
```

---

## 7. Fase 5 · Búsqueda, filtros y paginación

### Buscador (Client Component con debounce)

Actualiza la URL, y como la página es un Server Component, se vuelve a renderizar sola con los resultados.

```tsx
// src/app/admin/crm/_components/SearchBox.tsx
"use client";
import { useRouter, useSearchParams } from "next/navigation";
import { useEffect, useState } from "react";
import { Search } from "lucide-react";

const STATUSES = ["", "ACTIVO", "EN_GESTION", "PRESUPUESTO", "INSTALACION", "SEGUIMIENTO"];

export function SearchBox({ defaultValue = "", status = "" }: { defaultValue?: string; status?: string }) {
  const router = useRouter();
  const params = useSearchParams();
  const [q, setQ] = useState(defaultValue);

  useEffect(() => {
    const t = setTimeout(() => {
      const p = new URLSearchParams(params.toString());
      q ? p.set("q", q) : p.delete("q");
      p.delete("page");
      router.replace(`/admin/crm?${p}`);
    }, 350);
    return () => clearTimeout(t);
  }, [q]); // eslint-disable-line react-hooks/exhaustive-deps

  function setStatus(value: string) {
    const p = new URLSearchParams(params.toString());
    value ? p.set("status", value) : p.delete("status");
    p.delete("page");
    router.replace(`/admin/crm?${p}`);
  }

  return (
    <div className="flex gap-2">
      <label className="flex flex-1 items-center gap-2 rounded-lg border border-line px-3">
        <Search size={16} className="text-muted" />
        <input value={q} onChange={(e) => setQ(e.target.value)} placeholder="Buscar cliente…"
               className="w-full bg-transparent py-2 outline-none" />
      </label>
      <select value={status} onChange={(e) => setStatus(e.target.value)}
              className="rounded-lg border border-line bg-panel px-2 text-sm">
        {STATUSES.map((s) => <option key={s} value={s}>{s || "Todos"}</option>)}
      </select>
    </div>
  );
}
```

> En el diseño original el filtro es un botón "Filtros". Para arrancar simple usá un `select`; después lo convertís en un popover.

### Paginación

```tsx
// src/app/admin/crm/_components/Pagination.tsx
import Link from "next/link";

export function Pagination({ page, total, pageSize, sp }: any) {
  const pages = Math.max(1, Math.ceil(total / pageSize));
  const href = (n: number) => `/admin/crm?${new URLSearchParams({ ...sp, page: String(n) })}`;

  return (
    <nav className="flex justify-center gap-2 pt-2">
      {Array.from({ length: pages }, (_, i) => i + 1).map((n) => (
        <Link key={n} href={href(n)}
              className={`grid h-9 w-9 place-items-center rounded-md text-sm
                ${n === page ? "bg-brand font-bold text-black" : "hover:bg-white/5"}`}>
          {n}
        </Link>
      ))}
      {page < pages && <Link href={href(page + 1)} className="grid h-9 w-9 place-items-center">›</Link>}
    </nav>
  );
}
```

---

## 8. Fase 6 · Server Actions

### Validaciones (Zod)

```ts
// src/lib/validations.ts
import { z } from "zod";

export const clientSchema = z.object({
  firstName: z.string().trim().min(1, "Nombre requerido"),
  lastName:  z.string().trim().min(1, "Apellido requerido"),
  phone:     z.string().trim().min(6, "Teléfono inválido"),
  email:     z.string().email("Email inválido").optional().or(z.literal("")),
  city:      z.string().optional(),
  address:   z.string().optional(),
  type:      z.enum(["RESIDENCIAL", "COMERCIAL", "INDUSTRIAL", "RURAL"]).default("RESIDENCIAL"),
});

export const statusSchema = z.object({
  clientId: z.string(),
  status: z.enum(["ACTIVO", "EN_GESTION", "PRESUPUESTO", "INSTALACION", "SEGUIMIENTO"]),
});

export const noteSchema = z.object({
  clientId: z.string(),
  body: z.string().trim().min(1).max(2000),
});

export const inquirySchema = z.object({
  clientId: z.string(),
  channel: z.enum(["WEB", "WHATSAPP", "TELEFONO"]),
  message: z.string().trim().min(1).max(2000),
});
```

### Acciones

Cada acción **valida → verifica permisos → escribe → registra actividad → revalida**.

```ts
// src/app/admin/crm/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { prisma } from "@/lib/prisma";
import { requireAdmin } from "@/lib/auth"; // tu helper de autenticación existente
import { clientSchema, statusSchema, noteSchema, inquirySchema } from "@/lib/validations";
import { statusLabels } from "./_components/StatusBadge";

const touch = (clientId: string) =>
  prisma.client.update({ where: { id: clientId }, data: { lastActivityAt: new Date() } });

export async function createClient(formData: FormData) {
  await requireAdmin();
  const data = clientSchema.parse(Object.fromEntries(formData));
  const client = await prisma.client.create({
    data: { ...data, email: data.email || null },
  });
  await prisma.activity.create({
    data: { clientId: client.id, kind: "ESTADO", text: "Cliente registrado" },
  });
  revalidatePath("/admin/crm");
  return client.id;
}

export async function updateStatus(formData: FormData) {
  await requireAdmin();
  const { clientId, status } = statusSchema.parse(Object.fromEntries(formData));
  await prisma.$transaction([
    prisma.client.update({ where: { id: clientId }, data: { status, lastActivityAt: new Date() } }),
    prisma.activity.create({
      data: { clientId, kind: "ESTADO", text: `Se actualizó el estado a: ${statusLabels[status]}` },
    }),
  ]);
  revalidatePath("/admin/crm");
}

export async function addNote(formData: FormData) {
  await requireAdmin();
  const { clientId, body } = noteSchema.parse(Object.fromEntries(formData));
  await prisma.$transaction([
    prisma.note.create({ data: { clientId, body } }),
    prisma.activity.create({ data: { clientId, kind: "NOTA", text: "Se agregó una nota" } }),
  ]);
  await touch(clientId);
  revalidatePath("/admin/crm");
}

export async function registerInquiry(formData: FormData) {
  await requireAdmin();
  const d = inquirySchema.parse(Object.fromEntries(formData));
  await prisma.$transaction([
    prisma.inquiry.create({ data: d }),
    prisma.activity.create({
      data: { clientId: d.clientId, kind: "CONSULTA", text: `Se registró nueva consulta por ${d.channel}` },
    }),
  ]);
  await touch(d.clientId);
  revalidatePath("/admin/crm");
}
```

### Usar las acciones desde un formulario (sin JavaScript extra)

```tsx
// src/app/admin/crm/_components/StatusCard.tsx
import { Card } from "@/components/ui/Card";
import { updateStatus } from "../actions";
import { statusLabels } from "./StatusBadge";

export function StatusCard({ client, users }: any) {
  return (
    <Card title="Estado y Seguimiento">
      <form action={updateStatus} className="flex flex-col gap-3">
        <input type="hidden" name="clientId" value={client.id} />
        <label className="flex items-center justify-between gap-3 text-sm">
          <span className="text-muted">Estado actual:</span>
          <select name="status" defaultValue={client.status}
                  className="rounded-md border border-line bg-panel px-3 py-2">
            {Object.entries(statusLabels).map(([v, l]) => <option key={v} value={v}>{l}</option>)}
          </select>
        </label>
        <button className="self-end rounded-md bg-brand px-3 py-1.5 text-sm font-semibold text-black">
          Guardar estado
        </button>
      </form>

      <dl className="mt-3 space-y-2 text-sm">
        <div className="flex justify-between"><dt className="text-muted">Responsable:</dt>
          <dd>{client.responsible?.name ?? "Sin asignar"}</dd></div>
        <div className="flex justify-between"><dt className="text-muted">Fecha última actividad:</dt>
          <dd>{new Date(client.lastActivityAt).toLocaleString("es-AR")}</dd></div>
        <div className="flex justify-between"><dt className="text-muted">Próxima acción:</dt>
          <dd className="rounded-md bg-warn/20 px-2 py-1 text-warn">{client.nextAction ?? "—"}</dd></div>
      </dl>
    </Card>
  );
}
```

```tsx
// src/app/admin/crm/_components/NotesCard.tsx
import { Card } from "@/components/ui/Card";
import { addNote } from "../actions";

export function NotesCard({ clientId, lastNote }: { clientId: string; lastNote?: string }) {
  return (
    <Card title="Notas">
      <p className="mb-3 text-sm">{lastNote ?? "Sin notas todavía."}</p>
      <form action={addNote} className="flex gap-2">
        <input type="hidden" name="clientId" value={clientId} />
        <textarea name="body" required placeholder="Agregar nota…"
                  className="min-h-16 flex-1 rounded-md border border-line bg-transparent p-2 text-sm" />
        <button className="self-end rounded-md bg-brand px-3 py-2 text-sm font-semibold text-black">
          Guardar Nota
        </button>
      </form>
    </Card>
  );
}
```

### Actividad reciente (línea de tiempo)

```tsx
// src/app/admin/crm/_components/ActivityCard.tsx
import Link from "next/link";
import { Card } from "@/components/ui/Card";

const dot: Record<string, string> = {
  CONSULTA: "bg-cyan-400", ESTADO: "bg-violet", PRESUPUESTO: "bg-slate-400",
  NOTA: "bg-brand", INSTALACION: "bg-warn",
};

export function ActivityCard({ activities, clientId }: any) {
  return (
    <Card title="Actividad Reciente"
          action={<Link href={`?client=${clientId}&tab=historial`} className="text-sm text-brand">Ver todo →</Link>}>
      <ul className="space-y-2 text-sm">
        {activities.map((a: any) => (
          <li key={a.id} className="flex items-start gap-3">
            <span className={`mt-1.5 h-2.5 w-2.5 rounded-full ${dot[a.kind] ?? "bg-muted"}`} />
            <time className="w-32 shrink-0 text-muted">{new Date(a.createdAt).toLocaleString("es-AR")}</time>
            <span>{a.text}</span>
          </li>
        ))}
      </ul>
    </Card>
  );
}
```

### Botón "Nuevo Cliente"

Un modal con un formulario cuyo `action` sea `createClient`. Podés hacerlo con la etiqueta nativa `<dialog>` y un pequeño Client Component que llame `showModal()`. Después de crear, hacé `redirect` a `/admin/crm?client=<id>`.

---

## 9. Fase 7 · Tabs de la ficha

Las tabs son **links** que cambian `?tab=`. Se renderizan en el servidor y no hace falta estado.

```tsx
// src/app/admin/crm/_components/ClientTabs.tsx
import Link from "next/link";

export function ClientTabs({ active, clientId, counts }: any) {
  const tabs = [
    { id: "resumen",       label: "Resumen" },
    { id: "consultas",     label: `Consultas (${counts.inquiries})` },
    { id: "instalaciones", label: `Instalaciones (${counts.installations})` },
    { id: "historial",     label: "Historial" },
    { id: "documentos",    label: "Documentos" },
  ];
  return (
    <nav className="flex gap-1 border-b border-line">
      {tabs.map((t) => (
        <Link key={t.id} href={`/admin/crm?client=${clientId}&tab=${t.id}`}
              className={`px-4 py-2 text-sm ${active === t.id
                ? "border-b-2 border-brand font-semibold text-brand" : "text-muted hover:text-text"}`}>
          {t.label}
        </Link>
      ))}
    </nav>
  );
}
```

En `page.tsx`, debajo de la tab "resumen", agregás el resto:

```tsx
{tab === "consultas"     && <InquiriesTab clientId={selected.id} />}
{tab === "instalaciones" && <InstallationsTab clientId={selected.id} />}
{tab === "historial"     && <HistoryTab clientId={selected.id} />}
{tab === "documentos"    && <DocumentsTab clientId={selected.id} />}
```

Cada una es un Server Component que hace su propio `prisma.<modelo>.findMany({ where: { clientId } })`:

| Tab | Qué muestra | Acción asociada |
|---|---|---|
| Consultas | Lista (canal, mensaje, fecha) | `registerInquiry` |
| Instalaciones | Tipo, potencia kW, fecha programada, estado | `addInstallation` (similar a `registerInquiry`) |
| Historial | Todas las `Activity` del cliente, paginadas | — |
| Documentos | Lista con link de descarga | Subida a Cloudinary + `addDocument` |

**Documentos:** subí el archivo a Cloudinary (usa la misma integración que ya tenés para imágenes de productos), guardá la `url` en `Document` y listalo.

---

## 10. Fase 8 · Importar y exportar Excel

Se usa **ExcelJS** (mantenido y sin las vulnerabilidades conocidas del paquete `xlsx` de npm).

### Exportar reporte (Route Handler)

```ts
// src/app/admin/crm/export/route.ts
import ExcelJS from "exceljs";
import { prisma } from "@/lib/prisma";
import { requireAdmin } from "@/lib/auth";

export async function GET() {
  await requireAdmin();
  const clients = await prisma.client.findMany({
    orderBy: { lastName: "asc" }, include: { responsible: true },
  });

  const wb = new ExcelJS.Workbook();
  const ws = wb.addWorksheet("Clientes");
  ws.columns = [
    { header: "Apellido",    key: "lastName",  width: 20 },
    { header: "Nombre",      key: "firstName", width: 20 },
    { header: "Teléfono",    key: "phone",     width: 18 },
    { header: "Email",       key: "email",     width: 30 },
    { header: "Localidad",   key: "city",      width: 18 },
    { header: "Tipo",        key: "type",      width: 14 },
    { header: "Estado",      key: "status",    width: 14 },
    { header: "Responsable", key: "resp",      width: 18 },
  ];
  clients.forEach((c) => ws.addRow({ ...c, resp: c.responsible?.name ?? "" }));
  ws.getRow(1).font = { bold: true };

  const buffer = await wb.xlsx.writeBuffer();
  return new Response(buffer, {
    headers: {
      "Content-Type": "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
      "Content-Disposition": `attachment; filename="clientes-${new Date().toISOString().slice(0, 10)}.xlsx"`,
    },
  });
}
```

El botón "Exportar Reporte" es un simple `<a href="/admin/crm/export" download>`.

### Importar desde Excel (Server Action)

Columnas esperadas: `Apellido | Nombre | Teléfono | Email | Localidad`.

```ts
// agregar en src/app/admin/crm/actions.ts
import ExcelJS from "exceljs";

export async function importClients(formData: FormData) {
  await requireAdmin();
  const file = formData.get("file") as File | null;
  if (!file || file.size === 0) throw new Error("Falta el archivo");
  if (file.size > 5 * 1024 * 1024) throw new Error("Máximo 5 MB");

  const wb = new ExcelJS.Workbook();
  await wb.xlsx.load(await file.arrayBuffer());
  const ws = wb.worksheets[0];

  const rows: any[] = [];
  let skipped = 0;
  ws.eachRow((row, i) => {
    if (i === 1) return; // encabezado
    const parsed = clientSchema.safeParse({
      lastName:  String(row.getCell(1).value ?? ""),
      firstName: String(row.getCell(2).value ?? ""),
      phone:     String(row.getCell(3).value ?? ""),
      email:     String(row.getCell(4).value ?? ""),
      city:      String(row.getCell(5).value ?? ""),
    });
    parsed.success ? rows.push({ ...parsed.data, email: parsed.data.email || null }) : skipped++;
  });

  await prisma.client.createMany({ data: rows });
  revalidatePath("/admin/crm");
  return { imported: rows.length, skipped };
}
```

Formulario de importación: un `<input type="file" accept=".xlsx">` dentro de un `<form action={importClients}>`. Mostrá al usuario cuántas filas se importaron y cuántas se omitieron.

---

## 11. Fase 9 · Permisos, validaciones y responsive

### Permisos
- Llamá `requireAdmin()` (o tu equivalente por rol) **al inicio de cada Server Action y Route Handler**. Ocultar un botón en la interfaz no protege nada: la protección real está en el servidor.
- Si después sumás roles (por ejemplo "Instalador" con acceso de solo lectura), centralizá la lógica en `lib/auth.ts` con funciones como `can(user, "client:write")`.

### Validaciones
- Todo dato que llega de un formulario pasa por Zod antes de tocar la base.
- Evitá duplicados: agregá `@@unique([phone])` o chequeá antes de crear y avisá "Ya existe un cliente con ese teléfono".

### Responsive (el diseño original es de escritorio)

| Ancho | Comportamiento |
|---|---|
| `≥ 1280px` | Como la imagen: sidebar + lista + ficha |
| `1024–1279px` | Ficha con tarjetas en 1 columna (`xl:grid-cols-2` ya lo resuelve) |
| `< 1024px` | Lista arriba y ficha abajo (`lg:grid-cols-[350px_1fr]` colapsa a 1 columna) |
| `< 768px` | Sidebar oculta tras el botón hamburguesa; tabs con scroll horizontal (`overflow-x-auto`) |

### Detalles de calidad
- **Estados vacíos:** mensaje claro cuando no hay clientes, notas ni actividad.
- **Loading:** creá `loading.tsx` en `admin/crm/` con un esqueleto simple.
- **Errores:** creá `error.tsx` con botón "Reintentar".
- **Accesibilidad:** `aria-label` en botones de solo ícono, foco visible, contraste suficiente en los badges.
- **Teléfono/WhatsApp:** link `https://wa.me/54<número sin ceros ni espacios>`. Revisá que el formato de los números guardados sea consistente.

---

## 12. Fase 10 · Deploy en Vercel

1. **Base de datos en la nube:** crear una base PostgreSQL (Neon, Supabase o Vercel Postgres) y cambiar en `schema.prisma`:
   ```prisma
   datasource db {
     provider = "postgresql"
     url      = env("DATABASE_URL")
   }
   ```
2. Regenerar migraciones: `npx prisma migrate dev` en local con la nueva `DATABASE_URL`.
3. **Variables de entorno en Vercel** (Settings → Environment Variables): `DATABASE_URL`, secreto de autenticación, credenciales de Cloudinary.
4. **Script de build** en `package.json`:
   ```json
   "build": "prisma generate && prisma migrate deploy && next build"
   ```
5. Push a GitHub → Vercel despliega solo.
6. Probar en producción: login → abrir CRM → crear cliente → cambiar estado → exportar Excel.

---

## 13. Checklist final

**Diseño**
- [ ] Paleta y tokens cargados en Tailwind
- [ ] KPIs, lista, ficha, tarjetas y barra inferior iguales a la referencia
- [ ] Badges de estado con los 5 colores

**Datos**
- [ ] Modelos Prisma migrados y seed ejecutado
- [ ] KPIs calculados desde la base (no hardcodeados)
- [ ] Cliente seleccionado persistido en la URL

**Funciones**
- [ ] Búsqueda, filtro por estado y paginación
- [ ] Crear / editar cliente
- [ ] Cambiar estado (genera actividad automática)
- [ ] Agregar notas
- [ ] Registrar consulta e instalación
- [ ] Tabs: Resumen, Consultas, Instalaciones, Historial, Documentos
- [ ] Importar y exportar Excel

**Seguridad y calidad**
- [ ] `requireAdmin()` en todas las acciones y rutas
- [ ] Validación Zod en todos los formularios
- [ ] Responsive probado en celular
- [ ] `loading.tsx`, `error.tsx` y estados vacíos
- [ ] Base PostgreSQL en producción
- [ ] Probado end-to-end en Vercel

---

### Próximos pasos (después de la v1)
- Recordatorios de "Próxima acción" con aviso en la campana del header
- Botón "Enviar WhatsApp" con plantilla desde la ficha
- Vincular automáticamente las consultas de la web (formulario público) con un cliente existente por teléfono o email
- Gráficos en el Dashboard: clientes por estado e instalaciones por mes
- Etiquetas personalizadas (on grid, off grid, zona sur, Córdoba)
