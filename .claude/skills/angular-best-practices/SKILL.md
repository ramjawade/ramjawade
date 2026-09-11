---
name: angular-best-practices
description: TypeScript/Angular coding rules for this repo (components, signals, templates, services, data loading, accessibility). Load whenever writing or reviewing Angular code in projects/home or [...]
---

# Angular & TypeScript best practices

Project is on **Angular 20.3** (`projects/home/package.json` / root `package.json`).

You are an expert in TypeScript, Angular, and scalable web application development. Write functional, maintainable, performant, and accessible code following the rules below.

## TypeScript

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid `any`; use `unknown` when the type is uncertain

## Angular

- Always use standalone components over NgModules
- Must NOT set `standalone: true` in decorators — it's the default in v20+
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use `@HostBinding`/`@HostListener` — put host bindings in the `host` object of `@Component`/`@Directive`
- Use `NgOptimizedImage` for static images (does not work for inline base64 images)

## SOLID

- **Single Responsibility**: a component owns one page/view's presentation and local state; a service owns one domain's logic and persistence. If a component is doing data-fetch orchestration, business logic, or state lifecycle management, refactor.
- **Open/Closed**: extend via composition (new component, new `input()`/`output()`, a new service method) rather than branching an existing method on a type flag to add a case.
- **Liskov**: an abstract-service consumer (e.g. code depending on `IWeatherService`) must work unchanged against any implementation (`WeatherService`, a future mock/alt source) — don't have implementation-specific logic at the call site.
- **Interface Segregation**: keep `input()`/`output()` surfaces to what a component actually uses — don't hand a child the whole parent service just so it can call one method; pass the specific data or function instead.
- **Dependency Inversion**: components and services depend on `inject()`-provided abstractions (interfaces like `IStorageService`/`IWeatherService`, or a service's public API), never reach into another service's internals or assume a singleton instance.

In practice for this repo: a routed page component (e.g. `MapComponent`) owns its page's data lifecycle (fetch-on-init, local state) and delegates the actual business logic and persistence to its service.

## Data loading (routes)

- Fetch page data with a **functional route resolver** (`ResolveFn`, `inject()`), not from a component's `ngOnInit`/constructor. The resolver calls the owning service's load method directly; wire it up in the route definition.
- `provideRouter(routes, withComponentInputBinding())` is enabled — prefer resolved data arriving as a component `input()`/`input.required()` over reading `ActivatedRoute.data` manually.
- A resolver re-runs every time its route activates — this is how a page gets fresh data on every visit instead of once per session.
- Domain services keep their signals for cross-component sharing; the resolver's only job is to trigger the fetch that populates them, not to own the data itself.

## Accessibility

- MUST pass all AXE checks
- MUST follow WCAG AA minimums: focus management, color contrast, ARIA attributes

## Components

- Keep components small, single-responsibility
- Use `input()`/`output()` functions instead of decorators
- Use `model()` for `[(prop)]` two-way binding instead of pairing `input()`+`output()`
- Use `computed()` for derived state
- Use `linkedSignal()` for state derived from multiple reactive sources that must stay synchronized
- Prefer inline templates for small components
- Prefer Reactive forms over template-driven
- Do NOT use `ngClass`/`ngStyle` — use `class`/`style` bindings
- Do NOT import `CommonModule` — import only the directives/pipes the template uses (e.g. `AsyncPipe`, `DatePipe`)
- External templates/styles: use paths relative to the component TS file

## State management

- Signals for local component state
- `computed()` for derived state
- Keep state transformations pure and predictable
- Do NOT use `.mutate()` on signals — use `.update()` or `.set()`

## Templates

- Keep templates simple, avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`/`*ngFor`/`*ngSwitch`
- Use the async pipe for observables
- Don't assume globals like `new Date()` are available

## Services

- Single responsibility per service
- Use `providedIn: 'root'` for singletons
- Use `inject()` instead of constructor injection
