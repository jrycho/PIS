



In addition to the core optimization process, both optimizers incorporate user-defined constraints, namely fixed ingredient quantities and discrete (integer-multiplied) portions. Fixed quantities are enforced directly within the bounds creation step, where the corresponding decision variables are restricted to a single value, effectively removing their degrees of freedom from the optimization process.

Handling of discrete portions is performed in a two-stage procedure. First, an initial optimization is carried out without enforcing integrality constraints. Based on this preliminary solution, the continuous values of selected ingredients are approximated to the nearest feasible integer multiples of their respective portion sizes. These adjusted values are then fixed by updating their bounds accordingly. Finally, a second optimization is executed over the remaining variables, ensuring that the overall solution respects both the discrete constraints and the original optimization objective. make according plantuml diagram that represents it well

@startuml
title Optimization with Fixed Quantities and Discrete Portions

start

:Load user-defined constraints;
note right
Includes:
- fixed ingredient quantities
- discrete portion requirements
end note

:Create variable bounds;

if (Ingredient has fixed quantity?) then (yes)
  :Set lower bound = upper bound = fixed value;
  note right
  Variable is fixed,
  no freedom in optimization
  end note
else (no)
  :Keep variable bounds flexible;
endif

:Run initial optimization\n(without integrality enforcement);

if (Any discrete-portion ingredients?) then (yes)
  :Identify selected ingredients\nwith discrete constraints;
  :Approximate continuous solution values\nto nearest feasible integer multiples\nof portion sizes;
  :Update bounds for these ingredients\nso lower bound = upper bound = adjusted value;
  note right
  Discrete variables are now fixed
  to feasible portion-based values
  end note
  :Run second optimization\non remaining variables;
else (no)
endif

:Return final solution;
note right
Final solution satisfies:
- fixed quantity constraints
- discrete portion constraints
- optimization objective
end note

