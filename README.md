# 🏭 Production Analytics

Analyzes production run data across sorting, bagging, and bailing operations — computing staff efficiency, equipment throughput, rejection rates, material category breakdowns, and best/worst run identification across shifts and process-equipment combinations.


## The Problem It Solves

Production records have a complex multi-row structure — one production run has one row for time/staff, but multiple rows for each output material. Calculating `Kg per Person` or `Kg per Hour` naively by summing quantities across all rows gives inflated numbers because staff and hours are repeated on every material row.

On top of that, sorting and bagging/bailing use different quantity columns — comparing them directly gives misleading efficiency numbers.

Managers were spending **2–3 hours per facility per month** manually computing efficiency metrics, identifying slow shifts, and building equipment comparisons. This notebook produces the full analysis in **under 90 seconds**.

---

## Architecture

```
Cleaned Production CSV Upload
            │
            ▼
┌──────────────────────────────────┐
│  1. Column Validation            │  ← checks all required columns present
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  2. Process/Equipment Split      │  ← "Sorting-Manual" → Process + Equipment
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  3. Run-Level Table Build        │  ← staff/time taken ONCE per run (max agg)
│     (avoids multi-row inflation) │
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  4. Quantity Logic by Process    │
│   ├── Sorting → Quantity in Kg   │
│   └── Bagging/Bailing → sum of   │
│        Material Quantity rows    │
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  5. Efficiency Metrics Compute   │  ← Kg/Person, Kg/Hour, Kg/Staff Hour
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  6. Material Category Mapping    │  ← exact + fuzzy + keyword fallback
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  7. Multi-Dimensional Analysis   │
│   ├── Overall by Process         │
│   ├── Process × Equipment        │
│   ├── Daily & Shift Breakdowns   │
│   ├── Equipment Rankings         │
│   ├── Best / Worst Runs          │
│   └── Material Flow Table        │
└───────────────┬──────────────────┘
                │
                ▼
         Inline Colab Output
```

---

## Languages & Libraries

| Layer | Tool | Purpose |
|---|---|---|
| Language | Python 3 | Core logic |
| Data processing | Pandas | Multi-level groupby, merge, concat |
| Numeric ops | NumPy | Conditional efficiency calculations |
| Fuzzy matching | `difflib.get_close_matches` | Material name normalization (0.88 cutoff) |
| Text processing | `re`, `unicodedata` | Material name canonicalization |
| Environment | Google Colab | Upload / display |

---

## Key Metrics Computed

### Run-Level Efficiency
| Metric | Formula |
|---|---|
| `Kg per Person` | Production Qty ÷ No. of Staff Present |
| `Kg per Hour` | Production Qty ÷ Time Taken in Hrs |
| `Kg per Staff Hour` | Production Qty ÷ (Staff × Hours) |
| `Explicit Rejection %` | Total Rejection in Kg ÷ Production Qty × 100 |

### Quantity Source by Process
| Process | Quantity Column Used |
|---|---|
| Sorting | `Quantity in Kg` (single output row per run) |
| Bagging | Sum of `Material Quantity` rows (multiple per run) |
| Bailing | Sum of `Material Quantity` rows (multiple per run) |

---

## Analysis Outputs

### Overall Process Analysis
- Total Kg produced per process type
- Avg Kg/Person, Kg/Hour, Kg/Staff Hour per process
- Explicit rejection kg and % per process
- Rejects appearing as output material (tracked separately)
- % of total production per process

### Process × Equipment Analysis
- Same efficiency metrics broken down per equipment within each process
- Equipment ranked by Kg/Person, Kg/Hour, Kg/Staff Hour within process
- Rejection rate ranked (lowest = best) within process

### Daily & Shift Breakdowns
- Daily totals by process — spot which dates underperformed
- Shift-level summaries (Shift × Process × Equipment)

### Best / Worst Run Identification
- Top 5 production runs by Kg/Person per process type
- Bottom 5 production runs by Kg/Person per process type
- Top 5 / Bottom 5 days by Kg/Person per process type

### Material Flow Table (Long Format)
- Every material output per run with quantity, category, and source category
- For bagging/bailing: traces which material came from which source
- Material category mix per process-equipment combination
- % contribution of each material within each process-equipment

### Material Category Mapping (3-pass)
1. Exact match against known material names
2. Normalized exact match (lowercase, strip punctuation, unicode normalize)
3. Fuzzy match (`get_close_matches` at 0.88 cutoff)
4. Keyword fallback (e.g. any name containing `"pet"` → Plastics)

Categories: Paper, Metal, Glass, Plastics, Textile Waste, E-Waste, Wet Waste, Sanitary Waste, C & D Waste, Others

---

## Use Cases

- **Shift performance review** — facility managers see exactly which shifts and equipment combinations are underperforming vs the facility benchmark
- **Equipment allocation decisions** — equipment rankings show which machines produce the most per staff hour — inform maintenance and procurement priorities
- **Staffing optimization** — Kg/Staff Hour metric shows whether adding staff improves output or creates bottlenecks
- **Rejection root cause** — flagging rejects-as-output-material separately from explicit rejection helps distinguish process rejection from material quality issues
- **Material flow auditing** — long-format table shows exactly which materials passed through which equipment, useful for reconciliation with inward and outward records

---

## Time Saved

| Task | Manual (Excel) | This Notebook |
|---|---|---|
| Deduplicating staff/time across multi-material rows | 30–45 min | < 5 sec |
| Computing Kg/Person and Kg/Staff Hour | 20–30 min | < 5 sec |
| Shift-level efficiency comparison | 20–30 min | < 5 sec |
| Equipment ranking per process | 20–30 min | < 5 sec |
| Best/worst run identification | 15–20 min | < 5 sec |
| Material flow tracing | 30–45 min | < 5 sec |
| **Total per reporting cycle** | **~2–3 hours** | **< 90 sec** |

**Estimated time saving: ~97% reduction per reporting cycle.**

---

## Input / Output

| | Detail |
|---|---|
| **Input** | `Production_cleaned.csv` from the [Data Cleaning Pipeline](../data-cleaning-pipeline/) |
| **Output** | Inline analysis tables displayed in Colab |

---

## How to Use

### Google Colab
1. Click **Open in Colab** above
2. Run all cells — you'll be prompted to upload a file
3. Upload your cleaned production CSV
4. All analysis tables appear inline in the notebook

