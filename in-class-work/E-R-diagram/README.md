# E-R Diagram - Courier Food Delivery Database

## Overview

This task contains an **Entity-Relationship diagram** describing a basic database structure for a courier-based food delivery service. The model captures the main entities involved in order delivery and the relationships between customers, couriers, restaurants, vehicles, and orders.

The purpose of the diagram is to show the core data structure of the system and demonstrate the design process of an E-R model for a real-world service domain.

## Main Entities

The diagram includes the following entities:

* **Courier** - represents a delivery courier responsible for transporting orders.
* **Vehicle** - represents a vehicle used by couriers during deliveries.
* **Order** - represents a food delivery order placed by a customer and assigned to a courier and restaurant.
* **Customer** - represents a customer who creates food delivery orders.
* **Vehicle Usage** - represents the use of a specific vehicle by a courier during a defined time interval.
* **Restaurant** - represents a restaurant that prepares orders for customers.

## Relationships

The model captures several important relationships:

* a **Customer** can create multiple **Orders**
* a **Courier** can deliver multiple **Orders**
* a **Restaurant** can prepare multiple **Orders**
* a **Courier** can use different **Vehicles** over time through the **Vehicle Usage** entity

The **Vehicle Usage** entity is used to model the time-based assignment of vehicles to couriers, including the start and end of usage.

## Diagram Scope

The diagram focuses on the **core database-level structure** of a courier food delivery system. It represents:

* main entities
* primary and foreign keys
* relationships between entities

The diagram does not include:

* detailed business rules
* application logic
* payment processing
* menu or item-level ordering details

Its purpose is to present a simple and understandable relational model for the selected domain.
