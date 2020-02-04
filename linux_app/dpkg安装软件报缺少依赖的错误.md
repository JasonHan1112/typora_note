
# dpkg安装软件报缺少依赖的错误
- 安装软件
  dpkg -i xxxx.deb

- 修复缺少依赖
  sudo apt --fix-broken install

# 查看linux发行版版本
- lsb_release -a
  查看全称
  
- lsb_release -cs
  查看简称
  
# 添加额外的源
- 修改/etc/apt/sources.list
- 添加key: sudo apt-key add xxxx.gpg
- sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
- sudo apt-get update
- sudo apt-get install xxxxxxxxxxx

