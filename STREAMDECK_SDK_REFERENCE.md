# Stream Deck SDK Reference for AI Agents

> Comprehensive reference for developing Stream Deck plugins using the official Node.js SDK (v2.0)

## Quick Start

### Prerequisites
- Node.js 20+
- Stream Deck 6.9+
- Stream Deck device

### Create New Plugin
```bash
npm install -g @elgato/cli
streamdeck create
```

### Project Structure
```
.
├── *.sdPlugin/           # Compiled plugin directory
│   ├── bin/              # Compiled JS output
│   ├── imgs/             # Images/icons
│   ├── logs/             # Log files
│   ├── ui/               # Property inspector HTML
│   └── manifest.json     # Plugin metadata
├── src/
│   ├── actions/          # Action implementations
│   └── plugin.ts         # Entry point
├── package.json
├── rollup.config.mjs
└── tsconfig.json
```

### Development Commands
```bash
npm run watch    # Build + watch + auto-restart
npm run build    # Production build
streamdeck link  # Symlink plugin to Stream Deck
streamdeck pack  # Create .streamDeckPlugin file
```

---

## Manifest.json Reference

```json
{
    "$schema": "https://schemas.elgato.com/streamdeck/plugins/manifest.json",
    "UUID": "com.example.myplugin",
    "Name": "My Plugin",
    "Version": "1.0.0.0",
    "Author": "Your Name",
    "Description": "Plugin description",
    "Icon": "imgs/plugin-icon",
    "CategoryIcon": "imgs/category-icon",
    "Category": "My Plugin",
    "CodePath": "bin/plugin.js",
    "SDKVersion": 2,
    "Software": {
        "MinimumVersion": "6.6"
    },
    "OS": [
        { "Platform": "mac", "MinimumVersion": "10.15" },
        { "Platform": "windows", "MinimumVersion": "10" }
    ],
    "Nodejs": {
        "Version": "20",
        "Debug": "enabled"
    },
    "Actions": [
        {
            "UUID": "com.example.myplugin.myaction",
            "Name": "My Action",
            "Icon": "imgs/action-icon",
            "Tooltip": "Action description",
            "Controllers": ["Keypad"],
            "PropertyInspectorPath": "ui/my-action.html",
            "States": [
                {
                    "Image": "imgs/action-key",
                    "TitleAlignment": "middle"
                }
            ]
        }
    ]
}
```

### Key Manifest Properties

| Property | Required | Description |
|----------|----------|-------------|
| `UUID` | Yes | Reverse-DNS identifier (e.g., `com.example.plugin`) |
| `Name` | Yes | Plugin name |
| `Version` | Yes | Format: `{major}.{minor}.{patch}.{build}` |
| `Author` | Yes | Author name |
| `CodePath` | Yes | Entry point (e.g., `bin/plugin.js`) |
| `SDKVersion` | Yes | Use `2` |
| `Software.MinimumVersion` | Yes | Min Stream Deck version |
| `Actions` | Yes | Array of action definitions |
| `Category` | No | Action list group name |
| `PropertyInspectorPath` | No | Default PI HTML for all actions |

### Action Properties

| Property | Required | Description |
|----------|----------|-------------|
| `UUID` | Yes | Action identifier (prefix with plugin UUID) |
| `Name` | Yes | Display name |
| `Icon` | Yes | 20x20 / 40x40 @2x PNG/SVG (no extension) |
| `States` | Yes | Array of state definitions |
| `Controllers` | No | `["Keypad"]`, `["Encoder"]`, or both |
| `PropertyInspectorPath` | No | Action-specific PI HTML |
| `Tooltip` | No | Hover tooltip text |
| `VisibleInActionsList` | No | Set `false` to hide |
| `DisableAutomaticStates` | No | Disable auto state toggle |

### State Properties

| Property | Required | Description |
|----------|----------|-------------|
| `Image` | Yes | 72x72 / 144x144 @2x (no extension) |
| `Name` | No | State name (for multi-actions) |
| `Title` | No | Default title |
| `TitleAlignment` | No | `"top"`, `"middle"`, `"bottom"` |
| `ShowTitle` | No | Boolean |

### Encoder Properties (Stream Deck +)

```json
{
    "Controllers": ["Encoder"],
    "Encoder": {
        "layout": "$B1",
        "Icon": "imgs/encoder-icon",
        "TriggerDescription": {
            "Rotate": "Adjust volume",
            "Push": "Mute/Unmute",
            "Touch": "Toggle",
            "LongTouch": "Reset"
        }
    }
}
```

