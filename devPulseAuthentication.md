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



### 5. Handling Register/SignUp new users  and Login Credentials files in `modules`

`modules/auth/ a.auth.route.ts b.auth.controller.ts c.auth.service.ts` 

a) auth.route.ts file:

```ts

import { Router } from "express";
import { authController } from "./auth.controller";



const router = Router();

router.post("/signup", authController.signUpUser);

router.post("/login", authController.loginUser);

export const authRoutes = router;


```

b) auth.controller.ts file:


```ts

import type { Request, Response } from "express";
import { authService } from "./auth.service";
import sendResponse from "../../utility/sendResponse";

const signUpUser = async (req: Request, res: Response) => {
    //  console.log(req.body);

    // console.log("coonection ok");
    try {

        const result = await authService.signUpUserIntoDB(req.body);

        // console.log(req.body);
        // console.log(result);

        sendResponse(res, {
            statusCode: 201,
            success: true,
            message: "User registered successfully",
            // data: result.rows[0],
            data: result,
        });

    }


    catch (error: any) {

        sendResponse(res, {
            statusCode: 500,
            success: false,
            message: "Failed",
            error: error.message
        });

    };



}

const loginUser = async (req: Request, res: Response) => {
    try {
        const result = await authService.loginUserIntoDB(req.body);


     // Here we are getting jwtAccessToken which will verify user
     // This AccessToken works as 1.verify user 2. Give permission to certain task 3.Protected routes for specific user 
     // 4. Hide unnecessary information to certain user 
        const { token } = result;

        // console.log(token);

        // console.log(result);

        res.status(200).json({
            success: true,
            message: "Login successful ",
            data: result,
        });

    }

    catch (error: any) {
        res.status(500).json({
            success: false,
            message: "Failed to login",
            error: error.message,
        });
    }


};

export const authController = {
    signUpUser,
    loginUser,
}


```


c) auth.service.ts file :


```ts

import bcrypt from "bcrypt";
// import bcrypt from "bcryptjs";

import type { IUser } from "./authUser.interface"
import { pool } from "../../db";
import jwt from "jsonwebtoken";
import config from "../../config";




const signUpUserIntoDB = async (payload: IUser) => {

    const { name, email, password, role } = payload;

    // console.log(payload);

    const hashPassword = await bcrypt.hash(password, 10);


    const result = await pool.query(
        `
    INSERT INTO users (name, email, password,role) VALUES($1, $2, $3, COALESCE($4, 'contributor')) RETURNING *
    
    
    `, [name, email, hashPassword, role],);

    delete result.rows[0].password;

    // return result;
    return result.rows[0];
}


const loginUserIntoDB = async (payload: { email: string, password: string }) => {

    const { email, password } = payload;

    // console.log(payload);

    const userData = await pool.query(
        `
    SELECT * FROM users WHERE email = $1
    `, [email]);


    if (userData.rows.length === 0) {
        throw new Error("Invalid email or password");
    }

    const user = userData.rows[0];

    // const isMatch = await bcrypt.compare(password, userData.rows[0].password);

    const isMatch = await bcrypt.compare(password, user.password);


    if (!isMatch) {
        throw new Error("Invalid password");
    }


    // Generate Token
    const jwtPayload = {
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role,
    };

    const accessToken = jwt.sign(jwtPayload, config.jwt_secret_key, { expiresIn: "7d" });

    const userAdditionalInfo = {
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role,
        created_at: user.created_at,
        updated_at: user.updated_at,
    };

    return { token: accessToken, user: userAdditionalInfo };
}



export const authService = {
    signUpUserIntoDB,
    loginUserIntoDB,
}


```


Handling type erros we have created a helper typeScript interface for IUser

d) authUser.interface.ts file:


```ts

export interface IUser {
    id: number;
    name: string;
    email: string;
    password: string;
    role?: string;
    created_at: Date;
    updated_at: Date;

}


```

### 6. Now Accesstoken Needs to Verify for Authentication and Give further Permission or Not to users

Now will do the validation in middleware/auth.ts file

