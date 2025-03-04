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

```mermaid
erDiagram
  user {
    uuid userId PK "Bind with Supabase"
    varchar(20) userName UK "Index"
    nvarchar fullName
    char(12) citizenId
    date birthDay
    text pin "Hashed"
    bytea avatar "Optional, fallback oauth image"
    text address "Optional"
    text email "Optional"
    char(10) phoneNumber "Optional"
  }

  source {
    varchar(20) sourceId PK "User choice"
    uuid userId FK
    money balance "Default 0, >= 0"
  }

  settings {
    money minimumInitMoney
  }

  method {
    %% Non-renewal, principal rollover, principal & interest rollover
    varchar(3) methodId "Seed(NR, PR, PIR)"
  }

  interest_rate {
    serial interestRateId PK "Auto inc"
    int planId FK
    timestamp definedDate
    decimal rate
  }

  ticket {
    uuid ticketId PK
    int sourceId FK
    int methodId FK
    %% So what if I changed and there're other ... already, will be failed
    money initMoney ">= settings[minimumInitMoney] when insert"
    %% Calculate before return to client (not in DB)?
    timestamp issueDate "Default now, dynamic"
    timestamp maturityDate "= issueDate + plan[days], dynamic"
    boolean isActive "Default true, index"
  }

  %% Use latest plan
  %% The name is ambiguous
  ticket_interest_rate {
    uuid ticketId PK,FK
    int interestRateId FK
    timestamp issueDate PK
  }

  plan {
    serial planId PK "Auto inc"
    int days UK ">= -1, Seed(-1, 90, 180)"
    boolean isDisabled
    serial latestInterestRate FK "Nullable"
  }

  transaction {
    uuid transactionId PK
    int sourceId FK
    money amount "> 0"
    enum type "deposit, withdraw, interest_payment"
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

  source }o--|| transaction : "has"
  user }o--|| notification : "has"
  user }|--|| source : "has"
  source }o--|| ticket : "has"
  ticket ||--o{ method : "has"
  ticket }|--|| ticket_interest_rate : "has sequential"
  plan }o--|| interest_rate : "has history"
  ticket_interest_rate ||--}o interest_rate : "has latest"
```

> [!NOTE]
>
> - `user[phoneNumber]`, `user[email]` are optional (for contact)

Backend:

- Ticket:
  - Return `paidInterest` = `initMoney` \* `interest_rate[rate]` / 100"
  - Return `anticipatedInterest` is when instantly settle (latest `interest_rate[rate]` where `plan[days=-1]`)

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
