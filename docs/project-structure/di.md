# Dependency Injection

Dependency Injection is used heavily in `skeleton-ts`.

Dependency Injection provides:

- Loosely couple in each modules
- Testable code for each modules
- Concurrent development for multiple developers
- Inverse of control
- Remove any need of singleton object

## Using `InversifyJS`

Our dependency injection framework is `InversifyJS`

It provides:

- Easy to implement container object
- `ExpressJS` routing annotation with [inversify-express-utils](https://github.com/inversify/inversify-express-utils)

### Example: New Endpoint

```ts
// In server.ts

import "./controller/todo";

(async () => {
  // Connect to MongoDB
  await MongoDBConnection.connect();

  // Load everything needed to the Container
  const container = new Container();
  container.bind<TodoService>(TYPES.TodoService).to(TodoService);
  container.bind<TodoRepository>(TYPES.TodoRepository).to(TodoRepository);

  // Start the server
  const server = new InversifyExpressServer(container, null, {
    rootPath: "/api/v1"
  });

  server.setConfig(app => {
    app.use(
      bodyParser.urlencoded({
        extended: true
      })
    );
  });

  server.setErrorConfig(app => {
    // Error Logger
    const errorMiddleware: ErrorMiddleware = new ErrorMiddleware();
    app.use(errorMiddleware.process());
  });

  const serverInstance = server.build();

  const port: string = process.env.PORT || "3000";

  serverInstance.listen(parseInt(port));
})();
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
    const locale: Localization = Localization.shared(this.defaultLocale);
    const messageStore: LocalizedMessage = locale.of(req.acceptsLanguages());

    // Bind LocalizedMessage to container
    this.bind<LocalizedMessage>(TYPES.LocalizedMessage).toConstantValue(
      messageStore
    );
    next();
  }
  constructor(private defaultLocale: string) {
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
