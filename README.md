# StreamSpace Plugins

Official plugin repository for [StreamSpace](https://github.com/JoshuaAFerguson/streamspace).

## Overview

This repository contains official and community-contributed plugins that extend StreamSpace functionality. Plugins can add new features, integrate with external services, customize workflows, and enhance the user interface.

## Repository Structure

```
streamspace-plugins/
├── official/          # Official StreamSpace plugins
│   ├── session-recorder/
│   ├── audit-logger/
│   └── slack-integration/
├── community/         # Community-contributed plugins
│   ├── github-integration/
│   └── custom-theme/
├── catalog.yaml       # Plugin discovery metadata
├── README.md          # This file
└── CONTRIBUTING.md    # Contribution guidelines
```

## Plugin Categories

### Official Plugins

Official plugins are maintained by the StreamSpace team and follow strict quality standards:

- **session-recorder**: Record and replay user sessions
- **audit-logger**: Enhanced audit logging with external storage
- **slack-integration**: Slack notifications for session events
- **metrics-exporter**: Export metrics to external monitoring systems

### Community Plugins

Community plugins are contributed by users and may have varying levels of support:

- **github-integration**: GitHub issue creation from sessions
- **custom-theme**: Example custom UI theme
- **jira-integration**: Create Jira tickets from sessions
- **ldap-sync**: Sync users from LDAP/Active Directory

## Usage

### Install from Catalog

StreamSpace can automatically sync plugins from this repository:

```yaml
# In Helm values.yaml
repositories:
  plugins:
    enabled: true
    url: https://github.com/JoshuaAFerguson/streamspace-plugins
    branch: main
    syncInterval: "1h"
```

### Install Individual Plugin

```bash
# Using StreamSpace CLI
streamspace plugin install session-recorder

# Or manually
kubectl apply -f https://raw.githubusercontent.com/JoshuaAFerguson/streamspace-plugins/main/official/session-recorder/manifest.yaml
```

### List Available Plugins

```bash
# Via API
curl https://streamspace.local/api/plugins/catalog

# Or view catalog.yaml
curl https://raw.githubusercontent.com/JoshuaAFerguson/streamspace-plugins/main/catalog.yaml
```

## Plugin Types

StreamSpace supports several plugin types:

### 1. Extension Plugins
Add new features and UI components to the platform.

**Example**: Custom dashboard widgets, new session actions

### 2. Webhook Plugins
React to system events in real-time.

**Example**: Send notifications on session creation, log user activity

**Available Events**:
- `session.created`, `session.started`, `session.stopped`, `session.deleted`
- `user.created`, `user.updated`, `user.deleted`, `user.login`
- `template.created`, `template.updated`, `template.deleted`
- `plugin.installed`, `plugin.enabled`, `plugin.disabled`, `plugin.uninstalled`
- `system.startup`, `system.shutdown`, `audit.violation`

### 3. API Integration Plugins
Connect StreamSpace to external services.

**Example**: Slack, GitHub, Jira, PagerDuty integrations

### 4. UI Theme Plugins
Customize the web interface appearance.

**Example**: Dark theme, company branding, accessibility themes

## Plugin Structure

Each plugin must follow this structure:

```
my-plugin/
├── manifest.json      # Plugin metadata (required)
├── index.js          # Entry point (required)
├── README.md         # Documentation (required)
├── config.schema.json # Configuration schema (optional)
├── package.json      # Node.js dependencies (if needed)
└── assets/           # Images, icons, etc.
    └── icon.png
```

### Minimal manifest.json

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "displayName": "My Plugin",
  "description": "Brief description",
  "type": "extension",
  "author": "Your Name",
  "license": "MIT",
  "permissions": ["read:sessions"],
  "entrypoints": {
    "main": "index.js"
  }
}
```

### Minimal index.js

```javascript
module.exports = {
  async onLoad() {
    console.log('Plugin loaded!');
  },

  async onUnload() {
    console.log('Plugin unloaded!');
  }
};
```

## Quick Start

### 1. Create a New Plugin

```bash
# Clone this repository
git clone https://github.com/JoshuaAFerguson/streamspace-plugins.git
cd streamspace-plugins

# Create plugin directory (community or official)
mkdir community/my-plugin
cd community/my-plugin

