# Class Diagram – Climbing Database Application

## Overview

This task contains a Class diagram describing the data structure of a web application designed to store and present information about climbing locations and routes, similar to a climbing database such as the Czech Climbing Association (Horosvaz) website.

The diagram models the main domain entities of the system and the relationships between them. It represents how climbing regions, areas, locations, rocks, and individual climbing routes are organized, as well as additional attributes describing climbing conditions, route characteristics, and related media.

The purpose of the diagram is to capture the structural design of the system and to illustrate how different components of the climbing database are related.

## Core Entities

The system is organized hierarchically to reflect the structure of real-world climbing locations.

The main entities are:

* **Region** – represents a large geographic climbing region.
* **Oblast (Area)** – a climbing area located within a region.
* **Lokalita (Location)** – a specific climbing location within an area.
* **Skala (Rock / Sector)** – a rock formation or sector containing multiple climbing routes.
* **Cesta (Route)** – an individual climbing route with its own characteristics.

The hierarchical structure of the climbing database is:
Region → Oblast → Lokalita → Skala → Cesta


Each level contains multiple elements of the next level, allowing users to navigate from large geographic regions down to individual climbing routes.

## Route Attributes

The **Cesta (Route)** entity represents the most detailed level of the climbing database. Each route contains descriptive attributes that define its climbing characteristics, including:

* route name
* difficulty classification
* route length
* danger level
* route inclination
* orientation
* textual description
* first ascent author
* date of first ascent

These attributes allow climbers to understand the difficulty, safety, and historical background of each route.

## Route Classification and Protection

Routes can also be categorized by additional properties describing how they are climbed and protected.

Two classification entities are used:

* **TypLezeni (Climbing Type)** – defines the style of climbing (e.g., sport climbing, traditional climbing, sandstone climbing).
* **TypJisteni (Protection Type)** – defines the types of protection used on the route (e.g., bolts, rings, cams, nuts).

A route may have multiple climbing types and multiple protection types, which is represented by many-to-many relationships.

## Sun Exposure

The **TypSlunce (Sun Type)** entity describes the sunlight conditions affecting a route.  
This can represent different sunlight exposure categories that help climbers determine when a route is suitable for climbing.

Routes may be associated with multiple sun exposure types.

## Photographs

The **Fotografie (Photograph)** entity stores images related to climbing routes.  
Each photograph contains information such as:

* image URL
* description
* author of the photograph

A single route can have multiple associated photographs.

## Diagram Scope

The diagram focuses on the **structural data model** of the climbing database. It describes the entities, their attributes, and the relationships between them.

User interactions, backend services, and application logic are outside the scope of this diagram. The goal of this model is to represent how climbing data is organized and structured within the system.