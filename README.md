# Grouped Operator Scheduling & Balancing Tool (Gurobi Optimization)

This Gurobi model is the second iteration of my **Line Balancing Optimization Tool** developed during my internship at MSA.  
It builds on Model 1’s station-level operator allocation by introducing **group constraints** and a **two-phase optimization** to create implementable daily schedules — not just numbers in a table.

**Goal:** Assign a fixed number of operators to sequential stations (with real-world group restrictions) to first **maximize throughput**, then **minimize workload imbalance** between stations.

This project was built using my **academic Gurobi license**, as MSA did not have one available.  

It replaced hours of manual Excel guesswork with a mathematically optimal, schedule-level staffing plan — making recommendations both faster to generate and easier to justify.

---

## Background & Context

The production line has **25 sequential stations** with known average cycle times (minutes/unit).  
Operators are split into **hard-coded groups** based on layout and staffing rules (e.g., Group 1 = Stations 1–10, Group 2 = Stations 11–25).  
Within each group:
- Stations must have **at least 1 operator**.
- **Parallel work** is allowed: long tasks can be split between operators to reduce time proportionally.
- Each operator may work at **no more than 2 stations** in their group during an hour.
- Each station may have **no more than 2 operators** assigned per day.

---

## Two-Phase Optimization Approach

**Phase 1 — Throughput Maximization**
- Maximizes `mu` (average units/hour across stations).
- Uses fractional time allocations to allow realistic splitting within the 2-per-station cap.
- Respects all group, station, and assignment limits.

**Phase 2 — Workload Balancing**
- Keeps throughput at or above Phase-1 level (with a small tolerance).
- Minimizes `∑(throughput[i] – avg_throughput)^2` to smooth station performance.
- Warm-starts from Phase-1 assignments for faster solve time.

- In simple terms: Phase 1 tells you the best possible output under constraints. Phase 2 shows how close you can get to equal workloads without sacrificing that speed.

---

## Core Constraints

For each station `i` and operator `j`:
- **Operator hour budget:** `∑_i x[j,i] = 1` hour.
- **Assignment rules:** If assigned, `x[j,i] ≥ ε` (5% minimum) and `x[j,i] ≤ z[j,i]` (binary assignment flag).
- **Operator caps:** `∑_i z[j,i] ≤ 2` (max 2 stations/hour).
- **Station caps:** `∑_j z[j,i] ≤ 2` (max 2 operators).
- **Group lock:** Operators can only work in their assigned group’s stations.
- **Throughput definition:**  
  `throughput[i] = ∑_j (60 * x[j,i] / cycle_time[i])`.

---

## Output

The model produces:
- **Station Summary:** throughput (units/hr), daily units (9-hr shift), and deviation from average.
- **Deviation Report:** stations >10% above/below average flagged.
- **Operator Schedule:** for each operator — primary and secondary station, units/hr contribution, minutes/hour allocation.

In simple terms:  
> “Phase 1 tells you the best possible output under constraints. Phase 2 shows how close you can get to equal workloads without sacrificing that speed.”

---

## 💡 Key Note

This project was built using my **academic Gurobi license**, as MSA did not have one available.  
It replaced hours of manual Excel guesswork with a mathematically optimal, schedule-level staffing plan — making recommendations both faster to generate and easier to justify.