stop
@enduml
I can also make it as a cleaner activity diagram with swimlanes or as a flowchart version for a thesis/report.


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
        TODO: undividable foods.
        """
    def bounds_creator(self):
        #print("creating")
        bounds = []
        for i, item in enumerate(self.input_list):
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

        return json_ingredient_weights, json_total_macros check consistency with this

Yes — the original description is broadly consistent with this code, but there are a few important implementation-specific details. The biggest one is that your linprog optimizer fixes indivisible ingredients only when the first continuous solution gives them a nonzero value. Then it reruns the optimization with those rounded values locked.

Here is a PlantUML activity diagram that matches the code more closely:

@startuml
title linprog optimizer - constraint handling and two-stage indivisible logic

start

:solve();

:Create objective vector c;
note right
c_creator()

- ingredient variables: coefficient 0
- slack variables d+: slack weights
- excess variables d-: excess weights
end note

:Create equality matrix A_eq;
note right
A_matrix_creator()

A_eq = [food nutrient matrix | I | -I]
b_eq = target_goal
end note

:Create bounds;
note right
bounds_creator()

For each ingredient i:
- if user_designated_values[i] > 0
    bound = (fixed, fixed)
- else if priority == 1
    bound = (0.1, None)
- else
    bound = (0.1, 2)

Then extend bounds for deviation variables:
(0, None) for all slack/excess variables
end note

if (Any ingredient indivisible?) then (no)
  :Run linprog once with original bounds;
  :Store solution;
  :update_flag = False;
  stop
else (yes)

  :Run first linprog with original bounds;
  :Take ingredient part of solution x[:n_in];

  :Keep only indivisible ingredients;
  note right
  only_indivisible =
  where(is_indivisible == 0, 0, optimal_solution)
  end note

  :Convert to portion counts;
  note right
  result = optimal_solution / portion_size
  only for indivisible ingredients
  end note

  :Round portion counts to nearest integer;
  :Compute rounded weights;
  note right
  rounded_vals = round(result)
  rounded_weight = rounded_vals * portion_size
  end note

  :Build filtered bounds;
  note right
  For each ingredient i:
  - if rounded_vals[i] != 0
      bound = (rounded_weight[i], rounded_weight[i])
  - else
      keep original bound
  end note

  :Extend filtered bounds for deviation variables;
  :Run second linprog with filtered bounds;
  :Store final solution;
  :update_flag = False;
  stop
endif

@enduml
A few consistency notes with your actual implementation:

Fixed user quantities
Your textual description matches the code exactly:
user_designated_values[i] > 0 leads to (value, value) in bounds_creator().

Discrete handling is only approximate, not true integer optimization
Your description says discrete values are “approximated to the nearest feasible integer multiples,” which is correct.
This is not a mixed-integer solve. It is:

continuous LP

rounding

fixing rounded variables

second continuous LP

Only nonzero rounded indivisible ingredients get fixed
This is a subtle but important code detail:

if rounded_vals[i] != 0:
    filtered_bounds.append((rounded_weight[i], rounded_weight[i]))
else:
    filtered_bounds.append(self.bounds[i])
So if an indivisible ingredient gets rounded to zero, it is not explicitly fixed to zero. It keeps its original bounds and may still change in the second optimization.

Indivisible values represent portion sizes
From this line:

result = optimal_solution / self.is_indivisible
self.is_indivisible is functioning like a vector of portion sizes, with 0 meaning divisible ingredient.

Deviation variables are always continuous and nonnegative
Both solves keep slack/excess variables with bounds (0, None).

If you want, I can also make:

a sequence diagram for solve(),

a class-method interaction diagram, or

a cleaner thesis-style flowchart with more formal wording.


import numpy as np
from app.optimizers.abstract_optimizer_base import AbstractOptimizerBase
from mealpy import FloatVar

""" TODO: Needs further commenting, error handling, testing """



"""  
BaseOptimizer - class for swarm optimizers, used in GWO, WOA, PSO
solve: uses solver on given problem
print solution: ensures return of previously calculated values
get solution: returns solution if previously calculated, otherwise calls solve
A_matrix_actualize: A matrix is a properties matrix used in optimizers (differently on usage), this method actualizes it by calling relevant creator, should be called upon settings and input changes
set_input_list - 
get_input_list - set/get method to change input list; should actualize A matrix, should set update_flag = True, indicating need for recalculation if called print_solution
set_settings
get_settings - set/get method to change settings; should actualize A matrix, should set update_flag = True, indicating need for recalculation if called print_solution
bounds_creator - creates bounds for optimization problem, should be called in solve, different for each optimizer
>>
swarn_fitness_function__for_genA - fitness func, calculate solution difference of all propperties, splits them to negative and positive vals, multiplies them by weights, returns sum of differences to find minimal solution.
"""


class BaseOptimizer(AbstractOptimizerBase):


    def __init__(self, settings, input_obj):
        self.settings = settings
        self.input_list = input_obj.get_input_list()
        self.is_indivisible = input_obj.get_is_indivisible()
        self.user_designated_values = input_obj.get_user_designated_values()
        self.solution = None
        self.A_matrix = None
        self.update_flag = False
        self.used_optimizer = None
        self.swarm_settings = None
        
    """  
    Prints self.solution, if not calculated yet, calls solve, terminal results
    """
    def print_solution(self):
        if self.solution is None or self.update_flag == True:
            self.solve()
            print("recalculated")
        else:
            pass
        print(self.solution)
        for parameter in self.settings.get_optimized_properties():
                #print(parameter)
            val = 0
            for item in range(len(self.input_list)):
                val += self.solution[item] * getattr(self.input_list[item], parameter)
            print(f"{parameter} amount is: {val}")

    
    def get_solution(self):
        return self.solution

    """  
    Should be called upon settings and input changes, creates A matrix for optimization
    """
    def A_matrix_actualize(self):
        self.A_matrix = self.properties_matrix_creator_for_genA(self.input_list, self.settings.get_optimized_properties())


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
    
    """ 
    For whole food item infinite upper bound for non-whole foods minimum is 10g max 200g
    TODO: user bound settings, fruit/veggies
    """
    def bounds_creator(self, input_list):
        print(self.user_designated_values)
        lower_bounds = []
        upper_bounds = []
        for item in range(len(input_list)):
            if self.user_designated_values[item] != 0:
                lower_bounds.append(self.user_designated_values[item])
                upper_bounds.append(self.user_designated_values[item])

            elif self.input_list[item].priority == 1:
                lower_bounds.append(0.1)
                upper_bounds.append(15)
            else:
                lower_bounds.append(0.1)
                upper_bounds.append(2)
        return np.array(lower_bounds), np.array(upper_bounds)
    

    """  
    makes list of lists of properties for each item in input list, then transposes it to get matrix of properties for each item in input list
    return:
             item1   item2   item3   item4
    cals      100     200     150     180
    carbs      20      40      30      35
    protein    10      15      12      14 ...
                                        .
                                        .
                                        .
    should be scalable on properties                          
    """
    def properties_matrix_creator_for_genA(self, input_list, optimized_properties):
        temp_A_list = []
        for item in input_list:
            row = []
            for atribute in optimized_properties:
                row.append(getattr(item, atribute))
            temp_A_list.append(row)
        #ret_mat = np.array(temp_A_list).T
        #print(ret_mat)
        return np.array(temp_A_list).T
    

    """  
    Function to be minimized, wieghted squared sum of differences between target goal and solution
    """
    def swarm_fitness_function_for_genA(self, sol, A_matrix, target_goal):
        solution = target_goal - np.matmul(A_matrix,sol)
        neg_sol = np.where(solution < 0, solution, 0)
        pos_sol = np.where(solution > 0, solution, 0)
        minimize_func = (-np.dot(neg_sol, self.settings.get_excess_weights().T ) + np.dot(pos_sol, self.settings.get_slack_weights().T))
        return minimize_func


    """  
    mealpy problem dict creator:
    Args
    lower_bounds: lower bounds for optimization problem
    upper_bounds: upper bounds for optimization problem
    A_matrix: properties matrix for optimization problem, calculated with in fitness, must be dynamically input
    target_goal: target goal for optimization problem, should be dynamic, calculated with in fitness


    obj_func: fitness function
    bounds: lower and upper bounds for optimization problem
    minmax: min or max
    verbose: print progress
    log_to: print to file
    """
    def problem_dict_creator(self, lower_bounds, upper_bounds, A_matrix, target_goal):
        """ Problem dict method """
        problem_dict = {
        "obj_func": lambda sol: self.swarm_fitness_function_for_genA(sol, A_matrix, target_goal),  # Pass target
        "bounds": FloatVar(lb=lower_bounds, ub=upper_bounds, name="delta"),
        "minmax": "min",  # Minimize the difference
        "verbose": False,
        "log_to": None,
        }
        return problem_dict


    """  
    Epoch and Population size form swarm settings, returns mealpy dictionary with according parameters and solver
    """    
    def create_model(self, specification):
        if specification == "final_calculation":
            params = self.swarm_settings.get_params()

        elif specification == "guess_calculation":
            params = self.swarm_settings.get_guess_params()

        return self.used_optimizer(epoch=params["epoch"], pop_size=params["pop_size"], verbose=False)


    
    def solve(self):
        """ If no indicator, do a solve. If indivisibility indicator run solve, extract optimal values and round, cut from A matrix (make lambda parsable), cut from bounds, run GWO, reconstruct"""

        if self.A_matrix is None: #called if Not calculated before
            self.A_matrix = self.properties_matrix_creator_for_genA(self.input_list,self.settings.get_optimized_properties()) #changes should automatically call properties_matrix_creator_for_genA in BaseOptimizer
            #self.bounds = self.bounds_creator(self.input_list)
        else:
            pass
        
            """ Continuous space search """
        if any(self.is_indivisible) == False:
            lower_bounds, upper_bounds = self.bounds_creator(self.input_list) #creates bounds
            problem_dict = self.problem_dict_creator(lower_bounds, upper_bounds, self.A_matrix, self.settings.get_target_goal())

            model = self.create_model("final_calculation") #creates model with default parameters

            self.solution = model.solve(problem_dict).solution #solve problem
            self.update_flag = False #indicates calculated solution for printing


            """ Space with required indivisibility """
        elif any(self.is_indivisible) == True:
            """ Quicksearch to get closer to optimal solution by indivisible ingredients """
            #region
            lower_bounds, upper_bounds = self.bounds_creator(self.input_list) #creates bounds
            problem_dict = self.problem_dict_creator(lower_bounds, upper_bounds, self.A_matrix,self.settings.get_target_goal())
            model = self.create_model("guess_calculation") #creates model with default parameters
            optimal_solution = model.solve(problem_dict).solution #optimal guess
            print(f"optimal solution", optimal_solution)
            print("self.is_indivisible", self.is_indivisible)
            only_indivisible = np.where(self.is_indivisible ==0, 0, optimal_solution) #if self.is_indivisible ==0; puts there 0, else puts optimal value


            print(f"only indivisible", only_indivisible)
            #endregion  first round search


            result = np.divide(only_indivisible, self.is_indivisible, out=np.zeros_like(only_indivisible, dtype=float), where=self.is_indivisible !=0) #divide optimal values by pieces
            result = np.where((result > 0) & (result < 1), 1, result) #if result > 0 and result < 1, replace with 1, else keep result
            rounded_vals = np.round(result+1e-8).astype(int) #rounding to full pieces
            #put somewhere to reconstruct results
            filtered_A = self.A_matrix[:, rounded_vals == 0] #should leave only columns where indivisible
            filtered_lower_bounds = lower_bounds[rounded_vals == 0] #should leave only columns where indivisible
            filtered_upper_bounds = upper_bounds[rounded_vals == 0] #should leave only columns where indivisible
            mask = rounded_vals.copy()
            #print(mask)
            results = np.multiply(mask.astype(float), self.is_indivisible)
            new_target_goal = self.settings.get_target_goal() - np.matmul(self.A_matrix,results).T
            #print(new_target_goal)
            problem_dict = self.problem_dict_creator(filtered_lower_bounds, filtered_upper_bounds, filtered_A, new_target_goal) #new dict
            model = self.create_model("final_calculation") #creates model with default parameters
            
            #values reconstruction
            solution = model.solve(problem_dict)
            results[mask == 0] = solution.solution
            self.solution = results
            self.update_flag = False #indicates calculated solution for printing

        else:
            print("Error")

    """  
    method to be called via API to get results in JSON format
    dictionaries
    json - weights of ingredients, macros, etc
    json_total_macros - total macros of all 
     
    
    """
    def get_json_results(self):
        print(self.solution)
        if self.solution is None:
            self.solve()

        rounded_solution = 0.05* np.round(self.solution/0.05)
        json_ingredient_weights = []
        json_total_macros = {}
        for iterator in range(len(self.input_list)):
            barcode = self.input_list[iterator].barcode
            name = self.input_list[iterator].name
            grams = int(float(rounded_solution[iterator] * 100))
            json_ingredient_weights.append({
            "barcode": barcode,
            "name": name,
            "grams": grams,})
        
        
        #print(json_ingredient_weights)
        total_macros = np.matmul(self.A_matrix, rounded_solution)
        for iterator in range(len(total_macros)):
            json_total_macros[self.settings.get_optimized_properties()[iterator]] = int(total_macros[iterator])



        return json_ingredient_weights, json_total_macros

"""  
Swarm settings class for GWO, WOA, PSO, GA
sets epoch and pop_size for each optimizer
!Less ingredients needs longer epochs? 

