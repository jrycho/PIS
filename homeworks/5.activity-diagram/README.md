# Linear Programming Optimizer – Nutrition Optimization Application

## Overview

This module implements a **linear programming (LP) optimizer** for solving nutrition-based meal composition problems using SciPy (`scipy.optimize.linprog`).

The optimizer determines optimal ingredient quantities such that selected nutritional properties (e.g., calories, protein, carbohydrates) match predefined target values while minimizing deviations.

The system supports:

- **user-defined fixed ingredient quantities**
- **soft constraint handling via slack and excess variables**
- **approximate handling of discrete (portion-based) ingredients**

The goal is to produce a nutritionally balanced solution while respecting both mathematical constraints and practical usability.

---

## Core Concept

The optimization problem is formulated as a **linear program with equality constraints**:

- Ingredient quantities are decision variables  
- Nutritional targets are enforced through equality constraints  
- Deviations from targets are captured using additional variables  

The optimizer minimizes a weighted sum of deviations:

- **slack variables (`d+`)** → underachieving targets  
- **excess variables (`d-`)** → exceeding targets  

---

## Optimization Model

### Decision Variables

The optimization vector has the following structure:
[x₁, x₂, ..., xₙ, d₁⁺, ..., d_k⁺, d₁⁻, ..., d_k⁻]
Where:

- `xᵢ` = ingredient quantities  
- `d⁺` = slack variables (deficit)  
- `d⁻` = excess variables (surplus)  

---

### Objective Function

The optimizer minimizes deviation from nutritional targets:


minimize: Σ (w_slack * d⁺ + w_excess * d⁻)


- `w_slack` = penalty for underachieving targets  
- `w_excess` = penalty for exceeding targets  

---

### Constraints

The system enforces:


A_food · x + d⁺ - d⁻ = target_goal


Where:

- `A_food` = nutrient matrix (properties × ingredients)  
- `x` = ingredient vector  
- `target_goal` = desired nutrient values  

---

## Constraint Handling

### Fixed Ingredient Quantities

User-defined ingredient quantities are enforced directly through bounds:


xᵢ = fixed_value


This removes the variable from the optimization search space.

---

### Standard Bounds

For each ingredient:

- Priority ingredient → `(0.1, ∞)`
- Regular ingredient → `(0.1, 2)`
- Fixed ingredient → `(value, value)`

Deviation variables always have bounds:


(0, ∞)


---

## Discrete (Indivisible) Ingredient Handling

Some ingredients must be used in discrete portions (e.g., whole eggs, packaged items).

Since `linprog` does not support integer constraints, a **two-stage approximation strategy** is used.

---

### Step 1 – Continuous Optimization

The problem is first solved without discrete constraints:

- Produces continuous solution `x*`

---

### Step 2 – Discrete Approximation

For indivisible ingredients:

1. Convert values to portion counts:

count = x* / portion_size


2. Apply minimum threshold:

if 0 < count < 0.5 → count = 0.5


3. Round to nearest integer:

count = round(count)


4. Convert back to weights:

weight = count × portion_size


---

### Step 3 – Bound Fixing

Rounded values are enforced:


xᵢ = rounded_weight


Only nonzero rounded values are fixed; others remain flexible.

---

### Step 4 – Final Optimization

A second optimization is performed:

- Fixed discrete ingredients remain constant  
- Remaining variables are optimized  

This ensures:

- feasibility with discrete constraints  
- minimal deviation from targets  

---

## Nutrient Matrix Construction

The nutrient matrix is constructed dynamically:


A_food =
[
[calories₁, calories₂, ...],
[protein₁, protein₂, ...],
...
]


It is expanded internally as:


A_eq = [A_food | I | -I]


Where:

- `I` = identity matrix for slack variables  
- `-I` = identity matrix for excess variables  

---

## Workflow Summary

1. Load settings and ingredient data  
2. Build objective vector and constraint matrix  
3. Apply bounds (including fixed ingredients)  
4. Check for indivisible ingredients  

**If none:**
- Run single optimization  

**If present:**
- Run initial optimization  
- Approximate discrete values  
- Fix rounded values  
- Run second optimization  

5. Store final solution  

---

## Key Characteristics

- **Deterministic optimization** using linear programming  
- **Soft constraint handling** via deviation variables  
- **Efficient and fast** (HiGHS solver)  
- **Approximate discrete handling** without integer programming  
- **Flexible structure** for dynamic nutrient sets  

---

## Limitations

- Discrete constraints are approximated, not exact (no MILP)  
- Rounding may introduce slight deviations from optimality  
- Solution quality depends on penalty weight tuning  

---

## Summary

The linear programming optimizer provides a **robust and efficient method** for nutritional optimization by combining:

- strict equality-based modeling  
- flexible deviation penalties  
- practical handling of real-world constraints  

It serves as a reliable baseline solver within the nutrition optimization system.