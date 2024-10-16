---
title: 解决brew services restart nginx报错
date: 2024-09-09
category: 技术分享
tags: [Develop, Front]
---

最近brew update后，重启nginx服务发现报错

```jsx
 james@wx  ~  brew services restart nginx              ✔  10543  08:34:
	Error: uninitialized constant Homebrew::Service
	/opt/homebrew/Library/Homebrew/formulary.rb:396:in `block in load_formula_from_api'
	/opt/homebrew/Library/Homebrew/formulary.rb:285:in `initialize'
	/opt/homebrew/Library/Homebrew/formulary.rb:285:in `new'
	/opt/homebrew/Library/Homebrew/formulary.rb:285:in `load_formula_from_api'
	/opt/homebrew/Library/Homebrew/formulary.rb:969:in `load_from_api'
	/opt/homebrew/Library/Homebrew/formulary.rb:962:in `klass'
	/opt/homebrew/Library/Homebrew/formulary.rb:570:in `get_formula'
	/opt/homebrew/Library/Homebrew/formulary.rb:1016:in `factory'
	/opt/homebrew/Library/Taps/homebrew/homebrew-services/cmd/services.rb:89:in `services'
	/opt/homebrew/Library/Homebrew/brew.rb:96:in `public_send'
	/opt/homebrew/Library/Homebrew/brew.rb:96:in `<main>'
	If reporting this issue please do so at (not Homebrew/brew or Homebrew/homebrew-core):
	  https://github.com/homebrew/homebrew-services/issues/new
```

开发环境：Mac M2

以前一直是正常的，在brew update更新后发现失效了。

解决方案：

1. 找到brew安装位置

```jsx
 james@wx  ~  which brew                           127 ↵  10555  09:25:59
	/opt/homebrew/bin/brew
```

可以看到我这边的安装位置在/opt/homebrew/bin/brew

1. 根据brew安装位置找到homebrew-services

```jsx
cd / opt / homebrew / Library / Taps / homebrew
```

1. 删掉并重新安装

```jsx
sudo mv homebrew-services /tmp
brew tap homebrew/services
```

1. 重启nginx服务

```jsx
 james@wx  /opt/homebrew/Library/Taps/homebrew   stable 4.3.19  brew services restart nginx
Stopping `nginx`... (might take a while)
==> Successfully stopped `nginx` (label: homebrew.mxcl.nginx)
==> Successfully started `nginx` (label: homebrew.mxcl.nginx)
```

根据上面操作就已经恢复正常了~
