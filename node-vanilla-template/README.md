# Getting up to speed

The following repo is a project for demonstrating best practices in structuring a node.js project
Note that when a file or directory contains "_EXAMPLE" at its extension or name, it has to be removed. We have added this to demonstration purposes, because the main files would be ignored by git.

## Initial steps

* initialize git: `Git init`
* install necessary packages: `npm install express nodemon pg dotenv`
* make the package.json file: `npm init --yes`
* add the following to "scripts" value package.json: `"dev": "nodemon app.js"`
* add the following key value pair to package.json: `"type": "module"`

## Explanation of Key Components

1. **src/**: Keeps all your source code organized.
   * **config/**: Stores configuration logic (e.g., connecting to a database or loading .env variables).
   * **controllers/**: Contains the logic for handling requests and responses (e.g., CRUD operations). Each file in this directory typically corresponds to a specific resource or entity (e.g., userController.js for user-related operations, orderController.js for orders), grouping related functions together.
        * Process Requests: Extract data from `HTTP` requests (e.g., req.body, req.params).
        * Interact with Models: Use models to query or manipulate data in PostgreSQL.
        * Handle Logic: Apply business rules (e.g., "only admins can delete users").
        * Send Responses: Return data or errors to the client (e.g., JSON responses via res.json()).
   * **models/**: Defines database schemas (e.g., for MongoDB with Mongoose). Think of it as the blueprint for your app’s data.
        * Define the shape of your data (e.g., what fields a "User" has: name, email, etc.).
        * Specify validation rules (e.g., "email must be unique" or "age must be a number").
        * Provide an interface to interact with the database (e.g., `MongoDB`, `PostgreSQL`, etc.).
        * Abstract database-specific logic away from the rest of your app (like controllers or routes).
   * **routes/**: Maps endpoints to controller functions (e.g., `/api/users` → `userController`). The `routes/` folder is where you define the API endpoints of your application—essentially mapping HTTP methods (`GET`, `POST`, `PUT`, `DELETE`) and URL paths to specific controller functions. It acts as the entry point for incoming requests, directing them to the appropriate logic in the `controllers/` folder.
        * Defines Endpoints: Specifies the URL paths (e.g., `/users`, `/users/:id`) and `HTTP` methods (e.g., `GET`, `POST`) that clients can hit.
        * Connects to Controllers: Links each endpoint to a controller function that handles the request.
        * Organizes API Structure: Groups related endpoints (e.g., all user-related routes in one file).
        * Middleware Integration: Applies middleware (e.g., authentication, validation) before passing requests to controllers.
   * **middleware/**: Houses reusable middleware like authentication or error handling. The middleware/ folder is where you store reusable functions that sit between incoming requests and the route handlers (controllers). Middleware functions in `Express.js` process requests, modify them, enforce rules, or terminate them early, making them a powerful tool for handling cross-cutting concerns like authentication, validation, logging, or error handling.
        * Processes Requests: Modifies req/res objects or extracts data before passing control to the next handler.
        * Enforces Rules: Checks conditions (e.g., authentication) and can block requests if they fail.
        * Handles Cross-Cutting Logic: Centralizes functionality like logging, validation, or error handling that applies across multiple routes.
        * Improves Modularity: Keeps routes and controllers clean by offloading reusable logic.
   * **utils/**: Miscellaneous helper functions (e.g., formatting dates, logging).
   * **app.js**: Sets up the Express app, middleware, and routes. The `app.js` file serves as the central configuration hub for your Express application. It’s where you initialize the Express app, set up global middleware, mount routes, and configure other app-wide settings. While it doesn’t typically contain business logic or database queries (those belong in controllers), it orchestrates how the app handles incoming requests and responses.

2. **tests/**: Separates test files for unit and integration testing (e.g., using `Jest` or `Mocha`).

3. **public/**: Serves static assets like `HTML`, `CSS`, or client-side `JavaScript` (if applicable).

4. **.env**: Stores sensitive data like API keys or database credentials (never commit this to version control).

5. **.gitignore**: Excludes files like node_modules/ and .env from `Git`.

6. **package.json**: Defines scripts (e.g., `npm start`), dependencies, and project info.

7. **server.js**: The entry point that starts the server and imports `app.js`.

## Best practices for each directory

1. **src/controllers/**:
    * Single Responsibility Principle:
    * Each controller function should handle one specific task (e.g., `createUser` only creates a user, not unrelated tasks like sending emails).
    * Consistent Response Structure: Standardize your API responses for predictability
            ```javascript
            res.status(200).json({
              success: true,
              data: result,
            });
            ```
            ```javascript
            res.status(400).json({
              success: false,
              error: 'Something went wrong',
            });
            ```
    * Error Handling: Always use try/catch with async operations to catch Sequelize or database errors. Consider a global error-handling middleware to avoid repetitive code (more on this later).
        * Return meaningful HTTP status codes:
        * 201: Resource created.
        * 200: Success (GET or PUT).
        * 204: Success, no content (DELETE).
        * 400: Bad request (validation errors).
        * 404: Not found.
        * 500: Server error.
    * Input Validation:
        * Validate `req.body` or `req.params` before passing to the model.
        * Use a library like `Joi` or `express-validator` in middleware, not in the controller, to keep it focused on logic.
        * Example with manual validation:

            ```javascript
            if (!name || !email) {
              return res.status(400).json({ success: false, error: 'Name and email are required' });
            }
            ```

    * Keep Controllers Thin:
        * Avoid heavy logic (e.g., complex calculations, external API calls). Move such tasks to a services/ folder or utils/.
        * Example: If createUser needs to hash a password, delegate that to a utility:

            ```javascript
            const { hashPassword } = require('../utils/password');
            const hashedPassword = await hashPassword(req.body.password);
            ```

    * Use Async/Await:
        * Stick to `async/await` for database operations instead of callbacks or raw promises—it’s cleaner and easier to handle errors.
        * Example: `await User.create()` vs.`User.create().then()`.
    * Naming Conventions:
        * Use descriptive, action-oriented names: createUser, getUsers, updateUserById.
        * Match controller filenames to resources: userController.js, orderController.js.
    * Avoid Business Logic in Models:
        * Models should define data structure and basic CRUD. Put app-specific rules (e.g., "users under 18 can’t register") in controllers or a service layer.

            ```javascript
            if (age < 18) {
              return res.status(400).json({ success: false, error: 'User must be 18 or older' });
            }
            ```

    * Centralize Reusable Logic:
        * If multiple controllers need authentication checks, move them to middleware/:

            ```javascript
            const authMiddleware = require('../middleware/auth');
            router.get('/:id', authMiddleware, getUserById);
            ```

2. **src/models/**:
    * One Model per File: Keep each entity in its own file (e.g., userModel.js, postModel.js) for clarity.
    * Validation: Add validation rules in the schema to catch errors early.
    * Relationships: If your database supports it (e.g., SQL), define relationships (e.g., a User has many Posts).
    * Naming: Use singular, descriptive names (e.g., userModel.js not users.js), as the file defines a single model type.
    * Indexes: For performance, add indexes to frequently queried fields (e.g., email in MongoDB).
3. **src/routes/**:
    * An example of user router is below:<br>
        `./src/routers/userRoutes.js`***(The routers file for users API)***<br> 

        ```javascript
        import express from 'express';
        import { createUser, getUsers, getUserById } from '../controllers/userController.js';
        
        const router = express.Router();
        
        // Create a user
        router.post('/', createUser);
        
        // Get all users
        router.get('/', getUsers);
        
        // Get a single user by ID
        router.get('/:id', getUserById);
        
        export default router;
        ```  

        `./src/app.js` ***(The main app file)***<br> 

        ```javascript
        import express from 'express';
        import userRoutes from './routes/userRoutes.js';
        
        const app = express();
        app.use(express.json()); // Parse JSON bodies
        app.use('/users', userRoutes); // Mount user routes at /users
        
        export default app;
        ```

        `./server.js` ***(The startup file)***<br> 

        ```javascript
        import app from './app.js';
        
        const PORT = process.env.PORT || 3000;
        app.listen(PORT, () => {
          console.log(`Server running on port ${PORT}`);
        });
        ```

    * RESTful Conventions: Stick to RESTful naming:
        * `GET /users` → List users.
        * `POST /users` → Create a user.
        * `GET /users/:id` → Get a user. <br>
        Example:<br>

            ```javascript
            router.get('/', getUsers);
            router.post('/', createUser);        
            ```

    * Modular Routing with `Router`: Use `express.Router()` for modularity:<br>

        ```javascript
        import express from 'express';
        const router = express.Router();      
        ```

    * Consistent Path Naming:
        * Use lowercase, kebab-case: /users, not /Users.
        * Avoid trailing slashes: /users, not /users/.
    * Group by Resource: Organize routes by resource:

        ```javascript
        import userRoutes from './routes/userRoutes.js';
        import orderRoutes from './routes/orderRoutes.js';
        app.use('/users', userRoutes);
        app.use('/orders', orderRoutes);
        ```

    * Middleware Integration:<br>
        Apply middleware at the route level:

        ```javascript
        import { authMiddleware } from '../middleware/auth.js';
        router.get('/:id', authMiddleware, getUserById);
        ```

        Example middleware `(src/middleware/auth.js)`:

        ```javascript
        export const authMiddleware = (req, res, next) => {
          if (!req.headers.authorization) {
            return res.status(401).json({ success: false, error: 'Unauthorized' });
          }
          next();
        };
        ```

    * Route Parameters: Use `:param` and validate in controllers or middleware:

        ```javascript
        router.get('/:id', getUserById);
        ```

    * API Versioning: Prefix routes with a version.

        ```javascript
        app.use('/api/v1/users', userRoutes);
        ```

    * Keep Routes Logic-Free: Avoid business logic or queries in routes:<br>
        Bad:

        ```javascript
        router.get('/', async (req, res) => {
          const result = await pool.query('SELECT * FROM users');
          res.json(result.rows);
        });
        ```  

        Good:

        ```javascript
        router.get('/', getUsers);
        ```

4. **src/middleware/**:
    * Single Responsibility:Each middleware should do one thing well:
        * `authMiddleware:` Checks authentication.
        * `validateUser:` Validates input.
        * `loggerMiddleware:` Logs requests.
    * Error Handling: Middleware can terminate requests with errors:

        ```javascript
        if (!authHeader) {
          return res.status(401).json({ success: false, error: 'Unauthorized' });
        }
        ```  

        For async middleware, wrap in `try/catch` and pass errors to next:

        ```javascript
        export const asyncMiddleware = async (req, res, next) => {
          try {
            const data = await someAsyncOperation();
            req.data = data;
            next();
          } catch (error) {
            next(error); // Pass to error-handling middleware
          }
        };
        ```

    * Keep Middleware Thin:
        * Avoid heavy logic (e.g., database queries). Delegate to controllers: <br>
        Bad:

            ```javascript
            export const userMiddleware = async (req, res, next) => {
              const result = await pool.query('SELECT * FROM users WHERE id = $1', [req.params.id]);
              req.user = result.rows[0];
              next();
            };
            ```

        Good: Do this in the controller; middleware should only prepare or enforce.
    * Global vs. Route-Specific: Use global middleware in `app.js` for app-wide tasks (e.g., logging). Use route-specific middleware for targeted needs (e.g., `authentication`):
    * File Naming:
        * Name files after their purpose: auth.js, validation.js, logger.js.
        * For multiple related middleware, group in one file or use a subdirectory:

            ```javascript
            middleware/
            ├── auth/
            │   ├── auth.js
            │   └── admin.js
            └── validation.js
            ```

## Points to note

1. The app variable (express()) has a local variable dbPool which can be used in every file for interacting with the database
2. We have used postgreSQL in this template. Helper functions for interacting with postgreSQL are added to `./src/utils/dbUtils.js`
3. In all the codes of the following documentation, ES6 module has been used.

## TODO

1. Add logger.js to src dir.

## Overall structure of the project

```python
my-node-app/
├── src/                 # Source code directory
│   ├── config/          # Configuration files (e.g., database, environment variables)
│   │   └── db.js
│   ├── controllers/     # Business logic for handling requests
│   │   └── userController.js
│   ├── models/          # Database models/schemas (e.g., Mongoose schemas)
│   │   └── userModel.js
│   ├── routes/          # API route definitions
│   │   └── userRoutes.js
│   ├── middleware/      # Custom middleware (e.g., authentication, error handling)
│   │   └── auth.js
│   ├── utils/           # Utility functions (e.g., helpers, reusable code)
│   │   └── logger.js
│   └── app.js           # Main application file (entry point for Express)
├── tests/               # Test files (unit, integration tests)
│   ├── unit/
│   └── integration/
├── public/              # Static files (e.g., CSS, images, client-side JS)
│   ├── css/
│   └── js/
├── .env                 # Environment variables (e.g., PORT, DB_URI)
├── .gitignore           # Git ignore file (e.g., node_modules, .env)
├── package.json         # Project metadata and dependencies
├── README.md            # Project documentation
└── server.js            # Server startup file (loads app.js and starts the server)
```
