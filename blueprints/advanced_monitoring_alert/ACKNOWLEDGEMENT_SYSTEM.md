# Advanced Home Assistant Monitoring System with Acknowledgements

Complete enterprise-grade monitoring system for Home Assistant with acknowledgeable notifications, escalation policies, and maintenance windows.

## 🏗️ For high-volume environments, you can create multiple handlers:
```yaml
# Create separate helpers for different systems
input_text:
  infrastructure_acknowledgements:
    max: 255
    initial: "{}"
  application_acknowledgements:
    max: 255
    initial: "{}"
```

This monitoring system consists of **three blueprints** that work together:

1. **Advanced Monitoring Alert** - Main monitoring blueprint (enhanced)
2. **Global Acknowledgement Handler** - Handles all acknowledgement actions system-wide
3. **Acknowledgement Cleanup** - Periodic cleanup of old acknowledgements

## 📋 **Installation Guide**

### **Step 1: Create Required Helper**

> ⚠️ **Storage Limitation**: Home Assistant input_text helpers are limited to 255 characters. This means you can store approximately 3-5 acknowledgements simultaneously. For high-volume environments, consider using multiple helpers for different alarm categories or shorter alarm names.

Create a single input_text helper that will store acknowledgement state for ALL alarms:

#### Via UI:
1. Go to **Settings → Devices & Services → Helpers**
2. Click **Create Helper → Text**
3. Configure:
   - **Name**: `Global Alarm Acknowledgements`
   - **Entity ID**: `input_text.global_alarm_acknowledgements`
   - **Maximum length**: `255`
   - **Initial value**: `{}`

#### Via YAML (configuration.yaml):
```yaml
input_text:
  global_alarm_acknowledgements:
    name: "Global Alarm Acknowledgements"
    max: 255
    initial: "{}"
    icon: mdi:bell-check
```

### **Step 2: Install Blueprint System**

#### 2.1 Install Global Acknowledgement Handler (REQUIRED for acknowledgements)
1. Copy `global_acknowledgement_handler.yaml` to your blueprints folder
2. Create automation using this blueprint:
   - **Global Acknowledgement Helper**: `input_text.global_alarm_acknowledgements`
   - **Clear Acknowledged Notifications**: `true`
   - **Log Acknowledgements**: `true`

#### 2.2 Install Acknowledgement Cleanup (RECOMMENDED)
1. Copy `acknowledgement_cleanup.yaml` to your blueprints folder  
2. Create automation using this blueprint:
   - **Global Acknowledgement Helper**: `input_text.global_alarm_acknowledgements`
   - **Cleanup Schedule**: `02:00:00` (daily at 2 AM)
   - **Retention Period**: `7` days

#### 2.3 Use Enhanced Monitoring Blueprint
1. Copy `advanced_monitoring_alert.yaml` to your blueprints folder
2. Create monitoring automations with acknowledgement features enabled

## 🎯 **Usage Examples**

### **Basic Monitoring with Acknowledgements**
```yaml
alias: "Temperature Monitor with Acknowledgement"
use_blueprint:
  path: advanced_monitoring_alert.yaml
  input:
    monitored_entity: sensor.server_room_temperature
    alarm_name: "Server Room Temperature"
    condition_type: "above"
    threshold_value: 25
    severity_level: "critical"
    enable_acknowledgements: true
    acknowledgement_timeout: 30
    enable_escalation: true
    escalation_delay: 15
    primary_notify_devices:
      - device_id_1
    escalation_notify_devices:
      - device_id_2
```

### **Traditional Monitoring (No Acknowledgements)**
```yaml
alias: "Simple Disk Monitor"
use_blueprint:
  path: advanced_monitoring_alert.yaml
  input:
    monitored_entity: sensor.disk_use_percent
    alarm_name: "Disk Space"
    condition_type: "above"
    threshold_value: 85
    enable_acknowledgements: false  # Traditional behavior
    primary_notify_devices:
      - device_id_1
```

## 🔄 **How Acknowledgements Work**

