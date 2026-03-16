## Flow 1 – User Registration, Login and Goal Setup

**Name:** User registration and initial setup
**ID:** UC 01
**Description:** The goal is to create a new user account, successfully log in, configure nutrition goals, and reach the application homepage.
**Primary actor:** User
**Secondary actor:** System
**Input requirements:** User opens the application and is not logged in.
**Output requirements:** User account is created, the user is logged in, and nutrition goals are stored.

### Scenario

| No. | Actor  | Action                                          |
| --- | ------ | ----------------------------------------------- |
| 1.  | User   | Opens the registration page.                    |
| 2.  | User   | Enters email and password.                      |
| 3.  | User   | Submits the registration form.                  |
| 4.  | System | Validates the entered data.                     |
| 5.  | System | Creates a new user account.                     |
| 6.  | User   | Opens the login page.                           |
| 7.  | User   | Enters email and password and submits the form. |
| 8.  | System | Authenticates the user.                         |
| 9.  | User   | Opens nutrition goal settings.                  |
| 10. | User   | Selects goal configuration (smart or direct).   |
| 11. | System | Stores the nutrition goals.                     |
| 12. | System | Redirects the user to the homepage.             |

### Alternative Flows

| No.  | Actor  | Action                                                                                             |
| ---- | ------ | -------------------------------------------------------------------------------------------------- |
| 4a.  | System | If registration data is invalid, the system asks the user to correct the input.                    |
| 8a.  | System | If login credentials are incorrect, the system displays an error and allows another login attempt. |
| 10a. | User   | If the user selects **smart goals**, the system generates recommended nutrition goals.             |
| 10b. | User   | If the user selects **direct goals**, the user manually enters goal values.                        |

### Error Messages

| No. | Message                                                                 |
| --- | ----------------------------------------------------------------------- |
| 1.  | Invalid registration data. Please check the entered email and password. |
| 2.  | Login failed. Incorrect email or password.                              |

---

# Flow 2 – Meal Log Creation and Optimization

**Name:** Create meal log and perform optimization
**ID:** UC 02
**Description:** The user creates a meal log for a selected date, adds ingredients with quantity rules, sets nutrition goals, and sends the meal for optimization to receive optimized ingredient quantities.
**Primary actor:** User
**Secondary actor:** System
**Input requirements:** User is logged in and has access to the meal logging interface.
**Output requirements:** A meal log is created and optimized ingredient quantities are displayed.

### Scenario

| No. | Actor  | Action                                                                         |
| --- | ------ | ------------------------------------------------------------------------------ |
| 1.  | User   | Selects a date.                                                                |
| 2.  | User   | Clicks the button to create a meal log.                                        |
| 3.  | System | Creates a new meal entry for the selected date.                                |
| 4.  | User   | Adds ingredients to the meal.                                                  |
| 5.  | User   | For each ingredient selects whether it is defined in pieces or exact quantity. |
| 6.  | System | Stores ingredients and quantity rules.                                         |
| 7.  | User   | Reviews or updates nutrition goals.                                            |
| 8.  | User   | Sends the meal for optimization.                                               |
| 9.  | System | Performs optimization based on ingredients and goals.                          |
| 10. | System | Displays the optimized ingredient quantities.                                  |

### Alternative Flows

| No.  | Actor | Action                                                                                            |
| ---- | ----- | ------------------------------------------------------------------------------------------------- |
| 4a.  | User  | If the user wants to add more ingredients, they repeat step 4.                                    |
| 5a.  | User  | If the ingredient is piece-based, the user enters number of pieces.                               |
| 5b.  | User  | If the ingredient requires an exact amount, the user enters weight or quantity.                   |
| 10a. | User  | If the user wants to adjust the result, they modify ingredients or goals and repeat optimization. |

### Error Messages

| No. | Message                                                  |
| --- | -------------------------------------------------------- |
| 1.  | Optimization failed. Please adjust ingredients or goals. |
| 2.  | Invalid ingredient quantity entered.                     |

---

# Flow 3 – Admin Login and User Management

**Name:** Administrator login and user update
**ID:** UC 03
**Description:** The administrator logs into the system, searches for a user account, modifies user data, and saves the changes.
**Primary actor:** Admin
**Secondary actor:** System
**Input requirements:** Administrator has valid login credentials.
**Output requirements:** Selected user data is successfully updated and stored in the system.

### Scenario

| No. | Actor  | Action                                   |
| --- | ------ | ---------------------------------------- |
| 1.  | Admin  | Opens the admin login page.              |
| 2.  | Admin  | Enters email and password.               |
| 3.  | Admin  | Submits the login form.                  |
| 4.  | System | Verifies administrator credentials.      |
| 5.  | System | Displays the administration dashboard.   |
| 6.  | Admin  | Searches for a user.                     |
| 7.  | System | Displays matching user records.          |
| 8.  | Admin  | Selects a user account.                  |
| 9.  | Admin  | Modifies user information.               |
| 10. | Admin  | Saves the updated data.                  |
| 11. | System | Stores the changes and confirms success. |

### Alternative Flows

| No. | Actor  | Action                                                            |
| --- | ------ | ----------------------------------------------------------------- |
| 7a. | System | If no user is found, the system informs the administrator.        |
| 9a. | Admin  | If the admin wants to modify another user, they return to step 6. |

### Error Messages

| No. | Message                                    |
| --- | ------------------------------------------ |
| 1.  | Admin login failed. Incorrect credentials. |
| 2.  | User not found.                            |
| 3.  | Failed to save changes. Please try again.  |

---

