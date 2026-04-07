# State Diagram – Nutrition Optimization Application

## Overview

This task contains a **UML State diagram** describing the dynamic behavior of the optimization component within the nutrition application. The diagram models how the optimizer object transitions between different internal states during its lifecycle.

The purpose of the state diagram is to represent the **behavioral logic and state transitions** of the optimization process, particularly how the system reacts to data changes, executes computations, and handles results.

The diagram focuses on the **state management of the optimization workflow**, rather than the structural design of the system.

## Main States

The optimizer is modeled using several key states:

- **Initialized** – the optimizer object is created and basic data references are assigned  
- **Ready** – the optimizer has valid input data and is prepared to perform optimization, this state together with state DIRTY is signaled by update_flag atribute  
- **Solving** – the optimization process is actively being executed  
- **Solved** – a valid solution has been successfully computed  
- **Failed** – the optimization process finished without finding a valid solution  
- **Dirty** – the input data or settings have changed, and the current solution is no longer valid  

These states describe the lifecycle of the optimizer and ensure correct handling of computations and updates.

## Initialization Phase

The optimizer begins in the **Initialized** state after object creation.

In this state:

- input data and settings are assigned  
- no optimization has been performed yet  
- no valid solution exists  

After initialization, the optimizer transitions to the **Ready** state, indicating that it is prepared to run the optimization process.

## Optimization Execution

When the optimization is triggered (e.g., via `solve()`, `get_solution()`, or `get_json_results()`), the optimizer transitions from **Ready** to **Solving**.

During the **Solving** state:

- the objective function is constructed  
- constraint matrices are generated  
- bounds are defined  
- the linear programming solver is executed  

After computation, two outcomes are possible:

- **Solved** – a feasible and valid solution is found  
- **Failed** – no feasible solution exists or an error occurs  

## Result Handling

In the **Solved** state:

- the solution is stored internally  
- repeated calls to retrieve results do not trigger recomputation  
- the optimizer remains stable until input data changes  

In the **Failed** state:

- no valid result is available  
- the system may attempt recomputation when requested  

## Data Updates and Recalculation

When input data changes (e.g., via `set_input_list()` or `set_settings()`), the optimizer transitions into the **Dirty** state.

In this state:

- the previous solution is considered outdated  
- internal matrices may be updated  
- recomputation is required before obtaining valid results  

From **Dirty**, the optimizer transitions back to **Ready**, indicating that it can perform optimization again with updated data.

## Automatic Recalculation

If result retrieval is requested while the optimizer is in the **Dirty** or **Failed** state:

- the system automatically triggers the optimization process  
- the optimizer transitions to **Solving**  

This ensures that the user always receives up-to-date results without needing to manually manage the state.

## Diagram Scope

The diagram focuses on the **behavior of the optimization component** and represents:

- internal states of the optimizer  
- transitions triggered by method calls  
- handling of successful and failed computations  
- reaction to data changes  

The diagram does not include:

- detailed mathematical formulation of the optimization problem  
- internal implementation of the solver  
- user interface interactions  
- database or persistence logic  

Its purpose is to describe the **state-driven behavior and lifecycle management of the optimization process** within the application.