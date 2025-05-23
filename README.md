# Kubernetes Ingress Monitor Controller

## Overview

Kuma ingress watcher is a Kubernetes controller designed to automatically monitor `Kubernetes Ingress` and `Traefik Ingressroutes` in a Kubernetes cluster and create corresponding monitors in `Uptime Kuma`. It provides seamless integration between Kubernetes ingress resources and Uptime Kuma monitoring, allowing for easy and efficient monitoring of web services deployed on Kubernetes.

## Features

- Automatically creates, updates, and deletes monitors in Uptime Kuma for `Kubernetes Ingress` and `Traefik Ingressroutes`.
- Supports both single and multiple routes per Ingress resource.
- Customizable monitors by annotating Ingressroutes and Ingress.
- **File-based Monitor Configuration**: Define static monitors using a YAML file.

## Installation

### Requirements

- Python 3.6+
- Kubernetes cluster
- Uptime Kuma instance

## Configuration

### Environment variables
Before running the controller, make sure to configure the following environment variables:

- `UPTIME_KUMA_URL`: The URL of your Uptime Kuma instance.
- `UPTIME_KUMA_USER`: The username for authenticating with Uptime Kuma.
- `UPTIME_KUMA_PASSWORD`: The password for authenticating with Uptime Kuma.
- `WATCH_INGRESSROUTES`: Set to `True` to enable monitoring of Traefik IngressRoutes.
- `WATCH_INGRESS`: Set to `True` to enable monitoring of Kubernetes Ingress resources.
- `WATCH_INTERVAL`: Interval in seconds between each check for changes in Ingress or IngressRoutes (default is `10` seconds).
- `USE_TRAEFIK_V3_CRD_GROUP`: Whether to use Traefik V3 API CRD group (`traefik.io`); default to `False`.
- `ENABLE_FILE_MONITOR`: Set to `True` to enable file-based monitor configuration.
- `FILE_MONITOR_PATH`: The path to the YAML file containing monitor definitions (default `/etc/kuma-controller/monitors.yaml`).

### File-Based Monitor Configuration

If `ENABLE_FILE_MONITOR` is set to `True`, the controller will load monitor definitions from the YAML file specified by `FILE_MONITOR_PATH`.

#### Example YAML File

```yaml
  - name: example-monitor
    url: https://example.com
    type: http
    interval: 60
  - name: another-monitor
    url: https://another.com
    type: tcp
    interval: 120
```

#### Validation Rules

- Each monitor entry must include `name` and `url` fields.
- Optional fields: `type` (default: `http`), `interval` (default: `60`), `headers` (default `{}`).

The controller will create or update these monitors in Uptime Kuma upon startup.

### Annotations for Uptime Kuma Autodiscovery

These annotations apply to both Kubernetes Ingress and Traefik Ingressroutes resources, allowing you to customize the behavior of Uptime Kuma monitors for each:

1. **`uptime-kuma.autodiscovery.probe.interval`**
   - Sets the probing interval in seconds for the monitor.
   - **Type:** Integer
   - **Default:** `60`
   - **Example:** `uptime-kuma.autodiscovery.probe.interval: 60`

2. **`uptime-kuma.autodiscovery.probe.name`**
   - Name of the monitor in Uptime Kuma.
   - **Type:** String
   - **Default:** `${name}-${namespace}`
   - **Example:** `uptime-kuma.autodiscovery.probe.name: my-monitor`

3. **`uptime-kuma.autodiscovery.probe.enabled`**
   - Enables or disables monitoring for this resource.
   - **Type:** Boolean (accepted values: 'true' or 'false')
   - **Default:** `true`
   - **Example:** `uptime-kuma.autodiscovery.probe.enabled: true`

4. **`uptime-kuma.autodiscovery.probe.type`**
   - Probe type for the monitor (e.g., HTTP, TCP, etc.).
   - **Type:** String
   - **Default:** `http`
   - **Example:** `uptime-kuma.autodiscovery.probe.type: http`