Built-in layouts: `$X1`, `$A0`, `$A1`, `$B1`, `$B2`, `$C1`

---

## Actions

### Basic Action Structure

```typescript
import streamDeck, { action, KeyDownEvent, SingletonAction } from "@elgato/streamdeck";

@action({ UUID: "com.example.myplugin.myaction" })
export class MyAction extends SingletonAction {
    override onKeyDown(ev: KeyDownEvent): void | Promise<void> {
        // Handle key press
    }
}
```

### Register Actions (plugin.ts)

```typescript
import streamDeck from "@elgato/streamdeck";
import { MyAction } from "./actions/my-action";

streamDeck.actions.registerAction(new MyAction());
streamDeck.connect();  // Always call last
```

### Key Events

| Event | Description |
|-------|-------------|
| `onKeyDown(ev: KeyDownEvent)` | Key pressed |
| `onKeyUp(ev: KeyUpEvent)` | Key released |
| `onWillAppear(ev: WillAppearEvent)` | Action appears on canvas |
| `onWillDisappear(ev: WillDisappearEvent)` | Action removed from canvas |
| `onDidReceiveSettings(ev: DidReceiveSettingsEvent)` | Settings changed |
| `onTitleParametersDidChange(ev: TitleParametersDidChangeEvent)` | User changed title |
| `onPropertyInspectorDidAppear(ev)` | PI opened |
| `onPropertyInspectorDidDisappear(ev)` | PI closed |
| `onSendToPlugin(ev: SendToPluginEvent)` | Message from PI |

### Dial Events (Stream Deck +)

| Event | Description |
|-------|-------------|
| `onDialDown(ev: DialDownEvent)` | Dial pressed |
| `onDialUp(ev: DialUpEvent)` | Dial released |
| `onDialRotate(ev: DialRotateEvent)` | Dial rotated |
| `onTouchTap(ev: TouchTapEvent)` | Touch strip tapped |

### Action Commands

```typescript
// Title and Image
ev.action.setTitle("New Title");
ev.action.setTitle("Title", { state: 0, target: Target.Hardware });
ev.action.setImage("imgs/new-icon.png");
ev.action.setImage(`data:image/svg+xml,${encodeURIComponent(svg)}`);

// States
ev.action.setState(0);  // or 1 for second state

// Feedback
ev.action.showOk();     // Green checkmark (keys only, not dials)
ev.action.showAlert();  // Yellow warning (keys and dials)

// Settings
const settings = await ev.action.getSettings<MySettings>();
await ev.action.setSettings({ count: 5 });

// Send to PI (via streamDeck.ui, NOT on action instances)
streamDeck.ui.sendToPropertyInspector({ type: "update", data: {} });
```

### Dial Commands

```typescript
// Set layout
ev.action.setFeedbackLayout("$B1");
ev.action.setFeedbackLayout("layouts/custom.json");

// Update layout items
ev.action.setFeedback({
    title: "Volume",
    value: "75%",
    indicator: { value: 75 }
});

// Trigger descriptions
ev.action.setTriggerDescription({
    rotate: "Adjust",
    push: "Mute",
    touch: "Toggle"
});
```

### Iterate Visible Actions

```typescript
// All plugin actions
streamDeck.actions.forEach((action) => {
    action.setTitle("Updated");
});

// Specific action type (inside SingletonAction)
this.actions.forEach((action) => {
    action.setTitle("Updated");
});
```

### Property Inspector Communication (`streamDeck.ui`)

**Important:** `sendToPropertyInspector` lives on `streamDeck.ui`, NOT on action instances. It sends to the currently visible Property Inspector.

```typescript
import streamDeck from "@elgato/streamdeck";

// Send data to the currently visible PI
await streamDeck.ui.sendToPropertyInspector({ type: "update", data: {} });

// Get the action whose PI is currently visible (if any)
const currentAction = streamDeck.ui.action; // DialAction | KeyAction | undefined

// Listen for PI lifecycle events
streamDeck.ui.onDidAppear((ev) => { /* PI opened */ });
streamDeck.ui.onDidDisappear((ev) => { /* PI closed */ });
```

**Note:** `streamDeck.ui.sendToPropertyInspector()` only sends if a PI is currently visible. It automatically routes to the correct action's PI based on internal context.

