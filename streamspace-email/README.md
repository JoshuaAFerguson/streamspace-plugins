# StreamSpace Email SMTP Integration Plugin

Send email notifications via SMTP for StreamSpace session and user events with rich HTML formatting.

## Features

- SMTP email notifications for session and user events
- Support for multiple SMTP providers (Gmail, Office365, custom)
- HTML and plain text email formats
- Multiple recipients (To, CC)
- TLS/STARTTLS encryption support
- Configurable notification preferences
- Rate limiting to prevent email spam
- Detailed or summary notifications
- Customizable sender name and address

## Installation

### Via StreamSpace UI

1. Navigate to **Admin** → **Plugins**
2. Search for "Email SMTP Integration"
3. Click **Install**
4. Configure your SMTP settings
5. Enable the plugin

### Via kubectl

```bash
kubectl apply -f - <<EOF
apiVersion: stream.space/v1alpha1
kind: InstalledPlugin
metadata:
  name: streamspace-email
  namespace: streamspace
spec:
  catalogPluginName: streamspace-email
  enabled: true
  config:
    smtpHost: "smtp.gmail.com"
    smtpPort: 587
    username: "notifications@example.com"
    password: "your-smtp-password"
    fromAddress: "notifications@example.com"
    fromName: "StreamSpace Notifications"
    toAddresses:
      - "admin@example.com"
      - "team@example.com"
    ccAddresses: []
    useTLS: true
    notifyOnSessionCreated: true
    notifyOnSessionHibernated: true
    notifyOnUserCreated: true
    includeDetails: true
    htmlFormat: true
    rateLimit: 30
EOF
```

## Configuration

### SMTP Provider Setup

#### Gmail

1. Enable 2-factor authentication on your Google account
2. Generate an App Password:
   - Go to Google Account → Security → 2-Step Verification → App passwords
   - Select "Mail" and your device
   - Copy the generated 16-character password
3. Use these settings:
   - Host: `smtp.gmail.com`
   - Port: `587`
   - Username: Your Gmail address
   - Password: App password (not your regular password)
   - TLS: `true`

#### Office 365 / Outlook

1. Use your Office 365 account credentials
2. Settings:
   - Host: `smtp.office365.com`
   - Port: `587`
   - Username: Your Office 365 email
   - Password: Your Office 365 password
   - TLS: `true`

#### Custom SMTP Server

Configure with your own SMTP server details:
- Port `587` for STARTTLS (recommended)
- Port `465` for TLS/SSL
- Port `25` for plain SMTP (not recommended)

### Plugin Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `smtpHost` | string | *required* | SMTP server hostname |
| `smtpPort` | integer | `587` | SMTP port (25, 465, 587, 2525) |
| `username` | string | *required* | SMTP authentication username |
| `password` | string | *required* | SMTP authentication password |
| `fromAddress` | string | *required* | Sender email address |
| `fromName` | string | `StreamSpace Notifications` | Sender display name |
| `toAddresses` | array | *required* | List of recipient email addresses |
| `ccAddresses` | array | `[]` | List of CC recipient addresses |
| `useTLS` | boolean | `true` | Enable TLS encryption |
| `notifyOnSessionCreated` | boolean | `true` | Send email when session created |
| `notifyOnSessionHibernated` | boolean | `true` | Send email when session hibernated |
| `notifyOnUserCreated` | boolean | `true` | Send email when user created |
| `includeDetails` | boolean | `true` | Include CPU and memory information |
| `htmlFormat` | boolean | `true` | Send HTML formatted emails |
| `rateLimit` | number | `30` | Maximum emails per hour (1-100) |

### Configuration Example

```json
{
  "smtpHost": "smtp.gmail.com",
  "smtpPort": 587,
  "username": "notifications@example.com",
  "password": "app-specific-password",
  "fromAddress": "notifications@example.com",
  "fromName": "StreamSpace Notifications",
  "toAddresses": [
    "admin@example.com",
    "devops@example.com"
  ],
  "ccAddresses": [
    "manager@example.com"
  ],
  "useTLS": true,
  "notifyOnSessionCreated": true,
  "notifyOnSessionHibernated": true,
  "notifyOnUserCreated": true,
  "includeDetails": true,
  "htmlFormat": true,
  "rateLimit": 30
}
```

