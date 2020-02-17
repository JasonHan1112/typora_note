# 新的veloce上EMU使用流程

## 新EMU环境使用与旧的使用方法是一样的，只是登录路径不同。

1. 进入EOD：cdra011, xconfig：dcuxxx
2. ssh -XY cpfg002
3. 新的EMU环境在vhost0~5，使用不同的module需要ssh到不同的vhost上，如果使用module0 需要ssh -XY vhost2, 如果使用module1需要ssh -XY vhost3
4. 在vhost上找到/project/d2n7/a0/sw/hanxueqing（vhost0～5上对应的目录是一致的，在edr：epf010的机器上看到的是/project/d2n7/a0/sw_cdc/hanxueqing）
5. 在相应的vhost机器上，进入emuxxx文件夹路径分布和旧的一致，有run_emu.sh是执行的emu run相关命令