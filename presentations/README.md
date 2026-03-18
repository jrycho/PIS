# Class Diagram – Climbing Database Application

## Overview

This task contains a **UML Class diagram** describing the data structure of a web application designed to store and present information about climbing locations and routes, similar to a climbing database such as the Czech Climbing Association (Horosvaz) website.

The diagram models the main domain entities of the system, their **attributes**, **methods**, and the **relationships** between them. It represents how climbing regions, areas, locations, rocks, and individual climbing routes are organized, as well as additional attributes describing climbing conditions, route characteristics, and related media.

The purpose of the diagram is to capture the structural design of the system and to illustrate how different components of the climbing database are related and what responsibilities individual classes have.

## Core Entities

The system is organized hierarchically to reflect the structure of real-world climbing locations.

The main entities are:

* **Region** – represents a large geographic climbing region.
* **Oblast (Area)** – a climbing area located within a region.
* **Lokalita (Location)** – a specific climbing location within an area.
* **Skala (Rock / Sector)** – a rock formation or sector containing multiple climbing routes.
* **Cesta (Route)** – an individual climbing route with its own characteristics.

The hierarchical structure of the climbing database is:

**Region → Oblast → Lokalita → Skala → Cesta**

Each level contains multiple elements of the next level, allowing users to navigate from large geographic regions down to individual climbing routes.

## Attributes and Class Responsibilities

Each class contains attributes describing its properties and methods expressing its responsibilities within the model.

Higher-level classes such as **Region**, **Oblast**, **Lokalita**, and **Skala** include methods for managing their contained objects, for example:

* adding nested entities
* removing nested entities
* retrieving collections of child objects
* updating descriptive information

This reflects the hierarchical organization of the climbing database and makes the model more expressive than a diagram containing attributes only.

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

In addition to its attributes, the `Cesta` class also contains methods for updating route data and managing its linked classifications and photographs.

## Route Classification and Protection

Routes can also be categorized by additional properties describing how they are climbed and protected.

Two classification entities are used:

* **TypLezeni (Climbing Type)** – defines the style of climbing, such as sport climbing, traditional climbing, or sandstone climbing.
* **TypJisteni (Protection Type)** – defines the types of protection used on the route, such as bolts, rings, cams, or nuts.

A route may have multiple climbing types and multiple protection types, which is represented by many-to-many relationships.

These classes may also contain simple maintenance methods, such as renaming or updating classification records.

## Sun Exposure

The **TypSlunce (Sun Type)** entity describes the sunlight conditions affecting a route.

This can represent different sunlight exposure categories that help climbers determine when a route is suitable for climbing.

Routes may be associated with multiple sun exposure types.

As with the other classification classes, this entity mainly serves as a descriptive type connected to routes.

## Photographs

The **Fotografie (Photograph)** entity stores images related to climbing routes.

Each photograph contains information such as:

* image URL
* description
* author of the photograph

A single route can have multiple associated photographs.

The `Fotografie` class may also include simple methods for updating the image URL, description, or author information.

## Methods in the Diagram

Unlike a purely static data model, this class diagram also includes selected **methods** that fit the responsibilities of the entities.

These methods are mainly focused on:

* managing hierarchical relationships between entities
* updating important descriptive attributes
* attaching and removing related classifications
* managing route photographs

The included methods are not intended to represent full backend implementation, but rather the most natural object-oriented responsibilities of the classes in the model.

## Diagram Scope

The diagram focuses on the **structural data model** of the climbing database. It describes:

* entities
* attributes
* methods
* relationships between classes

The diagram does not describe:

* user interface interactions
* backend services
* database queries
* low-level application logic

The goal of this model is to represent how climbing data is organized, structured, and managed within the system using an object-oriented design.