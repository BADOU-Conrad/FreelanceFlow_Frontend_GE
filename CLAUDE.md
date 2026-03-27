# CLAUDE.md — FreelanceFlow Frontend

> Fichier de contexte pour Claude Code (`/init`).
> À placer à la racine de `freelanceflow-frontend/`.
> Repo : `github.com/FreelanceFlow-Team/freelanceflow-frontend`

---

## 📋 Présentation du Projet

**FreelanceFlow** est une application web SaaS mono-tenant pour qu'un freelance gère son activité : clients, prestations, factures, génération PDF. Ce repo est le **frontend uniquement**. Le backend est dans un repo séparé (`freelanceflow-backend`).

### Périmètre V1
1. Authentification (inscription + connexion JWT)
2. Gestion des clients — CRUD complet
3. Gestion des prestations — intitulé + tarif horaire
4. Gestion des factures — création, calcul HT/TVA/TTC, statuts, téléchargement PDF

---

## 🛠️ Stack & Versions — NE PAS DÉVIER

| Outil | Version | Notes critiques |
|---|---|---|
| **Node.js** | `22 LTS` | Requis |
| **Next.js** | `16.1.x` | `params`/`searchParams` sont des Promises. `proxy.ts` remplace `middleware.ts`. Turbopack par défaut. |
| **React** | `19.2.x` | React Compiler stable intégré |
| **TypeScript** | `5.7.x` | |
| **Tailwind CSS** | `4.x` | `@import "tailwindcss"` — plus de directives `@tailwind` |
| **@tanstack/react-query** | `5.x` | Gestion requêtes async |
| **react-hook-form** | `7.x` | Formulaires |
| **zod** | `3.x` | Validation |
| **lucide-react** | latest | Icônes |
| **geist** | latest | Police officielle (next/font/google) |
| **ESLint** | `9.x` | Flat config `eslint.config.mjs` — plus de `.eslintrc.json` |
| **Prettier** | `3.x` | |
| **Husky** | `9.x` | Git hooks |
| **lint-staged** | `15.x` | |

---

## 🚨 Règles Breaking Next.js 16 — TOUJOURS APPLIQUER

### 1. `params` et `searchParams` sont des Promises

```typescript
// ❌ INTERDIT — Next.js 15 et avant
export default function Page({ params }: { params: { id: string } }) {
  const { id } = params;
}

// ✅ OBLIGATOIRE — Next.js 16
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params;
}
```

### 2. `middleware.ts` → `proxy.ts`

```typescript
// Fichier : src/proxy.ts (pas middleware.ts)
// Fonction exportée : proxy (pas middleware)
export function proxy(request: NextRequest) { ... }
// Note : tourne uniquement sur Node.js runtime (pas Edge)
```

### 3. Scripts `package.json` — sans flag `--turbopack`

```json
// ❌ INTERDIT
{ "dev": "next dev --turbopack" }

// ✅ CORRECT (Turbopack est le défaut dans Next 16)
{ "dev": "next dev", "build": "next build" }
```

### 4. `next.config.ts`

```typescript
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  output: 'standalone',   // requis pour Docker
  reactCompiler: true,    // stable dans Next 16 (plus dans experimental)
};

export default nextConfig;
```

### 5. Tailwind CSS 4 — plus de config JS

```css
/* globals.css */
@import "tailwindcss";   /* ← remplace @tailwind base/components/utilities */

@theme {
  --color-navy: #1A2332;
  /* ... variables custom ici */
}
```

---

## 📁 Arborescence

