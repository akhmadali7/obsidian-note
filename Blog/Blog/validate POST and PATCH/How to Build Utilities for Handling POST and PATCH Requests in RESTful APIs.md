
So, I’ve been diving into building RESTful APIs, and one thing I’m learning is how important it is to understand the difference between `POST` and `PATCH` methods. From what I gather, `POST` is for **creating new resources**, while `PATCH` is for **updating existing ones**. But here’s the tricky part—how you handle things like form validation and input data is different for each.

In this post, I’m going to walk through how I’m learning to create some **utility functions** that can help make the process of validating data in `POST` and `PATCH` requests a little easier, so that users send the right info for each case.

### POST vs PATCH: A Quick Recap

Before I get too deep into the utilities, let me quickly refresh what I’ve picked up about these two HTTP methods:

- **`POST`**: Used for creating a new resource. The user needs to provide **all the required fields** to make that happen.
    
- **`PATCH`**: This is used to **partially update** an existing resource. The user only needs to send the fields they want to change, not everything.
    

### Why Do I Need Different Utilities for POST and PATCH?

From what I’ve learned, when you’re handling user input, it’s important to make sure users send **all required fields** when creating something with a `POST` request. But with `PATCH`, they only need to send the ones they want to update.

Creating separate utilities for each request type can save a lot of time, reduce repeating code, and make sure the right kind of validation is happening for each one.

### STEP 1: Create a Utility to Ensure Required Fields Are in the Request Body

So, why add this utility? Think about it: when a user is interacting with your API, they might not always know exactly what should be included in the request body (especially if they didn’t bother reading the API documentation). This utility ensures that when they forget to add something, they’ll get a helpful message letting them know exactly what fields are expected.

Here’s the code for that utility:

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

#### How to Use This in Your Controller

Once you've got the utility set up, you can easily use it inside your controller to check whether the request body has the expected fields. Here's how you might do that:

```js
const acceptableFields = ['email', 'password'];
if (validateReqBodyExistence(acceptableFields, req, next)) {  
    return;  // Early exit if the body is invalid, prevents further processing  
}
```

#### Why This Helps

The main benefit of this utility is that it’s a simple way to ensure the user has provided the right information. Instead of silently failing or sending a vague error, you’re proactively telling the user what they missed or did wrong. Plus, you’re making your code more reusable and cleaner, rather than repeating validation logic in multiple places.


### STEP 2: Create a Function to Validate Required Fields (for POST Requests)

Now that we’ve ensured the request body exists, the next step is to check if all the required fields are actually present. This is crucial for `POST` requests where we need the user to provide specific fields for creating a new resource.

If a required field is missing, the function will notify the user by passing an error with the details of what’s missing.

Here’s how you can set up the function to validate the required fields:

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

#### How to Use It in Your Controller

Once you've set up the `validateRequiredFields` function, you can use it to check if the request body includes all the necessary fields for a `POST` request:

```js
const requiredFields = ['email', 'password'];
const body = req.body;

if (validateRequiredFields(requiredFields, body, next)) {  
    return;  // Early exit if any fields are missing, prevents further processing  
}
```

#### Why This Helps

When users submit data for a `POST` request, you want to make sure they provide **all** the required fields. If anything’s missing, this function will catch it early and notify the user with a clear error message. It also keeps your code clean by centralizing the validation logic instead of duplicating it in multiple places. This way, you ensure that your API requests are always valid and prevent unnecessary errors downstream.

### STEP 3: Create a Function to Handle Partial Updates (for PATCH Requests)

In a `PATCH` request, we don’t expect users to provide all the fields—just the ones they want to update. The challenge here is ensuring that at least **one** of the updatable fields is included in the request body. This way, we can avoid scenarios where users send an empty body or don’t include any valid fields for update.

Here’s the function that checks if at least one of the allowed updatable fields is provided:

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

#### How to Use It in Your Controller

Once the `validateUpdateAbleFields` function is ready, you can use it in your controller to check if the user has included at least one field that can be updated:

```js
const updateAbleFields = ['email', 'username', 'fullname'];
const body = req.body;

if (validateUpdateAbleFields(updateAbleFields, body, next)) {  
    return;  // Stop execution if no valid fields are provided  
}
```

#### Why This Makes Sense

In a `PATCH` request, users are only updating specific fields. But you still want to make sure they don’t send an empty request body or update fields that aren't allowed. This function ensures that at least one of the updatable fields is included, and if not, it sends a clear error message. It makes your API more predictable and prevents unnecessary processing of invalid requests, saving time for both you and the user.

### STEP 4: Adding Zod Validation (If You're Not Using ORM)

Now that we’ve verified the request body and checked for the required fields, the next step is to **validate the values** of those fields. For this, we'll use **Zod**, a TypeScript-first schema declaration and validation library.

Zod allows us to define a schema for our data and validate it easily. Here, we’ll define the schemas for both `POST` (creating new resources) and `PATCH` (updating existing ones), ensuring that the data adheres to the expected format and business rules.

Let’s break down the code:

### Defining the Field Schemas

First, we'll define a schema for each field we want to validate. For example, we might validate an email, password, username, and profile picture URL.

```js
import { z } from 'zod';

// Define field schemas
const userFieldSchemas = {
  email: z.string().email("Please enter a valid email address"), // Email validation
  password: z.string().min(8, "Password must be at least 8 characters"), // Password must be at least 8 characters
  username: z.string().min(3, "Username must be at least 3 characters"), // Username must be at least 3 characters
  profile_picture: z.string().url("Profile picture must be a valid URL") // URL validation for profile picture
};
```

### POST Schema (For Creating a New User)

