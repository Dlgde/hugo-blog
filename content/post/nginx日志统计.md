---
title: "Nginx日志统计"
date: 2021-04-08T15:16:44+08:00
draft: 
categories:
  - shell
tags:
  - nginx
---

是一个比较简单的脚本用来nginx的access日志简单处理，主要实现以下三个功能：
* 统计几分钟之内，请求最多的前几个ip（默认是3分钟，前10个ip）
* 统计几分钟之内，请求最多的接口(默认是3分钟，前10个接口)
* 统计前几ip，分别请求的前几个接口

具体脚本下：
```shell
#输出日志访问量排名前top_num条数据，可自定义
top_num=10
time=$(date)
analyze_dir=/home/ap/nginx
access_log=/usr/local/openresty/nginx/logs/access.log
top_ip_file=$analyze_dir/ngx_log_top_ip.txt
top_dest_url_file=$analyze_dir/ngx_log_top_dest_url.txt
log_file=$analyze_dir/minago_ngx_log.txt

mkdir -p $analyze_dir
#截取最近3分钟的日志文件，时间可修改
tac $access_log| awk 'BEGIN{ "date -d \"-3 minute\" +\"%H:%M:%S\"" | getline min3ago } { if (substr($4, 14) > min3ago) print $0;else exit }' | tac >$log_file

#获取最活跃IP
cat $log_file | awk '{print $1}' | sort | uniq -c | sort -rn | head -${top_num} > $top_ip_file

#获取请求最多的url
cat $log_file | awk '{print $7}' | sort | uniq -c | sort -rn | head -${top_num} > $top_dest_url_file

simple(){

echo "+-+-+-+-+-+- 下面是${time}的日志分析内容 +-+-+-+-+-+-"

#获取最活跃IP
printf "最活跃的前${top_num}个访问IP: \n"

cat $top_ip_file

echo ""

#获取请求最多的url
printf "请求最多的前${top_num}个url: \n"

cat $top_dest_url_file

echo ""


echo '-----------------------------------------------'

cat /home/ap/nginx/ngx_log_top_ip.txt | while read line

do

ip=$(echo $line | cut -d ' ' -f2)
count=$(echo $line | cut -d ' ' -f1)
printf "${ip}请求最多的10个接口如下：\n"

grep $ip $log_file |awk '{print $7}'|grep -v 'images\|js\|css\|png\|fonts\|html\|jpg'|sort |uniq -c |sort -rn |head -n 10 

echo '-----------------------------------------------'

done
}
simple
exit 0

```

