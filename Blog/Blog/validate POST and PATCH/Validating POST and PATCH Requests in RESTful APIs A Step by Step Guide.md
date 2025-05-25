
# How to Build Utilities for Handling POST and PATCH Requests in RESTful APIs (if you are not using ORM)

When building RESTful APIs, one of the most important tasks is ensuring that the data received in requests is valid. Handling `POST` and `PATCH` requests requires specific validation strategies, as each method serves different purposes: `POST` is used to **create new resources**, while `PATCH` is used to **partially update existing ones**.

In this post, I'll walk you through creating utility functions to make validating data in these requests easier, ensuring that users send the right information depending on whether they're creating or updating a resource. By the end, you'll have reusable validation utilities that streamline your API's behavior, reduce code duplication, and improve maintainability.

### **POST vs PATCH: A Quick Recap**

Before diving into the utilities, let's quickly refresh the difference between `POST` and `PATCH`:

- **`POST`**: This method is used for creating new resources. It requires the user to provide **all required fields** necessary to create the resource.
    
- **`PATCH`**: This method is used for partially updating an existing resource. Only the fields the user wants to update need to be sent. If no fields are provided, the update request is invalid.
    

Understanding the difference is key to implementing appropriate validation logic, as `POST` requires all fields, whereas `PATCH` only needs specific updates.

### **Why Do I Need Different Utilities for POST and PATCH?**

When handling user input, it’s important to ensure that users provide **all required fields** for `POST` requests. However, with `PATCH`, the user only needs to provide the fields they want to modify.

Creating separate utilities for each request type saves time, reduces repetitive code, and ensures the correct validation logic is applied. Here’s why we need them:

- **For `POST` requests**: We need to ensure the user includes all fields to create a new resource.
    
- **For `PATCH` requests**: We need to make sure that at least one valid field is provided for updating the resource.
    

### **Step 1: Create a Utility to Ensure Required Fields Are in the Request Body**

The first step is to ensure the request body isn't empty or missing. If a user submits an empty body or forgets to include required fields, it can lead to confusing errors down the line. This utility checks if the required fields are present in the request body and gives a helpful error message if they’re missing.

Here’s how you can implement it:

```js
/**  
 * Check if the request body is missing or empty.  
 * @param acceptableFields  
 * @param {Object} req - The Express request object.  
 * @param {NextFunction} next - The Express next function to pass errors.  
 */  
  
function validateReqBodyExistence(acceptableFields, req, next) {  
    if (!req.body || Object.keys(req.body).length === 0) {  
        const error = new Error(`Request body is missing or empty. Acceptable fields are: ${acceptableFields.join(', ')}`);  
        error.statusCode = 400;  
        error.code = "MISSING_BODY";  
        next(error);  // Pass the error to the error handler  
        return true;  // We handled the error, so exit early  
    }  
  
    return false;  // No issues with the body (optional, just keeps things clean)  
}  
  
export default validateReqBodyExistence;
```

#### **How to Use This in Your Controller**

In your controller, use this utility to check if the body has the expected fields:

```js
const acceptableFields = ['email', 'password'];
if (validateReqBodyExistence(acceptableFields, req, next)) {  
    return;  // Early exit if the body is invalid, prevents further processing  
}
```

This ensures that the user has provided the right data, helping to avoid vague or incorrect error responses.

### **Step 2: Create a Function to Validate Required Fields (for POST Requests)**

After confirming that the body exists, the next step is to ensure all **required fields** for a `POST` request are present. For creating a new resource, you typically have a set of required fields, and if any of them are missing, the request should be rejected.

Here’s the utility for validating the required fields:

```js
/**  
 * Checks if all required fields are present in the request body.  
 * @param {Array} requiredFields - List of required fields.  
 * @param {Object} body - Request body.  
 * @param {NextFunction} next - Express next function for error handling.  
 * @returns {boolean} - Returns true if missing fields, else false.  
 */  

function validateRequiredFields(requiredFields, body, next) {  
    const missingFields = [];  
    for (let field of requiredFields) {  
        if (!body[field]) {  
            missingFields.push(field);  
        }  
    }  
  
    if (missingFields.length > 0) {  
        // Pass an error to the next middleware with status 400 (Bad Request)  
        const error = new Error(`Missing required fields: ${missingFields.join(', ')}`);  
        error.statusCode = 400;  
        error.code = "MISSING_FIELD";  
        return next(error);  // Pass the error to the error handler  
    }  
  
    return false;  // No missing fields  
}  
  
export default validateRequiredFields;
```

