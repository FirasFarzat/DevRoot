#Browserify
ow let’s move this project from Node to the browser. To do this, we’d like to bundle all our modules into one JavaScript file. Fortunately, that’s exactly what Browserify does. Even better, it lets us use the CommonJS module system used by Node, which is the default TypeScript emit. That means our TypeScript and Node setup will transfer to the browser basically unchanged.

First, install browserify, tsify, and vinyl-source-stream. tsify is a Browserify plugin that, like gulp-typescript, gives access to the TypeScript compiler. vinyl-source-stream lets us adapt the file output of Browserify back into a format that gulp understands called vinyl.

```powershell
npm install --save-dev browserify tsify vinyl-source-stream
```
Create a page
Create a file in src named index.html:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello World!</title>
    </head>
    <body>
        <p id="greeting">Loading ...</p>
        <script src="bundle.js"></script>
    </body>
</html>

```

Now change main.ts to update the page:

```typescript
import { sayHello } from './greet';

function showHello(divName: string, name: string) {
    const elt = document.getElementById(divName);
    elt.innerText = sayHello(name);
}

showHello('greeting', 'TypeScript');

```

Calling showHello calls sayHello to change the paragraph’s text. Now change your gulpfile to the following:

```typescript
var gulp = require('gulp');
var browserify = require('browserify');
var source = require('vinyl-source-stream');
var tsify = require('tsify');
var paths = {
    pages: ['src/*.html']
};

gulp.task('copy-html', function () {
    return gulp.src(paths.pages)
        .pipe(gulp.dest('dist'));
});

gulp.task('default', gulp.series(gulp.parallel('copy-html'), function () {
    return browserify({
        basedir: '.',
        debug: true,
        entries: ['src/main.ts'],
        cache: {},
        packageCache: {}
    })
    .plugin(tsify)
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(gulp.dest('dist'));
}));

```

This adds the copy-html task and adds it as a dependency of default. That means any time default is run, copy-html has to run first. We’ve also changed default to call Browserify with the tsify plugin instead of gulp-typescript. Conveniently, they both allow us to pass the same options object to the TypeScript compiler.

After calling bundle we use source (our alias for vinyl-source-stream) to name our output bundle bundle.js.

Test the page by running gulp and then opening dist/index.html in a browser. You should see “Hello from TypeScript” on the page.

```powershell
gulp
dist/index.html
```

Notice that we specified debug: true to Browserify. This causes tsify to emit source maps inside the bundled JavaScript file. Source maps let you debug your original TypeScript code in the browser instead of the bundled JavaScript. You can test that source maps are working by opening the debugger for your browser and putting a breakpoint inside main.ts. When you refresh the page the breakpoint should pause the page and let you debug greet.ts.