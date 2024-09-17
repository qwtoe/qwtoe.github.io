操作系统：Ubuntu Server 20.04 LTS 64bit

VS Code终端命令执行后老是出现 `__vsc_prompt_cmd_original: command not found` ：

![](https://moonpic.oss-cn-beijing.aliyuncs.com/tf-feb/20240603222452.png)

解决方案：

1、`vim ~/.bashrc` 
2、在~/.bashrc里面加入命令：`unset PROMPT_COMMAND`
3、`source ~/.bashrc` 

<!-- ##{"script":"<script src='https://blog.meekdai.com/Gmeek/plugins/GmeekVercount.js'></script>"}## -->