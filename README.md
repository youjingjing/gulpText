#gulpSkills
var gulp  = require ('gulp'),
		$=require("gulp-load-plugins")();
var del = require('del');

// 解析css
gulp.task("css",function () {
  gulp.src("./app/assets/css/*.less")
  .pipe($.less())
  .pipe($.autoprefixer({
      browsers: ['last 2 versions', 'Android >= 4.0'],
      cascade: true, //是否美化属性值 默认：true
      remove:true //是否去掉不必要的前缀 默认：true
  }))
	.pipe($.rename({suffix: '.min'}))   //rename压缩后的文件名
  .pipe( gulp.dest("./app/assets/css/"))
});

//解析js
gulp.task("js",function(){
  gulp.src('./app/assets/javascripts/index.js')  
		.pipe($.babel({
				presets: ['es2015']
			}))
		.pipe($.rename({suffix: '.min'}))   //rename压缩后的文件名
		.pipe(gulp.dest('./app/assets/javascripts/'));  
});

//监听
gulp.watch([
  "./app/assets/css/*.less"
],["css","reload"])//关联文件

gulp.watch([
  "./app/assets/javascripts/*.js"
],["js","reload"])//关联文件

gulp.watch([
  "./app/index.html"
],["reload"])//关联文件

//热重载
gulp.task("reload",function () {
  gulp.src([
    "./app/assets/css/*.css",
    "./app/assets/javascripts/*.js",
    "./app/*.html"
    ])
  .pipe($.connect.reload())
})
//开启服务器
gulp.task("webserver",function () {
  $.connect.server({
    port : "8080",
    livereload : true,
    root: "./app"
  })
});

//mock server
gulp.task('mock', function() {
  gulp.src('.')
    .pipe($.mockServer({
      port: 8090
    }));
});

gulp.task("default",["css","js","reload","webserver","mock"]);











//打包
// 解析css
gulp.task("buildCss",function () {
  gulp.src("./app/assets/css/index.min.css")
  .pipe($.minifyCss() )
	.pipe($.rev()) //- 文件名加MD5后缀
  .pipe(gulp.dest("./dist/assets/css"))
	.pipe($.rev.manifest('rev-css-manifest.json'))
	.pipe(gulp.dest('./app/rev'))
});

//解析js
gulp.task("buildJs",function(){
  gulp.src(['./app/assets/javascripts/index.min.js'])
		.pipe($.uglify())
		.pipe($.rev()) //- 文件名加MD5后缀
		.pipe(gulp.dest('./dist/assets/javascripts/'))
		.pipe($.rev.manifest('rev-js-manifest.json')) //- 生成一个rev-manifest.json
		.pipe(gulp.dest('./app/rev')) 
});

gulp.task('rev',function() {
	gulp.src(['./app/rev/*.json', './app/index.html']) 
		.pipe($.revCollector())     //- 执行文件内css名的替换
		.pipe($.minifyHtml())
		.pipe(gulp.dest('./dist'));  //- 替换后的文件输出的目录
});



gulp.task("build",["buildCss","buildJs","rev"]);