For a `POST` request, you usually want to validate the creation of a new resource, like a user. In this case, we require both the **email** and **password** fields to be present.

```js
// Login schema - requires both email and password
export const loginSchema = z.object({
  email: userFieldSchemas.email,  // Validate email
  password: userFieldSchemas.password // Validate password
});
```

This schema will ensure that both fields are present and follow the validation rules we’ve defined (valid email, valid password length).

### PATCH Schema (For Updating a User)

For `PATCH` requests, users are only required to send the fields they want to update. We can make each field optional and add a refinement to ensure at least one field is provided for an update.

```js
// Update user schema - requires at least one field
export const updateUserSchema = z.object({
  email: userFieldSchemas.email.optional(),
  password: userFieldSchemas.password.optional(),
  username: userFieldSchemas.username.optional(),
  profile_picture: userFieldSchemas.profile_picture.optional()
}).refine(
  data => Object.keys(data).length > 0, // Ensures at least one field is provided
  { message: "At least one field must be provided for update" }
);
```

### Validation Functions

Now, let's define the validation functions that will use the Zod schemas to validate the incoming data. We’ll use `safeParse`, which will return a result indicating whether the validation was successful or not.

```js
// Validation functions
export const validateLoginForm = (data) => {
  const result = loginSchema.safeParse(data);
  return processResult(result);
};

export const validateUpdateUserForm = (data) => {
  const result = updateUserSchema.safeParse(data);
  return processResult(result);
};
```

### Helper Function to Process Results

Zod’s `safeParse` function returns an object that indicates whether validation was successful (`success: true/false`). If the validation fails, we’ll map the errors to a more readable format using a helper function. This function formats the error messages and returns a consistent result format.

```js
// Helper function to format validation results
const processResult = (result) => {
  if (!result.success) {
    return {
      success: false,
      errors: result.error.errors.map(err => {
        // Handle refinement errors (which don't have a path)
        if (err.path.length === 0) {
          return err.message; // If no path, just return the error message
        }
        return `${err.path.join('.')}: ${err.message}`; // Otherwise, show field path with message
      }),
    };
  }
  
  return {
    success: true,
    data: result.data,
    errors: [],
  };
};
```

### Full Code Example

Here’s the full implementation put together: (add copy toggle)

```js
import { z } from 'zod';

// Define field schemas
const userFieldSchemas = {
  email: z.string().email("Please enter a valid email address"),
  password: z.string().min(8, "Password must be at least 8 characters"),
  username: z.string().min(3, "Username must be at least 3 characters"),
  profile_picture: z.string().url("Profile picture must be a valid URL")
};

// Login schema - requires both email and password
export const loginSchema = z.object({
  email: userFieldSchemas.email,
  password: userFieldSchemas.password
});

// Update user schema - requires at least one field
export const updateUserSchema = z.object({
  email: userFieldSchemas.email.optional(),
  password: userFieldSchemas.password.optional(),
  username: userFieldSchemas.username.optional(),
  profile_picture: userFieldSchemas.profile_picture.optional()
}).refine(
  data => Object.keys(data).length > 0,
  { message: "At least one field must be provided for update" }
);

// Validation functions
export const validateLoginForm = (data) => {
  const result = loginSchema.safeParse(data);
  return processResult(result);
};

export const validateUpdateUserForm = (data) => {
  const result = updateUserSchema.safeParse(data);
  return processResult(result);
};

// Helper function to format validation results
const processResult = (result) => {
  if (!result.success) {
    return {
      success: false,
      errors: result.error.errors.map(err => {
        if (err.path.length === 0) {
          return err.message;
        }
        return `${err.path.join('.')}: ${err.message}`;
      }),
    };
  }
  
  return {
    success: true,
    data: result.data,
    errors: [],
  };
};
```

### Why This Works

Using Zod to validate your input ensures that the data adheres to strict formats. With schemas defined for both `POST` and `PATCH` requests, you’re guaranteeing that the input follows business logic (valid email, password length, etc.).

The beauty of Zod here is that it makes the validation process easier to maintain and read, and you get detailed, human-readable error messages if something goes wrong.

### Why Don't We Discuss `PUT`?

While we've focused on `POST` and `PATCH`, you may have noticed that we haven't specifically discussed the `PUT` method. The reason for this is simple: **`PUT` behaves similarly to `POST`** when it comes to handling required fields.

- **`POST`**: Used to create new resources. Requires **all required fields** to create the resource.
    
- **`PUT`**: Used to **replace** an existing resource. Like `POST`, it typically requires **all fields** to fully define the resource. (The key difference is that `PUT` involves validation to check whether the data to be updated exists or not.)
    

Since `PUT` often requires **the entire resource** to be sent (just like `POST`), it doesn't introduce any unique utility or validation logic compared to `POST`. The main distinction between the two is the action performed: creating a new resource with `POST` vs. replacing an existing one with `PUT`. However, in terms of validation, both methods require complete data for the resource.

### Conclusion

When building APIs, it’s essential to provide clear and structured validation logic to handle both `POST` and `PATCH` methods effectively. By creating utilities like the ones we've discussed, you can ensure that users are submitting the right data for each type of request — requiring **all fields** for a `POST` request and **only the fields that need to be updated** for a `PATCH` request.

Using a library like **Zod** further enhances this process by allowing you to define flexible, reusable validation schemas that make it easier to manage and scale your API as it grows. With Zod, you're able to enforce strict validation, ensuring your data meets the required formats and rules without redundant code. It simplifies error handling, gives clear feedback, and ultimately helps keep your validation logic organized and maintainable.

These utilities, combined with Zod, not only reduce code duplication but also improve the consistency and scalability of your API, so you can spend more time building new features and less time worrying about data integrity.
