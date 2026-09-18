# Sibling Rules Guide

Sibling Rules enable you to automate state and field updates across related work items that share the same parent. This is useful for dependent tasks, sequential workflows, and team coordination scenarios.

## What Are Sibling Rules?

Siblings are work items that have the same parent work item. In a typical hierarchy:

```
Feature (Parent)
├── Task A (Development)
├── Task B (Testing)
├── Task C (Deployment)
└── Task D (Documentation)
```

All four tasks are siblings because they share the same parent (Feature).

**Sibling Rules** automatically update the state, fields, or both for one or more siblings when another sibling changes state. This enables workflow automation without complex manual coordination.

## Real-World Use Cases

### Use Case 1: Sequential Task Workflow

**Scenario:** Your team works through tasks in a strict sequence: first development, then review, then testing.

**Problem:** Developers forget to mark the review task as ready when they finish development.

**Solution:** Create a Sibling Rule that automatically moves the next task to "Ready" when the current task moves to "Closed".

**Configuration:**
- Trigger when: Task transitions to `Closed` (with tag "DEV")
- Update: Next Task (with tag "REVIEW") to `Ready`
- Mode: `Next` (only the next task in sequence)

**Result:** When a dev task is closed, the review task automatically moves to "Ready" - the next person in line knows it's their turn.

---

### Use Case 2: Bulk Dependency Resolution

**Scenario:** An approval phase blocks multiple dependent tasks. When approval is complete, many tasks should start.

**Problem:** Manager must manually update 10+ tasks after approval.

**Solution:** Create a Sibling Rule that updates all dependent tasks at once when approval is complete.

**Configuration:**
- Trigger when: Task transitions to `Closed` (with tag "APPROVAL")
- Update: All Tasks (with tag "BLOCKED")
- To state: `Ready`
- Mode: `All` (all matching tasks)
- **Field Setter:** Tags → Operation `Remove tags` → `BLOCKED`

> **Tip:** The Field Setter allows you to modify additional fields beyond just the state. Common uses:
> - Remove status tags after updating (e.g., remove "BLOCKED" when moving to "Ready")
> - Set custom fields (e.g., set "Department" field)
> - Clear or update field values as part of the workflow

**Result:** When the approval task closes, all blocked tasks automatically transition to "Ready" and the "BLOCKED" tag is removed simultaneously.

---

### Use Case 3: Phase Gate Automation

**Scenario:** Work items progress through phases (Planning → Development → Review → QA). Each phase has multiple work items that should complete before the next phase starts.

**Problem:** Phase gates are managed manually - no automation to prevent starting a phase before the previous one completes.

**Solution:** Create Sibling Rules for each phase transition using tags to identify work items in each phase. When all tasks in one phase are marked as complete, automatically activate tasks in the next phase.

## Getting Started

### Step 1: Access Sibling Rules Admin

1. Open your Azure DevOps project
2. Navigate to **Project Settings** → **Extensions** → **COSMO Smart Automate** → **Sibling Rules**

### Step 2: Understand the Rule Components

When creating a Sibling Rule, you'll configure:

| Component | Purpose | Example |
| --------- | ------- | ------- |
| **Trigger** | Which work item type and state change activates this rule | Task transitions to "Closed" |
| **Trigger Filters** | Additional conditions the triggering work item must meet | Tag = "DEV" |
| **Sibling Target Type** | What type of sibling to update | Task |
| **Sibling Filters** | Which siblings match the rule | Tag = "REVIEW" |
| **Target State** | Optional: What state to set the matching siblings to | "Ready" |
| **Mode** | How to select siblings to update | "Next" or "All" |
| **Keep Assignee** | Preserve the sibling's current assignee | On/Off |
| **Field Setter** | Optional: Additional fields to modify when updating (e.g., remove tags) | Remove tag "BLOCKED" |
| **Options** | Additional behavior settings | Keep the original assignee |

