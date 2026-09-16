# COSMO Smart Automate

COSMO Smart Automate is an Azure DevOps extension that automates work item state and field updates with configurable rules.

It supports:

- Parent Rules that update parent work items when child work items change.
- Sibling Rules that coordinate work items sharing the same parent.
- Work Item Calculation Rules that calculate field values from fields and constants.
- Rule Tester previews before changes are applied.
- Import and Export for moving rule sets between projects.
- Presets for common Azure DevOps processes.

## Documentation

Start with the [documentation index](docs/index.md), or go directly to a guide:

- [Parent Rules](docs/PARENT_RULES.md)
- [Sibling Rules](docs/SIBLING_RULES.md)
- [Work Item Calculation Rules](docs/CALCULATION_RULES.md)
- [Settings](docs/SETTINGS.md)
- [Preset Rules](docs/PRESETS.md)

## Supported Views

Rules execute when supported work item changes are saved from the work item form or backlog. Boards and query result views do not currently trigger rules.

The extension does not process mass updates.

## Support

For product or usage questions, contact [COSMO CONSULT](https://www.cosmoconsult.com/contact/).

## License

COSMO Smart Automate is distributed under the [MIT License](LICENSE). The project includes original work from [Auto State by Joachim Dalen](https://github.com/joachimdalen/azdevops-auto-state), also licensed under MIT.
