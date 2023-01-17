---
title: Mac 安装 nvm
date: 2022-09-06 18:51:30
tags:
---
1. 从github下载nvm仓库到 ~/目录  地址：https://github.com/nvm-sh/nvm.git
    git clone https://github.com/nvm-sh/nvm.git
2. 进入 nvm目录中执行install.sh 等待执行完成
    sh install.sh
3. 配置nvm环境变量将下述代码复制到 ~/.bash_profile
vim ~/.bash_profile

~~~
export NVM_DIR="$HOME/.nvm"
 
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" 
 
 # This loads nvm
 
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
 
# This loads nvm bash_completion
~~~
4. 执行source  ~/.bash_profile
5. 执行nvm --version是否可以正常输出，若不行则重启终端再次尝试
6. nvm操作
   ①：使用  nvm install  node版本号  也可直接输入nvm install node 最新版本
   ②：使用 nvm list  或  nvm ls  可查看当前安装的node版本
   ③：使用 nvm use node版本 可以切换当前使用的node
   ④：使用 nvm alias default node版本  可以指定默认打开终端时的node版本

**问题**

每开一次终端，要 source ~/.bash_profile 环境变量才生效。

**原因**

>MacOS Catalina(10.15)，macOS的默认终端从bash变成了zsh。
Mac10.15以下版本,默认shell环境是bash，系统环境变量的配置文件是 /etc/profile 文件。
Mac10.15以上版本,默认shell环境是zsh，系统环境变量的配置文件是 /etc/zshrc 文件。

**解决方法**

编辑个人主目录下的.zshrc 这个文件

~~~
vim ~/.zshrc
~~~
在最后一行少添加一句：source ~/.bash_profile

这样每次打开新窗口或标签页就自动执行了source ~/.bash_profile，环境变量就有了