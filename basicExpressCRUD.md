## Basic CRUD Operation using Express JS



```ts

import express, {
  type Application,
  type Request,
  type Response,
} from "express";
import { Pool } from "pg";
import config from "./config";

//  Working with Express and NeonDB Postgre Database

// Three Basci (3) steps Only

// 1. Connect TO Database URL / Connection
// 2. Create or Initialize your Database Table

// You ARE READY FOR APPLY CRUD OPPERATION on DATABASE TABLE NOW

// 3. Apply CRUD OPERATION

// GOOD LUCK


const app: Application = express();
const port = config.port;

app.use(express.json());
app.use(express.text());
app.use(express.urlencoded({ extended: true }));

// Without Confugyration file

const pool = new Pool({
  connectionString: "postgresql://neondb_owner:BolboNA_BolboNA_BolboNA_BolboNA_channel_binding=require",
});

// After Database connection
// Table Created successfully
// Update Delete Create Get , Single user by ID


// Configuration file added for connection string from database
// const pool = new Pool({
//   connectionString: config.connection_string,
// });

const initDB = async () => {

  try 
  {
    await pool.query(`
        CREATE TABLE IF NOT EXISTS users(
        id SERIAL PRIMARY KEY,
        name VARCHAR(20),
        email VARCHAR(20) UNIQUE NOT NULL,
        password VARCHAR(20) NOT NULL,
        is_active BOOLEAN DEFAULT true,
        age INT,

        created_at TIMESTAMP DEFAULT NOW(),
        updated_at TIMESTAMP DEFAULT NOW()
        )
            
            `);


    console.log("Database connected successfully!");
  } 
  catch (error)
  {
    console.log(error);
  }

};

initDB();



app.get("/", (req: Request, res: Response) => 
{
  //res.send("Hello World!");
  res.status(200).json
  (
  {
    message: "Express Server",
    author: "Next Level",
  }

  );

});



// CREATE


// READ


// UPDATE


// DELETE






// Without Config File

app.listen(5000, () => {
  console.log(`Example app listening on port on 5000`);
});

// Config file added
// app.listen(port, () => {
//   console.log(`Example app listening on port ${port}`);
// });



```