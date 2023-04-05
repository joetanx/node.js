## 1. Callbacks

A callback is a function which is passed as argument to another function

### 1.1. Callbacks concept

Consider the example below:

```js
function getSum(a,b,logResult) {
  logResult('something')
  console.log('Checkpoint 3')
  logResult(a+b)
  console.log('Checkpoint 4')
  logResult(a*b)
}
console.log('Checkpoint 1')
getSum(2,3,function(response){
  console.log('Result: '+response)
})
console.log('Checkpoint 2')
```

- The first code executed is `console.log('Checkpoint 1')` which corresponds to **output line 1**
- The code then calls the `getSum` function, so the execution flow now goes into the `getSum` function
  - Line 1 in `getSum` calls the function which was passed from the caller (hence, "callback"), with `something` as the argument; this corresponds to **output line 2**
  - Line 2 in `getSum` logs `Checkpoint 3`, which corresponds to **output line 3**
  - Line 3 in `getSum` calls the function which was passed from the caller, this time with `a+b` as the argument; this corresponds to **output line 4**
  - Line 4 in `getSum` logs `Checkpoint 4`, which corresponds to **output line 5**
  - Line 5 in `getSum` calls the function which was passed from the caller, this time with `a*b` as the argument; this corresponds to **output line 6**
- The execution flow now redirects back to the main block and executes `console.log('Checkpoint 2')`, which corresponds to **output line 7**

Output:

```console
Checkpoint 1
Result: something
Checkpoint 3
Result: 5
Checkpoint 4
Result: 6
Checkpoint 2
```

> The functions can also be written in arrow notation
> 
> ```js
> const getSum = (a,b,logResult) => {
>   logResult('something')
>   console.log('Checkpoint 3')
>   logResult(a+b)
>   console.log('Checkpoint 4')
>   logResult(a*b)
> }
> console.log('Checkpoint 1')
> getSum(2,3,response => console.log('Result: '+response))
> console.log('Checkpoint 2')
> ```

### 1.2. Asynchronus callbacks

Callbacks allows asynchronous code execution: the execution flow can continue while the callbacks run asynchronously in the background

This is useful to allow functions that requires more time (such as web API call or file read) to run asynchronously in the background without holding up the code execution flow

Consider the `getSum` example changing into the `getSumAsync` function below:

- `logResult(a+b)` and `logResult(a*b)` are simulated long running function using `setTimeout` with 200ms and 100ms respectively

```js
function getSumAsync(a,b,logResult){
  logResult('something')
  console.log('Checkpoint 3')
  setTimeout(function(){
    logResult(a+b)
  },200)
  console.log('Checkpoint 4')
  setTimeout(function(){
    logResult(a*b)
  },100)
}
console.log('Checkpoint 1')
getSumAsync(2,3,function(response)  
{
    console.log('Result: '+response)
})
console.log('Checkpoint 2')
```

- The first code executed is `console.log('Checkpoint 1')` which corresponds to **output line 1**
- The code then calls the `getSumAsync` function, so the execution flow now goes into the `getSum` function
  - Line 1 in `getSum` calls the function which was passed from the caller (hence, "callback"), with `something` as the argument; this corresponds to **output line 2**
  - Line 2 in `getSum` logs `Checkpoint 3`, which corresponds to **output line 3**
  - Line 3 in `getSum` calls the first `setTimeout` function which has a delay of **200ms**
  - The first `setTimeout` function continues to run in background, and the execution flow continues with line 4 in `getSum` logs `Checkpoint 4`, which corresponds to **output line 4**
  - Line 5 in `getSum` calls the second `setTimeout` function which has a delay of **100ms**
- The second `setTimeout` function continues to run in background, and the execution flow now redirects back to the main block and executes `console.log('Checkpoint 2')`, which corresponds to **output line 5**
- After **100ms**, the **second** `setTimeout` function completes (**before the first** `setTimeout` function) and calls back to the logResult function, which corresponds to **output line 6**
- Finally, after **200ms**, the **first** `setTimeout` function completes and calls back to the logResult function, which corresponds to **output line 7**

Output:

```console
Checkpoint 1
Result: something
Checkpoint 3
Checkpoint 4
Checkpoint 2
Result: 6
Result: 5
```

### 1.3. File read example

☝️ Using `readFileSync()` and `readFile()` with `utf-8` option returns the content with UTF-8 character encoding, if `utf-8` is not specified, the output returned withh be of `Buffer` data type

☝️ The content read has a newline appended, `.trim()` removes the newline appended

#### 1.3.1. Synchronous

Example file `sync.txt`: `This is synchronous example`

Code:

```js
var fs = require('fs');
var dataRead = fs.readFileSync('sync.txt','utf-8')
console.log(dataRead.trim())
console.log('Checkpoint')
```

Output:

```console
This is synchronous example
Checkpoint
```

#### 1.3.2. Asynchronous

Example file `async.txt`: `This is asynchronous example`

Code:

```js
var fs = require('fs')
fs.readFile('async.txt','utf-8',function(err,dataRead){  
  if (err) return console.error(err)
  console.log(dataRead.trim())
})
console.log('Checkpoint')
```

Output:

```console
Checkpoint
This is asynchronous example
```

### 1.4. Callback hell example

Refs:
- https://www.knowledgehut.com/blog/web-development/nodejs-call-backs
- https://www.scaler.com/topics/callback-hell-in-javascript/

Callback hell describes when multiple callbacks are nested within a function is called a callback hell; this makes the code very difficult to understand and maintain

Example files:

- `asia.json`:

  ```json
  {
    "one": "southernAndCentralAsia.json",
    "two": "middleEast.json",
    "three": "southEastAsia.json",
    "four": "easternAsia.json"
  }
  ```

