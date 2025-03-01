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
    nvarchar userName  "[UNIQUE] For sending money (?)"
    nvarchar fullName
    date birthDay
    nvarchar email "For OTP"
    nvarchar pin "Hashed"
    bytea avatar "Optional, fallback oauth image"
    nvarchar address "Optional"
    varchar phoneNumber "(10) Optional (for contact)"
  }

  fund_source {
    serial sourceId PK "Auto inc"
    uuid userId FK "Not null ref user(userId)"
    money balance "Default 0, constraint >= 0"
  }

  method {
    serial methodId PK "Auto inc"
    nvarchar name
    text description
  }


  interest {
    serial interestId PK "Auto inc"
    int typeId FK "Not null ref saving_plan(typeId), PK too"
    timestamp issueDate "type changed date => use newest"
    decimal rate
  }

  ticket {
    uuid ticketId PK
    int sourceId FK "Not null ref fund_source(sourceId)"
    int methodId FK "Not null ref method(methodId)"
    money initMoney "> 100000"
    money paidInterest ">= 0"
    timestamp issueDate "default now"
    timestamp maturityDate "not null, // get issuseDate + days from saving_plan"
    bool isActive "not null, default true"
  }

  type_history{
    uuid ticketId FK "Not null ref ticket(ticketId), PK too"
    int typeId FK "Not null ref saving_plan(typeId), PK too"
    timestamp issueDate "type changed date => use newest"
  }

  saving_plan{
    serial typeId PK "Auto inc"
    nvarchar name
    text description
    int days "not null, >= 0, (= 0 if flexible savings)"
  }


  transaction {
    uuid transactionId PK
    int sourceId FK "Not null ref fund_source(sourceId)"
    money amount "not null, constraint > 0"
    enum type "('deposit', 'withdraw', 'interest_payment')"
    timestamp createdAt "Default now"
  }

  notification {
    serial notificationId PK
    uuid userId FK "Index"
    nvarchar title
    text content
    timestamp createdAt "Default now"
  }

  admin {
    nvarchar username PK "UNIQUE"
    nvarchar password "Hashed"
  }

  fund_source }o--|| transaction : has
  user }o--|| notification : has
  user }|--|| fund_source : has
  fund_source }o--|| ticket : has
  ticket ||--o{ method : has
  ticket }|--|| type_history : has
  type_history ||--|{ saving_plan : has
  saving_plan }|--|| interest : has
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