```
freelanceflow-frontend/
├── .github/workflows/ci.yml
├── public/logo.svg
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx              # Sidebar + auth guard
│   │   │   ├── page.tsx                # Dashboard home
│   │   │   ├── clients/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── new/page.tsx
│   │   │   │   └── [id]/page.tsx       # ⚠️ params async
│   │   │   ├── services/
│   │   │   │   ├── page.tsx
│   │   │   │   └── new/page.tsx
│   │   │   └── invoices/
│   │   │       ├── page.tsx
│   │   │       ├── new/page.tsx
│   │   │       └── [id]/page.tsx       # ⚠️ params async
│   │   ├── layout.tsx                  # Root layout (Geist + providers)
│   │   └── globals.css                 # Tailwind 4 + CSS variables
│   │
│   ├── components/
│   │   ├── ui/                         # Primitives : Button, Input, Badge, Modal, Table
│   │   ├── layout/
│   │   │   ├── sidebar.tsx             # Nav principale (fond Navy #1A2332)
│   │   │   └── page-header.tsx         # Titre + bouton action
│   │   └── shared/
│   │       ├── status-badge.tsx        # Badge statut facture
│   │       └── confirm-dialog.tsx
│   │
│   ├── features/
│   │   ├── auth/
│   │   │   ├── components/login-form.tsx
│   │   │   ├── hooks/use-auth.ts
│   │   │   └── api.ts
│   │   ├── clients/
│   │   │   ├── components/
│   │   │   │   ├── client-table.tsx
│   │   │   │   └── client-form.tsx
│   │   │   ├── hooks/use-clients.ts
│   │   │   ├── api.ts
│   │   │   └── types.ts
│   │   ├── services/
│   │   │   ├── components/
│   │   │   │   ├── service-table.tsx
│   │   │   │   └── service-form.tsx
│   │   │   ├── hooks/use-services.ts
│   │   │   ├── api.ts
│   │   │   └── types.ts
│   │   └── invoices/
│   │       ├── components/
│   │       │   ├── invoice-form.tsx
│   │       │   ├── invoice-table.tsx
│   │       │   └── invoice-line-item.tsx
│   │       ├── hooks/use-invoices.ts
│   │       ├── api.ts
│   │       └── types.ts
│   │
│   ├── lib/
│   │   ├── api-client.ts               # Client fetch (Bearer token)
│   │   ├── auth.ts                     # Helpers token/session
│   │   └── utils.ts                    # formatCurrency, formatDate…
│   │
│   ├── types/index.ts
│   └── providers/query-provider.tsx    # QueryClientProvider
│
├── .env.local.example
├── .husky/pre-commit
├── eslint.config.mjs
├── .prettierrc
├── Dockerfile
├── next.config.ts
├── package.json
└── tsconfig.json
```

---

## 🔑 Fichiers de configuration clés

### `src/app/layout.tsx`
```typescript
import type { Metadata } from 'next';
import { Geist, Geist_Mono } from 'next/font/google';
import './globals.css';

const geist = Geist({ subsets: ['latin'], variable: '--font-geist' });
const geistMono = Geist_Mono({ subsets: ['latin'], variable: '--font-geist-mono' });

export const metadata: Metadata = {
  title: 'FreelanceFlow',
  description: 'Gérez votre activité freelance',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fr">
      <body className={`${geist.variable} ${geistMono.variable} font-sans`}>
        {children}
      </body>
    </html>
  );
}
```

### `src/app/globals.css`
```css
@import "tailwindcss";

@theme {
  --color-navy:   #1A2332;
  --color-slate:  #2D4A6E;
  --color-cobalt: #3B82C4;
  --color-azure:  #63A3FF;
  --color-frost:  #E8F4FF;

  --color-bg-page: #F8F7F4;
  --color-bg-card: #FFFFFF;
  --color-border:  #E2E0D8;
  --color-muted:   #8A8880;
  --color-text:    #2C2C2A;

  --color-success:       #0D6E4F;
  --color-success-light: #D4F0E8;
  --color-warning:       #B85C00;
  --color-warning-light: #FFF0D4;
  --color-danger:        #C0392B;
  --color-danger-light:  #F5E8E8;

  --font-sans: "Geist", system-ui, sans-serif;
  --font-mono: "Geist Mono", monospace;

  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 10px;
  --radius-xl: 12px;
}

body {
  background: var(--color-bg-page);
  color: var(--color-text);
  font-family: var(--font-sans);
  -webkit-font-smoothing: antialiased;
}
```

### `src/lib/api-client.ts`
```typescript
const API_BASE = process.env.NEXT_PUBLIC_API_URL ?? 'http://localhost:3001/api';

function getToken(): string | null {
  if (typeof window === 'undefined') return null;
  return localStorage.getItem('ff_token');
}

async function fetchApi<T>(path: string, options?: RequestInit): Promise<T> {
  const token = getToken();
  const res = await fetch(`${API_BASE}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options?.headers,
    },
  });

  if (res.status === 401) {
    localStorage.removeItem('ff_token');
    window.location.href = '/login';
    throw new Error('Unauthorized');
  }

  if (!res.ok) {
    const error = await res.json().catch(() => ({}));
    throw new Error(error.message ?? `HTTP ${res.status}`);
  }

  if (res.status === 204) return undefined as T;
  return res.json() as Promise<T>;
}

