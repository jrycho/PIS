



ER diagram in plantuml creation mapping mongo db.

you have these collections

users_collection = db["users"]
meals_collection = db["meal_logs"]
ingredients_collection = db["ingredients_collection"]
user_settings = db["user_settings"]
optimized_weights_collection = db["optimized_weights_collection "]
optimized_macros_collection = db["optimized_macros_collection"] 
user_ingredients_collection = db["user_ingredients_collection"]
user_meals_collection = db["user_meals_collection"]
temp_ingredients_collection = db["temp_ingredients_collection"]
user_goals_collection = db["user_goals_collection"]

Below is a clean ER diagram in PlantUML that maps your MongoDB collections into entities and relationships. Since MongoDB is non-relational, this is a logical ER representation (based on references like user_id, meal_id, etc.).

🧩 Assumptions (important)
Each collection has _id as primary key

Relationships are handled via references (user_id, meal_id, etc.)

Users are central to most collections

📊 PlantUML ER Diagram
@startuml
!define entity(x) class x << (E,#FFAAAA) >>

entity Users {
  +_id : ObjectId
  ---
  username : string
  email : string
}

entity UserSettings {
  +_id : ObjectId
  ---
  user_id : ObjectId
  settings : json
}

entity UserGoals {
  +_id : ObjectId
  ---
  user_id : ObjectId
  goals : json
}

entity Ingredients {
  +_id : ObjectId
  ---
  name : string
  macros : json
}

entity Meals {
  +_id : ObjectId
  ---
  name : string
  ingredients : array
}

entity MealLogs {
  +_id : ObjectId
  ---
  user_id : ObjectId
  meal_id : ObjectId
  timestamp : datetime
}

entity UserIngredients {
  +_id : ObjectId
  ---
  user_id : ObjectId
  ingredient_id : ObjectId
}

entity UserMeals {
  +_id : ObjectId
  ---
  user_id : ObjectId
  meal_id : ObjectId
}

entity TempIngredients {
  +_id : ObjectId
  ---
  user_id : ObjectId
  ingredient_data : json
}

entity OptimizedWeights {
  +_id : ObjectId
  ---
  user_id : ObjectId
  weights : json
}

entity OptimizedMacros {
  +_id : ObjectId
  ---
  user_id : ObjectId
  macros : json
}

' Relationships
Users ||--o{ UserSettings
Users ||--o{ UserGoals
Users ||--o{ MealLogs
Users ||--o{ UserIngredients
Users ||--o{ UserMeals
Users ||--o{ TempIngredients
Users ||--o{ OptimizedWeights
Users ||--o{ OptimizedMacros

Meals ||--o{ MealLogs
Meals ||--o{ UserMeals

Ingredients ||--o{ UserIngredients

@enduml
🧠 Notes
Users is the core entity

MealLogs acts as a transaction/history table

UserMeals and UserIngredients behave like many-to-many bridges

OptimizedWeights & OptimizedMacros are computed result storage

TempIngredients is likely session-based or draft data

⚠️ Potential Improvement Ideas
Merge optimized_weights + optimized_macros into one collection

Embed ingredients inside meals (Mongo-style) instead of referencing

Add versioning/timestamps to optimization results

If you want, I can:

Generate a more Mongo-native schema (embedding vs referencing)

Or convert this into a visual diagram image

Or tailor it exactly to your FastAPI models


{
  _id: ObjectId('69b5377d45306b0d110f30c4'),
  username: 'tester72',
  email: 'tester72@example.com',
  password: '$2b$12$0EVE6oj3.pUDtr9RGObd4OTJbfTzqgJxYJnrex2fiEAg9mc5N7XYe',
  is_admin: false,
  created_at: ISODate('2026-03-14T10:25:01.113Z'),
  share_key: '71b6f6ec38734c88bd98341344633afc',
  shared_keys: []
} users

{
  _id: ObjectId('69b5377dc98a95e3c5e58173'),
  user_id: '69b5377d45306b0d110f30c4',
  meals: {
    Breakfast: {
      excess_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ],
      slack_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ],
      target_goal: [
        Double('600'),
        Double('30'),
        Double('60'),
        Double('20')
      ],
      optimized_properties: [
        'calories',
        'protein',
        'carbs',
        'fats'
      ]
    },
    'Snack 1': {
      optimized_properties: [
        'calories',
        'protein',
        'carbs',
        'fats'
      ],
      target_goal: [
        Double('600'),
        Double('30'),
        Double('60'),
        Double('20')
      ],
      excess_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ],
      slack_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ]
    },
    Lunch: {
      optimized_properties: [
        'calories',
        'protein',
        'carbs',
        'fats'
      ],
      target_goal: [
        Double('600'),
        Double('30'),
        Double('60'),
        Double('20')
      ],
      excess_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ],
      slack_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ]
    },
    'Snack 2': {
      optimized_properties: [
        'calories',
        'protein',
        'carbs',
        'fats'
      ],
      target_goal: [
        Double('600'),
        Double('30'),
        Double('60'),
        Double('20')
      ],
      excess_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ],
      slack_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ]
    },
    Dinner: {
      optimized_properties: [
        'calories',
        'protein',
        'carbs',
        'fats'
      ],
      target_goal: [
        Double('600'),
        Double('30'),
        Double('60'),
        Double('20')
      ],
      excess_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ],
      slack_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ]
    },
    'Snack 3': {
      optimized_properties: [
        'calories',
        'protein',
        'carbs',
        'fats'
      ],
      target_goal: [
        Double('600'),
        Double('30'),
        Double('60'),
        Double('20')
      ],
      excess_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ],
      slack_weights: [
        Double('1'),
        Double('1'),
        Double('1'),
        Double('1')
      ]
    }
  },
  updated_at: ISODate('2026-03-14T10:48:11.225Z')
} user settings

{
  _id: ObjectId('69b53475516371fd9d6fb4a5'),
  code: 'custom-693199fac4aaf933b7168f5e-61e9cbec',
  product_name: 'Ovesné vločky',
  nutriments: {
    energy_kcal_100g: Double('386'),
    carbohydrates_100g: Double('66'),
    proteins_100g: Double('11'),
    fat_100g: Double('5')
  },
  categories_tags: [
    'custom'
  ],
  priority_user: NumberInt('1'),
  user_id: '693199fac4aaf933b7168f5e',
  share_key: '97ce3ab5691f4a45aadf9a620f968164'
} user ingredients collection, for nutriments oNly say dic, not everything in it 

{
  _id: ObjectId('69b53794c98a95e3c5e58188'),
  user_id: '69b5377d45306b0d110f30c4',
  target_macros: {
    calories: NumberInt('2304'),
    protein: Double('153'),
    fat: Double('76.5'),
    carbs: Double('251'),
    sat_fat: Double('17.9'),
    fiber: Double('32.3'),
    sodium: NumberInt('2300'),
    salt: Double('5.8'),
    cholesterol: NumberInt('300')
  }
} user goal collection

{
  _id: ObjectId('69b538ce45306b0d110f30c8'),
  created_at: ISODate('2026-03-14T10:30:38.013Z'),
  ingredients: [],
  user_id: '69b5377d45306b0d110f30c4'
} temp ings col

{
  _id: ObjectId('69a5ac7786a5671f0c99c334'),
  meal_id: '2b86b3a8-ba4d-484f-84f2-b53ff4442b1e',
  user_id: '693199fac4aaf933b7168f5e',
  results: [
    {
      barcode: 'custom-693199fac4aaf933b7168f5e-215539fb',
      name: 'Test ingredient4',
      grams: Double('120.00000000000001')
    }
  ]
} optimized weights col

{
  _id: ObjectId('69a5c1b986a5671f0c99ceae'),
  meal_id: '232a8dd8-46a0-4157-90b2-a316a4540fd2',
  user_id: '693199fac4aaf933b7168f5e',
  results: {
    calories: NumberInt('300'),
    protein: NumberInt('12'),
    carbs: NumberInt('36'),
    fats: NumberInt('6')
  }
} opt macros col

{
  _id: ObjectId('69b817c447386ef06b07eac8'),
  meal_id: '2bbdc4c9-4831-4aca-b946-45ead55ea417',
  date: '2026-03-16',
  ingredients: [
    {
      barcode: 'custom-693199fac4aaf933b7168f5e-f0b62399',
      priority: NumberInt('1'),
      piece_weight: Double('0'),
      set_amount: Double('30')
    },
    {
      barcode: 'custom-693199fac4aaf933b7168f5e-c464ab66',
      priority: NumberInt('1'),
      piece_weight: Double('0'),
      set_amount: Double('30')
    }
  ],
  type_of_meal: 'Breakfast',
  user_id: '693199fac4aaf933b7168f5e'
} meal logs

{
  _id: '8005110517006',
  barcode: '8005110517006',
  categories_tags: [
    'plant-based-foods-and-beverages',
    'plant-based-foods',
    'fruits-and-vegetables-based-foods',
    'condiments',
    'vegetables-based-foods',
    'cooking-helpers',
    'sauces',
    'tomatoes-and-their-products',
    'meal-sauces',
    'pasta-sauces',
    'tomato-sauces',
    'tomato-sauces-with-basil'
  ],
  name: 'Pastasaus m/Basilikum',
  nova_group: NumberInt('4'),
  nutrients: {
    energy_kcal_100g: Double('56'),
    carbohydrates_100g: Double('5.3'),
    carbohydrates_serving: Double('3.98'),
    sugars_100g: Double('4.5'),
    sugars_serving: Double('3.38'),
    proteins_100g: Double('1.6'),
    proteins_serving: Double('1.2'),
    fat_100g: Double('2.8'),
    fat_serving: Double('2.1'),
    saturated_fat_100g: Double('0.5'),
    saturated_fat_serving: Double('0.375'),
    cholesterol_100g: Double('0.0007'),
    cholesterol_serving: Double('0.000525'),
    potassium_100g: Double('0.383'),
    potassium_serving: Double('0.287'),
    calcium_100g: Double('0.173'),
    calcium_serving: Double('0.13'),
    iron_100g: Double('0.0038'),
    iron_serving: Double('0.00285'),
    sodium_100g: Double('0.243'),
    sodium_serving: Double('0.182'),
    salt_100g: Double('0.61'),
    salt_serving: Double('0.457')
  },
  pnns_groups_1: 'Fat and sauces',
  pnns_groups_2: 'Dressings and sauces',
  priority_auto: NumberInt('0')
} ings coll same as for user ings, dont map nutrients

Here is a PlantUML ER diagram based on the actual collection samples you provided.

I kept nutriments / nutrients as just dict, as requested.

@startuml
hide methods
hide circle
skinparam linetype ortho

entity "users" as users {
  * _id : ObjectId <<PK>>
  --
  username : string
  email : string
  password : string
  is_admin : bool
  created_at : datetime
  share_key : string
  shared_keys : array
}

entity "user_settings" as user_settings {
  * _id : ObjectId <<PK>>
  --
  user_id : ObjectId <<FK>>
  meals : dict
  updated_at : datetime
}

entity "user_goals_collection" as user_goals {
  * _id : ObjectId <<PK>>
  --
  user_id : ObjectId <<FK>>
  target_macros : dict
}

entity "user_ingredients_collection" as user_ingredients {
  * _id : ObjectId <<PK>>
  --
  code : string
  product_name : string
  nutriments : dict
  categories_tags : array
  priority_user : int
  user_id : ObjectId <<FK>>
  share_key : string
}

entity "ingredients_collection" as ingredients {
  * _id : string <<PK>>
  --
  barcode : string
  categories_tags : array
  name : string
  nova_group : int
  nutrients : dict
  pnns_groups_1 : string
  pnns_groups_2 : string
  priority_auto : int
}

entity "temp_ingredients_collection" as temp_ingredients {
  * _id : ObjectId <<PK>>
  --
  created_at : datetime
  ingredients : array
  user_id : ObjectId <<FK>>
}

entity "optimized_weights_collection" as optimized_weights {
  * _id : ObjectId <<PK>>
  --
  meal_id : string
  user_id : ObjectId <<FK>>
  results : array
}

entity "optimized_macros_collection" as optimized_macros {
  * _id : ObjectId <<PK>>
  --
  meal_id : string
  user_id : ObjectId <<FK>>
  results : dict
}

entity "meal_logs" as meal_logs {
  * _id : ObjectId <<PK>>
  --
  meal_id : string
  date : date
  ingredients : array
  type_of_meal : string
  user_id : ObjectId <<FK>>
}

users ||--|| user_settings : has
users ||--|| user_goals : has
users ||--o{ user_ingredients : owns
users ||--|| temp_ingredients : has
users ||--o{ optimized_weights : has
users ||--o{ optimized_macros : has
users ||--o{ meal_logs : logs

user_ingredients ..> users : user_id
user_settings ..> users : user_id
user_goals ..> users : user_id
temp_ingredients ..> users : user_id
optimized_weights ..> users : user_id
optimized_macros ..> users : user_id
meal_logs ..> users : user_id

meal_logs ..> optimized_weights : meal_id
meal_logs ..> optimized_macros : meal_id
meal_logs ..> user_ingredients : ingredients[].barcode
meal_logs ..> ingredients : ingredients[].barcode
optimized_weights ..> meal_logs : meal_id
optimized_macros ..> meal_logs : meal_id

@enduml
A few modeling notes:

user_settings.meals is best represented as a dict, because meal names like Breakfast, Snack 1, Lunch are dynamic nested keys.

meal_logs.ingredients is an embedded array, so I mapped its barcode relation as a dotted reference to ingredient collections.

optimized_weights and optimized_macros are linked to meals through meal_id, but meal_id is not the Mongo _id of meal_logs; it is an application-level identifier.

ingredients_collection uses _id = barcode in your sample, so I modeled _id as string.

There is one inconsistency in your data:

some collections use user IDs like 69b5377d45306b0d110f30c4

others use 693199fac4aaf933b7168f5e

So in the real database, some documents may belong to different users or test datasets.

If you want, I can also make you a cleaner crow’s foot version with associative entities for embedded arrays, which usually looks better for thesis/project documentation.


generate README for this inspired by # Class Diagram – Nutrition Optimization Application

## Overview

This task contains a **UML Class diagram** describing the structural design of an application for optimizing food portions based on nutritional goals (macronutrients). The diagram models the main domain objects of the system, their attributes, methods, and relationships.

The purpose of the class diagram is to represent the **core data model and object interactions** required to support meal creation, ingredient management, optimization configuration, and result generation.

The diagram focuses on the logical structure of the system and the responsibilities of individual classes.

## Main Classes

The system is composed of several key domain classes:

* **User** – represents a registered user of the application who creates meals, configures optimization settings, and tracks nutrition progress.
* **Admin** – a privileged system actor responsible for managing user accounts.
* **Settings** – stores configuration parameters used by the optimization process.
* **MealLog** – represents a meal record for a specific date that contains selected ingredients.
* **MealIngredient** – represents a specific ingredient used in a meal together with its quantity and quantity rules.
* **Ingredient** – represents a base ingredient with its nutritional values.
* **OptimizationResult** – represents the output of the optimization process.
* **NutritionGoal (Tracker)** – tracks the user's nutritional progress compared to target values.

## User and Administration

The **User** class represents a standard application user. Users can:

* create meal logs
* configure optimization settings
* track nutritional progress
* add ingredients to meals
* initiate the optimization process

The **Admin** class represents a system administrator. The administrator can:

* log into the administration interface
* search for user accounts
* modify user information
* maintain system data

## Optimization Configuration

The **Settings** class represents the configuration object used by the optimization algorithm. It contains parameters defining how the optimization should behave.

The settings object includes:

* a list of **optimized nutrients**
* a list of **goal values**
* a list of **excess penalties**
* a list of **slack penalties**

These values define the optimization objective and constraints.

During optimization, the **MealLog** and **Settings** objects together produce an **OptimizationResult**.

## Meal Structure

Meals are represented using two classes:

* **MealLog**
* **MealIngredient**

A **MealLog** represents a single meal entry for a specific date.

A **MealIngredient** represents an ingredient used in the meal together with information such as:

* quantity
* quantity type (for example grams or pieces)
* quantity rule (piece-based or exact amount)

Each MealIngredient references a base Ingredient object containing nutritional information such as calories, proteins, fats, and carbohydrates.

## Optimization Process

The optimization process uses two main inputs:

* the **MealLog** containing ingredients and quantities
* the **Settings** object defining optimization parameters

Based on these inputs, the system generates an **OptimizationResult**, which contains:

* the status of the optimization
* the objective function value
* additional messages or information about the result

The optimized ingredient quantities are then presented to the user.

## Nutrition Tracking

The **NutritionGoal** class functions as a **tracker** for monitoring nutritional intake relative to target values.

It stores both:

* current nutrient intake values
* target nutrient values

Using these values, the system can calculate progress toward the user's nutritional goals and provide feedback about goal completion.

## Diagram Scope

The diagram focuses on the **core domain model and system structure**. It represents:

* domain entities
* class attributes
* class methods
* relationships between objects

The diagram does not include:

* frontend user interface components
* database schema details
* low-level implementation logic

Its purpose is to describe the **object-oriented design of the system and the relationships between its core components**.

Here is a README adapted to your MongoDB ER diagram, keeping the same style, structure, and level of professionalism as your class diagram description.

ER Diagram – Nutrition Optimization Application (MongoDB)
Overview
This task contains an Entity-Relationship (ER) diagram representing the data model of a nutrition optimization application built on MongoDB.

The diagram describes how application data is structured across collections, how documents relate to each other through references, and how core features such as meal tracking, ingredient management, and optimization results are persisted.

The purpose of this diagram is to provide a logical overview of the database design, focusing on relationships between collections and their responsibilities within the system.

Although MongoDB is a NoSQL database, the diagram models relationships conceptually to improve understanding of data flow and system structure.

Main Collections
The system is composed of several key collections:

users – stores user accounts and authentication-related data.

user_settings – contains optimization configuration for each user, including per-meal goals and weights.

user_goals_collection – stores overall nutritional targets for the user.

ingredients_collection – contains global ingredient data (e.g. from OpenFoodFacts or similar sources).

user_ingredients_collection – stores custom or user-specific ingredients.

meal_logs – represents meals created by users with selected ingredients and quantities.

optimized_weights_collection – stores optimization outputs in terms of ingredient quantities.

optimized_macros_collection – stores computed nutritional values after optimization.

temp_ingredients_collection – temporary storage for ingredients during editing or preprocessing.

User and Data Ownership
The users collection is the central entity of the system.

Each user can:

define personal optimization settings

set nutritional goals

create and log meals

manage their own ingredients

run optimization and store results

Most collections reference users via user_id, forming one-to-many relationships:

one user → multiple meal logs

one user → multiple optimization results

one user → multiple custom ingredients

This structure ensures data isolation per user while allowing scalable storage.

Optimization Configuration
The user_settings collection stores configuration data used during optimization.

Unlike a traditional relational model, settings are stored as a nested dictionary (meals), where each meal type (e.g. Breakfast, Lunch) contains:

target_goal – desired nutrient values

optimized_properties – list of tracked nutrients

excess_weights – penalties for exceeding targets

slack_weights – penalties for not meeting targets

This flexible structure allows dynamic meal definitions without schema changes.

Nutritional Goals
The user_goals_collection represents global daily nutritional targets.

It contains:

calories

protein

fats

carbohydrates

additional nutrients (fiber, sodium, etc.)

This collection acts as a reference baseline for evaluating overall nutrition intake and optimization success.

Ingredient Management
Ingredients are divided into two collections:

Global Ingredients
The ingredients_collection contains standardized ingredient data with:

barcode (used as identifier)

categories

nutritional values (dict)

classification metadata

User Ingredients
The user_ingredients_collection allows users to define custom ingredients.

These include:

custom product names

user-specific priority

nutritional values (dict)

sharing capabilities via share_key

This separation allows combining external datasets with user-defined data.

Meal Structure
Meals are represented in the meal_logs collection.

Each meal log contains:

meal_id – application-level identifier

date – when the meal was consumed

type_of_meal – e.g. Breakfast, Lunch

ingredients – embedded array of ingredient entries

Each ingredient entry includes:

barcode reference

quantity (set_amount)

optional piece-based configuration

This structure follows MongoDB principles by embedding meal ingredients directly inside the meal document.

Optimization Results
Optimization outputs are stored in two collections:

Optimized Weights
The optimized_weights_collection contains:

ingredient-level results

calculated quantities (grams)

Each result entry includes:

ingredient identifier (barcode)

ingredient name

optimized amount

Optimized Macros
The optimized_macros_collection contains:

aggregated nutritional values for the meal

calories, protein, carbs, fats, etc.

Both collections are linked to meals via meal_id.

Temporary Data Handling
The temp_ingredients_collection is used for transient data during:

ingredient selection

preprocessing steps

intermediate UI states

This collection prevents incomplete or temporary data from affecting permanent meal records.

Relationships and Data Flow
Key relationships in the system include:

User → MealLogs (one-to-many)

User → Settings / Goals (one-to-one)

User → Ingredients (one-to-many)

MealLogs → Ingredients (via embedded references)

MealLogs → Optimization Results (via meal_id)

The typical data flow is:

User defines settings and goals

User creates a meal with ingredients

Optimization process runs using:

meal data

user settings

Results are stored in:

optimized weights

optimized macros

Diagram Scope
The diagram focuses on the database structure and relationships between collections.

It represents:

MongoDB collections as entities

document fields and key attributes

logical relationships via references

embedded document structures

The diagram does not include:

frontend components

API endpoints (FastAPI layer)

optimization algorithm implementation details

Its purpose is to describe the data architecture and persistence layer of the system.

If you want, I can also:

align this README with your FastAPI models (Pydantic schemas)

or generate a combined Class + ER documentation for your assignment


do it as md file

Here is your README as a ready-to-use .md file:

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
If you want, I can also generate:

a matching Class Diagram README (so both docs align perfectly)

or a short version for submission (1–2 pages)



