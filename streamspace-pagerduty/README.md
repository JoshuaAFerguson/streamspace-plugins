# StreamSpace PagerDuty Integration Plugin

Send incident alerts to PagerDuty for critical StreamSpace events with configurable severity levels.

## Features

- Incident alerting for session and user events
- Configurable severity levels (info, warning, error, critical)
- PagerDuty Events API v2 integration
- Auto-resolve capability for informational alerts
- Rate limiting to prevent alert fatigue
- Detailed custom fields with resource information
- Event deduplication support

## Installation

### Via StreamSpace UI

1. Navigate to **Admin** → **Plugins**
2. Search for "PagerDuty Integration"
3. Click **Install**
4. Configure your PagerDuty integration key
5. Enable the plugin

### Via kubectl

```bash
kubectl apply -f - <<EOF
apiVersion: stream.space/v1alpha1
kind: InstalledPlugin
metadata:
  name: streamspace-pagerduty
  namespace: streamspace
spec:
  catalogPluginName: streamspace-pagerduty
  enabled: true
  config:
    routingKey: "your-32-character-integration-key"
    notifyOnSessionCreated: false
    notifyOnSessionHibernated: true
    notifyOnUserCreated: false
    sessionCreatedSeverity: "info"
    sessionHibernatedSeverity: "warning"
    userCreatedSeverity: "info"
    includeDetails: true
    autoResolve: false
    rateLimit: 50
EOF
```

## Configuration

### PagerDuty Integration Setup

1. Log in to your PagerDuty account
2. Navigate to **Services** → **Service Directory**
3. Select an existing service or create a new one
4. Go to the **Integrations** tab
5. Click **Add Integration**
6. Select **Events API v2** as the integration type
7. Name your integration (e.g., "StreamSpace Events")
8. Click **Add Integration**
9. Copy the **Integration Key** (32-character routing key)

### Plugin Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `routingKey` | string | *required* | PagerDuty Events API v2 integration key (32 chars) |
| `notifyOnSessionCreated` | boolean | `false` | Send alert when session created |
| `notifyOnSessionHibernated` | boolean | `true` | Send alert when session hibernated |
| `notifyOnUserCreated` | boolean | `false` | Send alert when user created |
| `sessionCreatedSeverity` | string | `info` | Severity: info, warning, error, critical |
| `sessionHibernatedSeverity` | string | `warning` | Severity: info, warning, error, critical |
| `userCreatedSeverity` | string | `info` | Severity: info, warning, error, critical |
| `includeDetails` | boolean | `true` | Include CPU and memory in custom details |
| `autoResolve` | boolean | `false` | Auto-resolve events after sending |
| `rateLimit` | number | `50` | Maximum events per hour (1-200) |

### Configuration Example

```json
{
  "routingKey": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "notifyOnSessionCreated": false,
  "notifyOnSessionHibernated": true,
  "notifyOnUserCreated": false,
  "sessionCreatedSeverity": "info",
  "sessionHibernatedSeverity": "warning",
  "userCreatedSeverity": "info",
  "includeDetails": true,
  "autoResolve": false,
  "rateLimit": 50
}
```

## Example Alerts

### Session Hibernated (Warning Severity)
```
Summary: StreamSpace Session Hibernated
Severity: warning
Source: StreamSpace

Custom Details:
- User: john@example.com
- Session ID: john-firefox-abc123
- Memory: 2Gi
- CPU: 1000m
- Event: session.hibernated
```

### Session Created (Info Severity)
```
Summary: StreamSpace Session Created
Severity: info
Source: StreamSpace

Custom Details:
- User: jane@example.com
- Template: vscode-dev
- Session ID: jane-vscode-xyz789
- Memory: 4Gi
- CPU: 2000m
- Event: session.created
```

## Events

This plugin listens to the following StreamSpace events:

- `session.created` - New session created
- `session.hibernated` - Session hibernated
- `user.created` - New user created

## Severity Levels

Configure different severity levels for each event type:

- **info** - Informational events, lowest priority
- **warning** - Warning level, moderate priority
- **error** - Error level, high priority
- **critical** - Critical level, highest priority, immediate attention

### Best Practices

- Use **info** for routine events like session/user creation
- Use **warning** for events that may require attention (hibernation)
- Use **error** or **critical** for failures and critical issues
- Enable `autoResolve: true` for informational alerts to prevent incident buildup

## Rate Limiting

To prevent alert fatigue and PagerDuty API limits:

- Default: 50 events per hour
- Range: 1-200 events per hour
- Configurable via `rateLimit` setting
- Excess events are logged but not sent
- Consider higher limits for production environments

## Permissions

This plugin requires:

- `network` - To send HTTP requests to PagerDuty Events API

## Troubleshooting

### Integration Key Not Working

1. Verify integration key is exactly 32 characters
2. Ensure you're using Events API v2 integration (not v1)
3. Check that the PagerDuty service is active
4. Verify integration hasn't been deleted in PagerDuty
5. Check StreamSpace logs for error messages

### No Alerts Appearing

1. Verify plugin is enabled
2. Check notification settings for each event type
3. Verify rate limit hasn't been exceeded
4. Ensure events are being triggered in StreamSpace
5. Check PagerDuty service integration status

### Too Many Incidents

1. Reduce `rateLimit` setting
2. Disable specific event notifications (e.g., `notifyOnSessionCreated`)
3. Enable `autoResolve: true` for informational events
4. Increase severity thresholds appropriately
5. Use PagerDuty's event rules to suppress or aggregate events

### Invalid Routing Key Error

The routing key must be exactly 32 alphanumeric characters. Verify:
- Key is from Events API v2 integration
- No extra spaces or characters
- Key hasn't been regenerated in PagerDuty

## Auto-Resolve Feature

When `autoResolve: true`:
- Events are sent as "trigger" and immediately "resolved"
- Useful for informational alerts that don't require action
- Prevents incident buildup in PagerDuty
- Maintains audit trail without creating actionable incidents

## Development

### Building from Source

```bash
cd official/streamspace-pagerduty
go build -o pagerduty-plugin.so -buildmode=plugin pagerduty_plugin.go
```

### Testing

```bash
go test ./...
```

## License

MIT License - see LICENSE file

## Support

- [Documentation](https://docs.streamspace.io/plugins/pagerduty)
- [PagerDuty Events API v2 Docs](https://developer.pagerduty.com/docs/ZG9jOjExMDI5NTgw-events-api-v2-overview)
- [Issues](https://github.com/JoshuaAFerguson/streamspace-plugins/issues)
- [Discord](https://discord.gg/streamspace)

## Version History

### 1.0.0 (2025-11-16)
- Initial release
- Events API v2 support
- Configurable severity levels
- Auto-resolve capability
- Rate limiting
- Custom details with resource information
