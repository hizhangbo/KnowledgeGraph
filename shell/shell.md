# shell
1. 字符串 （单双引号的区别）  
_双引号内支持变量符号替换_   
```
echo "Parameter 1: $1"  
echo 'Parameter 1: $1'
```  
输出  
```
Parameter 1: 1  
Parameter 1: $1
```  
2. if条件表达式  
```
if condition  
then  
    command1   
    command2  
    ...  
    commandN   
fi
```  
3. case  
```
case $1 in  
1)  
    command1  
    command2  
    ...
    commandN  
    ;;  
2）  
    command1  
    command2  
    ...  
    commandN  
    ;;  
esac  
```  
4. while  
```
while condition  
do  
    command  
done  
```  
5. until  
 _循环执行一系列命令直至条件为 true 时停止_  
```
until condition  
do  
    command  
done  
```  
6. for  
```
for var in item1 item2 ... itemN  
do  
    command1  
    command2  
    ...  
    commandN  
done  
```  
7. 跳出循环  
break  
_命令允许跳出所有循环（终止执行后面的所有循环）_   
continue  
_不会跳出所有循环，仅仅跳出当前循环（继续执行后续循环）_  
8. test  
_用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试_  

_数值测试_  

| 参数 | 说明 |
| ------ | ------ |
| -eq | 等于 |
| -ne | 不等于 |
| -gt | 大于 |
| -ge | 大于等于 |
| -lt | 小于 |
| -le | 小于等于 |

_字符串测试_  

| 参数 | 说明 |
| ------ | ------ |
| = | 等于 |
| != | 不等于 |
| -z 字符串 | 字符串长度为0 |
| -n 字符串 | 字符串长度不为0 |

_文件测试_  

| 参数 | 说明 |
| ------ | ------ |
| -e 文件名 | 文件存在 |
| -r | 文件可读 |
| -w | 文件可写 |
| -x | 文件可执行 |
| -s | 文件不为空 |
| -d | 文件为目录 |
| -f | 文件为普通文件 |
| -c | 文件为字符型特殊文件 |
| -b | 文件为块特殊文件 |  

9. 特殊变量

| 参数 | 说明 |
| ------ | ------ |
| $0 | 当前脚本名称 |
| $n | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。 |
| $# | 传递给脚本或函数的参数个数 |
| $* | 传递给脚本或函数的所有参数 |
| $@ | 传递给脚本或函数的所有参数 |
| $? | 上个命令的退出状态，或函数的返回值 |
| $$ | 当前Shell进程ID |

10. 文件内容
```
#内容/连接改为驼峰方式
context_path=`cat $file|grep server.context-path=|sed -r 's/([a-z])\/([a-z])/\1\L\U\2/g'`
```
11. 批量文件内容替换
#! /bin/bash
```
for file in `ls ./graph-*/conf/application-haizhi${profile}.properties`
  do
    context_path=`cat $file|grep server.context-path=|sed -r 's/([a-z])\/([a-z])/\1\L\U\2/g'`
    echo $context_path
    sed -i '/^server.context-path/c\'$context_path $file
  done
```

