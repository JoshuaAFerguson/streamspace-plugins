# Contributing to StreamSpace Plugins

Thank you for your interest in contributing plugins to StreamSpace! This guide will help you create, test, and submit plugins.

## Table of Contents

- [Plugin Types](#plugin-types)
- [Getting Started](#getting-started)
- [Plugin Structure](#plugin-structure)
- [Manifest Requirements](#manifest-requirements)
- [Testing Your Plugin](#testing-your-plugin)
- [Submitting Your Plugin](#submitting-your-plugin)
- [Security Guidelines](#security-guidelines)
- [Code Review Process](#code-review-process)

## Plugin Types

StreamSpace supports four types of plugins:

### 1. Extension Plugins
Add new features and UI components.

**Use Cases**: Custom widgets, new session actions, enhanced functionality

### 2. Webhook Plugins
React to system events in real-time.

**Use Cases**: Notifications, logging, external integrations, automation

### 3. Integration Plugins
Connect to external services.

**Use Cases**: Slack, GitHub, Jira, PagerDuty, custom APIs

### 4. Theme Plugins
Customize UI appearance.

**Use Cases**: Dark mode, company branding, accessibility

## Getting Started

### 1. Fork and Clone

```bash
git fork https://github.com/JoshuaAFerguson/streamspace-plugins
git clone https://github.com/YOUR_USERNAME/streamspace-plugins
cd streamspace-plugins
```

### 2. Create Plugin Directory

Choose between `official/` (for core team) or `community/` (for contributors):

```bash
mkdir community/my-plugin
cd community/my-plugin
npm init -y
```

### 3. Create Required Files

Every plugin must have:
- `manifest.json` - Plugin metadata
- `index.js` - Entry point
- `README.md` - Documentation

## Plugin Structure

### Directory Layout

```
my-plugin/
├── manifest.json       # Required: Plugin metadata
├── index.js           # Required: Entry point
├── README.md          # Required: Documentation
├── config.schema.json # Optional: Configuration schema
├── package.json       # Optional: Node.js dependencies
├── test/             # Optional: Tests
│   └── index.test.js
└── assets/           # Optional: Images, icons
    └── icon.png
```

### Example Files

**manifest.json**:
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "displayName": "My Plugin",
  "description": "Brief description of what the plugin does",
  "type": "extension",
  "author": "Your Name <your.email@example.com>",
  "homepage": "https://github.com/yourusername/my-plugin",
  "license": "MIT",
  "permissions": [
    "read:sessions"
  ],
  "entrypoints": {
    "main": "index.js"
  },
  "compatibility": {
    "streamspace": ">=1.0.0"
  },
  "tags": ["utility", "automation"]
}
```

**index.js**:
```javascript
module.exports = {
  /**
   * Called when plugin is loaded
   */
  async onLoad({ api, config, logger }) {
    logger.info('My plugin loaded!');

    // Example: Listen to session events
    api.webhooks.on('session.created', async (event) => {
      logger.info('New session created:', event.data.session.name);
    });
  },

  /**
   * Called when plugin is unloaded
   */
  async onUnload({ logger }) {
    logger.info('My plugin unloaded!');
  }
};
```

**README.md**:
```markdown
# My Plugin

Brief description of the plugin.

## Features

- Feature 1
- Feature 2

## Configuration

```yaml
myPlugin:
  enabled: true
  setting1: value1
```

## Usage

Instructions for using the plugin.

## License

MIT
```

## Manifest Requirements

### Required Fields

- ✅ `name` - Lowercase, hyphenated (e.g., `my-plugin`)
- ✅ `version` - Semantic version (e.g., `1.0.0`)
- ✅ `displayName` - Human-readable name
- ✅ `description` - Clear, concise description (max 200 chars)
- ✅ `type` - One of: `extension`, `webhook`, `integration`, `theme`
- ✅ `author` - Name and email
- ✅ `license` - Open source license (MIT, Apache-2.0, GPL-3.0)
- ✅ `permissions` - Array of required permissions
- ✅ `entrypoints.main` - Path to entry file

### Optional Fields

- `homepage` - Project homepage URL
- `repository` - Git repository URL
- `bugs` - Issue tracker URL
- `keywords` - Search keywords
- `compatibility.streamspace` - Version requirement
- `config.schema` - Path to config schema file
- `icon` - Path to icon file

### Permissions

Declare all required permissions:

**Read Permissions**:
- `read:sessions` - View sessions
- `read:users` - View users
- `read:templates` - View templates
- `read:plugins` - View plugins

**Write Permissions**:
- `write:sessions` - Create/modify sessions
- `write:users` - Create/modify users
- `write:templates` - Create/modify templates

**Dangerous Permissions** (require approval):
- `admin` - Administrative access
- `network` - External HTTP requests
- `filesystem` - File system access

## Testing Your Plugin

### Local Testing

```bash
# Package plugin
cd my-plugin
tar -czf ../my-plugin.tar.gz .

# Upload to local StreamSpace instance
curl -X POST http://localhost:8000/api/plugins/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@../my-plugin.tar.gz"

# Enable plugin
curl -X POST http://localhost:8000/api/plugins/my-plugin/enable \
  -H "Authorization: Bearer $TOKEN"

# Check logs
curl http://localhost:8000/api/plugins/my-plugin/logs \
  -H "Authorization: Bearer $TOKEN"
```

### Automated Testing

Create tests in `test/index.test.js`:

```javascript
const plugin = require('../index');

describe('My Plugin', () => {
  test('loads successfully', async () => {
    const mockApi = {
      webhooks: { on: jest.fn() }
    };
    const mockConfig = { get: jest.fn() };
    const mockLogger = { info: jest.fn() };

    await plugin.onLoad({
      api: mockApi,
      config: mockConfig,
      logger: mockLogger
    });

    expect(mockLogger.info).toHaveBeenCalledWith('My plugin loaded!');
  });
});
```

Run tests:
```bash
npm test
```

## Submitting Your Plugin

### 1. Update Catalog

Add your plugin to `catalog.yaml`:

```yaml
plugins:
  - name: my-plugin
    category: community
    displayName: My Plugin
    description: Brief description
    type: extension
    author: Your Name
    version: "1.0.0"
    path: community/my-plugin
    permissions:
      - read:sessions
    tags: [utility, automation]
    verified: false
```

### 2. Create Pull Request

```bash
git checkout -b add-my-plugin
git add community/my-plugin
git add catalog.yaml
git commit -m "feat(plugins): add my-plugin"
git push origin add-my-plugin
```

Create PR with this template:

```markdown
## Plugin Submission

**Plugin Name**: My Plugin
**Type**: Extension
**Category**: Community

## Description

Brief description of what the plugin does and why it's useful.

## Features

- Feature 1
- Feature 2

## Permissions Required

- `read:sessions` - Required to view session information

## Testing

- [x] Tested locally
- [x] All tests passing
- [x] No security issues
- [x] Documentation complete

## Checklist

- [x] manifest.json is valid
- [x] README.md included
- [x] No credentials or secrets in code
- [x] Follows code style guidelines
- [x] Added to catalog.yaml

## Screenshots

(Optional) Screenshots of the plugin in action
```

## Security Guidelines

### ⚠️ Critical Rules

1. **Never include credentials** - Use config for API keys/tokens
2. **Validate all input** - Prevent injection attacks
3. **Use HTTPS** - For all external requests
4. **Rate limit** - Prevent abuse
5. **Handle errors** - Don't expose sensitive info

### Code Security

**❌ Bad**:
```javascript
// DON'T hardcode credentials
const apiKey = 'sk-1234567890';

// DON'T use eval
eval(userInput);

// DON'T expose errors
throw new Error(secret);
```

**✅ Good**:
```javascript
// DO use config
const apiKey = config.get('apiKey');

// DO validate input
if (!isValid(userInput)) {
  throw new Error('Invalid input');
}

// DO sanitize errors
logger.error('Operation failed', { code: 'ERR_001' });
```

### Network Security

- ✅ Validate URLs before making requests
- ✅ Use allowlists for external domains
- ✅ Implement timeouts for HTTP requests
- ✅ Check response size limits

### Data Security

- ✅ Never log sensitive data (passwords, tokens)
- ✅ Encrypt sensitive config values
- ✅ Use secure random for tokens
- ✅ Follow GDPR/privacy regulations

## Code Review Process

All plugins undergo review:

1. **Automated Checks** (GitHub Actions)
   - YAML/JSON validation
   - Security scanning
   - Dependency audit

2. **Manual Review** (Maintainer)
   - Code quality
   - Security review
   - Documentation check

3. **Testing** (Maintainer)
   - Functional testing
   - Performance testing
   - Integration testing

### Review Timeline

- Initial response: **Within 48 hours**
- Full review: **Within 1 week**
- Revisions: **Within 48 hours of update**

### Approval Criteria

- ✅ No security vulnerabilities
- ✅ Complete documentation
- ✅ Follows coding standards
- ✅ Tests passing
- ✅ No breaking changes

## Code Style

### JavaScript

```javascript
// Use async/await
async function doSomething() {
  const result = await api.sessions.list();
  return result;
}

// Handle errors
try {
  await doSomething();
} catch (error) {
  logger.error('Failed:', error.message);
}

// Use const/let, not var
const API_URL = 'https://api.example.com';
let counter = 0;
```

### Naming Conventions

- **Files**: `kebab-case.js`
- **Functions**: `camelCase()`
- **Constants**: `UPPER_SNAKE_CASE`
- **Classes**: `PascalCase`

## Getting Help

- **Questions**: [GitHub Discussions](https://github.com/JoshuaAFerguson/streamspace-plugins/discussions)
- **Bugs**: [GitHub Issues](https://github.com/JoshuaAFerguson/streamspace-plugins/issues)
- **Documentation**: [Plugin Development Guide](https://github.com/JoshuaAFerguson/streamspace/blob/main/PLUGIN_DEVELOPMENT.md)
- **API Reference**: [Plugin API](https://github.com/JoshuaAFerguson/streamspace/blob/main/docs/PLUGIN_API.md)

## License

All contributed plugins must use an OSI-approved open source license (MIT, Apache-2.0, GPL-3.0, etc.).

## Thank You!

Thank you for contributing to StreamSpace! 🚀