---

## Settings

### Action Settings

```typescript
type Settings = {
    count: number;
    name: string;
};

@action({ UUID: "com.example.counter" })
class Counter extends SingletonAction<Settings> {
    override async onKeyDown(ev: KeyDownEvent<Settings>): Promise<void> {
        // Read from event payload
        let { count = 0 } = ev.payload.settings;
        count++;

        // Save settings
        await ev.action.setSettings({ count });
    }

    override onDidReceiveSettings(ev: DidReceiveSettingsEvent<Settings>): void {
        // React to PI changes
        streamDeck.logger.info(JSON.stringify(ev.payload.settings));
    }
}
```

### Global Settings

```typescript
// Write
await streamDeck.settings.setGlobalSettings({ apiKey: "xxx" });

// Read
const settings = await streamDeck.settings.getGlobalSettings<GlobalSettings>();

// Listen for changes
streamDeck.settings.onDidReceiveGlobalSettings((ev) => {
    streamDeck.logger.info(JSON.stringify(ev.settings));
});
```

### Security Notes
- **Action settings**: Plain text, included in profile exports. Never store secrets.
- **Global settings**: Stored securely. Use for OAuth tokens and user-provided API keys.

---

## Property Inspector (UI)

### Basic HTML Structure

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="utf-8" />
    <script src="sdpi-components.js"></script>
</head>
<body>
    <sdpi-item label="Name">
        <sdpi-textfield setting="name"></sdpi-textfield>
    </sdpi-item>

    <sdpi-item label="Enabled">
        <sdpi-checkbox setting="enabled"></sdpi-checkbox>
    </sdpi-item>

    <sdpi-item label="Mode">
        <sdpi-select setting="mode">
            <option value="a">Option A</option>
            <option value="b">Option B</option>
        </sdpi-select>
    </sdpi-item>
</body>
</html>
```

### The `setting` Attribute

All persistent components use the `setting` attribute to bind to a property in the action's settings. Nested paths are supported:

```html
<!-- Saves to settings.network.host -->
<sdpi-textfield setting="network.host"></sdpi-textfield>
```

Resulting settings:
```json
{
    "network": {
        "host": "192.168.1.1"
    }
}
```

Add `global` attribute to persist to global settings instead of action settings:
```html
<sdpi-textfield setting="apiKey" global></sdpi-textfield>
```

### SDPI Components Quick Reference

| Component | Element | Value Type | Description |
|-----------|---------|------------|-------------|
| Item | `<sdpi-item label="...">` | N/A | Layout wrapper with label |
| Text field | `<sdpi-textfield>` | `string` | Single-line text input |
| Textarea | `<sdpi-textarea>` | `string` | Multi-line text input |
| Password | `<sdpi-password>` | `string` | Masked text input |
| Checkbox | `<sdpi-checkbox>` | `boolean` | Boolean toggle |
| Checkbox list | `<sdpi-checkbox-list>` | `string[]` | Multi-select checkboxes |
| Radio | `<sdpi-radio>` | `string` | Single-select radio buttons |
| Select | `<sdpi-select>` | `string` | Dropdown select |
| Range | `<sdpi-range>` | `number` | Slider |
| Color | `<sdpi-color>` | `string` (hex) | Color picker |
| File | `<sdpi-file>` | `string` (path) | File picker |
| Button | `<sdpi-button>` | N/A | Clickable button (no persistence) |
| Delegate | `<sdpi-delegate>` | `unknown` | Plugin-delegated value |
| Calendar | `<sdpi-calendar type="...">` | `string` | date/time/week/month/datetime-local |
| i18n | `<sdpi-i18n key="...">` | N/A | Localized text display |

### Component Details

#### `<sdpi-item>` - Layout Wrapper

```html
<sdpi-item label="Field Label">
    <sdpi-textfield setting="name"></sdpi-textfield>
</sdpi-item>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Label text; clicking focuses the first input |

#### `<sdpi-textfield>` - Text Input

