# Calculation Rules

Work Item Calculation Rules calculate a value for a field on the work item that triggered the rule. A calculation can use supported work item fields, constants, arithmetic operations, and optional filters.

## Create a Calculation Rule

1. Open **Project Settings** -> **Extensions** -> **COSMO Smart Automate**.
2. Open **Work Item Calculation Rules** and select **Add Rule**.
3. Enter a description and select the **Result field**.
4. Open the **Calculation** tab and add calculation lines in the order they should be evaluated.
5. Optionally add conditions in the **Filters** tab.
6. Save the rule and ensure it is enabled.

The result field is required. The rule description is also required and is used to identify the rule in the administration page.

## Calculation Lines

Each line supplies a field value or a constant and an operation:

- **Field value** reads a value from a supported field on the current work item.
- **Const value** uses a manually entered value instead of a work item field.
- **Data Type** determines the type of a constant, such as String, Integer, Double, Boolean, PlainText, Identity, or PickList where supported by the field.
- **Calculation Method** selects Addition, Subtraction, Multiplication, or Division.
- **Left Bracket** and **Right Bracket** are advanced options for grouping expressions.

Fields used in calculations are selected from the current process and must use a supported value type. System State, Work Item Type, ID, and Parent are excluded from calculation field selection.

Use compatible values and result fields. For example, numeric calculations should use numeric fields or numeric constants, and a numeric result should be written to a numeric field.

## Filters

Filters limit when the calculation is applied. Supported operators depend on the field type and include:

- Equals and Not Equals
- Contains for supported text and path fields
- Greater Than, Less Than, Greater Than or Equals, and Less Than or Equals for supported numeric fields

Multiple filter conditions can be organized into logical groups. Use filters when a calculation should apply only to selected work items.

## Execution and Testing

Calculation Rules run when the configured work item event is processed. The **Parent Rule Tester** does not preview Calculation Rules, so validate a new calculation on a test work item before relying on it in production.

Rules can be disabled without deleting them. Disabled rules remain available for editing and export but are not applied.

## Import and Export

Calculation Rules can be included in the **Import / Export** project-settings page. Exported bundles may contain Parent Rules, Work Item Calculation Rules, and Sibling Rules. Importing an equivalent calculation rule skips it as a duplicate, even if the imported rule has a different generated ID.

Child Calculation Rules are reserved for a planned future engine and are not currently supported.
