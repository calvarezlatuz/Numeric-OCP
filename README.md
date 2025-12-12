# CHEMOSTAT Optimal Control

Toolkit for solving and visualizing optimal control problems in chemostat systems with mutation.

## View current results
 **[plots/index.html](plots/index.html)**

## Files

### `chemostat_solver.jl`
Optimal control solver for maximizing production and biodiversity on a chemostat systems with mutation using JuMP and Ipopt.

### `batch_runner.jl`
Runs multiple simulations with different parameters and saves results to CSV files.

### `plot_results.py`
Generates interactive HTML visualizations from simulation results.

## Run simulations (Julia)
```julia
include("chemostat_solver.jl")
include("batch_runner.jl")
include("batch_runner.jl")

# Basic production maximization
sol, status = CHEMOSTAT(
    model=1,                      # Maximize production
    params=(0, 1, 0, 0.1),       # ε=0, α=1, β=0, γ=0.1
    tf=30.0,
    nsteps=150,
    display=true
)

# Extract results
t = sol.t
x = sol.x  # Species concentrations (n × time)
s = sol.s  # Substrate
u = sol.u  # Control (dilution rate)

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



## 📊 File Naming

- **One criterion**: `ICX-TY.csv` (e.g., `IC1-T1.csv`)
- **Both criteria**: `ICX-TY-aZ-bW.csv` (e.g., `IC1-T1-a0.5-b0.3.csv`)



