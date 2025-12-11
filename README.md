# CHEMOSTAT Optimal Control

A Julia/Python toolkit for solving and visualizing optimal control problems in chemostat systems using direct collocation methods.

## 🎯 Overview

This repository provides:
- **Improved optimal control solver** for chemostat systems using JuMP and Ipopt
- **Batch simulation runner** for parameter studies
- **Interactive visualization tool** for results analysis using Plotly

### Key Features

- ✅ Works for any number of species (n)
- ✅ Handles zero parameters (α, β, γ = 0) without degeneracy
- ✅ Robust numerical methods (implicit trapezoidal rule)
- ✅ Control regularization to prevent chattering
- ✅ Batch processing for multiple scenarios
- ✅ Hierarchical organization of results
- ✅ Interactive HTML plots with zoom, pan, and hover

## 📋 Requirements

### Julia Dependencies
```julia
using JuMP
using Ipopt
using DelimitedFiles
using Dates
```

### Python Dependencies
```bash
pip install numpy pandas plotly
```

## 🚀 Quick Start

### 1. Setup

Clone the repository:
```bash
git clone https://github.com/yourusername/chemostat-optimal-control.git
cd chemostat-optimal-control
```

### 2. Run Simulations (Julia)

```julia
include("chemostat_solver.jl")
include("batch_runner.jl")

# Define simulation configurations
configs = [
    (name="diversity_IC1-T1", model=2, x0=[5.0, 0.1, 0.1, 0.1, 0.1], 
     epsilon=0.01, alpha=0.0, beta=1.0, tf=20.0),
    (name="diversity_IC1-T2", model=2, x0=[5.0, 0.1, 0.1, 0.1, 0.1], 
     epsilon=0.01, alpha=0.0, beta=1.0, tf=30.0),
    # ... add more configurations
]

# Run batch simulations
results = run_chemostat_batch(configs, output_dir="results/diversity")
```

### 3. Visualize Results (Python)

```python
from plot_results import generate_all_plots

# Generate interactive HTML plots
generate_all_plots("results", "plots", open_in_browser=True)
```

This will automatically create an `index.html` with hierarchical navigation to all plots.

## 📁 Project Structure

```
chemostat-optimal-control/
├── chemostat_solver.jl          # Improved CHEMOSTAT function
├── batch_runner.jl               # Batch simulation runner
├── plot_results.py               # Visualization tool
├── README.md                     # This file
├── results/                      # Simulation outputs
│   ├── diversity/               # Single criterion: diversity
│   │   ├── IC1-T1.csv
│   │   ├── IC1-T2.csv
│   │   └── ...
│   ├── production/              # Single criterion: production
│   │   └── ...
│   └── both/                    # Both criteria
│       ├── IC1-T1-a0.5-b0.3.csv
│       └── ...
└── plots/                       # Generated HTML visualizations
    ├── index.html              # Main navigation page
    └── *.html                  # Individual plots
```

## 📊 File Naming Convention

### One Criterion (Diversity/Production)
Format: `ICX-TY.csv`
- `ICX`: Initial condition identifier (e.g., IC1, IC2)
- `TY`: Time horizon (e.g., T1 = 20.0, T2 = 30.0)

**Example:** `IC1-T2.csv` → Initial condition 1, time horizon 2

### Both Criteria
Format: `ICX-TY-aZ-bW.csv`
- `ICX`: Initial condition
- `TY`: Time horizon
- `aZ`: Alpha parameter value
- `bW`: Beta parameter value

**Example:** `IC1-T1-a0.5-b0.3.csv` → IC1, T1, α=0.5, β=0.3

## 🎨 Visualization Structure

The generated `index.html` organizes results hierarchically:

```
📊 One Criteria
  → Only diversity
    → IC1 (all times side-by-side)
    → IC2 (all times side-by-side)
  → Only production
    → IC1 (all times side-by-side)

📊 Both Criteria
  → IC1
    → T1 (varying α, β side-by-side)
    → T2 (varying α, β side-by-side)
```

Each plot contains three panels:
1. **Top**: State variables (x₁, x₂, ..., xₙ, s)
2. **Middle**: Control input u(t)
3. **Bottom**: Production P(x,u) and Biodiversity S(x)

## 🔧 Key Improvements Over Original Code

### 1. Zero Parameter Handling
**Problem:** When α, β, or γ = 0, the optimization becomes ill-conditioned.

**Solution:** Automatically replaces exact zeros with small values (1e-10):
```julia
α_reg = α == 0 ? 1e-10 : α
β_reg = β == 0 ? 1e-10 : β
γ_reg = γ_raw == 0 ? 1e-10 : γ_raw
```

### 2. Better Convergence
- **Enhanced Ipopt settings**: Adaptive barrier, gradient-based scaling
- **Smart initial guess**: Linear interpolation between initial and expected final states
- **Improved discretization**: Implicit trapezoidal rule (2nd order, A-stable)

### 3. Control Regularization
Uses second-order differences to prevent chattering:
```julia
# Penalizes acceleration in control
du2[j] = (u[j+1] - 2*u[j] + u[j-1])^2
```

### 4. Comprehensive Error Handling
- Input validation
- Clear error messages with diagnostic hints
- Graceful handling of convergence failures

## 📖 Usage Examples

### Single Simulation

```julia
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
```

### Batch Simulations with Different Initial Conditions

