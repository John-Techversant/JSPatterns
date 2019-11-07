## Express API design guidelines 

install express,bodyparser and necessary packages needed for the project 

Setting up  basic  Index.js

    const express = require("express");
    const bodyParser = require("body-parser");
    require('dotenv').config(); // Simply load the ENV 
    //Get the Port from env or set it as 3000
    
    const port = process.env.PORT || 3000;
    const app = express();
    
    //Load the basic middlewares 
    app.use(bodyParser.urlencoded({ extended: false}));
    app.use(bodyParser.json());
   
    //  load routes from sub-directories
    app.use('/api', require("./routes"));
    
    //handle routes that doesn't exist  with a 404 html
    app.use(function (req, res) {
     res.status(404).sendFile(__dirname'/404page/404.html');
    });
    
    app.listen(port,console.log("The API server started on Port", port));


The '/api' route has a dependency that is required from the routes folder 

The routes folder contains the following 

![enter image description here](https://lh3.googleusercontent.com/c68YqnTUbxKfaINGTcqGHXdo4A0v52gPRrHihlk4mkmfjEOHP1lPmnn-eP8QvEQARVFaLFtldjbL)
 
the routes folder also has an index.js which will be the default child routes for  exports .


*api.js*

      const api = require('express').Router();
      api.use('/retail',  require("./retail"));
      api.use('/corp',  require("./corp"));
      export default api	
      


Configuring DB  lets create a db_confing.js under the config folder 

 *db.config.js*

       module.exports  =  {
        pg_config:  {
        client:  "pg",
        connection:  {
        host:  process.env.db_host,
        user:  process.env.db_user,
        password:  process.env.db_password,
        database:  process.env.db_name
       }
        }  
          }

Let connect the db with a connection.js file and export the connection driver 

*connection.js*
 

     const  db  =  require("./db.config");
     //use a try - catch handler if possible 
     const  connection  =  require("knex")(db.pg_config);

     export default  connection;

Now the **connection**  object has the driver which could be reused to the routes that needs db access , which isolates 
and easy to debug routes

 
The Effective login   /api/login  , Lets take a look at api.js with some modification 



    const api = require('express').Router();
   
    const  validator  =  require('../helpers/validator');
    const connection =require('../config/connection ')
    
      // Function for user auth 
        
    const  userauth  =  require('../controllers/userauth');
    
      // Function for Generating token based on user model
        
    const  tokengen  =  require("../controllers/tokengen");
    
    api.route('/login').post(
			    validator.login,
			    userauth(connection),
			    tokengen);
			    
	api.use('/retail', require("./retail"));
    api.use('/corp',   require("./corp"));
    export default api		    



validator.login is an input sanitizer function that sanitizes the JSON  body 

we have followed a middleware pattern here 

  ![enter image description here](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/index/_static/request-delegate-pipeline.png?view=aspnetcore-3.0)


each middle ware function retruns a callback with req ,res 
if the conditions are met the next() is called on and reaches the last middelware function 

Lets look at the validator.login function  *example*

    validator.login =    ()  =>  {
    return  (req,res,next)  =>
    {
       const  result  =  SomeWeirdValidation(req.body)
        if(result.error){
    
          return  res.status(500).json(
                { "status"  :  "error",
                 "message":result.error  
                 });
             }
    
         else{

           next();
      }
                           }



 next() is called on success which will go to the userauth function with connection (DB driver as dependancy)
