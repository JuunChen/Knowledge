#### config 的三个作用域
```
缺省等同于 local 
$ git config --local  #只对某个仓库有效
$ git config --global #对当前用户所有仓库有效
$ git config --system 对系统所有登录的用户有效
```

#### 查看 config 的配置
```ß
$ git config --list --local
$ git config --list --global
$ git config --list --system
```