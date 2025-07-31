# xdebug-docker-vscode

This repository provides a Docker-based PHP development environment with Xdebug configured for use in Visual Studio Code.

## Prerequisites

- Docker Engine and Docker Compose installed
- Visual Studio Code with the [PHP Debug extension](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug)

## Repository Structure

```
.
├── .docker
│   └── php-fpm
│       └── dev
│           └── xdebug.ini      # Xdebug configuration
├── Dockerfile                  # PHP + Xdebug setup
├── docker-compose.yml          # Service definitions
└── README.md                   # This file
```

## Installation & Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/yourusername/xdebug-docker-vscode.git
   cd xdebug-docker-vscode
   ```

2. **Build and run the containers**

   ```bash
   docker-compose up --build -d
   ```

3. **Verify services are running**

   ```bash
   docker-compose ps
   ```

## Customizing the Debug Port

By default, Xdebug connects on port `9005`. To use a different port:

- **Xdebug configuration**: In `.docker/php-fpm/dev/xdebug.ini`, update:

  ```ini
  xdebug.client_port = YOUR_PORT
  ```

- **Docker Compose mapping**: In `docker-compose.yml`, under the `php` service, change the port mapping:

  ```yaml
  ports:
    - "YOUR_PORT:YOUR_PORT"
  ```

- **VSCode launch settings**: In `.vscode/launch.json`, set the `port` field to match:

  ```json
  {
    "configurations": [
      {
        "port": YOUR_PORT
      }
    ]
  }
  ```

## Xdebug Configuration (`.docker/php-fpm/dev/xdebug.ini`)

```ini
zend_extension = xdebug.so

; Enable debug mode
xdebug.mode = debug

; Start debugging on each request
xdebug.start_with_request = yes

; Hostname or IP of your IDE
xdebug.client_host = host.docker.internal

; Port for Xdebug to connect back to VSCode
xdebug.client_port = 9005

; Do not override client_host via discovery
xdebug.discover_client_host = false

; IDE key (optional)
xdebug.idekey = VSCODE

; Log file for troubleshooting
xdebug.log = /tmp/xdebug.log
```

## Dockerfile

```dockerfile
FROM php:8.1-fpm

# Install and enable Xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

# Copy Xdebug configuration
COPY .docker/php-fpm/dev/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

# (Other PHP extensions and setup goes here...)
```

## Docker Compose (`docker-compose.yml`)

```yaml
version: '3.8'
services:
  php:
    build: .
    ports:
      - "9005:9005"   # Xdebug port mapping
    volumes:
      - ./:/var/www/html
    # (Other service settings)
```

## VSCode Debug Configuration (`.vscode/launch.json`)

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for XDebug",
      "type": "php",
      "request": "launch",
      "port": 9005,
      "log": true
    }
  ]
}
```

## Usage

1. Set breakpoints in your PHP code.
2. In VSCode, select **Listen for XDebug** and click **Start Debugging** (F5).
3. Trigger a request to your application in the browser or via CLI.
4. VSCode will pause execution at your breakpoints.

## Troubleshooting

- **View Xdebug logs** inside the container:
  ```bash
  docker-compose exec php tail -f /tmp/xdebug.log
  ```
- **Firewall settings**: Ensure your host allows inbound connections on the configured port.
- **Hostname resolution**: Verify `host.docker.internal` resolves inside the container:
  ```bash
  docker-compose exec php ping host.docker.internal
  ```

