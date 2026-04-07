from scipy.optimize import linprog
from app.optimizers.abstract_optimizer_base import AbstractOptimizerBase
import numpy as np

class linprog_optimizer(AbstractOptimizerBase):


    def __init__(self, settings, input_obj):
        self.settings = settings
        self.input_list = input_obj.get_input_list()
        self.is_indivisible = input_obj.get_is_indivisible()
        self.user_designated_values = input_obj.get_user_designated_values()
        self.n = len(self.settings.get_optimized_properties())
        self.n_in = len(self.input_list)
        self.bounds = None
        self.A_eq = None
        self.b_eq = np.array(self.settings.get_target_goal()) #b_eq is the goal in liner proggramming
        self.c = None
        self.solution = None
        self.update_flag = False


    def solve(self):
        self.c_creator()
        self.A_matrix_creator()
        self.bounds_creator()
        
        if any(self.is_indivisible) == False:
            self.solution = linprog(self.c, A_eq=self.A_eq, b_eq=self.b_eq, bounds=self.bounds, method="highs")
            #print(self.solution.x[:self.n_in])
            self.update_flag = False
        
        elif any(self.is_indivisible) == True:
            optimal_solution = linprog(self.c, A_eq=self.A_eq, b_eq=self.b_eq, bounds=self.bounds, method="highs").x[:self.n_in]
            only_indivisible = np.where(self.is_indivisible ==0, 0, optimal_solution)
            result = np.divide(only_indivisible, self.is_indivisible, out=np.zeros_like(only_indivisible, dtype=float), where=self.is_indivisible !=0)
            result = np.where((result < 0.5) & (result != 0), 0.5, result)
            rounded_vals = np.round(result+1e-8).astype(int) #rounding to full pieces
            rounded_weight = np.multiply(rounded_vals, self.is_indivisible)
            

            filtered_bounds = []
            for i in range(self.n_in):
                if rounded_vals[i] != 0:
                    filtered_bounds.append((rounded_weight[i],rounded_weight[i]))
                else:
                    filtered_bounds.append(self.bounds[i])
                #print(filtered_bounds)


            filtered_bounds.extend([(0, None) for _ in range(2 * self.n)])
            self.solution = linprog(self.c, A_eq=self.A_eq, b_eq=self.b_eq, bounds=filtered_bounds, method="highs")
            print(self.solution.x[:self.n_in])
            self.update_flag = False

        else:
            print("Error")
        
        

        """  
        Minimization vector creating, in shape of [x1, x2, x3, ..., xn, [d+], [d-]
        When calculated with, the value to minimize is [eye vector for values], + 2*[n vector for slack/excess in nutrientes]
        """
    def c_creator(self):
        c = np.zeros(self.n_in + 2 * self.n)  # Zero coefficients for x
        c[self.n_in:(self.n_in + self.n)] = self.settings.get_slack_weights()  # Weights for d+
        c[self.n_in + self.n:] = self.settings.get_excess_weights()
        self.c = c
        #minimize deviation w_plus*d_plus + w_minus*d_minus


        """ 
        Automatically creating matrix A_eq, used standart 'linprog' syntax
        for each item in input_list:
              for each atribute in optimized_properties:
                  append atribute to row
            append row to temp_A_list

        return np form (after transposition) temp_A_list:

                item1   item2   item3   item4
        cals      100     200     150     180
        carbs      20      40      30      35
        protein    10      15      12      14 ...
                                            .
                                            .
                                            .
        should be scalable on properties                     
        expanded by n*n identity matrix and n*n negative identity matrix for deviation variables
                item1   item2   item3   item4   expanded...
        cals      100     200     150     180   0
        carbs      20      40      30      35   0
        protein    10      15      12      14   0 ...
                                            .
                                            .
                                            .
        """

    def A_matrix_creator(self):
        temp_A_list = []
        for item in self.input_list:
            row = []
            for atribute in self.settings.get_optimized_properties():
                row.append(getattr(item, atribute))
            temp_A_list.append(row)
        temp_A_list = np.array(temp_A_list).T
        self.A_eq = np.hstack([temp_A_list, np.eye(self.n), -np.eye(self.n)])



        """
        results printing, needs to go to UI too, here for testing purposes, 
        TODO: flooring it to 5g? two decimal places... or try to set it already in linprog values if possible?
        """
    def print_solution(self):
        if self.update_flag == True or self.solution is None:
            self.solve()
        else:
            pass

        for parameter in self.settings.get_optimized_properties():
            #print(parameter)
            val = 0
            for item in range(self.n_in):
                val += self.solution.x[item] * getattr(self.input_list[item], parameter)
            print(f"{parameter} amount is: {val}")



    """ TODO: when known, define what from solution to return """
    """ 
    set, get methods as found in swarm_utils/AbstractOptimizerBase some references there
    """
    def get_solution(self):
        if self.update_flag is True or self.solution is None:
            self.solve()
            self.update_flag = False
        else:
            pass
        return self.solution

    def A_matrix_actualize(self):
        self.A_matrix_creator()

    def set_input_list(self, new_input_list):
        self.input_list = new_input_list
        self.A_matrix_actualize()
        self.update_flag = True

    def set_settings(self, new_settings):
        self.settings = new_settings
        self.A_matrix_actualize()
        self.update_flag = True


    def get_settings(self):
        return self.settings

    def get_input_list(self):
        return self.input_list



        """ prolly add more coefficients than priority eg. veggie part, good protein source, oils, automatic? manual override? """

        """ BOUNDS ASSESMENT LOGIC
        Creates bounds for each ingredient, based on the priority if whole food unlimited, otherwise limited by the max amount of 200g
        Later expanded for syntax purposes so deviations are unlimited

        """
    def bounds_creator(self):
        #print("creating")
        bounds = []
        for i, item in enumerate(self.input_list):
            print(item.priority)
            if self.user_designated_values[i] > 0:
                bounds.append((self.user_designated_values[i], self.user_designated_values[i]))   # lock variable
            elif item.priority == 1:
                bounds.append((0.1, None))
            else:
                bounds.append((0.1, 2))
    
        bounds.extend([(0, None) for _ in range(2 * self.n)])
        #print(bounds)
        self.bounds = bounds


    def get_json_results(self):
        if self.solution is None or self.update_flag:
            self.solve()

        if self.solution is None or not self.solution.success:
            return [], {}

        # use ingredient part only
        solution_vector = np.array(self.solution.x[:self.n_in], dtype=float)


        rounded_solution = 0.05 * np.round(solution_vector / 0.05)

        json_ingredient_weights = []
        json_total_macros = {}

        for i in range(self.n_in):
            barcode = self.input_list[i].barcode
            name = self.input_list[i].name
            grams = int(round(rounded_solution[i] * 100))

            json_ingredient_weights.append({
                "barcode": barcode,
                "name": name,
                "grams": grams,
            })

        # nutrient matrix only, shape: properties x ingredients
        A_food = self.A_eq[:, :self.n_in]
        total_macros = A_food @ rounded_solution

        for i, value in enumerate(total_macros):
            json_total_macros[self.settings.get_optimized_properties()[i]] = int(round(value))

        return json_ingredient_weights, json_total_macros co state diagram tohoto objektu

