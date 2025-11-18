# StreamSpace Plugins Repository

## Project Overview

This is the official plugin repository for [StreamSpace](https://github.com/JoshuaAFerguson/streamspace), a Kubernetes-based remote development environment platform. This repository serves as a centralized catalog for both official and community-contributed plugins that extend StreamSpace's functionality.

## Purpose

StreamSpace Plugins provides:

1. **Official Plugins**: Maintained by the StreamSpace team with strict quality standards
2. **Community Plugins**: User-contributed extensions with varying support levels
3. **Plugin Catalog**: Centralized discovery and installation system
4. **Plugin Framework**: Standards and tools for plugin development

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
├── README.md          # User documentation
├── CONTRIBUTING.md    # Contribution guidelines
└── claude.md          # This file - AI context
```

## Plugin Types

StreamSpace supports four main plugin types:

### 1. Extension Plugins
Add new features and UI components to the platform.
- Custom dashboard widgets
- New session actions
- Enhanced functionality

### 2. Webhook Plugins
React to system events in real-time.
- Session lifecycle events (created, started, stopped, deleted)
- User events (created, updated, deleted, login)
- Template events
- System events

### 3. Integration Plugins
Connect StreamSpace to external services.
- Slack notifications
- GitHub/Jira integrations
- PagerDuty alerts
- Custom third-party services

### 4. Theme Plugins
Customize the web interface appearance.
- Dark/light themes
- Company branding
- Accessibility themes

## Plugin Structure

Each plugin must follow this structure:

```
my-plugin/
├── manifest.json      # Plugin metadata (REQUIRED)
├── index.js          # Entry point (REQUIRED)
├── README.md         # Documentation (REQUIRED)
├── config.schema.json # Configuration schema (optional)
├── package.json      # Node.js dependencies (if needed)
└── assets/           # Images, icons, etc.
    └── icon.png
```

### Required Files

#### manifest.json
Defines plugin metadata, permissions, and entry points.

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

#### index.js
Plugin entry point with lifecycle hooks.

```javascript
module.exports = {
  async onLoad({ api, config }) {
    // Plugin initialization
  },

  async onUnload() {
    // Cleanup
  }
};
```

## Plugin API

Plugins have access to the StreamSpace API:

### Available Modules
- `api.sessions` - Session management
- `api.users` - User management
- `api.templates` - Template operations
- `api.plugins` - Plugin management
- `api.webhooks` - Event subscriptions
- `api.metrics` - Metrics collection
- `api.logs` - Logging utilities

### Example Usage

```javascript
module.exports = {
  async onLoad({ api, config }) {
    // List sessions
    const sessions = await api.sessions.list();

    // Register webhook
    api.webhooks.on('session.created', async (event) => {
      console.log('New session:', event.data.session);
    });

    // Access config
    const setting = config.get('mySetting');
  }
};
```

## Permissions System

Plugins must declare required permissions:

- `read:sessions` - View session information
- `write:sessions` - Create/modify sessions
- `read:users` - View user information
- `write:users` - Create/modify users
- `read:templates` - View templates
- `write:templates` - Create/modify templates
- `admin` - Administrative access (dangerous)
- `network` - Make external HTTP requests (dangerous)

## Development Workflow

### Creating a New Plugin

1. Fork this repository
2. Create plugin directory in `community/` or `official/`
3. Implement required files (manifest.json, index.js, README.md)
4. Test locally with StreamSpace
5. Update catalog.yaml
6. Submit pull request

### Testing

```bash
# Package plugin
tar -czf my-plugin.tar.gz manifest.json index.js README.md

# Upload to StreamSpace
curl -X POST https://streamspace.local/api/plugins/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@my-plugin.tar.gz"

# Enable plugin
curl -X POST https://streamspace.local/api/plugins/my-plugin/enable \
  -H "Authorization: Bearer $TOKEN"
```

### Submission Guidelines

See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Code quality standards
- Testing requirements
- Security guidelines
- Review process
- Documentation requirements

## Security

All plugins are:
- Sandboxed in isolated environments
- Rate-limited to prevent abuse
- Audited for security issues
- Reviewed before inclusion in official catalog

### Security Best Practices

1. Minimize permission requests
2. Validate all inputs
3. Sanitize external data
4. Use secure dependencies
5. Follow principle of least privilege

## Integration with StreamSpace

### Installation Methods

#### 1. Catalog Sync (Recommended)

```yaml
# In Helm values.yaml
repositories:
  plugins:
    enabled: true
    url: https://github.com/JoshuaAFerguson/streamspace-plugins
    branch: main
    syncInterval: "1h"
```

#### 2. CLI Installation

```bash
streamspace plugin install session-recorder
```

#### 3. Manual Installation

```bash
kubectl apply -f https://raw.githubusercontent.com/JoshuaAFerguson/streamspace-plugins/main/official/session-recorder/manifest.yaml
```

## catalog.yaml

The catalog.yaml file maintains:
- Plugin metadata and discovery information
- Category definitions (official/community)
- Plugin type definitions
- Permission schemas
- Event definitions
- Statistics and compatibility info

This file is the central registry for all plugins and is consumed by StreamSpace for plugin discovery and installation.

## Common Tasks

### Adding a New Official Plugin

1. Create directory: `official/plugin-name/`
2. Add required files (manifest.json, index.js, README.md)
3. Update catalog.yaml with plugin entry
4. Update README.md plugin list
5. Commit and push changes

### Reviewing Community Contributions

1. Check plugin structure and required files
2. Review code for security issues
3. Test functionality locally
4. Verify documentation quality
5. Approve or request changes

### Updating Existing Plugin

1. Navigate to plugin directory
2. Update version in manifest.json
3. Make code/documentation changes
4. Update catalog.yaml if metadata changed
5. Commit with clear changelog

## Related Repositories

- **Main Project**: [streamspace](https://github.com/JoshuaAFerguson/streamspace)
- **Templates**: [streamspace-templates](https://github.com/JoshuaAFerguson/streamspace-templates)
- **Documentation**: [streamspace/docs](https://github.com/JoshuaAFerguson/streamspace/tree/main/docs)

## Key Concepts

### Plugin Lifecycle

1. **Discovery**: Listed in catalog.yaml
2. **Installation**: Downloaded and validated
3. **Configuration**: User provides settings
4. **Activation**: onLoad() called
5. **Runtime**: Hooks and API access
6. **Deactivation**: onUnload() called
7. **Uninstallation**: Cleanup

### Event System

Plugins can subscribe to system events:
- Session lifecycle events
- User management events
- Template operations
- Plugin state changes
- System events (startup/shutdown)
- Audit violations

### Configuration

Plugins can define configuration schemas:
```json
{
  "configSchema": {
    "type": "object",
    "properties": {
      "webhookUrl": {
        "type": "string",
        "description": "Webhook URL for notifications"
      }
    },
    "required": ["webhookUrl"]
  }
}
```

## Working with Claude Code

When modifying this repository:

1. **Adding plugins**: Create complete directory structure with all required files
2. **Updating catalog**: Keep catalog.yaml in sync with plugin additions/removals
3. **Documentation**: Update README.md when adding new plugins or categories
4. **Testing**: Ensure manifest.json is valid JSON and follows schema
5. **Security**: Review permissions and external dependencies

## Questions to Consider

When working on this repository:

- Is the plugin structure complete?
- Are all required files present?
- Is manifest.json valid and complete?
- Are permissions minimized?
- Is documentation clear and comprehensive?
- Is catalog.yaml updated?
- Does it follow security best practices?
- Is the version number incremented appropriately?

## Next Steps

After initial setup, common tasks include:

1. Migrating existing plugins from main StreamSpace repo
2. Creating official plugin implementations
3. Setting up CI/CD for plugin validation
4. Documenting plugin development workflow
5. Building plugin testing infrastructure
