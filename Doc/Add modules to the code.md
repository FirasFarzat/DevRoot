#Add modules to the code
Before we get to Browserify, let’s build our code out and add modules to the mix. This is the structure you’re more likely to use for a real app.

Create a file called src/greet.ts:
```typescript
export function sayHello(name: string) {
    return `Hello from ${name}`;
}

```
Now change the code in src/main.ts to import sayHello from greet.ts:

```typescript
import { sayHello } from './greet';

console.log(sayHello('TypeScript'));

```
Finally, add src/greet.ts to tsconfig.json:
```typescript
{
    "files": [
        "src/main.ts",
        "src/greet.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es5"
    }
}

```
Make sure that the modules work by running gulp and then testing in Node:
```typescript
gulp
node dist/main.js

```

```
Notice that even though we used ES2015 module syntax, TypeScript emitted CommonJS modules that Node uses. We’ll stick with CommonJS for this tutorial, but you could set module in the options object to change this.

```