```ts

import type { NextFunction, Request, Response } from "express";
import type { ROLES } from "../types";
import config from "../config";
import jwt, { type JwtPayload } from "jsonwebtoken";
import { pool } from "../db";


const auth = (...roles: ROLES[]) => {


    // console.log(roles);

    return async (req: Request, res: Response, next: NextFunction) => {

        try {

            //  console.log(roles);


            const token = req.headers.authorization;

            // console.log(token);

            if (!token) {
                return res.status(401).json({
                    success: false,
                    message: "Unauthorized, token is missing",
                });
            }


            const decodedToken = jwt.verify(token as string, config.jwt_secret_key) as JwtPayload;

            // console.log(decodedToken);


            const useData = await pool.query(
                `
            SELECT * FROM users WHERE email=$1
            `, [decodedToken.email],
            );

            // console.log(useData);

            const user = useData.rows[0];

            if (useData.rows.length === 0) {
                return res.status(401).json({
                    success: false,
                    message: "Unauthorized, user not found",
                });
            }



            if (roles.length > 0 && !roles.includes(user.role)) {
                return res.status(403).json({
                    success: false,
                    message: "Forbidden, you don't have permission to Access",
                });
            }



            req.user = decodedToken;


            next();
        }

        catch (error: any) {
            // console.log(error.message);
            next(error);
        }

    };

};


export default auth;



```

### 7. We need UserInformation To validate User Permission in Future (middleware/index.d.ts)

We have validated the user using jwt Access Token matched in 6. Now
So we Have to save userInformation somewhere to check again in future. Like:Role Base Permission
That's why we are saving UserInformation to jwtPayload where it's called `decodedToken`
This decodedToken next merged with express request metadata and set the method gloabally as req.user=decodedToken




```ts
import type { JwtPayload } from "jsonwebtoken";

declare global {
    namespace Express {
        interface Request {
            user?: JwtPayload;
        }
    }
}


```




### 8. After Authenticated Users and Validation Users Can do This task (Create,Update,Read,Delete)

`modules/issuetracker/ a.issue.route.ts b.issue.controller.ts c.issue.service.ts` 


a) issue.route.ts file:


```ts
import { Router } from "express";
import { issueController } from "./issue.controller";
import auth from "../../middleware/auth";
import { USER_ROLES } from "../../types";



const router = Router();

// router.post("/", issueController.createIssue);
router.post("/", auth(USER_ROLES.contributor, USER_ROLES.maintainer), issueController.createIssue);



router.get("/", issueController.getAllIssues);


router.get("/:id", issueController.getSingleIssue);


router.put("/:id", auth(USER_ROLES.contributor, USER_ROLES.maintainer), issueController.updateIssue);


router.delete("/:id", auth(USER_ROLES.maintainer), issueController.deleteIssue);





export const issueRoutes = router;


```


b) issue.controller.ts file:


