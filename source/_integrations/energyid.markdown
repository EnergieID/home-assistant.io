---
title: EnergyID
description: Instructions for integrating EnergyID energy management platform with Home Assistant.
ha_category:
  - Energy
  - Hub
ha_iot_class: Cloud Push
ha_domain: energyid
ha_integration_type: integration
ha_release: 2023.6
ha_config_flow: true
ha_codeowners:
  - '@JrtPec'
  - '@molier'
related:
  - docs: /integrations/energy
    title: Energy management documentation
  - url: https://www.energyid.eu/
    title: EnergyID Platform
---

The **EnergyID** integration enables seamless transfer of energy metrics from Home Assistant to [EnergyID](https://www.energyid.eu/), a professional energy management platform for long-term tracking and analysis.

## About EnergyID

EnergyID provides tools for:

- Historical energy consumption analysis
- Solar production tracking across multiple sites
- Utility bill management and validation
- Carbon footprint reporting
- Multi-property energy comparisons

## Prerequisites

You will need:

1. Active EnergyID account with webhook permissions
2. Webhook URL from EnergyID's integration settings
3. Sensor entity in Home Assistant with valid numerical readings
4. Configured metric type in EnergyID that matches your sensor data

## How you can use this integration

- **Long-term trend analysis**: Track energy patterns over years
- **Solar performance monitoring**: Compare PV output across seasons
- **Multi-site management**: Aggregate data from multiple properties
- **Bill validation**: Cross-check utility invoices with actual consumption
- **Sustainability reporting**: Generate carbon emission reports

{% include integrations/config_flow.md %}

## Configuration Options

{% configuration_basic %}
EnergyID Webhook URL:
  description: "Unique URL provided by EnergyID's webhook configuration (required)"
Home Assistant Entity ID:
  description: "Sensor entity to monitor for state changes (required)"
EnergyID Metric:
  description: "Measurement category from EnergyID's supported metrics (required)"
EnergyID Metric Kind:
  description: "Data interpretation method - cumulative/total/delta/gauge (required)"
Unit of Measurement:
  description: "Units matching EnergyID's system (kWh, m³, °C, etc.) (required)"
EnergyID Data Interval:
  description: "Data resolution (P1D, PT1H, PT15M etc.) - defaults to daily (optional)"
Upload Interval (seconds):
  description: "Minimum time between uploads - defaults to 300s (5 mins) (optional)"
{% endconfiguration_basic %}

## Key Features

### Data Handling

- **Smart Upload Throttling**: Minimum 5-minute interval between updates
- **Value Validation**: Automatic filtering of non-numeric states
- **Connection Resilience**:
  - 3 retry attempts with exponential backoff
  - Automatic reconnection after network failures
  - Configurable retry intervals (1s, 2s, 4s)

### Metric Types

- **Cumulative**: For ever-increasing counters (e.g., total kWh)
- **Total**: Interval-based consumption (e.g., daily usage)
- **Delta**: Change between measurements
- **Gauge**: Instantaneous readings (e.g., temperature)

### Data Resolution Options

| Interval | Description | Subscription Required |
|----------|-------------|-----------------------|
| PT5M     | 5-minute    | Premium               |
| PT15M    | 15-minute   | Premium               |
| PT1H     | Hourly      | Free                  |
| P1D      | Daily       | Free                  |

## Installation Guide

### Obtaining Webhook URL

1. Log in to your EnergyID account
2. Navigate to **Integrations** → **Incoming Webhooks**
3. Create new webhook with:
   - Target record
   - Descriptive name
4. Copy the generated URL (format: `https://app.energyid.eu/integrations/WebhookIn/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)

### Home Assistant Setup

1. {% my config_flow_start title="Add EnergyID Integration" %} via Home Assistant UI
2. Paste Webhook URL when prompted
3. Select entity from dropdown list
4. Choose matching EnergyID metric type
5. Configure advanced options if needed

## Data Flow Management

### Update Behavior

- Immediate upload on state change
- Minimum 5-minute interval between updates (configurable)
- Last valid value persists during connection issues

### Connection Monitoring

- Automatic health checks every 15 minutes
- System log entries for connection status changes
- Visual indicator in Home Assistant UI

## Known Limitations

- **One-way synchronization**: Data flows only from Home Assistant → EnergyID
- **Premium features**: Sub-1hour intervals require EnergyID subscription
- **Entity limitations**: Each sensor requires separate webhook configuration
- **Data latency**: EnergyID typically processes data within 5-15 minutes

## Troubleshooting

### Common Issues

- **`Invalid webhook URL`**: Regenerate in EnergyID and update configuration
- **`Non-numeric state`**: Check sensor's unit_of_measurement
- **`Connection lost`**:
  1. Verify network connectivity
  2. Check EnergyID service status
  3. Validate authentication credentials

### Diagnostic Tools

- Enable debug logging:

  ```yaml
  logger:
    default: info
    logs:
      homeassistant.components.energyid: debug
  ```

- Review `energyid_*.log` files in Home Assistant config directory

## Removing the Integration

1. Go to {% my integrations title="**Settings** > **Devices & Services**" %}
2. Locate EnergyID integration card
3. Select **Configure**
4. Choose **Delete Integration**

{% include integrations/remove_device_service.md %}

## Advanced Usage

### Automation Examples

```yaml
# Force immediate upload
service: energyid.force_upload
data:
  entity_id: sensor.solar_production

# Change update interval
service: energyid.set_update_interval
data:
  entity_id: sensor.grid_consumption
  interval: 60  # 1 minute
```

### Energy Dashboard Integration

Combine with Home Assistant's [Energy Dashboard](/integrations/energy) for:

- Real-time local visualization
- Short-term trend analysis
- System health monitoring

---

*This integration is maintained by the EnergyID core team. For feature requests or issues, please [open a GitHub ticket](https://github.com/energyid/homeassistant-integration/issues).*
