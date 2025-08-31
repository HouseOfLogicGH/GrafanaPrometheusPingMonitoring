# Blackbox Exporter Setup

If you have a need to monitor specific applications within your environment, the Prometheus Black Box exporter allows you to probe either ports or endpoints and track the success and response times of these.

The Prometheus Blackbox Exporter can be found here:[https://github.com/prometheus/blackbox_exporter](https://github.com/prometheus/blackbox_exporter)

A demonstration video can be found here: [https://youtu.be/58LKuEhfF6E](https://youtu.be/58LKuEhfF6E)

## Configuration using Docker

To extend an existing Prometheus and Grafana stack, you can update the configuration as follows:

Create a directory and cd to it:
```
mkdir blackbox
cd blackbox
```
Launch nano to create a config file for the exporter:
```
nano blackbox.yml
```
Add the modules you wish to use to the config file - note that these are very simple. Please refer to the blackbox_exporter documentation for more complex configurations.
```
modules:
  # Module for HTTP probing
  http_simple:
    prober: http
    timeout: 5s
    http:
      # Valid HTTP status codes for success (2xx by default)
      valid_status_codes: [] # Empty means any 2xx status code is valid
      # No HTTP method specified, defaults to GET
      # No TLS config, assumes plain HTTP
      # No HTTP headers or other specific settings needed for a basic check

  # Module for TCP probing
  tcp_simple:
    prober: tcp
    timeout: 5s
    tcp:
      # No TLS config, assumes plain TCP connection
      # We just want to check if the port is open and connectable
      # send and expect are not needed for a basic connect check
```
You can now start the container using docker (which maps in the config file):
```
sudo docker run --rm \
  -d \
  -p 9115:9115 \
  --name blackbox_exporter \
  -v $(pwd):/config \
  quay.io/prometheus/blackbox-exporter:latest --config.file=/config/blackbox.yml
```
Next, it is best to check that your blackbox exporter container can reach your targets by polling it manually.

For example (where 192.168.2.10 is the Docker host and 192.168.2.194 is the target to be checked), use a browser or curl command to poll the following:

```
http://192.168.2.10:9115/probe?module=http_simple&target=192.168.2.194:9000

http://192.168.2.10:9115/probe?module=tcp_simple&target=192.168.2.194:445

```
When you are happy that you have the blackbox exporter working, you need to add the configuration you want to poll to Prometheus.

Add the following jobs to the "scrape_configs" section of your prometheus config file (full example listing in the prometheus subdirectory).

Note that the blackbox exporter needs the target as a parameter, which the prometheus scrape job provides by relabelling.

```
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_simple]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://192.168.2.194:9000    # Target to probe with http.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.2.10:9115  # The blackbox exporter's real hostname:port.

  - job_name: 'blackbox-tcp'
    metrics_path: /probe
    params:
      module: [tcp_simple]
    static_configs:
      - targets:
        - 192.168.2.195:445    # Target to probe with tcp
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.2.10:9115  # The blackbox exporter's real hostname:port.

  - job_name: 'blackbox_exporter'  # collect blackbox exporter's operational metrics.
    static_configs:
      - targets: ['192.168.2.10:9115']

```

With the configuration updated, you should now be able to restart the prometheus instance and check the targets are responding (the /targets endpoint of prometheus will show this).


## Grafana Info

The metric shown in the demonstration video is "probe_duration_seconds" and the label used is as follows:
```
{{job}}-{{instance}}
```