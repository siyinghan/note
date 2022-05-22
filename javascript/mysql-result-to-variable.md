# MySQL results to variable in Node.js

- [MySQL results to variable in Node.js](#mysql-results-to-variable-in-nodejs)
    - [Knowledge](#knowledge)
    - [Issue](#issue)
    - [Solution (Promise+then)](#solution-promisethen)
    - [Solution (Promise+await)](#solution-promiseawait)
    - [Solution (Util+promisify)](#solution-utilpromisify)

### Knowledge

- Callback function
- Promise

### Issue

The results query from MySQL can be printed in the console:

```javascript
import mysql from 'mysql';

const connection = mysql.createConnection({
  user: 'root',
  host: '192.168.1.100',
  database: 'test',
  password: 'Admin1234!',
});

const sqlStr = 'select * from test_table';

connection.query(sqlStr, (err, results) => {
  if (err) throw err;
  console.log(results);
});

connection.end();
```

But the results cannot be used outside the scope of the `query` function.

```javascript
connection.query(sqlStr, (err, results) => {
  if (err) throw err;
  var resultsVariable = results;
});
console.log(resultsVariable);
```

No output

```javascript
let resultsVariable = '';
connection.query(sqlStr, (err, results) => {
  if (err) throw err;
  resultsVariable = results;
});
console.log(resultsVariable);
```

```bash
console.log(resultsVariable);
            ^
ReferenceError: resultsVariable is not defined
    at file:///index.js:17:13
    at ModuleJob.run (node:internal/modules/esm/module_job:198:25)
    at async Promise.all (index 0)
    at async ESMLoader.import (node:internal/modules/esm/loader:385:24)
    at async loadESM (node:internal/process/esm_loader:88:5)
    at async handleMainPromise (node:internal/modules/run_main:61:12)
(base) 
```

### Solution (Promise+then)

```javascript
import mysql from 'mysql';

const connection = mysql.createConnection({
  user: 'root',
  host: '192.168.1.100',
  database: 'test',
  password: 'Admin1234!',
});

const sqlStr = 'select * from test_table';

const getInfo = (sqlStr) => {
  return new Promise((resolve, reject) => {
    connection.query(sqlStr, (err, results) => {
      if (err) reject(err);
      resolve(results);
    });
  });
};

getInfo(sqlStr).then(
  (value) => {
    console.log(value);
  },
  (reason) => {
    console.log(reason);
  }
);

connection.end();
```

### Solution (Promise+await)

```javascript
import mysql from 'mysql';

const connection = mysql.createConnection({
  user: 'root',
  host: '192.168.1.100',
  database: 'test',
  password: 'Admin1234!',
});

const sqlStr = 'select * from test_table';

const getInfo = (sqlStr) => {
  return new Promise((resolve, reject) => {
    connection.query(sqlStr, (err, results) => {
      if (err) reject(err);
      resolve(results);
    });
  });
};
console.log(await getInfo(sqlStr));

connection.end();
```

### Solution (Util+promisify)

```javascript
import util from 'util';
import mysql from 'mysql';

const connection = mysql.createConnection({
  user: 'root',
  host: '192.168.1.100',
  database: 'test',
  password: 'Admin1234!',
});

const query = util.promisify(connection.query).bind(connection);
const sqlStr = 'select * from test_table';
console.log(await query(sqlStr));

connection.end();
```

