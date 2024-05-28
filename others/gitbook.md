# Gitbook 文档

## 安装 Node.js
选择版本 v10.x.x 版本，高版本 Node.js 安装 gitbook 以及 gitbook-cli 在执行 gitbook init 命令时会报错
```
TypeError [ERR_INVALID_ARG_TYPE]: The "data" argument must be of type string or an instance of Buffer, TypedArray, or DataView. Received an instance of Promise
```

## 安装 Gitbook
```
npm install gitbook -g
npm install gitbook-cli -g
gitbook -V

npm install ebook-convert -g // 生成 pdf 文件需要
```

## gitbook 命令

1. gitbook server 本地预览
2. gitbook build 生成 _book 目录 html 代码
3. gitbook pdf 生成 pdf 文件

## gitbook 生成 pdf 注意点
1. 需要安装 ebook-convert ,ebook-convert 是 calibre 软件的一个工具,所以需要安装 Calibre 软件
2. sudo ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/local/mysql/bin/ebook-convert

## gitbook plugin
https://gitbook.yangchi.tech/tools/gitbook.html
