# StreamSpace Microsoft Teams Integration Plugin

Send real-time notifications about StreamSpace events to your Microsoft Teams channels.

## Features

- Session event notifications (created, hibernated)
- User event notifications (created)
- Configurable notification preferences
- Rate limiting to prevent spam
- Rich Teams adaptive cards with formatting
- Detailed or summary notifications
- Microsoft Teams webhook integration

## Installation

### Via StreamSpace UI

1. Navigate to **Admin** → **Plugins**
2. Search for "Microsoft Teams Integration"
3. Click **Install**
4. Configure your Teams webhook URL
5. Enable the plugin

### Via kubectl

```bash
kubectl apply -f - <<EOF
apiVersion: stream.space/v1alpha1
kind: InstalledPlugin
metadata:
  name: streamspace-teams
  namespace: streamspace
spec:
  catalogPluginName: streamspace-teams
  enabled: true
  config:
    webhookUrl: "https://outlook.office.com/webhook/YOUR_WEBHOOK_URL"
    notifyOnSessionCreated: true
    notifyOnSessionHibernated: false
    notifyOnUserCreated: true
    includeDetails: true
    rateLimit: 20
EOF
```

## Configuration

### Microsoft Teams Webhook Setup

1. Open Microsoft Teams and navigate to your target channel
2. Click the three dots (**...**) next to the channel name
3. Select **Connectors** or **Workflows**
4. Search for **Incoming Webhook**
5. Click **Add** or **Configure**
6. Provide a name (e.g., "StreamSpace Notifications")
7. Optionally upload an image for the webhook
8. Click **Create**
9. Copy the webhook URL
10. Click **Done**

### Plugin Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `webhookUrl` | string | *required* | Your Microsoft Teams webhook URL |
| `notifyOnSessionCreated` | boolean | `true` | Send notification when session created |
| `notifyOnSessionHibernated` | boolean | `false` | Send notification when session hibernated |
| `notifyOnUserCreated` | boolean | `true` | Send notification when user created |
| `includeDetails` | boolean | `true` | Include detailed information in notifications |
| `rateLimit` | number | `20` | Maximum messages per hour (1-100) |

### Configuration Example

```json
{
  "webhookUrl": "https://outlook.office.com/webhook/12345678-1234-1234-1234-123456789012@abcdefgh-abcd-abcd-abcd-abcdefghijkl/IncomingWebhook/abcdefghijklmnop/12345678-1234-1234-1234-123456789012",
  "notifyOnSessionCreated": true,
  "notifyOnSessionHibernated": false,
  "notifyOnUserCreated": true,
  "includeDetails": true,
  "rateLimit": 20
}
```

## Example Notifications

### Session Created
```
🚀 New Session Created

User: john@example.com
Template: firefox-browser
Session ID: john-firefox-abc123
```

With `includeDetails: true`:
```
🚀 New Session Created

User: john@example.com
Template: firefox-browser
Session ID: john-firefox-abc123
Memory: 2Gi
CPU: 1000m
Timestamp: 2025-11-18 14:30:00
```

### Session Hibernated
```
💤 Session Hibernated

User: john@example.com
Session ID: john-firefox-abc123

Session hibernated due to inactivity.
```

### User Created
```
👤 New User Created

Username: jane
Email: jane@example.com
Full Name: Jane Smith
```

## Events

This plugin listens to the following StreamSpace events:

- `session.created` - New session created
- `session.hibernated` - Session hibernated
- `user.created` - New user created

## Rate Limiting

To prevent Microsoft Teams from being overwhelmed, the plugin includes configurable rate limiting:

- Default: 20 messages per hour
- Range: 1-100 messages per hour
- Configurable via `rateLimit` setting
- Excess notifications are logged but not sent

## Permissions

This plugin requires:

- `network` - To send HTTP requests to Microsoft Teams

## Troubleshooting

### Webhook Not Working

1. Verify webhook URL is correct and follows the pattern: `https://*.webhook.office.com/*`
2. Check that webhook hasn't been removed or disabled in Teams
3. Ensure channel still exists and webhook is active
4. Check StreamSpace logs for error messages
5. Verify you have permissions to the Teams channel

### No Notifications

1. Verify plugin is enabled
2. Check notification settings (e.g., `notifyOnSessionCreated`)
3. Check rate limit hasn't been exceeded
4. Verify events are being triggered in StreamSpace
5. Ensure webhook URL is valid and active
6. Check Teams channel notifications aren't muted

### Too Many Notifications

1. Reduce `rateLimit` setting
2. Disable specific event notifications
3. Set `includeDetails` to `false` for shorter messages
4. Consider using filters or notification rules in Teams

### Invalid Webhook URL Error

The webhook URL must match the pattern: `^https://.*\.webhook\.office\.com/.*$`

Ensure your URL:
- Starts with `https://`
- Contains `.webhook.office.com`
- Is a valid Incoming Webhook URL from Teams

### Webhook Deleted or Expired

1. Webhooks can be removed if the connector is deleted
2. Create a new webhook following the setup instructions
3. Update the plugin configuration with the new URL
4. Webhooks may be disabled if the channel is archived

## Microsoft Teams Connector Limits

Be aware of Microsoft Teams connector limits:

- Maximum message rate: Approximately 4 messages per second
- Message size: Up to 28 KB per message
- The plugin's rate limiting helps stay within these bounds

## Adaptive Cards

This plugin sends notifications as Microsoft Teams Adaptive Cards, which provide:

- Rich formatting and visual hierarchy
- Color-coded messages based on event type
- Structured layout for better readability
- Consistent appearance across Teams clients

## Development

### Building from Source

```bash
cd official/streamspace-teams
go build -o teams-plugin.so -buildmode=plugin teams_plugin.go
```

### Testing

```bash
go test ./...
```

### Testing Webhook

```bash
# Test webhook with curl
curl -H "Content-Type: application/json" -d '{
  "text": "Test notification from StreamSpace"
}' "YOUR_WEBHOOK_URL"
```

## License

MIT License - see LICENSE file

## Support

- [Documentation](https://docs.streamspace.io/plugins/teams)
- [Microsoft Teams Webhooks Docs](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook)
- [Issues](https://github.com/JoshuaAFerguson/streamspace-plugins/issues)
- [Discord](https://discord.gg/streamspace)

## Version History

### 1.0.0 (2025-11-16)
- Initial release
- Session event notifications
- User event notifications
- Rate limiting
- Microsoft Teams Adaptive Cards
- Configurable notification preferences
