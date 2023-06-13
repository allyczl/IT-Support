 IT Support Engineer Assignment

### 分析系统日志：interview_data_set.gz

分析系统日志得到关键信息，用 Json 的格式 POST 上传至服务器 ( https://foo.com/bar )，key 的名称在括号里

1. 设备名称: (deviceName)
2. 错误的进程号码: (processId)
3. 进程/服务名称: (processName)
4. 错误的原因（描述）(description)
5. 发生的时间（小时级），例如 0100-0200，0300-0400, (timeWindow)
6. 在小时级别内发生的次数 (numberOfOccurrence)

分别使用

1. Bash 或其他脚本语言，假设在 Mac 环境下，进行操作
2. Powershell，假设在 windows 环境下，进行操作
------------------------------------------------------------------------------------------------------------------------------------------------------

#在Mac环境下，可以使用Bash脚本语言进行操作。执行Bash命令: sh analyse_log.sh interview_data_set

#在Windows环境下，可以使用Powershell进行操作。powershell命令行执行命令：.\analyse_log.ps1 -log_file_name interview_data_set
