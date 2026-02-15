# streamdeck-ai-docs

Stream Deck SDK reference documentation optimized for AI agent development.

## What is this?

A single, comprehensive markdown file ([STREAMDECK_SDK_REFERENCE.md](STREAMDECK_SDK_REFERENCE.md)) that consolidates the [Elgato Stream Deck SDK v2.0](https://docs.elgato.com/streamdeck/sdk/introduction/getting-started) documentation into a format optimized for use by AI coding agents.

Instead of navigating dozens of HTML pages, an AI agent can ingest one file and have everything it needs to build Stream Deck plugins.

## What's covered

- **Quick Start** - Project scaffolding and file structure
- **Manifest.json** - Complete schema reference with all properties
- **Actions** - SingletonAction class, key events, dial events, commands
- **Settings** - Action settings, global settings, typed settings
- **Property Inspector (UI)** - Full SDPI components reference with every component, attribute, and example
- **Data Sources** - Dynamic population of select/radio/checkbox-list from the plugin
- **Stream Deck Client** - PI-side JavaScript API
- **System Events** - Device events, app monitoring, deep linking, system wake
- **Profiles** - Bundling and switching
- **Logging & Localization** - Logger API, i18n setup
- **CLI Commands** - All `streamdeck` CLI commands
- **Common Patterns** - Counter, toggle, SVG rendering, volume dial examples
- **Image Specifications** - Size requirements for all asset types
- **Debugging & Distribution** - Debug setup and marketplace submission

## Usage

Point your AI agent at `STREAMDECK_SDK_REFERENCE.md` as context when working on Stream Deck plugin projects. For example, add it to your `CLAUDE.md`:

```
Read STREAMDECK_SDK_REFERENCE.md for Stream Deck SDK documentation.
```

## Sources

Compiled from the official Elgato documentation:

- [Stream Deck SDK docs](https://docs.elgato.com/streamdeck/sdk/introduction/getting-started)
- [SDPI Components docs](https://sdpi-components.dev)

## License

The content is derived from Elgato's public documentation. This repository is an unofficial community resource.
