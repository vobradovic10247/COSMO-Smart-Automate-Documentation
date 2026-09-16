# Import / Export

The Import / Export page moves Parent Rules, Work Item Calculation Rules, and Sibling Rules between projects using a JSON rule bundle.

## Open Import / Export

1. Open **Project Settings** -> **Extensions** -> **COSMO Smart Automate**.
2. Select **Import / Export Rules**.

The page has separate actions for exporting selected rule sets and importing a bundle from a JSON file.

## Export Rule Sets

1. Select the rule sets to include in the export:
   - **Parent Rules**
   - **Work Item Calculation Rules**
   - **Sibling Rules**
2. Select **Export selected rule sets**.
3. Save the downloaded JSON file for the target project or as a backup.

The export includes only the selected rule engines. The default selection includes all three supported engines. An export cannot be created when the selected engines contain no rules.

Export files use a name similar to:

`cosmo-smart-automate-rules-all-2026-09-16.json`

The bundle contains the format identifier, bundle version, export timestamp, and one document collection for each rule engine:

- `parentRules`
- `calculationRules`
- `siblingRules`
- `childCalculationRules` (reserved for a planned engine and currently empty)

## Import Rule Sets

1. Select **Import rules** on the Import / Export page.
2. Choose a JSON rule bundle.
3. Review the rule counts shown for each engine.
4. Select **Import**.

The import panel previews the number of Parent, Calculation, and Sibling Rules in the selected file before it changes the current project.

Imports are applied to the project where the extension is currently open. Work item types, states, fields, and process configuration are project-specific, so verify that the target project supports the imported rule settings.

## Validation and Limits

Before importing, the extension checks that the file:

- Is not empty and contains valid JSON.
- Uses the `cosmo-smart-automate-rules` format.
- Uses a supported integer bundle version. The current version is `1`.
- Is no larger than 2 MB.
- Contains valid Parent, Calculation, and Sibling Rule documents.

Bundles containing non-empty `childCalculationRules` are rejected because Child Calculation Rules are not supported yet.

## Duplicates and Updates

Equivalent existing rules are skipped and counted in the import result. New configurations are imported. A rule with the same ID but changed configuration updates the existing rule where supported by the rule engine.

After a successful import, the extension reports the total number of imported and skipped rules, for example:

`Imported 4 rules. Skipped 2 duplicates.`

## Failed Imports

Before writing imported rules, the extension snapshots the existing documents for each engine included in the bundle. If a persistence operation fails, it attempts to restore those snapshots so a failed import does not leave a partially imported configuration.

If the restore also fails, the error identifies that the previous rules could not be fully restored. Keep the original export file and review the target project's storage and permissions before trying again.

## Recommended Workflow

- Export the current rule sets before making a large change.
- Import into a test project first when moving rules between different processes.
- Use the **Parent Rule Tester** to verify imported Parent Rule behavior before enabling rules for production use.
- Keep the exported JSON file with the project documentation so the configuration can be reviewed or restored later.