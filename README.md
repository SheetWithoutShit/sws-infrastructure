# Spentless. Infrastructure

### How to run development env?
1. Create new directory: spentless.
2. Go to created directory: `cd spentless`
3. Clone spentless services:
    ```shell script
    git clone git@github.com:SpentlessInc/spentless-server.git
    git clone git@github.com:SpentlessInc/spentless-collector.git
    git clone git@github.com:SpentlessInc/spentless-telegram.git
    ```
4. Create docker-compose.yml with the following configurations:
    ```yaml
    version: '3.7'

    services:
      ngrok:
        container_name: spentless-ngrok
        image: wernight/ngrok
        env_file:
          - .env
        environment:
          - NGROK_PORT=collector:5010
        depends_on:
          - collector
        ports:
          - 4040:4040
        restart: always

      postgres:
        image: postgres:10
        container_name: spentless-postgres
        working_dir: /docker-entrypoint-initdb.d
        env_file:
          - .env
        ports:
          - 5432
        volumes:
          - postgres-data:/var/lib/postgresql/data/
        restart: always

      redis:
        image: redis:5
        container_name: spentless-redis
        ports:
          - 6379
        env_file:
          - .env
        restart: always

      collector:
        container_name: spentless-collector
        build: ./spentless-collector
        command: adev runserver run.py --app-factory=init_app --port=5010
        stdin_open: true
        tty: true
        env_file:
          - .env
        volumes:
          - ./spentless-collector/collector:/collector
        depends_on:
          - postgres
          - redis
        ports:
          - 5010:5010
        restart: always

      server:
        container_name: spentless-server
        build: ./spentless-server
        command: adev runserver run.py --app-factory=init_app --port=5000
        stdin_open: true
        tty: true
        environment:
          - PYTHONPATH=$PYTHONPATH:/server
        env_file:
          - .env
        volumes:
          - ./spentless-server/server:/server
          - ./spentless-server/scripts:/scripts
        depends_on:
          - postgres
          - redis
          - ngrok
        ports:
          - 5000:5000
        restart: always

      telegram:
        container_name: spentless-telegram
        build: ./spentless-telegram
        command: adev runserver run.py --app-factory=init_app --port=5020
        stdin_open: true
        tty: true
        env_file:
          - .env
        volumes:
          - ./spentless-telegram/telegram:/telegram
        depends_on:
          - postgres
          - redis
        restart: always

    volumes:
      postgres-data:
   ```
5. Create .env file with the following variables:
    ```shell script
    MONOBANK_WEBHOOK_SECRET=secretherekey

    POSTGRES_HOST=postgres
    POSTGRES_PORT=5432
    POSTGRES_USER=
    POSTGRES_PASSWORD=
    POSTGRES_DB=
    TZ=Europe/Kiev

    REDIS_URL=redis://redis:6379

    COLLECTOR_HOST=collector

    SERVER_HOST=server

    ACCESS_JWT_EXP_DAYS=7
    REFRESH_JWT_EXP_DAYS=30
    JWT_ALGORITHM=HS256
    JWT_SECRET_KEY=secretherekey

    SMTP_LOGIN=spentless.app@gmail.com
    SMTP_PASSWORD=

    TELEGRAM_BOT_TOKEN=
   ```
6. Run docker compose up command: `docker-compose up`
7. Run alembic migration and database seeding command:
    ```shell script
    docker exec -it spentless-server /bin/bash
    export PYTHONPATH=${PYTHONPATH}:/server
    alembic upgrade head
    python scripts/database_seed.py
    ```