5. **`uptime-kuma.autodiscovery.probe.headers`**
   - Optional HTTP headers to include in the probe request.
   - **Type:** String (HTTP headers format)
   - **Default:** `null` (no headers)
   - **Example:** `uptime-kuma.autodiscovery.probe.headers: {"Authorization": "Bearer token"}`

6. **`uptime-kuma.autodiscovery.probe.host`**
   - Force the host for the probe. **WARNING: Be careful with this parameter if you are using multiple hosts in the same Ingress object.**
   - **Type:** String
   - **Default:** `null` (host grabbed from ingress)
   - **Example:** `uptime-kuma.autodiscovery.probe.host: example.com`

7. **`uptime-kuma.autodiscovery.probe.path`**
   - URL sub-path for the probe.
   - **Type:** String
   - **Default:** `null`
   - **Example:** `uptime-kuma.autodiscovery.probe.path: /tintin`

8. **`uptime-kuma.autodiscovery.probe.port`**
   - Port to use for the probe.
   - **Type:** Integer
   - **Default:** `null` (default port for the protocol)
   - **Example:** `uptime-kuma.autodiscovery.probe.port: 8080`

9. **`uptime-kuma.autodiscovery.probe.method`**
   - HTTP method to use for the probe (GET, POST, etc.).
   - **Type:** String
   - **Default:** `GET`
   - **Example:** `uptime-kuma.autodiscovery.probe.method: GET`

### Complete Example

Here's an example of annotations configured in a Kubernetes Ingress Resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: my-namespace
  annotations:
    uptime-kuma.autodiscovery.probe.interval: 120
    uptime-kuma.autodiscovery.probe.name: my-monitor
    uptime-kuma.autodiscovery.probe.enabled: true
    uptime-kuma.autodiscovery.probe.type: http
    uptime-kuma.autodiscovery.probe.headers: '{"Authorization": "Bearer token"}'
    uptime-kuma.autodiscovery.probe.host: 'example.com'
    uptime-kuma.autodiscovery.probe.path: '/tintin'
    uptime-kuma.autodiscovery.probe.port: 8080
    uptime-kuma.autodiscovery.probe.method: GET
spec:
# Your Ingress route specification here
```

## Usage

Once the controller is running, it will automatically monitor any changes to Ingress resources in your Kubernetes cluster and create/update corresponding monitors in Uptime Kuma. Simply deploy your applications using Kubernetes Ingress, and the controller will take care of the rest!

If `ENABLE_FILE_MONITOR` is enabled, the controller will also create monitors defined in the specified YAML file.

## Important Notes

### Tag Addition Limitation

Currently, the addition of tags to monitors is not supported due to limitations in the Uptime Kuma API. Attempting to add tags through the controller may result in unexpected behavior or errors. Please refer to the Uptime Kuma documentation for updates on tag management capabilities.

### Custom Watcher for IngressRoutes

The Kubernetes event watcher (`watch`) does not provide specific details on creation, modification, or deletion events for IngressRoutes. To overcome this limitation, this controller implements a custom watcher mechanism that continuously monitors IngressRoutes and triggers appropriate actions based on changes detected. This custom solution ensures accurate monitoring and synchronization with Uptime Kuma configurations.

## Improvements

- **File-Based Monitor Configuration**: Define static monitors using a YAML file.
- **IngressRoute Version Selection**: You can now choose which version of IngressRoutes to watch. This allows you to customize the controller's behavior based on the version of Traefik you're using.

## Contributing

Contributions are welcome! If you have any ideas, suggestions, or bug reports, please open an issue or submit a pull request on GitHub.

## Running Unit Tests

To run unit tests for this project:

### Prerequisites

- Python 3.12 or higher installed.
- Poetry installed.

```bash
poetry install
poetry run pytest
```

### Pre-commit Hook

```bash
pre-commit install
pre-commit run --all-files
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