```ts

import type { Request, Response } from "express"
import { issueService } from "./issue.service";
import sendResponse from "../../utility/sendResponse";


const createIssue = async (req: Request, res: Response) => {

    try {

        const userID = req.user?.id;


        const result = await issueService.createIssueIntoDB(req.body, userID);

        // console.log("Issue Controller:", req.user?.id);
        // console.log(result);



        sendResponse(res, {
            statusCode: 201,
            success: true,
            message: "Issue created successfully",
            data: result,
        });


    }

    catch (error: any) {

        sendResponse(res, {
            statusCode: 500,
            success: false,
            message: "Failed to create issue",
            error: error.message,
        });


    }

}


const getAllIssues = async (req: Request, res: Response) => {

    try {

        const query = req.query;

        const result = await issueService.getAllIssuesFromDB(query);

        // console.log("query result:", result);


        sendResponse(res, {
            statusCode: 200,
            success: true,
            message: "Issues retrieved successfully",
            data: result,
        });

    }

    catch (error: any) {

        sendResponse(res, {
            statusCode: 500,
            success: false,
            message: "Failed to get issues",
            error: error.message,
        });
    }

};





const getSingleIssue = async (req: Request, res: Response) => {

    const { id } = req.params;
    // console.log(req.params.id);

    // console.log(id);

    try {
        const result = await issueService.getSingleIssueByIdFromDB(id as string);

        // console.log(result);

        if (!result) {

            return sendResponse(res, {
                statusCode: 404,
                success: false,
                message: "Issue not found",
                data: {},
            });
        }


        sendResponse(res, {
            statusCode: 200,
            success: true,
            message: "Single Issue retrieved successfully",
            data: result,
        });

    }

    catch (error: any) {

        sendResponse(res, {
            statusCode: 500,
            success: false,
            message: "Failed to get issue",
            error: error.message,
        });
    }


};


const updateIssue = async (req: Request, res: Response) => {

    const { id } = req.params;

    try {

        const userID = req.user?.id;
        // console.log("User ID from JWT Payload:", userID);

        const result = await issueService.updateIssueInDB(id as string, req.body, userID);

        // console.log("Controller result:", result);


        if (!result) {

            return sendResponse(res, {
                statusCode: 404,
                success: false,
                message: "Issue not found",
                data: {},
            });
        }



        sendResponse(res, {
            statusCode: 200,
            success: true,
            message: "Issue updated successfully",
            data: result,
        });
    }

    catch (error: any) {


        sendResponse(res, {
            statusCode: 500,
            success: false,
            message: "Failed to update issue",
            error: error.message,
        });
    }
};



const deleteIssue = async (req: Request, res: Response) => {

    const { id } = req.params;

    try {

        const result = await issueService.deleteIssueFromDB(id as string);

        // console.log("Controller result:", result);

        if (result.rowCount === 0) {


            return sendResponse(res, {
                statusCode: 404,
                success: false,
                message: "Issue not found",
                data: {},
            });
        }



        sendResponse(res, {
            statusCode: 200,
            success: true,
            message: "Issue deleted successfully",
            data: {},
        });

    }

    catch (error: any) {

        sendResponse(res, {
            statusCode: 500,
            success: false,
            message: "Failed to delete issue",
            error: error.message,
        });
    }
}


export const issueController = {
    createIssue,
    getAllIssues,
    getSingleIssue,
    updateIssue,
    deleteIssue,
}




```


c) issue.service.ts file:


```ts

import type { Request } from "express";
import { pool } from "../../db";
import type { IIssue, IssueQuery } from "./issues.interface";


const createIssueIntoDB = async (payload: any, userID: number) => {

    const { title, description, type } = payload;

    const reporter_id = userID;
    // const { title, description, type } = payload;



    const result = await pool.query(`
        INSERT INTO issues (title, description, type, reporter_id)
        VALUES ($1, $2, $3,$4)
        RETURNING *
        `, [title, description, type, reporter_id]);

    return result.rows[0];



};


const getAllIssuesFromDB = async (query: IssueQuery) => {



    const { sort = "newest", type, status } = query;


    let queryfilter = `SELECT * FROM issues `;


    if (type === "bug") {
        queryfilter += `WHERE type='bug' `;

    }
    if (type === "feature_request") {
        queryfilter += `WHERE type='feature_request' `;
    }

    if (status === "open") {
        queryfilter += `WHERE status='open' `;
    }
    if (status === "in_progress") {
        queryfilter += `WHERE status='in_progress' `;
    }

    if (status === "resolved") {
        queryfilter += `WHERE status='resolved' `;
    }

    if (sort === "newest") {
        queryfilter += `ORDER BY created_at DESC`;
    }
    if (sort === "oldest") {
        queryfilter += `ORDER BY created_at ASC`;
    }

    const issueResult = await pool.query(queryfilter);

    // const issueResult = await pool.query(`
    //     SELECT * FROM issues
    // `);

    const reporterInfo = await pool.query(`
        SELECT id, name, role FROM users WHERE id IN (SELECT reporter_id FROM issues)
    `);



    const issues = issueResult.rows;

    const users = reporterInfo.rows;


    const formattedIssues = issues.map((issue) => {

        const reporter = users.find((user) => user.id === issue.reporter_id);

        return {
            id: issue.id,
            title: issue.title,
            description: issue.description,
            type: issue.type,
            status: issue.status,

            reporter: {
                id: reporter?.id,
                name: reporter?.name,
                role: reporter?.role,
            },

            created_at: issue.created_at,
            updated_at: issue.updated_at,
        };


    });

    // return formattedIssues || issueResult.rows;


    return formattedIssues;





};



