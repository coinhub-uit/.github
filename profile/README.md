## ToC

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [DB](#db)
- [Architect](#architect)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

### DB

```mermaid
---
title: Database
---
erDiagram
  USER {
    uuid id PK
    string userName
    string pin "Hashed"
    string address "Nullable"
    money balance "Default 0, constrain >= 0"
  }

  METHOD {
    int id PK "Auto inc, from 0"
    string name
  }

  RATE {
    int id PK "Auto inc, from 0"
    int days
    decimal percentage
  }

  TICKET {
    uuid id PK
    uuid userId FK
    int methodId FK
    int rateId FK
    money money
    date startDate
    date endDate
    status bool "True means done (current date > endDate)"
  }

  TRANSACTION {
    int id PK "Auto inc, from 0"
    uuid userId FK
    money amount "Negative means out"
  }

  NOTIFICATION {

  }

  USER |o--|| TICKET : have
  TICKET ||--|| METHOD : have
  TICKET ||--|| RATE : have
  USER |o--|| TRANSACTION : have
```

### Architect

```mermaid
architecture-beta
  group supabase[Supabase]
  service authentication_server(server)[Authenticate Server] in supabase
  service database(database)[Database] in supabase

  group own_server[Own Server]
  service api_server(server)[API] in own_server
  service admin_web_server(server)[Admin Web] in own_server

  group ai_server[AI Server]
  service api_ai(server)[API] in ai_server

  service mobile[Mobile]
  service admin[Admin]

  junction server_api_supabase in supabase

  api_server:T -- B:api_ai{group}
  api_server:L -- R:server_api_supabase
  server_api_supabase:T -- B:database
  server_api_supabase:B -- T:authentication_server

  admin_web_server:L -- R:api_server

  admin:T -- B:admin_web_server

  mobile:T -- B:api_server
  mobile:L -- R:authentication_server
```