### **Flow Overview:**
1. **Alert Triggered** → Monitoring blueprint sends notification with "Acknowledge" button
2. **User Acknowledges** → Mobile app sends action event to Home Assistant
3. **Global Handler** → Catches event, stores acknowledgement in JSON helper
4. **Escalation Prevention** → Monitoring blueprint checks acknowledgement before escalating
5. **Automatic Cleanup** → Old acknowledgements are cleaned up periodically

### **Acknowledgement States:**
- **Active**: Recently acknowledged, prevents escalation
- **Expired**: Acknowledgement timeout reached, escalation can proceed
- **Missing**: No acknowledgement found, escalation can proceed

### **JSON Helper Structure:**
```json
{
  "ack_server_temp_1694467200": {
    "timestamp": "2024-09-11T14:00:00.000000+00:00",
    "device": "mobile_app_johns_phone",
    "tag": "server_temp",
    "automation": "automation.global_alarm_acknowledgement_handler"
  }
}
```

## ⚙️ **Configuration Options**

### **Advanced Monitoring Alert Blueprint**

#### **Acknowledgement Settings:**
- **Enable Acknowledgeable Notifications**: `true/false`
- **Global Acknowledgement Helper**: Entity ID of the helper
- **Acknowledgement Timeout**: How long acknowledgements remain valid (5-480 minutes)

#### **Monitoring Settings:**
- **Entity to Monitor**: Any Home Assistant entity
- **Condition Type**: above, below, between, outside, equal, not_equal, changed
- **Threshold Values**: Primary and secondary thresholds
- **Evaluation Period**: How long condition must persist (0-60 minutes)

#### **Severity & Escalation:**
- **Severity Level**: critical, high, medium, low, info
- **Enable Escalation**: Escalate if not acknowledged
- **Escalation Delay**: Time to wait before escalating (5-120 minutes)

#### **Notification Channels:**
- **Primary Devices**: Mobile devices for immediate notifications
- **Escalation Devices**: Additional devices for escalated alerts
- **Additional Services**: Email, Slack, etc.

#### **Advanced Options:**
- **OK Notifications**: Send recovery notifications
- **Metrics Collection**: Log events for analysis
- **Custom Tags**: JSON metadata for categorization
- **Maintenance Windows**: Suppress alerts during maintenance

### **Global Acknowledgement Handler Blueprint**

#### **Settings:**
- **Global Acknowledgement Helper**: The input_text helper entity
- **Clear Acknowledged Notifications**: Auto-clear notifications when acknowledged
- **Log Acknowledgements**: Log acknowledgement events to logbook

### **Acknowledgement Cleanup Blueprint**

#### **Settings:**
- **Global Acknowledgement Helper**: The input_text helper entity
- **Cleanup Schedule**: When to run cleanup (time selector)
- **Retention Period**: How many days to keep acknowledgements (1-30 days)

## 🔍 **Troubleshooting**

### **Acknowledgements Not Working**

#### Check Helper Exists:
```yaml
# Test in Developer Tools → Templates
{{ states('input_text.global_alarm_acknowledgements') }}
# Should return: {} (not 'unknown' or 'unavailable')
```

#### Check Global Handler:
- Verify Global Acknowledgement Handler automation is enabled
- Check automation logs for errors
- Test with Developer Tools → Events → Listen for `mobile_app_notification_action`

#### Check Monitoring Automation:
- Verify `enable_acknowledgements: true` in automation config
- Check for persistent notifications about missing helper
- Review automation traces for acknowledgement logic

### **Mobile App Service Errors**

If you see errors like `"Action notify.mobile_app_xyz not found"`:

#### Check Device Name Resolution:
```yaml
# In Developer Tools → Templates, test with your device_id:
{% set device_id = "YOUR_DEVICE_ID_HERE" %}
{% set device_name = device_attr(device_id, 'name') %}
{% set clean_name = device_name | lower | regex_replace('[^a-z0-9]', '_') %}
Device Name: {{ device_name }}
Service Name: mobile_app_{{ clean_name }}
```

