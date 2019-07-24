﻿# mpvue

> A Mpvue project

## Build Setup

``` bash
# 下载项目
git clone https://github.com/syhITMAN/weatherWx.git

cd weatherWx

# 安装依赖
npm/cnpm install --save

# 开发时构建
npm/cnpm dev

# 打包构建
npm build

# 指定平台的开发时构建(微信、百度、头条、支付宝)
npm dev:wx
npm dev:swan
npm dev:tt
npm dev:my

# 指定平台的打包构建
npm build:wx
npm build:swan
npm build:tt
npm build:my

# 生成 bundle 分析报告
npm run build --report
```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
