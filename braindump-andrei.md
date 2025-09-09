# Braindump

## Architecture 
### Component Diagrams

```puml
@startuml
skin rose
frame "Component Diagram" {

    () Auth
	() postgREST
	() DB as db
	() Storage
	
	' note right of db: Direct UDP Connection

    cloud "Vercel" as front {
        [Next.js Web App] as next
        [drizzle-orm] as drizzle
        [supabase-js] as supajs
    }
    
    next .u-> drizzle
    next .u-> supajs

    cloud Supabase as supa {
        database "Postgresql" as pg
        [GoTrue] as auth
        note right of auth: Auth Server
        database "Storage Bucket" as bkt
    }
    supa -d--|> Auth
	supa -d--|> postgREST
	supa -d--|> db
	supa -d--|> Storage
}

 supajs .-u--> postgREST
 supajs .-u--> Auth
 drizzle .-u--> db
 supajs .-u--> Storage
 auth  .-l--> pg

@enduml
```
This is a simplified version that uses Supabase for everything.


```puml
@startuml
skin rose
frame "Component Diagram" {

    cloud Supabase as supa {
        [Postgres DB for pgvector] as pg
        [Auth] as auth
    }
    
    cloud "Vercel" as front {
        [Next.js Web App] as next
    }
    
    cloud "Cloudflare" as edge {
        database "D1 Database" as d1
        [API Worker] as api
    }
    
    
    node "Student ID Validator" as id {
        component [ID Validation Service] as s_id 
    }
}

' Vertical layout
edge <-l-. front 
api .-r-> d1
front .-d-> supa
pg <-d-. api
edge .-> id
id .-l-> supa
auth .-l-> pg

' Original layout
' edge <--. front 
' api .-r-> d1
' front .--> supa
' pg <--. api
' edge .-> id
' id .--> supa
' auth .-l-> pg

@enduml
```
This version uses a multi-cloud architecture with next.js on vercel for the frontend, supabase for auth and postgres/pgvector database, and cloudflare for edge computing with d1 database and api workers. A separate student id validation service handles user verification. The design prioritizes performance through edge distribution, industry standard and scalable, and vector search capabilities for content recommendations.
### Sequence Diagrams
Some poorly done auth diagrams

```puml
@startuml
skin rose
title Login Flow
actor User
User -> Nextjs: login
Nextjs -> Auth: middleware request
Nextjs <-- Auth: jwt & refresh token
Nextjs -->> Nextjs: set cookies
User <-- Nextjs: route /
Nextjs -> Auth: refrtesh jwt
Nextjs <-- Auth: jwt & refresh token
Nextjs -->> Nextjs: set cookies
```

```puml
@startuml
skin rose
title Signup Flow
actor User
User -> Nextjs: create account
Nextjs -> Auth: middleware request
Nextjs <-- Auth: account crteated
User <-- Nextjs: route /onboarding
Nextjs -->> API: (async) POST /validate
API -> IDValidator: validate id
API <-- IDValidator: student ID Valid
API -> EmailService: notify user
API <- EmailService: email sent successfuly
```


