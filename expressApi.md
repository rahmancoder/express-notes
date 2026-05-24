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

```


```