```julia
# Varying initial conditions
configs = [
    (name="diversity_IC1-T1", model=2, 
     x0=[5.0, 0.1, 0.1, 0.1, 0.1], 
     epsilon=0.01, alpha=0.0, beta=1.0, tf=20.0),
    
    (name="diversity_IC2-T1", model=2, 
     x0=[5.0, 0.1, 5.0, 0.1, 0.1], 
     epsilon=0.01, alpha=0.0, beta=1.0, tf=20.0),
    
    (name="diversity_IC3-T1", model=2, 
     x0=[5.0, 0.1, 5.0, 0.1, 5.0], 
     epsilon=0.01, alpha=0.0, beta=1.0, tf=20.0),
]

results = run_chemostat_batch(configs, output_dir="results/diversity")
```

### Varying Time Horizons

```julia
configs = [
    (name="production_IC1-T1", model=1, 
     x0=[5.0, 6.0, 7.0, 5.0, 4.0], 
     epsilon=0.0, alpha=1.0, beta=0.0, tf=20.0),
    
    (name="production_IC1-T2", model=1, 
     x0=[5.0, 6.0, 7.0, 5.0, 4.0], 
     epsilon=0.0, alpha=1.0, beta=0.0, tf=30.0),
    
    (name="production_IC1-T3", model=1, 
     x0=[5.0, 6.0, 7.0, 5.0, 4.0], 
     epsilon=0.0, alpha=1.0, beta=0.0, tf=40.0),
]

results = run_chemostat_batch(configs, output_dir="results/production")
```

### Multi-Objective Optimization (Both Criteria)

```julia
# Varying α and β
configs = []
for α in [0.3, 0.5, 0.7]
    for β in [0.3, 0.5, 0.7]
        push!(configs, (
            name="both_IC1-T1-a$(α)-b$(β)",
            model=0,  # Multi-objective
            x0=[5.0, 6.0, 7.0, 5.0, 4.0],
            epsilon=0.0,
            alpha=α,
            beta=β,
            tf=20.0
        ))
    end
end

results = run_chemostat_batch(configs, output_dir="results/both")
```

## 🔬 Model Options

The `model` parameter determines the objective function:

| Model | Objective | Description |
|-------|-----------|-------------|
| 0 | `-α·P[T] + β·B[T] + γ·Σ(Δ²u)²` | Multi-objective (weighted) |
| 1 | `-P[T] + γ·Σ(Δ²u)²` | Maximize production |
| 2 | `B[T] + γ·Σ(Δ²u)²` | Minimize biodiversity |

Where:
- `P[T]`: Final production
- `B[T]`: Final biodiversity
- `γ·Σ(Δ²u)²`: Control regularization (smoothness)

## 🎛️ Parameter Guide

### Optimization Parameters
- `tf`: Time horizon (e.g., 20.0, 30.0, 40.0)
- `nsteps`: Number of discretization points (default: 150)
  - More steps → higher accuracy but slower
  - Recommended: 100-200
- `tol`: Optimization tolerance (default: 1e-8)
- `max_iter`: Maximum iterations (default: 600)

### Control Bounds
- `umin`: Minimum dilution rate (default: 0.0)
- `umax`: Maximum dilution rate (default: 5.0)

### Model Parameters
- `ε` (epsilon): Migration rate between species
- `α` (alpha): Weight for production term
- `β` (beta): Weight for biodiversity term
- `γ` (gamma): Regularization weight (controls smoothness)
  - Small γ (0.01-0.1): Less smooth, more aggressive control
  - Large γ (1.0-10.0): Very smooth, conservative control

## 🐛 Troubleshooting

### Common Issues

**1. "Restoration failed" error**
- **Cause**: Problem may be infeasible
- **Solution**: 
  - Check constraint compatibility
  - Adjust control bounds (`umin`, `umax`)
  - Try different initial conditions

**2. Chattering in control signal**
- **Cause**: Regularization parameter γ too small
- **Solution**: Increase γ (try 0.1 → 1.0)

**3. Flat/uninteresting control**
- **Cause**: Regularization parameter γ too large
- **Solution**: Decrease γ (try 1.0 → 0.01)

**4. Slow convergence**
- **Cause**: Poor initial guess
- **Solution**: 
  - Start with coarser discretization (`nsteps=50`)
  - Use solution as warm-start for finer grid

**5. Zero parameter issues**
- **Note**: Now automatically handled! 
- Zero values are replaced with 1e-10 internally

## 📈 Performance Tips

1. **Start coarse, refine gradually:**
   ```julia
   # First: coarse grid
   sol1, _ = CHEMOSTAT(nsteps=50, tol=1e-6)
   
   # Then: fine grid
   sol2, _ = CHEMOSTAT(nsteps=150, tol=1e-8)
   ```

2. **Tune regularization parameter:**
   - Start with γ = 0.1
   - Check control smoothness in output
   - Adjust based on needs

3. **Use appropriate discretization:**
   - Short time (tf < 20): `nsteps = 100`
   - Medium time (tf = 20-40): `nsteps = 150`
   - Long time (tf > 40): `nsteps = 200`

## 📝 Citation

If you use this code in your research, please cite:

```bibtex
@software{chemostat_optimal_control,
  title = {CHEMOSTAT Optimal Control Toolkit},
  author = {Your Name},
  year = {2024},
  url = {https://github.com/yourusername/chemostat-optimal-control}
}
```

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Authors

- Your Name - Initial work

## 🙏 Acknowledgments

- Anthropic's Claude for code improvements and optimization
- JuMP.jl and Ipopt for optimization framework
- Plotly for interactive visualizations

## 📞 Contact

For questions or issues, please:
- Open an issue on GitHub
- Email: your.email@example.com

---

**Happy optimizing! 🧬🔬**