The trigger state, sibling type, and mode are required. Target and excluded states are optional. Leave **Target State** empty when the rule should only update fields through the **Field Setter**.

Use the **Rule enabled** toggle to disable a rule temporarily without deleting it. Disabled rules remain saved for editing and export but are skipped during processing.

### Step 3: Create Your First Rule

**Simple Example:** Update the next task when current task closes

1. Click **+ Add Rule**
2. **Trigger Section:**
   - Work item type: `Task`
   - Transition state: `Closed`
   - (Leave trigger filters empty for now)

3. **Sibling Target Section:**
   - Sibling type: `Task`
   - Sibling target state: `Ready` (optional; leave empty for a field-only rule)

4. **Sibling Filters Section:**
   - Add filter: Field = "Tag", Operator = "Equals", Value = "REVIEW"

5. **Options Section:**
   - Mode: `Next`
   - Keep assignee: `On`

6. Click **Save**

**How it works:**
- When ANY task transitions to "Closed"
- Find all sibling tasks that are not "Closed"
- Update only the FIRST one (Next mode, by Stack Rank + ID)
- Set it to "Ready"
- Preserve its current assignee

---

### Field-Only Example: Tag the Next Sibling Without Changing Its State

Use an empty **Sibling target state** when the rule should update a field but leave the matching sibling's state unchanged.

**Configuration:**
- Trigger when: Task transitions to `Closed` (with tag `APPROVAL`)
- Update: Next Task (with tag `BLOCKED`)
- Sibling target state: Leave empty
- Mode: `Next`
- Field Setter: Tags -> Operation `Add tags` -> `READY_FOR_REVIEW`

**Result:** The next matching sibling keeps its current state and receives the `READY_FOR_REVIEW` tag. No state transition is sent for that sibling.

---

## Modes: "All" vs "Next"

### Mode: "All"

Updates **every sibling** that matches your filters.

**Use when:**
- Multiple siblings should update simultaneously
- Dependencies are released together
- Bulk status changes across work items
- Parallel workflows (not sequential)

**Example:** When approval completes, mark all dev tasks as "Ready"

**Behavior:**
```
Rule triggers on Task A (APPROVAL) → Closed
All siblings matching "TAG=DEV" → Updated to "Ready"
Result: Tasks B, C, D all become "Ready" at once
```

### Mode: "Next"

Updates **only the next sibling** in sequence (sorted by Stack Rank, then work item ID).

**Use when:**
- Work items must be done in a specific order
- Only one task should be active at a time
- Sequential hand-offs between team members
- Queue-based workflows

**Example:** When Dev task closes, move QA task to "In Progress"

**Behavior:**
```
Rule triggers on Task A (DEV) → Closed
Sibling matches TAG=QA (Tasks B, C, D)
Only Task B selected (first in Stack Rank order)
Task B → Updated to "In Progress"
Tasks C, D remain unchanged
```

**Pro Tip:** Order your sibling work items using Stack Rank to control the sequence. Drag tasks in the backlog or work item editor to adjust priority.

---

## Filtering Siblings

The **Sibling Filters** determine which siblings are affected by the rule.

### Available Filter Fields

You can filter siblings by:
- **Tags** (recommended) - Most reliable way to categorize work items
- **Custom Fields** - Project-specific fields
- **Text Fields** - Title, description, area path, iteration path

> **Important:** The following fields are **NOT** available as filters for design reasons (to prevent circular rule logic):
> - `State` - Use tags to categorize instead
> - `Work Item Type` - Set the sibling type directly in the rule
> - `ID` - Not filterable
> - `Parent` - Siblings already share the same parent

> **Note:** Trigger filters and sibling filters only offer fields of the work item itself. There is no parent field selection for Sibling Rules, since siblings are matched through the shared parent work item and not through a fixed parent type.

### Common Filter Patterns

**Pattern 1: Filter by Tag**
```
Tag = "REVIEW"
→ Only updates siblings with the "REVIEW" tag
```

