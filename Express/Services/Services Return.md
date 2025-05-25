```js
export const getOneAlternative = async (id_alternative, user_id) => {  
    const SQLQuery = `SELECT * FROM alternatives WHERE ID_alternative = ? AND user_id = ?`;  
    try {  
        const [rows] = await dbPool.execute(SQLQuery, [id_alternative, user_id]);  
  
        if (rows.length === 0) {  
            return [];  // No results, return an empty array  
        }  
        // Return the first row if found  
        return rows[0];  
    } catch (err) {  
        console.error('Error executing query:', err);  
        // Rethrow the error to ensure it reaches the global error handler  
        throw err;  
    }  
};
```

better to return array of data if data found, or array of empty if data empty, let controller do the error handling