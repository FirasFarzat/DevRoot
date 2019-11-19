#Watchify, Babel, and Uglify
Now that we are bundling our code with Browserify and tsify, we can add various features to our build with browserify plugins.

- Watchify starts gulp and keeps it running, incrementally compiling whenever you save a file. This lets you keep an edit-save-refresh cycle going in the browser.

- Babel is a hugely flexible compiler that converts ES2015 and beyond into ES5 and ES3. This lets you add extensive and customized transformations that TypeScript doesn’t support.

- Uglify compacts your code so that it takes less time to download.

##Watchify 
We’ll start with Watchify to provide background compilation:

```node
npm install --save-dev watchify fancy-log

```

Now change your gulpfile to the following:

```typescript
var gulp = require('gulp');
var browserify = require('browserify');
var source = require('vinyl-source-stream');
var watchify = require('watchify');
var tsify = require('tsify');
var fancy_log = require('fancy-log');
var paths = {
    pages: ['src/*.html']
};

var watchedBrowserify = watchify(browserify({
    basedir: '.',
    debug: true,
    entries: ['src/main.ts'],
    cache: {},
    packageCache: {}
}).plugin(tsify));

gulp.task('copy-html', function () {
    return gulp.src(paths.pages)
        .pipe(gulp.dest('dist'));
});

function bundle() {
    return watchedBrowserify
        .bundle()
        .on('error', fancy_log)
        .pipe(source('bundle.js'))
        .pipe(gulp.dest('dist'));
}

gulp.task('default', gulp.series(gulp.parallel('copy-html'), bundle));
watchedBrowserify.on('update', bundle);
watchedBrowserify.on('log', fancy_log);

```
There are basically three changes here, but they require you to refactor your code a bit.
- We wrapped our browserify instance in a call to watchify, and then held on to the result.
- We called watchedBrowserify.on('update', bundle); so that Browserify will run the bundle function every time one of your TypeScript files changes.
- We called watchedBrowserify.on('log', fancy_log); to log to the console.

Together (1) and (2) mean that we have to move our call to browserify out of the default task. And we have to give the function for default a name since both Watchify and Gulp need to call it. Adding logging with (3) is optional but very useful for debugging your setup.

Now when you run Gulp, it should start and stay running. Try changing the code for showHello in main.ts and saving it. You should see output that looks like this:


```output 
proj$ gulp
[10:34:20] Using gulpfile ~/src/proj/gulpfile.js
[10:34:20] Starting 'copy-html'...
[10:34:20] Finished 'copy-html' after 26 ms
[10:34:20] Starting 'default'...
[10:34:21] 2824 bytes written (0.13 seconds)
[10:34:21] Finished 'default' after 1.36 s
[10:35:22] 2261 bytes written (0.02 seconds)
[10:35:24] 2808 bytes written (0.05 seconds)
```

##Uglify 
First install Uglify. Since the point of Uglify is to mangle your code, we also need to install vinyl-buffer and gulp-sourcemaps to keep sourcemaps working.

```node
npm install --save-dev gulp-uglify vinyl-buffer gulp-sourcemaps

```
Now change your gulpfile to the following:
```typescript
var gulp = require('gulp');
var browserify = require('browserify');
var source = require('vinyl-source-stream');
var tsify = require('tsify');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');
var buffer = require('vinyl-buffer');
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
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
    .pipe(uglify())
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('dist'));
}));

```

Notice that uglify itself has just one call — the calls to buffer and sourcemaps exist to make sure sourcemaps keep working. These calls give us a separate sourcemap file instead of using inline sourcemaps like before. Now you can run Gulp and check that bundle.js does get minified into an unreadable mess:

```node
gulp
cat dist/bundle.js
```

#Babel
First install Babelify and the Babel preset for ES2015. Like Uglify, Babelify mangles code, so we’ll need vinyl-buffer and gulp-sourcemaps. By default Babelify will only process files with extensions of .js, .es, .es6 and .jsx so we need to add the .ts extension as an option to Babelify.

```node
npm install --save-dev babelify@8 babel-core babel-preset-es2015 vinyl-buffer gulp-sourcemaps
```
Now change your gulpfile to the following:

```typescript
var gulp = require('gulp');
var browserify = require('browserify');
var source = require('vinyl-source-stream');
var tsify = require('tsify');
var sourcemaps = require('gulp-sourcemaps');
var buffer = require('vinyl-buffer');
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
    .transform('babelify', {
        presets: ['es2015'],
        extensions: ['.ts']
    })
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('dist'));
}));

```

We also need to have TypeScript target ES2015. Babel will then produce ES5 from the ES2015 code that TypeScript emits. Let’s modify tsconfig.json:

```json
{
    "files": [
        "src/main.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": true,
        "target": "es2015"
    }
}

```

Babel’s ES5 output should be very similar to TypeScript’s output for such a simple script.

