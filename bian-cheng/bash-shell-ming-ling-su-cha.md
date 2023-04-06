# bash shell命令速查

## 基础脚本

* 多命令执行：`ls;pwd`
* 命令别命：`alias cl='clear'`
* 环境变量使用：`$date`
*   用户自定义变量引用：

    `name="Jim"`

    `echo "$name"`
* 命令替换(命令输出赋予变量)：`` time=`date` `` or `time=$(date)`
* 依次执行命令(管道符)：ls -l `| sort`
* 数学运算：`expr 5 \* 2` or `$[5 * 2]`
* 浮点数运算(scale为保留小数点数)：`echo "scale=4; 3.44 / 5" | bc`
* 查看退出状态码：`echo $?`

## 结构化命令

* if-then

```
if pwd
then
    echo "Succ"
fi
```

* if-then-else

```
if grep flag= flag.txt
then
    cat flag.txt
else
    echo "404"
fi
```

* if-elif

```
if grep flag= flag.txt
then
    cat flag.txt
elif grep a flag.txt
then
    echo "200"
fi
```

*   test命令(用于条件判断)与字符串比较



    <figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>



    <figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>



    <figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

```
nb="lbw"
lbw="nbw"
if [ $nb = $lbw]
then 
    echo "lbwnb"
else
    echo"No lbw"
fi
```

* case命令

```
cmd=$1

case $cmd in 
lbw | nb)
    echo "lbwnb";;
cxk | ikun)
    echo "cxknb"
esac
```

