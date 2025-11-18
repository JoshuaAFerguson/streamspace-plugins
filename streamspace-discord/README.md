# StreamSpace Discord Integration Plugin

Send real-time notifications about StreamSpace events to your Discord channels.

## Features

- Session event notifications (created, hibernated)
- User event notifications (created)
- Configurable notification preferences
- Rate limiting to prevent spam
- Customizable bot username and avatar
- Rich Discord embeds with colors and formatting
- Detailed or summary notifications

## Installation

### Via StreamSpace UI

1. Navigate to **Admin** → **Plugins**
2. Search for "Discord Integration"
3. Click **Install**
4. Configure your Discord webhook URL
5. Enable the plugin

### Via kubectl

```bash
kubectl apply -f - <<EOF
apiVersion: stream.space/v1alpha1
kind: InstalledPlugin
metadata:
  name: streamspace-discord
  namespace: streamspace
spec:
  catalogPluginName: streamspace-discord
  enabled: true
  config:
    webhookUrl: "https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/YOUR_WEBHOOK_TOKEN"
    username: "StreamSpace"
    avatarUrl: ""
    notifyOnSessionCreated: true
    notifyOnSessionHibernated: true
    notifyOnUserCreated: true
    includeDetails: false
    rateLimit: 20
EOF
```

## Configuration

### Discord Webhook Setup

1. Go to your Discord server
2. Navigate to **Server Settings** → **Integrations** → **Webhooks**
3. Click **New Webhook** or **Create Webhook**
4. Select the channel for notifications
5. Customize the webhook name and avatar (optional)
6. Copy the webhook URL
7. Click **Save**

### Plugin Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `webhookUrl` | string | *required* | Your Discord webhook URL (must match pattern) |
| `username` | string | `StreamSpace` | Override the default webhook username |
| `avatarUrl` | string | - | Override the default webhook avatar image URL |
| `notifyOnSessionCreated` | boolean | `true` | Send notification when session created |
| `notifyOnSessionHibernated` | boolean | `true` | Send notification when session hibernated |
| `notifyOnUserCreated` | boolean | `true` | Send notification when user created |
| `includeDetails` | boolean | `false` | Include CPU and memory information |
| `rateLimit` | number | `20` | Maximum messages per hour (1-100) |

### Configuration Example

```json
{
  "webhookUrl": "https://discord.com/api/webhooks/123456789/abcdefghijklmnop",
  "username": "StreamSpace Bot",
  "avatarUrl": "https://example.com/avatar.png",
  "notifyOnSessionCreated": true,
  "notifyOnSessionHibernated": true,
  "notifyOnUserCreated": true,
  "includeDetails": false,
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
```

## Events

This plugin listens to the following StreamSpace events:

- `session.created` - New session created
- `session.hibernated` - Session hibernated
- `user.created` - New user created

## Rate Limiting

To prevent Discord from being overwhelmed, the plugin includes configurable rate limiting:

- Default: 20 messages per hour
- Range: 1-100 messages per hour
- Configurable via `rateLimit` setting
- Excess notifications are logged but not sent

## Permissions

This plugin requires:

- `network` - To send HTTP requests to Discord

## Troubleshooting

### Webhook Not Working

1. Verify webhook URL is correct and follows the pattern: `https://discord.com/api/webhooks/{id}/{token}`
2. Check that webhook hasn't been deleted in Discord
3. Ensure Discord server allows incoming webhooks
4. Check StreamSpace logs for error messages

### No Notifications

1. Verify plugin is enabled
2. Check notification settings (e.g., `notifyOnSessionCreated`)
3. Check rate limit hasn't been exceeded
4. Verify events are being triggered in StreamSpace
5. Ensure webhook URL is valid and active

### Too Many Notifications

1. Reduce `rateLimit` setting
2. Disable specific event notifications
3. Set `includeDetails` to `false` for shorter messages

### Invalid Webhook URL Error

The webhook URL must match the pattern: `^https://discord\.com/api/webhooks/[0-9]+/[a-zA-Z0-9_-]+$`

Ensure your URL:
- Starts with `https://discord.com/api/webhooks/`
- Contains a numeric webhook ID
- Contains an alphanumeric token

## Development

### Building from Source

```bash
cd official/streamspace-discord
go build -o discord-plugin.so -buildmode=plugin discord_plugin.go
```

### Testing

```bash
go test ./...
```

## License

MIT License - see LICENSE file

## Support

- [Documentation](https://docs.streamspace.io/plugins/discord)
- [Issues](https://github.com/JoshuaAFerguson/streamspace-plugins/issues)
- [Discord](https://discord.gg/streamspace)

## Version History

### 1.0.0 (2025-11-16)
- Initial release
- Session event notifications
- User event notifications
- Rate limiting
- Rich Discord embeds
- Customizable bot appearance