# Initialize
npm init -y

# Create manifest
cat > manifest.json <<EOF
{
  "name": "my-plugin",
  "version": "1.0.0",
  "displayName": "My Plugin",
  "description": "My custom StreamSpace plugin",
  "type": "extension",
  "author": "Your Name",
  "license": "MIT",
  "permissions": ["read:sessions"],
  "entrypoints": {
    "main": "index.js"
  }
}
EOF

# Create entry point
cat > index.js <<EOF
module.exports = {
  async onLoad() {
    console.log('My plugin loaded!');
  }
};
EOF
```

### 2. Test Your Plugin

```bash
# Package plugin
tar -czf my-plugin.tar.gz manifest.json index.js

# Upload to StreamSpace
curl -X POST https://streamspace.local/api/plugins/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@my-plugin.tar.gz"

# Enable plugin
curl -X POST https://streamspace.local/api/plugins/my-plugin/enable \
  -H "Authorization: Bearer $TOKEN"
```

### 3. Submit for Inclusion

1. Fork this repository
2. Add your plugin to `community/`
3. Update `catalog.yaml`
4. Submit a pull request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## API Reference

Plugins have access to the StreamSpace API:

```javascript
module.exports = {
  async onLoad({ api, config }) {
    // List sessions
    const sessions = await api.sessions.list();

    // Get user info
    const user = await api.users.get('username');

    // Access plugin config
    const setting = config.get('mySetting');

    // Register webhook
    api.webhooks.on('session.created', async (event) => {
      console.log('New session:', event.data.session);
    });
  }
};
```

### Available API Modules

- `api.sessions` - Session management
- `api.users` - User management
- `api.templates` - Template operations
- `api.plugins` - Plugin management
- `api.webhooks` - Event subscriptions
- `api.metrics` - Metrics collection
- `api.logs` - Logging utilities

See [API Reference](https://github.com/JoshuaAFerguson/streamspace/blob/main/docs/PLUGIN_API.md) for complete documentation.

## Permissions

Plugins must declare required permissions:

- `read:sessions` - View session information
- `write:sessions` - Create/modify sessions
- `read:users` - View user information
- `write:users` - Create/modify users
- `read:templates` - View templates
- `write:templates` - Create/modify templates
- `admin` - Administrative access
- `network` - Make external HTTP requests

## Security

All plugins are:
- ✅ Sandboxed in isolated environments
- ✅ Rate-limited to prevent abuse
- ✅ Audited for security issues
- ✅ Reviewed before inclusion in official catalog

See [Security Guidelines](CONTRIBUTING.md#security) for details.

## Examples

### Webhook Plugin - Slack Notifications

```javascript
// official/slack-integration/index.js
module.exports = {
  async onLoad({ api, config }) {
    const webhookUrl = config.get('slackWebhookUrl');

    api.webhooks.on('session.created', async (event) => {
      await fetch(webhookUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          text: `New session created: ${event.data.session.name}`
        })
      });
    });
  }
};
```

### Extension Plugin - Custom Dashboard Widget

```javascript
// community/custom-widget/index.js
module.exports = {
  async onLoad({ api, ui }) {
    ui.registerWidget({
      id: 'my-widget',
      title: 'My Custom Widget',
      component: 'MyWidget.vue',
      location: 'dashboard'
    });
  }
};
```

## Support

- **Documentation**: [Plugin Development Guide](https://github.com/JoshuaAFerguson/streamspace/blob/main/PLUGIN_DEVELOPMENT.md)
- **API Reference**: [Plugin API](https://github.com/JoshuaAFerguson/streamspace/blob/main/docs/PLUGIN_API.md)
- **Discussions**: [GitHub Discussions](https://github.com/JoshuaAFerguson/streamspace-plugins/discussions)
- **Issues**: [GitHub Issues](https://github.com/JoshuaAFerguson/streamspace-plugins/issues)

## License

MIT License - see [LICENSE](LICENSE)

## Links

- **Main Project**: [StreamSpace](https://github.com/JoshuaAFerguson/streamspace)
- **Template Repository**: [streamspace-templates](https://github.com/JoshuaAFerguson/streamspace-templates)
- **Documentation**: [StreamSpace Docs](https://github.com/JoshuaAFerguson/streamspace/tree/main/docs)
