# 使用sh脚本删除ES数据库30天前的日志文件

删除文件的格式为`log_dtm_20211212`

```shell
#!bin/bash

es_ip=$1
echo $es_ip
delete_indices(){
    check_day=`date -d "30 day ago" +"%Y%m%d"`
    index_day=$1

    #将日期转换为时间戳
    check_day_timestamp=`date -d "$check_day" +%s`
    index_day_timestamp=`date -d "$index_day" +%s`
    #当索引的时间戳值小于当前日期30天前的时间戳时，删除此索引
    if [ ${index_day_timestamp} -lt ${check_day_timestamp} ];then
        #转换日期格式
    
        curl -XDELETE http://${es_ip}:9200/*$index_day
        echo "删除的索引日期后缀：$index_day"
    fi

}

#拿到文件的日期后缀，并作为上面删除函数的输入
curl -XGET http://${es_ip}:9200/_cat/indices|awk -F " " '{print $3}'  | egrep "[0-9][0-9]+" |awk -F  "-" '{print $NF}' | awk -F  "_" '{print $NF}' | sort |uniq| while read LINE   
do
    #调用索引删除函数
    delete_indices $LINE
done
```


