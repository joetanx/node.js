## 1. Functions

Exploring node.js as a newbie can be confusing when you encounter codes like these:

- Example: simple number check

  > ❔: `largeNum` is assigned to `num` and pointed to `num > 3`

  ```js
  const largeNum = num => num > 3
  ```

  > Ref: https://www.scaler.com/topics/callback-hell-in-javascript/

- Example: AWS IAM authentication for database access

  > ❔: `mysql_clear_password` is assigned to empty `()` and pointed to empty `()` then pointed to function `signer.getAuthToken()`

  - Ref: https://docs.aws.amazon.com/lambda/latest/dg/configuration-database.html

    ```js
    authPlugins: { mysql_clear_password: () => () => signer.getAuthToken() }
    ```

  - Ref: https://stackoverflow.com/questions/58067254/node-mysql2-aws-rds-signer-connection-pooling

    ```js
    authPlugins: {
      mysql_clear_password: () => () =>
        signer.getAuthToken({
          region: '...',
          hostname: '...',
          port: '...',
          username: '...'
        })
    }
    ```

### 1.1. Function expressions

Functions in node.js can be expressed in several ways

#### Traditional function expression

```js
function getSum(a,b){
  return a+b
}
console.log(getSum(2,3))
```

#### Arrow function expression

Refs:
- https://www.w3schools.com/js/js_arrow_function.asp
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions

The arrow function expression simply can be a drop in replace of `function getSum(a,b)` to `const getSum = (a,b) =>`

```js
const getSum = (a,b) => {
  return a+b
}
console.log(getSum(2,3))
```

The code can be even shorter when:
1. If the function has only one statement, and the statement returns a value, the brackets and the return keyword can be removed
2. If the function takes only one parameter, the parentheses can be skipped as well

This reults in the following one-liner function

```js
const getSum = (a,b) => a+b
console.log(getSum(2,3))
```

## 2. Variables

### 2.1. Declare variables without keywords

Ref: https://www.tutorialsteacher.com/javascript/javascript-variable

Variables in node.js can be declared with `var`, `let` or `const`, depending on the scope of the variable

Variables can also be declared without keywords; they become global variables, irrespective of where they are declared

The above example will then be:

```js
getSum = (a,b) => a+b
console.log(getSum(2,3))
```

### 2.2. Scope

Variables declared as `var` or `let` outside a block can be updated anywhere

Code:

```js
let something = 'this'
console.log('Something declared as: '+something)
const updateSomething = newValue => something = newValue
updateSomething('that')
console.log('Something updated to: '+something)
```

Output:

```
Something declared as: this
Something updated to: that
```
