بسیار عالی. با توجه به لیست کامل متریک‌هایی که ارائه کردید، فایل `Readme.md` را به‌روزرسانی می‌کنم تا به طور دقیق قابلیت‌های این مجموعه تنظیمات را منعکس کند.

در ادامه نسخه به‌روز شده‌ی فایل `Readme.md` آمده است.

---

# QNAP SNMP Exporter for Prometheus

This repository contains a set of configuration files to monitor QNAP storage devices using Prometheus and Grafana. It utilizes the Prometheus SNMP Exporter to collect a wide range of metrics, providing deep insights into your QNAP's performance and health.

This configuration is based on the OIDs provided in the `snmp.yml` file and includes metrics for system health, storage components (disks, volumes, RAID, pools), and services like iSCSI.

**Note:** These configurations do not include metrics from `IF-MIB`, so network interface statistics are not collected.

## Collected Metrics

This configuration collects the following metrics, categorized for clarity:

### System Health & Information
-   **System Info:** Device model, system name, firmware version, hardware revision, and manufacturer.
-   **Uptime:** System uptime from multiple MIBs.
-   **CPU:** Current CPU usage percentage and temperature.
-   **Memory:** Total and free system memory.
-   **Temperature:** System and CPU temperatures.
-   **Fans:** Speed (RPM) and status for each system fan.
-   **Power:** Status of the power supply units.

### Hard Disk (HD) Metrics
-   **Disk Info:** Model name for each disk.
-   **Disk Status:** General status (`Ready`, `noDisk`, etc.) and S.M.A.R.T. status (`Good`, `Warning`, etc.).
-   **Disk Temperature:** Temperature for each individual hard disk.
-   **Disk Capacity:** Total capacity for each disk.
-   **Hot Spare:** Status indicating if a disk is configured as a hot spare.

### Storage Metrics
-   **Storage Pools:** Total capacity, free space, and status for each storage pool.
-   **RAID Volumes:** Total capacity, free space, current state, and RAID level for each volume.
-   **System Volumes:** Total and free space, plus filesystem type and status for each logical volume.

### iSCSI & LUN Metrics
-   **iSCSI Service:** Status of the main iSCSI service.
-   **iSCSI Targets:** Connection status for each iSCSI target.
-   **LUNs:**
    -   Capacity and used space percentage.
    -   Status (e.g., `Ready`, `Unmapping`).
    -   Mapping and backup status.

### Performance Metrics
-   **Disk I/O:** Total disk IOPS (Input/Output Operations Per Second).
-   **Disk Latency:** Average disk latency in milliseconds.

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

-   **`snmp.yml`**: The configuration for the Prometheus SNMP Exporter. It defines the specific OIDs to be scraped from the QNAP device, covering system health, storage, performance, and more.
-   **`alerts.yml`**: Contains a set of alerting rules for Prometheus. These rules will trigger alerts for conditions like high CPU usage, low disk space, or high temperatures.
-   **`grafana_dashboard.json`**: A JSON model for a Grafana dashboard. It provides a visual representation of the metrics collected from your QNAP device.

## Contributing

Contributions are welcome! If you have any improvements or new features to suggest, feel free to open an issue or create a pull request.

## License

This project is licensed under the MIT License.
