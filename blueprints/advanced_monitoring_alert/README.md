# Advanced Home Assistant Monitoring Blueprints

Enterprise-grade monitoring blueprints for Home Assistant that provide cloud-like alerting capabilities similar to Azure Monitor, AWS CloudWatch, and other professional monitoring solutions.

## 🚀 Features

### Core Monitoring Capabilities
- **Multiple Condition Types**: Above, below, between, outside range, equal, not equal, changed
- **Configurable Thresholds**: Primary and secondary thresholds for range-based conditions  
- **Evaluation Periods**: Set how long a condition must persist before triggering
- **Acknowledgeable Notifications**: Prevent escalations by acknowledging alerts (NEW!)

### Enterprise Alerting Features
- **Severity Levels**: Critical, High, Medium, Low, Info with appropriate notification settings
- **Escalation Policies**: Automatic escalation to additional contacts with acknowledgement checking
- **Multiple Notification Channels**: Mobile apps, email, Slack, and other services
- **Rich Notifications**: Platform-specific features like acknowledgement actions, priority, TTL
- **OK Notifications**: Automatic recovery notifications when conditions return to normal

### Operational Features  
- **Maintenance Windows**: Suppress alerts during planned maintenance
- **Metrics Collection**: Optional event logging for dashboards and analysis
- **Custom Tagging**: Add metadata for categorization and filtering
- **Template Support**: Dynamic titles and messages with Jinja2 templates
- **Global Acknowledgement System**: Centralized acknowledgement handling across all alarms

## 📋 Blueprints in This System

### 1. **Advanced Monitoring Alert** (`advanced_monitoring_alert.yaml`)
Main monitoring blueprint with enterprise features including acknowledgeable notifications.

### 2. **Global Acknowledgement Handler** (`global_acknowledgement_handler.yaml`)  
System-wide acknowledgement processor. Install once to enable acknowledgements across all alarms.

### 3. **Acknowledgement Cleanup** (`acknowledgement_cleanup.yaml`)
Periodic cleanup of old acknowledgements to prevent system bloat.

## 🔧 Quick Start

### Basic Installation (Traditional Monitoring)
1. Copy `advanced_monitoring_alert.yaml` to your blueprints folder
2. Create automations using the blueprint with `enable_acknowledgements: false`

### Full System Installation (With Acknowledgements)
1. **Follow the complete [Acknowledgement System Guide](ACKNOWLEDGEMENT_SYSTEM.md)**
2. Create the required input_text helper
3. Install all three blueprints
4. Create automations with acknowledgement features enabled

## 💡 Usage Examples

### Critical Temperature Monitor
```yaml
use_blueprint:
  path: advanced_monitoring_alert.yaml
  input:
    monitored_entity: sensor.server_room_temperature
    alarm_name: "Server Room Temperature"
    condition_type: "above"
    threshold_value: 25
    severity_level: "critical"
    enable_escalation: true
    escalation_delay: 10
```

### Disk Space Warning
```yaml
use_blueprint:
  path: advanced_monitoring_alert.yaml  
  input:
    monitored_entity: sensor.disk_use_percent
    alarm_name: "Disk Space Usage"
    condition_type: "above"
    threshold_value: 85
    evaluation_period: 5
    severity_level: "high"
```

### Network Connectivity Monitor
```yaml
use_blueprint:
  path: advanced_monitoring_alert.yaml
  input:
    monitored_entity: binary_sensor.internet_connectivity
    alarm_name: "Internet Connection"
    condition_type: "equal"
    threshold_value: 0  # 0 = offline
    enable_ok_notifications: true
```

## 📊 Condition Types Explained

| Condition | Description | Example Use Case |
|-----------|-------------|------------------|
| **Above** | Value > threshold | Temperature alerts, CPU usage |
| **Below** | Value < threshold | Battery level, disk space |
| **Between** | threshold ≤ value ≤ high_threshold | Optimal temperature range |
| **Outside** | value < threshold OR value > high_threshold | Humidity out of range |
| **Equal** | Value == threshold | Binary sensor states |
| **Not Equal** | Value != threshold | Device status changes |
| **Changed** | Value changed from previous | Any state change detection |

## 🎚️ Severity Levels

