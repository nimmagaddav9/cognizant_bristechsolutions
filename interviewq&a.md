## 1. Which version of Angular you are using

I am currently working with **Angular 16/17**, leveraging features like:

- Standalone APIs (`bootstrapApplication`, `provideRouter`)
- New control flow syntax (`@if`, `@for`)
- Deferrable views for performance optimization
- Optional zone-less change detection

I also have experience with previous Angular versions (8+), so I can work on migration and legacy support when needed.

## 2. Organization wants to change Angular 15 to 17 — Steps I Follow

- Commit and backup code, ensure tests/build pass.
- Check Angular Update Guide for breaking changes.
- Run:
  ```bash
  ng update @angular/core@17 @angular/cli@17
  ng update @angular/material@17  # if used
  ```
- Update **TypeScript** (≥5.x), **RxJS** (7.x), and **Zone.js** if needed.
- Refactor for **standalone APIs**, use new control flow (`@if`, `@for`), and remove deprecated code.
- Lint, test, and build in production mode.
- Perform regression testing, deploy to staging, then release to production.

## 3. Lifecycle Hooks in Angular

1. **ngOnChanges** — Called when any `@Input` property value changes.
2. **ngOnInit** — Called once after the first `ngOnChanges`.
3. **ngDoCheck** — Custom change detection logic.
4. **ngAfterContentInit** — Called once after projecting external content into the component.
5. **ngAfterContentChecked** — Called after every check of projected content.
6. **ngAfterViewInit** — Called once after component’s view (and child views) has initialized.
7. **ngAfterViewChecked** — Called after every check of the component’s view (and child views).
8. **ngOnDestroy** — Called just before the component is destroyed; used for cleanup like unsubscribing.

## 4. ngOnChanges vs ngOnInit

- **ngOnChanges**

  - Triggered whenever an `@Input` property changes (including on first binding).
  - Receives a `SimpleChanges` object with previous and current values.
  - Good for reacting immediately to input changes.

- **ngOnInit**
  - Called once after the first `ngOnChanges`.
  - Used for initialization logic that runs only once.
  - Ideal for fetching data or setting up state that doesn't depend on subsequent input changes.

## 5. Dependency Injection in Angular

Angular uses a **hierarchical injector** to provide services or values to classes.  
Services can be provided at:

- **Root** (`providedIn: 'root'`) → Singleton app-wide.
- **Component** (`providers` array) → New instance for that component scope.
- **Feature/module** level.

Example:

```ts
@Injectable({ providedIn: 'root' })
export class UserService {}

constructor(private userService: UserService) {}
```

## 6. How Angular Application Starts

1. **index.html**

   - Contains the root component selector (e.g., `<app-root>`).
   - Loads compiled JavaScript bundles.

2. **main.ts**

   - Bootstraps the application:
     ```ts
     bootstrapApplication(AppComponent, {
       providers: [provideRouter(routes)],
     });
     ```
     _(Older versions use `platformBrowserDynamic().bootstrapModule(AppModule)`)_

3. **Bootstrap Process**

   - Angular sets up the root component and the dependency injection hierarchy.
   - Initializes change detection.

4. **Rendering**

   - Root component template is rendered.
   - Child components load based on the component tree and routing configuration.

## 7. Two-Way Data Binding

- **Definition**: Synchronizes data between the component class and the template.
- **Syntax**: Uses `[( )]` (banana in a box) for property + event binding.
- **Example with ngModel**:

  ```html
  <input [(ngModel)]="username" />
  <p>{{ username }}</p>
  ```

  Custom Component Two-Way Binding:

```
  @Input() value = '';
  @Output() valueChange = new EventEmitter<string>();

```

## 8. How Components Communicate with Each Other

- **Parent → Child**: `@Input()` property binding.
- **Child → Parent**: `@Output()` with `EventEmitter`.
- **Siblings**: Shared service with `Subject`/`BehaviorSubject`.
- **Across Routes**: Router parameters, query params, or state management (NgRx, ComponentStore).

