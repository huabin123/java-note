# 加载顺序

- 系统级别

  ```sh
  /etc/profile
  /etc/paths 
  ```

- 用户级别

  ```sh
  ~/.bash_profile 
  ~/.bash_login 
  ~/.profile 
    
  ~/.bashrc
  ```

==先加载系统级别再加载用户级别==

# Maven配置环境变量的例子

- 打开终端输入命令 vim ~/.bash_profile （编辑环境变量配置文件）

- 在环境变量文件中加上如下的配置

  ```sh
  export MAVEN_HOME=yourpath
    
  export PATH=$PATH:$MAVEN_HOME/bin
  ```

- wq退出

- 输入 source ~/.bash_profile，按下Enter键使bash_profile生效。

- 输入 mvn -v测试

# 不生效可能的原因

- 可能是在当前窗口不生效：换一个窗口
- 查看是否路径配置的不对