---
name: Explicit Measures
description: Use this skill when the user is building, reviewing, or modifying Power BI visuals (charts, cards, tables, matrices) that aggregate numeric fields directly from the fields pane. Detects implicit measures (automatic aggregations like Sum of, Average of, Count of) and replaces them with versioned, explicit DAX measures in the model. Also triggers on mentions of "measure", "KPI", "calculated field", "aggregation", or when the user pastes/describes a visual with fields dragged in without an associated DAX measure.
---

## Goal

Prevent Power BI visuals from depending on implicit measures (automatically created by the engine when a numeric field is dragged in and an aggregation is applied from the UI). Every number that appears in a visual must originate from an explicit DAX measure — named, documented, reusable, and **hosted in a dedicated measures table**, not in fact or dimension tables.

## Why it matters

- Implicit measures cannot be referenced from other measures or reused across visuals.
- They don't allow control over formatting, custom filter context, or conditional logic.
- They make the model harder to trace and maintain as the report grows.
- They create inconsistencies if two visuals aggregate the same field differently without it being documented.
- If measures stay "stuck" inside fact tables, they get visually mixed in with data columns in the fields pane, get lost when the model is restructured, and it becomes harder to see the full KPI catalog at a glance.

## ⚠️ Mandatory execution tool: powerbi-modeling-mcp