```html
<sdpi-textfield setting="first_name" placeholder="Enter name" pattern="/^[a-z ,.'-]+$/i" maxlength="100" required></sdpi-textfield>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `global` | `boolean` | Persist to global settings |
| `disabled` | `boolean` | Disable input |
| `maxlength` | `number` | Max character length |
| `pattern` | `string` | Regex validation |
| `placeholder` | `string` | Placeholder text |
| `required` | `boolean` | Show icon if empty |
| `value` | `string` | Current value |

#### `<sdpi-textarea>` - Multi-line Text

```html
<sdpi-textarea setting="description" maxlength="250" rows="3" showlength></sdpi-textarea>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `global` | `boolean` | Persist to global settings |
| `disabled` | `boolean` | Disable input |
| `maxlength` | `number` | Max character length |
| `rows` | `number` | Visible rows |
| `showlength` | `boolean` | Show character count |

#### `<sdpi-password>` - Password Input

```html
<sdpi-password setting="api_key" placeholder="Enter API key" required></sdpi-password>
```

Same attributes as `<sdpi-textfield>`.

#### `<sdpi-checkbox>` - Boolean Toggle

```html
<sdpi-checkbox setting="is_enabled" label="Enable feature" default></sdpi-checkbox>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `global` | `boolean` | Persist to global settings |
| `default` | `boolean` | Default value |
| `disabled` | `boolean` | Disable input |
| `label` | `string` | Label text right of checkbox |

#### `<sdpi-checkbox-list>` - Multi-select

```html
<sdpi-checkbox-list setting="selected_items" columns="2">
    <option value="1">One</option>
    <option value="2">Two</option>
    <option value="3">Three</option>
</sdpi-checkbox-list>
```

Value is an **array** of selected option values (not booleans).

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `global` | `boolean` | Persist to global settings |
| `columns` | `number` | Column count (1-6) |
| `disabled` | `boolean` | Disable input |
| `value-type` | `string` | `"boolean"`, `"number"`, or `"string"` (default) |
| `datasource` | `string` | Remote data source event name |
| `hot-reload` | `boolean` | Monitor sendToPropertyInspector for updates |
| `loading` | `string` | Loading text |

#### `<sdpi-radio>` - Single-select Radio

```html
<sdpi-radio setting="size" columns="3" default="medium">
    <option value="small">Small</option>
    <option value="medium">Medium</option>
    <option value="large">Large</option>
</sdpi-radio>
```

Same attributes as `<sdpi-checkbox-list>` plus `default`. Supports `datasource`.

#### `<sdpi-select>` - Dropdown

```html
<sdpi-select setting="color" placeholder="Choose a color">
    <optgroup label="Primary">
        <option value="#ff0000">Red</option>
        <option value="#00ff00">Green</option>
        <option value="#0000ff">Blue</option>
    </optgroup>
    <option value="#000000">Black</option>
</sdpi-select>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `global` | `boolean` | Persist to global settings |
| `default` | `string` | Default value |
| `disabled` | `boolean` | Disable input |
| `placeholder` | `string` | Placeholder text |
| `value-type` | `string` | `"boolean"`, `"number"`, or `"string"` (default) |
| `label-setting` | `string` | Also persist the selected label |
| `datasource` | `string` | Remote data source event name |
| `hot-reload` | `boolean` | Monitor sendToPropertyInspector for updates |
| `loading` | `string` | Loading text |
| `show-refresh` | `boolean` | Show refresh button |

#### `<sdpi-range>` - Slider

```html
<sdpi-range setting="brightness" min="0" max="100" step="5" default="50" showlabels>
    <span slot="min">0%</span>
    <span slot="max">100%</span>
</sdpi-range>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `global` | `boolean` | Persist to global settings |
| `min` | `number` | Minimum value |
| `max` | `number` | Maximum value |
| `step` | `number` | Step granularity |
| `default` | `string` | Default value |
| `showlabels` | `boolean` | Show min/max labels |

#### `<sdpi-color>` - Color Picker

```html
<sdpi-color setting="bg_color" default="#ffffff"></sdpi-color>
```

Value is a hex string (e.g., `"#00aaff"`).

#### `<sdpi-file>` - File Picker

```html
<sdpi-file setting="avatar" accept="image/png, image/jpeg" label="Browse..."></sdpi-file>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `accept` | `string` | File type filter |
| `label` | `string` | Button label (default: `...`) |

#### `<sdpi-button>` - Button (No Persistence)

```html
<sdpi-button onclick="alert('clicked')">Click Me</sdpi-button>
```

#### `<sdpi-delegate>` - Plugin-delegated Value

Sends `sendToPlugin` when invoked; the plugin sets the value via settings.

