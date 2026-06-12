# HRAP: Adaptive Efficiency Optimization in SDLC

This repository contains notebook-based implementations for the paper:

**Adaptive Efficiency Optimization in SDLC: An MILP Approach for Balanced and Cost-Effective Resource Allocation**  
Lokendra Kumar, Neelesh S. Upadhye, and Kannan Piedy, 2025.

The project addresses the **Human Resource Allocation Problem (HRAP)** in the Software Development Life Cycle (SDLC). It formulates task assignment as a **Mixed Integer Linear Programming (MILP)** problem so that project tasks are assigned to qualified employees while balancing workload, accounting for employee efficiency, and optionally minimizing cost.

## Key ideas

- Assign each task to exactly one qualified employee.
- Balance employee workload using efficiency-adjusted task durations.
- Identify tasks that cannot be assigned because no employee has the required skill.
- Support both open-source and commercial MILP solvers.
- Extend the base workload-balancing model with a cost-aware objective using task duration, employee efficiency, performance rating, and task complexity.
- Tune cost-objective weights using Optuna in the cost-aware notebook.

## Repository contents

```text
HRAP/
├── Data_generation.ipynb                    # Synthetic/sample data generation
├── HRAP_using_pulp_solver.ipynb             # MILP implementation using PuLP/CBC
├── HRAP_with_Gurobi_Solver.ipynb            # MILP implementation using Gurobi
├── HRAP_with_cost.ipynb                     # Cost-aware HRAP optimization with Optuna
├── Employee_table.xlsx                      # Employee/task workbook for base HRAP model
├── employee_task_data.xlsx                  # Employee/task workbook for cost-aware model
├── resource_allocation_master_data.xlsx     # Master resource allocation data
├── generated_employee_task_*.xlsx           # Generated benchmark datasets
├── assigned_tasks.csv                       # Sample assignment output
├── pulp_assigned_tasks.csv                  # PuLP assignment output
├── pulp_unassigned_tasks.csv                # PuLP unassigned-task output
├── Gurobi_assigned_tasks*.csv               # Gurobi assignment outputs
├── Gurobi_unassigned_tasks*.csv             # Gurobi unassigned-task outputs
└── optimized_assignment_with_cost.csv       # Cost-aware optimization output
```

## Method overview

For each employee `i` and task `t`, the decision variable is:

```text
x[i,t] = 1 if task t is assigned to employee i, otherwise 0
```

The efficiency-adjusted workload for employee `i` is:

```text
W[i] = sum_t x[i,t] * duration[t] / efficiency[i, required_skill[t]]
```

The base MILP minimizes the deviation between each employee's workload and the target workload:

```text
minimize D_plus + D_minus
```

subject to:

```text
Each assignable task is assigned exactly once.
Only employees with the required skill can receive a task.
Employee workload must stay within target workload +/- deviation.
x[i,t] is binary.
D_plus and D_minus are non-negative.
```

The cost-aware version adds an assignment cost term. The paper defines the full cost idea as a weighted combination of efficiency-adjusted duration, skill mismatch, and complexity-to-performance ratio. The current cost notebook implements a simplified cost expression based on:

```text
cost = alpha * duration / efficiency
     + beta  * complexity
     + gamma * complexity / performance_rating
```

The final cost-aware objective is:

```text
minimize lambda * workload_deviation + (1 - lambda) * total_assignment_cost
```

## Data format

### Base HRAP workbook: `Employee_table.xlsx`

Expected sheets and columns:

**Sheet: `person`**

| Column | Description |
|---|---|
| `name` | Employee name or ID |
| `skills` | Skill possessed by the employee |
| `efficiency` | Employee efficiency score for the skill |

**Sheet: `tasks`**

| Column | Description |
|---|---|
| `task_name` | Task identifier |
| `required_skills` | Skill required to perform the task |
| `duration` | Estimated task duration in hours |