## Example Notifications

### Session Created (HTML Format)

**Subject:** StreamSpace: New Session Created

**Body:**
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

**Subject:** StreamSpace: Session Hibernated

**Body:**
```
💤 Session Hibernated

User: john@example.com
Session ID: john-firefox-abc123

Session hibernated due to inactivity.
Timestamp: 2025-11-18 16:45:00
```

### User Created

**Subject:** StreamSpace: New User Created

**Body:**
```
👤 New User Created

Username: jane
Email: jane@example.com
Full Name: Jane Smith
Timestamp: 2025-11-18 09:15:00
```

## Events

This plugin listens to the following StreamSpace events:

- `session.created` - New session created
- `session.hibernated` - Session hibernated
- `user.created` - New user created

## Rate Limiting

To prevent email spam and respect SMTP provider limits:

- Default: 30 emails per hour
- Range: 1-100 emails per hour
- Configurable via `rateLimit` setting
- Excess notifications are logged but not sent

### SMTP Provider Limits

Be aware of your provider's sending limits:
- **Gmail:** 500 emails/day for free accounts, 2000/day for Google Workspace
- **Office 365:** 10,000 emails/day
- **Custom servers:** Check with your administrator

## Permissions

This plugin requires:

- `network` - To send SMTP requests

## Troubleshooting

### Authentication Failed

1. Verify username and password are correct
2. For Gmail, ensure you're using an App Password, not your regular password
3. Check that 2FA is enabled (required for App Passwords)
4. Verify SMTP host and port are correct for your provider
5. Check StreamSpace logs for detailed error messages

### Connection Timeout

1. Verify SMTP host is reachable from StreamSpace
2. Check firewall rules allow outbound connections to SMTP port
3. Ensure correct port (587 for STARTTLS, 465 for TLS)
4. Try toggling `useTLS` setting
5. Test connection from StreamSpace pod: `telnet smtp.example.com 587`

### No Emails Received

1. Verify plugin is enabled
2. Check notification settings for each event type
3. Verify rate limit hasn't been exceeded
4. Check spam/junk folders
5. Ensure recipient addresses are correct
6. Check SMTP provider sending limits
7. Review StreamSpace logs for errors

### TLS/SSL Errors

1. For port 587, use `useTLS: true` (STARTTLS)
2. For port 465, use `useTLS: true` (TLS/SSL)
3. Verify your SMTP server supports TLS
4. Check certificate validity
5. Try different port/TLS combinations

### Emails in Spam Folder

1. Configure SPF records for your domain
2. Set up DKIM signing
3. Configure DMARC policy
4. Use a reputable SMTP provider
5. Avoid spam trigger words in notifications

## Security Best Practices

1. **Never commit SMTP credentials to version control**
2. Use Kubernetes secrets for sensitive data
3. Enable TLS whenever possible
4. Use strong, unique passwords
5. For Gmail, use App Passwords instead of account password
6. Regularly rotate SMTP credentials
7. Monitor for unauthorized access

## Development

### Building from Source

```bash
cd official/streamspace-email
go build -o email-plugin.so -buildmode=plugin email_plugin.go
```

### Testing

```bash
go test ./...
```

### Testing SMTP Connection

```bash
# Test SMTP connection
openssl s_client -starttls smtp -connect smtp.gmail.com:587

# Verify credentials (base64 encode)
echo -n 'username@example.com' | base64
echo -n 'password' | base64
```

## License

MIT License - see LICENSE file

## Support

- [Documentation](https://docs.streamspace.io/plugins/email)
- [Issues](https://github.com/JoshuaAFerguson/streamspace-plugins/issues)
- [Discord](https://discord.gg/streamspace)

## Version History

### 1.0.0 (2025-11-16)
- Initial release
- SMTP email notifications
- HTML and plain text formats
- Multiple recipients support
- TLS/STARTTLS encryption
- Rate limiting
- Support for major SMTP providers