For this object specifically, the best state diagram is not the whole app flow, but the lifecycle of the optimizer instance.

This class clearly changes behavior based on:

whether data is initialized
whether internal matrices/bounds are prepared
whether the solution is current or outdated
whether solving succeeds or fails

So I would model the state diagram around optimizer readiness and solution validity.

A good version would be:

Initialized
object created in __init__
settings/input references loaded
no model built yet
solution = None
Prepared
after c_creator(), A_matrix_creator(), and bounds_creator()
optimization problem is fully assembled
Solving
solve() is running
either standard LP branch or indivisible-food branch
Solved
self.solution exists and self.solution.success == True
Failed
solve finished but no valid feasible solution
Dirty / Needs Recompute
after set_input_list() or set_settings()
internal data changed
update_flag = True
previous solution is outdated

That is the cleanest state diagram for this class.

Main transitions
__init__ → Initialized
solve() → Prepared
Prepared → Solving
Solving → Solved
Solving → Failed
set_input_list() / set_settings() → Dirty
Dirty → solve() → Prepared
get_solution() in Dirty state triggers recomputation
get_json_results() in Dirty state also triggers recomputation
Important note

In your code, Prepared is not stored as a persistent flag, it is more like an internal phase inside solve().
So in a strict UML state diagram, you can either:

keep it simple and use only stable states:
Initialized → Solving → Solved / Failed, plus Dirty

or

include Prepared as a substate inside solve().

For this class, I would recommend this simpler and cleaner interpretation:

Initialized
Dirty
Solving
Solved
Failed

Because those are the real meaningful states from outside the object.

Here is a good PlantUML version:

@startuml
title linprog_optimizer - State Diagram

[*] --> Initialized

state Initialized
state Dirty
state Solving
state Solved
state Failed

Initialized --> Solving : solve() / get_solution() / get_json_results()
Solving --> Solved : solution.success == true
Solving --> Failed : solution is None or success == false

