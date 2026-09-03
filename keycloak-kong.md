## compose.yaml
```yaml
volumes:
  keycloak_data: {}
  keycloak_db_data: {}
  kong_db_data: {}

networks:
  app-network:
    driver: bridge

# Common Kong environment variables (reusable)
x-kong-config: &kong-env
  KONG_DATABASE: postgres
  KONG_PG_HOST: kong-db
  KONG_PG_DATABASE: kong
  KONG_PG_USER: kong
  KONG_PG_PASSWORD: kongpass

services:
  # Keycloak Database
  keycloak-db:
    image: postgres:16
    container_name: keycloak-db
    restart: on-failure
    volumes:
      - keycloak_db_data:/var/lib/postgresql/data
    networks:
      - app-network
    environment:
      POSTGRES_USER: keycloak
      POSTGRES_DB: keycloak
      POSTGRES_PASSWORD: keycloakpass
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "keycloak"]
      interval: 5s
      timeout: 10s
      retries: 10

  # Keycloak Service
  keycloak:
    image: quay.io/keycloak/keycloak:26.7.1
    container_name: keycloak
    command: start-dev
    #command: start
    restart: on-failure
    ports:
      - "8080:8080"
    environment:
      KC_BOOTSTRAP_ADMIN_USERNAME: admin
      KC_BOOTSTRAP_ADMIN_PASSWORD: strongpassword123
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://keycloak-db:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: keycloakpass
    volumes:
      - keycloak_data:/opt/keycloak/data
    depends_on:
      keycloak-db:
        condition: service_healthy
    networks:
      - app-network

  # Kong Database
  kong-db:
    image: postgres:16
    container_name: kong-db
    restart: on-failure
    volumes:
      - kong_db_data:/var/lib/postgresql/data
    networks:
      - app-network
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kongpass
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "kong"]
      interval: 5s
      timeout: 10s
      retries: 10

  # Kong Bootstrap (runs migrations once)
  kong-bootstrap:
    image: kong:3.7
    container_name: kong-bootstrap
    restart: on-failure
    networks:
      - app-network
    depends_on:
      kong-db:
        condition: service_healthy
    environment:
      <<: *kong-env
    command: kong migrations bootstrap

  # Kong Gateway + Manager
  kong:
    image: kong:3.7
    container_name: kong
    restart: on-failure
    networks:
      - app-network
    environment:
      <<: *kong-env
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001, 0.0.0.0:8444 ssl
      KONG_ADMIN_GUI_LISTEN: 0.0.0.0:8002, 0.0.0.0:8445 ssl
      KONG_ADMIN_GUI_URL: http://localhost:8002
      KONG_PASSWORD: handyshake   # RBAC password for Kong Manager login
    ports:
      - "8000:8000"   # Proxy HTTP
      - "8443:8443"   # Proxy HTTPS
      - "8001:8001"   # Admin API HTTP
      - "8444:8444"   # Admin API HTTPS
      - "8002:8002"   # Kong Manager HTTP
      - "8445:8445"   # Kong Manager HTTPS
    depends_on:
      kong-bootstrap:
        condition: service_completed_successfully
```