const getSingleIssueByIdFromDB = async (id: string) => {

    // const issueId = parseInt(id);
    const issueResult = await pool.query(`
        SELECT * FROM issues WHERE id=$1
    `,
        [id],);

    // return issueResult.rows[0];

    const issues = issueResult.rows[0];

    if (!issues) {
        throw new Error("Issue not found");
    }


    const reporterInfo = await pool.query(`
        SELECT id, name, role FROM users WHERE id IN (SELECT reporter_id FROM issues)
    `);


    // const issues = issueResult.rows[0];
    // const users = reporterInfo.rows[0];

    const reporter = reporterInfo.rows[0];


    if (!reporter) {
        throw new Error("Reporter User not found");
    }

    // return { issues, users };


    const formattedIssue = {
        id: issues.id,
        title: issues.title,
        description: issues.description,
        type: issues.type,
        status: issues.status,

        reporter: {
            id: reporter?.id,
            name: reporter?.name,
            role: reporter?.role,
        },

        created_at: issues.created_at,
        updated_at: issues.updated_at,
    };

    return formattedIssue;



    // return formattedIssues.rows[0];
};



const updateIssueInDB = async (id: string, payload: IIssue, userID: string) => {


    const userInfo = await pool.query(`
        SELECT id, name, role FROM users WHERE id=$1
    `, [userID]);


    const reporterInfo = await pool.query(`
        SELECT * FROM issues WHERE id=$1
    `, [id]);


    if (userInfo.rows.length === 0) {
        throw new Error("User not found");
    }

    if (reporterInfo.rows.length === 0) {
        throw new Error("Issue not found");
    }

    const user = userInfo.rows[0];
    const issue = reporterInfo.rows[0];


    const { title, description, type, status } = payload;



    if (user.id === issue.reporter_id && user.role == "contributor" && issue.status == "open"
        || user.role == "maintainer") {
        const result = await pool.query(`
        UPDATE issues
        SET 
        title=COALESCE($1, title), 
        description=COALESCE($2, description), 
        type=COALESCE($3, type), 
        status=COALESCE($4, status),
        updated_at=NOW()
        WHERE id=$5
        RETURNING *
    `, [title, description, type, status, id]);

        return result.rows[0];
    }

    else {
        throw new Error("Unauthorized to update this issue");
    }



};






const deleteIssueFromDB = async (id: string) => {


    const result = await pool.query(`
        DELETE FROM issues WHERE id=$1 RETURNING *
    `, [id]);

    // return result.rows[0];
    return result;
};

export const issueService = {
    createIssueIntoDB,
    getAllIssuesFromDB,
    getSingleIssueByIdFromDB,
    updateIssueInDB,
    deleteIssueFromDB,
}


```


d) For type Safety, `issues.interface.ts` file:


```ts

export interface IIssue {
    id: number;
    title: string;
    description: string;
    type: string;
    status?: string;
    reporter_id: number;
    created_at: Date;
    updated_at: Date;
}

export interface IssueQuery {
    sort?: "newest" | "oldest";
    type?: "bug" | "feature_request";
    status?: "open" | "in_progress" | "resolved";
}



```


e) Check Role Base Permission Type and Type Safety, added `types/index.ts` file:


```ts


export const USER_ROLES = {
    maintainer: "maintainer",
    contributor: "contributor",
} as const;

export type ROLES = "maintainer" | "contributor";



```


### 9. Adding sendResponse utility and globalErrorHandler


a) `utility/sendResponse.ts` file:

```ts
import type { Response } from "express";


type TResponse<T> = {
    statusCode: number;
    success: boolean;
    message: string;
    data?: T;
    error?: any;
}


const sendResponse = <T>(res: Response, data: TResponse<T>) => {

    res.status(data.statusCode).json({
        success: data.success,
        message: data.message,
        data: data.data,
        error: data.error,
    });
}

export default sendResponse;


```


b) `middleware/globalErrorHandler.ts` file:


```ts
import type { NextFunction, Request, Response } from "express";


const globalErrorHandler = (
    err: Error,
    req: Request,
    res: Response,
    next: NextFunction) => {
    // console.error(err.stack);
    res.status(500).json(
        {
            success: false,
            message: err.message || "Internal Server Error",
        });
};

export default globalErrorHandler;

```

### 10. Build and Deploy to Vercel Now 