7. Write a piece of code how child and parent components communicate

## 9. Write a Piece of Code: Child and Parent Components Communication

**Child Component (`child.component.ts`)**

```ts
import { Component, Input, Output, EventEmitter } from "@angular/core";

@Component({
  selector: "app-child",
  template: ` <button (click)="sendData()">Send to Parent</button> `,
})
export class ChildComponent {
  @Input() childData = "";
  @Output() dataEvent = new EventEmitter<string>();

  sendData() {
    this.dataEvent.emit("Hello from Child");
  }
}
```

Parent Component (parent.component.html)
<app-child
[childData]="parentMessage"
(dataEvent)="receiveData($event)">
</app-child>

<p>Message from Child: {{ messageFromChild }}</p>

Parent Component (parent.component.ts)

export class ParentComponent {
parentMessage = 'Hello from Parent';
messageFromChild = '';

receiveData(data: string) {
this.messageFromChild = data;
}
}

## 10. What Are Directives and Their Types

**Definition**:  
Directives are classes that add behavior to elements in Angular applications.

**Types**:

1. **Component Directives**

   - Directives with a template.
   - Example: Any Angular component.

2. **Attribute Directives**

   - Change the appearance or behavior of an element.
   - Examples: `ngClass`, `ngStyle`.

3. **Structural Directives**

   - Change the DOM layout by adding or removing elements.
   - Examples: `*ngIf`, `*ngFor`, `*ngSwitchCase`, or new syntax `@if`, `@for`.

## 11. How to Create a Custom Directive

**Example: Highlight Directive**

```ts
import {
  Directive,
  ElementRef,
  HostBinding,
  HostListener,
  Input,
} from "@angular/core";

@Directive({
  selector: "[appHighlight]",
})
export class HighlightDirective {
  @Input() appHighlight = "yellow";
  @HostBinding("style.backgroundColor") bgColor = "";

  constructor(private el: ElementRef) {}

  @HostListener("mouseenter") onMouseEnter() {
    this.bgColor = this.appHighlight;
  }

  @HostListener("mouseleave") onMouseLeave() {
    this.bgColor = "";
  }
}
```

Usage in Template

<p [appHighlight]="'lightblue'">Hover to highlight</p>

## 12. What Is a Pipe in Angular

A **pipe** is a feature in Angular that transforms data in templates without changing the original value.

**Example**:

```html
{{ price | currency:'USD' }} {{ today | date:'shortDate' }}
```

- **Built-in Pipes**: `date`, `currency`, `uppercase`, `json`, etc.
- **Custom Pipes**: Created by developers for specific data transformations.

## 13. Impure vs Pure Pipes

- **Pure Pipes** (default)

  - Executed only when the input value changes by reference.
  - Optimized for performance.
  - Example: Formatting static data.

- **Impure Pipes** (`pure: false`)
  - Executed on every change detection cycle.
  - Useful when data changes without reference change (e.g., pushing into an array).
  - Can impact performance if overused.

## 14. How to Create a Custom Pipe

**Example: Title Case Pipe**

```ts
import { Pipe, PipeTransform } from "@angular/core";

@Pipe({
  name: "titleCase",
})
export class TitleCasePipe implements PipeTransform {
  transform(value: string): string {
    return (
      value?.toLowerCase().replace(/\b\w/g, (char) => char.toUpperCase()) ?? ""
    );
  }
}
```

Usage in Template

<p>{{ 'hello world' | titleCase }}</p>

8. Write logic for custom pipe

## 16. Observable vs Promise

- **Observable**

  - Emits multiple values over time.
  - Lazy execution — starts when subscribed.
  - Can be canceled (unsubscribe).
  - Supports operators (map, filter, etc.).

- **Promise**
  - Emits a single value (or error).
  - Eager execution — starts immediately.
  - Cannot be canceled.
  - No built-in operators for transformation.

## 17. Subject vs BehaviorSubject

