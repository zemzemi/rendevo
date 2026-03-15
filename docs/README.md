# Documentation

## Structure

```
docs/
├── features/          # Feature specifications per bounded context
│   ├── _template.md   # Template to copy for new features
│   └── {context}/
│       └── {feature}.md
└── adr/               # Architecture Decision Records
    ├── _template.md   # Template to copy for new ADRs
    ├── 001-ddd-architecture.md
    └── 002-shadcn-vue-ui.md
```

## Features

Feature docs describe what is built — endpoints, entities, business rules, and known limitations. One file per feature, organized by bounded context. Written when a feature is completed.

## ADRs

Architecture Decision Records capture *why* a significant technical decision was made. Written at decision time, never deleted (mark as deprecated or superseded instead).