#### Find Your Mobile App Service:
```yaml
# List all mobile app services in Developer Tools → Templates:
{% for entity in states.notify %}
  {% if entity.entity_id.startswith('notify.mobile_app_') %}
    - {{ entity.entity_id }}
  {% endif %}
{% endfor %}
```

#### Common Issues:
- **Device name mismatch**: Home Assistant may clean device names differently
- **Service not registered**: Mobile app might not be fully configured
- **Device ID vs Service name**: These are different identifiers

#### Solution:
1. Enable logging in Global Handler to see debug info
2. Check logbook for "Acknowledgement Debug" entries
3. Compare resolved service name with actual available services
4. Adjust device naming in mobile app if needed

### **Helper Overflow**

If the helper reaches the 255 character limit:
- Check Cleanup automation is running
- Reduce retention period  
- Manually clear helper: Set value to `{}`
- Consider reducing alarm names to save space

### **Escalations Still Happening**

- Check acknowledgement timeout hasn't expired
- Verify Global Handler is processing acknowledgements
- Check helper contains recent acknowledgement for the alarm

## 📊 **Monitoring Dashboard**

### **Helper Status Card**
```yaml
type: entities
title: "Acknowledgement System Status"
entities:
  - entity: input_text.global_alarm_acknowledgements
    name: "Active Acknowledgements"
  - entity: automation.global_alarm_acknowledgement_handler
    name: "Acknowledgement Handler"
  - entity: automation.alarm_acknowledgement_cleanup
    name: "Cleanup Automation"
```

### **Acknowledgement History**
```yaml
type: logbook
title: "Recent Acknowledgements"
entities:
  - input_text.global_alarm_acknowledgements
hours_to_show: 24
```

## 🚀 **Advanced Usage**

### **Multiple Acknowledgement Handlers**

For high-volume environments, you can create multiple handlers:
```yaml
# Create separate helpers for different systems
input_text:
  infrastructure_acknowledgements:
    max: 8192
    initial: "{}"
  application_acknowledgements:
    max: 8192
    initial: "{}"
```

### **Custom Acknowledgement Actions**

Extend the system with custom actions:
```yaml
# In notification data
actions:
  - action: "ack_{{ alarm_id }}"
    title: "Acknowledge"
  - action: "snooze_{{ alarm_id }}_30"
    title: "Snooze 30m"
  - action: "escalate_{{ alarm_id }}"
    title: "Escalate Now"
```

### **Integration with External Systems**

Forward acknowledgements to external monitoring:
```yaml
# In Global Acknowledgement Handler
- service: rest_command.forward_acknowledgement
  data:
    alarm_id: "{{ alarm_id }}"
    device: "{{ device_id }}"
    timestamp: "{{ now().isoformat() }}"
```

## 📈 **Benefits**

### **Enterprise Features:**
- ✅ **Acknowledgeable Notifications** - Prevent false escalations
- ✅ **Centralized Management** - Single helper for all alarms
- ✅ **Automatic Cleanup** - No maintenance required
- ✅ **Scalable Architecture** - Handles hundreds of alarms
- ✅ **Blueprint-Based** - Consistent, reusable components

### **Operational Benefits:**
- ✅ **Reduced Alert Fatigue** - Acknowledge once, prevent escalation
- ✅ **Better Incident Response** - Clear audit trail of acknowledgements
- ✅ **Flexible Configuration** - Per-alarm acknowledgement settings
- ✅ **Zero External Dependencies** - Works entirely within Home Assistant

## 🤝 **Contributing**

This system follows Home Assistant blueprint best practices:
- Single-purpose blueprints that work together
- Comprehensive input validation
- Clear error messages and guidance
- Backward compatibility with existing automations

When contributing:
1. Test blueprints thoroughly with various configurations
2. Update documentation for any new features
3. Maintain backward compatibility
4. Follow YAML formatting standards

---

*Built with ❤️ for enterprise Home Assistant deployments*

