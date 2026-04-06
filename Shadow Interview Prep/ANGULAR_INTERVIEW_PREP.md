# 🔺 Angular — Complete Interview Guide (v21, 2025)

> **Part 3 of Ahmed's shadow interview prep.**
> Written against the live [angular.dev](https://angular.dev) v21 docs.
> Every section shows **the old way → the new way** so you understand *why* things changed, not just *what* they are.

---

## 📑 Table of Contents

1. [What is Angular?](#1-what-is-angular)
2. [The Big Picture — Architecture](#2-the-big-picture--architecture)
3. [Angular CLI](#3-angular-cli)
4. [Components — The Core Building Block](#4-components--the-core-building-block)
5. [Signals — The New Reactivity Model](#5-signals--the-new-reactivity-model)
6. [Templates & Data Binding](#6-templates--data-binding)
7. [New Built-in Control Flow](#7-new-built-in-control-flow)
8. [Directives](#8-directives)
9. [Pipes](#9-pipes)
10. [Dependency Injection](#10-dependency-injection)
11. [Services](#11-services)
12. [Routing](#12-routing)
13. [Forms](#13-forms)
14. [HTTP Client](#14-http-client)
15. [Component Lifecycle Hooks](#15-component-lifecycle-hooks)
16. [Change Detection](#16-change-detection)
17. [Component Communication](#17-component-communication)
18. [NgModules — The Old Way (and why it's now optional)](#18-ngmodules--the-old-way-and-why-its-now-optional)
19. [Standalone Components — The New Way](#19-standalone-components--the-new-way)
20. [Server-Side Rendering & Hydration](#20-server-side-rendering--hydration)
21. [RxJS in Angular](#21-rxjs-in-angular)
22. [Performance Best Practices](#22-performance-best-practices)
23. [Testing](#23-testing)
24. [Common Pitfalls 🪤](#24-common-pitfalls-)
25. [Quick-Fire Q&A Cheatsheet](#25-quick-fire-qa-cheatsheet)

---

## 1. What is Angular?

Angular is a **platform and framework** for building scalable single-page web applications (and now server-rendered apps) using **TypeScript**. It is maintained by Google and is currently at **v21** (April 2026).

### Angular vs React vs Vue — The Key Difference

| | Angular | React | Vue |
|---|---|---|---|
| Type | Full framework | UI library | Progressive framework |
| Language | TypeScript (mandatory) | JS/TS | JS/TS |
| State management | Signals (built-in) | External (Redux, Zustand) | Composition API / Pinia |
| Routing | Built-in `@angular/router` | External (React Router) | Built-in Vue Router |
| Forms | Built-in reactive + template forms | External (React Hook Form) | Built-in |
| HTTP | Built-in `HttpClient` | `fetch` / Axios | `fetch` / Axios |
| Learning curve | Steep (more opinions) | Moderate | Gentle |
| Best for | Large enterprise apps | Flexible/varied apps | Medium apps |

### Angular's Evolution at a Glance

| Version | Year | Key Change |
|---|---|---|
| Angular 2 | 2016 | Complete rewrite from AngularJS, TypeScript |
| Angular 6-8 | 2018-2019 | Ivy compiler preview |
| Angular 9 | 2020 | Ivy enabled by default |
| Angular 14 | 2022 | Standalone components (experimental) |
| Angular 15 | 2022 | Standalone stable, directive composition |
| Angular 16 | 2023 | Signals introduced (developer preview) |
| Angular 17 | 2023 | New control flow `@if/@for`, new docs site (angular.dev) |
| Angular 18-19 | 2024 | Signals stable, zoneless preview, SSR improvements |
| **Angular 21** | **2025** | **Signals fully stable, linkedSignal, resource API, signal forms** |

### 📖 Resources
- [angular.dev — What is Angular?](https://angular.dev/overview)
- [angular.dev — Installation](https://angular.dev/installation)
- [Angular blog](https://blog.angular.dev)
- [Angular Roadmap](https://angular.dev/roadmap)

---

## 2. The Big Picture — Architecture

An Angular application is a **tree of components**, connected by services via Dependency Injection, with the router controlling which component tree is active.

```
AppComponent (root)
├── NavbarComponent
├── RouterOutlet ← router swaps views here
│   ├── HomeComponent
│   ├── UsersComponent
│   │   ├── UserCardComponent
│   │   └── UserCardComponent
│   └── SettingsComponent
└── FooterComponent

Services (provided at root level — singletons)
├── AuthService       ← shared state, login logic
├── UserService       ← HTTP calls for users
└── ThemeService      ← app-wide theme signal

DI System glues it all together
```

### Core Building Blocks

```
Component    → UI building block (template + logic + styles)
Template     → HTML with Angular syntax (bindings, control flow)
Signal       → reactive state value (the modern way)
Directive    → extend HTML elements with behavior (ngClass, custom)
Pipe         → transform values in templates (date | currency)
Service      → shared logic / data layer
Router       → navigation between views
Module       → (legacy) grouping mechanism, now optional
```

---

## 3. Angular CLI

The CLI is your best friend. Know these commands cold.

```bash
# Install
npm install -g @angular/cli

# Create new project
ng new my-app
ng new my-app --standalone # standalone by default (v17+)

# Serve locally (dev server)
ng serve
ng serve --open  # opens browser

# Generate
ng generate component users/user-card    # or: ng g c
ng generate service services/auth        # ng g s
ng generate directive directives/highlight # ng g d
ng generate pipe pipes/truncate          # ng g p
ng generate guard guards/auth            # ng g guard

# Build
ng build              # production build
ng build --watch      # rebuild on change

# Test
ng test               # unit tests (Karma + Jasmine default)
ng e2e                # end-to-end tests

# Update Angular
ng update @angular/core @angular/cli

# Analyze bundle
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json
```

### 📖 Resources
- [angular.dev — CLI reference](https://angular.dev/tools/cli)

---

## 4. Components — The Core Building Block

A component controls a **patch of the screen** called a view. Every Angular app has at least one component: the root `AppComponent`.

### Modern Component (v17+ standalone)
```typescript
// user-card.component.ts
import { Component, input, output, signal, computed } from '@angular/core';
import { NgClass } from '@angular/common';

@Component({
  selector: 'app-user-card',        // how you use it in HTML: <app-user-card />
  standalone: true,                  // ✅ no NgModule needed (modern default)
  imports: [NgClass],                // import what this template needs
  templateUrl: './user-card.component.html',
  styleUrl: './user-card.component.css',
  // OR inline:
  // template: `<h2>{{ user().name }}</h2>`,
  // styles: `h2 { color: red; }`,
})
export class UserCardComponent {
  // Inputs, outputs, state — all here
}
```

### Old Way (module-based, pre-v14)
```typescript
// ❌ OLD: Required NgModule declaration
@NgModule({
  declarations: [UserCardComponent], // had to declare every component
  imports: [CommonModule],
})
export class UsersModule {}
```

### Component Metadata Options
```typescript
@Component({
  selector: 'app-user-card',        // CSS selector to use it
  standalone: true,                  // doesn't need NgModule (v14+)
  imports: [],                       // dependencies this component needs
  template: `...`,                   // inline HTML
  templateUrl: './comp.html',        // external HTML file
  styles: `...`,                     // inline CSS (v17+ singular styleUrl)
  styleUrl: './comp.css',            // single external CSS file (v17+)
  styleUrls: ['./a.css', './b.css'], // multiple CSS files (old way)
  changeDetection: ChangeDetectionStrategy.OnPush, // performance
  host: { class: 'block w-full' },   // attributes/classes on host element
  encapsulation: ViewEncapsulation.Emulated, // default CSS scoping
})
```

### 📖 Resources
- [angular.dev — Components](https://angular.dev/essentials/components)
- [angular.dev — In-depth Components guide](https://angular.dev/guide/components)

---

## 5. Signals — The New Reactivity Model

> **This is the biggest change in modern Angular. Understand it deeply.**

Signals are Angular's new primitive for reactive state. They replace the need for Zone.js and `ChangeDetectorRef` in most cases.

### The Old Way — Zone.js + Properties
```typescript
// ❌ OLD: Plain class properties — Angular had to "guess" what changed
// using Zone.js to monkey-patch all browser APIs and check everything
export class CounterComponent {
  count = 0;               // just a number — Angular checks this on every tick
  doubled = 0;             // had to manually keep in sync

  increment() {
    this.count++;
    this.doubled = this.count * 2; // manually update derived state
  }
}
```

### The New Way — Signals (v16+, stable v18+)
```typescript
// ✅ NEW: Signals — Angular knows exactly what changed and what depends on it
import { signal, computed, effect } from '@angular/core';

export class CounterComponent {
  // signal() — writable state
  count = signal(0);

  // computed() — derived state, auto-updates when count changes
  doubled = computed(() => this.count() * 2);
  isEven = computed(() => this.count() % 2 === 0);

  increment() {
    this.count.update(v => v + 1); // or: this.count.set(this.count() + 1)
  }

  reset() {
    this.count.set(0);
  }
}
```

### Reading Signals — Call Them Like Functions
```typescript
// A signal is a getter function — you call it to read its value
const name = signal('Ahmed');
console.log(name());        // "Ahmed"  ← notice the ()

name.set('Ali');            // set a new value
name.update(n => n + '!'); // update based on previous value
```

### Computed Signals — Lazy & Memoized
```typescript
const firstName = signal('Ahmed');
const lastName = signal('Ali');

// Computed — only recalculates when dependencies change
const fullName = computed(() => `${firstName()} ${lastName()}`);

console.log(fullName()); // "Ahmed Ali"
firstName.set('Sara');
console.log(fullName()); // "Sara Ali" — auto-updated
```

### `effect()` — Run Side Effects on Signal Changes
```typescript
import { effect } from '@angular/core';

// Runs whenever any signal read inside changes
effect(() => {
  console.log(`Count is now: ${this.count()}`);
  // Angular tracks that this.count() was read
  // Re-runs whenever count changes
});

// ⚠️ Important: Read signals BEFORE any async boundary
effect(async () => {
  const currentCount = this.count(); // ✅ read before await
  const data = await fetchData();    // async boundary
  console.log(currentCount, data);   // currentCount is tracked
});
```

### `linkedSignal()` — Writable Computed (v19+)
```typescript
import { linkedSignal } from '@angular/core';

// A writable signal whose default is derived from another signal
// but can be overridden manually
const options = signal(['apple', 'banana', 'cherry']);
const selectedOption = linkedSignal(() => options()[0]); // defaults to first option

selectedOption(); // 'apple'
options.set(['grape', 'mango']); // linkedSignal resets to 'grape'
selectedOption(); // 'grape'
selectedOption.set('mango'); // manually override — stays 'mango' until options changes
```

### `resource()` — Async Data in Signals (v19+)
```typescript
import { resource, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { firstValueFrom } from 'rxjs';

export class UsersComponent {
  private http = inject(HttpClient);
  page = signal(1);

  // resource() ties async data loading into the signal system
  usersResource = resource({
    params: () => ({ page: this.page() }), // reactive params
    loader: async ({ params }) => {
      return firstValueFrom(
        this.http.get<User[]>(`/api/users?page=${params.page}`)
      );
    }
  });

  // Access via:
  // usersResource.value()   → the data (or undefined)
  // usersResource.isLoading() → boolean
  // usersResource.error()   → error if any
  // usersResource.reload()  → manually trigger reload
}
```

### Encapsulating Signal State — The Pattern
```typescript
// Best practice: make writable signal private, expose readonly
@Injectable({ providedIn: 'root' })
export class CounterService {
  private _count = signal(0);
  readonly count = this._count.asReadonly(); // public can read, not write

  increment() { this._count.update(v => v + 1); }
  reset()     { this._count.set(0); }
}
```

### `untracked()` — Read Without Creating Dependency
```typescript
import { untracked } from '@angular/core';

effect(() => {
  const user = this.currentUser(); // tracked — reruns when user changes
  const log = untracked(() => this.auditLog()); // NOT tracked — won't trigger rerun
  console.log(user, log);
});
```

### 📖 Resources
- [angular.dev — Signals (in-depth)](https://angular.dev/guide/signals)
- [angular.dev — linkedSignal](https://angular.dev/guide/signals/linked-signal)
- [angular.dev — resource API](https://angular.dev/guide/signals/resource)
- [angular.dev — Effects](https://angular.dev/guide/signals/effect)

---

## 6. Templates & Data Binding

Angular templates are HTML files with extra superpowers: bindings, control flow, and component usage.

### The 4 Types of Binding

```html
<!-- 1. Interpolation — render a value as text -->
<h1>Hello, {{ userName() }}</h1>
<p>Items: {{ items().length }}</p>

<!-- 2. Property binding — set a DOM property [ ] -->
<button [disabled]="!isValid()">Submit</button>
<img [src]="user().avatarUrl" [alt]="user().name">
<input [value]="searchQuery()">

<!-- 3. Event binding — listen to DOM events ( ) -->
<button (click)="saveUser()">Save</button>
<input (input)="onInput($event)" (keydown.enter)="submit()">
<form (submit)="onSubmit($event)">

<!-- 4. Two-way binding — [(ngModel)] or signal-based -->
<!-- Old way (still works with FormsModule): -->
<input [(ngModel)]="userName">

<!-- New way with signals: use property + event -->
<input [value]="userName()" (input)="userName.set($event.target.value)">
```

### Attribute vs Property Binding
```html
<!-- Property binding — sets the DOM property (boolean, object, etc.) -->
<button [disabled]="isLoading()">Save</button>

<!-- Attribute binding — for HTML attributes that have no DOM property -->
<td [attr.colspan]="colSpan()">...</td>
<div [attr.aria-label]="label()">...</div>

<!-- Class binding -->
<div [class.active]="isActive()">...</div>
<div [class]="{ active: isActive(), disabled: isDisabled() }">...</div>

<!-- Style binding -->
<div [style.color]="textColor()">...</div>
<div [style]="{ 'font-size': fontSize() + 'px' }">...</div>
```

### Template Reference Variables
```html
<!-- Reference a DOM element or component instance -->
<input #emailInput type="email">
<button (click)="submit(emailInput.value)">Submit</button>

<!-- Reference a component instance -->
<app-user-form #userForm></app-user-form>
<button (click)="userForm.reset()">Reset</button>
```

### 📖 Resources
- [angular.dev — Templates (in-depth)](https://angular.dev/guide/templates)
- [angular.dev — Binding](https://angular.dev/guide/templates/binding)

---

## 7. New Built-in Control Flow

> **Major change in Angular v17.** The old structural directives (`*ngIf`, `*ngFor`, `*ngSwitch`) still work but are now legacy. The new `@if / @for / @switch` blocks are built into the template engine and are faster, simpler, and don't require importing `CommonModule`.

### Conditional Rendering

```html
<!-- ❌ OLD: Required importing CommonModule or NgIf -->
<div *ngIf="isLoggedIn; else loginTpl">
  <h1>Welcome back!</h1>
</div>
<ng-template #loginTpl>
  <p>Please log in</p>
</ng-template>

<!-- ✅ NEW: @if — no imports needed, reads like real code -->
@if (isLoggedIn()) {
  <h1>Welcome back!</h1>
} @else if (isPending()) {
  <p>Verifying...</p>
} @else {
  <p>Please log in</p>
}
```

### List Rendering

```html
<!-- ❌ OLD: *ngFor required trackBy function separately -->
<ul>
  <li *ngFor="let user of users; trackBy: trackByUserId; let i = index">
    {{ i }}: {{ user.name }}
  </li>
</ul>

<!-- ✅ NEW: @for — track is mandatory (better performance by default) -->
<ul>
  @for (user of users(); track user.id) {
    <li>{{ user.name }}</li>
  } @empty {
    <!-- 🆕 @empty block — no equivalent in old syntax -->
    <li>No users found.</li>
  }
</ul>

<!-- @for context variables -->
@for (item of items(); track item.id; let i = $index, isFirst = $first, isLast = $last, isEven = $even) {
  <div [class.first]="isFirst" [class.last]="isLast">
    {{ i }}: {{ item.name }}
  </div>
}
```

### Switch

```html
<!-- ❌ OLD: ngSwitch was verbose and needed [ngSwitchCase] on each child -->
<div [ngSwitch]="status">
  <span *ngSwitchCase="'loading'">Loading...</span>
  <span *ngSwitchCase="'error'">Error!</span>
  <span *ngSwitchDefault>Ready</span>
</div>

<!-- ✅ NEW: @switch -->
@switch (status()) {
  @case ('loading') { <app-spinner /> }
  @case ('error')   { <app-error-message /> }
  @default          { <app-content /> }
}
```

### Deferrable Views — Lazy Loading in Templates

```html
<!-- 🆕 NEW in Angular v17: @defer — lazy load components -->

<!-- Load HeavyChart only when it enters the viewport -->
@defer (on viewport) {
  <app-heavy-chart />
} @placeholder {
  <div class="chart-placeholder">Chart loading...</div>
} @loading (minimum 500ms) {
  <app-spinner />
} @error {
  <p>Failed to load chart</p>
}

<!-- Other defer triggers -->
@defer (on idle)           { ... }  <!-- when browser is idle -->
@defer (on interaction)    { ... }  <!-- when user interacts with placeholder -->
@defer (on timer(2s))      { ... }  <!-- after 2 seconds -->
@defer (when isVisible())  { ... }  <!-- when signal condition is true -->
@defer (prefetch on idle)  { ... }  <!-- prefetch while idle but show on demand -->
```

### 📖 Resources
- [angular.dev — Control flow](https://angular.dev/guide/templates/control-flow)
- [angular.dev — Deferrable views](https://angular.dev/guide/defer)

---

## 8. Directives

Directives are classes that add behavior to elements in the DOM. There are three types.

### 1. Structural Directives (change DOM structure)
The `@if`, `@for`, `@switch` blocks replaced most built-in structural directives. But you can still write custom ones.

```typescript
// Custom structural directive (still valid pattern)
import { Directive, TemplateRef, ViewContainerRef, input, effect } from '@angular/core';

@Directive({ selector: '[appShowIf]', standalone: true })
export class ShowIfDirective {
  appShowIf = input<boolean>();

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {
    effect(() => {
      if (this.appShowIf()) {
        this.viewContainer.createEmbeddedView(this.templateRef);
      } else {
        this.viewContainer.clear();
      }
    });
  }
}
```

### 2. Attribute Directives (change appearance/behavior)
```typescript
// Classic highlight directive
import { Directive, ElementRef, HostListener, input } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true,
})
export class HighlightDirective {
  color = input('yellow', { alias: 'appHighlight' }); // alias lets you write [appHighlight]="'red'"

  constructor(private el: ElementRef) {}

  @HostListener('mouseenter') onMouseEnter() {
    this.el.nativeElement.style.backgroundColor = this.color();
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.el.nativeElement.style.backgroundColor = '';
  }
}

// Usage: <p [appHighlight]="'lightblue'">Hover me</p>
```

### 3. Component Directives
Components are directives with a template. They're the most common kind.

### Built-in Attribute Directives (still used)
```html
<!-- NgClass — dynamic class binding -->
<div [ngClass]="{ 'active': isActive(), 'disabled': isDisabled() }"></div>

<!-- NgStyle — dynamic style binding -->
<div [ngStyle]="{ 'color': textColor(), 'font-size': fontSize() + 'px' }"></div>
```

### 📖 Resources
- [angular.dev — Directives](https://angular.dev/guide/directives)
- [angular.dev — Attribute directives](https://angular.dev/guide/directives/attribute-directives)

---

## 9. Pipes

Pipes transform data **in templates** without changing the underlying data. They're pure by default (only re-run when input changes).

### Built-in Pipes
```html
<!-- Import from @angular/common, or they're auto-available in standalone -->
{{ birthday | date:'longDate' }}         <!-- April 6, 2025 -->
{{ price | currency:'USD':'symbol':'1.2-2' }}  <!-- $1,234.56 -->
{{ ratio | percent:'1.1-2' }}            <!-- 75.5% -->
{{ longText | slice:0:100 }}             <!-- first 100 chars -->
{{ user | json }}                        <!-- debugging: stringify to JSON -->
{{ name | uppercase }}                   <!-- AHMED -->
{{ name | lowercase }}                   <!-- ahmed -->
{{ 'hello world' | titlecase }}          <!-- Hello World -->
{{ items | keyvalue }}                   <!-- iterate object as key-value pairs -->
```

### Custom Pipe
```typescript
// truncate.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate',
  standalone: true,
  pure: true, // ← default — only recalculates when input reference changes
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, maxLength: number = 50, suffix: string = '...'): string {
    if (!value || value.length <= maxLength) return value;
    return value.substring(0, maxLength) + suffix;
  }
}

// Usage: {{ description | truncate:100:'…' }}
```

### Pure vs Impure Pipes
```typescript
// Pure (default) — only runs when input reference changes (fast)
@Pipe({ name: 'double', pure: true })

// Impure — runs on every change detection cycle (slow, use sparingly)
// Use case: filtering arrays that mutate in place
@Pipe({ name: 'filterActive', pure: false })
```

### 📖 Resources
- [angular.dev — Pipes](https://angular.dev/guide/templates/pipes)

---

## 10. Dependency Injection

DI is how Angular supplies objects (services, config) to components and other services. It's one of Angular's most powerful features.

### The Old Way — Constructor Injection
```typescript
// ❌ OLD (still works, but verbose)
export class UserComponent {
  constructor(
    private userService: UserService,
    private authService: AuthService,
    private router: Router,
  ) {}
}
```

### The New Way — `inject()` Function (v14+)
```typescript
// ✅ NEW: inject() — more composable, works in more contexts
import { inject } from '@angular/core';

export class UserComponent {
  private userService = inject(UserService);
  private authService = inject(AuthService);
  private router = inject(Router);
  // Can be used in field initializers, constructor is optional
}
```

### Why `inject()` is Better
```typescript
// inject() enables shared logic in plain functions (not just classes)
// This is called an "injection function" — you can't do this with constructor injection

function useAuthGuard() {
  const authService = inject(AuthService);
  const router = inject(Router);
  return () => {
    if (!authService.isLoggedIn()) {
      router.navigate(['/login']);
      return false;
    }
    return true;
  };
}
```

### Injection Tokens (for non-class dependencies)
```typescript
import { InjectionToken, inject } from '@angular/core';

// Use when you need to inject a primitive or config object
export const API_URL = new InjectionToken<string>('API_URL');
export const APP_CONFIG = new InjectionToken<AppConfig>('APP_CONFIG');

// Provide it
providers: [
  { provide: API_URL, useValue: 'https://api.example.com' },
  { provide: APP_CONFIG, useFactory: () => ({ theme: 'dark', language: 'en' }) },
]

// Inject it
const apiUrl = inject(API_URL);
```

### Provider Types
```typescript
providers: [
  // useClass — create an instance of a class
  { provide: LoggingService, useClass: ConsoleLoggingService },

  // useExisting — alias one token to another
  { provide: AbstractLogger, useExisting: LoggingService },

  // useValue — inject a literal value
  { provide: API_URL, useValue: 'https://api.example.com' },

  // useFactory — create dynamically
  { provide: HttpClient, useFactory: (backend: HttpBackend) =>
    new HttpClient(backend), deps: [HttpBackend] },
]
```

### Injection Scope (Hierarchical DI)
```
Root injector (providedIn: 'root') — singleton, app-wide
  └── Component injector (providers: [] in @Component) — one instance per component
        └── Child component injector
```

```typescript
// Singleton — shared across the whole app
@Injectable({ providedIn: 'root' })
export class AuthService {}

// Per-component instance — new instance per component
@Component({
  providers: [CartService] // each instance of this component gets its own CartService
})
export class CheckoutComponent {}
```

### 📖 Resources
- [angular.dev — Dependency Injection](https://angular.dev/guide/di)
- [angular.dev — inject() function](https://angular.dev/api/core/inject)

---

## 11. Services

Services are where you put **business logic, shared state, and HTTP calls** — anything that shouldn't live in a component.

```typescript
// auth.service.ts
import { Injectable, inject, signal, computed } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { tap } from 'rxjs/operators';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);

  // State as signals — any component can react to this
  private _currentUser = signal<User | null>(null);
  readonly currentUser = this._currentUser.asReadonly();
  readonly isLoggedIn = computed(() => this._currentUser() !== null);
  readonly isAdmin = computed(() => this._currentUser()?.role === 'admin');

  login(credentials: LoginDto) {
    return this.http.post<AuthResponse>('/api/auth/login', credentials).pipe(
      tap(res => {
        localStorage.setItem('token', res.token);
        this._currentUser.set(res.user);
      })
    );
  }

  logout() {
    localStorage.removeItem('token');
    this._currentUser.set(null);
    this.router.navigate(['/login']);
  }

  loadCurrentUser() {
    return this.http.get<User>('/api/auth/me').pipe(
      tap(user => this._currentUser.set(user))
    );
  }
}
```

---

## 12. Routing

### Setup (Standalone, modern)
```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  {
    path: 'users',
    // ✅ Lazy loading — split code, load on demand
    loadComponent: () => import('./users/users.component').then(m => m.UsersComponent),
    children: [
      { path: '', component: UserListComponent },
      { path: ':id', component: UserDetailComponent },
    ]
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes').then(m => m.adminRoutes),
    canActivate: [authGuard],    // route guard
  },
  { path: '**', component: NotFoundComponent }, // wildcard — must be last
];

// app.config.ts (standalone bootstrap)
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withComponentInputBinding } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding()), // ← enables route params as inputs!
  ],
};
```

### Router in Templates
```html
<!-- Router outlet — where the matched component renders -->
<router-outlet></router-outlet>

<!-- Navigation links -->
<a routerLink="/home" routerLinkActive="active-link">Home</a>
<a [routerLink]="['/users', userId]">User Profile</a>

<!-- Active link options -->
<a routerLink="/users" [routerLinkActiveOptions]="{ exact: true }" routerLinkActive="active">
  Users
</a>
```

### Navigation in Component
```typescript
import { Router, ActivatedRoute } from '@angular/router';

export class UserDetailComponent {
  private router = inject(Router);
  private route = inject(ActivatedRoute);

  // Old way — subscribe to params
  ngOnInit() {
    this.route.params.subscribe(params => {
      this.loadUser(params['id']);
    });
  }

  // New way — withComponentInputBinding() makes route params into @Input()
  // (when enabled in provideRouter)
  id = input<string>(); // auto-populated from route :id param!

  navigateToList() {
    this.router.navigate(['/users']);
  }

  navigateWithState() {
    this.router.navigate(['/users'], {
      queryParams: { page: 2, filter: 'active' },
      fragment: 'top',
    });
  }
}
```

### Route Guards
```typescript
// ✅ NEW: Functional guards (v15+) — no class needed
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isLoggedIn()) return true;
  return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
};

// ❌ OLD: Class-based guard (still works but deprecated pattern)
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  canActivate(): boolean { /* ... */ }
}
```

### Route Resolvers
```typescript
// Pre-fetch data before the component activates
export const userResolver: ResolveFn<User> = (route) => {
  return inject(UserService).getUser(route.paramMap.get('id')!);
};

// In routes:
{ path: ':id', component: UserDetailComponent, resolve: { user: userResolver } }
```

### 📖 Resources
- [angular.dev — Routing](https://angular.dev/guide/routing)
- [angular.dev — Route guards](https://angular.dev/guide/routing/router-guards)

---

## 13. Forms

Angular has two approaches to forms. You need to know both.

### Template-Driven Forms (simple, less control)
```typescript
// Requires FormsModule in imports
import { FormsModule } from '@angular/forms';

@Component({
  imports: [FormsModule],
  template: `
    <form (ngSubmit)="onSubmit()" #f="ngForm">
      <input name="email" [(ngModel)]="email" required email #emailField="ngModel">
      @if (emailField.invalid && emailField.touched) {
        <span>Invalid email</span>
      }
      <button [disabled]="f.invalid">Submit</button>
    </form>
  `
})
export class LoginComponent {
  email = '';
  onSubmit() { console.log(this.email); }
}
```

### Reactive Forms (powerful, testable, recommended for complex forms)
```typescript
import { ReactiveFormsModule, FormBuilder, Validators, FormGroup } from '@angular/forms';
import { inject } from '@angular/core';

@Component({ imports: [ReactiveFormsModule] })
export class RegisterComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    name:     ['', [Validators.required, Validators.minLength(2)]],
    email:    ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]],
    confirmPassword: [''],
  }, { validators: passwordMatchValidator }); // group-level validator

  // Getters for template convenience
  get email() { return this.form.get('email')!; }
  get name()  { return this.form.get('name')!; }

  onSubmit() {
    if (this.form.invalid) return;
    console.log(this.form.value); // typed values
  }
}

// Custom validator
function passwordMatchValidator(group: FormGroup) {
  const pw = group.get('password')?.value;
  const confirm = group.get('confirmPassword')?.value;
  return pw === confirm ? null : { mismatch: true };
}
```

```html
<!-- Reactive forms template -->
<form [formGroup]="form" (ngSubmit)="onSubmit()">
  <input formControlName="email" type="email">
  @if (email.hasError('required') && email.touched) {
    <span>Email is required</span>
  }
  @if (email.hasError('email') && email.touched) {
    <span>Must be a valid email</span>
  }
  <button [disabled]="form.invalid">Register</button>
</form>
```

### Signal Forms — NEW (v19+, experimental → stable in v21)
```typescript
// The newest approach — signals + forms unified
import { signalForm, field, Validators as V } from '@angular/forms'; // signal-based API

// Note: This API is still evolving — check angular.dev/essentials/signal-forms for latest
```

### Comparison
| | Template-Driven | Reactive | Signal Forms |
|---|---|---|---|
| Setup | Simple, in HTML | More code, in TS | Signal-based |
| Testing | Harder (async) | Easy (sync) | Easy |
| Dynamic forms | Harder | Easy | Easy |
| Validation | HTML attributes | Validator functions | Validator functions |
| State access | Via `#f="ngForm"` | FormControl/FormGroup | Signals |
| Best for | Simple forms | Complex forms | Modern signal apps |

### 📖 Resources
- [angular.dev — Forms](https://angular.dev/guide/forms)
- [angular.dev — Reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [angular.dev — Signal forms](https://angular.dev/essentials/signal-forms)

---

## 14. HTTP Client

### Setup
```typescript
// app.config.ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, loggingInterceptor])
    ),
  ],
};
```

### Basic Usage
```typescript
import { HttpClient, HttpParams } from '@angular/common/http';
import { inject } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private apiUrl = inject(API_URL);

  getUsers(page = 1, limit = 20) {
    const params = new HttpParams()
      .set('page', page)
      .set('limit', limit);
    return this.http.get<User[]>(`${this.apiUrl}/users`, { params });
  }

  getUserById(id: string) {
    return this.http.get<User>(`${this.apiUrl}/users/${id}`);
  }

  createUser(data: CreateUserDto) {
    return this.http.post<User>(`${this.apiUrl}/users`, data);
  }

  updateUser(id: string, data: Partial<User>) {
    return this.http.patch<User>(`${this.apiUrl}/users/${id}`, data);
  }

  deleteUser(id: string) {
    return this.http.delete<void>(`${this.apiUrl}/users/${id}`);
  }

  uploadAvatar(id: string, file: File) {
    const formData = new FormData();
    formData.append('avatar', file);
    return this.http.post(`${this.apiUrl}/users/${id}/avatar`, formData, {
      reportProgress: true,
      observe: 'events', // see upload progress
    });
  }
}
```

### HTTP Interceptors (modern functional style)
```typescript
// auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';

// ✅ NEW: Functional interceptor (v15+)
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('token');
  if (!token) return next(req);

  const authReq = req.clone({
    setHeaders: { Authorization: `Bearer ${token}` }
  });
  return next(authReq);
};

// Error handling interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((err: HttpErrorResponse) => {
      if (err.status === 401) inject(AuthService).logout();
      if (err.status === 403) inject(Router).navigate(['/forbidden']);
      return throwError(() => err);
    })
  );
};

// ❌ OLD: Class-based interceptor (still works but verbose)
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler) {
    /* ... */
  }
}
```

### Consuming HTTP in Components — Signals Way
```typescript
// Using resource() for clean async data in signals (v19+)
export class UsersComponent {
  private userService = inject(UserService);

  usersResource = resource({
    loader: () => firstValueFrom(this.userService.getUsers()),
  });
}
```

```html
@if (usersResource.isLoading()) {
  <app-spinner />
} @else if (usersResource.error()) {
  <app-error [error]="usersResource.error()" />
} @else {
  @for (user of usersResource.value(); track user.id) {
    <app-user-card [user]="user" />
  }
}
```

### 📖 Resources
- [angular.dev — HTTP Client](https://angular.dev/guide/http)
- [angular.dev — HTTP Interceptors](https://angular.dev/guide/http/interceptors)

---

## 15. Component Lifecycle Hooks

Lifecycle hooks let you run code at specific moments in a component's life.

### The Full Lifecycle (in order)

```typescript
import {
  Component, OnInit, OnChanges, DoCheck,
  AfterContentInit, AfterContentChecked,
  AfterViewInit, AfterViewChecked,
  OnDestroy, SimpleChanges,
  input, signal, effect
} from '@angular/core';

@Component({ /* ... */ })
export class MyComponent implements OnInit, OnDestroy {
  // 1. constructor — DI happens here, no template access
  constructor() {
    // inject() calls go here or as field initializers
  }

  // 2. ngOnChanges — called BEFORE ngOnInit, then on every @Input() change
  ngOnChanges(changes: SimpleChanges) {
    if (changes['userId']) {
      console.log('Previous:', changes['userId'].previousValue);
      console.log('Current:', changes['userId'].currentValue);
    }
  }

  // 3. ngOnInit — called once after first ngOnChanges
  // ✅ Put HTTP calls and initialization logic here
  ngOnInit() {
    this.loadData();
  }

  // 4. ngDoCheck — called on every change detection run (use sparingly!)
  ngDoCheck() { /* custom change detection */ }

  // 5. ngAfterContentInit — after <ng-content> is initialized
  ngAfterContentInit() { /* projected content is ready */ }

  // 6. ngAfterContentChecked — after projected content is checked
  ngAfterContentChecked() { }

  // 7. ngAfterViewInit — after component's view (and child views) initialized
  // ✅ Access @ViewChild here (DOM is ready)
  ngAfterViewInit() { /* this.myInput.nativeElement.focus(); */ }

  // 8. ngAfterViewChecked — after view is checked
  ngAfterViewChecked() { }

  // 9. ngOnDestroy — just before component is destroyed
  // ✅ Clean up subscriptions, event listeners, timers
  ngOnDestroy() {
    this.subscription?.unsubscribe();
    clearInterval(this.timer);
  }
}
```

### Signals Replace Most Lifecycle Needs

```typescript
// With signals, you often don't need ngOnChanges at all
// Use input signals + computed/effect instead

export class UserCardComponent {
  userId = input<string>();  // signal — reactive to changes

  // Automatically reacts when userId changes — no ngOnChanges needed
  private userService = inject(UserService);

  userResource = resource({
    params: () => ({ id: this.userId() }),
    loader: ({ params }) => firstValueFrom(this.userService.getUser(params.id!)),
  });
}
```

### `afterRender` and `afterNextRender` (v17+)
```typescript
import { afterRender, afterNextRender } from '@angular/core';

export class ChartComponent {
  constructor() {
    // Run after every render
    afterRender(() => {
      // Safe to access DOM / third-party libraries here
    });

    // Run only after the next render (once)
    afterNextRender(() => {
      this.initChart();
    });
  }
}
```

### 📖 Resources
- [angular.dev — Component lifecycle](https://angular.dev/guide/components/lifecycle)

---

## 16. Change Detection

Change detection is how Angular knows when to update the DOM. This is deeply connected to Signals.

### Zone.js (the old approach)
```
Zone.js monkey-patches browser APIs (setTimeout, fetch, event listeners, Promise).
When any async operation completes, it triggers Angular to check ALL components
from the root down — even if nothing changed.
This is called "dirty checking" and it's expensive at scale.
```

### OnPush Strategy (optimization)
```typescript
// ❌ Default — check this component on every cycle
@Component({ changeDetection: ChangeDetectionStrategy.Default })

// ✅ OnPush — only check when:
// - @Input() reference changes
// - An event fires from this component or child
// - A signal in the template changes
// - You manually trigger with ChangeDetectorRef.markForCheck()
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserCardComponent {
  user = input<User>(); // ✅ signals as inputs work perfectly with OnPush
}
```

### Signals + OnPush = Fine-Grained Reactivity
```typescript
// With signals, Angular can track EXACTLY which component depends on which value
// and only update those — no more full tree traversal

@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ count() }}</p>`
})
export class CounterComponent {
  count = signal(0); // When count changes, only THIS component updates
}
```

### Zoneless Angular (experimental → stable in v19+)
```typescript
// You can now run Angular without Zone.js entirely!
// Uses signals for all reactivity
export const appConfig: ApplicationConfig = {
  providers: [
    provideExperimentalZonelessChangeDetection(), // v18+
  ],
};
// This makes your app smaller and faster — no Zone.js bundle
```

### 📖 Resources
- [angular.dev — Change detection](https://angular.dev/guide/change-detection)
- [angular.dev — Skipping subtrees (OnPush)](https://angular.dev/best-practices/skipping-subtrees)

---

## 17. Component Communication

### Parent → Child: `input()` (Signals-based, v17.1+)

```typescript
// ✅ NEW: input() function — signal-based input
import { input } from '@angular/core';

export class UserCardComponent {
  // Required input — TypeScript enforces it must be provided
  user = input.required<User>();

  // Optional input with default
  isHighlighted = input(false);

  // Input with transform
  userId = input<string, number>(0, { transform: (v: number) => v.toString() });

  // Alias — parent uses [userId] but class uses id
  id = input<string>('', { alias: 'userId' });
}

// ❌ OLD: @Input() decorator
export class UserCardOld {
  @Input({ required: true }) user!: User;
  @Input() isHighlighted = false;
}
```

```html
<!-- Parent template -->
<app-user-card [user]="selectedUser()" [isHighlighted]="true" />
```

### Child → Parent: `output()` (v17.3+)

```typescript
// ✅ NEW: output() function
import { output } from '@angular/core';

export class UserCardComponent {
  user = input.required<User>();
  userSelected = output<User>();         // emit User when clicked
  userDeleted = output<string>();        // emit user id

  onCardClick() {
    this.userSelected.emit(this.user());
  }
}

// ❌ OLD: @Output() + EventEmitter
export class UserCardOld {
  @Output() userSelected = new EventEmitter<User>();
}
```

```html
<!-- Parent listens -->
<app-user-card
  [user]="user"
  (userSelected)="onUserSelected($event)"
  (userDeleted)="onUserDeleted($event)"
/>
```

### Sibling / Any-to-Any: Shared Service with Signals
```typescript
@Injectable({ providedIn: 'root' })
export class SelectionService {
  private _selected = signal<User | null>(null);
  readonly selected = this._selected.asReadonly();

  select(user: User) { this._selected.set(user); }
  clear() { this._selected.set(null); }
}

// Any component can inject and react to this
export class SidebarComponent {
  selection = inject(SelectionService);
  // In template: {{ selection.selected()?.name }}
}
```

### `viewChild()` and `contentChild()` — Signal-based Queries (v17.2+)
```typescript
import { viewChild, contentChild, ElementRef } from '@angular/core';

export class FormComponent {
  // ✅ NEW: viewChild() — signal-based
  emailInput = viewChild<ElementRef>('emailInput'); // refs a #emailInput in template
  emailInput = viewChild.required<ElementRef>('emailInput'); // throws if not found

  // ❌ OLD:
  @ViewChild('emailInput') emailInput!: ElementRef;

  focusEmail() {
    this.emailInput()?.nativeElement.focus(); // signal — call it with ()
  }
}
```

---

## 18. NgModules — The Old Way (and why it's now optional)

Before Angular 14, **every component, pipe, and directive had to be declared inside an NgModule**.

```typescript
// ❌ OLD: Required for every Angular app
@NgModule({
  declarations: [
    AppComponent,
    UserListComponent,
    UserCardComponent,
    TruncatePipe,
    HighlightDirective,
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModule.forRoot(routes),
  ],
  providers: [UserService, AuthService],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### Why NgModules Existed
- Group related components into cohesive features
- Control what's publicly visible (exports) vs private (declarations only)
- Provide services at a feature level

### Why They're Now Optional
NgModules had a major DX problem: **you couldn't use a component without declaring it in a module**. This caused circular imports, forced you to understand the module system before writing your first component, and made lazy loading complex.

Standalone components solve all of this more elegantly.

### 📖 Resources
- [angular.dev — NgModules](https://angular.dev/guide/ngmodules/overview)

---

## 19. Standalone Components — The New Way

Standalone is the **default in Angular v17+**. Every new project uses standalone.

```typescript
// ✅ NEW: Standalone component — imports what IT needs directly
@Component({
  selector: 'app-user-list',
  standalone: true, // (default in v17+, you can omit this)
  imports: [
    UserCardComponent,    // other standalone components
    AsyncPipe,            // pipes from @angular/common
    RouterLink,           // router directives
    ReactiveFormsModule,  // form modules still exist
  ],
  template: `...`
})
export class UserListComponent {}
```

### Bootstrapping (no AppModule)
```typescript
// main.ts — standalone app bootstrap
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch(console.error);

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimations(),
  ],
};
```

### Migrating from NgModules
```bash
# Angular provides a schematic to migrate automatically
ng generate @angular/core:standalone
```

### Old Way vs New Way Summary

| Concern | Old (NgModule) | New (Standalone) |
|---|---|---|
| Component registration | Declare in NgModule | `standalone: true`, imports directly |
| App bootstrap | `AppModule` + `bootstrapModule()` | `bootstrapApplication()` + `appConfig` |
| Service provision | `providers` in NgModule | `providedIn: 'root'` or `app.config.ts` |
| Lazy loading | `loadChildren: () => import('./mod').then(m => m.Mod)` | `loadComponent:` or `loadChildren:` with routes array |
| HTTP setup | `HttpClientModule` in imports | `provideHttpClient()` in appConfig |

---

## 20. Server-Side Rendering & Hydration

### SSR with Angular Universal / @angular/ssr
```bash
# Add SSR to existing project
ng add @angular/ssr
```

```typescript
// app.config.server.ts — server-specific config
import { provideServerRendering } from '@angular/platform-server';

export const serverConfig: ApplicationConfig = {
  providers: [provideServerRendering()]
};
```

### Hydration (v16+, stable v17+)
Hydration means Angular reuses the server-rendered HTML instead of destroying and re-rendering it on the client. This eliminates the "flash of content" (FOUC) and makes initial paint faster.

```typescript
// app.config.ts — enable hydration
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration(), // ✅ enable hydration
  ],
};
```

### Deferrable Views + SSR
`@defer` blocks are automatically skipped during SSR and loaded on the client, making them perfect for non-critical interactive content.

### 📖 Resources
- [angular.dev — Server-side rendering](https://angular.dev/guide/ssr)
- [angular.dev — Hydration](https://angular.dev/guide/hydration)

---

## 21. RxJS in Angular

Angular was built with RxJS at its core. Even with signals, RxJS is still widely used for HTTP, routing events, and complex async flows.

### Essential RxJS Operators for Angular

```typescript
import { map, filter, switchMap, mergeMap, exhaustMap,
         debounceTime, distinctUntilChanged, catchError,
         takeUntilDestroyed, tap, shareReplay, of, from } from 'rxjs';

// switchMap — cancel previous, start new (search, route data)
searchQuery$.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => this.http.get<User[]>(`/api/users?q=${query}`)),
).subscribe(results => this.results.set(results));

// mergeMap — run all in parallel, keep all results
// exhaustMap — ignore new while previous is pending (prevent double submit)
submitBtn$.pipe(
  exhaustMap(() => this.http.post('/api/submit', this.form.value))
).subscribe();

// shareReplay — cache the last N emissions (great for data that multiple components need)
readonly users$ = this.http.get<User[]>('/api/users').pipe(
  shareReplay(1) // cache the response, share with all subscribers
);
```

### Converting Between Signals and RxJS
```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class SearchComponent {
  searchQuery = signal('');

  // Convert signal → Observable
  searchQuery$ = toObservable(this.searchQuery);

  // Convert Observable → Signal
  results = toSignal(
    this.searchQuery$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(q => this.userService.search(q)),
    ),
    { initialValue: [] }
  );
}
```

### `takeUntilDestroyed` — Auto Unsubscribe (v16+)
```typescript
// ✅ NEW: No need for ngOnDestroy or Subject tricks
export class UserListComponent {
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.userService.getUsers().pipe(
      takeUntilDestroyed(this.destroyRef) // auto-unsubscribes on destroy
    ).subscribe(users => this.users.set(users));
  }
}

// Even simpler — no destroyRef needed when inject context is available:
export class UserListComponent {
  users = toSignal(
    this.userService.getUsers(), // Observable auto-cleaned up with component
    { initialValue: [] }
  );
}
```

### 📖 Resources
- [angular.dev — RxJS interop with signals](https://angular.dev/ecosystem/rxjs-interop)
- [RxJS docs](https://rxjs.dev/)
- [RxJS Marbles (visual)](https://rxmarbles.com/)

---

## 22. Performance Best Practices

```typescript
// 1. Always use OnPush change detection
@Component({ changeDetection: ChangeDetectionStrategy.OnPush })

// 2. Use signals — fine-grained reactivity, no zone.js overhead
count = signal(0);

// 3. Lazy load routes
{ path: 'admin', loadComponent: () => import('./admin.component') }

// 4. Use @defer for heavy components
@defer (on viewport) { <app-heavy-chart /> }

// 5. Use track in @for (mandatory but easy to forget the right key)
@for (item of items(); track item.id) { ... } // ✅ use unique stable ID
@for (item of items(); track $index) { ... }   // ❌ avoid — causes re-renders on reorder

// 6. Use .lean() equivalent — avoid unnecessary computations in templates
// Put computed values in computed() signals, not in template expressions
// ❌ {{ items().filter(x => x.active).length }}  — runs every render cycle
// ✅ activeCount = computed(() => this.items().filter(x => x.active).length);

// 7. Avoid memory leaks — unsubscribe from Observables
// ✅ Use takeUntilDestroyed, toSignal, or async pipe

// 8. Use the async pipe for Observables in templates
// (it handles subscription and unsubscription automatically)
// <div>{{ user$ | async }}</div>

// 9. Use shareReplay for HTTP calls shared across components
users$ = this.http.get('/api/users').pipe(shareReplay(1));

// 10. Enable SSR + Hydration for fast initial load
```

### Bundle Analysis
```bash
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/browser/stats.json
```

### 📖 Resources
- [angular.dev — Performance](https://angular.dev/best-practices/performance)
- [angular.dev — Deferrable views](https://angular.dev/guide/defer)

---

## 23. Testing

### Unit Testing Components
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { signal } from '@angular/core';

describe('UserCardComponent', () => {
  let component: UserCardComponent;
  let fixture: ComponentFixture<UserCardComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserCardComponent], // standalone — just import it
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
    component = fixture.componentInstance;

    // Set required signal inputs
    fixture.componentRef.setInput('user', { id: '1', name: 'Ahmed' });
    fixture.detectChanges();
  });

  it('should display user name', () => {
    const h2 = fixture.nativeElement.querySelector('h2');
    expect(h2.textContent).toContain('Ahmed');
  });

  it('should emit userSelected on click', () => {
    let emittedUser: User | undefined;
    component.userSelected.subscribe(u => emittedUser = u);

    fixture.nativeElement.querySelector('.card').click();
    expect(emittedUser?.name).toBe('Ahmed');
  });
});
```

### Unit Testing Services
```typescript
import { TestBed } from '@angular/core/testing';
import { HttpTestingController, provideHttpClientTesting } from '@angular/common/http/testing';
import { provideHttpClient } from '@angular/common/http';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        UserService,
        provideHttpClient(),
        provideHttpClientTesting(),
      ],
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => httpMock.verify()); // ensure no pending requests

  it('should fetch users', () => {
    service.getUsers().subscribe(users => {
      expect(users.length).toBe(2);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush([{ id: '1', name: 'Ahmed' }, { id: '2', name: 'Sara' }]);
  });
});
```

### 📖 Resources
- [angular.dev — Testing](https://angular.dev/guide/testing)

---

## 24. Common Pitfalls 🪤

```typescript
// ❌ 1. Forgetting to call signals as functions
const name = signal('Ahmed');
console.log(name);   // [Function] — wrong!
console.log(name()); // "Ahmed" — correct

// ❌ 2. Mutating signal values directly (for objects/arrays)
const items = signal<string[]>([]);
items().push('new');       // ❌ mutates the array, signal doesn't know!
items.set([...items(), 'new']); // ✅ create new reference

// ❌ 3. Reading signals after async boundary in effect
effect(async () => {
  const id = userId(); // ✅ read before await
  const data = await fetchUser(id);
  const theme = theme(); // ❌ NOT tracked — after await
});

// ❌ 4. Memory leaks with Observables
ngOnInit() {
  this.userService.getUsers().subscribe(u => this.users.set(u));
  // ❌ Never unsubscribed!
}
// ✅ Fix:
users = toSignal(this.userService.getUsers(), { initialValue: [] });

// ❌ 5. Using ngOnChanges with signal inputs — it doesn't fire for signal inputs!
// For signal inputs, use effect() or computed() to react to changes
// @Input() → ngOnChanges works
// input() signal → use effect(() => { this.userId(); }) instead

// ❌ 6. Wrong track in @for
@for (item of items(); track $index) { /* re-renders on every reorder! */ }
@for (item of items(); track item.id) { /* ✅ stable identity */ }

// ❌ 7. Calling inject() outside injection context
// inject() only works in: constructor, field initializers, injection functions
function badHelper() {
  const service = inject(MyService); // ❌ Error: inject() must be called within an injection context
}
// ✅ Pass service as argument or call inside constructor/effect

// ❌ 8. Putting business logic in ngAfterViewInit when ngOnInit is enough
// Only use AfterViewInit when you actually need DOM / @ViewChild access

// ❌ 9. Subscribing inside subscribe (callback hell with RxJS)
this.route.params.subscribe(params => {
  this.userService.getUser(params['id']).subscribe(user => { // ❌ nested!
    /* ... */
  });
});
// ✅ Use switchMap
this.route.params.pipe(
  switchMap(params => this.userService.getUser(params['id']))
).subscribe(user => { /* ... */ });

// ❌ 10. Modifying @Input()s directly from child (breaks unidirectional data flow)
export class ChildComponent {
  @Input() user!: User;
  updateUser() {
    this.user.name = 'new name'; // ❌ mutates parent's object!
  }
}
// ✅ Emit an output instead
```

---

## 25. Quick-Fire Q&A Cheatsheet

| Question | Answer |
|---|---|
| What is a Signal? | A reactive value wrapper. Read it by calling it as a function `signal()`. Angular tracks where it's read and updates only those places. |
| `signal` vs `computed` vs `effect`? | `signal` = writable state. `computed` = derived read-only state. `effect` = side effect that runs when dependencies change. |
| Standalone vs NgModule? | Standalone components declare their own imports. NgModules are a legacy grouping mechanism — still work but no longer needed. |
| `input()` vs `@Input()`? | `input()` returns a Signal — reactive, composable. `@Input()` is the old decorator approach. Prefer `input()` in new code. |
| `output()` vs `@Output()`? | `output()` is cleaner, no need for `EventEmitter`. Prefer `output()` in new code. |
| `inject()` vs constructor injection? | `inject()` works in field initializers and injection functions, enabling better code reuse. Constructor DI still works. |
| `@if` vs `*ngIf`? | `@if` is built-in control flow (v17+) — no imports, no `ng-template` needed, cleaner syntax. |
| `@for` vs `*ngFor`? | `@for` requires `track` (enforces performance), has built-in `@empty` block, no `trackBy` function needed. |
| `@defer` — what is it? | Lazy-loads a component/template block when a trigger fires (viewport, idle, interaction, timer). First-class lazy loading in templates. |
| What is Change Detection? | Angular's process of comparing current state to previous to know what to update in the DOM. |
| OnPush vs Default? | Default checks every component every cycle. OnPush only checks when inputs change, events fire, or signals update. Prefer OnPush. |
| What is Zone.js? | A library that patches browser APIs to notify Angular when async work completes, triggering change detection. Signals allow Angular to work without it. |
| `switchMap` vs `mergeMap`? | `switchMap` cancels the previous inner observable when a new value arrives. `mergeMap` runs all in parallel. Use `switchMap` for search, `mergeMap` for parallel requests. |
| `toSignal` vs `toObservable`? | `toSignal(obs$)` converts Observable → Signal. `toObservable(sig)` converts Signal → Observable. From `@angular/core/rxjs-interop`. |
| What is hydration? | After SSR, Angular reuses server-rendered HTML instead of destroying and re-rendering it. Enabled with `provideClientHydration()`. |
| Route guard old vs new? | Old: class implementing `CanActivate`. New: `CanActivateFn` — just a function, no class/decorator. |
| What is `providedIn: 'root'`? | Makes a service a singleton available across the whole app via the root injector. No need to add to providers array anywhere. |
| `shareReplay(1)` — why? | Caches the last HTTP response so multiple subscribers don't trigger multiple HTTP calls. Essential for shared data streams. |
| `takeUntilDestroyed` — why? | Automatically unsubscribes an Observable when the injection context (component) is destroyed. Replaces manual `ngOnDestroy` + Subject patterns. |
| Reactive vs Template-driven forms? | Reactive: defined in TS, testable, for complex forms. Template-driven: defined in HTML, simpler, for basic forms. |
| What is `linkedSignal`? | A writable signal whose default value is derived from another signal, but can be overridden. Think of it as "a computed that you can also set manually." |

---

## 🎯 Senior SWE Tips — Angular Interview Edition

1. **Lead with Signals** — If asked about state management or data binding, bring up Signals. They're central to modern Angular and show you're current.
2. **Know old vs new** — Interviewers often use old syntax themselves. Recognize `*ngIf`, `@Input()`, and class guards, but explain why the new way is better.
3. **Change detection is a depth test** — Most Angular devs don't fully understand Zone.js and OnPush. Knowing it signals seniority.
4. **DI hierarchy** — Know the difference between `providedIn: 'root'`, `providers` in `@Component`, and injection context. This comes up in architecture questions.
5. **RxJS fluency matters** — Explain `switchMap` vs `mergeMap` vs `exhaustMap` with use cases (search, parallel, prevent double submit).
6. **Performance vocabulary** — Mention `@defer`, `OnPush`, `trackBy/track`, `shareReplay`, `toSignal` — shows you've built real apps.
7. **`inject()` over constructor** — Using `inject()` in field initializers looks cleaner and shows you know modern Angular.
8. **Don't dismiss NgModules** — Many teams still use them. Know how to work with both worlds.

---

*Angular is evolving fast — you're studying the right version. Go land that interview, Ahmed. 🔺🚀*

> **Legend:** 📖 = Read this | ✅ = Do this | ❌ = Avoid / old way | 🆕 = New in v17+
