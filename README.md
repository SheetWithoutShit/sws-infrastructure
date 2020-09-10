# Sheet Without Shit. Infrastructure

### How to run development env?
1. Create new directory: sws-infrastructure.
2. Go to created directory: `cd sws-infrastructure`
3. Clone SWS services:
    ```shell script
    git clone git@github.com:SheetWithoutShit/sws-server.git
    git clone git@github.com:SheetWithoutShit/sws-collector.git
    git clone git@github.com:SheetWithoutShit/sws-postgres.git
    git clone git@github.com:SheetWithoutShit/sws-telegram.git
    ```
4. Create docker-compose.yml with the following configurations:
    ```yaml
    version: '3.7'

    services:
      ngrok:
        container_name: sws-ngrok
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
        container_name: sws-postgres
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
        container_name: sws-redis
        ports:
          - 6379
        env_file:
          - .env
        restart: always

      collector:
        container_name: sws-collector
        build: ./sws-collector
        command: adev runserver run.py --app-factory=init_app --port=5010
        stdin_open: true
        tty: true
        env_file:
          - .env
        volumes:
          - ./sws-collector/collector:/collector
        depends_on:
          - postgres
          - redis
        ports:
          - 5010:5010
        restart: always

      server:
        container_name: sws-server
        build: ./sws-server
        command: adev runserver run.py --app-factory=init_app --port=5000
        stdin_open: true
        tty: true
        env_file:
          - .env
        volumes:
          - ./sws-server/server:/server
        depends_on:
          - postgres
          - redis
          - ngrok
        ports:
          - 5000:5000
        restart: always

      telegram:
        container_name: sws-telegram
        build: ./sws-telegram
        command: python run.py
        stdin_open: true
        tty: true
        env_file:
          - .env
        volumes:
          - ./sws-telegram/telegram:/telegram
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

    REDIS_HOST=redis
    REDIS_PORT=6379

    COLLECTOR_HOST=collector

    SERVER_HOST=server

    ACCESS_JWT_EXP_DAYS=7
    REFRESH_JWT_EXP_DAYS=30
    JWT_ALGORITHM=HS256
    JWT_SECRET_KEY=secretherekey

    SMTP_LOGIN=sheetwithoutshit@gmail.com
    SMTP_PASSWORD=

    AWS_DEFAULT_REGION=eu-north-1
    AWS_ACCESS_KEY_ID=
    AWS_SECRET_ACCESS_KEY=

    TELEGRAM_BOT_TOKEN=
   ```
6. Run docker compose up command: `docker-compose up`
