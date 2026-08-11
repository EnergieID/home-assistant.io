---
title: EnergyID
description: Instructions on how to integrate EnergyID into Home Assistant to send your sensor data to the EnergyID platform.
ha_category:
  - Energy
ha_iot_class: Cloud Polling
ha_domain: energyid
ha_integration_type: service
ha_config_flow: true
ha_codeowners:
  - '@JrtPec'
  - '@Molier'
ha_release: 2025.12
ha_quality_scale: silver
---

The **EnergyID** {% term integration %} connects your Home Assistant to [EnergyID](https://www.energyid.eu/), a cloud platform for energy monitoring and optimization. This integration uploads your Home Assistant sensor data and provides advanced analytics and performance tracking for solar, battery, energy consumption, and more. It can also receive EnergyID **directives** (steering signals such as the Stroomplanner), exposing each one as a sensor you can use in dashboards and automations.

## Prerequisites

1. An active account on [EnergyID](https://www.energyid.eu/).

2. A **Provisioning Key** and **Provisioning Secret** generated from your EnergyID portal. These credentials allow Home Assistant to securely connect to your account.

- For detailed instructions, refer to the [official EnergyID Home Assistant documentation](https://help.energyid.eu/en/apps/home-assistant/).


{% include integrations/config_flow.md %}

During the setup, you will be prompted for the following information:

{% configuration_basic %}
Provisioning Key:
  description: The Provisioning Key obtained from your EnergyID portal.

Provisioning Secret:
  description: The Provisioning Secret associated with your key, obtained from your EnergyID portal.
{% endconfiguration_basic %}

### Initial setup steps

1. After adding the integration, you will first be asked to enter your **Provisioning Key** and **Secret**.
    <p class='img'><img src='/images/integrations/energyid/image-2.png' alt="Screenshot of the EnergyID connection screen in Home Assistant, asking for Provisioning Key and Secret."/></p>
2. If this is the first time you are connecting this Home Assistant instance, you will be directed to the EnergyID website to **claim** your device. This step links your Home Assistant instance to a specific record (e.g., your house) in your EnergyID account.
3. Once claimed, the setup will automatically complete.
4. Right after setup, you are asked whether to **receive EnergyID directives**. This is optional and can be changed later through the integration's options.

## Managing sensor mappings

After the initial setup, you can manage which Home Assistant sensors send data to EnergyID.

1. Go to {% my integrations title="**Settings > Devices & services**" %}.
2. Find the EnergyID integration and select **Configure**.

From here, you can add new sensor mappings. When adding a mapping, you will be asked for the following:

{% configuration_basic %}
Home Assistant sensor:
  description: Select the sensor entity from your Home Assistant instance whose data you want to send. The list is automatically filtered to suggest suitable numeric sensors.
{% endconfiguration_basic %}

<p class='img'><img src='/images/integrations/energyid/image-1.png' alt="Screenshot of the EnergyID configuration screen in Home Assistant, showing options to add and manage sensor mappings."/></p>

When you select a sensor, its `object_id` (the part of the entity ID after the dot) will be used as the **EnergyID Metric Key**. For example, mapping `sensor.total_active_power` will send data to EnergyID with the key `total_active_power`.

## Receiving directives

EnergyID records can be granted access to **directives**: steering signals, such as an energy community's Stroomplanner, that tell you when it is a good or bad moment to consume electricity.

When directives are enabled in the integration's options, every directive authorized for your record is discovered automatically, including directives granted later, and exposed as a sensor. The sensor state is one of five moments: *Very bad moment*, *Bad moment*, *Neutral*, *Good moment*, or *Very good moment*. The `next_change` and `next_state` attributes tell you when the signal will change next and to what, which makes them convenient automation triggers.

Directive access is managed on the EnergyID side; the integration polls for the current schedules every 5 minutes.

{% configuration_basic %}
Receive EnergyID directives:
  description: When enabled, an entity is created for every directive this record may access. Disable to remove the directive entities again.
{% endconfiguration_basic %}

## Actions

### Action `energyid.get_directive_schedule`

Returns the complete current schedule for the selected EnergyID directive entities, including one timestamped entry per interval with the signal, its color, and the raw value. Use this in scripts or custom dashboard cards that need the full day plan rather than only the current state.

| Data attribute | Optional | Description |
| -------------- | -------- | ----------- |
| `entity_id`    | no       | The EnergyID directive sensors to return schedules for. |

The response contains one schedule per targeted entity, keyed by entity ID.

## Data updates

Outgoing measurements use a push-based mechanism with batching:

- It listens for {% term state %} changes on your mapped sensors.
- When a sensor's value changes, the new value and timestamp are queued.
- The queued data is automatically sent to EnergyID in batches. The upload interval is determined by the policy received from EnergyID (typically every 60 seconds).

This is more efficient than traditional {% term polling %}, as it only sends data when there are new updates.

Incoming directives are {% term polling polled %} every 5 minutes.

## Use cases

1. Send anything in Home Assistant to EnergyID for long term storage/graphing and detailed analysis.
2. Use EnergyID's features to compare your energy usage against anonymized data from similar households and generate detailed reports.
3. Many more [advantages of EnergyID](https://help.energyid.eu/en/using-energyid/getting-started-with-energyid/) and a brief intro can be found.

## Troubleshooting

If you're experiencing issues with your EnergyID integration, please try these general troubleshooting steps:

### Data not appearing in EnergyID

1. Verify that the linked entities from your Home Assistant are actually being updated and are not just stationary or stale. Not all entities send out changes frequently.
2. Make sure that your entities are correctly mapped in the integration settings.
3. Try reloading the EnergyID integration or even try reloading the integration of the entity which is not updating data in EnergyID
4. Be sure to check Home Assistant logs for any errors or issues, or turn on debugging for the integration to receive more info on its workings.{% my logs title="**Settings > System > Logs**" %}
