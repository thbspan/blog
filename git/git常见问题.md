# 1、ssh-keygen参数说明
    http://killer-jok.iteye.com/blog/1853451

# 2、本地添加多个ssh key
### a、ssh-keygen指定新文件
    ssh-keygen -t rsa -b 4096 -f ~/.ssh/github_rsa

### b、在~/.ssh目录下添加config文件
    # gitlab
    # 别名
    Host gitlab.com
    # 主机名
    HostName gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa

    # github
    Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_rsa

# 3、版本回退
    使用git reset回退版本之后直接push会报错，因为服务端head在你的前面，系统会提示你更新（但是如果更新的话，git reset就会失效）
    可以添加-f参数（git push -f origin master）
