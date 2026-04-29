# Component Diagram - Nutrition Optimization Application

## Overview

This task contains a **UML Component diagram** describing the high-level architecture of the nutrition optimization application.

The diagram shows how the main software components are organized into layers and how they communicate with each other. It focuses on the relationship between the frontend, backend API modules, database, and external food data service.

The purpose of this diagram is to provide a **clear structural overview of the deployed application components**, including the main backend route groups and their dependencies.

Unlike the class diagram, which describes internal object structure, this component diagram represents the **system architecture and communication between larger application parts**.

---

## Main Layers

The system is divided into four main layers:

* **Client Layer** - contains the user-facing Next.js frontend
* **Backend Layer** - contains the FastAPI application and its route modules
* **Database Layer** - contains MongoDB Atlas used for persistent storage
* **External Services** - contains the Open Food Facts API used for product and barcode data

This separation makes the responsibilities of each part of the system clear and keeps the architecture modular.

---

## Client Layer

The **Next.js Frontend** is responsible for the user interface and user interactions.

It allows users to:

* log in and manage account data
* create and edit meals
* add ingredients
* configure nutrition settings
* run meal optimization
* view tracking and optimization results

The frontend communicates with the backend using **HTTP / JSON** requests.

It can also communicate with the **Open Food Facts API** for product search and lookup functionality.

---

## Backend Layer

The backend is represented by the **FastAPI Backend** component.

It acts as the central application component and exposes multiple route groups:

* **Login Routes** - handle authentication and login-related operations
* **Users Routes** - manage user account data
* **User Functions** - provide shared user-related backend operations
* **Meal Logs Routes** - create, update, and manage meal logs
* **Optimization Routes** - run the nutrition optimization process
* **Settings Routes** - manage nutrition goals and optimization settings
* **Tracker Routes** - provide progress and history tracking
* **Testing Routes** - expose testing or development support endpoints

The backend coordinates business logic, database access, optimization processing, and communication with external services.

---

## Database Layer

The application uses **MongoDB Atlas** as the persistent database.

Backend route modules access the database for different types of data:

* authentication data
* user profile data
* user settings
* meal logs and ingredients
* optimization inputs and results
* progress and history records

MongoDB Atlas stores the main application state and supports user-specific data isolation.

---

## External Services

The system uses the **Open Food Facts API** as an external service.

This service provides product and food data, especially for:

* barcode lookup
* product search
* ingredient information
* nutritional values

Both the frontend and backend can interact with the Open Food Facts API:

* the frontend uses it for direct product search and lookup
* the backend uses it when product data is needed during barcode or ingredient processing

---

## Component Relationships

The main communication paths are:

* **Next.js Frontend -> FastAPI Backend** through HTTP / JSON
* **FastAPI Backend -> Route Components** through internal backend routing
* **Route Components -> MongoDB Atlas** for persistent data operations
* **Frontend <-> Open Food Facts API** for product search and lookup
* **Backend <-> Open Food Facts API** for barcode and product data

The backend is the central coordinator of the system. It receives requests from the frontend, delegates them to the correct route module, accesses the database, and returns structured responses.

---

## Typical Application Flow

1. User performs an action in the Next.js frontend
2. Frontend sends an HTTP / JSON request to FastAPI
3. FastAPI forwards the request to the appropriate route component
4. Route component reads or writes data in MongoDB Atlas
5. If needed, product data is retrieved from Open Food Facts
6. Backend returns the result to the frontend
7. Frontend displays the updated state to the user

This flow applies to common features such as login, meal creation, ingredient lookup, settings management, optimization, and progress tracking.

---

## Architecture Characteristics

Key characteristics of the component architecture:

* clear separation between frontend, backend, database, and external services
* FastAPI backend acts as the main application controller
* route modules divide backend responsibilities by feature area
* MongoDB Atlas is used only for persistence, not application logic
* Open Food Facts is isolated as an external data provider
* communication between frontend and backend uses standard HTTP / JSON

This structure supports maintainability because each component has a specific responsibility.

---

## Diagram Scope

The diagram focuses on **high-level system components and their dependencies**.

It represents:

* application layers
* frontend-backend communication
* backend route components
* database dependencies
* external API usage

The diagram does not include:

* internal class structure
* database schema details
* exact API endpoint definitions
* low-level optimization algorithm steps
* frontend UI implementation details

Its purpose is to describe the **component-level architecture of the nutrition optimization application**.
