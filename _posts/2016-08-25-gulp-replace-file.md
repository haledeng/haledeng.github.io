---
layout:  post
title:  "gulp-replace-file"
date:   2016-08-25
---

# gulp-replace-file
A gulp plugin to parse reference files in JS files. Different files will be parsed into different contents.
+ Template files (with a file extension name `.html` or `.tpl`) => JS functions.
+ Css files => string.
+ Other files => its file content.

### Install
```
npm i gulp-replace-file
```

### Usage
```
// __inline is the key function to reference file, which can be configed in gulpfile.
var htmlFunc = __inline('./temp.html');
var htmlStr = htmlFunc(data);
var cssText = __inline('./test.css');
```

### Config in gulpfile
```
var replace = require('gulp-replace-file');
gulp.task('replace', function() {
	gulp.src('replace/test.js')
		.pipe(replace())
		.pipe(gulp.dest('./build/'))
});
```

### config option
**prefix** : The key value to reference file. `__inline` is a default value.