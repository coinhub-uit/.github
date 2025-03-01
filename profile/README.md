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
    uuid userId PK
    string userName  "[UNIQUE] For sending money"
    string fullName
    string birthDay
    string email "For OTP"
    string pin "Hashed"
    bytea avatar "Optional, fallback oauth image"
    string address "Optional"
    string phoneNumber "(10) Optional (for contact)"
  }

  fund_source {
    int sourceId PK "Auto inc"
    uuid userId FK
    money balance "Default 0, constraint >= 0"
  }

  method {
    int methodId PK "Auto inc"
    string name
    string description
  }

  ticket {
    uuid ticketId PK
    int sourceId FK
    int methodId FK
    money initMoney "> 0"
    money paidInterest ">= 0"
    timestamp issueDate "default now"
    timestamp maturityDate
    bool isActive
  }

  type_history{
    uuid ticketId FK "PK too"
    int typeId FK "PK too"
    timestamp issueDate "type changed date => use newest"
  }

  saving_plan{
    int typeId PK "Auto inc"
    string name
    string description
    int days "= 0 if flexible savings"
  }

  interest_history{
    int typeId FK "PK too"
    int interestId FK "PK too"
    timestamp issueDate "type changed date => use newest"
  }

  interest{
    int interestId PK "Auto inc"
    decimal rate
  }

  transaction {
    uuid transactionId PK
    int sourceId FK "Index"
    money amount
    enum type "('deposit', 'withdraw', 'interest_payment')"
    timestamp createdAt "Default now"
  }

  notification {
    int notificationId PK
    uuid userId FK "Index"
    string title
    string content
    timestamp createdAt "Default now"
  }

  admin {
    string username PK "UNIQUE"
    string password "Hashed"
  }

  fund_source }o--|| transaction : has
  user }o--|| notification : has
  user }|--|| fund_source : has
  fund_source }o--|| ticket : has
  ticket ||--o{ method : has
  ticket }|--|| type_history : has
  type_history ||--|{ saving_plan : has
  saving_plan }|--|| interest_history : has
  interest_history ||--|{ interest : has
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
