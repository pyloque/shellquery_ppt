## sql与shell对应关系 ##
    table   file || /tmp/file ==> data source
    select  cut -d';' -f1,2 || awk -F';' {print $1';'$2} ==> map
    filter  sed -n -e '/pattern/' || awk -F';' '$2 > 20 {print $0}' ==> filter
    group_by    awk '{a[$2] += $3}END{for(i in a)print i";"a[i]}' ==> reduce
    order_by    sort -r -k2,3 -n ==> sort
    join    join -t';' -1 1 -2 1 -o 1.1 2.2 file1 file2 ==> join
    distinct    uniq || sort -u ==> reduce
    limit   head -n10 ==> slice
    offset  sed -n 'offset,offset+limit p' ==> slice
    update sed -n -e -i 's/pattern/newvalue/g' ==> update
    insert sed '/pattern/ a new line content' -i ==> insert
    delete sed '/pattern/ d' -i ==> delete

## 类比 ##

1. 写命令就像写sql
2. 写shell脚本就像写存储过程

## 数据样例 ##
    rank_items.txt(258770)
    "5b94c293-bdd0-4569-8203-0dbb9eeab432";"academy_hot_course";"6";"2013-06-03 21:37:41.189835+08";10
    "55a2d8d2-9734-43ba-b8ee-a55fc0596bd4";"hot_question:微观经济学";"469734";"2013-06-05 18:08:05.741067+08";1
    "28f7d770-49c9-47b9-9435-5f8c8de06a78";"hot_question:动物";"438176";"2013-06-05 18:08:41.168943+08";1
    "931bc2ff-9145-4765-813d-768b05fa9372";"hot_post:69";"463554";"2013-06-03 21:38:58.224445+08";1
    "da55cbcf-ce8e-47d2-a88d-d879801205f6";"group_hot_member:198";"p9hg1u";"2013-06-03 21:39:04.28642+08";1
    "e9594fdc-9961-4cc4-b43a-9cedd323a9b0";"hot_ask_user";"et4hhg";"2013-06-05 18:08:41.202577+08";1
    "f0613373-1108-4cd6-9856-c4b42dc3ecee";"hot_tag";"求真相";"2013-06-01 10:03:36.764314+08";1
    "a59f3536-7133-4c1f-a3da-a02ec57eac51";"hot_post:27";"459797";"2013-06-03 21:40:20.346828+08";1
    "8aec1661-7ada-4b8f-b815-993039b281d7";"hot_post:198";"464179";"2013-06-03 21:40:34.297701+08";1

    groups.txt(217)
    205;"真要瘦不瘦不罢休";"2012-11-23 13:42:38+08"
    28;"健康朝九晚五";"2010-10-20 16:20:43+08"
    280;"核谐家园";"2013-04-17 17:11:49.545351+08"
    38;"创意科技";"2010-10-20 16:20:44+08"
    39;"死理性派";"2010-10-20 16:20:44+08"
    175;"魔兽世界";"2012-10-08 14:35:20+08"
    29;"爱宠";"2010-10-20 16:20:44+08"


## 总数  ##
    select count(1) from rank_item

    wc -l rank_items.txt | cut -d' ' -f1
    cat rank_items.txt | wc -l
    wc -l < rank_items.txt

## 性情小组活跃用户数量 ##
    select count(distinct(object_name))
    from rank_item
    where rank_name='group_hot_member:30'

    sed -n -e '/"group_hot_member\:30"\;/ p' rank_items.txt 
    | cut -d';' -f3 | sort | uniq | wc -l

