Let’s write a Hello World program. In src, create the file main.ts:
```typescript
function hello(compiler: string) {
    console.log(`Hello from ${compiler}`);
}
hello('TypeScript');

```

In the project root, proj, create the file tsconfig.json 
```typescript
{
    "files": [
        "src/main.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es5"
    }
}

```
Create a gulpfile.js
In the project root, create the file gulpfile.js:
```typescript
var gulp = require('gulp');
var ts = require('gulp-typescript');
var tsProject = ts.createProject('tsconfig.json');

gulp.task('default', function () {
    return tsProject.src()
        .pipe(tsProject())
        .js.pipe(gulp.dest('dist'));
});

```

#### Test the resulting app #

```powershell
gulp
node dist/main.js

```
The program should print “Hello from TypeScript!”.

