# Calculation Rules

Work Item Calculation Rules calculate a value for a field on the work item that triggered the rule. A calculation can use supported work item fields, constants, arithmetic operations, and optional filters.

Calculation Rules are field-only. They have no transition-state or target-state setting, and the processor writes only the configured result field. The supported field picker excludes `System.State`, so Calculation Rules do not change a work item's state.

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

Fields used in calculations are selected from the current process and must use a supported value type. System State, Work Item Type, ID, and Parent are excluded from calculation field selection and result-field selection.

Use compatible values and result fields. For example, numeric calculations should use numeric fields or numeric constants, and a numeric result should be written to a numeric field.

## Example: Calculate Remaining Work

**Scenario:** Keep a task's Remaining Work field up to date from its Original Estimate and Completed Work values.

Configure the calculation as follows:

| Calculation line | Value | Calculation method |
| ---------------- | ----- | ------------------ |
| 1 | Field value: `Original Estimate` | Subtraction |
| 2 | Field value: `Completed Work` | Addition |

In the editor, the method on a row is the operation between that row and the next row. The final row still requires a method selection, but the processor ignores it because there is no following row; `Addition` is a safe value to select for the final row.

Set **Result field** to `Remaining Work` and choose an integer result type. When the calculation rule runs, it writes `Original Estimate - Completed Work` to `Remaining Work`. The work item's state is not used as an operand or result and is not changed by this rule.

To enter this example:

1. On **Details**, enter `Calculate remaining work` and select `Remaining Work` as the result field.
2. On **Calculation**, add a field-value line for `Original Estimate` and select `Subtraction`.
3. Add a second field-value line for `Completed Work` and select `Addition` to satisfy the editor's required method field.
4. Save the rule. The effective expression is `Original Estimate - Completed Work`.

The calculation field picker shows field values such as `Original Estimate`; the result-field picker shows the user-facing field name such as `Remaining Work`.

## Filters

Filters limit when the calculation is applied. Supported operators depend on the field type and include:

- Equals and Not Equals
- Contains for supported text and path fields
- Greater Than, Less Than, Greater Than or Equals, and Less Than or Equals for supported numeric fields

Multiple filter conditions can be organized into logical groups. Use filters when a calculation should apply only to selected work items.

## Execution, State Behavior, and Testing

The work item observer tracks the field reference names used by calculation lines. On save, it invokes the Calculation Processor only when at least one of those referenced input fields changed. The processor then evaluates matching enabled rules and writes each result with a field update.

- Changing only `System.State` or an unrelated field does not trigger Calculation Rules.
- Calculation Rules never call the state-update operation and do not change `System.State`.
- A rule that contains only constants has no referenced input field, so it has no automatic field-change trigger in the current observer. Include at least one field-value line when the rule should run automatically.

The **Parent Rule Tester** does not preview Calculation Rules, so validate a new calculation on a test work item before relying on it in production.

Rules can be disabled without deleting them. Disabled rules remain available for editing and export but are not applied.

## Import and Export

Calculation Rules can be included in the **Import / Export** project-settings page. Exported bundles may contain Parent Rules, Work Item Calculation Rules, and Sibling Rules. Importing an equivalent calculation rule skips it as a duplicate, even if the imported rule has a different generated ID.

Child Calculation Rules are reserved for a planned future engine and are not currently supported.