## 性能 ##
![并发模型](http://www.plc100.com/siemens/shili/chuansongdai.files/image002.jpg)

## 性情小组活跃用户TOP10  ##

    select object_name,sum(score) total_score
    from rank_item
    where rank_name='group_hot_member:30'
    group by object_name
    order by total_score desc
    limit 10

    sed -n -e '/"group_hot_member\:30"\;/ p' rank_items.txt 
    | awk -F';' '{a[$3]+=$5}END{for(i in a)print i,a[i]}' 
    | sort -t' ' -n -r -k 2 | head -n 10

## 性情小组活跃用户(总分大于80的)  ##

    select object_name,sum(score) total_score
    from rank_item
    where rank_name='group_hot_member:30'
    group by object_name
    order by total_score desc
    limit 10

    sed -n -e '/"group_hot_member\:30"\;/ p' rank_items.txt 
    | awk -F';' '{a[$3]+=$5}END{for(i in a)print i,a[i]}' 
    | awk -F' ' '$2 > 80 {print $0}' 
    | sort -t' ' -n -r -k 2 | head -n 10

##  活跃小组按总分倒排 ##
    select substr(rank_name,18) group_id, sum(score) total_score
    from rank_item
    where rank_name like 'group_hot_member:%'
    group by group_id
    order by total_score desc
    limit 100

    sed -n -e '/"group_hot_member\:/ p' rank_items.txt 
    | cut -d';' -f2,5 | sed -e s/\"//g -e 's/group_hot_member\://g' 
    | awk -F';' '{a[$1]+=$2}END{for(i in a)print i,a[i]}' 
    | sort -t' ' -n -r -k2 | head -n100

## 活跃小组按总分倒排（显示小组名称） ##
    select b.group_id,a.total_score from(
        select substr(rank_name,18) group_id, sum(score) total_score
        from rank_item
        where rank_name like 'group_hot_member:%'
        group by group_id
        order by total_score desc
        limit 100
    ) a join group b on a.group_id=b.id
    order by a.total_score desc

    sed -n -e '/"group_hot_member\:/ p' rank_items.txt 
    | cut -d';' -f2,5 | sed -e s/\"//g -e 's/group_hot_member\://g' 
    | awk -F';' '{a[$1]+=$2}END{for(i in a)print i";"a[i]}' 
    | sort -t';' -n -r -k2 | head -n100 
    | sort -n -t';' -k1 > /tmp/a.txt &;

    sed -e s/\"//g groups.txt | cut -d';' -f1,2 
    | sort -n -t';' -k 1 > /tmp/b.txt &;

    wait;

    join -t';' -1 1 -2 1 -o2.2 1.2 /tmp/a.txt /tmp/b.txt | sort -t';' -k2 -n -r;

## 其它特殊的非常有用的shell命令 ##
    yes 无限重复，配合head使用
    seq 生成序列，类似于python:range
    paste 按列合并文件
    tee 管道分流
    xargs 分割参数，并行计算
    parallel 并行计算
    awk '{system("cmd")}' 逐行调用外部命令

## 牛逼的管道合并和分流操作符 ##
    <()
    cat <(command1) <(command2)
    paste <(command1) <(command2)
    >()
    command0 | tee >(command1) >(command2) >(command3) | command4
    http://serverfault.com/questions/171095/how-do-i-join-two-named-pipes-into-single-input-stream-in-linux
    http://tldp.org/LDP/abs/html/process-sub.html

## 最后来一个比较酷的 抓取所有主题站所有文章的缩略图##
    curl http://www.guokr.com/site/ 2>/dev/null 
    | awk '/<ul class=\"all-sites\">/{p=1};p;/<\/ul>/{p=0}' 
    | sed -n '/<a itemprop/p' | sed 's/  //g' 
    | cut -d' ' -f3 | cut -d'"' -f2 
    | awk '{printf("%s ", $1) ; system("curl "$0" 2>/dev/null | grep 末页 | cut -d= -f3 | cut -d\047\042\047 -f1")}' 
    | awk -F' ' '{printf("%s ",$1);system("seq -s\047 \047 "$2)}' 
    | awk -F' ' '{for(i=2;i<NF;i++){print $1"?page="$i}}' 
    | xargs -P 10 -n 1 curl 2>/dev/null 
    | awk '/<div class=\"article-pic\">/{p=1};p;/<\/div>/{p=0}' 
    | sed -n '/<img src/ p' | sed 's/  //g' | cut -d'"' -f8 
    | sed -n '/img1\.guokr\.com\/thumbnail.*166x119\.\(jpg\|png\|gif\)$/p' > seeds.txt;

    cat seeds.txt | sort -u | xargs -P 50 -n 1 wget;

    paste <(yes "wget" | head -n `cat seeds.txt | wc -l`) seeds.txt -d' ' | parallel -j20;

## 推荐资源 ##
    《Unix Shell编程》
    《The AWK programming language》
    《Sed & Awk 101 Hacks》
     GNU Parallel http://www.gnu.org/software/parallel/
