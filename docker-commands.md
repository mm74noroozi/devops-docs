# useful docker commands
- docker run and remove
- ```
  docker run --rm
  ```
  - backup db
  ```
  docker run --rm postgres \
  pg_dump $CONNECTION_STRING > backup.sql
  ```
