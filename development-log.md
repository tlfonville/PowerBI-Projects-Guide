# Development Log

*A lightweight daily notebook for quick progress notes and reminders.*  
Keep entries short (3–8 bullets). One section per day.

> Tip: Markdown is preferred, but plain text is fine. If you have plain notes you want in Markdown, you can quickly convert with tools like **Pandoc** (`pandoc input.txt -o output.md`) or paste into a `.md` file and tidy in VS Code with the **Markdown All in One** extension.

---

## Daily Entry Template (copy/paste)

```md
## YYYY-MM-DD

**Who:** [Your Name]

**What changed**
- Visuals:
- DAX/measures:
- Data/model:

**Why / decisions**
-

**Data updates**
- Raw added: data/raw/Raw_<Source>_<YYYY-MM-DD>.csv
- Clean replaced: data/ExampleData.csv (archived previous to data/archive/ExampleData_<YYYY-MM-DD>.csv)

**Publish / refresh**
- Workspace: [name] | Refresh: [Manual/Auto] | Result: [OK/Fail + note]

**What’s next**
- [ ] Task 1
- [ ] Task 2
```

---

## Example (optional — delete after first use)

### 2025-10-24
**Who:** Jane Doe

**What changed**
- Visuals: Updated Queue Performance chart to use 15-min bins.
- DAX/measures: Added `Abandon Rate %` and `Avg Wait (s)` (see DAX Library).
- Data/model: Trimmed unused columns in Calls table.

**Why / decisions**
- Align with Help Desk reporting cadence; simpler comparisons across shifts.

**Data updates**
- Raw added: `data/raw/Raw_Genesys_2025-10-24.csv`
- Clean replaced: `data/ExampleData.csv` (archived previous to `data/archive/ExampleData_2025-10-23.csv`)

**Publish / refresh**
- Workspace: Help Desk Analytics | Refresh: Manual | Result: OK

**What’s next**
- [ ] Add slicer for “After-hours vs Business Hours”
- [ ] Validate abandon calc against Analytics UI
