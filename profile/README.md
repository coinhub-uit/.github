## ToC

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [DB](#db)
- [Architect](#architect)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

### DB

> [!WARNING]
>
> - UUID or serial? if delete, that empty slot is skipped

> [!TODO]
>
> - Check 0 1 n relation
> - Check `%%`

```mermaid
erDiagram
  user {
    uuid userId PK "Bind with Supabase"
    varchar(20) userName UK "Index"
    nvarchar fullName
    char(12) citizenId
    date birthDay
    varchar(30) email "For OTP"
    text pin "Hashed"
    bytea avatar "Optional, fallback oauth image"
    text address "Optional"
    har(10) phoneNumber "Optional, (for contact)"
  }

  fund_source {
    serial sourceId PK "Auto inc"
    uuid userId FK
    money balance "Default 0, >= 0"
  }

  method {
    %% Non-renewal, principal rollover, principal & interest rollover
    varchar(3) methodId "NR, PR, PIR"
  }

  interest_history {
    serial interestId PK "Auto inc"
    int planId FK "Index"
    timestamp issueDate
    decimal rate
  }

  ticket {
    uuid ticketId PK
    int sourceId FK
    int methodId FK
    %% So what if I changed and there're other ... already, will be failed
    money initMoney "> 100000"
    %% Calculate before return to client (not in DB)?
    money paidInterest ">= 0"
    money anticipatedInterest ">= 0"
    timestamp issueDate "Default now"
    timestamp maturityDate "Get issuseDate + days from saving_plan"
    bool isActive "Default true"
  }

  %% Use latest plan
  plan_history {
    uuid ticketId PK,FK
    int planId PK,FK
    timestamp issueDate
  }

  plan {
    serial planId PK "Auto inc"
    nvarchar name
    text description
    int days UK ">= -1"
  }

  transaction {
    uuid transactionId PK
    int sourceId FK
    money amount "> 0"
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
    nvarchar username PK,UK
    text password "Hashed"
  }

  fund_source }o--|| transaction : has
  user }o--|| notification : has
  user }|--|| fund_source : has
  fund_source }o--|| ticket : has
  ticket ||--o{ method : has
  ticket }|--|| plan_history : has
  plan_history ||--|{ plan : has
  plan }o--|| interest_history : has
  plan_history }|--}o interest_history : has
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
