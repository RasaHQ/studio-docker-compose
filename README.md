# Rasa Studio Docker Setup

This directory contains the Docker Compose configuration to run Rasa Studio locally. Further information about this can be found [here](https://rasa.com/docs/studio/installation/installation-guides/docker-compose-guide).

## Prerequisites

- [Docker](https://docs.docker.com/desktop/) (version 20.10.0 or higher)
- Docker Compose (version 2.0.0 or higher) (Not required if you have Docker Desktop installed)
- At least 4GB of available RAM
- The following ports must be available on your host machine:
  - 4000: Studio Backend API
  - 5005, 5006, 8000: Model Service
  - 5432: PostgreSQL Database
  - 8080: Web Client Interface
  - 8081: Keycloak Authentication
  - 19092, 29092: Kafka Broker

## Environment Variables

Before running the application, set up the required environment variables. You can either export them in your shell or put them in a `.env` file in this directory (same folder as `docker-compose.yml`).

```bash
# Required for the Model Service
export RASA_PRO_LICENSE=your_license_key
export OPENAI_API_KEY=your_openai_key  # If using OpenAI features

# Optional - only if using custom image paths/tags
export STUDIO_IMAGE_PATH=your_custom_image_path
export STUDIO_IMAGE_TAG=your_custom_tag
export RASA_PRO_IMAGE=your_custom_rasa_pro_image  # Custom Rasa Pro image for model-service

# Optional - for voice/audio features in the backend
export DEEPGRAM_API_KEY=your_deepgram_key
export CARTESIA_API_KEY=your_cartesia_key
```

## Running the Application

1. Navigate to this directory containing the `docker-compose.yml` file.

2. Start all services:

```bash
docker compose up
```

The startup process will show logs from all services in your terminal. The application is ready when you see the following message from the startup-helper service:

```
🚀 Rasa Studio is ready!
📱 Studio URL: http://localhost:8080
🔐 Keycloak User Management URL: http://localhost:8081/auth/
```

If you prefer to run the services in the background, you can use:

```bash
docker compose up -d
```

Then monitor the progress with:

```bash
docker compose logs -f startup-helper
```

## Accessing the Application

- Main Application: http://localhost:8080
- Keycloak Admin Console: http://localhost:8081/auth/
- Backend API: http://localhost:4000/api
- Model Service: http://localhost:8000

## Default Credentials

Default values are defined for a number of credentials in the Docker Compose setup. You can override them either by putting the variables in a `.env` file in this directory or by passing them on the command line when you run `docker compose up`.

### Overriding defaults

**Option 1: `.env` file**  
Create a `.env` file in this directory (same folder as `docker-compose.yml`) and add the variables you want to override. Then run `docker compose up` as usual.

**Option 2: Inline when starting**  
Pass the variables before the command so they apply to that run only:

```bash
KEYCLOAK_ADMIN=myadmin \
KEYCLOAK_ADMIN_PASSWORD=mypass \
KEYCLOAK_DEFAULT_USER_PASSWORD=userpass \
KEYCLOAK_API_PASSWORD=apipass \
DB_USER=dbuser \
DB_PASS=dbpass \
docker compose up
```

Replace the example values with your own. You can omit any variable you do not want to override.

### Credentials reference

| Credential                      | Purpose                                                                                   | Default Value | Environment Variable to Override |
| ------------------------------- | ----------------------------------------------------------------------------------------- | ------------- | -------------------------------- |
| Keycloak Admin User             | The username for the admin Keycloak user, used to create and manage users                 | `kcadmin`     | `KEYCLOAK_ADMIN`                 |
| Keycloak Admin Password         | The password for the admin Keycloak user, used to create and manage users                 | `kcadmin`     | `KEYCLOAK_ADMIN_PASSWORD`        |
| Keycloak Default Users Password | The password for the default Rasa Studio users that are automatically created in Keycloak | `rasa`        | `KEYCLOAK_DEFAULT_USER_PASSWORD` |
| Keycloak API Password           | Password used by the backend to communicate with the Keycloak API                         | `realmadmin`  | `KEYCLOAK_API_PASSWORD`          |
| Database User                   | The Postgres user for Studio and Keycloak databases                                       | `studio`      | `DB_USER`                        |
| Database Password               | The password for the Postgres DB that Studio uses                                         | `studio`      | `DB_PASS`                        |

## Logging in

To sign in to the Studio web client (http://localhost:8080), use one of the default users created in Keycloak:

- **Username:** `superuser`
- **Password:** The default is `rasa`, or the value you set with `KEYCLOAK_DEFAULT_USER_PASSWORD` if you override it.

Example: if you run with `KEYCLOAK_DEFAULT_USER_PASSWORD=userpass`, log in with username `superuser` and password `userpass`.

## Stopping the Application

To stop all services, press `Ctrl+C` if running in the foreground, or if running in the background:

```bash
docker compose down
```

To stop all services and remove volumes (this will delete all data):

```bash
docker compose down -v
```

## Data Persistence

The following data is persisted:

- **Docker volumes:** `studio-db-data` (PostgreSQL database data), `kafka-data` (Kafka message broker data)
- **Local directories:** The model service uses `./rasa-pro` and `./rasa-pro/enterprise-search-data` in this directory for working data and enterprise search data; create them if needed or they will be created when the service starts

## Troubleshooting

1. To view logs for a specific service:

```bash
docker compose logs -f [service-name]
```

Service names: studio-backend, studio-web-client, studio-keycloak, studio-event-ingestion, model-service, kafka, database, startup-helper

2. If you encounter memory issues, ensure Docker has enough memory allocated in your Docker Desktop settings.

## Health Checks

All services have built-in health checks. You can monitor their status using:

```bash
docker compose ps
```
