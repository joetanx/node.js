## 1. Preparations

- Setup MySQL database according to this guide: https://github.com/joetanx/setup/blob/main/mysql.md
- Prepare required npm packages and firewall rule for the listener example

  ```console
  npm install mysql2 express
  firewall-cmd --add-port 8080/tcp --permanent && firewall-cmd --reload
  ```

- The SQL query used in below examples retrieves a random city from the MySQL sample `world` database

  ```sql
  SELECT city.Name as City,country.name as Country,city.District,city.Population FROM city,country WHERE city.CountryCode = country.Code ORDER BY RAND() LIMIT 0,1;
  ```

## 2. Retrieve and print output to console

Code:

```js
const mysql = require('mysql2')
const query = 'SELECT city.Name as City,country.name as Country,city.District,city.Population FROM city,country WHERE city.CountryCode = country.Code ORDER BY RAND() LIMIT 0,1'
const connectionConfig = {
  host: 'localhost',
  user: 'cityapp',
  password: 'Cyberark1',
  database: 'world'
}
const connection = mysql.createConnection(connectionConfig)
connection.connect(function(err){
  if (err) throw err
  connection.query(query,function(err,results,fields){
    if (err) throw err
    for(row of results){
      console.log(row)
    }
  })
  connection.end()
})
```

> If using `mysql_clear_password` authentication plugin, the `connectionConfig` block needs to include the `mysql_clear_password` under `authPlugins`:
> 
> ```js
> const connectionConfig = {
>   host: 'localhost',
>   user: 'cityapp',
>   password: 'Cyberark1',
>   database: 'world',
>   authPlugins: {
>     mysql_clear_password: () => () => {
>       return Buffer.from(password + '\0')
>     }
>   }
> }
> ```

Output:

```console
[root@foxtrot ~]# node cityapp.js
{
  City: 'Safi',
  Country: 'Morocco',
  District: 'Doukkala-Abda',
  Population: 262300
}
```

## 3. Using express web framework listener

Code:

```js
const mysql = require('mysql2')
const query = 'SELECT city.Name as City,country.name as Country,city.District,city.Population FROM city,country WHERE city.CountryCode = country.Code ORDER BY RAND() LIMIT 0,1'
const connectionConfig = {
  host: 'localhost',
  user: 'cityapp',
  password: 'Cyberark1',
  database: 'world'
}
const express = require('express')
const app = express()
app.get('/',function(request,response){-
  response.setHeader('Access-Control-Allow-Origin', '*')
  getCity(request,response)
})
app.listen(8080)
const connection = mysql.createConnection(connectionConfig)
function getCity(request,response) {
  connection.connect(function(err){
    if (err) {
      response.json({'code':300, 'status':'Database connection errror', 'error':err.message})
      return
    }
    connection.query(query,function(err,results,fields){
      if(!err) {
        response.json(results[0])
      }
    })
  })
}
```

### 3.1. Using connection pool

Change `connection` object and `getCity` function from above to the code below:

```js
const pool = mysql.createPool(connectionConfig)
function getCity(request,response) {
  pool.getConnection(function(err,connection){
    if (err) {
      response.json({'code':300, 'status':'Database connection errror', 'error':err.message})
      return
    }
    connection.query(query,function(err,rows){
      connection.release()
      if(!err) {
        response.json(rows[0])
      }
    })
    connection.on('error', function(err){
      response.json({'code':300, 'status':'Database connection errror', 'error':err.message})
      return
    })
  })
}
```

### 3.2. Running the listener

To run listener in background: `node cityapp.js > /dev/null 2>&1 &`
- `> /dev/null 2>&1` sends all `stdout` and `stderr` to `/dev/null`
- `&` runs the task in background
- Background jobs controls:
  - `jobs`: lists the background jobs
  - `fg`: brings the background job to foreground
  - `Ctrl+Z`: sends the foreground to background in paused state
  - `bg`: gets the background job to continue running in background

Output:

```console
[root@foxtrot ~]# node cityapp.js > /dev/null 2>&1 &
[1] 3877
[root@foxtrot ~]# jobs
[1]+  Running                 node cityapp.js > /dev/null 2>&1 &
[root@foxtrot ~]# fg
node cityapp.js > /dev/null 2>&1
^Z
[1]+  Stopped                 node cityapp.js > /dev/null 2>&1
[root@foxtrot ~]# bg
[1]+ node cityapp.js > /dev/null 2>&1 &
[root@foxtrot ~]# jobs
[1]+  Running                 node cityapp.js > /dev/null 2>&1 &
```

### 3.3. Example listener query output

```powershell
PS C:\Users\Administrator> Invoke-RestMethod -Method Get -Uri 'http://foxtrot.vx:8080' -ContentType application/json

City   Country  District     Population
----   -------  --------     ----------
Pinang Malaysia Pulau Pinang     219603
```

## 4. Using callbacks

Instead of `console.log` within the `connection.query` block, the output of the query can be sent to another block for processing using callbacks

Code:

```js
const mysql = require('mysql2')
const query = 'SELECT city.Name as City,country.name as Country,city.District,city.Population FROM city,country WHERE city.CountryCode = country.Code ORDER BY RAND() LIMIT 0,1'
const connectionConfig = {
  host: 'localhost',
  user: 'cityapp',
  password: 'Cyberark1',
  database: 'world'
}
function getCity(cb){
  const connection = mysql.createConnection(connectionConfig)
  connection.connect(function(err){
    if (err) throw err
  })
  connection.query(query,function(err,results){
    if (err) throw err
    cb(results[0])
  })
  connection.end()
}
getCity(function(response){
  console.log(response)
})
```

### 4.1. Using callbacks + handlers

One scenario to use callbacks is when the query block needs to be exported as a handler (e.g. in AWS Lambda)

Handler: `cityapp.js`:

```js
const mysql = require('mysql2')
const query = 'SELECT city.Name as City,country.name as Country,city.District,city.Population FROM city,country WHERE city.CountryCode = country.Code ORDER BY RAND() LIMIT 0,1'
const connectionConfig = {
  host: 'localhost',
  user: 'cityapp',
  password: 'Cyberark1',
  database: 'world'
}
exports.handler = function(cb){
  const connection = mysql.createConnection(connectionConfig)
  connection.connect(function(err){
    if (err) throw err
  })
  connection.query(query,function(err,results){
    if (err) throw err
    cb(results[0])
  })
  connection.end()
}
```

Runner: `run.js`:

```js
cityapp = require('./cityapp.js')
cityapp.handler(function(response){
  console.log(response)
})
```

## 5. Using promises

The `mysql2` npm also [supports promises](https://www.npmjs.com/package/mysql2#using-promise-wrapper)

Code:

```js
const mysql = require('mysql2/promise')
const query = 'SELECT city.Name as City,country.name as Country,city.District,city.Population FROM city,country WHERE city.CountryCode = country.Code ORDER BY RAND() LIMIT 0,1'
const connectionConfig = {
  host: 'localhost',
  user: 'cityapp',
  password: 'Cyberark1',
  database: 'world'
}
async function getCity(){
  const connection = await mysql.createConnection(connectionConfig)
  const [rows,fields] = await connection.execute(query)
  connection.end()
  return rows[0]
}
getCity().then(data => console.log(data))
```

### 5.1. Using promises + handlers

With promises, both the handler and runner codes can be simplified

Handler: `cityapp.js`:

```js
const mysql = require('mysql2/promise')
const query = 'SELECT city.Name as City,country.name as Country,city.District,city.Population FROM city,country WHERE city.CountryCode = country.Code ORDER BY RAND() LIMIT 0,1'
const connectionConfig = {
  host: 'localhost',
  user: 'cityapp',
  password: 'Cyberark1',
  database: 'world'
}
exports.handler = async function(){
  const connection = await mysql.createConnection(connectionConfig)
  const [rows,fields] = await connection.execute(query)
  connection.end()
  return rows[0]
}
```

Runner: `run.js`:

```js
cityapp = require('./cityapp.js')
cityapp.handler().then(data => console.log(data))
```
