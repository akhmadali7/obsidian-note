
**How to Build Utilities for Handling POST and PATCH Requests in RESTful APIs 

When building RESTful APIs, one of the most important things to think about is ensuring the data in incoming requests is valid. But if you're not using an ORM, you’ll often find yourself manually validating that data. Handling `POST` and `PATCH` requests is especially tricky since they serve different purposes: `POST` is used to **create new resources**, while `PATCH` is used to **partially update** existing ones.

In this post, I'll walk through how I built some utility functions that help with validating data for `POST` and `PATCH` requests. These functions make it easier to ensure users are sending the right data, and they save time by reusing logic for validation in different parts of the application.

### **POST vs PATCH: A Quick Recap**

Before diving into the utilities, let me quickly explain the difference between `POST` and `PATCH`:

- **`POST`**: This is used to create new resources. It requires the user to send **all required fields** necessary to create the resource.
    
- **`PATCH`**: This method updates an existing resource, but only the fields that the user wants to change. If no fields are sent, the update is considered invalid.
    

Understanding this difference is essential because `POST` typically requires all fields, while `PATCH` requires only specific fields to update.

### **Why Different Utilities for POST and PATCH?**

In my experience, when handling requests, it’s critical to validate that the **correct** fields are present. The issue with `POST` and `PATCH` is that their validation needs are different:

- **For `POST` requests**, you need to ensure that **all required fields** are present to create a new resource.
    
- **For `PATCH` requests**, the user might only send a subset of fields. The request is valid as long as it includes at least one updatable field.
    

Having separate utilities for each makes things a lot easier and cleaner, and I can reuse them across different routes. Here’s why I split them up:

- **`POST`**: We check for all fields because creating something new requires full data.
    
- **`PATCH`**: Here, we just need to validate that at least one field is included, or else the update doesn’t make sense.
    

### **Step 1: Ensure the Request Body Isn't Empty**

The first step in validation is making sure that the request body isn’t empty or missing. It’s easy to forget to send data in a `POST` or `PATCH` request, and you don't want your app to fail silently.

Here's how I built the utility to check for missing data:

```js
// validateReqBodyExistence.js
/**  
 * Checks if the request body is missing or empty.  
 * @param acceptableFields  
 * @param {Object} req - Express request object  
 * @param {NextFunction} next - Express next function to pass errors  
 */  

function validateReqBodyExistence(acceptableFields, req, next) {  
    if (!req.body || Object.keys(req.body).length === 0) {  
        const error = new Error(`Request body is missing or empty. Acceptable fields are: ${acceptableFields.join(', ')}`);  
        error.statusCode = 400;  
        error.code = "MISSING_BODY";  
        next(error);  // Pass the error to the next handler  
        return true;  
    }  
    return false;  
}  

export default validateReqBodyExistence;
```

#### **How to Use It in Your Controller**

I use this utility to check if the body is missing or empty:

```js
const acceptableFields = ['email', 'password'];
if (validateReqBodyExistence(acceptableFields, req, next)) {  
    return;  // Stop if the body is empty  
}
```

This prevents errors further down the line by making sure the user provided something in the body.

### **Step 2: Validate Required Fields for POST Requests**

For `POST` requests, the user needs to provide all the necessary fields to create a new resource. If any required field is missing, I reject the request. Here’s the utility I wrote to check that:

```js
//validateRequiredFields.js
/**  
 * Ensures all required fields are in the request body  
 * @param {Array} requiredFields - List of required fields  
 * @param {Object} body - Request body  
 * @param {NextFunction} next - Express next function for error handling  
 * @returns {boolean} - Returns true if missing fields, otherwise false  
 */  

function validateRequiredFields(requiredFields, body, next) {  
    const missingFields = [];  
    for (let field of requiredFields) {  
        if (!body[field]) {  
            missingFields.push(field);  
        }  
    }  

    if (missingFields.length > 0) {  
        const error = new Error(`Missing required fields: ${missingFields.join(', ')}`);  
        error.statusCode = 400;  
        error.code = "MISSING_FIELD";  
        return next(error);  
    }  
    return false;  
}  

export default validateRequiredFields;
```