- `southEastAsia.json`:

  ```json
  {
    "one": "brunei.json",
    "two": "indonesia.json",
    "three": "cambodia.json",
    "four": "laos.json",
    "five": "myanmar.json",
    "six": "malaysia.json",
    "seven": "philippines.json",
    "eight": "singapore.json",
    "nine": "thailand.json",
    "ten": "eastTimor.json",
    "eleven": "vietnam.json"
  }
  ```

- `singapore.json`:

  ```json
  {
    "name": "Singapore",
    "countryCode": "SGP",
    "surfaceArea": "728",
    "indepYear": "1965"
  }
  ```

Code:

- The output from the outermost callback is returned to `region`, which is parsed and passed into the middle callback
- The output from the middle callback is returned to `country`, which is parsed and passed into the innermost callback
- Finally, the innermost callback reads the last file and output to console
- The code is already hard to read with 3 levels of nested callback, nesting more callbacks will indeed be a `callback hell`

```js
var cbh = require('fs')
cbh.readFile('asia.json','utf-8',function(err,region){
  if (err) return console.error(err)
  cbh.readFile(JSON.parse(region).three, function(err,country){
    if (err) return console.error(err)
    cbh.readFile(JSON.parse(country).eight, function(err,singapore){
      if (err) return console.error(err)
      console.log(JSON.parse(singapore).countryCode)
    })
    console.log('Checkpoint 1')
  })
  console.log('Checkpoint 2')
})
console.log('Checkpoint 3')
```

Output:

Also notice the reverse order of the checkpoints compared to the order that is is written in the code → this is how asynchronous execution works in node.js

```console
Checkpoint 3
Checkpoint 2
Checkpoint 1
SGP
```

## 2. Promises

### 2.1. Creating promises

The three functions in the callback hell examples can be converted into promises

Code:
- A promise can be declared as-is if it doesn't take any arguments (e.g. `getRegion`)
- A promise can also be returned by a function to work on arguments provided to the function (e.g. `getCountry` and `getCode`)

```js
const fs = require('fs')
const getRegion = new Promise((resolve,reject) => {
  dataRead = fs.readFileSync('asia.json','utf-8')
  resolve(JSON.parse(dataRead)['three'])
})
function getCountry(file,element) {
  return new Promise((resolve,reject) => {
    dataRead = fs.readFileSync(file)
    resolve(JSON.parse(dataRead)[element])
  })
}
function getCode(file,element) {
  return new Promise((resolve,reject) => {
    dataRead = fs.readFileSync(file)
    resolve(JSON.parse(dataRead)[element])
  })
}
```

### 2.2. Consuming promises

Code:

```js
getRegion.then(region => console.log(region))
getCountry('southEastAsia.json','eight').then(country => console.log(country))
getCode('singapore.json','countryCode').then(countryCode => console.log(countryCode))
```

Output:

```
southEastAsia.json
singapore.json
SGP
```

### 2.3. Chaining Promises

Ref: https://blog.logrocket.com/guide-promises-node-js/

Code:

```js
getRegion.then(region => {
  return region
})
.then(region => {
  getCountry(region,'eight').then(country => {
    return country
  })
  .then(country => {
    getCode(country,'countryCode').then(code => {
      console.log(code)
    })
  })
})
```

or collapsing the code slightly:

```js
getRegion.then(region => {
  return region
}).then(region => {
  return getCountry(region,'eight').then(country => {
    return getCode(country,'countryCode').then(code => {
      console.log(code)
    })
  })
})
```

Output: `SGP`

### 2.4. async/await

Chaining promises doesn't seem to be any better than callback hell, but there is a better way to do it: `async`/`await`

Code:

```js
const getAnswer = async function () {
  region = await getRegion.then(region => {
    return region
  })
  country = await getCountry(region,'eight').then(country => {
    return country
  })
  code = await getCode(country,'countryCode').then(code => {
    return code
  })
  console.log(code)
}
getAnswer()
```

Output: `James`

### 2.5. Examples of how promises doesn't work

#### 2.5.1. Trying to assign results of promises to a variable directly

Example:

```js
let answer = getRegion.then(region => {
  return region
}).then(region => {
  return getCountry(region,'eight').then(country => {
    return getCode(country,'countryCode').then(code => {
      return code
    })
  })
})
console.log(answer)
```

Result: `Promise { <pending> }`

☝️ The object returned is a promise that has yet to be fulfilled 

#### 2.5.2. Trying to assign results of promises to a variable within the promise chain block

```js
let answer
getRegion.then(region => {
  return region
}).then(region => {
  return getCountry(region,'eight').then(country => {
    return getCode(country,'countryCode').then(code => {
      answer = code
      console.log('This is the inner result: '+answer)
    })
  })
})
console.log('This is the outer result: '+answer)
```

Result:

```
This is the outer result: undefined
This is the inner result: James
```

☝️ Remember that promises are asynchronous:
- The outer result was executed **before** the promise was fulfilled, and since `answer` was declared without a value, the data is `undefined`
- The inner result is **after** the promise was fulfilled, it has the correct answer, but the intention to assign the answer to the outer variable was not acheived

#### 2.5.3. Trying to assign results of promises to a variable using `await`

Example:

```js
let answer = await getRegion.then(region => {
  return region
}).then(region => {
  return getCountry(region,'eight').then(country => {
    return getCode(country,'countryCode').then(code => {
      return code
    })
  })
})
console.log(answer)
```

Result: `SyntaxError: await is only valid in async functions and the top level bodies of modules`

☝️ `await` can only be used in async functions
