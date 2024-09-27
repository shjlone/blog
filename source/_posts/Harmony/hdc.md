---
title: hdc
tags: Harmony 
toc: true
---

Harmony Device Connector

```shell

# 查看连接设备
hdc list targets

# 安装应用
# -r 表示覆盖安装
hdc app install -r aa/bb/cc.hap

# 卸载应用
hdc app uninstall com.xxx.yyy

# 重启、关闭
hdc start
hdc kill

# 登录设备
hdc -t deviceId shell


# 将打包好的hap包推送至设备中
hdc file send "{PROJECT_PATH}/entry/build/default/outputs/default/entry-default-signed.hap" "data/local/tmp/entry-default-signed.hap"
# 安装hap包
hdc shell bm install -p "data/local/tmp/entry-default-signed.hap"
# 删除hap包
hdc shell rm -rf "data/local/tmp/entry-default-signed.hap"

```



## 参考

- [https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/ide-command-line-hdc-0000001237908229-V2](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/ide-command-line-hdc-0000001237908229-V2)