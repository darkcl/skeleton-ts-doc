# Overview

<script>mermaid.initialize({startOnLoad:true});</script>

## Folder Structure

```
app
├── common
│   └── { Common Module, e.g. Data Object }
├── constant
│   └── { Constant Variable, e.g. DI Types }
├── controller
│   └── { Controllers, e.g. TodoController }
├── locale
│   └── { Locale Releated Resources }
├── middleware
│   └── { Middlewares, e.g. LocaleMiddleware }
├── repositories
│   └── { Repositories , e.g. TodoRepository}
├── server.ts
├── service
│   └── { Services, e.g. TodoService }
└── utils
    └── { Utitlities, e.g. MongoConnection }
```

## Relationship between modules

### Global Scope

<div class="mermaid">
graph LR
  id1[server.ts]-->id2[Setup Project Modules]
  id1-->id3[Setup Global Middleware]
  id2-->id4[Controller]
  id2-->id5[Service]
  id2-->id6[Repository]
  id2-->id7[Middleware]
  id3-->id8[CORS]
  id3-->id9[Helmet]
  id3-->id10[Logger]
</div>

### Domain Scope

<div class="mermaid">
graph TB
  a[User Request]-->id0[Middleware]
  id0-->id1[Controller]
  id1-->|Uses|id2[Service]
  id2-->|Result|id1
  id2-->|Uses|id3[Repository]
  id2-->|Uses|id4[Utility]
  id3-->|Query|id5[Persistence Data]
  id4-->|Interact|id6[Thrid-party Data]
</div>