```html
<sdpi-delegate setting="folder_path" invoke="browseFolder" label="..." format-type="path"></sdpi-delegate>
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `setting` | `string` | Settings property path |
| `invoke` | `string` | Event name sent to plugin |
| `label` | `string` | Button text |
| `format-type` | `string` | `"path"` to show filename only |

When invoked, sends to plugin:
```json
{ "payload": { "event": "browseFolder" } }
```

#### `<sdpi-calendar>` - Date/Time Inputs

```html
<sdpi-calendar type="date" setting="start_date" min="2024-01-01"></sdpi-calendar>
<sdpi-calendar type="time" setting="alarm_time"></sdpi-calendar>
<sdpi-calendar type="datetime-local" setting="event_time"></sdpi-calendar>
<sdpi-calendar type="month" setting="birth_month"></sdpi-calendar>
<sdpi-calendar type="week" setting="target_week"></sdpi-calendar>
```

Types: `date`, `time`, `datetime-local`, `month`, `week`

#### `<sdpi-i18n>` - Localized Text (No Persistence)

```html
<sdpi-button><sdpi-i18n key="click_prompt"></sdpi-i18n></sdpi-button>
```

### Data Sources

Dynamically populate `<sdpi-select>`, `<sdpi-radio>`, or `<sdpi-checkbox-list>` from the plugin.

**In PI HTML:**
```html
<sdpi-select setting="device" datasource="getDevices" loading="Loading devices..." hot-reload></sdpi-select>
```

When the component initializes, it sends `sendToPlugin`:
```json
{
    "payload": {
        "event": "getDevices",
        "isRefresh": false
    }
}
```

**In Plugin - respond with `streamDeck.ui.sendToPropertyInspector`:**
```typescript
override onSendToPlugin(ev: SendToPluginEvent): void {
    if (ev.payload.event === "getDevices") {
        streamDeck.ui.sendToPropertyInspector({
            event: "getDevices",
            items: [
                {
                    label: "Audio Devices",
                    children: [
                        { label: "Speakers", value: "speakers" },
                        { label: "Headphones", value: "headphones" }
                    ]
                },
                { label: "Default", value: "default" }
            ]
        });
    }
}
```

**Data source item types:**
```typescript
type Item = { value: string; label?: string; disabled?: boolean };
type ItemGroup = { label?: string; children: Item[] };
type DataSourceResult = (Item | ItemGroup)[];
```

Call `.refresh()` on the element to re-request data. During refresh, `isRefresh: true` is sent.

### Stream Deck Client (PI JavaScript API)

Access via `SDPIComponents.streamDeckClient`:

```javascript
const client = SDPIComponents.streamDeckClient;
```

| Method | Returns | Description |
|--------|---------|-------------|
| `getSettings()` | `Promise<ActionSettingsPayload>` | Get action settings |
| `setSettings(value)` | `void` | Set action settings |
| `getGlobalSettings()` | `Promise<Record<string, unknown>>` | Get global settings |
| `setGlobalSettings(value)` | `void` | Set global settings |
| `getConnectionInfo()` | `Promise<ConnectionInfo>` | Registration info (devices, OS, plugin info) |
| `send(event, payload?)` | `Promise<void>` | Send event to Stream Deck |

Valid `send()` events: `getSettings`, `setSettings`, `getGlobalSettings`, `setGlobalSettings`, `logMessage`, `openUrl`, `sendToPlugin`

**Event subscriptions:**
```javascript
client.didReceiveSettings.subscribe((ev) => { /* settings changed */ });
client.didReceiveGlobalSettings.subscribe((ev) => { /* global settings changed */ });
client.sendToPropertyInspector.subscribe((ev) => { /* message from plugin */ });
```

### PI Localization

```javascript
SDPIComponents.i18n.locales = {
    en: { name: "Name", greeting: "Hello" },
    de: { name: "Name", greeting: "Hallo" },
    fr: { name: "Nom", greeting: "Bonjour" }
};

// Use in HTML with __MSG_{key}__ format
// Programmatic access:
const msg = SDPIComponents.i18n.getMessage("greeting");
```

```html
<sdpi-item label="__MSG_name__">
    <sdpi-textfield setting="name"></sdpi-textfield>
</sdpi-item>
<sdpi-i18n key="greeting"></sdpi-i18n>
```

Fallback: user locale -> English (`en`) -> raw key.

### PI to Plugin Communication

**From PI (JavaScript):**
```javascript
const { streamDeckClient } = SDPIComponents;

