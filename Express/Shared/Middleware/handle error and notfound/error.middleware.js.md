```js
/**  
 * Middleware function to handle 404 errors when a route is not found. */const notFound = (req, res, next) => {  
    const error = new Error(`Not found - ${req.originalUrl}`) // Create error with the original URL  
    res.status(400) // Set status code to 400 (Bad Request)  
    next(error) // Pass the error to the next error-handling middleware  
}  
  
/**  
 * Global error handler for catching and processing errors. */const errorHandler = (err, req, res, next) => {  
    let statusCode = res.statusCode === 200 ? 500 : res.statusCode // If status code is 200, change to 500 (Internal Server Error)  
    let message = err.message // Default message from the error  
  
    // Mongoose error handling: Specifically for invalid ObjectId errors    if (err.name === 'CastError' && err.kind === 'ObjectId') {  
        statusCode = 404 // Change status code to 404 for resource not found  
        message = 'Resource not found' // Customize the error message  
    }  
  
    // Respond with status code and a JSON response containing the message and stack trace  
    res.status(statusCode).json({  
        message: message, // Send the error message  
        // Conditionally include stack trace based on environment        // stack: process.env.NODE_ENV === "production" ? null : err.stack    })  
}  
  
export {  
    notFound,  
    errorHandler  
}
```


### 1. **notFound Middleware:**

This middleware function handles situations where a route is not found (i.e., the URL requested by the user doesn't match any of the defined routes in the application).

- **Purpose:** It generates a 404 error when no other route matches the request.
    
- **How it works:**
    
    - The function creates a new `Error` object, passing a custom message that includes the original URL the user tried to access (`req.originalUrl`).
        
    - The response status is set to `400` (Bad Request), although typically for "not found" errors, it would be more appropriate to use `404`. (This might be a mistake or choice based on the application's specific needs.)
        
    - The error is then passed to the next middleware (`next(error)`), which should be an error-handling middleware (like the `errorHandler`).
        

### 2. **errorHandler Middleware:**

This is a global error handler for catching and processing errors that occur during the execution of requests.

- **Purpose:** It processes any error and formats the response with the appropriate status code and error message. It can also handle specific types of errors, like those coming from the database (e.g., Mongoose).
    
- **How it works:**
    
    - First, the middleware checks the `statusCode`. If the current status is `200` (success), it changes the code to `500` (Internal Server Error). This is a safeguard to ensure that the server responds with an error code in case something went wrong during the request.
        
    - It grabs the error message from the error object (`err.message`).
        
    - If the error is a `CastError` (which is common in Mongoose when an invalid ObjectId is provided), it changes the status code to `404` and sets a custom message: "Resource not found".
        
    - Finally, it sends the error details back to the client as a JSON object, including the error message. Optionally, it could include the error stack trace, but it's commented out, presumably because, in a production environment, stack traces are often hidden for security reasons.
        

### Exporting the Functions:

Both `notFound` and `errorHandler` are exported so that they can be used in other parts of the application, typically in the main app or router files.

### Summary:

- **notFound:** Handles cases where a route is not found and creates an error with a message indicating that.
    
- **errorHandler:** Catches and handles errors, provides status codes, and formats error messages to respond with appropriate details based on the error type.
    

In general, this pattern is commonly used in Express applications to handle errors gracefully and ensure that users get appropriate error responses when something goes wrong.


## How to use it?
To use the `notFound` and `errorHandler` middleware functions in an Express application, you need to integrate them into your route handling and error handling flow. Here's how to do it:

### Step-by-Step Integration:

1. **Set up your Express app**: You likely already have an Express app. Here's a simple structure:
    
    ```javascript
    import express from 'express';
    import { notFound, errorHandler } from './middlewares';  // Adjust the import path if needed
    
    const app = express();
    
    // Your regular routes go here
    
    app.get('/', (req, res) => {
        res.send('Hello, world!');
    });
    
    // Place your other routes here
    ```
    
2. **Use the `notFound` middleware**: The `notFound` middleware should be placed **after** all of your route handlers. This ensures that if no route matches the request, it triggers the `notFound` middleware to handle the 404 error.
    
    ```javascript
    // Use the notFound middleware at the end
    app.use(notFound);
    ```
    
    This will catch any requests that do not match any routes defined above.
    
3. **Use the `errorHandler` middleware**: The `errorHandler` middleware should also be placed at the very end, **after** all your routes and the `notFound` middleware. It will catch any errors passed via `next(error)` from previous middleware or route handlers, as well as the errors created by the `notFound` middleware.
    
    ```javascript
    // Use the errorHandler middleware last
    app.use(errorHandler);
    ```
    
    It ensures that any error that gets passed into `next()` will be processed, and the appropriate response is sent to the client.
    
4. **Complete example**:
    
    Here’s how everything comes together in an Express app:
    
    ```javascript
    import express from 'express';
    import { notFound, errorHandler } from './middlewares';  // Adjust the import path
    
    const app = express();
    
    // Sample route
    app.get('/', (req, res) => {
        res.send('Hello, world!');
    });
    
    // Other routes can go here
    // app.get('/some-route', (req, res) => { ... });
    
    // Use the notFound middleware (this handles 404 errors)
    app.use(notFound);
    
    // Use the errorHandler middleware (this catches and handles all errors)
    app.use(errorHandler);
    
    // Set the server to listen on a port
    app.listen(3000, () => {
        console.log('Server is running on port 3000');
    });
    ```
    
5. **How errors are handled**:
    
    - If a user visits an undefined route (for example, `/unknown`), the `notFound` middleware will create a 404 error with a message like `"Not found - /unknown"`.
        
    - If any error occurs in your route handlers (e.g., database errors or internal server issues), you can pass the error to `next(error)`, and the `errorHandler` middleware will handle it and send a proper error response to the client.
        
    
    For example, if a route encounters an error:
    
    ```javascript
    app.get('/error', (req, res, next) => {
        const error = new Error('Something went wrong');
        next(error);  // This will trigger the errorHandler
    });
    ```
    

### **Order of Middleware**:

- **Routes**: First, define all your route handlers.
    
- **notFound middleware**: After defining all your routes, you place the `notFound` middleware to handle unmatched URLs.
    
- **errorHandler middleware**: Finally, you use the `errorHandler` middleware to catch all errors that occur in the app.
    

### **Testing**:

- Visiting `/unknown` (a non-existent route) should return a 404 error with the message `"Not found - /unknown"`.
    
- If there is an error in one of your routes, like trying to access a database resource that doesn't exist, it will be caught by `errorHandler`, and a structured error message will be sent back to the client.
    

### Summary of Steps:

1. **Define routes** first.
    
2. **Use `notFound` middleware** for 404 handling (at the end of routes).
    
3. **Use `errorHandler` middleware** to handle all errors (at the end of all middleware).