- **Subject**

  - No initial value.
  - Subscribers only receive values emitted after subscription.
  - Example: Event streams.

- **BehaviorSubject**
  - Requires an initial value.
  - Emits the current value immediately to new subscribers.
  - Stores the latest emitted value (`.value` property).
  - Example: State management.

## 18. Reactive Forms

- **Definition**: Model-driven approach to handling form inputs in Angular.
- **Key Features**:
  - Uses `FormControl`, `FormGroup`, and `FormBuilder`.
  - Supports synchronous and asynchronous validators.
  - Easy to test and dynamically update.
- **Example**:

  ```ts
  form = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    age: [null, [Validators.min(18)]]
  });

  constructor(private fb: FormBuilder) {}
  ```

  Template:

<form [formGroup]="form" (ngSubmit)="submit()">
  <input formControlName="email">
  <input formControlName="age" type="number">
  <button type="submit">Submit</button>
</form>

## 19. How to Fetch Data from Backend API

- **Using HttpClient Service**:

  ```ts
  import { HttpClient } from "@angular/common/http";
  import { Injectable } from "@angular/core";
  import { Observable } from "rxjs";

  @Injectable({ providedIn: "root" })
  export class UserService {
    constructor(private http: HttpClient) {}

    getUsers(): Observable<User[]> {
      return this.http.get<User[]>("/api/users");
    }
  }
  ```

Usage in Component:
users: User[] = [];

ngOnInit() {
this.userService.getUsers().subscribe(data => this.users = data);
}

## 20. How Routing Works in Angular

- **Purpose**: Maps URL paths to components.
- **Setup**:

  1. Define routes:

     ```ts
     import { Routes } from "@angular/router";

     export const routes: Routes = [
       { path: "", component: HomeComponent },
       { path: "users/:id", component: UserComponent },
       { path: "**", redirectTo: "" },
     ];
     ```

  2. Provide the router:
     ```ts
     bootstrapApplication(AppComponent, {
       providers: [provideRouter(routes)],
     });
     ```
  3. Add `<router-outlet></router-outlet>` in a template to render matched components.

- **Navigation**:
  - Template: `[routerLink]="'/path'"`
  - Code: `this.router.navigate(['/path'])`

## 21. What Is a Router Guard

- **Definition**: A router guard controls whether navigation to or from a route is allowed.
- **Types**:
  - `CanActivate` — Controls access before entering a route.
  - `CanDeactivate` — Checks before leaving a route.
  - `Resolve` — Pre-fetches data before route activation.
  - `CanLoad` / `CanMatch` — Controls loading of lazy-loaded routes.
- **Example**:

  ```ts
  import { CanActivateFn } from "@angular/router";

  export const authGuard: CanActivateFn = () => {
    return isLoggedIn() ? true : createUrlTree(["/login"]);
  };
  ```

## 22. Stores in Angular

- **Purpose**: Manage application state in a predictable and centralized way.

- **Popular Options**:

  - **NgRx Store** — Redux-style, global state with actions, reducers, selectors, and effects.
  - **ComponentStore (NgRx)** — Localized store for a single component or feature.
  - **Signals (Angular 16+)** — Fine-grained reactivity, can be used with services for lightweight state.
  - **Other Libraries** — NGXS, Akita.

- **When to Use**:
  - Complex state shared across many components.
  - Need for state history, debugging, or centralized updates.

## 23. Lazy Loading in Angular

- **Definition**: Loading feature modules or components only when needed, reducing initial bundle size.

- **Benefits**:

  - Faster initial load time.
  - Loads code for a route only when that route is visited.

- **Example (Standalone Component)**:
  ```ts
  const routes: Routes = [
    {
      path: "admin",
      loadComponent: () =>
        import("./admin/admin.component").then((m) => m.AdminComponent),
    },
  ];
  ```

```

```

Example (Lazy-Loaded Module):
const routes: Routes = [
{
path: 'products',
loadChildren: () =>
import('./products/products.module').then(m => m.ProductsModule)
}
];