export const apiClient = {
  get:    <T>(path: string)                => fetchApi<T>(path),
  post:   <T>(path: string, body: unknown) => fetchApi<T>(path, { method: 'POST',   body: JSON.stringify(body) }),
  patch:  <T>(path: string, body: unknown) => fetchApi<T>(path, { method: 'PATCH',  body: JSON.stringify(body) }),
  delete: <T>(path: string)               => fetchApi<T>(path, { method: 'DELETE' }),
};
```

---

## 🎨 Charte Graphique

### Direction artistique
Sobriété professionnelle — inspiré de Linear/Notion. Pas de dégradés, pas de glassmorphisme. Outil quotidien = reposant et efficace.

### Layout
- **Sidebar** : 240px, fond `#1A2332` (Navy), texte `#8BA5C2`
- **Item actif sidebar** : fond `rgba(99,163,255,0.15)`, texte `#63A3FF`
- **Page** : fond `#F8F7F4` (off-white chaud), padding `p-6`
- **Cards** : fond blanc, border `#E2E0D8`, `border-radius: 10px`, shadow `0 1px 3px rgba(0,0,0,0.08)`

### Classes Tailwind courantes
```typescript
// Bouton primaire
"bg-[#3B82C4] text-white rounded-md px-4 py-2 text-sm font-medium hover:bg-[#2D74B5] transition-colors focus:ring-2 focus:ring-[#63A3FF] focus:ring-offset-2"

// Bouton secondaire
"border border-[#E2E0D8] text-[#1A2332] rounded-md px-4 py-2 text-sm font-medium hover:bg-[#F8F7F4] transition-colors"

// Card
"bg-white rounded-[10px] border border-[#E2E0D8] shadow-[0_1px_3px_rgba(0,0,0,0.08)] p-6"

// Input
"w-full border border-[#E2E0D8] rounded-md px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-[#63A3FF] focus:border-transparent"

// Table row
"border-b border-[#E2E0D8] hover:bg-[#F8F7F4] transition-colors"
```

### Badges statuts facture
```typescript
const INVOICE_STATUS_STYLES = {
  DRAFT:     'bg-[#ECEAE4] text-[#4A4845]',
  SENT:      'bg-[#D6E8FF] text-[#0A3A6E]',
  PAID:      'bg-[#D4F0E8] text-[#0A5E42]',
  CANCELLED: 'bg-[#F5E8E8] text-[#7A1A1A]',
} as const;

const INVOICE_STATUS_LABELS = {
  DRAFT:     'Brouillon',
  SENT:      'Envoyée',
  PAID:      'Payée',
  CANCELLED: 'Annulée',
} as const;
```

---

## 🔌 API — Endpoints consommés par le frontend

Backend URL : `NEXT_PUBLIC_API_URL` (ex: `http://localhost:3001/api`)
Toutes les routes sauf `/auth/*` nécessitent `Authorization: Bearer <token>`.

### Auth
| Méthode | Route | Body |
|---|---|---|
| `POST` | `/auth/register` | `{ email, password, firstName, lastName }` |
| `POST` | `/auth/login` | `{ email, password }` → retourne `{ access_token, user }` |

### Clients
| Méthode | Route | Notes |
|---|---|---|
| `GET` | `/clients` | Liste des clients de l'utilisateur connecté |
| `GET` | `/clients/:id` | Détail |
| `POST` | `/clients` | `{ name, email, company?, address? }` |
| `PATCH` | `/clients/:id` | Partial update |
| `DELETE` | `/clients/:id` | 204 No Content |

### Services (Prestations)
| Méthode | Route | Notes |
|---|---|---|
| `GET` | `/services` | |
| `POST` | `/services` | `{ label, hourlyRate }` |
| `PATCH` | `/services/:id` | |
| `DELETE` | `/services/:id` | |

### Invoices (Factures)
| Méthode | Route | Notes |
|---|---|---|
| `GET` | `/invoices` | |
| `GET` | `/invoices/:id` | Avec client + lignes |
| `POST` | `/invoices` | `{ clientId, lines: [{serviceId, quantity, unitPrice, description?}], vatRate?, dueAt?, notes? }` |
| `PATCH` | `/invoices/:id/status` | `{ status: 'DRAFT'\|'SENT'\|'PAID'\|'CANCELLED' }` |
| `DELETE` | `/invoices/:id` | |
| `GET` | `/invoices/:id/pdf` | Retourne `application/pdf` — déclencher un download |

### Téléchargement PDF
```typescript
async function downloadInvoicePdf(invoiceId: string) {
  const token = localStorage.getItem('ff_token');
  const res = await fetch(
    `${process.env.NEXT_PUBLIC_API_URL}/invoices/${invoiceId}/pdf`,
    { headers: { Authorization: `Bearer ${token}` } }
  );
  const blob = await res.blob();
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `facture-${invoiceId}.pdf`;
  a.click();
  URL.revokeObjectURL(url);
}
```

---

