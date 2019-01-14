# Development

After writing your documentation, you can start implementation.

## Listing the dependency

Before implementation, it is always good to list out what dependency you will use.

You may ask:

- Does this new feature relate to any existing `Controller`, `Service` or `Repository`?
- Any new third-party library you will use?
- Any sharable logic (e.g. Rate Limiting, Localization)?

## Update application container

### Adding `Controller`

In this example, we add a `TodoController`

```ts
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

We need to add this line in `server.ts` for the server to read meta data

```ts
import "./controller/todo";
```

Also, we need to load `TodoService` into `AppContainer`

```ts
export class AppContainer {
  constructor() {}

  public async load(): Promise<Container> {
    const container = new Container();
    container.bind<TodoRepository>(TYPES.TodoRepository).to(TodoRepository);
    return container;
  }
}
```

### Adding `Service`

### Adding `Repository`

### Adding `Middleware`

`Middleware` helps to pre-process `request` object.

In this example, we use `LocalizationMiddleware` to get current locale (a `LocalizedMessage` object).

`LocalizationMiddleware` will use `Localization` to get a `LocalizedMessage`, so we need to inject it.

```ts
@injectable()
export class LocalizationMiddleware extends BaseMiddleware {
  handler(req: Request, res: Response, next: NextFunction): void {
    const messageStore: LocalizedMessage = this.localeManager.of(
      req.acceptsLanguages()
    );

    // Bind it into a TYPES instead of adding it into req object
    this.bind<LocalizedMessage>(TYPES.LocalizedMessage).toConstantValue(
      messageStore
    );
    next();
  }

  // Inject Localization object, instead of using a singleton object
  constructor(@inject(TYPES.Localization) private localeManager: Localization) {
    super();
  }
}
```

After implementing the middleware, we need to load it in `AppContainer`

```ts
export class AppContainer {
  constructor() {}

  public async load(): Promise<Container> {
    const container = new Container();

    container
      .bind<LocalizationMiddleware>(TYPES.LocalizationMiddleware)
      .to(LocalizationMiddleware);

    const localeManger: Localization = new Localization("en");
    container
      .bind<Localization>(TYPES.Localization)
      .toConstantValue(localeManger);

    const defaultMessage: LocalizedMessage = localeManger.defaultStore();
    container
      .bind<LocalizedMessage>(TYPES.LocalizedMessage)
      .toConstantValue(defaultMessage);
    return container;
  }
}
```

Finally, we can use it in a `Controller`

```ts
@controller("/todo", TYPES.LocalizationMiddleware)
export class TodoController extends BaseHttpController {
  constructor(
    @inject(TYPES.TodoService) private todoService: TodoService,
    @inject(TYPES.LocalizedMessage) private messageStore: LocalizedMessage
  ) {
    super();
  }
}
```

## Working with third-party library

## Pattern to avoid

### Singleton

### Adding value to request object
