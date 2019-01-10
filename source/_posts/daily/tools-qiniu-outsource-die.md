---
title: 解决七牛云图床测试域名收回问题
date: 2019-01-10 18:11:13
updated: 2019-01-10 18:11:13
categories:
- 其它
tags:
- 修复问题
---
# 前言
之前没把七牛云收回测试域名当回事，今天看到博客里好多图片都宕掉了，心塞之下，上网各种查，总算找到了应对的办法，其中部分参考资料的总结有误。重新总结一下，给可能需要的人。

<!-- more -->
# 主要步骤
## 配置qshell
0. 下载[qshell](https://developer.qiniu.com/kodo/tools/1302/qshell)
1. 在对象存储选项卡页面，新建对应备份地区的存储空间，记住它的名称`<new存储空间名称>`。
2. 配置环境，由于只是想把图片重新下载到本地。因此，也不那么讲究了。

```bash
# 创建快捷方式，对应平台使用对应命令行工具，这里以mac为例
sudo chmod +x qshell_darwin_x64
ln -s qshell_darwin_x64 qshell
# 去 个人中心->密钥管理 AK/SK
# name字段随便写的名字
./qshell account <AK> <SK> <name>
# 将旧空间的文件列表保存到list.txt
./qshell listbucket <old存储空间名称> >> list.txt
# 切割出文件名
cat list.txt | awk -F '\t' '{print $1}' > list_final.txt
# 把过期的文件列表搬迁到新的存储空间,我这里会出现让输入一个确认字符串，照着输入就行
./qshell batchcopy --force <old存储空间名称> <new存储空间名称> -i list_final.txt
```

## 批量下载到本地
0. 配置批量下载的文件`batch_download.conf`
这里的`cdn_domain`是新存储空间的测试域名。并且，需要注意的是，要自己收到创建`dest_dir`字段对应的文件夹。
```
{
    "dest_dir"   :   "/xxx/xxx/Downloads/qiniu",
    "bucket"     :   "<new存储空间名称>",
    "prefix"     :   "",
    "suffixes"   :   "",
    "cdn_domain" :   "http://pgiolcvny.bkt.clouddn.com",
    "referer"    :   "",
    "log_file"   :   "download.log",
    "log_level"  :   "info",
    "log_rotate" :   1,
    "log_stdout" :   false
}
```

1. 执行`./qshell qdownload batch_download.conf`

# 结束语
当然你也可以写一个批处理去弄，但是我的需求比较简答，以上就可以解决我的问题。另外，其实可以把图片保存到GitHub，替换一下URI的前缀即可。非常方便。

# 参考资料
- [七牛云测试域名过期，存储图片的下载方法](http://zsx-cup.top/2018/11/09/%E4%B8%83%E7%89%9B%E4%BA%91%E6%B5%8B%E8%AF%95%E5%9F%9F%E5%90%8D%E8%BF%87%E6%9C%9F%EF%BC%8C%E5%AD%98%E5%82%A8%E5%9B%BE%E7%89%87%E7%9A%84%E4%B8%8B%E8%BD%BD%E6%96%B9%E6%B3%95/)
- [qshell官方仓库README](https://github.com/qiniu/qshell)