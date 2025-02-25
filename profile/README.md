## ToC

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [DB](#db)
- [Architect](#architect)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

### DB

```mermaid
erDiagram
  USER {
    uuid id PK
    string userName
    string email "For OTP"
    string masterPassword "Hashed"
    string pin "Hashed"
    string address "Nullable"
    string(10) phoneNumber "Optional, for contact or sth"
    %% For what??
    bool isActive
    money balance "Default 0, constrain >= 0"
  }

  METHOD {
    int id PK "Auto inc"
    string name
  }

  RATE {
    int id PK "Auto inc"
    int days
  }

  TICKET {
    uuid id PK
    uuid userId FK
    int methodId FK
    int rateId FK
    money money "Constrain > 0"
    date startDate
    date endDate
    %% or try calculating status by curDate and endDate maybe?
    bool status "True means done (current date > endDate)"
  }

  TICKET_RATE {
    int ticketId PK,FK
    int rateId PK,FK
    decimal percentage
  }

  TRANSACTION {
    int id PK "Auto inc"
    uuid userId FK
    money amount
    enum type "('deposit' || 'withdraw' || 'interest_payment')"
    timestamp createdAt "Default now"
  }

  NOTIFICATION {
    int id PK "Auto inc"
    uuid userId FK
    string title
    string content
    timestamp createdAt "Default now"
  }

  USER }o--|| TICKET : has
  USER }o--|| TRANSACTION : has
  USER }o--|| NOTIFICATION : has
  TICKET ||--}o METHOD : has
  TICKET ||--|| TICKET_RATE : has
  TICKET_RATE ||--}o RATE : has
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

  group firebase[Firebase]
  service fcm(server)[FCM] in firebase

  group ai_server[AI Server]
  service api_ai(server)[API] in ai_server

  junction junction_bot_left

  service mobile[Mobile]
  service admin[Admin]

  api_server:L -- R:authentication_server{group}
  authentication_server:L -- R:database

  admin_web_server:L -- R:api_server

  fcm{group}:R -- L:junction_bot_left

  api_ai{group}:B -- T:api_server

  admin:T -- B:admin_web_server

  mobile:T -- B:api_server
  mobile:L -- R:junction_bot_left
  junction_bot_left:T -- B:authentication_server{group}
```