#### **How to Use It in Your Controller**

I use it like this in the controller:

```js
const requiredFields = ['email', 'password'];
const body = req.body;

if (validateRequiredFields(requiredFields, body, next)) {  
    return;  // Stop if any fields are missing  
}
```

This helps ensure that the `POST` request is valid before trying to create a new resource.

### **Step 3: Handle Partial Updates for PATCH Requests**

`PATCH` requests are different because the user doesn’t need to provide all fields. They can update only the fields they want. The tricky part is ensuring at least one valid field is provided, so here's a utility to handle that:

```js
/**  
 * Ensures at least one valid updatable field is in the request body  
 * @param {Array} updatableFields - List of fields allowed to update  
 * @param {Object} body - Request body  
 * @param {NextFunction} next - Express next function for error handling  
 */  

function validateUpdateAbleFields(updatableFields, body, next) {  
    const providedFields = Object.keys(body);  
    const validFields = providedFields.filter(field => updatableFields.includes(field));  

    if (validFields.length === 0) {  
        const error = new Error(`At least one field must be provided from: ${updatableFields.join(', ')}`);  
        error.statusCode = 400;  
        error.code = "MISSING_FIELD";  
        next(error);  
        return true;  
    }  

    return false;  
}  

export default validateUpdateAbleFields;
```

#### **How to Use It in Your Controller**

You can use it like this in your controller:

```js
const updatableFields = ['email', 'username', 'profile_picture'];
const body = req.body;

if (validateUpdateAbleFields(updatableFields, body, next)) {  
    return;  // Stop if no valid fields are provided  
}
```

This makes sure the `PATCH` request is valid before proceeding with the update.

### **Step 4: Add Zod Validation (If You're Not Using ORM)**

For further validation, I’ve been using **Zod**—a TypeScript-first schema validation library. It’s super useful for validating things like email formats, passwords, and even URLs.

Here’s how I use Zod to validate data:

```js
import { z } from 'zod';

// Define the schema for the fields
const userFieldSchemas = {
  email: z.string().email("Invalid email format"),
  password: z.string().min(8, "Password must be at least 8 characters"),
  username: z.string().min(3, "Username must be at least 3 characters"),
  profile_picture: z.string().url("Profile picture must be a valid URL")
};
```

#### **POST Schema (For Creating a New User)**

```js
export const loginSchema = z.object({
  email: userFieldSchemas.email,
  password: userFieldSchemas.password
});
```

#### **PATCH Schema (For Updating a User)**

```js
export const updateUserSchema = z.object({
  email: userFieldSchemas.email.optional(),
  password: userFieldSchemas.password.optional(), 
  username: userFieldSchemas.username.optional(), 
  profile_picture: userFieldSchemas.profile_picture.optional()
});
```

Using Zod like this allows me to set clear expectations for the shape of the data. It also reduces the need for manual checks.

! ADD the function and how to use it in controller

### **Why Not PUT?**

`PUT` is used to replace an existing resource entirely, which means it requires **all fields**. Since it shares a lot of validation characteristics with `POST`, I usually handle `PUT` requests in a similar way to `POST` requests, reusing most of the validation functions.

### **Conclusion**

In this post, I’ve shared how I built utilities for handling `POST` and `PATCH` requests in RESTful APIs. By creating reusable validation functions, I’m able to keep my code clean, reduce repetition, and ensure incoming data is valid before processing it. Tools like Zod make this process easier and more robust, especially when handling more complex validation.

I hope these examples help you get started with your own API validation! Feel free to experiment with these utilities and adjust them for your specific needs. As always, don’t forget to test everything to make sure it works as expected!
