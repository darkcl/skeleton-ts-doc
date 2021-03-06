# Dependency Injection

Dependency Injection is used heavily in `skeleton-ts`.

Dependency Injection provides:

- Loosely couple in each modules
- Testable code for each modules
- Concurrent development for multiple developers
- Inversion of control
- Remove any need of singleton object

## Using `InversifyJS`

Our dependency injection framework is `InversifyJS`

It provides:

- Easy to implement container object
- `ExpressJS` routing annotation with [inversify-express-utils](https://github.com/inversify/inversify-express-utils)

### Example: New Endpoint

With `InversifyJS`, creating new endpoint will require editing `common/container.ts`, `server.ts` and create your controller.

```ts
// In server.ts

import "./controller/todo";

// .. Rest of file ...
```

```ts
// In common/container.ts
export class AppContainer {
  constructor() {}

  public async load(): Promise<Container> {
    const container = new Container();

    await this.loadMongoDB(container);

    container.bind<TodoService>(TYPES.TodoService).to(TodoService);
    container.bind<TodoRepository>(TYPES.TodoRepository).to(TodoRepository);

    // .. Other Modules ...

    return container;
  }
}
```

```ts
// In controller/todo.ts

@controller("/todo")
export class TodoController extends BaseHttpController {
  constructor(@inject(TYPES.TodoService) private todoService: TodoService) {
    super();
  }

  @httpGet("/")
  public async getTodos(@response() res: Response) {
    const result: DataObject<ITodo[]> = new DataObject(
      await this.todoService.getTodos(),
      200
    );
    res.status(result.status).send(result.asJson());
  }
}
```

### Example: Middleware Injection

If you are using `ExpressJS` before, you may implement some middleware that modify `Request` object for controller to use.

For example, a locale middleware

```js
function localeMiddleware(req, res, next) {
  // Read locale setting from request
  const locale = Localization.shared(this.defaultLocale);
  const messageStore = locale.of(req.acceptsLanguages());
  req.messageStore = messageStore;
  next();
}
```

This implementation will couple the middleware with controller, made it hard to test.

With `InversifyJS`, we can inject the `messageStore` to controllers

```ts
// In server.ts add one more bind in container

const container = new Container();
// ... Other Modules ...
container
  .bind<LocalizationMiddleware>(TYPES.LocalizationMiddleware)
  .to(LocalizationMiddleware);
const defaultMessage: LocalizedMessage = Localization.shared().defaultStore();
container
  .bind<LocalizedMessage>(TYPES.LocalizedMessage)
  .toConstantValue(defaultMessage);
```

```ts
// In middleware/localization.middleware.ts

@injectable()
export class LocalizationMiddleware extends BaseMiddleware {
  handler(req: Request, res: Response, next: NextFunction): void {
    const messageStore: LocalizedMessage = this.localeManager.of(
      req.acceptsLanguages()
    );
    // Bind LocalizedMessage to container
    this.bind<LocalizedMessage>(TYPES.LocalizedMessage).toConstantValue(
      messageStore
    );
    next();
  }
  constructor(@inject(TYPES.Localization) private localeManager: Localization) {
    super();
  }
}
```

```ts
// In controller/todo.ts
export class TodoController extends BaseHttpController {
  constructor(
    @inject(TYPES.TodoService) private todoService: TodoService,
    @inject(TYPES.LocalizedMessage) private messageStore: LocalizedMessage
  ) {
    super();
  }
  // ..Other Method...
}
```

When testing this controller, you no longer need to mock a request with `messageStore`, you can directly create an `LocalizedMessage` with specific locale.
