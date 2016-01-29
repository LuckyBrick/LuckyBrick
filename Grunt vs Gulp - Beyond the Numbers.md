# Grunt vs Gulp - 抛开数据

> Just when you think that you're in control,
> 
> Just when you think that you've got a hold,
> 
> Just when you get on a roll,
> 
> Here it goes, here it goes, here it goes again.
> 
> ***OK Go - Here It Goes Again***

前端开发的发展由Gulp延续，它是一个已经获得了绝大多数网络开发者认可的新的编译系统。

花了一些时间在阅读文档和试用Gulp之后，我决定将现有的项目上的Grunt迁移成Gulp。到目前为止，据我观察，在执行类似的任务时，Gulp比Grunt快超多。

抛开肤浅的速度对比，让我们进一步思考，去了解一些关于Grunt和Gulp的不同之处。

在这篇文章中，我将讨论：

- Gulp的简单介绍，以及他如何与Grunt比较
- 当在这两个工具之间进行选择时需要考虑的事情。

## 第一印象

我在使用Grunt的过程中遇到的其中一个痛点就是对于简单任务的过度配置。就拿样式编译为例。我的源文件采用SCSS编写，它们需要被编译成CSS文件，然后我想要通过一个自动前缀加载器来运行这些文件并添加兼容前缀。同时我还想在文件改变时就进行编译。

以下是我为了完成该任务而写的针对Grunt和Gulp的代码。

***Gruntfile.js***

<pre>
grunt.initConfig({
  sass: {
    dist: {
      files: [{
        cwd: 'app/styles',
        src: '**/*.scss',
        dest: '../.tmp/styles',
        expand: true,
        ext: '.css'
      }]
    }
  },
  autoprefixer: {
    options: ['last 1 version'],
    dist: {
      files: [{
        expand: true,
        cwd: '.tmp/styles',
        src: '{,*/}*.css',
        dest: 'dist/styles'
      }]
    }
  },
  watch: {
    styles: {
      files: ['app/styles/{,*/}*.scss'],
      tasks: ['sass:dist', 'autoprefixer:dist']
    }
  }
});
grunt.registerTask('default', ['styles', 'watch']);
</pre>

***Gulpfile.js***

<pre>
gulp.task('sass', function () {
  gulp.src('app/styles/**/*.scss')
    .pipe(sass())
    .pipe(autoprefixer('last 1 version'))
    .pipe(gulp.dest('dist/styles'));
});
gulp.task('default', function() {
  gulp.run('sass');
  gulp.watch('app/styles/**/*.scss', function() {
    gulp.run('sass');
  });
});
</pre>

正如你所见，在Gulp中，我们不需要临时的`.tmp`文件夹来存储编译过的、未添加前缀的CSS文件。这意味着更少的配置，同时减少了对I/O的消耗。

现在进行一些用时比较。在我的电脑上，Gulp只需 ***2.13ms*** 就能进行文件修改，而对于Grunt则需要 ***1.298s***

***Grunt展示的耗时数据***

![Alt text](http://jaysoo.ca/images/grunt-compile-2.png)

***Gulp展示的耗时数据***

![Alt text](http://jaysoo.ca/images/gulp-compile.png)

因为两者采用不同的计算方法，所以用时报告并没有可比性。如果我在二者的任务（SASS编译和自动添加前缀）中都添加`time`方法，则得到的数据更接近：Gulp是 ***0.641ms*** ，Grunt是 ***1.235s***。当然，这也包括了两个工具的启动时间，所以这也不是一个完美的对比。

## 继续深入

要理解Gulp你还需要理解Node的流概念。所有的Gul插件都是通过流的读入数据和输出数据实现的。而这一切都是在内存中进行的，通过将一个流管道的输出作为另一个的输入。与Unix中得管道很像。

这让Gulp比Grunt有巨大的速度优势，因为I/O相对于内存操作实在是太耗性能了。更重要的是，尽管只有一个文件修改Grunt也会编译所有的文件，这就增加了额外的构建时间。

***在Grunt中，我们不得不往硬盘上写临时文件***

![Alt text](http://jaysoo.ca/images/grunt-flow-2.png)

***在Gulp中，我们将临时文件在内存中传递给其他流***

![Alt text](http://jaysoo.ca/images/gulp-flow.png)

这也说明了Gulp插件其实只是映射流。与Grunt相比，Grunt有各种各样的插件，比如启动一个页面自动刷新服务器。若使用Gulp，你将需要做一些Node编程的功课来完成这件事了。

[gulp-reload](https://npmjs.org/package/gulp-livereload)中的例子：

<pre>
var lr = require('tiny-lr'),
    gulp = require('gulp'),
    less = require('gulp-less'),
    livereload = require('gulp-livereload'),
    server = lr();

gulp.task('less', function () {
  gulp.src('less/*.less')
    .pipe(less())
    .pipe(gulp.dest('css'))
    .pipe(livereload(server));
});

gulp.task('watch', function () {
  server.listen(35729, function (err) {
    if (err) return console.log(err);

    gulp.watch('less/*.less', function () {
        gulp.run('less');
    });
  });
});
</pre>

## 好，那么哪个更好呢？

不幸的是，我无法告诉你哪个工具更好，因为这属于个人偏好问题。

我希望一旦Grunt0.5上线则Grunt和Gulp的**速度差距**会更加接近。[Grunt0.5的路线图](https://github.com/gruntjs/grunt-docs/blob/master/Roadmap.md)包含了在处理多任务时的管道数据支持和触发任务输出时采用数据事件。这样可以显著的加快任务执行，同时意味着由于不需要临时文件而配置变得更加精简。

目前Grunt超过Gulp的一个优点是更广泛的**社区支持**，产生了大量的可用插件。当然，这在将来可能会改变，由于更多的人采用Gulp，但要记住Gulp插件与Grunt插件有很大的不同，所以不要幻想会有插件一模一样的迁移到Gulp。

**最重要的问题**是问你自己更倾向哪种理念？你是想使用代码而不是配置来构建一个系统？如果是，那么选Gulp就对了；若不是，继续用Grunt吧。


## 延伸阅读与信息

- [GulpJS](http://gulpjs.com/)
- [GruntJS](http://gruntjs.com/)
- [Gulp, Grunt, Whatever](http://blog.ponyfoo.com/2014/01/09/gulp-grunt-whatever)
- [And just like that Grunt and RequireJS are out, it’s all about Gulp and Browserify now](http://www.100percentjs.com/just-like-grunt-gulp-browserify-now/)
- [Speedtesting gulp.js and Grunt](http://tech.tmw.co.uk/2014/01/speedtesting-gulp-and-grunt/)

*第一次编辑于2014/01/28: 我添加了对比的时间数据，尽管这并不是我在这篇文章中的重点。*

*第二次编辑于2014/01/30: 一个读者指出grunt-sass比grunt-contrib-compass更适合作对比，因为前者采用的是与gulp-sass一样的node-sass。我已经对这次修改的影响进行了数据更新。*
