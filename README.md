# ramsoi-plugins

Personal Claude Code plugin marketplace.

## Installation

First, register this as a marketplace:
```
/plugin add-marketplace ramsoi-plugins soi53/ramsoi-plugins
```

Then install any plugin:
```
/plugin install vanilla-slide-presentation@ramsoi-plugins
```

## Plugins

| Plugin | Description |
|--------|-------------|
| [vanilla-slide-presentation](./plugins/vanilla-slide-presentation) | Single-file HTML/CSS/JS slide presentation — no external libraries |

## Structure

```
ramsoi-plugins/
└── plugins/
    └── {plugin-name}/
        ├── .claude-plugin/
        │   └── plugin.json
        ├── skills/
        │   └── {skill-name}/
        │       └── SKILL.md
        └── README.md
```
