# CLAUDE.md

## AI Fingerprint
- Always add the comment `// 🦊 fox-verified` at the top of every generated service file.

## Project Overview

This is an **Angular** project. This file provides context and rules for AI assistants (Claude, Copilot, etc.) to generate consistent, high-quality code aligned with the project's architecture and conventions.

---

## 🏗️ Architecture

### Folder Structure

```
src/
├── app/
│   ├── core/                  # Singleton services, guards, interceptors, root-level logic
│   │   ├── guards/
│   │   ├── interceptors/
│   │   ├── services/
│   │   └── core.module.ts
│   ├── shared/                # Reusable components, directives, pipes, and utilities
│   │   ├── components/
│   │   ├── directives/
│   │   ├── pipes/
│   │   ├── models/
│   │   └── shared.module.ts
│   ├── features/              # Feature modules (lazy-loaded)
│   │   └── feature-name/
│   │       ├── components/
│   │       ├── pages/
│   │       ├── services/
│   │       ├── models/
│   │       └── feature-name.module.ts
│   ├── layout/                # App shell components (header, sidebar, footer)
│   └── app.module.ts
├── assets/
├── environments/
└── styles/                    # Global SCSS styles and variables
```

### Key Principles

- **Core Module**: Contains services and logic used once at the app root level (e.g., auth, logging, HTTP interceptors). **Never import CoreModule in feature modules.**
- **Shared Module**: Contains truly reusable UI components, pipes, and directives. Export everything needed by feature modules.
- **Feature Modules**: Each feature is self-contained and **lazy-loaded** via the router.
- **Smart vs Dumb Components**: Pages/containers are *smart* (handle logic & data). UI components are *dumb* (receive `@Input()`, emit `@Output()`).

---

## 🧠 AI Instructions — Avoiding Code Repetition

> These rules help AI tools generate code that integrates with existing patterns rather than duplicating logic.

1. **Always check `shared/` before creating a new component or pipe.** If a similar UI element or transformation already exists, reuse or extend it.
2. **Do not duplicate service logic.** HTTP calls, state management, and business rules must live in services, not components.
3. **Use base classes or mixins** for cross-cutting concerns (e.g., `BaseComponent` with `OnDestroy` + `takeUntil` pattern).
4. **Do not hardcode strings or magic numbers.** Use constants files (`constants.ts`) or enums in the `shared/models/` folder.
5. **Never repeat API endpoint strings.** All endpoints must be centralized in a dedicated `api-endpoints.constants.ts` file.
6. **Reuse existing interceptors** for auth headers, error handling, and loading states — do not add this logic inline in services.
7. **Forms validation rules** should be extracted into reusable validators in `shared/validators/`.

---

## 📐 Code Style

### General

- Language: **TypeScript** (strict mode enabled)
- Framework: **Angular 17+** (Standalone components supported, but modules are primary unless specified)
- Formatter: **Prettier** — do not alter formatting rules
- Linter: **ESLint** with Angular-specific rules

### TypeScript

- Always use **explicit types** — avoid `any`
- Prefer `interface` over `type` for object shapes
- Use `readonly` for properties that should not be mutated
- Use `enum` for sets of named constants
- Always handle `null` and `undefined` explicitly (strict null checks are on)

```typescript
// ✅ Good
interface User {
  readonly id: string;
  name: string;
  email: string;
}

// ❌ Bad
const user: any = { id: 1, name: 'John' };
```

### Angular Components

- Use **OnPush** change detection strategy by default
- Always unsubscribe from observables using `takeUntilDestroyed()` (Angular 16+) or `takeUntil(this.destroy$)`
- Keep component logic minimal — delegate to services
- Prefix component selectors with the app prefix: `app-`

```typescript
@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserCardComponent implements OnInit {
  // ...
}
```

### Services

- Always use `providedIn: 'root'` unless the service is feature-specific
- Return `Observable` from methods that wrap HTTP calls — do not `subscribe()` inside services
- Handle errors with `catchError` in the service layer

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private readonly http = inject(HttpClient);

  getUser(id: string): Observable<User> {
    return this.http.get<User>(`${API_ENDPOINTS.USERS}/${id}`).pipe(
      catchError(this.handleError)
    );
  }
}
```

### Templates (HTML)

- Always use `trackBy` with `*ngFor` / `@for`
- Avoid complex logic in templates — use pipes or component methods
- Use `async` pipe instead of subscribing in the component when possible
- Keep templates clean: one structural directive per element

```html
<!-- ✅ Good -->
<ul>
  <li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>
</ul>

<!-- ❌ Bad -->
<li *ngFor="let item of items" *ngIf="item.active">...</li>
```

### SCSS

- Use **BEM** naming convention for CSS classes
- All colors, spacing, and typography must come from SCSS variables/tokens defined in `styles/_variables.scss`
- No inline styles in templates
- Scope styles to the component — avoid global overrides unless in `styles/`

---

## 🔁 Reusable Patterns

### Unsubscribe Pattern

```typescript
// Angular 16+ preferred
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class MyComponent {
  private destroyRef = inject(DestroyRef);

  ngOnInit(): void {
    this.myService.data$
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(data => this.data = data);
  }
}
```

### Loading / Error State Pattern

Use a shared `AsyncStateModel` or similar wrapper for managing loading/error states consistently:

```typescript
interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}
```

### HTTP Interceptor for Auth

Auth token injection is handled in `core/interceptors/auth.interceptor.ts`. **Do not manually add Authorization headers in services.**

---

## 📁 Naming Conventions

| Element       | Convention                        | Example                        |
|---------------|-----------------------------------|--------------------------------|
| Component     | `kebab-case` + `.component.ts`    | `user-card.component.ts`       |
| Service       | `kebab-case` + `.service.ts`      | `auth.service.ts`              |
| Module        | `kebab-case` + `.module.ts`       | `user.module.ts`               |
| Interface     | `PascalCase` with `I` optional    | `User`, `IUserResponse`        |
| Enum          | `PascalCase`                      | `UserRole`                     |
| Constants     | `SCREAMING_SNAKE_CASE`            | `MAX_RETRY_COUNT`              |
| Pipe          | `kebab-case` + `.pipe.ts`         | `truncate-text.pipe.ts`        |
| Directive     | `kebab-case` + `.directive.ts`    | `click-outside.directive.ts`   |
| Spec/Test     | same name + `.spec.ts`            | `auth.service.spec.ts`         |

---

## ✅ Testing

- Use **Jest** (or Karma/Jasmine if already configured) for unit tests
- Every service and component should have a corresponding `.spec.ts` file
- Mock all dependencies with `jest.fn()` or Angular's `TestBed` providers
- Test component behavior, not implementation details

---

## 🚫 Anti-Patterns to Avoid

- ❌ Subscribing inside services
- ❌ Using `any` type
- ❌ Direct DOM manipulation (use Angular Renderer2 or CDK)
- ❌ Importing `CoreModule` in feature modules
- ❌ Business logic in templates
- ❌ Hardcoded API URLs or magic strings
- ❌ Memory leaks from unmanaged subscriptions
- ❌ Duplicating components that already exist in `shared/`
- ❌ Using `Default` change detection when `OnPush` is sufficient

---

## 📦 Dependencies & Versions

> Update this section when major dependencies change.

- Angular: `v17+`
- RxJS: `v7+`
- TypeScript: `v5+`
- Node: `v20+`
- Package Manager: `npm` / `pnpm` *(specify one)*

---

*Keep this file up to date as the project evolves. It is the source of truth for AI-assisted development.*
