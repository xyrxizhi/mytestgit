#Gradle
基于groovy核心语法提供想用api的脚本语言
对于gradle而言，project和module都可以视为project
###project相关api
|api|作用|
|---|---|
|getAllprojects()|获取工程中所有的project（包括根project与子project）|
|getSubProjects()|获取当前project下，所有的子project（在不同的project下调用，结果会不一样，可能返回null）|
|getParent()|获取当前project的父project（若在rooProject的build.gradle调用，则返回null）|
|getRootProject()|获取项目的根project（一定不会为null）|
|project(String path, Closure configureClosure)|根据path找到project，通过闭包进行配置（闭包的参数是path对应的Project对象）|
|allprojects(Closure configureClosure)|配置当前project和其子project的所有project|
|subprojects(Closure configureClosure)|配置子project的所有project（不包含当前project）|
###属性相关api
####1、在gradle脚本文件中使用ext块扩展属性
在ext块中可以定义的扩展属性，子project可以直接访问
####2、在gradle.properties文件中扩展属性
在gradle.properties的属性可以直接访问，但是得到的数据类型为object，一般需要类型转换。
###文件相关api
|api|作用|
|---|---|
|getRootDir()|获取rootProject目录|
|getBuildDir()|获取当前project的build目录（每个project都有自己的build目录）|
|getProjectDir()|获取当前project目录|
|File file(Object path)|定位一个文件，相对于当前project开始查找|
|ConfigurableFileCollection files(Object... paths)|定位多个文件，与file类似|
|copy(Closure closure)|拷贝文件|
|fileTree(Object baseDir, Closure configureClosure)|定位一个文件树（目录+文件），可对文件树进行遍历|
###依赖相关api
```
buildscript {
    // 配置工程仓库地址
    repositories {
        jcenter()
        mavenCentral()
        mavenLocal()
        ivy {}
        maven {
            name 'name'
            url 'url'
            credentials {
                username = 'username'
                password = 'password'
            }
        }
    }
    // 配置工程的"插件"（编写gradle脚本使用的第三方库）依赖地址
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
        classpath 'com.tencent.tinker-patch-gradle-plugin:1.7.7'
    }
}
```
配置应用程序第三方库依赖

```// app : build.gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar']) // 依赖文件树
    // compile file() // 依赖单个文件
    // compile files() // 依赖多个文件
    compile 'com.android.support:appcompat-v7:26.1.0' // 依赖仓库中的第三方库（即：远程库）
    compile project('mySDK') { // 依赖工程下其他Module（即：源码库工程）
      exclude module: 'support-v4' // 排除依赖：排除指定module
      exclude group: 'com.android.support' // 排除依赖：排除指定group下所有的module
      transitive false // 禁止传递依赖，默认值为false
    }
  
    // 栈内编译
    provided('com.tencent.tinker:tinker-android-anno:1.9.1')
}
```
依赖包只在编译器起作用
被依赖的工程中已经有了相同版本的第三方库，为了避免重复引用，可以使用provided
###外部命令api
```
// copyApk任务：用于将app工程生成出来apk目录及文件拷贝到本机下载目录
task('copyApk') {
    doLast {
        // gradle的执行阶段去执行
        def sourcePath = this.buildDir.path + '/outputs/apk'
        def destinationPath = '/Users/lqr/Downloads'
        def command = "mv -f ${sourcePath} ${destinationPath}"
        // exec块代码基本是固定的
        exec {
            try {
                executable 'bash'
                args '-c', command
                println 'the command is executed success.'
            }catch (GradleException e){
                println 'the command is executed failed.'
            }
        }
    }
}
```
###task相关
####创建及配置
通过task函数或者TaskContainer创建
```
task testtask{
    println 'task'
}
this.task.create(name:'testtask'){
    println 'task'
}
```
通过task函数配置
```
// 给Task指定分组与描述
task testtask(group: 'study', description: 'task study'){
  ...
}
task testtask {
  group 'study' // setGroup('study')
  description 'task study' // setDescription('task study')
  ...
}
```
Task除了可以配置group、description外，还可以配置name、type、dependsOn、overwrite、action
添加分组和描述方便查找和可读
####执行相关
doFirst和doLast
```
task testtask{
    doFirst{
        println 'first'
    }
}

task testtask.doLast{
    println 'last'
}//外部指定first会比内部指定的first先执行
```
doFirst和doLast可以定义多个
task会在配置阶段执行，doFirst和doLast可以对原有的task扩展
####task的执行顺序
#####1、依赖关系
task按照task的依赖关系建树执行。
依赖关系可以直接静态指定也可以动态指定
```
task taskZ(dependsOn: [taskX, taskY]) {}//静态
task taskZ() {
    dependsOn this.tasks.findAll {    // 依赖所有以lib开头的task
        task -> return task.name.startsWith('lib')
    }
}
```
#####2.输入输出指定顺序
每个task的input可以是任意类型，output只能是文件
#####3、api指定顺序
```
    mustRunAfter taskX强制在taskX后执行
    shouldRunAfter taskX不强制，建议在taskX后执行
```
####task可以挂接到生命周期
```
this.afterEvaluate { Project project ->
    def buildTask = project.tasks.getByName('build')
    if (buildTask == null) throw GradleException('the build task is not found')
    buildTask.doLast {
        taskZ.execute()
    }
}
```
###其他模块
####setting类
settings.gradle决定那些工程需要被处理
```
include ':Demo'
```
也可以用来添加本地依赖
####sourceSets类
设置文件默认位置，告知gradle资源所在的路径
可调用多次，一次性配置
```
    sourceSets {
            main.jniLibs.srcDirs = ['../remote/libs']
    }
```
###Gradle Plugin
对指定功能的task进行封装
https://juejin.im/post/5c3077d7e51d45523f04a340






