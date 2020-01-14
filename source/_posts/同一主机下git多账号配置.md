---
title: 同一主机下git多账号配置
tags:
  - 开发工具
translate_title: git-multiple-account-configuration-under-the-same-host
date: 2018-11-27 11:37:33
---

在工作中，可能会碰到在同一台电脑上既要 git 公司远程仓库，也要 git 自己的 github 仓库，又或者同一个服务器上不同的开发人员都需要 git 自己的代码。那么这时候需要为不同 git 账户配置不同的ssh key。
<!--more-->

如果已经配置过 git 全局账户，那么首先取消 git 全局设置
{% codeblock  lang:shell %}
git config --global --unset user.name    
git config --global --unset user.email 
{% endcodeblock %} 
如果没有的话，那么新建账户即可。

# 新建账户
## SSH配置
### 一、生成ssh key
{% codeblock  lang:shell %}
ssh-keygen -t rsa -f ~/.ssh/id_rsa_user_company -C user@gmail.com
{% endcodeblock %} 
这里有两个参数，id_rsa_user_company 和 user@gmail.com ，前者是生成ssh密钥文件的文件名，不同用户和远程仓库取不一样的名字，比如 id_rsa_changhuan_github 等，后者是与远程仓库地址对应的邮箱，每个也可能不一样。

按照以上命令，生成不同的 ssh 密钥，并把对应的公钥（如id_rsa_changhuan_github.pub 文件里面key）添加到相应远端。

### 二、添加config文件
在.ssh目录下，新建config 文件（文件名就是config），设定不同git服务器的key。
这里重点是配置 Host 字段和IdentityFile 字段，前者对应git服务器域名，后者为对应的ssh密钥文件。
{% codeblock  lang:shell %}
# github
Host github.com
HostName github.com
User changhuan
IdentityFile ~/.ssh/id_rsa_changhuan_github

# company
Host company_host
HostName company_host_name
User changhuan
IdentityFile ~/.ssh/id_rsa_changhuan_company
{% endcodeblock %} 

# 项目配置
为每个项目单独配置 user.name 和 user.email
{% codeblock  lang:shell %}
git config user.name yourname
git config user.email youremail
{% endcodeblock %} 
注意这里的 yourname 和 youremail 千万不要加 ""号,加了可能会导致提交的时候不是自己的账户。