// Send message to plugin
streamDeckClient.send("sendToPlugin", { type: "refresh" });

// Get/set settings
streamDeckClient.setSettings({ key: "value" });
const settings = await streamDeckClient.getSettings();
```

**In Plugin:**
```typescript
@action({ UUID: "com.example.myaction" })
class MyAction extends SingletonAction {
    override onSendToPlugin(ev: SendToPluginEvent): void {
        if (ev.payload.type === "refresh") {
            // Handle message — use streamDeck.ui, NOT ev.action
            streamDeck.ui.sendToPropertyInspector({ type: "data", items: [] });
        }
    }
}
```

---

## System Events & Utilities

### Device Events

```typescript
streamDeck.devices.onDeviceDidConnect((ev) => {
    const { id, isConnected, name, size, type } = ev.device;
    streamDeck.logger.info(`Connected: ${name} (${id}), type: ${type}`);
});

streamDeck.devices.onDeviceDidDisconnect((ev) => {
    streamDeck.logger.info(`Disconnected: ${ev.device.id}`);
});

// Available from Stream Deck 7.0
streamDeck.devices.onDeviceDidChange((ev) => {
    const { id, isConnected, name, size, type } = ev.device;
    streamDeck.logger.info(`Device changed: ${name}`);
});
```

### App Monitoring

In manifest.json:
```json
{
    "ApplicationsToMonitor": {
        "mac": ["com.apple.Safari"],
        "windows": ["notepad.exe"]
    }
}
```

In plugin:
```typescript
streamDeck.system.onApplicationDidLaunch((ev) => {
    streamDeck.logger.info(`App launched: ${ev.application}`);
});

streamDeck.system.onApplicationDidTerminate((ev) => {
    streamDeck.logger.info(`App terminated: ${ev.application}`);
});
```

### Deep Linking

URL format: `streamdeck://plugins/message/<PLUGIN_UUID>/<path>?<query>`

```typescript
streamDeck.system.onDidReceiveDeepLink((ev) => {
    const { path, fragment } = ev.url;
    streamDeck.logger.info(`Path = ${path}`);
    streamDeck.logger.info(`Fragment = ${fragment}`);
});
```

OAuth redirect proxy: `https://oauth2-redirect.elgato.com/streamdeck/plugins/message/<PLUGIN_UUID>`

### System Wake

```typescript
streamDeck.system.onSystemDidWakeUp((ev) => {
    // Restore WebSocket connections, re-sync state
});
```

### Open URL

```typescript
streamDeck.system.openUrl("https://example.com");
```

---

## Profiles

### Bundle Profile in Manifest

```json
{
    "Profiles": [
        {
            "Name": "profiles/my-profile",
            "DeviceType": 0,
            "AutoInstall": true,
            "Readonly": false
        }
    ]
}
```

Device types: `0`=SD, `1`=Mini, `2`=XL, `3`=Mobile, `4`=Corsair GKeys, `5`=Pedal, `6`=Corsair Voyager, `7`=SD+, `8`=SCUF Controller, `9`=Neo, `10`=Studio, `11`=Virtual

### Switch to Profile

```typescript
streamDeck.profiles.switchToProfile(ev.action.device.id, "My Profile");
```

---

## Logging

```typescript
import streamDeck from "@elgato/streamdeck";

// Log levels (most to least severe): error, warn, info, debug, trace
streamDeck.logger.error("Error occurred");
streamDeck.logger.warn("Warning");
streamDeck.logger.info("Info message");
streamDeck.logger.debug("Debug info");
streamDeck.logger.trace("Detailed execution flow");

// Set log level
streamDeck.logger.setLevel("warn");

// Scoped logger
const log = streamDeck.logger.createScope("MyAction");
log.info("Scoped message");  // [MyAction] Scoped message
```

Logs saved to: `*.sdPlugin/logs/<plugin-uuid>.0.log` (rotation index 0-9)

---

## Localization (i18n)

### File Structure
```
*.sdPlugin/
    en.json      # Default (English)
    de.json      # German
    fr.json      # French
    manifest.json
```

### Localization File

```json
{
    "Name": "Mein Plugin",
    "Description": "Plugin-Beschreibung",
    "com.example.myaction": {
        "Name": "Meine Aktion",
        "Tooltip": "Aktionsbeschreibung"
    },
    "Localization": {
        "custom_key": "Benutzerdefinierter Text"
    }
}
```

