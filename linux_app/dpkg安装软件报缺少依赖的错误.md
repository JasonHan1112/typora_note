2019/9/7
# dpkg安装软件报缺少依赖的错误
- 安装软件
dpkg -i xxxx.deb
- 修复缺少依赖
sudo apt --fix-broken install