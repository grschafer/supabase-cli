# About Initial Schemas

These SQL files represent initial schemas needed to set up the database with Supabase stuff. These need to be manually generated for each Postgres major version. Which initial schema used depends on the Docker image tag used to run the local db, which in turn depends on the CLI's `db.major_version` config. 

The initial schema for PG12 is not available because the latest image (`supabase/postgres:12.5.0`) doesn't contain `wal2json`, which is required for Realtime to work.

# Why use the pg_dump output instead of running the `init.sql` directly?

Because Realtime, GoTrue, Logflare, and Storage have their own migrations, and these need to be included in the initial schema for e.g. `supabase db reset` to work.

# How to Generate Initial Schemas

1. Start supabase local development stack with the following `config.toml`:

Set `major_version` to 15 (13.sql and 14.sql are no longer supported).
Set `analytics` to false.

```bash
go run . start
```

2. Run the initial dump script and pipe output to `15.sql`

```bash
./tools/dump_initial_schema.sh > internal/utils/templates/initial_schemas/15.sql
```

3. Commit changes that are relevant for the update

`INSERT INTO _realtime.extensions` and `_realtime.tenants` usually do not require updating.
`INSERT INTO *.schema_migrations` statements are only required if there are new migrations.
`ALTER EVENT TRIGGER issue_pg_cron_access OWNER TO` should be followed by supabase_admin.
