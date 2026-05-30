## DevPulse Authentication and Authorization Code Summary

### 1. Server.ts file

```ts

import { initDB } from "./db";
import config from "./config";
import app from "./app";



const main = async () => {
    await initDB();


    const port = config.port || 3000;
    app.listen(port, () => {
        console.log(`Server is running on port ${port}`);
    });

}

main();


```


### 2. Connection to NeonDB Database (db/index.ts)

initDb or initializing the Database connection and Building 2 Tables, 1.users 2.issues


```ts

// database will conneect here
import { Pool } from "pg";
import config from "../config";

export const pool = new Pool({
    connectionString: config.connection_string,
});

export const initDB = async () => {
    try {
        await pool.query(`
            
            CREATE TABLE IF NOT EXISTS users (
            
            id SERIAL PRIMARY KEY,

            name VARCHAR(50) NOT NULL,

            email VARCHAR(30) UNIQUE NOT NULL,
            password TEXT NOT NULL,
            
            role VARCHAR(20) DEFAULT 'contributor'
            CHECK (role IN ('maintainer', 'contributor')),


            created_at TIMESTAMP DEFAULT NOW(),

            updated_at TIMESTAMP DEFAULT NOW()
        )
            
            
            
            `);





        await pool.query(`
            
            CREATE TABLE IF NOT EXISTS issues(
            
            id SERIAL PRIMARY KEY,

            title VARCHAR(150) NOT NULL,
            
            description TEXT NOT NULL
            CHECK(CHAR_LENGTH(description) >= 20),

            type VARCHAR(20) NOT NULL
            CHECK (type IN ('bug', 'feature_request')),


            status VARCHAR(20) DEFAULT 'open'
            CHECK (status IN ('open', 'in_progress', 'resolved')),


            reporter_id INT NOT NULL
            REFERENCES users(id),


            created_at TIMESTAMP DEFAULT NOW(),

            updated_at TIMESTAMP DEFAULT NOW()
            
            )
            
            `);



        console.log("Database connected successfully");
    }

    catch (error) {
        console.log(error);

    }
};


```


### 3. Establishing Config to handle environment files and secret key and Database url for safety (config/index.ts)


```ts


import { env } from "process";

import dotenv from "dotenv";
dotenv.config({ quiet: true });

const config = {

    port: env.PORT as string,
    connection_string: env.DATABASE_URL as string,
    jwt_secret_key: env.JWT_SECRET_KEY as string,
};

export default config;

```



### 4. Handling root Routes and Middleware in app.ts file



```ts


import cors from "cors";

import type { Application, Request, Response } from "express";

import express from "express";
import { authRoutes } from "./modules/auth/auth.route";
import { issueRoutes } from "./modules/issuetracker/issue.route";
import globalErrorHandler from "./middleware/globalErrorHandler";


const app: Application = express();

app.use(express.json());
app.use(express.text());
app.use(express.urlencoded({ extended: true }));


app.use(
    cors({
        origin: "http://localhost:3000",
    }),
);



app.get("/", (req: Request, res: Response) => {

    // res.send("Mustafiz Authentication DevPulse Project");

    res.status(200).json({
        message: "Mustafiz Authentication DevPulse Project",
        author: "Mustafizur Rahman",
    });
});

app.use("/api/auth", authRoutes);

app.use("/api/issues", issueRoutes);



app.use(globalErrorHandler);

export default app;






```