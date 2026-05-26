## Lets Be Familiar with Express JS Code and Functionalities

### First thing First Request and Response comes First

```js

import express from 'express';
const app = express();

app.get('/mustafiz', (req, res) => {
  
  // 1. Requesting some Information

  console.log(req.method);       
  console.log(req.url);          
  console.log(req.originalUrl);  
  console.log(req.path);         
  console.log(req.protocol);     
  console.log(req.hostname);     

  // Modern Node.js Tip: req.ip can sometimes return IPv6 loopback '::1' or '::ffff:127.0.0.1' 
  // on local machines depending on your Node.js/OS network settings.

  console.log(req.ip);           

  // 2. Headers 
  console.log(req.headers);      
  console.log(req.get('user-agent')); 

  // 3. Body (Requires built-in middleware like express.json())
  console.log(req.body); 


  // 4. Cookies (Requires external 'cookie-parser' middleware)
  console.log(req.cookies);      


  // 5. Files (Requires external 'multer' middleware)
  console.log(req.file);         
  console.log(req.files);        

   
  // End the request lifecycle.
  res.send('Check console'); 

  
  // Don't like to send text !!
  // Cleaner Approach is this
  res.sendStatus(200);

});



```


### Now What is req ? and What is res ?

```js


req (Request Object): Contains all the `incoming data` sent by the client (browser, mobile app, Application etc.).

res (Response Object): Contains `methods` used to `send data` back to the client.

Warning Notes: Don't Compare With Newton's Third law of Motions, You will Get confused! 


```

---

---

## Now What are this req.method, req.url, req.originalUrl, req.path etc ?? Request MetaData


```js
Now Lets Explain a little bit, What actually are these `req `.` dot something or somothing`:


1.  `req.method`: The HTTP action used (Like: "GET", "POST", "PUT", "DELETE")


2.  `req.url`: The path and query string currently being handled (example, /mustafiz?user=true) 
               Don't get confused with `?user=true` is called optional query parameters,
               for this you can try `req.query` to further check


3.  `req.originalUrl`: Similar to req.url, but contains the original path even if 
                       internal rewriting or routing middleware changes later


4.  `req.path`:  The routing path part of the `URL`, completely stripping out query strings (Like: /mustafiz)


5.  `req.protocol`: The transfer protocol, typically "http" or "https"


6.  `req.hostname`: The domain name or server address (Like: "localhost" or "example.com")


7.  `req.ip`: The IP address of the client making the `request`. 



Note:  On your local machine, this often prints ::1 or ::ffff:127.0.0.1, 
       which are just IPv6 and IPv4 notations for "userItself" known as localhost.


`Warning` : Dont put anything you like after `req. (dot)` this OR,  your bank money will transfer to My Account 
(Mustafizur Rahman something wrong with your Bank Account please check )


```


## Request Headers?

```js

8.  `req.headers`: An object containing all incoming HTTP headers (metadata like content type, 
                   authorization tokens, or hosting info)


9.  `req.get('user-agent')`: A built-in Express helper function to easily retrieve a `specific header`, 
                         here user-agent identifies the browser, operating system, or device making the `request`.


```


## What is Middleware Dependency?


```ts

Properties looking for data injected into the `request` by `intermediate helper` software called middleware:


10. `req.body`:  Holds data submitted in the `request payload` (like a form submission or JSON API data Using 
                `POSTMAN or Thunder Client`).  It will read undefined unless you register a body-parser middleware  
                 like app.use(express.json()) earlier in the file


11. `req.cookies`: Holds cookies sent by the browser. It requires the external `cookie-parser` package to populate.


12. `req.file / req.files`: Used when uploading files (like images or PDFs). These properties remain undefined unless
                            an upload handling middleware, like multer, is actively parsing the incoming request form-data.


13. `req.send / req.sendStatus`: This ends the request-response lifecycle. It stops the browser spinner from turning and 
                                 sends the text string "Check console" back to the screen so the user knows the server 
                                 successfully processed their hit.

```



## Modern ES Module (ESM)


