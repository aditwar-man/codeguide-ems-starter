# Frontend Guidelines for codeguide-ems-starter

These guidelines explain how the frontend of the Educational Management System (EMS) starter kit (`codeguide-ems-starter`) is built, styled, organized, and maintained. Anyone—whether you are new to the project or a seasoned developer—should be able to understand the architecture and best practices without needing a deep technical background.

## 1. Frontend Architecture

**Frameworks and Libraries**
- **Next.js (App Router)**: Provides file-based routing, server-side rendering (SSR), and React Server Components out of the box.
- **React & TypeScript**: Ensures a component-based UI and strong type safety to reduce runtime errors.
- **Better Auth**: Manages session-based authentication for the three user roles (Kurikulum, Guru, Siswa).
- **Tailwind CSS v4**: A utility-first CSS framework for rapid, consistent styling.
- **shadcn/ui**: A collection of pre-built, customizable UI components (DataTable, Chart, Forms, etc.).
- **Drizzle ORM & PostgreSQL**: Type-safe database layer (used on the backend) that integrates seamlessly with Next.js API routes.
- **Docker & Docker Compose**: Containerizes the app and its services (PostgreSQL, Redis) for consistent local and production environments.

**Scalability**
- File-based routing and route groups let you split features by user role.
- Modular folders (`/app`, `/components`, `/db`, `/lib`) keep features isolated and easy to grow.

**Maintainability**
- TypeScript enforces consistent data shapes across the app.
- Centralized theming and utility classes (Tailwind) reduce CSS duplication.
- Clear separation between UI components, API routes, and data schemas.

**Performance**
- Server Components fetch sensitive data server-side, keeping the client bundle small.
- Automatic code-splitting per page and dynamic imports for large modules.
- Image optimization via Next.js built-in `<Image>` component.

## 2. Design Principles

1. **Usability**: Intuitive navigation, clear labels, and consistent layouts help users complete tasks quickly.
2. **Accessibility**: All interactive elements meet WCAG 2.1 AA standards (semantic HTML, ARIA attributes, keyboard support).
3. **Responsiveness**: Mobile-first, fluid layouts scale from phones to desktop screens.
4. **Security**: Sensitive data (grades, student info) is fetched on the server. Role-based access control (RBAC) is enforced via middleware.
5. **Consistency**: Shared design tokens (colors, spacing, typography) ensure a unified look across portals.

## 3. Styling and Theming

**Styling Approach**
- **Utility-first with Tailwind CSS**: Avoids custom CSS files; most styling lives in class names (e.g., `p-4 bg-gray-100`).
- **No additional preprocessors**—Tailwind’s JIT engine covers most use cases.

**Theming**
- **Light and Dark Modes**: Controlled via CSS variables and Tailwind’s `dark:` variant.
- **Theme Switcher**: A small toggle in the header persists user preference in local storage.

**Visual Style**
- **Modern Flat Design**: Clean shapes, minimal shadows, subtle glassmorphism on modals and cards.

**Color Palette**
- Primary: #1E40AF (Blue-700)
- Secondary: #10B981 (Green-500)
- Accent: #F59E0B (Amber-500)
- Neutral Light: #F3F4F6 (Gray-100)
- Neutral Dark: #111827 (Gray-900)
- Error: #EF4444 (Red-500)
- Success: #22C55E (Green-400)

**Typography**
- **Font Family**: ‘Inter’, sans-serif for readability and a modern aesthetic.
- **Font Sizes**: Defined in Tailwind’s scale (e.g., `text-sm`, `text-base`, `text-lg`).

## 4. Component Structure

**Folder Organization**
- `/app`: Page files and layouts, organized into route groups for each portal (`/app/kurikulum`, `/app/guru`, `/app/siswa`).
- `/components/ui`: Reusable building blocks from shadcn/ui (buttons, tables, charts).
- `/components/common`: Shared app components (Sidebar, Header, ThemeToggle).
- `/lib`: Utility functions and type definitions.

**Reuse and Modularity**
- Each UI element is a self-contained React component.
- Props and events pass data in and out, avoiding global state where possible.
- Custom hooks (e.g., `useAuth`) abstract common logic.

**Benefits**
- Easier to find and fix bugs.
- Teams can work in parallel on different components.
- New features plug in without rewriting existing code.

## 5. State Management

**Authentication & User State**
- **Better Auth** handles sessions on the server, exposing a `getSession()` helper in React Server Components.
- A React Context (`AuthContext`) provides user info to client components that need it.

**Server State**
- **React Query** (`@tanstack/react-query`) for data fetching, caching, and background updates (e.g., class rosters, grades).

**Local UI State**
- Simple state (`useState`) for form inputs and toggles.
- **Zustand** for complex multi-step forms or cross-component UI state when needed.

## 6. Routing and Navigation

**Next.js App Router**
- **Page-based routing**: Files under `/app` automatically map to routes (e.g., `/app/kurikulum/dashboard/page.tsx` → `/kurikulum/dashboard`).
- **Route Groups**: Group shared layouts under `(portals)` and split by role.
- **Server Components**: Fetch data in `page.tsx` before sending HTML to the client.

**Navigation Structure**
- **Sidebar**: Role-aware links generated based on the user’s role.
- **Top Bar**: Theme toggle, user menu, notifications.
- **Breadcrumbs**: Show page hierarchy for easier back-and-forth movement.

## 7. Performance Optimization

1. **Server-side Rendering (SSR) & Server Components**: Reduces client JS bundle size.
2. **Code Splitting**: Dynamic `import()` for heavy components (charts, maps).
3. **Lazy Loading**: Images and non-critical modules load as they appear in view.
4. **Tailwind Purge**: Removes unused CSS in production.
5. **Next.js Image Component**: Automatic resizing and format selection.
6. **Caching Layer**: Redis (via Docker Compose) caches frequent queries (class lists, subject catalog).

## 8. Testing and Quality Assurance

**Linting & Formatting**
- **ESLint** with Next.js and TypeScript presets.
- **Prettier** for consistent code style.
- **Husky + lint-staged** to run checks on pre-commit.

**Unit Tests**
- **Vitest** + **React Testing Library** for components and utility functions.

**Integration Tests**
- **Mock Service Worker (MSW)** to simulate API responses without hitting a real database.

**End-to-End (E2E) Tests**
- **Playwright** or **Cypress** to automate critical user flows (login, grade entry, schedule viewing).

**CI/CD**
- **GitHub Actions** pipeline runs lint, tests, and builds the Docker image on every push.

## 9. Conclusion and Overall Frontend Summary

This guideline outlines how **codeguide-ems-starter** delivers a modern, modular, and scalable frontend for an Educational Management System. By combining Next.js App Router, TypeScript, Tailwind CSS, and shadcn/ui, we achieve fast performance, clear component boundaries, and an accessible, responsive UI. State management via React Query and simple React Contexts keeps data in sync. Automated tests and CI/CD ensure reliability as the system grows.

Unique strengths of this setup include:
- **Server Components** and SSR for security and speed.
- **Utility-first styling** for rapid, consistent theming.
- **Role-based routing** and layout groups for three distinct user portals.

With these guidelines, any developer can confidently extend the starter kit into a full-featured EMS, while maintaining high code quality and a great user experience.