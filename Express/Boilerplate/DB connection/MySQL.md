```js
import mysql from 'mysql2/promise';
import dotenv from 'dotenv';
dotenv.config();

const dbPool = mysql.createPool({
host: process.env.DB_HOST,
user: process.env.DB_USER,
password: process.env.DB_PASSWORD,
database: process.env.DB_NAME,

waitForConnections: true,
connectionLimit: 10,

maxIdle: 10, // max idle connections, the default value is the same as `connectionLimit`
idleTimeout: 60000, // idle connections timeout, in milliseconds, the default value 60000
queueLimit: 0,
enableKeepAlive: true,
keepAliveInitialDelay: 0,
});

export default dbPool;
```

```js
//services
import dbPool from "../config/database.js";
```