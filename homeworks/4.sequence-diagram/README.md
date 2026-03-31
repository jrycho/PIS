# Sequence Diagram – Nutrition Optimization Application (Workflow)

## Overview

This task contains a **Sequence Diagram** representing the runtime behavior of a nutrition optimization application.

The diagram illustrates how different system components interact over time, including:

* **User**
* **Frontend (FE)**
* **Backend (BE)**
* **Database (DB)**

The purpose of this diagram is to provide a **clear overview of system workflows**, focusing on request flow, backend processing, and data persistence.

Unlike the ER diagram, which describes static structure, this diagram captures **dynamic behavior and execution order**.

---

## Main Components

The system consists of four primary participants:

* **User** – initiates actions such as creating meals or running optimization  
* **Frontend (FE)** – handles UI interactions and communicates with the backend  
* **Backend (BE)** – processes requests, performs business logic, and runs optimization  
* **Database (DB)** – stores persistent data including meals, settings, and optimization results  

---

## Core Workflows

The sequence diagram includes several key workflows representing the main application functionality.

---

### 1. User Settings

The user defines their nutritional preferences and optimization configuration.

**Flow:**
1. User inputs settings in the frontend  
2. Frontend sends request to backend  
3. Backend stores settings in database  
4. Confirmation is returned to the user  

**Endpoint:**
POST /settings/save_settings

---

### 2. Meal Log Creation

The user creates a new meal entry.

**Flow:**
1. User creates a meal log  
2. Frontend sends meal data to backend  
3. Backend stores meal in database  
4. Meal log is returned and displayed  

**Endpoint:**
POST /logs/log_meal
---

### 3. Adding Ingredients

The user adds ingredients to a meal log.

**Flow:**
1. User selects an ingredient  
2. Frontend sends ingredient data (barcode)  
3. Backend updates meal log in database  
4. Updated meal is returned  

**Endpoint:**
POST /logs/add_ingredient/{barcode}
---

### 4. Meal Optimization

This is the core functionality of the system.

The backend performs optimization based on:
* meal ingredients  
* user settings  

**Flow:**
1. User triggers optimization  
2. Frontend calls backend endpoint  
3. Backend prepares input data from meal log  
4. Backend loads user settings  
5. Backend runs optimization algorithm  
   - primary: **Linear Programming (linprog)**  
   - fallback: **Grey Wolf Optimizer (GWO)**  
6. Backend computes:
   - optimized ingredient weights  
   - total nutritional values  
7. Results are saved to database  
8. Results are returned to frontend  
9. Frontend renders optimized meal  

**Endpoint:**
GET /optim/optimize/{meal_id}


---

## Optimization Logic

The backend uses a two-step optimization strategy:

1. **Primary solver**
   - Linear programming (`linprog`)  
   - Fast and deterministic  

2. **Fallback solver**
   - Grey Wolf Optimizer (GWO)  
   - Used when primary solver fails  

This ensures robustness and increases the chance of finding a valid solution.

---

## Data Persistence

During workflows, the backend interacts with the database to:

* store user settings  
* store meal logs and ingredients  
* store optimization results:
  - optimized weights  
  - optimized macros  

All operations follow a **request → process → store → return** pattern.

---

## Relationships and Flow Characteristics

Key characteristics of the system flow:

* **User → Frontend → Backend → Database → Backend → Frontend → User**  
* Backend acts as the **central orchestrator**  
* Database is used only for **persistence**, not logic  
* Optimization is fully handled inside the backend  

---

## Typical End-to-End Scenario

1. User sets nutritional preferences  
2. User creates a meal  
3. User adds ingredients  
4. User runs optimization  
5. Backend processes and computes optimal solution  
6. Results are stored and returned  
7. User sees optimized meal composition  

---

## Diagram Scope

The diagram focuses on **runtime interactions and system behavior**.

It represents:

* communication between system components  
* API request flow  
* backend processing steps  
* database interactions  

The diagram does not include:

* internal frontend implementation  
* database schema details (covered by ER diagram)  
* low-level optimization algorithm details  

Its purpose is to describe the **dynamic execution flow of the system**.