### Use Custom Strings

```typescript
const text = streamDeck.i18n.translate("custom_key");
```

Supported: `en`, `de`, `es`, `fr`, `ja`, `ko`, `zh_CN`, `zh_TW`

---

## CLI Commands

```bash
streamdeck create              # Create new plugin
streamdeck link                # Symlink to plugins folder
streamdeck restart <uuid>      # Restart plugin
streamdeck dev                 # Enable debug mode
streamdeck pack                # Create .streamDeckPlugin
streamdeck validate            # Validate manifest/layouts
```

---

## Common Patterns

### Counter Action

```typescript
type Settings = { count: number };

@action({ UUID: "com.example.counter" })
export class Counter extends SingletonAction<Settings> {
    override async onWillAppear(ev: WillAppearEvent<Settings>): Promise<void> {
        const { count = 0 } = ev.payload.settings;
        await ev.action.setTitle(`${count}`);
    }

    override async onKeyDown(ev: KeyDownEvent<Settings>): Promise<void> {
        let { count = 0 } = ev.payload.settings;
        count++;
        await ev.action.setSettings({ count });
        await ev.action.setTitle(`${count}`);
    }
}
```

### Toggle Action (Two States)

```json
{
    "States": [
        { "Image": "imgs/off", "Name": "Off" },
        { "Image": "imgs/on", "Name": "On" }
    ]
}
```

```typescript
@action({ UUID: "com.example.toggle" })
export class Toggle extends SingletonAction {
    override async onKeyDown(ev: KeyDownEvent): Promise<void> {
        const newState = ev.payload.state === 0 ? 1 : 0;
        await ev.action.setState(newState);
        // Perform toggle action...
    }
}
```

### Dynamic Image from SVG

```typescript
override async onKeyDown(ev: KeyDownEvent<Settings>): Promise<void> {
    const { count } = ev.payload.settings;
    const color = count % 2 === 0 ? "red" : "blue";

    const svg = `<svg width="72" height="72" xmlns="http://www.w3.org/2000/svg">
        <rect fill="${color}" width="72" height="72"/>
        <text x="36" y="40" text-anchor="middle" fill="white">${count}</text>
    </svg>`;

    await ev.action.setImage(`data:image/svg+xml,${encodeURIComponent(svg)}`);
}
```

### Volume Dial (Stream Deck +)

```typescript
@action({ UUID: "com.example.volume" })
export class Volume extends SingletonAction<{ volume: number }> {
    override async onWillAppear(ev: WillAppearEvent): Promise<void> {
        await ev.action.setFeedbackLayout("$B1");
        await this.updateFeedback(ev.action, ev.payload.settings.volume ?? 50);
    }

    override async onDialRotate(ev: DialRotateEvent): Promise<void> {
        let { volume = 50 } = ev.payload.settings;
        volume = Math.max(0, Math.min(100, volume + ev.payload.ticks * 2));
        await ev.action.setSettings({ volume });
        await this.updateFeedback(ev.action, volume);
    }

    private async updateFeedback(action: Action, volume: number): Promise<void> {
        await action.setFeedback({
            title: "Volume",
            value: `${volume}%`,
            indicator: { value: volume }
        });
    }
}
```

---

## Image Specifications

| Use Case | Size | @2x Size | Format | Notes |
|----------|------|----------|--------|-------|
| Plugin icon | 256x256 | 512x512 | PNG | Marketplace icon |
| Category icon | 28x28 | 56x56 | PNG/SVG | Monochrome white |
| Action icon | 20x20 | 40x40 | PNG/SVG | Monochrome white |
| Key image | 72x72 | 144x144 | PNG/SVG/GIF | Action state image |
| Encoder icon | 72x72 | 144x144 | PNG/SVG | Dial canvas |
| Touch background | 200x100 | 400x200 | PNG/SVG | SD+ touch strip |

**Note:** Omit file extensions in manifest paths.

---

## Debugging

1. Enable debug mode: `streamdeck dev`
2. In manifest: `"Nodejs": { "Debug": "enabled" }`
3. Open Chrome: `chrome://inspect`
4. Property Inspector: `http://localhost:23654/`

---

## Distribution

1. Validate: `streamdeck validate`
2. Pack: `streamdeck pack` (creates `.streamDeckPlugin`)
3. Submit to Marketplace: https://marketplace.elgato.com
