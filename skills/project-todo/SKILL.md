# Project: Todo App Dashboard

Full-stack todo app built on a shadcn/ui dashboard shell. The UI framework is already set up — you add features into the existing layout.

## Tech Stack
- **Framework**: Next.js 16 (App Router), React 19, TypeScript
- **UI**: shadcn/ui components (in `src/components/ui/`), Tailwind CSS v4
- **Charts**: Recharts (pre-installed)
- **Data tables**: @tanstack/react-table (pre-installed, with shadcn wrappers in `src/components/ui/table/`)
- **Forms**: React Hook Form + Zod validation (pre-installed, form components in `src/components/forms/`)
- **State**: Zustand (pre-installed)
- **Toasts**: Sonner (pre-installed)
- **Icons**: Lucide React + Tabler Icons
- **Database**: SQLite via better-sqlite3 (zero config, file-based)
- **Package manager**: npm

## Repo
- GitHub: `crazyclaw85-source/todo-app-dashboard`
- Clone to: `/tmp/todo-app-dashboard` (working directory for this task)
- Branch convention: `feat/<task-slug>` from `main`

## Project Structure
```
todo-app-dashboard/
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Root layout (providers, fonts)
│   │   ├── page.tsx                # Redirects to /dashboard
│   │   ├── dashboard/
│   │   │   ├── layout.tsx          # Dashboard shell (sidebar + header)
│   │   │   ├── page.tsx            # Dashboard overview
│   │   │   └── todos/
│   │   │       └── page.tsx        # Todo list page (placeholder — build this)
│   │   └── api/
│   │       └── todos/
│   │           ├── route.ts        # GET all, POST create
│   │           └── [id]/
│   │               └── route.ts    # PATCH update, DELETE
│   ├── components/
│   │   ├── ui/                     # shadcn components (button, card, dialog, input, table/, etc.)
│   │   ├── forms/                  # Form field components (input, select, checkbox, etc.)
│   │   ├── layout/                 # Dashboard layout (sidebar, header, providers)
│   │   └── breadcrumbs.tsx
│   ├── config/
│   │   └── nav-config.ts           # Sidebar navigation items
│   ├── lib/
│   │   ├── db.ts                   # SQLite connection + schema
│   │   ├── utils.ts                # cn() helper
│   │   └── data-table.ts           # Table pinning styles
│   ├── types/
│   │   ├── todo.ts                 # Todo interface
│   │   ├── base-form.ts            # Form field types
│   │   └── data-table.ts           # Table column meta types
│   └── hooks/                      # Custom hooks
├── data/                           # SQLite DB file (gitignored)
├── package.json
├── tsconfig.json
└── tailwind.config.ts
```

## Available UI Components (use these, don't build from scratch)
- **Layout**: Sidebar, SidebarHeader, SidebarContent, SidebarFooter, SidebarMenu, SidebarMenuItem
- **Data display**: Card, Badge, Table (with sorting/filtering/pagination via data-table), Avatar
- **Input**: Button, Input, Textarea, Select, Checkbox, Switch, Slider, DatePicker
- **Feedback**: Dialog, AlertDialog, Toast (via sonner), Tooltip, Popover
- **Navigation**: Breadcrumb, Tabs, DropdownMenu, Accordion, Collapsible

Import from `@/components/ui/<component>`. Example:
```tsx
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
```

## API Routes (already implemented)
- `GET /api/todos` — returns all todos as JSON array
- `POST /api/todos` — create todo, body: `{ "title": "..." }`
- `PATCH /api/todos/:id` — update todo, body: `{ "title"?: "...", "completed"?: true/false }`
- `DELETE /api/todos/:id` — delete todo

## Development Commands
```bash
npm install          # Install dependencies
npm run dev          # Start dev server (http://localhost:3000)
npm run build        # Production build (also validates TypeScript)
npm run lint         # ESLint check
npx tsc --noEmit     # Type check only
```

## Coding Standards
- Use TypeScript strict mode
- Use server components by default, `'use client'` only when needed
- Use shadcn/ui components — do NOT create custom UI primitives
- Use Tailwind for styling (no CSS modules)
- API routes return JSON with proper status codes
- Error handling: try/catch in API routes, toast notifications in UI
- State management: Zustand for global state, React state for local

## Verification
After every change, run in order:
1. `npx tsc --noEmit` — must pass
2. `npm run build` — must pass
3. `npm run lint` — must pass
4. Test manually by visiting http://localhost:3000/dashboard/todos