**Every creation or modification of tables and measures described in this skill must be executed through the `powerbi-modeling-mcp` MCP (https://github.com/microsoft/powerbi-modeling-mcp). Directly editing `.tmdl` files as plain text (`model.tmdl`, `relationships.tmdl`, files under `tables/`) for these operations is prohibited.**

**Why:**
- The MCP connects to the real tabular engine (Analysis Services/XMLA) over Power BI Desktop, a Fabric workspace, or the PBIP itself, and applies changes transactionally, validated by the engine.
- Editing TMDL as text doesn't validate syntax, cross-references, relationships, or `diagramLayout.json` — this is the typical cause of a schema breaking when going from a healthy model version to a corrupted one after creating tables/measures "by hand."
- The MCP supports transactions (`transaction_operations`: begin/commit/rollback), so a failed change doesn't leave the model half-done.

**Mandatory flow:**

0. **Connect before operating.** If the model lives in a PBIP, use the connection prompt/tool for the `definition/` (TMDL) folder of the semantic model — do not open those files directly in a code editor to write to them. If it's Power BI Desktop or Fabric, connect to the corresponding file or workspace.
1. **Dedicated measures table** (step 0 of the original process): create/verify the table via `table_operations` (create/get/list), never by manually creating a `.tmdl` table file.
2. **Measure creation**: use `measure_operations` (create/update/rename/move) to write the DAX formula, the explicit numeric format, and the display folder — never by inserting the `measure` block by hand in the `.tmdl`.
3. **Relationships** (if applicable): use `relationship_operations`, never by editing `relationships.tmdl` directly.
4. **Batch changes**: if several measures/tables need to be created in the same session, wrap them in a transaction (`transaction_operations`) so you can roll back if something fails midway.
5. **Validation**: use `dax_query_operations` to confirm the measure returns the same result as the implicit aggregation it replaces, instead of inspecting the `.tmdl` by eye.

If at any point there is no active connection to the model (for example, working offline only on the file repository), **stop and ask the user to open the model in Power BI Desktop/Fabric and confirm the MCP connection** before proposing any table or measure creation. Do not make the change "by hand" as a temporary workaround — that's exactly what breaks the schema.

## When to apply this skill

- At the start of every work session on a Power BI file/model.
- Every time the user describes, pastes, or creates a new visual.
- Every time an existing visual is modified (a field is added or changed in the values area).
- When the user asks for help to "chart," "sum," "average," "count," or "show" a field.

Does not apply to fields used only as axes, legends, or filters (categories, dates, slicers) — the focus is exclusively on the **Values** area.

## Step-by-step process

0. **Verify dedicated measures table** (via the MCP's `table_operations`): before creating any measure, confirm the model has a dedicated table to host them (e.g. `_Measures`, `Measures`, `Metrics`). If it doesn't exist:
   - Create it via `table_operations.create` as a table with no rows and no relationships to the rest of the model.
   - Hide any dummy column it comes with by default (`column_operations.update`).
   - All new measures (and, when reasonable, existing ones loose in fact tables) get organized here, not in `Sales_Table`, `Customers_Table`, etc.
   - If the user already has a different convention (e.g. one measures table per business area: `_Measures Sales`, `_Measures Finance`), respect it instead of imposing a single table.
1. **Visual inventory**: identify every field placed in the Values area of the visual.
2. **Classification**: for each field, determine whether it is:
   - An explicit measure that already exists → no action needed, just verify it's referenced correctly.
   - An implicit measure (raw field with automatic aggregation: Sum, Average, Count, Min, Max, etc.) → requires intervention.
3. **Search for an existing measure**: before creating a new measure, use `measure_operations.list`/`get` to check whether an equivalent DAX measure already exists in the model (same table, same field, same aggregation). Avoid duplicates.
4. **Measure creation** (if it doesn't exist), via `measure_operations.create`:
   - Write the explicit DAX formula equivalent to the implicit aggregation.
   - Apply the naming convention (see below).
   - Explicitly define the numeric format (currency, percentage, decimals) — never leave it on Automatic.
   - Create the measure directly inside the dedicated measures table (step 0), never in the source fact table.
   - Within that table, use display folders to group by business area, e.g. `Sales`, `Finance`, `Customers`.
5. **Replacement**: remove the raw field from the Values area of the visual and put the explicit measure in its place (this happens in the Report, and doesn't require the modeling MCP).
6. **Validation**: use `dax_query_operations` to confirm the visual shows the same numeric result as before the replacement (same filter context, same total).
7. **Documentation**: if the user maintains a measures dictionary or model documentation, suggest adding the new measure with its description and base table.

## Naming convention for measures

`[Verb/Type] [Entity] [Optional qualifier]`

Examples:
- `Sum of Sales`
- `Net Sales`
- `% Target Achievement`
- `Average Ticket`
- `Count of Active Customers`

Avoid generic names inherited from the original field (e.g. don't leave it as just `Sales` if a field with that name already exists in the table — this creates visual ambiguity in the fields pane).

## Replacement example

**Implicit** (dragged from the pane, "Sum" aggregation applied by the UI):
`Sum of Sales_Table[Amount]`

**Explicit** (DAX measure created via the MCP's `measure_operations.create`, not written by hand in the `.tmdl`):
```dax
Sum of Sales = SUM ( Sales_Table[Amount] )
```

If business logic requires it, go a step further and encapsulate rules:
```dax
Net Sales = 
CALCULATE (
    SUM ( Sales_Table[Amount] ),
    Sales_Table[Type] = "Valid Sale"
)
```

## Validation checklist before considering the visual done

- [ ] The table/measure creation or modification was executed via `powerbi-modeling-mcp` (`table_operations`/`measure_operations`/`transaction_operations`) — **not** by manually editing `.tmdl` files.
- [ ] No field in the Values area shows the automatic-aggregation icon (Σ) without a measure behind it.
- [ ] The measure lives in the dedicated measures table, not in a fact or dimension table.
- [ ] Every new measure follows the naming convention.
- [ ] Every new measure has an explicit numeric format.
- [ ] There are no duplicate measures serving the same purpose.
- [ ] Measures are organized into a display folder.
- [ ] The visual's numeric result didn't change after the replacement (validated with `dax_query_operations`).

## What NOT to do

- Do not directly edit `.tmdl` files (plain text) to create or modify tables, measures, or relationships — always use `powerbi-modeling-mcp`, even if it "works" in the short term.
- Do not create a new measure if an equivalent one already exists — reuse it.
- Do not create measures loose inside fact or dimension tables even if they technically work — they must live in the dedicated table.
- Do not leave the measure's format as "Automatic."
- Do not apply this rule to fields used as category/axis/filter, only to those in the Values area.
- Do not assume the filter context without verifying it — validate that the total matches before and after the replacement, using `dax_query_operations` instead of manual inspection.