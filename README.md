# EspoCRM Development Environments

This repository contains sample Docker development environments for EspoCRM with various PHP versions, web servers, and database configurations.

## Quick Start

1. Choose and download the desired environment directory (e.g., `php8.5-nginx-mariadb`).
2. Navigate to the downloaded directory and run:

```bash
docker compose up -d
```

3. Wait for the containers to start and initialize.
4. Download and extract EspoCRM files into the `html` directory.
5. Open `http://localhost:8080` in your browser and complete the EspoCRM installation.

## Docker Commands

### Start containers

```bash
docker compose up -d
```

### Start with rebuild

```bash
docker compose up -d --build "$@"
```

### Restart containers

```bash
docker compose restart
```

### Stop and remove containers

```bash
docker compose down
```

### Check container status

```bash
docker compose ps
```

### View logs

```bash
docker compose logs
```

## Database Configuration

### MySQL

Default connection credentials:

- **Host Name**: `espocrm-mysql`
- **Database Name**: Any name you choose
- **User**: `root`
- **Password**: `1`

### MariaDB

Default connection credentials:

- **Host Name**: `espocrm-mariadb`
- **Database Name**: Any name you choose
- **User**: `root`
- **Password**: `1`

### PostgreSQL

Default connection credentials:

- **Host Name**: `espocrm-postgres`
- **Database Name**: `espocrm`
- **User**: `espocrm`
- **Password**: `espo_password`

### PhpMyAdmin Setup

#### 1. Expose the MySQL port

Add a port mapping to `docker-compose.yml`:

```yaml
espocrm-mysql:
    .....
    ports:
      - 8033:3306
```

#### 2. Configure phpMyAdmin

Edit the `config.inc.php` file in your phpMyAdmin directory:

```php
$i++;
$cfg['Servers'][$i]['verbose'] = 'Docker: espocrm-mysql';
$cfg['Servers'][$i]['host'] = '127.0.0.1';
$cfg['Servers'][$i]['port'] = 8033;
$cfg['Servers'][$i]['socket'] = '';
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = '';
```

## Cron Job Setup (optional)

By default, cron jobs are executed by the daemon container. If you need to run them from the host machine instead, follow the instructions below.

### Configure from the host machine

Add the following cron job to execute EspoCRM scheduled tasks every minute:

```bash
* * * * * /usr/bin/docker exec --user www-data -i espocrm-php /bin/bash -c "cd /var/www/html; php cron.php" > /dev/null 2>&1
```

## WebSocket Configuration

### 1. Add WebSocket service to Docker Compose

Add the following service definition to your `docker-compose.yml`:

```yaml
espocrm-websocket:
    image: espocrm/espocrm
    container_name: espocrm-websocket
    volumes:
      - ./html:/var/www/html
    restart: always
    entrypoint: php websocket.php
    ports:
      - 8081:8080
```

### 2. Configure EspoCRM for WebSocket

Add the following settings to your `data/config.php`:

```php
'useWebSocket' => true,
'webSocketUrl' => 'ws://localhost:8081',
'webSocketZeroMQSubscriberDsn' => 'tcp://*:7777',
'webSocketZeroMQSubmissionDsn' => 'tcp://espocrm-websocket:7777',
```

### 3. Restart the containers

Stop and remove existing containers:

```bash
docker compose down -v
```

Rebuild and start the containers:

```bash
docker compose up -d --build "$@"
```

## Running Tests

### Unit tests

Execute unit tests with the following command:

```bash
/usr/bin/docker exec --user www-data phpunit --bootstrap vendor/autoload.php tests/unit
```

### Integration tests

Execute integration tests with the following command:

```bash
/usr/bin/docker exec --user www-data phpunit --bootstrap vendor/autoload.php tests/integration
```