## 🧩 Pattern Feature — Ordre de création

Pour chaque nouvelle feature (ex: clients), créer dans cet ordre :

```
1. types.ts        → interfaces TypeScript
2. api.ts          → appels apiClient
3. hooks/use-*.ts  → React Query (useQuery / useMutation)
4. components/     → composants UI
5. app/.../page.tsx → page Next.js (route)
```

### Exemple types.ts
```typescript
export interface Client {
  id: string;
  name: string;
  email: string;
  company: string | null;
  address: string | null;
  createdAt: string;
}

export interface CreateClientDto {
  name: string;
  email: string;
  company?: string;
  address?: string;
}

export type UpdateClientDto = Partial<CreateClientDto>;
```

### Exemple hook React Query
```typescript
'use client';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { clientsApi } from '../api';
import type { CreateClientDto } from '../types';

export const CLIENTS_KEY = ['clients'] as const;

export function useClients() {
  return useQuery({ queryKey: CLIENTS_KEY, queryFn: clientsApi.getAll });
}

export function useCreateClient() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateClientDto) => clientsApi.create(data),
    onSuccess: () => qc.invalidateQueries({ queryKey: CLIENTS_KEY }),
  });
}

export function useDeleteClient() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: clientsApi.delete,
    onSuccess: () => qc.invalidateQueries({ queryKey: CLIENTS_KEY }),
  });
}
```

---

## 🐳 Docker

### `Dockerfile`
```dockerfile
FROM node:22-alpine AS base
WORKDIR /app
COPY package*.json ./

FROM base AS builder
RUN npm ci
COPY . .
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
RUN npm run build

FROM node:22-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

---

## ⚙️ CI — GitHub Actions

```yaml
name: CI Frontend
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
      - run: npx tsc --noEmit
      - run: npm run build
        env:
          NEXT_PUBLIC_API_URL: ${{ vars.NEXT_PUBLIC_API_URL }}
      - name: Build Docker image
        run: docker build --build-arg NEXT_PUBLIC_API_URL=${{ vars.NEXT_PUBLIC_API_URL }} -t freelanceflow-frontend:${{ github.sha }} .
      - name: Login & Push GHCR
        if: github.ref == 'refs/heads/main'
        uses: docker/login-action@v3
        with: { registry: ghcr.io, username: ${{ github.actor }}, password: ${{ secrets.GITHUB_TOKEN }} }
      - if: github.ref == 'refs/heads/main'
        run: |
          docker tag freelanceflow-frontend:${{ github.sha }} ghcr.io/${{ github.repository_owner }}/freelanceflow-frontend:latest
          docker push ghcr.io/${{ github.repository_owner }}/freelanceflow-frontend:latest
      - name: Deploy to Vercel
        if: github.ref == 'refs/heads/main'
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

---

## 🔀 Git

```
main      ← production (PR obligatoire)
develop   ← intégration
feature/<id>-<description>
fix/<id>-<description>
```

Commits : Conventional Commits — `feat:`, `fix:`, `chore:`, `test:`, `docs:`
PRs vers `develop` uniquement. Referencer l'issue avec `Closes #<id>`.

### `lint-staged` (`package.json`)
```json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,yml}": ["prettier --write"]
  }
}
```

---

## 📝 Variables d'environnement

### `.env.local`
```env
NEXT_PUBLIC_API_URL="http://localhost:3001/api"
```

### Production (Vercel)
```env
NEXT_PUBLIC_API_URL="https://api.freelanceflow.up.railway.app/api"
```

---

## 🔍 Points d'attention Claude Code

1. **Toujours `await` les params** dans les `page.tsx` — c'est un `Promise<{...}>` dans Next 16
2. **Jamais `--turbopack`** dans les scripts npm — c'est le défaut dans Next 16
3. **`proxy.ts`** et non `middleware.ts` si besoin de middleware
4. **Tailwind 4** : `@import "tailwindcss"` uniquement, pas les directives `@tailwind`
5. **ESLint 9** : flat config `eslint.config.mjs`, pas de `.eslintrc.json`
6. **Le calcul HT/TVA/TTC** est fait côté backend — ne jamais recalculer côté frontend
7. **`'use client'`** obligatoire sur tous les composants utilisant hooks ou events
8. **React Query** pour tous les appels API — pas de `useEffect` + `fetch` direct
9. **Geist** via `next/font/google` — ne pas importer depuis un CDN externe
10. **`output: 'standalone'`** doit rester dans `next.config.ts` pour Docker

---

*Stack : Next.js 16.1 / React 19.2 / Tailwind 4 / Node.js 22 — Mars 2026*
