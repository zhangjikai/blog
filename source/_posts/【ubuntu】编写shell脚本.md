title: 【Ubuntu】编写shell脚本
date: "2015-04-01 21:15:53"
tags: Ubuntu
---

## 编写shell脚本
1. 开头加上下面命令 `#!/bin/sh`
2. 编写相应的linux命令
3. 赋予脚本可执行权限 `chmod +x [脚本名称]`

## 脚本示例
### 复制文件夹
`#!/bin/sh`  
`cp -R /home/zhangjikai/文档/source /home/zhangjikai/KuaiPan`    
`rm -rf /home/zhangjikai/KuaiPan/source/.git`  

### 自动提交git
`#!/bin/sh`  
`cd /home/zhangjikai/文档/source`    
`git add -A .`    
`git commit -m "自动提交"`    
`git push` 

