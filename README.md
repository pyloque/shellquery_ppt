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

## 总数  ##
    select count(1) from rank_item

    wc -l rank_items.txt | cut -d' ' -f1

    cat rank_items.txt | wc -l

## 性情小组活跃用户数量 ##
    select count(distinct(object_name))
    from rank_item
    where rank_name='group_hot_member:30'

    sed -n -e '/"group_hot_member\:30"\;/ p' rank_items.txt | cut -d';' -f3 | sort | uniq | wc -l

## 性情小组活跃用户TOP10  ##

    select object_name,sum(score) total_score
    from rank_item
    where rank_name='group_hot_member:30'
    group by object_name
    order by total_score desc
    limit 10

    sed -n -e '/"group_hot_member\:30"\;/ p' rank_items.txt | awk -F';' '{a[$3]+=$5}END{for(i in a)print i,a[i]}' | sort -t' ' -n -r -k 2 | head -n 10

## 性情小组活跃用户(总分大于80的)  ##

    select object_name,sum(score) total_score
    from rank_item
    where rank_name='group_hot_member:30'
    group by object_name
    order by total_score desc
    limit 10

    sed -n -e '/"group_hot_member\:30"\;/ p' rank_items.txt | awk -F';' '{a[$3]+=$5}END{for(i in a)print i,a[i]}' | awk -F' ' '$2 > 80 {print $0}' |sort -t' ' -n -r -k 2 | head -n 10

##  活跃小组按总分倒排 ##
    select substr(rank_name,18) group_id, sum(score) total_score
    from rank_item
    where rank_name like 'group_hot_member:%'
    group by group_id
    order by total_score desc
    limit 100

    sed -n -e '/"group_hot_member\:/ p' rank_items.txt | cut -d';' -f2,5 | sed -e s/\"//g -e 's/group_hot_member\://g' | awk -F';' '{a[$1]+=$2}END{for(i in a)print i,a[i]}' | sort -t' ' -n -r -k2 | head -n100

## 活跃小组按总分倒排（显示小组名称） ##
    select b.group_id,a.total_score from(
    select substr(rank_name,18) group_id, sum(score) total_score
    from rank_item
    where rank_name like 'group_hot_member:%'
    group by group_id
    order by total_score desc
    limit 100
    ) a join group b on a.group_id=b.id

    sed -n -e '/"group_hot_member\:/ p' rank_items.txt | cut -d';' -f2,5 | sed -e s/\"//g -e 's/group_hot_member\://g' | awk -F';' '{a[$1]+=$2}END{for(i in a)print i";"a[i]}' | sort -t';' -n -r -k2 | head -n100 | sort -n -t';' -k1 > /tmp/a.txt &;
    sed -e s/\"//g groups.txt | cut -d';' -f1,2 | sort -n -t';' -k 1 > /tmp/b.txt &;
    wait;
    join -t';' -1 1 -2 1 -o2.2 1.2 /tmp/a.txt /tmp/b.txt | sort -t';' -k2 -n -r;
