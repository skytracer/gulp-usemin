# gulp-usemin-query
> Replaces references to non-optimized scripts or stylesheets into a set of HTML files (or any templates/views) and allows a query-like suffix to be added to file name.

This task is designed for gulp 3.
> Attention: v0.3.0 options does not compatible with v0.2.0.

## Usage

First, install `gulp-usemin` as a development dependency:

```shell
npm install --save-dev gulp-usemin-query
```

Then, add it to your `gulpfile.js`:

```javascript
var usemin = require('gulp-usemin-query');
var uglify = require('gulp-uglify');
var minifyHtml = require('gulp-minify-html');
var minifyCss = require('gulp-minify-css');
var rev = require('gulp-rev');

gulp.task('usemin', function() {
  gulp.src('./*.html')
    .pipe(usemin({
      css: [minifyCss(), 'concat'],
      html: [minifyHtml({empty: true})],
      js: [uglify(), rev()]
    }))
    .pipe(gulp.dest('build/'));
});
```

The difference from the usemin plugin is that allows a query string to be appended to file name
It can be used as static or dynamic in conjunction with gulp-bump/gulp-template

For example
```html
<!-- build:js js/lib.js?v=<%= libversion%> -->
..some lib files...
<!-- endbuild -->
<!-- build:js js/app.js?v=<%= appversion%>-->
..some app files...
<!-- endbuild -->
```
will be translated to
```html
    <script src="js/lib.js?v=0.10.6"></script>
    <script src="js/app.js?v=0.7.3"></script>
```
```javascript
var conf = {app: 'app/', dist: 'dist/'};
var pkg = require('./package.json');
gulp.task('usemin-test', function() {
  return gulp.src(conf.app + 'index.html')
  .pipe($('template')(pkg))
  .pipe($('usemin')())
  .pipe(gulp.dest(conf.dist));
});
```
For regular usemin, see below.

## API

### Blocks
Blocks are expressed as:

```html
<!-- build:<pipelineId>(alternate search path) <path> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

- **pipelineId**: pipeline id for options
- **alternate search path**: (optional) By default the input files are relative to the treated file. Alternate search path allows one to change that
- **path**: the file path of the optimized file, the target output

An example of this in completed form can be seen below:

```html
<!-- build:css style.css -->
<link rel="stylesheet" href="css/clear.css"/>
<link rel="stylesheet" href="css/main.css"/>
<!-- endbuild -->

<!-- build:js js/lib.js -->
<script src="../lib/angular-min.js"></script>
<script src="../lib/angular-animate-min.js"></script>
<!-- endbuild -->

<!-- build:js1 js/app.js -->
<script src="js/app.js"></script>
<script src="js/controllers/thing-controller.js"></script>
<script src="js/models/thing-model.js"></script>
<script src="js/views/thing-view.js"></script>
<!-- endbuild -->
```

### Options

#### assetsDir
Type: `String`

Alternate root path for assets. New concated js and css files will be written to the path specified in the build block, relative to this path. Currently asset files are also returned in the stream.

#### any pipelineId
Type: `Array`

If exist used for modify files. If does not contain string 'concat', then it added as first member of pipeline


## Use case

```
|
+- app
|   +- index.html
|   +- assets
|       +- js
|          +- foo.js
|          +- bar.js
|   +- css
|       +- clear.css
|       +- main.css
+- dist
```

We want to optimize `foo.js` and `bar.js` into `optimized.js`, referenced using relative path. `index.html` should contain the following block:

```
    <!-- build:css style.css -->
    <link rel="stylesheet" href="css/clear.css"/>
    <link rel="stylesheet" href="css/main.css"/>
    <!-- endbuild -->

    <!-- build:js js/optimized.js -->
    <script src="assets/js/foo.js"></script>
    <script src="assets/js/bar.js"></script>
    <!-- endbuild -->
```

We want our files to be generated in the `dist` directory. `gulpfile.js` should contain the following block:

```javascript
gulp.task('usemin', function(){
  gulp.src('./app/index.html')
    .pipe(usemin({
      js: [uglify()]
      // in this case css will be only concatenated (like css: ['concat']).
    }))
    .pipe(gulp.dest('dist/'));
});
```

This will generate the following output:

```
|
+- app
|   +- index.html
|   +- assets
|       +- js
|          +- foo.js
|          +- bar.js
+- dist
|   +- index.html
|   +- js
|       +- optimized.js
|   +- style.css
```

`index.html` output:

```
    <link rel="stylesheet" href="style.css"/>

    <script src="js/optimized.js"></script>
```

## Changelog

#####0.3.3
- fixed dependencies
- Add support for multiple alternative paths (by peleteiro)

#####0.3.2
- fixed assetsDir option (by rovjuvano)

#####0.3.1
- fixed fails to create source map files by uglify({outSourceMap: true})

#####0.3.0
- new version of options

#####0.2.3
- fixed html minify bug

#####0.2.2
- allow gulp-usemin to work with minified source HTML (by CWSpear)
- fixed alternate path bug (by CWSpear)
- add assetsDir option (by pursual)
- add rev option (by pursual)

#####0.2.1
- fixed subfolders bug

#####0.2.0
- no minification by default. New options API

#####0.1.4
- add alternate search path support

#####0.1.3
- add support for absolute URLs (by vasa-chi)

#####0.1.1
- fixed aggressive replace comments

#####0.1.0
- fixed some bugs. Add tests.

#####0.0.2
- add minification by default

#####0.0.1
- initial release