| Level | Priority | Importance | TTL | Use Case |
|-------|----------|------------|-----|----------|
| **Critical** | High | High | No expiry | System down, security breach |
| **High** | High | High | 5 min | Service degraded, high resource usage |
| **Medium** | Normal | Default | 10 min | Warning thresholds, maintenance needed |
| **Low** | Normal | Low | 30 min | Informational, minor issues |
| **Info** | Min | Min | 1 hour | Status updates, non-urgent notifications |

## 🔄 Escalation Flow

1. **Initial Alert**: Sent to primary devices immediately
2. **Wait Period**: Configurable delay (5-120 minutes)  
3. **Condition Check**: Verify alert is still active and not acknowledged
4. **Escalation**: Send to escalation devices with "ESCALATED" prefix
5. **Enhanced Priority**: Escalated alerts use high priority settings

## 🛠️ Maintenance Windows

Enable maintenance mode to suppress alerts during planned work:

1. Create an input_boolean helper:
   ```yaml
   input_boolean:
     server_maintenance:
       name: "Server Maintenance Mode"
       icon: mdi:wrench
   ```

2. Reference in blueprint:
   ```yaml
   enable_maintenance_mode: true
   maintenance_entity: input_boolean.server_maintenance
   ```

3. Toggle maintenance mode via UI or automation

## 📈 Metrics Collection

When enabled, the blueprint emits `alarm_metrics` events with:
- Alarm name and entity
- Severity and condition details  
- Current value and threshold
- State (ALARM/OK) and timestamp
- Custom tags for categorization

Use these events to build dashboards, generate reports, or feed external monitoring systems.

## 🏷️ Custom Tags

Add metadata in JSON format for categorization:
```json
{
  "environment": "production",
  "system": "hvac", 
  "location": "server_room",
  "team": "infrastructure"
}
```

Tags are included in metrics events and can be used for:
- Filtering and grouping alerts
- Dashboard categorization  
- Integration with external systems
- Automated ticketing systems

## 🔍 Troubleshooting

### Empty Device Names
If `notify_names` is empty, check:
1. Device IDs are correct in your configuration
2. Mobile app integration is properly set up
3. Device names don't contain special characters

### Escalation Not Working  
Verify:
1. Escalation is enabled in blueprint config
2. Escalation devices are specified
3. Original condition is still true after delay
4. Maintenance mode is not active

### Notifications Not Received
Check:
1. Mobile app notification settings
2. Home Assistant companion app permissions
3. Do Not Disturb settings on devices
4. Network connectivity between HA and devices

## 🎯 Best Practices

### Threshold Setting
- Start with conservative thresholds and adjust based on experience
- Use evaluation periods to avoid false alarms from brief spikes
- Consider using "between" conditions for optimal ranges

### Notification Management  
- Use appropriate severity levels to control notification behavior
- Enable OK notifications for critical alerts only
- Set up escalation for unattended critical systems

### Maintenance Planning
- Always use maintenance windows for planned work
- Test escalation paths during low-impact periods
- Document threshold rationale for future reference

## 📚 Advanced Configurations

### Multiple Threshold Alerts
Create separate automations with different severity levels:
```yaml
# Warning at 80%
threshold_value: 80
severity_level: "medium"

# Critical at 95%  
threshold_value: 95
severity_level: "critical"
```

### Time-Based Conditions
Use templates for time-sensitive alerts:
```yaml
notification_message: |
  {% if now().hour >= 22 or now().hour <= 6 %}
  🌙 AFTER HOURS ALERT 🌙
  {% endif %}
  {{ monitored_entity }} is {{ states(monitored_entity) }}
```

### Environmental Correlation
Reference multiple sensors in messages:
```yaml
notification_message: |
  Server room conditions:
  - Temperature: {{ states('sensor.server_room_temperature') }}°C
  - Humidity: {{ states('sensor.server_room_humidity') }}%
  - Power: {{ states('sensor.ups_load') }}%
```

## 🤝 Contributing

This project follows Home Assistant blueprint best practices. When contributing:

1. Test blueprints thoroughly before submitting
2. Follow YAML formatting standards
3. Include example configurations
4. Document any new features or breaking changes
5. Ensure backward compatibility when possible

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For issues and questions:
1. Check the troubleshooting section above
2. Review Home Assistant logs for errors
3. Verify entity states and device connectivity
4. Test with simplified configurations first

---

*Built with ❤️ for the Home Assistant community*
