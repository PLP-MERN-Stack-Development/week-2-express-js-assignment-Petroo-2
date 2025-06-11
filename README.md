[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=19752612&assignment_repo_type=AssignmentRepo)
# Express.js RESTful API Assignment

This assignment focuses on building a RESTful API using Express.js, implementing proper routing, middleware, and error handling.

## Assignment Overview

You will:
1. Set up an Express.js server
2. Create RESTful API routes for a product resource
3. Implement custom middleware for logging, authentication, and validation
4. Add comprehensive error handling
5. Develop advanced features like filtering, pagination, and search

## Getting Started

1. Accept the GitHub Classroom assignment invitation
2. Clone your personal repository that was created by GitHub Classroom
3. Install dependencies:
   ```
   npm install
   ```
4. Run the server:
   ```
   npm start
   ```

## Files Included

- `Week2-Assignment.md`: Detailed assignment instructions
- `server.js`: Starter Express.js server file
- `.env.example`: Example environment variables file

## Requirements

- Node.js (v18 or higher)
- npm or yarn
- Postman, Insomnia, or curl for API testing

## API Endpoints

The API will have the following endpoints:

- `GET /api/products`: Get all products
- `GET /api/products/:id`: Get a specific product
- `POST /api/products`: Create a new product
- `PUT /api/products/:id`: Update a product
- `DELETE /api/products/:id`: Delete a product

## Submission

Your work will be automatically submitted when you push to your GitHub Classroom repository. Make sure to:

1. Complete all the required API endpoints
2. Implement the middleware and error handling
3. Document your API in the README.md
4. Include examples of requests and responses

## Resources

