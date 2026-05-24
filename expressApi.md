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