# Angular Cheatsheet — Quick Learning & Revision

Goal: concise explanations, code snippets, best practices, and real-world use cases for Angular (v12+).

---

## Table of contents
- Setup & CLI
- Project structure & NgModule
- Components & templates
- Data binding & directives
- Services & dependency injection
- Routing & guards
- Forms (template-driven & reactive)
- HTTP & interceptors
- RxJS & Observables (practical)
- Change detection & performance
- Pipes
- Testing (unit & e2e)
- Best practices & common patterns
- Security
- Tooling & deployment
- Real-world examples / snippets

---

## Setup & CLI
- Install CLI: npm i -g @angular/cli
- New app: ng new my-app --routing --style=scss
- Generate: ng generate component|service|module|pipe|guard
- Serve: ng serve
- Build prod: ng build --prod

---

## Project structure & NgModule
- App is composed of NgModules; root module bootstraps AppComponent.
- Feature modules group related functionality; use lazy loading via router.

Example module:
```ts
@NgModule({
  declarations: [MyComponent],
  imports: [CommonModule, RouterModule],
  providers: [MyService],
  exports: [MyComponent]
})
export class MyModule {}
```

Best practice: keep NgModule boundaries clear; prefer feature modules and lazy load large features.

---

## Components & templates
- Component = class + template + metadata.

Example component:
```ts
@Component({
  selector: 'app-user',
  template: `<h3>{{user?.name}}</h3><button (click)="save()">Save</button>`
})
export class UserComponent {
  @Input() user!: User;
  @Output() saved = new EventEmitter<User>();
  save(){ this.saved.emit(this.user); }
}
```

Template tips: use async pipe for Observables, avoid manual subscribe/unsubscribe when possible.

---

## Data binding & directives
- Interpolation: {{ value }}
- Property binding: [disabled]="isDisabled"
- Event binding: (click)="onClick()"
- Two-way: [(ngModel)]="model" (requires FormsModule)
- Structural directives: *ngIf, *ngFor
- Attribute directives: [ngClass], [ngStyle] or custom directives

TrackBy for lists:
```html
<li *ngFor="let item of items; trackBy: trackById">{{item.name}}</li>
```
```ts
trackById(_i: number, item: Item) { return item.id; }
```

---

## Services & Dependency Injection
- Services provide logic/state. Register in root or module/providers.
- Use providedIn: 'root' for tree-shakable singletons.

Example service:
```ts
@Injectable({ providedIn: 'root' })
export class ApiService {
  constructor(private http: HttpClient) {}
  getPosts() { return this.http.get<Post[]>('/api/posts'); }
}
```

Prefer constructor injection and readonly properties.

---

## Routing & guards
- Define routes and lazy load modules.

Routes:
```ts
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule), canLoad: [AuthGuard] }
];
```

Guards: CanActivate, CanLoad, CanDeactivate. Use for auth, unsaved changes.

---

## Forms
Template-driven (small forms) vs Reactive (complex forms).

Reactive form example:
```ts
form = this.fb.group({
  name: ['', [Validators.required]],
  email: ['', [Validators.required, Validators.email]],
  tags: this.fb.array([])
});
addTag(v: string){ (this.form.get('tags') as FormArray).push(this.fb.control(v)); }
```

Best practice: prefer reactive forms for predictable validation and testability.

---

## HTTP & interceptors
- Use HttpClient; returns Observables.
- Use interceptors for auth headers, logging, error handling.

Interceptor example:
```ts
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler) {
    const token = this.auth.token;
    const authReq = token ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }) : req;
    return next.handle(authReq);
  }
}
```

Register in app module providers with multi: true.

---

## RxJS & Observables (practical)
- Use Observables for async flows (HTTP, events, forms).
- Common operators: map, switchMap, mergeMap, concatMap, debounceTime, distinctUntilChanged, catchError, shareReplay.

Examples:
- Search with debounce:
```ts
this.results$ = fromEvent(this.input.nativeElement, 'input').pipe(
  map((e: any) => e.target.value),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(q => this.api.search(q)),
  catchError(() => of([]))
);
```

- Avoid nested subscriptions; compose streams and use async pipe to subscribe in template.

---

## Change detection & performance
- Default vs OnPush strategies. OnPush relies on immutable inputs and change via reference changes.
```ts
@Component({ changeDetection: ChangeDetectionStrategy.OnPush, ... })
```
- Use trackBy for ngFor, virtual scroll for large lists, lazy load modules and images.
- For heavy computations, use pure pipes or Web Workers.

---

## Pipes
- Built-in: date, currency, async, json
- Custom pure/impure pipe:
```ts
@Pipe({name: 'humanize', pure: true})
export class HumanizePipe implements PipeTransform {
  transform(val: number) { /* ... */ }
}
```

Prefer pure pipes for performance.

---

## Testing
- Unit: TestBed for components/services; use jasmine/jest.
- Component test example:
```ts
beforeEach(async () => {
  await TestBed.configureTestingModule({ declarations: [MyComp], providers: [MyService] }).compileComponents();
});
it('renders title', () => { const fixture = TestBed.createComponent(MyComp); fixture.detectChanges(); expect(fixture.nativeElement.textContent).toContain('Title'); });
```
- E2E: Cypress or Protractor (Cypress recommended).

---

## Best practices & common patterns
- Single Responsibility: small components, smart/dumb (container/presentational) split.
- Use services for state & side effects; components for UI.
- Use OnPush and immutability where possible.
- Centralize API and error handling (interceptors).
- Use environment files for config; never commit secrets.

---

## Security
- Avoid innerHTML with untrusted content; sanitize if needed.
- Use CSP and server-side protections.
- Protect routes with guards; secure tokens (httpOnly cookies preferred).
- Validate on server — client-side checks are UX only.

---

## Tooling & deployment
- Lint: eslint + angular-eslint; formatting: prettier.
- Build: ng build --prod; enable budgets and analyze bundle with source-map-explorer.
- AOT and production flags reduce size and improve performance.
- Use CI to run tests/lint/build.

---

## Real-world examples & snippets

1) Component + service + async pipe
```ts
// posts.service.ts
getPosts() { return this.http.get<Post[]>('/api/posts').pipe(shareReplay(1)); }
// posts.component.ts
posts$ = this.postsSvc.getPosts();
<!-- template -->
<ul><li *ngFor="let p of posts$ | async">{{p.title}}</li></ul>
```

2) Route Guard (auth)
```ts
@Injectable({providedIn:'root'})
export class AuthGuard implements CanActivate {
  constructor(private auth: AuthService, private router: Router) {}
  canActivate(): boolean {
    if (!this.auth.isLoggedIn()) { this.router.navigate(['/login']); return false; }
    return true;
  }
}
```

3) Optimistic update pattern
```ts
likePost(post: Post){
  post.liked = !post.liked; // optimistic
  this.api.like(post.id).pipe(catchError(err => {
    post.liked = !post.liked; // rollback
    return throwError(err);
  })).subscribe();
}
```

4) Lazy load feature module
```ts
{ path: 'shop', loadChildren: () => import('./shop/shop.module').then(m => m.ShopModule) }
```

5) State management (NgRx brief)
- Use NgRx for large apps: actions, reducers, selectors, effects. Keep side effects in effects.

---

## Further reading / next steps
- Angular update guide, RxJS deep dive, performance checklist, advanced change detection and zone-less strategies (NgZone).
- Explore Angular CDK for accessibility and utilities.

---

Keep this file as your Angular single-page quick reference. Add real-world snippets from your projects as needed.