#### **How to Use It in Your Controller**

In the controller, you can check if the body includes all the required fields for a `POST` request:

```js
const requiredFields = ['email', 'password'];
const body = req.body;

if (validateRequiredFields(requiredFields, body, next)) {  
    return;  // Early exit if any fields are missing, prevents further processing  
}
```

This utility helps ensure that your `POST` requests are valid before processing them further.

### **Step 3: Create a Function to Handle Partial Updates (for PATCH Requests)**

For `PATCH` requests, users only need to provide the fields they want to update. This function ensures that at least one valid updatable field is included in the request.

Here's the utility that validates partial updates:

```js
/**  
 * Checks if at least one updatable field is present in the request body.  
 * @param {Array} updatableFields - List of allowed fields for update.  
 * @param {Object} body - Request body.  
 * @param {NextFunction} next - Express next function for error handling.  
 */  

function validateUpdateAbleFields(updatableFields, body, next) {  
    const providedFields = Object.keys(body);  
    const validFields = providedFields.filter(field => updatableFields.includes(field));  
  
    if (validFields.length === 0) {  
        const error = new Error(`At least one of the following fields must be provided: ${updatableFields.join(', ')}`);  
        error.statusCode = 400;  // Bad Request  
        error.code = "MISSING_FIELD";  
        next(error);  // Pass the error to the error handler  
        return true;  // We handled the error, so exit early  
    }  
  
    return false;  // No issues with the body (not really needed, just keeps things clean)  
}  
  
export default validateUpdateAbleFields;
```

#### **How to Use It in Your Controller**

In your controller, check if the user has included at least one valid field:

```js
const updateAbleFields = ['email', 'username', 'fullname'];
const body = req.body;

if (validateUpdateAbleFields(updateAbleFields, body, next)) {  
    return;  // Stop execution if no valid fields are provided  
}
```

This ensures that the `PATCH` request has at least one field to update and prevents empty or invalid updates.

### **Step 4: Adding Zod Validation (If You're Not Using ORM)**

For further validation, especially when handling complex data types or ensuring specific patterns, **Zod** can be a great addition. Zod is a TypeScript-first schema declaration and validation library that makes it easy to define validation rules for your data.

Using Zod, we can validate that fields like `email`, `password`, or `username` meet specific criteria (e.g., valid email format, password length).

Here’s how to integrate Zod validation:

```js
import { z } from 'zod';

// Define field schemas
const userFieldSchemas = {
  email: z.string().email("Please enter a valid email address"),
  password: z.string().min(8, "Password must be at least 8 characters"),
  username: z.string().min(3, "Username must be at least 3 characters"),
  profile_picture: z.string().url("Profile picture must be a valid URL")
};
```

#### **POST Schema (For Creating a New User)**

For `POST` requests, we validate that both **email** and **password** are provided and meet the defined criteria:

```js
// Login schema - requires both email and password
export const loginSchema = z.object({
  email: userFieldSchemas.email,
  password: userFieldSchemas.password
});
```

#### **PATCH Schema (For Updating a User)**

For `PATCH` requests, users can update only the fields they want. We validate that at least one field is provided and that the values are valid:

```js
// Update user schema - requires at least one field
export const updateUserSchema = z.object({
  email: userFieldSchemas.email.optional(),
  password: userFieldSchemas.password.optional(), 
  username: userFieldSchemas.username.optional(), 
  profile_picture: userFieldSchemas.profile_picture.optional() });
```


Using Zod here adds robust validation, ensures the integrity of the data, and helps us avoid manual checks for specific formats.

### **Why Don’t We Discuss PUT?**

A `PUT` request, by definition, is used to **replace** a resource entirely, requiring all fields to be provided. Since it shares many characteristics with `POST` (especially in terms of required fields), it can often be handled by reusing the `POST` validation utilities. This blog focuses on `POST` and `PATCH` because they represent the more distinct validation scenarios.

### **Conclusion**

In this guide, we've walked through how to build utilities to validate `POST` and `PATCH` requests. By separating validation logic into reusable utilities, you can ensure that your APIs handle incoming data correctly, improving reliability and maintainability.

Additionally, tools like Zod can enhance validation capabilities, especially in scenarios where complex data structures are involved.

By using these techniques, you can focus more on building features and less on boilerplate validation code. Start applying these principles in your API and see how they simplify error handling and ensure your application behaves as expected.