---
layout: post
title: "Use MAMP mysql in Node.js (mac)"
---


安裝`mysql2`：

```bash
npm install mysql2 --save
```



設定網站接口：

```javascript
const mysql = require('mysql2');
const connection = mysql.createConnection({
  socketPath: '/Applications/MAMP/tmp/mysql/mysql.sock', // connect mamp mysql sock
  host: 'localhost',
  user: 'MYSQL_USERNAME',
  password: 'MYSQL_PASSWORD',
  database: 'MYSQL_DB_NAME',
  port: MYSQL_PORT
});
```



連接測試：

```javascript
connection.connect();

connection.query('SELECT 1 + 1 AS solution', function (error, results, fields) {
  if (error) throw error;
  console.log('The solution is: ', results[0].solution);
});

connection.end();
```



存取資料範例：

```javascript
const sql = 'SELECT * FROM example';

connection.query(sql, function(err, result) {
  if (err) {
    console.log('[SELECT ERROR] - ', err.message);
    return;
  }

  console.log(result);
});

connection.end();
```


進階存取範例：

```javascript
connection.connect();

function performQuery(sql) {
  return new Promise((resolve, reject) => {
    connection.query(sql, (error, results) => {
      if (error) {
        reject(error);
      } else {
        resolve(results);
      }
    });
  });
}

try {
  const results = await performQuery('SELECT * FROM example');
  console.log(results);
}
catch (error) {
  console.error('Error executing query:', error);
}

connection.end();
```


---

### 參考資料
- [https://chat.openai.com/](https://chat.openai.com/)
- [https://www.runoob.com/nodejs/nodejs-mysql.html](https://www.runoob.com/nodejs/nodejs-mysql.html)
