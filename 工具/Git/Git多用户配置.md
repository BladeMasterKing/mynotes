# Git多用户配置

## 为不同账户创建不同密钥
```shell
$ cd ~/.ssh
$ ssh-keygen -t rsa -b 4096 -C "wangbodang2@126.com"
# 你在公司用git，肯定已经生成了公私钥，id_rsa/id_rsa.pub
# 所以这里就需要重新生成了，名字不能和原来重复，可以叫id_rsa_github
Enter file in which to save the key (/Users/jarvan4dev/.ssh/id_rsa): id_rsa_github
# 然后就直接两次回车吧
# 将生成的key加入 ssh-agent

$ eval $(ssh-agent -s)
# 如果ssh-agent没有执行,将公钥添加到ssh-agent,https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

$ ssh-add ~/.ssh/id_rsa_github
# 将公钥加入到github，参考：https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/
```

## 设置不同账户使用不同
```shell
# 配置使用固定的公钥去访问github
vim ~/.ssh/config
# 加入如下配置
Host github
 HostName github.com
 User jarvan4dev
 IdentityFile ~/.ssh/id_rsa_github
```

```shell
# 测试是否连通
$ ssh -T git@github.com
# 如果连通，输出如下：
Hi BladeMasterKing! You've successfully authenticated, but GitHub does not provide shell access.
# 如果未连通，输出如下：
Permission denied (publickey).


# 查看当前在使用的key
$ ssh-add -l
4096 SHA256:0YGlLwQsShE/qclzOQ4J8NebTWhv9koB5ijsxkmr0HE 
wangbodang2@126.com (RSA)


# 如果当前没有key，则：The agent has no identities. 此时需要添加key
$ ssh-add ~/.ssh/id_rsa

# ssh-add 这个命令不是用来永久性的记住你所使用的私钥的。实际上，它的作用只是把你指定的私钥添加到 ssh-agent 所管理的一个session 当中。而 ssh-agent 是一个用于存储私钥的临时性的 session 服务，也就是说当你重启之后，ssh-agent服务也就重置了。
可以使用 :
$ ssh-add -K ~/.ssh/id_rsa_github
输出如下：
Passphrase stored in keychain: /Users/jarvan4dev/.ssh/id_rsa_github
Identity added: /Users/jarvan4dev/.ssh/id_rsa_github (/Users/jarvan4dev/.ssh/id_rsa_github)
```

```shell
# 配置git账户
$ git config --list #查看当前配置
$ git config --list [--local] #查看当前仓库配置
$ git config --list --global #查看全局配置
# 配置全局账户（配置文件位于 ~/.gitconfig中）
$ git config --global user.name "your_name"  # 如果是提到github上，your_name最好是你的github账户的名字
$ git config --global user.email "your_email@example.com"  # 如果是提到github上，your_email@example.com最好是你的github账户的邮箱
# 配置本地仓库账户 (配置文件位于当前仓库目录的.git/config中）
$ git config [--local] user.name "your_name_in_company"  # 如果是提到github上，your_name最好是你的github账户的名字
$ git config [--local] user.email "your_company_email@example.com"  # 如果是提到github上，your_email@example.com最好是你的github账户的邮箱
```