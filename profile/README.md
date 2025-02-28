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
  user {
    uuid id PK
    string userName UK "For sending money"
    string email "For OTP"
    string pin "Hashed"
    bytea avatar "Optional, fallback oauth image"
    string address "Optional"
    string(10) phoneNumber "Optional (for contact)"
    %% For what??
    %% bool isActive
  }

  fund_source {
    int sourceId PK
    uuid userId FK
    money balance "Default 0, constraint >= 0"
  }

  method {
    int id PK "Auto inc"
    string description
  }

  %% name??
  rate {
    int id PK "Auto inc"
    int days
  }

  ticket {
    uuid id PK
    uuid sourceId FK
    int methodId FK
    int rateId FK
    money initMoney "> 0"
    timestamp issueDate
    bool isActive
  }

  %% need to change the name
  ticket_detail {
    uuid ticketId FK
    decimal percentage
    date startDate
    date endDate
    bool isEnd "current date > endDate (constraint)"
  }

  transaction {
    uuid userId FK "Index"
    money amount
    enum type "('deposit', 'withdraw', 'interest_payment')"
    timestamp createdAt "Default now"
  }

  notification {
    uuid userId FK "Index"
    string title
    string content
    timestamp createdAt "Default now"
  }

  admin {
    string username PK,UK
    string password "Hashed"
  }

  user }o--|| transaction : has
  user }o--|| notification : has
  user }|--|| fund_source : has
  fund_source }o--|| ticket : has
  ticket ||--}o method : has
  ticket ||--}o rate : has
  ticket }|--|| ticket_detail : has
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