If we use multer or any other middleware to handle Files, we will eventually need to deal with file paths (like: saving a file to a directory).

In old CommonJS, we had access to global variables like `__dirname` and `__filename`. In modern ES Modules ("type": "module"), these variables do not exist.

If we need to construct file paths in `modern Express`, we have to do it using Node's built-in `import.meta.url`, Examples:


```js
import path from 'path';
import { fileURLToPath } from 'url';

// Recreate __dirname in modern ES Modules
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// Now we can safely use it with multer or static files:
// app.use(express.static(path.join(__dirname, 'public')));

```



### Modern Express V5+ Methods


```js

import express from 'express';
const app = express();


// 1. Standard Data Responses
app.get('/text', (req, res) => res.send('Hello')); // Auto text
app.get('/html', (req, res) => res.send('<h1>HTML</h1>')); //html
app.get('/json', (req, res) => res.json({ success: true })); // Best practice for API data



// 2. Custom Status Codes (Perfect for REST APIs)
app.post('/items', (req, res) => {
  
  res.status(201).json(
    { id: 1, 
      message: 'Created successfully' 
    });

});


// 3. Status Only (No Body)
// Cleanest way to handle an empty 200 OK or 404 Not Found in modern Express

app.delete('/items/:id', (req, res) => 
{
  res.sendStatus(204); // Sends "No Content" status and ends the request automatically

});



// 4. Redirects
app.get('/old-route', (req, res) => 
{
  // Modern standard format: Pass status first, destination second
  res.redirect(301, '/new-route'); 
});



// 5. Cookies & Headers
app.get('/cookie-demo', (req, res) => {
  // Setting a cookie with security best practices

  res.cookie('session_token', 'xyz123', {

    maxAge: 900000, // 15 mins
    httpOnly: true, // Shields against XSS attacks
    secure: true,   // Modern production requires HTTPS
    sameSite: 'strict' // Shields against CSRF attacks

  });


  // Setting multiple custom headers using Express wrapper syntax
  res.set(
  {
    'Cache-Control': 'no-cache',
    'X-Custom-Platform': 'Express-v5'
  });

  res.send('Cookie and headers configured.');
});


// 6. File Downloads
app.get('/receipt', (req, res) => 
{
  // Works perfectly for sending download attachments
  res.download('./invoice.pdf', 'custom-invoice-name.pdf');
  
});

```



## Express Middleware?

```ts

import express from 'express';
const app = express();

// 1. Custom Middleware (Logging)
// Modern Tip: While custom loggers are great for learning, production apps 
// usually use highly optimized packages like 'morgan' or 'pino'.

const logger = (req, res, next) => 
{
  console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
  next(); 
};

// Global application-level middleware
app.use(logger);


// 2. Path-specific Middleware
app.use('/api', logger); 


// 3. Multiple Middleware Passing
// Passing an array of middleware functions is the preferred modern syntax 
// for readability when combining multiple hooks.
app.use([express.json(), logger]);


// Mock auth middleware for example
const authMiddleware = (req, res, next) => next();


// 4. Route-specific Middleware
app.get('/protected', authMiddleware, (req, res) => {
  res.send('Protected content');
});


//  Express v5 Feature: Native Async Error Catching
// We no longer need an external utility like 'express-async-errors' or custom wrapper functions. 
// If an async operation rejects, Express v5 automatically passes it to the error-handling middleware below!
app.get('/async-data', async (req, res) => 
{
  const data = await fetchFromDatabase(); // If this throws an error, Express catches it natively
  res.json(data);
});


// 5. 404 Handler (Must be placed after all defined routes, but BEFORE the error handler)
app.use((req, res) => 
{
  res.status(404).json({ error: 'Not found' });
});


// 6. Error-handling Middleware (Must have exactly 4 parameters and be placed dead last)
app.use((err, req, res, next) => 
{
  console.error(err.stack);
  
  // Best practice: structure your error payloads uniformly
  res.status(err.status || 500).json(
  { 
    error: err.message || 'Something broke internally!' 
  }
  );
});


```