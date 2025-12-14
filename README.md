### Docker `docker-compose.yaml`

```yaml
services:
  postgres:
    image: postgres:12.3 // or other image if need to
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=<user>
      - POSTGRES_PASSWORD=<password>
      - POSTGRES_DB=<DB>
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### PostgreSQL `init.sql`:

```sql
CREATE SCHEMA IF NOT EXISTS news_portal_schema;

SET search_path TO news_portal_schema;
```

### Clear all the docker volumes
```
docker volume rm $(docker volume ls -q)
```
