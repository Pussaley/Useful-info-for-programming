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

# Преобразования в WebFlux

| Исходный тип | Метод | Результат | Пример |
|--------------|-------|-----------|--------|
| Mono<T>      | flatMapMany | Flux<T> | `Mono.just("a").flatMapMany(s -> Flux.just(s, s+"!"))` → "a", "a!" |
| Mono<T>      | flatMap | Mono<R> | `Mono.just("a").flatMap(s -> Mono.just(s.length()))` → 1 |
| Mono<T>      | map | Mono<R> | `Mono.just("a").map(String::toUpperCase)` → "A" |
| Flux<T>      | flatMap | Flux<R> | `Flux.just("a","bb").flatMap(s -> Mono.just(s.length()))` → 1,2 |
| Flux<T>      | map | Flux<R> | `Flux.just("a","bb").map(String::toUpperCase)` → "A","BB" |
| Flux<T>      | collectList() | Mono<List<T>> | `Flux.just("a","b").collectList()` → ["a","b"] |
| Flux<T>      | collectMap(keyMapper) | Mono<Map<K,T>> | `Flux.just("a","bb").collectMap(String::length)` → {1:"a",2:"bb"} |
| Flux<T>      | next() | Mono<T> | `Flux.just("a","b","c").next()` → "a" |
| Flux<T>      | reduce() | Mono<R> | `Flux.just(1,2,3).reduce(Integer::sum)` → 6 |

> 💡 **Правила на практике**
> 
> - **Mono → Flux** → `flatMapMany`
> - **Flux → Mono** → `collectList()`, `reduce()` или `next()`
> - **Flux → Flux** → `flatMap` / `map`
> - **Mono → Mono** → `map` / `flatMap`
