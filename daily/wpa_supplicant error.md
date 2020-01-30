# wpa_supplicant error
- 显示如下错误
![微信图片编辑_20190906125525.jpg](attachments\7777f3e5.jpg)
  - 注意错误提示（Device or resource busy）
  - 查看是否有进程占用资源：**ps aux | grep wpa**
  - kill进程：**pkill wpa_supplicant**