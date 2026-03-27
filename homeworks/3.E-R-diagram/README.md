# ER Diagram – Nutrition Optimization Application (MongoDB)

## Overview

This task contains an **Entity-Relationship (ER) diagram** representing the data model of a nutrition optimization application built on **MongoDB**.

The diagram describes how application data is structured across collections, how documents relate to each other through references, and how core features such as meal tracking, ingredient management, and optimization results are persisted.

The purpose of this diagram is to provide a **logical overview of the database design**, focusing on relationships between collections and their responsibilities within the system.

Although MongoDB is a **NoSQL database**, the diagram models relationships conceptually to improve understanding of data flow and system structure.

---

## Main Collections

The system is composed of several key collections:

* **users** – stores user accounts and authentication-related data.
* **user_settings** – contains optimization configuration for each user, including per-meal goals and weights.
* **user_goals_collection** – stores overall nutritional targets for the user.
* **ingredients_collection** – contains global ingredient data (e.g. from external datasets).
* **user_ingredients_collection** – stores custom or user-specific ingredients.
* **meal_logs** – represents meals created by users with selected ingredients and quantities.
* **optimized_weights_collection** – stores optimization outputs in terms of ingredient quantities.
* **optimized_macros_collection** – stores computed nutritional values after optimization.
* **temp_ingredients_collection** – temporary storage for ingredients during editing or preprocessing.

---

## User and Data Ownership

The **users** collection is the central entity of the system.

Each user can:

* define personal optimization settings
* set nutritional goals
* create and log meals
* manage their own ingredients
* run optimization and store results

Most collections reference users via `user_id`, forming **one-to-many relationships**:

* one user → multiple meal logs  
* one user → multiple optimization results  
* one user → multiple custom ingredients  

This structure ensures **data isolation per user** while allowing scalable storage.

---

## Optimization Configuration

The **user_settings** collection stores configuration data used during optimization.

Settings are stored as a **nested dictionary (`meals`)**, where each meal type (e.g. Breakfast, Lunch) contains:

* `target_goal` – desired nutrient values
* `optimized_properties` – list of tracked nutrients
* `excess_weights` – penalties for exceeding targets
* `slack_weights` – penalties for not meeting targets

This flexible structure allows dynamic meal definitions without schema changes.

---

## Nutritional Goals

The **user_goals_collection** represents global daily nutritional targets.

It contains:

* calories
* protein
* fats
* carbohydrates
* additional nutrients (fiber, sodium, etc.)

This collection acts as a **reference baseline** for evaluating overall nutrition intake and optimization success.

---

## Ingredient Management

Ingredients are divided into two collections:

### Global Ingredients

The **ingredients_collection** contains standardized ingredient data with:

* barcode (used as identifier)
* categories
* nutritional values (`dict`)
* classification metadata

### User Ingredients

The **user_ingredients_collection** allows users to define custom ingredients.

These include:

* custom product names
* user-specific priority
* nutritional values (`dict`)
* sharing capabilities via `share_key`

This separation allows combining **external datasets with user-defined data**.

---

## Meal Structure

Meals are represented in the **meal_logs** collection.

Each meal log contains:

* `meal_id` – application-level identifier
* `date` – when the meal was consumed
* `type_of_meal` – e.g. Breakfast, Lunch
* `ingredients` – embedded array of ingredient entries

Each ingredient entry includes:

* barcode reference
* quantity (`set_amount`)
* optional piece-based configuration

This structure follows MongoDB principles by **embedding meal ingredients directly inside the meal document**.

---

## Optimization Results

Optimization outputs are stored in two collections:

### Optimized Weights

The **optimized_weights_collection** contains:

* ingredient-level results
* calculated quantities (grams)

Each result entry includes:

* ingredient identifier (barcode)
* ingredient name
* optimized amount

### Optimized Macros

The **optimized_macros_collection** contains:

* aggregated nutritional values for the meal
* calories, protein, carbs, fats, etc.

Both collections are linked to meals via `meal_id`.

---

## Temporary Data Handling

The **temp_ingredients_collection** is used for transient data during:

* ingredient selection
* preprocessing steps
* intermediate UI states

This collection prevents incomplete or temporary data from affecting permanent meal records.

---

## Relationships and Data Flow

Key relationships in the system include:

* **User → MealLogs** (one-to-many)
* **User → Settings / Goals** (one-to-one)
* **User → Ingredients** (one-to-many)
* **MealLogs → Ingredients** (via embedded references)
* **MealLogs → Optimization Results** (via `meal_id`)

### Typical Workflow

1. User defines **settings and goals**
2. User creates a **meal with ingredients**
3. Optimization process runs using:
   * meal data
   * user settings
4. Results are stored in:
   * optimized weights
   * optimized macros

---

## Diagram Scope

The diagram focuses on the **database structure and relationships between collections**.

It represents:

* MongoDB collections as entities
* document fields and key attributes
* logical relationships via references
* embedded document structures

The diagram does not include:

* frontend components
* API endpoints (FastAPI layer)
* optimization algorithm implementation details

Its purpose is to describe the **data architecture and persistence layer** of the system.