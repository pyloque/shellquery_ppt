## select ##
cut -d';' -f1,2 questions.txt

## where ##
sed -n '/PATTERN/p' questions.txt

## sort ##
sort -t';' -k1 -n -r questions.txt

## distinct ##
sort -u

## limit ##
head -n5 
tail -n5

## join ##
join

    who.txt
    I:1
    You:2
    He:3
    She:4
    
    fruit.txt
    1:Apple
    2:Banana
    3:Pear
    4:Orange

`join -t: -j1 2 -j2 1 -o 1.1 2.2 who.txt fruit.txt`

    I:Apple
    You:Banana
    He:Pear
    She:Orange

## group by ##
awk
