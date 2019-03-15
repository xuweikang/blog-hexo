---
title: 基于vue的脚手架开发与发布到npm仓库
date: 2019-01-17 16:01:58
tags: cli, vue, node
cover: 'https://flaviocopes.com/nodejs/banner.png'
---
### 什么是脚手架
在项目比较多而且杂的环境下，有时候我们想统一一下各个项目技术栈或者一些插件/组件的封装习惯,但是每次从零开发一个新项目的时候，总是会重复做一些类似于复制粘贴的工作,这是一个很头疼的事情，所以各种各样的脚手架应用而生。
脚手架也就是为了方便我们做一些重复的事情，快速搭建一个基本的完整的项目结构。例如：vue-cli, react-cli, express-generator

### 以vue-cli为例
1. 全局安装vue-cli
```
npm install vue-cli -g
```
2. 然后在终端中键入vue,vue init或者vue-init命令,会出现以下的命令说明：
```
xwkdeMacBook-Pro:bin xwk$ vue
Usage: vue <command> [options]

Options:
  -V, --version  output the version number
  -h, --help     output usage information

Commands:
  init           generate a new project from a template
  list           list available official templates
  build          prototype a new project
  create         (for v3 warning only)
  help [cmd]     display help for [cmd]

xwkdeMacBook-Pro:bin xwk$ vue init
Usage: vue-init <template-name> [project-name]

Options:
  -c, --clone  use git clone
  --offline    use cached template
  -h, --help   output usage information
  Examples:

    # create a new project with an official template
    $ vue init webpack my-project

    # create a new project straight from a github template
    $ vue init username/repo my-project

```
可以根据这些命令说明，来快速生成一个项目骨架，例如：vue init webpack demo1

```
xwkdeMacBook-Pro:Practice xwk$ vue init webpack demo1

? Project name： demo1
? Project description： A Vue.js project
? Author xuweikang： <xuweikang@dajiazhongyi.com>
? Vue build standalone：
? Install vue-router? Yes
? Use ESLint to lint your code? No
? Set up unit tests？ No
? Setup e2e tests with Nightwatch? Yes
? Should we run `npm install` for you after the project has been created? (recomme
nded) npm

   vue-cli · Generated "demo1".


# Installing project dependencies ...

```
如上图所示，在输入vue init指令的时候，会有一些选项让我们去选择，选择完成后在当前目录就会多出一个demo1的文件夹，这就是脚手架生成的项目骨架，到目前为止，已经成功的使用脚手架工具创建出了一个我们半自定义（一些自定义配置选项是我们在刚开始选择的那些，脚手架会将这些选项应用到初始化项目中）的项目。

### vue-cli的原理分析

> 对于vue-cli的原理分析，其实无外乎有几个大点，首先从我刚开始在终端中输入vue/vue-init/vue init这些命令开始，为什么可以在终端中直接使用这些命令，这些命令的使用说明是怎么打印出来的，还有vue-cli是怎样在输入vue init webpack demo1命令后，成功的在当前目录创建出一个项目骨架的，这些项目是怎么来的。

1. 可执行的npm包
  如果我们想让一个模块全局可执行，就需要把这个模块配置到PATH路径下，npm让这个工作变得很简单，通过在package.json文件里面配置bin属性，这样该模块在安装的时候，若是全局安装，则npm会为bin里面的文件在PATH目录下配置一个软链接，若是局部安装，则会在项目里面的node_modules/.bin目录下创建一个软链接，例如：
  ```
  //package.json
  {
    "bin": { 
      "cvd": "bin/cvd"
    }
  }
  ```
  当我们安装这个模块的时候，npm就会为bin下面的cvd文件在/usr/local/bin创建一个软链接。在Mac系统下，usr/local/bin这个目录，是一个已经包含在环境变量里的目录，可以直接在终端中执行这里的文件。
  注意：windows下bin目录不一样，如果是直接在本地项目中进行包调试，可以通过npm link命令，将本项目的bin目录链接到全局目录里，这里面也可以看到对应的bin目录。
  ```
  xwkdeMacBook-Pro:vue-cli-demo1 xwk$ npm link
  npm WARN vue-cli-demo1@1.0.0 No description
  npm WARN vue-cli-demo1@1.0.0 No repository field.

  audited 1 package in 1.35s
  found 0 vulnerabilities

  /usr/local/bin/cvd -> /usr/local/lib/node_modules/vue-cli-demo1/bin/cvd
  /usr/local/bin/cvd-init -> /usr/local/lib/node_modules/vue-cli-demo1/bin/cvd-init
  /usr/local/lib/node_modules/vue-cli-demo1 -> /Users/admin/Project/vue-cli-demo1
  ```
  到目前为止可以解释了为什么我们在全局install了vue-cli后，可以直接使用vue/vue-init等命令。