"""
class swarm_settings():
    def __init__(self, used_optimizer):
        self.used_optimizer = used_optimizer
        print(self.used_optimizer)
        
        self.GWO_epoch = 100
        self.GWO_pop_size = 50
        self.WOA_epoch = 100
        self.WOA_pop_size = 50
        self.PSO_epoch = 100
        self.PSO_pop_size = 50
        self.GA_epoch = 100
        self.GA_pop_size = 50

        self.GWO_epoch_guess = 50
        self.GWO_pop_size_guess = 30
        self.WOA_epoch_guess = 100
        self.WOA_pop_size_guess = 50
        self.PSO_epoch_guess = 100
        self.PSO_pop_size_guess = 50
        self.GA_epoch_guess = 100
        self.GA_pop_size_guess = 50

    def get_params(self):
        name = self.used_optimizer
        return {
            "epoch": getattr(self, f"{name}_epoch"),
            "pop_size": getattr(self, f"{name}_pop_size"),
        }
    
    def get_guess_params(self):
        name = self.used_optimizer
        return {
            "epoch": getattr(self, f"{name}_epoch_guess"),
            "pop_size": getattr(self, f"{name}_pop_size_guess"),
        } and rom scipy.optimize import linprog from app.optimizers.abstract_optimizer_base import AbstractOptimizerBase import numpy as np class linprog_optimizer(AbstractOptimizerBase): def __init__(self, settings, input_obj): self.settings = settings self.input_list = input_obj.get_input_list() self.is_indivisible = input_obj.get_is_indivisible() self.user_designated_values = input_obj.get_user_designated_values() self.n = len(self.settings.get_optimized_properties()) self.n_in = len(self.input_list) self.bounds = None self.A_eq = None self.b_eq = np.array(self.settings.get_target_goal()) #b_eq is the goal in liner proggramming self.c = None self.solution = None self.update_flag = False def solve(self): self.c_creator() self.A_matrix_creator() self.bounds_creator() if any(self.is_indivisible) == False: self.solution = linprog(self.c, A_eq=self.A_eq, b_eq=self.b_eq, bounds=self.bounds, method="highs") #print(self.solution.x[:self.n_in]) self.update_flag = False elif any(self.is_indivisible) == True: optimal_solution = linprog(self.c, A_eq=self.A_eq, b_eq=self.b_eq, bounds=self.bounds, method="highs").x[:self.n_in] only_indivisible = np.where(self.is_indivisible ==0, 0, optimal_solution) result = np.divide(only_indivisible, self.is_indivisible, out=np.zeros_like(only_indivisible, dtype=float), where=self.is_indivisible !=0) rounded_vals = np.round(result+1e-8).astype(int) #rounding to full pieces rounded_weight = np.multiply(rounded_vals, self.is_indivisible) filtered_bounds = [] for i in range(self.n_in): if rounded_vals[i] != 0: filtered_bounds.append((rounded_weight[i],rounded_weight[i])) else: filtered_bounds.append(self.bounds[i]) #print(filtered_bounds) filtered_bounds.extend([(0, None) for _ in range(2 * self.n)]) self.solution = linprog(self.c, A_eq=self.A_eq, b_eq=self.b_eq, bounds=filtered_bounds, method="highs") print(self.solution.x[:self.n_in]) self.update_flag = False else: print("Error") """ Minimization vector creating, in shape of [x1, x2, x3, ..., xn, [d+], [d-] When calculated with, the value to minimize is [eye vector for values], + 2*[n vector for slack/excess in nutrientes] """ def c_creator(self): c = np.zeros(self.n_in + 2 * self.n) # Zero coefficients for x c[self.n_in:(self.n_in + self.n)] = self.settings.get_slack_weights() # Weights for d+ c[self.n_in + self.n:] = self.settings.get_excess_weights() self.c = c #minimize deviation w_plus*d_plus + w_minus*d_minus """ Automatically creating matrix A_eq, used standart 'linprog' syntax for each item in input_list: for each atribute in optimized_properties: append atribute to row append row to temp_A_list return np form (after transposition) temp_A_list: item1 item2 item3 item4 cals 100 200 150 180 carbs 20 40 30 35 protein 10 15 12 14 ... . . . should be scalable on properties expanded by n*n identity matrix and n*n negative identity matrix for deviation variables item1 item2 item3 item4 expanded... cals 100 200 150 180 0 carbs 20 40 30 35 0 protein 10 15 12 14 0 ... . . . """ def A_matrix_creator(self): temp_A_list = [] for item in self.input_list: row = [] for atribute in self.settings.get_optimized_properties(): row.append(getattr(item, atribute)) temp_A_list.append(row) temp_A_list = np.array(temp_A_list).T self.A_eq = np.hstack([temp_A_list, np.eye(self.n), -np.eye(self.n)]) """ results printing, needs to go to UI too, here for testing purposes, TODO: flooring it to 5g? two decimal places... or try to set it already in linprog values if possible? """ def print_solution(self): if self.update_flag == True or self.solution is None: self.solve() else: pass for parameter in self.settings.get_optimized_properties(): #print(parameter) val = 0 for item in range(self.n_in): val += self.solution.x[item] * getattr(self.input_list[item], parameter) print(f"{parameter} amount is: {val}") """ TODO: when known, define what from solution to return """ """ set, get methods as found in swarm_utils/AbstractOptimizerBase some references there """ def get_solution(self): if self.update_flag is True or self.solution is None: self.solve() self.update_flag = False else: pass return self.solution def A_matrix_actualize(self): self.A_matrix_creator() def set_input_list(self, new_input_list): self.input_list = new_input_list self.A_matrix_actualize() self.update_flag = True def set_settings(self, new_settings): self.settings = new_settings self.A_matrix_actualize() self.update_flag = True def get_settings(self): return self.settings def get_input_list(self): return self.input_list """ prolly add more coefficients than priority eg. veggie part, good protein source, oils, automatic? manual override? """ """ BOUNDS ASSESMENT LOGIC Creates bounds for each ingredient, based on the priority if whole food unlimited, otherwise limited by the max amount of 200g Later expanded for syntax purposes so deviations are unlimited TODO: undividable foods. """ def bounds_creator(self): #print("creating") bounds = [] for i, item in enumerate(self.input_list): if self.user_designated_values[i] > 0: bounds.append((self.user_designated_values[i], self.user_designated_values[i])) # lock variable elif item.priority == 1: bounds.append((0.1, None)) else: bounds.append((0.1, 2)) bounds.extend([(0, None) for _ in range(2 * self.n)]) #print(bounds) self.bounds = bounds def get_json_results(self): if self.solution is None or self.update_flag: self.solve() if self.solution is None or not self.solution.success: return [], {} # use ingredient part only solution_vector = np.array(self.solution.x[:self.n_in], dtype=float) rounded_solution = 0.05 * np.round(solution_vector / 0.05) json_ingredient_weights = [] json_total_macros = {} for i in range(self.n_in): barcode = self.input_list[i].barcode name = self.input_list[i].name grams = int(round(rounded_solution[i] * 100)) json_ingredient_weights.append({ "barcode": barcode, "name": name, "grams": grams, }) # nutrient matrix only, shape: properties x ingredients A_food = self.A_eq[:, :self.n_in] total_macros = A_food @ rounded_solution for i, value in enumerate(total_macros): json_total_macros[self.settings.get_optimized_properties()[i]] = int(round(value)) return json_ingredient_weights, json_total_macros if possible make it one extrapolated functionality diagram no need for big explicity


Yes — these two implementations are consistent enough to be merged into one higher-level functionality diagram.

The shared logic is:

load inputs, settings, bounds, matrix/objective prep

enforce fixed quantities directly in bounds

check whether indivisible ingredients exist

if not, run one optimization

if yes, run a preliminary optimization, derive rounded discrete assignments, then rerun optimization on the remaining free space

return final solution and JSON-style results

A compact extrapolated PlantUML diagram for both BaseOptimizer and linprog_optimizer would be:

@startuml
title Unified optimizer functionality

start

:Initialize optimizer;
note right
Load:
- settings
- input_list
- is_indivisible
- user_designated_values
end note

:Build optimization structures;
note right
Depending on optimizer:
- linprog: c, A_eq, b_eq, bounds
- swarm: A_matrix, problem_dict, bounds
end note

:Create bounds;
note right
For each ingredient:
- if user-designated value exists
  -> fix variable to that value
- else apply normal lower/upper bounds
end note

if (Any indivisible ingredient?) then (no)
  :Run main optimization once;
  note right
  linprog:
  continuous LP

  swarm:
  final continuous search
  end note

else (yes)
  :Run preliminary optimization;
  note right
  Used to estimate values
  of indivisible ingredients
  end note

  :Extract indivisible variables only;
  :Convert to portion counts;
  :Round to nearest feasible integer multiple;

  if (Rounded value > 0 ?) then (yes)
    :Fix rounded indivisible quantities;
  else (no)
    :Keep original bounds / keep variable free;
  endif

  :Rebuild reduced / filtered problem;
  note right
  linprog:
  fix rounded bounds, optimize again

  swarm:
  subtract fixed contribution
  from target, optimize remaining variables,
  then reconstruct full solution
  end note

  :Run final optimization;
endif

:Store final solution;
:update_flag = False;

:Optionally round output for presentation;
note right
Example:
round ingredient weights to 0.05
convert to grams
recompute final macro totals
end note

:Return solution / JSON results;

stop
@enduml
One sentence summary for your text below the diagram:

Both optimizers follow the same two-level constraint strategy: fixed quantities are enforced directly through variable bounds, while indivisible ingredients are handled through an initial continuous solve, rounding to feasible portion multiples, and a final re-optimization over the remaining degrees of freedom.

I can also make this into an even simpler thesis-ready block diagram with only 6–8 boxes.


are in those two the A matrixes same?

Short answer: yes in meaning, slightly different in structure and usage.

✅ What is the same
In both optimizers, the core A matrix (nutrient/property matrix) is identical in meaning:

Rows = optimized properties (e.g., calories, protein, carbs)

Columns = ingredients

Values = contribution of ingredient j to property i

So this part is the same:

          item1  item2  item3
protein     10     15     12
carbs       20     40     30
calories   100    200    150
Code equivalence
linprog:

temp_A_list = np.array(temp_A_list).T
swarm:

return np.array(temp_A_list).T
👉 This base matrix is identical in both implementations.

⚠️ What is different
1. linprog → expanded A matrix
In linprog_optimizer, the matrix is extended:

self.A_eq = np.hstack([
    temp_A_list,     # food matrix
    np.eye(self.n),  # slack (d+)
    -np.eye(self.n)  # excess (d-)
])
So actual structure is:

[A_food | I | -I]
This allows:

exact equality constraint:
A_food * x + d+ - d- = target

2. swarm → pure A matrix only
In BaseOptimizer, you only use:

self.A_matrix = np.array(temp_A_list).T
No expansion.

Instead:

deviations are handled inside the fitness function, not via variables

solution = target_goal - A_matrix @ sol
Then split into:

positive (slack)

negative (excess)

🔑 Key conceptual difference
Aspect	linprog	swarm optimizer
A matrix base	same	same
Matrix used in solver	expanded `[A	I
Slack/excess handling	explicit variables	implicit in fitness
Constraint type	hard equality	soft penalty
🧠 Interpretation
linprog = strict mathematical formulation
→ deviations are variables in the system

swarm = heuristic formulation
→ deviations are penalized errors

✅ Final answer
The base A matrix (ingredient–property matrix) is the same in both implementations.
However, in the linprog optimizer it is extended with identity matrices to explicitly model slack and excess variables, while in the swarm optimizer it remains unmodified and deviations are handled inside the fitness function.

If you want, I can turn this into a clean comparison diagram (very useful for thesis).


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

        return json_ingredient_weights, json_total_macros now make me nice plantuml diagram

@startuml
title linprog_optimizer - optimization workflow

start

:Initialize optimizer;
note right
Load:
- settings
- input_list
- is_indivisible
- user_designated_values
- n = number of optimized properties
- n_in = number of ingredients
end note

:solve();

:Create objective vector c;
note right
c_creator()

c = [ingredient vars, slack vars, excess vars]

- ingredient coefficients = 0
- slack coefficients = slack_weights
- excess coefficients = excess_weights
end note

:Create equality matrix A_eq;
note right
A_matrix_creator()

1. Build food-property matrix
2. Transpose it
3. Expand as:

A_eq = [A_food | I | -I]
b_eq = target_goal
end note

:Create bounds;
note right
bounds_creator()

For each ingredient i:
- if user_designated_values[i] > 0
    -> (fixed, fixed)
- else if priority == 1
    -> (0.1, None)
- else
    -> (0.1, 2)

Then append bounds for deviation variables:
(0, None)
end note

if (Any indivisible ingredient?) then (no)
  :Run single linprog solve;
  note right
  linprog(c, A_eq, b_eq, bounds)
  method = "highs"
  end note

  :Store final solution;
  :update_flag = False;

else (yes)
  :Run initial continuous linprog solve;
  :Extract ingredient part x[:n_in];

  :Keep only indivisible ingredients;
  note right
  only_indivisible =
  where(is_indivisible == 0, 0, optimal_solution)
  end note

  :Convert weights to portion counts;
  note right
  result = only_indivisible / is_indivisible
  end note

  :Apply minimum half-piece rule;
  note right
  if 0 < result < 0.5
  then result = 0.5
  end note

  :Round portion counts to integers;
  :Convert rounded counts back to weights;
  note right
  rounded_vals = round(result)
  rounded_weight = rounded_vals * is_indivisible
  end note

  :Build filtered ingredient bounds;
  note right
  For each ingredient i:
  - if rounded_vals[i] != 0
      -> fix to rounded_weight[i]
  - else
      -> keep original bound
  end note

  :Append deviation-variable bounds again;
  :Run second linprog solve with filtered bounds;
  :Store final solution;
  :update_flag = False;
endif

:Solution available;

if (JSON output requested?) then (yes)
  :Take ingredient solution x[:n_in];
  :Round solution to nearest 0.05;
  :Convert to grams;
  :Compute total macros from A_food;
  :Return ingredient list + macro totals;
else (no)
endif

stop
@enduml
A slightly cleaner thesis-style version:

@startuml
title Linear programming optimizer with fixed and indivisible ingredients

start

:Load settings and ingredient data;
:Build objective vector c;
:Build equality matrix A_eq = [A_food | I | -I];
:Build variable bounds;

if (Indivisible ingredients present?) then (No)
  :Solve LP once;
else (Yes)
  :Solve relaxed LP;
  :Extract indivisible ingredient values;
  :Convert to portion multiples;
  :Apply minimum half-piece threshold;
  :Round to feasible integer counts;
  :Fix rounded indivisible quantities in bounds;
  :Re-solve LP with updated bounds;
endif

:Obtain final solution;
:Round displayed quantities to 0.05 units;
:Compute final nutrient totals;
:Return result;

stop
@enduml
The second one is better for a report, while the first one matches the code logic more precisely.


skip the json part just end it

Here is the cleaned-up version ending at the final solution (no JSON part):

@startuml
title linprog_optimizer - optimization workflow

start

:Initialize optimizer;
note right
Load:
- settings
- input_list
- is_indivisible
- user_designated_values
end note

:solve();

:Create objective vector c;
note right
Ingredient vars = 0
Slack vars = slack_weights
Excess vars = excess_weights
end note

:Create equality matrix A_eq;
note right
A_eq = [A_food | I | -I]
b_eq = target_goal
end note

:Create bounds;
note right
For each ingredient:
- fixed value -> (val, val)
- priority 1 -> (0.1, ∞)
- otherwise -> (0.1, 2)

Deviation vars -> (0, ∞)
end note

if (Any indivisible ingredient?) then (no)
  :Run linprog (single solve);
else (yes)
  :Run initial linprog;

  :Extract indivisible values;
  :Convert to portion counts;
  :Apply minimum threshold (0.5);
  :Round to integer portions;
  :Convert to weights;

  :Update bounds (fix rounded values);
  :Run second linprog;
endif

:Store final solution;
:update_flag = False;

stop
@enduml
If you want it even more minimal (presentation-ready), I can compress it into ~6 blocks.


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

        return json_ingredient_weights, json_total_macros no title, size 18 text

@startuml
skinparam defaultFontSize 18

start

:Initialize optimizer;
note right
Load:
- settings
- input_list
- is_indivisible
- user_designated_values
- n
- n_in
end note

:Create objective vector c;
note right
Ingredient variables -> 0
Slack variables -> slack weights
Excess variables -> excess weights
end note

:Create equality matrix A_eq;
note right
Build food-property matrix
and expand it as:

[A_food | I | -I]
end note

:Create bounds;
note right
For each ingredient:
- fixed value -> (val, val)
- priority 1 -> (0.1, ∞)
- otherwise -> (0.1, 2)

Deviation variables:
(0, ∞)
end note

if (Any indivisible ingredient?) then (no)
  :Run linprog once;
  :Store solution;
  :Set update_flag = False;
else (yes)
  :Run initial linprog;
  :Extract ingredient solution;

  :Keep only indivisible values;
  :Convert to portion counts;
  :Apply minimum threshold 0.5;
  :Round to integer portions;
  :Convert rounded portions to weights;

  :Update bounds;
  note right
  Nonzero rounded indivisible values
  are fixed in filtered bounds
  end note

  :Run second linprog;
  :Store solution;
  :Set update_flag = False;
endif

stop
@enduml