**Pattern 2: Filter by State Exclusion**
```
This is NOT possible - State field is excluded from filters.
Instead, use tags to mark which tasks should/shouldn't be updated.

Example:
Tag = "ACTIVE" AND Tag ≠ "BLOCKED"
→ Only updates siblings with ACTIVE tag that don't have BLOCKED tag
```

**Pattern 3: Filter by Field Value**
```
Priority = "High"
→ Only updates high-priority siblings
```

**Pattern 4: Complex Filter (Multiple Conditions)**
```
Tag = "DEV" AND Custom Field "Status" = "Ready"
→ Dev tasks that are marked Ready in a custom field
```

### How Filters Work

- **Within a filter group:** All configured conditions must be true (AND).
- **Across filter groups:** At least one complete group must match (OR).
- **Use separate groups** to express alternatives, then combine conditions within each group for more specific matching.

Example: `(Tag = "DEV" OR Tag = "TESTING") AND Custom Status = "New"`
- Must have either DEV or TESTING tag
- AND must have the custom status value New

---

## Advanced Features

### Keep Assignee Option

When enabled, preserves the current assignee of sibling work items when their state changes.

**Example:**
- Sibling Task is assigned to Sarah, state is "New"
- Rule triggers and updates task to "Ready"
- Assignee remains Sarah (not changed to rule creator)