2. vue-cli源码分析
  > 找到vue-cli源码进行分析，有两种方法，可以直接去找刚刚安装的脚手架的位置，这里是全局安装的，mac会定位到/usr/local/lib/node_modules/vue-cli，或者直接看vue-cli的仓库源码，点击{% link 这里 'https://github.com/vuejs/vue-cli/tree/master' 'vue-cli' %}

  有了上面的分析，直接找到package.json，可以看到：
  ```
  {
  "bin": {
    "vue": "bin/vue",
    "vue-init": "bin/vue-init",
    "vue-list": "bin/vue-list"
    }
  }
  ```
  这里面定义了3个可执行文件命令，vue，vue-init和vue-list，分别对应到了bin目录下的vue,vue-init,vue-lsit文件，这里只分析下第一个和第二个文件。

  ### ** bin/vue **
  ```
  #!/usr/bin/env node     //声明下该文件要用node格式打开

  const program = require('commander')     //ti大神的nodejs命令行库

  program
    .version(require('../package').version)   //取包的版本为当前版本
    .usage('<command> [options]')  //定义使用方法
    .command('init', 'generate a new project from a template')   //有一个init方法，并且对其进行描述
    .command('list', 'list available official templates')  //有一个list方法，并且对其进行描述
    .command('build', 'prototype a new project')  //有一个build方法，并且对其进行描述
    .command('create', '(for v3 warning only)')   //有一个create方法，并且对其进行描述

  program.parse(process.argv)    //执行
  ```
  效果如下：
  ```
  xwkdeMacBook-Pro:bin xwk$ vue
  Usage: vue <command> [options]

  Options:
    -V, --version  output the version number
    -h, --help     output usage information

  Commands:
    init           generate a new project from a template
    list           list available official templates
    build          prototype a new project
    create         (for v3 warning only)
    help [cmd]     display help for [cmd]
  ```
  ### ** bin/vue-init **
  ```
  /**
 * Usage.
 */

  program
    .usage('<template-name> [project-name]')
    .option('-c, --clone', 'use git clone')
    .option('--offline', 'use cached template')

  /**
  * Help.
  */

  program.on('--help', () => {
    console.log('  Examples:')
    console.log()
    console.log(chalk.gray('    # create a new project with an official template'))
    console.log('    $ vue init webpack my-project')
    console.log()
    console.log(chalk.gray('    # create a new project straight from a github template'))
    console.log('    $ vue init username/repo my-project')
    console.log()
  })
  ```
  这部分主要是声明一些命令和使用方法介绍，其中chalk 是一个可以让终端输出内容变色的模块。
  下面这部分主要是一些变量的获取，定义项目名称，输出路径，以及本地存放模板的路径位置
  ```
  /**
  * Settings.
  */
  //vue init 命令后的第一个参数，template路径
  let template = program.args[0]    
  //template中是否带有路径标识
  const hasSlash = template.indexOf('/') > -1  
  //第二个参数是项目名称，如果没声明的话或者是一个“.”，就取当前路径的父目录名字
  const rawName = program.args[1]
  const inPlace = !rawName || rawName === '.'
  const name = inPlace ? path.relative('../', process.cwd()) : rawName
  //输出路径
  const to = path.resolve(rawName || '.')
  const clone = program.clone || false

  //存放template的地方，用户主目录/.vue-templates ，我这里是/Users/admin/.vue-templates/
  const tmp = path.join(home, '.vue-templates', template.replace(/[\/:]/g, '-'))
  //如果是离线状态，模板路径就取本地的
  if (program.offline) {
    console.log(`> Use cached template at ${chalk.yellow(tildify(tmp))}`)
    template = tmp
  }
  ```
  下面是一些对于项目初始化简单的问答提示，其中inquirer 是一个node在命令行中的问答模块，你可以根据答案去做不同的处理
  ```
  if (inPlace || exists(to)) {
    inquirer.prompt([{
      type: 'confirm',
      message: inPlace
        ? 'Generate project in current directory?'
        : 'Target directory exists. Continue?',
      name: 'ok'
    }]).then(answers => {
      if (answers.ok) {
        run()
      }
    }).catch(logger.fatal)
  } else {
    run()
  }

  ```
  接下来是下载模板的具体逻辑，如果是本地模板，则直接生成
  ```

  function run () {
  // check if template is local
    if (isLocalPath(template)) {
      //获取模版地址
      const templatePath = getTemplatePath(template)
      if (exists(templatePath)) {
        //开始生成模板
        generate(name, templatePath, to, err => {
          if (err) logger.fatal(err)
          console.log()
          logger.success('Generated "%s".', name)
        })
      } else {
        logger.fatal('Local template "%s" not found.', template)
      }
    } else {
      checkVersion(() => {
        //路径中是否包含 ‘/’，如果包含 ‘/’，则直接去指定模板路径去下载
        if (!hasSlash) {
          // use official templates
          //生产仓库里面的模板路径
          const officialTemplate = 'vuejs-templates/' + template
          if (template.indexOf('#') !== -1) {
            //下载仓库分支代码
            downloadAndGenerate(officialTemplate)
          } else {
            if (template.indexOf('-2.0') !== -1) {
              //如果存在 ‘-2.0’ 标识，则会输出模板废弃的警告并退出
              warnings.v2SuffixTemplatesDeprecated(template, inPlace ? '' : name)
              return
            }

            // warnings.v2BranchIsNowDefault(template, inPlace ? '' : name)
            //开始下载
            downloadAndGenerate(officialTemplate)
          }
        } else {
          downloadAndGenerate(template)
        }
      })
    }
  }
  ```
  vue-init源码的最后一部分，是对downloadAndGenerate方法的声明，是下载并在本地生产项目的具体逻辑。
  download是download-git-repo模块的方法，是来做从git仓库下载代码的，
  ```
  //第一个参数是仓库地址，如果指定分支用“#”分开，第二个为输出地址，第三个为是否clone，为flase的话就下载zip,
  //第四个参数是回调
  download('flipxfx/download-git-repo-fixture#develop', 'test/tmp',{ clone: true }, function (err) {
  console.log(err ? 'Error' : 'Success')
  })
  ```
  ```
    function downloadAndGenerate (template) {
      //ora库在终端中显示加载动画
      const spinner = ora('downloading template')
      spinner.start()
      // Remove if local template exists
      //如果有相同文件夹，则覆盖删除
      if (exists(tmp)) rm(tmp)
      download(template, tmp, { clone }, err => {
        spinner.stop()
        if (err) logger.fatal('Failed to download repo ' + template + ': ' + err.message.trim())
        //生成个性化内容
        generate(name, tmp, to, err => {
          if (err) logger.fatal(err)
          console.log()
          logger.success('Generated "%s".', name)
        })
      })
    }

  ```
  ### ** lib/generate.js **
  至此，还差最后一步，即generate方法的定义，这个方法在lib/generate.js中，主要作用是用于模板生成
  ```
  /**
 * Generate a template given a `src` and `dest`.
 *
 * @param {String} name
 * @param {String} src
 * @param {String} dest
 * @param {Function} done
 */

  module.exports = function generate (name, src, dest, done) {
    //设置meta.js/meta.json配置文件的name字段和auther字段为项目名和git用户名，同时还设置了校验npm包名的方法属性
    const opts = getOptions(name, src)
    //指定Metalsmith的模板目录路径
    const metalsmith = Metalsmith(path.join(src, 'template'))
    //将metalsmith的默认metadata和新增的3个属性合并起来
    const data = Object.assign(metalsmith.metadata(), {
      destDirName: name,
      inPlace: dest === process.cwd(),
      noEscape: true
    })
    //注册一些其他的渲染器，例如if_or/template_version
    opts.helpers && Object.keys(opts.helpers).map(key => {
      Handlebars.registerHelper(key, opts.helpers[key])
    })

    const helpers = { chalk, logger }

    if (opts.metalsmith && typeof opts.metalsmith.before === 'function') {
      opts.metalsmith.before(metalsmith, opts, helpers)
    }

    //metalsmith做渲染的时候定义了一些自定义插件
    //askQuestions是调用inquirer库询问了一些问题，并把回答结果放到metalsmithData中
    //生产静态文件时删除一些不需要的文件，meta文件的filters字段中进行条件设置
    //开始生产文件
    metalsmith.use(askQuestions(opts.prompts))
      .use(filterFiles(opts.filters))
      .use(renderTemplateFiles(opts.skipInterpolation))

    if (typeof opts.metalsmith === 'function') {
      opts.metalsmith(metalsmith, opts, helpers)
    } else if (opts.metalsmith && typeof opts.metalsmith.after === 'function') {
      opts.metalsmith.after(metalsmith, opts, helpers)
    }

    metalsmith.clean(false)
      .source('.') // start from template root instead of `./src` which is Metalsmith's default for `source`
      .destination(dest)
      .build((err, files) => {
        done(err)
        if (typeof opts.complete === 'function') {
          const helpers = { chalk, logger, files }
          opts.complete(data, helpers)
        } else {
          logMessage(opts.completeMessage, data)
        }
      })

    return data
  }
  ```

  

### 具体交互逻辑和命令补充

### 发布到npm

### 测试