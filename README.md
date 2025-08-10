# QNAP SNMP Exporter for Prometheus

This repository contains a set of configuration files to monitor QNAP storage devices using Prometheus and Grafana. It utilizes the Prometheus SNMP Exporter to collect metrics.

## Features

-   **System Health:** Monitor CPU usage, memory usage, and system temperature.
-   **Storage Monitoring:** Keep track of volume usage (total, free, and used space) and disk temperatures.
-   **Alerting:** Pre-configured alert rules for critical conditions.
-   **Visualization:** A ready-to-use Grafana dashboard to visualize all the metrics.

**Note:** These configurations do not include metrics from `IF-MIB`, so network interface statistics are not collected.

## Prerequisites

Before you begin, ensure you have the following components set up and running:

-   A QNAP storage device with SNMP enabled.
-   A running instance of [Prometheus](https://prometheus.io/).
-   A running instance of [Prometheus SNMP Exporter](https://github.com/prometheus/snmp_exporter).
-   A running instance of [Grafana](https://grafana.com/).
-   [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) configured with Prometheus to handle alerts.

## Configuration

### 1. Enable SNMP on your QNAP device

1.  Log in to your QNAP's web interface (QTS).
2.  Go to **Control Panel > Network & File Services > SNMP**.
3.  Enable the SNMP service.
4.  It is recommended to use SNMPv2/v3 for better security. Configure a community string if you are using SNMPv2c.

### 2. Configure the SNMP Exporter

The `snmp_exporter` needs the `snmp.yml` file to know which metrics to scrape from your QNAP device.

1.  Copy the `snmp.yml` file from this repository to your SNMP Exporter configuration directory.
2.  Start the SNMP Exporter, pointing to this configuration file. For example:

    ```bash
    ./snmp_exporter --config.file="snmp.yml"
    ```

### 3. Configure Prometheus to Scrape the SNMP Exporter

Add the following job to your `prometheus.yml` configuration file to start scraping metrics from your QNAP device via the SNMP Exporter.

```yaml
scrape_configs:
  - job_name: 'qnap'
    static_configs:
      - targets:
        - <YOUR_QNAP_IP_ADDRESS>  # Replace with your QNAP's IP address
    metrics_path: /snmp
    params:
      module: [qnap]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <YOUR_SNMP_EXPORTER_IP_ADDRESS>:9116  # Replace with your SNMP Exporter's address
```

Reload your Prometheus configuration after making the changes.

### 4. Configure Prometheus Alerting Rules

The `alerts.yml` file contains predefined alert rules.

1.  Copy the `alerts.yml` file to your Prometheus rules directory.
2.  Add the following lines to your `prometheus.yml` to load the alert rules:

    ```yaml
    rule_files:
      - "alerts.yml"
    ```

3.  Reload your Prometheus configuration. Alerts will be sent to your configured Alertmanager instance.

### 5. Import the Grafana Dashboard

A pre-built Grafana dashboard is available in this repository to visualize the collected metrics.

1.  Copy the JSON from the `grafana_dashboard.json` file.
2.  In Grafana, go to **Dashboards > Import**.
3.  Paste the JSON into the text area or upload the file.
4.  Select your Prometheus data source and click **Import**.

## Files in this Repository

-   **`snmp.yml`**: The configuration for the Prometheus SNMP Exporter. It defines the OIDs to be scraped from the QNAP device.
-   **`alerts.yml`**: Contains a set of alerting rules for Prometheus. These rules will trigger alerts for conditions like high CPU usage, low disk space, or high temperatures.
-   **`grafana_dashboard.json`**: A JSON model for a Grafana dashboard. It provides a visual representation of the metrics collected from your QNAP device.

## Contributing

Contributions are welcome! If you have any improvements or new features to suggest, feel free to open an issue or create a pull request.

## License

This project is licensed under the MIT License.