**When to use:**
- Keeping work item context (don't want to lose the owner)
- Sequential workflows where the next person is already assigned
- Preventing accidental ownership changes

### Field Setter Option

Automatically modify work item fields when the sibling rule updates. This goes beyond just changing the state—you can set custom fields, clear values, or modify tags.

The sibling target state is optional. Leave it empty to apply only the configured field setters; matching siblings keep their current state.

**Supported Fields:**
- **Tags** - Set, add, or remove tags
- **Custom Fields** - Any text, number, or custom field in your work item type
- **Built-in Fields** - Priority, severity, area path, iteration path, etc.

**Common Use Cases:**

1. **Remove Status Tags After Workflow Progression**
   - When updating from "Waiting" → "Ready", remove the "BLOCKED" tag
   - When moving to "In Progress", remove both "PENDING" and "READY" tags
   - Clean up workflow-stage tags as work progresses

2. **Set Responsible Party**
   - Automatically assign to team lead when moving to "Review" state
   - Assign to QA when transitioning to "Testing"

3. **Update Custom Tracking Fields**
   - Set "Phase" field to match the current stage
   - Mark "Date Activated" with current date
   - Set "Department" based on the workflow stage

**How to Configure Field Setters:**

1. In the Sibling Modal, open the **Details** tab and find the **Field Setter** section
2. Click **"Add"** to add a new field modification
3. Select the **Field** dropdown to choose which field to modify (Tags, Custom Fields, etc.)
4. For the **Tags** field, choose an **Operation**:
   - **Set tags (replaces all tags)** - the selected tags become the complete tag list
   - **Add tags** - the selected tags are added, existing tags are kept
   - **Remove tags** - the selected tags are removed, all other tags are kept
5. Select or enter the **Value** to set

> **Note:** Field Setters for Sibling Rules always apply to the siblings being updated. The parent option is not offered, since siblings are matched by the shared parent work item and not by a fixed parent type.

**Example: Remove a Tag**

To remove the "BLOCKED" tag from a work item that has tags `BLOCKED;DEV;PHASE-2`:

1. **Field:** `Tags`
2. **Operation:** `Remove tags`
3. **Value:** `BLOCKED`

The sibling keeps `DEV;PHASE-2` and loses only the "BLOCKED" tag. Tag matching is case-insensitive.

**Example: Add a Tag**

To add an "IN-PROGRESS" tag to work items:

1. **Field:** `Tags`
2. **Operation:** `Add tags`
3. **Value:** `IN-PROGRESS`

Existing tags are kept, and the tag is only added if it is not already present.

---

## Testing Your Rules

### Dry Run Test

Sibling Rules cannot be previewed. The **Parent Rule Tester** only simulates Parent Rules, so validate Sibling Rules on test work items.

### Real-World Testing

1. Create test work items with the same parent
2. Add appropriate tags/fields matching your filters
3. Transition the trigger work item to the target state
4. Observe which siblings update and verify behavior
5. Check work item history to see rule execution details

### Debugging Tips

If a rule isn't working:

1. **Verify the trigger:** Is the work item type and state transition correct?
2. **Check trigger filters:** Do they match your test work item?
3. **Review sibling filters:** Do siblings actually match the filters?
4. **Confirm the parent:** Do siblings share the same parent work item?
5. **Check permissions:** Rule engine needs read/write access to work items
6. **Review history:** Check work item history/updates for error messages

---

## Common Mistakes & Solutions

### ❌ "Rule never triggers"

**Causes:**
- Trigger filters too restrictive (no work items match)
- Working in a view that doesn't support rules (Boards, Query)
- Rule disabled in admin page
- Parent/child hierarchy not set correctly

**Solution:**
- Simplify the trigger filters and confirm the trigger work item matches them
- Save state changes in Work Item Form or Backlog
- Enable the rule in admin page
- Confirm parent work item relationship exists

---

### ❌ "Siblings not updating"

**Causes:**
- Sibling filters excluding all work items
- No siblings match the filter criteria
- Mode is "Next" but no siblings in sequence
- Sibling type mismatch

**Solution:**
- Check which siblings match the sibling filters by reviewing them in a query
- Simplify filters to start (remove unnecessary conditions)
- Check parent work item - siblings must share same parent
- Verify Stack Rank order for "Next" mode

---

### ❌ "Wrong siblings updating"

**Causes:**
- Sibling filters too loose/broad
- Mode "All" when you meant "Next"
- Multiple matching rules with conflicting actions

**Solution:**
- Make filters more specific (add tag or field filters)
- Change to "Next" mode if only one should update
- Disable conflicting rules or adjust priorities
- Review rule order in admin page

---

## Best Practices

✅ **DO:**
- Use **tags** to categorize work items and filter siblings
- Use **Field Setters** to clean up tags as work progresses through workflow stages
- Test rules with **test work items** before enabling in production
- Keep **filter logic simple** - complex filters are hard to maintain
- Document **rule purpose** in the description/group key
- Use **Stack Rank** to control sequence for "Next" mode
- Start with **simple rules** and build complexity gradually

❌ **DON'T:**
- Create rules that cause **circular updates** (A→B, B→A)
- Use **overly broad filters** (rules that match everything)
- Enable rules **without testing first**
- Rely on **field values** for filters (use tags instead - more reliable)
- Create **duplicate rules** that do the same thing

---

## Rule Examples

### Example 1: Bug Triage Workflow

```
Rule: Mark next bug for fixing when current bug fixed

When: Bug transitions to "Resolved" (tag = "CRITICAL")
Update: Next Bug (tag = "CRITICAL", state = "New")
To: "In Progress"
Mode: Next
Keep assignee: On
```

**Result:** Critical bugs automatically queue up for the next developer.

---

### Example 2: Release Phase Management

```
Rule 1: When code review done, start testing

When: Task transitions to "Closed" (tag = "CODE_REVIEW")
Update: All Task (tag = "QA", state = "New")
To: "Ready"
Mode: All
Keep assignee: Off
Field Setter:
  - Remove "QA" tag from Tags field (if you prefer)
  - OR set Tags = "QA;ACTIVE" (add workflow tag)
```

```
Rule 2: When testing done, start deployment

When: Task transitions to "Closed" (tag = "QA")
Update: All Task (tag = "DEPLOY", state = "Ready")
To: "In Progress"
Mode: All
Keep assignee: On
Field Setter:
  - Set Tags = "DEPLOY;ACTIVE" (mark as active)
  - Set Custom Field "Phase" = "Deployment"
```

**Result:** Release phases progress automatically from review → testing → deployment, with automatic tag and field updates.

---

### Example 3: Story Point Estimation Workflow

```
Rule: When estimation done, start planning

When: Task transitions to "Closed" (tag = "ESTIMATION")
Update: Next Task (tag = "PLANNING", type = "Task")
To: "In Progress"
Mode: Next
Keep assignee: Off
```

**Result:** Planning tasks are sequentially activated as estimation completes.

---

## Performance Considerations

### Rule Processing

- Rules execute **every time** a work item transitions state
- Processing happens **asynchronously** (doesn't delay your save)
- **Filter evaluation** is optimized (complex filters have minimal impact)
- **Cascade effects** occur if rules trigger other rules (by design)

### Best Practices for Performance

- **Avoid overly complex filter combinations**
- **Use "Next" mode instead of "All"** when possible (updates fewer items)
- **Limit rule count** per work item type (10-20 rules is healthy)
- **Test filter logic** to ensure they're selective

---

## Troubleshooting

### Check the Extension Logs

1. Open browser Developer Tools (F12)
2. Go to **Console** tab
3. Filter by "COSMO Smart Automate" or "SiblingRule"
4. Look for error messages

### Review Rule Execution

There is no separate in-app logging toggle. To inspect rule errors:
- Open browser Developer Tools (F12) and select the **Console** tab
- Filter by "COSMO Smart Automate" or "SiblingRule"
- Check work item update history for rule execution details

### Contact Support

If issues persist:
- Include error messages from logs
- Describe the rule configuration
- Provide example work items that aren't working
- Include project/process details

---

## FAQ

**Q: Can sibling rules work across different work item types?**
A: Yes! Siblings just need to share the same parent. The parent could have mixed types (Tasks, Bugs, Features).

**Q: What happens if no siblings match the filters?**
A: Nothing. The rule executes, finds no matches, and completes silently. This is not an error.

**Q: Can I chain sibling rules together?**
A: Yes. When a sibling updates state, other sibling rules can trigger. Be careful to avoid circular chains.

**Q: Does "Next" mode work with only one sibling?**
A: Yes. If there's only one matching sibling, it updates. "Next" means "the next in sequence" - even if there's only one.

**Q: Can I undo a sibling rule update?**
A: Rule updates are visible in work item history. You can manually revert, but there's no undo button for rules.

**Q: Do sibling rules work on work item forms and backlog?**
A: Yes to both. Both save operations trigger rule processing.

**Q: Can I delete a rule that's being used?**
A: Yes, you can delete any rule. Future state changes won't trigger it, but past executions aren't affected.

**Q: Can I remove tags automatically when a sibling rule updates a work item?**
A: Yes. Use the **Field Setter**, select the Tags field and choose the operation **Remove tags**. Only the selected tags are removed, all other tags on the sibling are kept. Example: a task tagged "BLOCKED;DEV" with the operation `Remove tags` and the value `BLOCKED` ends up with "DEV".

**Q: What fields can I set with Field Setter?**
A: Any field available in your work item type - Tags, custom fields, Priority, Area Path, Iteration Path, etc. You can set string, number, and identity fields.

**Q: Can Field Setter remove all tags from a work item?**
A: Yes, use the operation **Set tags (replaces all tags)** without selecting any tag. To keep some tags, either select the tags you want to keep with `Set tags`, or use `Remove tags` and select only the tags you want to get rid of.

---

## Next Steps

1. **Read** [Parent Rules](./PARENT_RULES.md) for parent-rule documentation
2. **Try** creating a simple test rule with test work items
3. **Test** on test work items before enabling for your team
4. **Document** your rules and share configuration with the team
5. **Monitor** work item history to verify rule execution
