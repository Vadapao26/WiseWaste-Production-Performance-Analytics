Analyzes production run data across processes, equipment, and shifts — computing staff efficiency, equipment throughput, rejection rates, and material category breakdowns across sorting, bagging, and bailing operations.

What it does:


Run-level efficiency metrics — Kg per Person, Kg per Hour, Kg per Staff Hour computed per production run, avoiding double-counting from multi-material rows
Process & equipment rankings — efficiency ranked within and across processes; best and worst performers flagged
Shift-wise comparison — daily and shift-level summaries across all process-equipment combinations
Material category mapping — 100+ material names auto-mapped to Paper, Metal, Glass, Plastics, Textile, E-Waste, C&D Waste, Others using exact + fuzzy matching
Rejection tracking — explicit rejection kg and % per process/equipment, plus rejects-appearing-as-output-material flagging
Best/worst run identification — top 5 and bottom 5 days and individual runs by Kg per Person per process
Material flow table — long-format material breakdown per run, with source category tracing for bagging/bailing operations


Quantity logic by process type:

ProcessQuantity SourceSortingQuantity in Kg column (single output row per run)Bagging / BailingSum of Material Quantity rows (multiple material rows per run)

Input: Production_cleaned.csv from the Data Cleaning Pipeline
Output: Inline analysis tables and charts in Colab
