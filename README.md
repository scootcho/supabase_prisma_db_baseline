### Supabase Prisma Database Baseline

- Schema dump of initialized Supabase DB
- Prisma migration to baseline Supabase schema

### Supabase Local Setup

[Local Development Supabase Docs](https://supabase.com/docs/guides/cli/local-development)
https://youtu.be/iXXgUeKt-tM?feature=shared&t=1819

```
supabase login
supabase init
supabase start
supabase status
npx supabase stop --backup
```

## Baseline your database

[Baseline your database typescript-postgresql Prisma Docs](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project/relational-databases/baseline-your-database-typescript-postgresql)

```
mkdir -p prisma/migrations/0_init
npx prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.prisma --script > prisma/migrations/0_init/migration.sql
npx prisma migrate resolve --applied 0_init
```

### Backup Database

```
psql 'postgresql://postgres:postgres@localhost:54322/postgres'
postgres=# psql \! pg_dump 'postgresql://postgres:postgres@localhost:54322/postgres' > ./mydb.sql
```

### Install Prisma

```
pnpm install typescript ts-node @types/node --save-dev
pnpm install prisma --save-dev
pnpm install @prisma/client
npx prisma init --datasource-provider postgresql
```

`schema.prisma`

```
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["multiSchema"]
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
  schemas   = ["auth", "public"]
  // to generate all schemas
  // schemas   = ["auth", "public", "extensions", "graphql", "graphql_public", "realtime", "storage"]
}
```

```
npx prisma db pull
npx prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.prisma --script > prisma/migrations/0_init/migration.sql
```

### Resolving self referential columns that will not work with Prisma

[Making Prisma and Supabase play nicely by Warren Sadler Aug, 2023 Medium](https://medium.com/@warren_74490/making-prisma-and-supabase-play-nicely-5acfe2255591)

```
// in create auth.users
-- "confirmed_at" TIMESTAMPTZ(6) DEFAULT LEAST(email_confirmed_at, phone_confirmed_at),
++ "confirm_at" TIMESTAMPTZ(6) GENERATED ALWAYS AS (LEAST(email_confirmed_at, phone_confirmed_at)) STORED
```

```
// in create auth.identities
-- "email" TEXT DEFAULT lower((identity_data->>'email'::text)),
++ "email" TEXT GENERATED AS (lower((identity_data->>'email'::text))) STORED,
```