Solved --> Dirty : set_input_list()
Solved --> Dirty : set_settings()

Failed --> Dirty : set_input_list()
Failed --> Dirty : set_settings()

Dirty --> Solving : solve() / get_solution() / get_json_results()
Dirty --> Dirty : A_matrix_actualize()\nupdate_flag = true

Solved --> Solved : get_solution()
Solved --> Solved : get_json_results()

Failed --> Solving : get_solution() / get_json_results()

@enduml

If you want, I can also make a more detailed version with composite states, where Solving contains:

creating objective vector c
creating matrix A_eq
creating bounds
solving standard LP / solving indivisible LP
do it with ready and the rest
@startuml
title linprog_optimizer - State Diagram

[*] --> Initialized

state Initialized
state Ready
state Solving
state Solved
state Failed
state Dirty

Initialized --> Ready : object created\nsolution = None\nupdate_flag = false

Ready --> Solving : solve()\nget_solution()\nget_json_results()
Solving --> Solved : solution.success == true
Solving --> Failed : solution == None\nor success == false

Solved --> Solved : get_solution()\nget_json_results()
Failed --> Solving : get_solution()\nget_json_results()

Solved --> Dirty : set_input_list()\nset_settings()
Failed --> Dirty : set_input_list()\nset_settings()
Ready --> Dirty : set_input_list()\nset_settings()

Dirty --> Ready : A_matrix_actualize()\nupdate_flag = true

@enduml

If you want it a bit more accurate to your code, Dirty --> Ready could also be labeled just with data updated, because set_input_list() and set_settings() already call A_matrix_actualize().

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

Its purpose is to describe the **object-oriented design of the system and the relationships between its core components**. do me readme in this shape for this current topic of state diagram

Here is your README-style description for the state diagram, matching the structure and tone of your class diagram:

State Diagram – Nutrition Optimization Application
Overview

This task contains a UML State diagram describing the dynamic behavior of the optimization component within the nutrition application. The diagram models how the optimizer object transitions between different internal states during its lifecycle.

The purpose of the state diagram is to represent the behavioral logic and state transitions of the optimization process, particularly how the system reacts to data changes, executes computations, and handles results.

The diagram focuses on the state management of the optimization workflow, rather than the structural design of the system.

Main States

The optimizer is modeled using several key states:

Initialized – the optimizer object is created and basic data references are assigned.
Ready – the optimizer has valid input data and is prepared to perform optimization.
Solving – the optimization process is actively being executed.
Solved – a valid solution has been successfully computed.
Failed – the optimization process finished without finding a valid solution.
Dirty – the input data or settings have changed, and the current solution is no longer valid.

These states describe the lifecycle of the optimizer and ensure correct handling of computations and updates.

Initialization Phase

The optimizer begins in the Initialized state after object creation.

In this state:

input data and settings are assigned
no optimization has been performed yet
no valid solution exists

After initialization, the optimizer transitions to the Ready state, indicating that it is prepared to run the optimization process.

Optimization Execution

When the optimization is triggered (e.g., via solve(), get_solution(), or get_json_results()), the optimizer transitions from Ready to Solving.

During the Solving state:

the objective function is constructed
constraint matrices are generated
bounds are defined
the linear programming solver is executed

After computation, two outcomes are possible:

Solved – a feasible and valid solution is found
Failed – no feasible solution exists or an error occurs
Result Handling

In the Solved state:

the solution is stored internally
repeated calls to retrieve results do not trigger recomputation
the optimizer remains stable until input data changes

In the Failed state:

no valid result is available
the system may attempt recomputation when requested
Data Updates and Recalculation

When input data changes (e.g., via set_input_list() or set_settings()), the optimizer transitions into the Dirty state.

In this state:

the previous solution is considered outdated
internal matrices may be updated
recomputation is required before obtaining valid results

From Dirty, the optimizer transitions back to Ready, indicating that it can perform optimization again with updated data.

Automatic Recalculation

If result retrieval is requested while the optimizer is in the Dirty or Failed state:

the system automatically triggers the optimization process
the optimizer transitions to Solving

This ensures that the user always receives up-to-date results without needing to manually manage the state.

Diagram Scope

The diagram focuses on the behavior of the optimization component and represents:

internal states of the optimizer
transitions triggered by method calls
handling of successful and failed computations
reaction to data changes

The diagram does not include:

detailed mathematical formulation of the optimization problem
internal implementation of the solver
user interface interactions
database or persistence logic

Its purpose is to describe the state-driven behavior and lifecycle management of the optimization process within the application.

If you want, I can also align this exactly to UML terminology (e.g., events, guards, actions) for a more “academic” version.