podman run -d -e POSTGRES_PASSWORD=admin -v postgres_data:/var/lib/postgresql/data -p 5432:5432 --name postgres_container  docker.io/library/postgres
podman run -d -e PGADMIN_DEFAULT_EMAIL=admin@cdac.in -e PGADMIN_DEFAULT_PASSWORD=admin -p 5050:80 --name pgadmin_container docker.io/dpage/pgadmin4

## Postgres 17 with vector
podman run -d -e POSTGRES_PASSWORD=admin -v postgres_data:/var/lib/postgresql/data -p 5432:5432 --name postgres_vector_container docker.io/pgvector/pgvector:pg17
podman exec -it postgres_vector_container psql -U postgres
CREATE EXTENSION IF NOT EXISTS vector;
