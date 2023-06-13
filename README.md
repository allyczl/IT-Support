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

#在Mac环境下，可以使用Bash脚本语言进行操作。执行命令 sh analyse_log.sh interview_data_set
#代码如下：

#!/bin/bash
#判断脚本执行参数的日志文件interview_data_set是否存在
if [ $# == 0 ];then
    echo "Usage: sh ./analysis_log.sh log_file"
    exit 1
elif [ ! -f $1 ];then
    echo "log_File does not exist!"
    exit 1
fi

#!/bin/bash
#---------------------------------------------------------------------------------------
# 分析系统日志，由于没有给出关键信息的定义，所以下面对日志所有信息进行分析处理
#---------------------------------------------------------------------------------------
analysis_upto(){
# 分析处理日志文件，用awk命令分析处理，
#以timeWindow|deviceName|processName|processId|description 作为数组来统计在小时级别内发生的次数
    jsonDatas=$(cat $1 \
    | sed 's/\"//g' `# 除去双引号，便于下面格式化JSON格式数据` \
    | sed "s/'//g" `# 除去单引号，便于下面格式化JSON格式数据` \
    | sed 'N;s/\n\t/ /' `# 合并tab键开头的信息到上一行` \
    | sed 'N;s/\n\t/ /' `# 再执行一次以上命令`\
    | sed '/---/d' `# 删除日志提示信息` \
    | awk -F' ' '{split($3,time_window,":"); timeWindow=time_window[1];\
    split($5,process_name,"["); processName=process_name[1]; \
    split(process_name[2],process_id,"]"); processId=process_id[1]; description=""; \
    for(i=6;i<=NF;i++)description=description$i; \
    device_log_msg[timeWindow"|"$4"|"processName"|"processId"|"description]++} \
    END {for(i in device_log_msg) {print i"|"device_log_msg[i]}}' \
    | sort `# 按发生时间（小时级）timeWindow升序排序` \
    `# 下面的awk命令用于生成JSON格式的数据` \
    | awk -F'|' 'BEGIN {print "["} \
    {start_hour=$1"00"; \
    end_hour=$1+1; \
    if(end_hour<10){end_hour="0"end_hour"00"} \
    else{end_hour=end_hour"00"}; \
    jsonData=jsonData"\t{\"deviceName\" : \""$2"\", \
    \"processId\" : \""$4"\", \
    \"processName\" : \""$3"\", \
    \"description\" : \""$5"\", \
    \"timeWindow\" : \""start_hour"-"end_hour"\", \
    \"numberOfOccurrence\" : \""$6"\"},\n" } \
    END {print substr(jsonData,1,length(jsonData)-2)"\n]"}')
    echo $jsonDatas
    response=$(curl -s -k -H 'Content-Type':'application/json' -d $jsonDatas -X POST https://foo.com/bar)
    if [ "$response" = "200" ]; then
    echo "Upload successful"
    else
    echo "Upload failed $response"
    fi
}

#执行分析日志函数以及post上传函数
analysis_upto $1

#***************************************************************************************************************************************
#在Windows环境下，可以使用Powershell进行操作。powershell命令行执行命令：.\analyse_log.ps1 -log_file_name interview_data_set
#代码如下：
param($log_file_name)

# 读取日志文件，优化日志文件信息
# 以timeWindow|deviceName|processName|processId|description 作为数组来统计在小时级别内发生的次数

$groupBy = @{}
Get-Content $log_file_name |  where {$_ -match "^[^\t]"} | % {
    $line = $_ -replace '"','' # 除去双引号，便于下面格式化JSON格式数据
    $line = $line -replace "'","" # 除去单引号，便于下面格式化JSON格式数据

    $line_a =  $line -split " "  
    $timeWindow = ($line_a[2] -split ":")[0]
    $deviceName = $line_a[3]
    $processName = ($line_a[4] -split "\[")[0]
    $processId = (($line_a[4] -split "\[")[1] -split "\]")[0]

    $description = ""
    For ($i = 5; $i -lt $line_a.Length; $i++) {
        $description += $line_a[$i]
    }

    $output = $timeWindow + "|"
    $output += $deviceName + "|"
    $output += $processName + "|"
    $output += $processId + "|"
    $output += $description

    $groupBy[$output] = $groupBy[$output] + 1
}

#   按需求以小时内产生的日志情况统计日志信息

$records_array = New-Object -TypeName System.Collections.ArrayList

foreach($i in $groupBy.Keys){
    $rec_array = $i -split '\|'
    $rec = '{' 
    $rec += '"deviceName" : "' + $rec_array[1] + '", '
    $rec += '"processId" : "' + $rec_array[3] + '", '
    $rec += '"processName" : "' + $rec_array[2] + '", '
    $rec += '"description" : "' + $rec_array[4] + '", '
    $rec += '"timeWindow" : "' + ('{0:d2}' -f $rec_array[0]) + "00-" + ('{0:d2}' -f ([System.Convert]::ToInt16($rec_array[0]) + 1)) + '00", '
    $rec += '"numberOfOccurrence" : "' + $groupBy[$i]
    $rec += '"}'

    $tmp = $records_array.Add($rec)
}

#   转化为JSON数据

$jsontable = ('{}' | ConvertFrom-Json )
$jsontable | Add-Member "records" ($records_array | ConvertFrom-Json)
$json_data = ($jsontable | ConvertTo-Json)

#   Post上传JSON数据

$url = "https://foo.com/bar"

Invoke-WebRequest -UseBasicParsing $url  -ContentType "application/json" -Method POST -Body $json_data
