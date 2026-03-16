# Class Diagram – Nutrition Optimization Application

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

Each `MealIngredient` references a base `Ingredient` object containing nutritional information such as calories, proteins, fats, and carbohydrates.

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