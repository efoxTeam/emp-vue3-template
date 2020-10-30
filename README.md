# EMP Vue3 Template

## 目录架构

``` 

|--src
    |--components // 组件目录
        |--Layout // Layout组件
    |--main.js // 入口文件
    |--config.js // host配置文件
|--emp-config.js // emp配置文件
```

## 使用说明

1. 在项目 host 配置文件 config.js 中引入基站地址

``` js
const dev = {
    host: "localhost",
    port: 8005,
    publicPath: "http://localhost:8005/",
    // 远程基站地址
    baseRemote: '你的基站地址',
    baseRemoteEntry: `你的基站地址/emp.js`,
};
const prod = {
    host: "localhost",
    port: 8005,
    publicPath: "http://localhost:8005/",
    // 远程基站地址
    baseRemote: '你的基站地址',
    baseRemoteEntry: `你的基站地址/emp.js`,
};
```

2. 在项目的emp-config.js引入emp.js

``` js
  config.plugin('html').tap(args => {
      args[0] = {
          ...args[0],
          ...{
              filename: 'emp.js',
          },
      }
      return args
  })
```

3. 在MF的配置中引入

``` js
config.plugin('mf').tap(args => {
    args[0] = {
        ...args[0],
        ...{
            remotes: {
                // '基站别名': '基站暴露的别名(因基站而定)',
                vue3Components: 'vue3Components',
            },
        },
    },
}
return args
})
```

4. 暴露当前项目的组件、工具方法等等

``` js
  config.plugin('mf').tap(args => {
      args[0] = {
          ...args[0],
          ...{
              exposes: {
                  // '暴露对外的别名': '组件在项目的相对路径',
                  './Button': './src/components/Button',
              },
          },
      }
      return args
  })
```

5. 同步类型文件

+  通过配置scripts手动更新（emp tss https://你的项目地址/index.d.ts -n @emp-react-base.d.ts）
+ [安装vscode插件 emp-sync-base自动同步](https://marketplace.visualstudio.com/items?itemName=Benny.emp-sync-base)


4. 使用基站组件（远程组件）

main.js
```js
import {createApp, defineAsyncComponent} from 'vue'
import Layout from './Layout.vue'

const Button = defineAsyncComponent(() => import('vue3Components/Button'))

const app = createApp(Layout)

app.component('button-element', Button)

app.mount('#emp-root')

```

Layout.vue
```js

<template>
<div>
  <h1>EMP vue3 Components 调用远程组件</h1>
  <div>
        <h2>远程元素 Button.js</h2>
        <button-element />
  </div>
</div>
</template>

```


## 依赖库 package.json

``` json
  "devDependencies": {
    "@efox/emp-cli": "^1.1.17",
    "@vue/compiler-sfc": "^3.0.0-rc.10",
    "vue-loader": "^16.0.0-beta.5"
  },
  "dependencies": {
    "vue": "^3.0.0-rc.10"
  },
```

## 微前端配置 emp-config.js

``` javascript
const path = require('path')
const {
    VueLoaderPlugin
} = require('vue-loader')
//
const ProjectRootPath = path.resolve('./')
// const packagePath = path.join(ProjectRootPath, 'package.json')
// const {dependencies} = require(packagePath)
//
const {
    getConfig
} = require(path.join(ProjectRootPath, './src/config'))
//
module.exports = ({
    config,
    env,
    empEnv
}) => {
    const confEnv = env === 'production' ? 'prod' : 'dev'
    const conf = getConfig(empEnv || confEnv)
    // config.entry('web')
    const srcPath = path.resolve('./src')
    config.entry('index').clear().add(path.join(srcPath, 'main.js'))
    // vue 3 编译构建
    config.resolve.alias.set('vue', '@vue/runtime-dom')
    config.plugin('vue3').use(VueLoaderPlugin, [])
    config.module

        .rule('vue')
        .test(/\.vue$/)
        .use('vue-loader')
        .loader('vue-loader')

    //
    const host = conf.host
    const port = conf.port
    const projectName = 'vue3Base'
    const publicPath = conf.publicPath
    config.output.publicPath(publicPath)
    config.devServer.port(port)
    //
    config.plugin('mf').tap(args => {

        args[0] = {
            ...args[0],
            ...{
                name: projectName,
                library: {
                    type: 'var',
                    name: projectName
                },
                filename: 'emp.js',
                remotes: {
                    vue3Components: 'vue3Components',
                },
                exposes: {},
                /*  shared: {
                  ...dependencies,
                }, */
            },
        }
        return args

    })
    //
    config.plugin('html').tap(args => {

        args[0] = {
            ...args[0],
            ...{
                title: 'EMP Vue3 Base',
                files: {
                    js: [conf.baseRemoteEntry],
                },
            },
        }
        return args

    })
}
```
