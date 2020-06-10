# AppHost
![MacDown logo](public/favicon.ico)

[![Build Status](https://travis-ci.org/pluosi/app-host.svg?branch=master)](https://travis-ci.org/pluosi/app-host)
[![License](https://img.shields.io/github/license/mashape/apistatus.svg)](https://travis-ci.org/pluosi/app-host)
[![Gems](https://img.shields.io/gem/u/raphink.svg)]()

## 介绍
一个轻量级的包托管网站，app-host 主要用于 iOS 和 Android 的包管理，作用类似于fir.im，不同之处是可以自由部署在内网，方便了公司项目保密。并且代码开源也可以方便根据各自需求进行定制化开发。


## 目前能实现
1.新建包<br>
2.包底下新建渠道（ iOS，安卓，各种环境都归为渠道，例如 iOS 生产，iOS 沙盒，iOS 越狱版，Android 生产等）<br>
3.渠道下面上传包<br>
4.帐号和权限管理<br>
5.api 和页面表单上传包<br>
6.解析包信息，包括 iOS 的包类型 ADHOC 还是 release，udid，安卓的签名证书等<br>
7.我编不下去了···哈哈~~<br>

## 用法 1 Docker
```
1. > git clone https://github.com/pluosi/app-host.git /opt/app-host
2. > cd /opt/app-host
3. > cp config/settings.local.example.yml config/settings.local.yml
4. 修改 config/settings.local.yml 中 `PROTOCOL` 和 `HOST` ,本地测试PROTOCOL可以为 http,生产环境必须设置为https,因为 iOS OTA 安装需要
5. > ./docker/launcher bootstrap -v #该步骤依赖网络，所以如果网络不稳定报错了，可以重试几次
6. > ./docker/launcher start
7. 尝试访问 http://localhost:3000 ,如果不希望用3000端口，可以手动修改 docker/launcher 里的`local_port`值
```


## 用法 2 源码运行
```
1. > git clone https://github.com/JiaHaoGong/app-host.git  /opt/app-host
2. > cd /opt/app-host
3. > cp config/settings.local.example.yml config/settings.local.yml
4. 修改 config/settings.local.yml 中 `PROTOCOL` 和 `HOST` ,本地测试PROTOCOL可以为 http,生产环境必须设置为https,因为 iOS OTA 安装需要
5. bundle install
6. rails s 运行测试环境
7. 关于部署到生成环境的话请参照一下 rails puma 部署等教程，需要修改一下 config/deply.rb 的部署地址
8. 尝试访问 http://localhost:3000
```

## 关于 https
1. https其实不属于本项目涉及的范畴，大家可以 google 一下 https 证书配置，挂 nginx 或者 apache 上都行，有条件的可以购买域名证书，没条件的自签名证书也是可以的

## 已知问题
1. 目前只以单线程运行，因为ruby_android这个 gem 在解压 apk 时内存消耗很大，开的线程多了会跑满内存被系统干掉。当然，机器内存大的可以直接修改 config/puma.rb 里的 threads_max 数量


## License
AppHost is released under the MIT license. See LICENSE for details.

## 截图-PC
![MacDown logo](screenshots/index.png)

![MacDown logo](screenshots/plat.png)

![MacDown logo](screenshots/pkg.png)

## 截图-Mobile
![MacDown logo](screenshots/index_mobile.png)

![MacDown logo](screenshots/plat_mobile.png)

![MacDown logo](screenshots/pkg_mobile.png)


## 备注 部署与mac上的相关问题

### 问题一: mac上ruby 版本和gem 指定版本不统一的问题;
> 处理,安装rvm 进行多版本ruby 管理;
>此外
安装bundle install therubyracer 会遇到异常
可采用

```bash
$ brew install v8@3.15

$ bundle config build.libv8 --with-system-v8

$ bundle config build.therubyracer --with-v8-dir=$(brew --prefix v8@3.15)

$ bundle install
```

或者

```bash
brew uninstall v8

brew install v8-315

gem uninstall -a libv8

gem uninstall -a therubyracer

gem install libv8 -v '3.16.14.17' -- --with-system-v8

gem install therubyracer -v '0.12.2' -- --with-v8-dir=$(brew --prefix v8-315)

```

### 问题二: https 证书的问题

参考https://www.betaflare.com/3572.html
和https://madeintandem.com/blog/rails-local-development-https-using-self-signed-ssl-certificate/

此外
1. Gemfile 中

 将 ` gem 'puma', '~> 3.7' `

 替换为

` gem 'puma', git: 'https://github.com/eric-norcross/puma.git', branch: 'chrome_70_ssl_curve_compatiblity' `

2. 在config/puma.rb中

 修改最大线程数

 `threads_max = ENV.fetch("RAILS_MAX_THREADS") { 10 }`



3.  vim config/settings.local.yml

 ```
  PROTOCOL: "https://"

  HOST: "yourIPorDomian:3000"
 ```


4. 最后
` rails s -b 'ssl://yourIPorDomian:3000?key=path/to/file/localhost.key&cert=path/to/file/localhost.crt' `

  备注:前提是先先用openSSL或者其他工具生成SSL自建证书
  
5. 相关提示
   
   关于rails 失去响应

  https://github.com/greatghoul/profile/issues/34

  在 Ruby on Rails 的日常开发时，有时候不小心写出个坏代码（比如死循环），导致 rails dev server 进程失去响应，这时候如果想杀掉进    程，按 Cmd+C 可能没有效果。

  除了去 grep ps 或者查看 server.pid 外，还有一个快捷的方法快速结束进程。

按下 Cmd+Z，进程会自动转到后台运行，此时终端上会输出进程 PID
 

 