- [Express.js Documentation](https://expressjs.com/)
- [RESTful API Design Best Practices](https://restfulapi.net/)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)



// server.js - Express.js RESTful API Server for Products

// Import necessary modules
const express = require('express'); // Express.js framework for building web applications
const bodyParser = require('body-parser'); // Middleware to parse incoming request bodies
const dotenv = require('dotenv'); // Loads environment variables from a .env file
const { v4: uuidv4 } = require('uuid'); // For generating unique IDs

// Load environment variables from .env file
dotenv.config();

// Initialize the Express application
const app = express();
const PORT = process.env.PORT || 3000; // Define the port, default to 3000 if not specified in .env
const API_KEY = process.env.API_KEY || 'your_secret_api_key'; // Define the API key for authentication

// Middleware Setup

// 1. Body Parser Middleware: Parses incoming request bodies (JSON and URL-encoded)
app.use(bodyParser.json()); // For parsing application/json
app.use(bodyParser.urlencoded({ extended: true })); // For parsing application/x-www-form-urlencoded

// 2. Custom Logging Middleware: Logs details about each incoming request
const loggerMiddleware = (req, res, next) => {
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
    next(); // Pass control to the next middleware or route handler
};
app.use(loggerMiddleware); // Apply the logger middleware globally

// 3. Custom Authentication Middleware: Checks for a valid API key in the 'x-api-key' header
const authenticateMiddleware = (req, res, next) => {
    const apiKey = req.headers['x-api-key']; // Get the API key from the request headers
    if (!apiKey || apiKey !== API_KEY) {
        // If API key is missing or invalid, send a 401 Unauthorized response
        return res.status(401).json({ message: 'Unauthorized: Invalid or missing API Key' });
    }
    next(); // If authentication is successful, pass control to the next middleware
};

// In-memory Product Data (simulates a database)
// This array will hold our product objects. In a real application, this would be a database.
let products = [
    {
        id: uuidv4(),
        name: 'Laptop Pro',
        description: 'Powerful laptop for professional use with 16GB RAM and 512GB SSD.',
        price: 1200.00,
        category: 'Electronics',
        stock: 50
    },
    {
        id: uuidv4(),
        name: 'Wireless Mouse X',
        description: 'Ergonomic wireless mouse with 8000 DPI sensor.',
        price: 25.50,
        category: 'Accessories',
        stock: 200
    },
    {
        id: uuidv4(),
        name: 'Mechanical Keyboard RGB',
        description: 'Full-size mechanical keyboard with customizable RGB backlighting.',
        price: 99.99,
        category: 'Accessories',
        stock: 100
    },
    {
        id: uuidv4(),
        name: '4K Monitor Ultra',
        description: '27-inch 4K UHD monitor with HDR support.',
        price: 450.00,
        category: 'Electronics',
        stock: 30
    },
    {
        id: uuidv4(),
        name: 'USB-C Hub 7-in-1',
        description: 'Multi-port USB-C hub with HDMI, USB 3.0, and SD card reader.',
        price: 35.00,
        category: 'Accessories',
        stock: 150
    },
    {
        id: uuidv4(),
        name: 'Smartphone Elite',
        description: 'Latest model smartphone with advanced camera system and long battery life.',
        price: 899.00,
        category: 'Electronics',
        stock: 75
    },
    {
        id: uuidv4(),
        name: 'External SSD 1TB',
        description: 'Portable 1TB SSD for fast data transfer.',
        price: 150.00,
        category: 'Storage',
        stock: 120
    }
];

// 4. Validation Middleware: Validates product data for POST and PUT requests
const validateProduct = (req, res, next) => {
    const { name, description, price, category, stock } = req.body;

    if (!name || typeof name !== 'string' || name.trim() === '') {
        return res.status(400).json({ message: 'Product name is required and must be a non-empty string.' });
    }
    if (!description || typeof description !== 'string' || description.trim() === '') {
        return res.status(400).json({ message: 'Product description is required and must be a non-empty string.' });
    }
    if (typeof price !== 'number' || price <= 0) {
        return res.status(400).json({ message: 'Product price is required and must be a positive number.' });
    }
    if (!category || typeof category !== 'string' || category.trim() === '') {
        return res.status(400).json({ message: 'Product category is required and must be a non-empty string.' });
    }
    if (typeof stock !== 'number' || stock < 0 || !Number.isInteger(stock)) {
        return res.status(400).json({ message: 'Product stock is required and must be a non-negative integer.' });
    }
    next(); // If validation passes, proceed
};

// API Endpoints

// GET /api/products: Get all products with filtering, pagination, and search capabilities
app.get('/api/products', (req, res) => {
    let filteredProducts = [...products]; // Create a mutable copy of products

    // 1. Search: Filter by 'q' (query string) in name or description
    const { q, category, minPrice, maxPrice, page = 1, limit = 10 } = req.query;

    if (q) {
        const lowerQ = q.toLowerCase();
        filteredProducts = filteredProducts.filter(product =>
            product.name.toLowerCase().includes(lowerQ) ||
            product.description.toLowerCase().includes(lowerQ)
        );
    }

    // 2. Filtering: Filter by category
    if (category) {
        const lowerCategory = category.toLowerCase();
        filteredProducts = filteredProducts.filter(product =>
            product.category.toLowerCase() === lowerCategory
        );
    }

    // 3. Filtering: Filter by price range (minPrice, maxPrice)
    if (minPrice) {
        const parsedMinPrice = parseFloat(minPrice);
        if (!isNaN(parsedMinPrice)) {
            filteredProducts = filteredProducts.filter(product => product.price >= parsedMinPrice);
        }
    }
    if (maxPrice) {
        const parsedMaxPrice = parseFloat(maxPrice);
        if (!isNaN(parsedMaxPrice)) {
            filteredProducts = filteredProducts.filter(product => product.price <= parsedMaxPrice);
        }
    }

    // 4. Pagination: Apply limit and page offset
    const startIndex = (parseInt(page) - 1) * parseInt(limit);
    const endIndex = startIndex + parseInt(limit);

    const paginatedProducts = filteredProducts.slice(startIndex, endIndex);

    // Return the response with paginated products and total count for client-side pagination UI
    res.json({
        total: filteredProducts.length,
        page: parseInt(page),
        limit: parseInt(limit),
        products: paginatedProducts
    });
});

// GET /api/products/:id: Get a specific product by ID
app.get('/api/products/:id', (req, res) => {
    const { id } = req.params; // Extract product ID from URL parameters
    const product = products.find(p => p.id === id); // Find the product in the array

    if (!product) {
        // If product not found, send a 404 Not Found response
        return res.status(404).json({ message: 'Product not found' });
    }
    res.json(product); // Send the product data as JSON
});

// POST /api/products: Create a new product
// Uses authenticateMiddleware and validateProduct middleware before handling the request
app.post('/api/products', authenticateMiddleware, validateProduct, (req, res) => {
    const { name, description, price, category, stock } = req.body; // Extract product data from request body
    const newProduct = {
        id: uuidv4(), // Generate a unique ID for the new product
        name,
        description,
        price,
        category,
        stock
    };
    products.push(newProduct); // Add the new product to our in-memory array
    res.status(201).json(newProduct); // Send the newly created product with a 201 Created status
});

// PUT /api/products/:id: Update a product
// Uses authenticateMiddleware and validateProduct middleware before handling the request
app.put('/api/products/:id', authenticateMiddleware, validateProduct, (req, res) => {
    const { id } = req.params; // Extract product ID from URL parameters
    const { name, description, price, category, stock } = req.body; // Extract updated data from request body

    const productIndex = products.findIndex(p => p.id === id); // Find the index of the product

    if (productIndex === -1) {
        // If product not found, send a 404 Not Found response
        return res.status(404).json({ message: 'Product not found' });
    }

    // Update the product's properties
    products[productIndex] = {
        ...products[productIndex], // Keep existing properties
        name,
        description,
        price,
        category,
        stock
    };

    res.json(products[productIndex]); // Send the updated product data
});

// DELETE /api/products/:id: Delete a product
// Uses authenticateMiddleware before handling the request
app.delete('/api/products/:id', authenticateMiddleware, (req, res) => {
    const { id } = req.params; // Extract product ID from URL parameters
    const initialLength = products.length; // Store initial length to check if a product was deleted

    // Filter out the product to be deleted
    products = products.filter(p => p.id !== id);

    if (products.length === initialLength) {
        // If length hasn't changed, product was not found
        return res.status(404).json({ message: 'Product not found' });
    }
    res.status(204).send(); // Send a 204 No Content status for successful deletion
});

// Error Handling Middleware

// 1. Not Found Middleware: Handles requests to routes that do not exist
app.use((req, res, next) => {
    const error = new Error(`Not Found - ${req.originalUrl}`);
    res.status(404); // Set status to 404
    next(error); // Pass the error to the global error handler
});

// 2. Global Error Handling Middleware: Catches all errors passed by `next(error)`
app.use((err, req, res, next) => {
    // If headers have already been sent, delegate to default Express error handler
    if (res.headersSent) {
        return next(err);
    }

    const statusCode = res.statusCode === 200 ? 500 : res.statusCode; // Determine status code, default to 500
    res.status(statusCode);
    res.json({
        message: err.message, // Send the error message
        stack: process.env.NODE_ENV === 'production' ? null : err.stack // Only send stack trace in development
    });
});

// Start the server
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
    console.log(`Access API at: http://localhost:${PORT}/api/products`);
    console.log(`API Key for authenticated routes: ${API_KEY}`);
