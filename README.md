# CHEMOSTAT Optimal Control

Toolkit for solving and visualizing optimal control problems in chemostat systems.

## 📁 Files

### `chemostat_solver.jl`
Improved optimal control solver for chemostat systems using JuMP and Ipopt.
- Handles zero parameters (α, β, γ = 0)
- Prevents control chattering
- Works for any number of species

### `batch_runner.jl`
Runs multiple simulations with different parameters and saves results to CSV files.

### `plot_results.py`
Generates interactive HTML visualizations from simulation results.
- Automatically organizes plots by category (diversity, production, both)
- Creates hierarchical navigation
- Side-by-side comparisons

## 🚀 Quick Start

### 1. Run simulations (Julia)
```julia
include("batch_runner.jl")

configs = [
    (name="diversity_IC1-T1", model=2, x0=[5.0, 0.1, 0.1, 0.1, 0.1], 
     epsilon=0.01, alpha=0.0, beta=1.0, tf=20.0),
]

results = run_chemostat_batch(configs, output_dir="results/diversity")
```

### 2. Generate plots (Python)
```python
from plot_results import generate_all_plots
generate_all_plots("results", "plots")
```

### 3. View results
Open the generated plots: **[plots/index.html](plots/index.html)**

## 📊 File Naming

- **One criterion**: `ICX-TY.csv` (e.g., `IC1-T1.csv`)
- **Both criteria**: `ICX-TY-aZ-bW.csv` (e.g., `IC1-T1-a0.5-b0.3.csv`)

## 📄 License

MIT License



