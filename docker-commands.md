# useful docker commands
- docker run and remove
- ```
  docker run --rm
  ```
  - backup db
  ```
  docker run --rm -e PGPASSWORD=your_password postgres \
  pg_dump $CONNECTION_STRING > backup.sql
  ```
