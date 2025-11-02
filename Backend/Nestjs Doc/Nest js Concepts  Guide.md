## NestJS Concepts and Dependency Injection

---
## 1. How Dependency Injection Works in NestJS

- NestJS uses **TypeScript decorators** `@Injectable()` and metadata to manage DI via its built-in **IoC (Inversion of Control) container**.
- Nest uses this registration in a **module** to manage **lifecycle and dependencies**.

---

## 2. Difference Between `@Injectable()`, `@Module()`, and `@Controller()`

### `@Injectable()`
- Marks a class as a **provider** that can be injected as a dependency.
- Used on **services**, **repositories**, or any class that holds **business logic** or interacts with the **database**.
- Enables **dependency injection** via Nest’s IoC container.

### `@Controller()`
- Marks a class as a **controller** that handles incoming HTTP requests and returns responses.
- Controllers define **routes** and delegate logic to **providers** (e.g., services).

### `@Module()`
- A **class decorator** that defines a module.
- A module **groups controllers and providers together**.
- Every NestJS app has at least **one module** (typically `AppModule`).

---

## 3. `@Global()`

- The `@Global()` decorator is used to make a module **globally available** throughout the application.
- When a module is marked as global, its **providers become accessible** in any other module **without importing it explicitly**.

---

## 4. Provider Configuration: `useClass`, `useValue`, and `useFactory`

### 4.1 `useClass`
- Tells Nest to use a **particular class** to provide a service.

```typescript
class MyService {
  get() {
    return 'From MyService';
  }
}

@Module({
  providers: [
    {
      provide: 'MyServiceToken',
      useClass: MyService,
    },
  ],
})
export class AppModule {}

// Usage in constructor
@Inject('MyServiceToken') private readonly myService: { get: () => string };
```

### 4.2 `useValue`
- Allows you to provide a **constant or object** instead of a class.

```typescript
const config = {
  host: 'localhost',
  port: 3000,
};

@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: config,
    },
  ],
})
export class AppModule {}
```

### 4.3 `useFactory`
- Allows you to use a **factory function** to generate the provider value, optionally injecting other providers.

```typescript
{
  provide: 'DATABASE_OPTIONS',
  useFactory: (configService: ConfigService) => configService.getDbOptions(),
  inject: [ConfigService],
}
```
## NestJS Provider Strategies: `useFactory` Use Cases

### 🟢 When to use `useFactory`

`useFactory` is useful when the value must be **created dynamically**, often depending on runtime conditions, environment, or other services. NestJS will call the factory function and return the result.

---

### ✅ Real-World Use Cases

### 1. Database Connection Options
- Connect to different databases based on the environment (`development`, `staging`, `production`).
- The factory reads from a `ConfigService` and returns the correct host, username, and password.

---

### 2. API Client Configuration
- External APIs like **Stripe, PayPal, or Google Maps** require API keys.
- The factory fetches the right API key from environment variables or a config service and builds the client configuration.

---

### 3. Feature Flag Management
- Enable or disable certain features depending on **user roles** or **application version**.
- The factory decides which features to activate based on environment or runtime conditions.

---

### 4. Caching Strategy
- In development → use **in-memory caching**.  
- In production → use **Redis** for distributed caching.  
- The factory selects the correct caching option depending on the environment.

---

### 5. Dynamic Logging Configuration
- Development → verbose logs for debugging.  
- Production → minimal logs for performance.  
- The factory dynamically sets the logging level.

---

## 🔑 Summary
Use `useFactory` whenever the value:
- Depends on **runtime conditions**  
- Relies on **environment variables**  
- Needs **other services** to be computed  
- Cannot be provided as a static value (`useValue`) or a simple class (`useClass`)

---

## 5. Difference Between Middleware, Guard, Interceptor, and Exception Filter

| Feature             | Middleware                               | Guard                                    | Interceptor                                         | Exception Filter                          |
| ------------------- | ---------------------------------------- | ---------------------------------------- | --------------------------------------------------- | ----------------------------------------- |
| Purpose             | Process incoming requests early          | Control whether a request can proceed    | Modify or extend request/response lifecycle         | Handle and format uncaught exceptions     |
| Execution Order     | First to run (before guards)             | Runs before the route handler            | Runs before and after route handler                 | Runs only on exceptions                   |
| Typical Use         | Logging, setting headers, parsing tokens | Auth, RBAC (role-based access control)   | Transform response, logging time, caching           | Custom error responses, global handling   |
| Access to Metadata  | No                                       | Yes (via ExecutionContext and Reflector) | Yes                                                 | Yes                                       |
| Can Modify Request  | Yes                                      | No                                       | Yes (through transformation)                        | No                                        |
| Can Modify Response | Yes (through res object)                 | No                                       | Yes (via RxJS pipe)                                 | Yes (formats error response)              |
| Best For            | Cross-cutting concerns (e.g., logging)   | Access control, authentication           | Cross-cutting features like timing, transformation  | Global error handling                     |
| Example             | Add `X-Custom-Header`, log `req.body`    | Allow route only if user has role admin  | Wrap response in `{ data: ... }`, log response time | Return `{ statusCode, message }` on error |

---
At this stage, NestJS doesn’t yet know **which route** is being accessed or what **metadata** (like roles/permissions) is attached to it.
### Examples

#### Middleware Example
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: Function) {
    console.log('Request...', req.method, req.url);
    next();
  }
}
```

#### Guard Example
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return request.user.role === 'admin';
  }
}
```

#### Interceptor Example
```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(map(data => ({ data })));
  }
}
```

#### Exception Filter Example
```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    response.status(status).json({
      statusCode: status,
      message: exception.message,
    });
  }
}
```