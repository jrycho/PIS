# Use Case Diagram – Nutrition Optimization Application

## Overview

This task contains a Use Case diagram describing the frontend functionality of an application designed to optimize food portions based on nutritional goals (macronutrients). The diagram models the interaction between system actors and the main user-facing features of the application.

The purpose of the diagram is to capture the functional requirements of the system from the user’s perspective and to illustrate how different actors interact with the application.

## Actors

Two actors interact with the system:

* **User** – a standard application user who manages meals, nutritional goals, and ingredients, and uses the optimization functionality.
* **Admin** – a privileged actor responsible for system administration tasks.

## User Functionality

The user can perform the following actions within the application:

* Register a new account
* Log in and log out
* Manage personal nutrition goals
* Create and manage meals
* Send meals and goals to the optimization process
* View optimization results
* Track progress toward nutrition goals
* Manage custom ingredients

Ingredient management supports several ways of creating ingredients:

* adding an ingredient directly,
* adding an ingredient as a complete nutritional entry,
* creating an ingredient from multiple ingredients.

Users can also delete their ingredients and share their custom ingredient database.

## Administration

The administrator has additional permissions to maintain the system. Administrative actions include:

* accessing system administration
* managing users
* deleting system logs

## Diagram Scope

The diagram focuses on **frontend-level use cases**, representing interactions visible to users of the application. Internal backend processes and database operations are not included in this model.