### Cost-aware workbook: `employee_task_data.xlsx`

Expected sheets and columns:

**Sheet: `Employee_table`**

| Column | Description |
|---|---|
| `name` | Employee name or ID |
| `skills` | Skill possessed by the employee |
| `efficiency` | Employee efficiency score |
| `performance_rating` | Employee performance rating |

**Sheet: `Task_table`**

| Column | Description |
|---|---|
| `task_name` | Task identifier |
| `required_skills` | Skill required by the task |
| `duration` | Estimated task duration in hours |
| `complexity` | Numeric task-complexity score |

## Installation

Clone the repository:

```bash
git clone https://github.com/Lokendra5298/HRAP.git
cd HRAP
```

Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate      # Linux/macOS
# .venv\Scripts\activate       # Windows
```

Install the recommended Python packages:

```bash
pip install pandas numpy openpyxl pulp optuna jupyter
```

For the Gurobi notebook, install `gurobipy` separately and ensure that a valid Gurobi license is available:

```bash
pip install gurobipy
```

## Running the notebooks

Start Jupyter:

```bash
jupyter notebook
```

Then run one of the following notebooks:

| Notebook | Purpose | Main input | Main output |
|---|---|---|---|
| `Data_generation.ipynb` | Generate sample employee-task data | Configured inside notebook | Generated Excel files |
| `HRAP_using_pulp_solver.ipynb` | Solve the base HRAP model using PuLP/CBC | `Employee_table.xlsx` | `pulp_assigned_tasks.csv`, `pulp_unassigned_tasks.csv` |
| `HRAP_with_Gurobi_Solver.ipynb` | Solve the base HRAP model using Gurobi | `Employee_table.xlsx` | `Gurobi_assigned_tasks*.csv`, `Gurobi_unassigned_tasks*.csv` |
| `HRAP_with_cost.ipynb` | Solve cost-aware HRAP and tune weights | `employee_task_data.xlsx` | `optimized_assignment_with_cost.csv` |

### Google Colab note

Some notebooks use paths such as:

```python
pd.read_excel('/content/Employee_table.xlsx')
```

When running locally, replace `/content/...` with the local file path, for example:

```python
pd.read_excel('Employee_table.xlsx')
```

## Outputs

The solvers produce assignment tables that typically include:

- assigned employee
- assigned tasks
- required skills
- total assigned hours
- day-by-day utilization
- employee contribution percentage

The workflow also records unassigned tasks when no employee has the required skill. In the sample notebooks, examples include tasks requiring skills such as `nodejs` or `go` when those skills are absent from the employee pool.

## Reported results from the paper

The paper reports that the MILP-based assignment improves workload fairness compared with conventional/manual assignment strategies. For a sample of 20 employees:

| Metric | Manager/random assignment | MILP optimized assignment |
|---|---:|---:|
| Workload variance | 172.48 | 26.95 |
| Gini coefficient | 0.340 | 0.093 |
| Jain's fairness index | 0.768 | 0.964 |

The scalability study reports average solver times from approximately `0.16 s` for 20 employees and 80 tasks to approximately `42.96 s` for 250 employees and 600 tasks using the PuLP/CBC setup.

## Citation

If you use this repository or build on this work, please cite:

```bibtex
@misc{kumar2025adaptiveefficiencyoptimizationsdlc,
  title={Adaptive Efficiency Optimization in SDLC: An MILP Approach for Balanced and Cost-Effective Resource Allocation},
  author={Lokendra Kumar and Neelesh S. Upadhye and Kannan Piedy},
  year={2025},
  eprint={2512.13577},
  archivePrefix={arXiv},
  primaryClass={math.OC},
  url={https://arxiv.org/abs/2512.13577}
}
```

## Acknowledgment

The paper acknowledges support from Prodapt Solutions Private Limited and the Department of Mathematics, Indian Institute of Technology Madras.

## License

MIT
