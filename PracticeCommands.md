-- 注： 下面所有的命令都是shell脚本

# 行寻址

格式: [address]command

## 数字行寻址
格式: N command

### 单行
sed -n '3p' books.txt

### 行数范围(2~5行, 以逗号隔开)
sed -n '2, 5 p' books.txt

### 特殊字符($, 表示文本的最后一行)
sed -n '$p' books.txt

### 行数范围+特殊符号(2～最后一行)
sed -n '2, $ p' books.txt

### 加号操作符(+)和逗号操作符一起使用时，表示从第M行开始的N行(M, +N)
sed -n '2, +3 p' books.txt

### 波浪线操作符(~)， 表示从第M行开始没隔N行执行一次(M~N), 注：行数范围使用的是逗号操作符(,)
sed -n '2~2 p' books.txt #打印偶数行

注：如果使用的是Mac系统自带的sed命令，可能不支持~和+操作符。可以使用brewhome 重新安装GNU-SED。 brew install gnu-sed --with-default-names

## 文本行寻址： 文本模式过滤器， 正则匹配模式行寻址， SED编辑器允许指定文本模式来过滤出命令要作用的行
格式： /pattern/command

### 必须用正斜线将要指定的pattern封起来。sed编辑器会将该命令作用到包含指定文本模式的行上
sed -n '/Paulo/ p' books.txt

### 文本行寻址 + 数字行寻址(从模式匹配到的那一行起数字匹配的那一行的范围)
sed -n '/Storm/, 5 p' books.txt

### 逗号操作符(,)：指定匹配多个匹配模式
sed -n '/Storm/, /Paulo/ p' books.txt

### 加号操作符(+), 表示从第一次匹配行数开始的某N行(/pattern/, +N)
sed -n '/Paulo/, +2 p' books.txt

# 保持空间

## 用途: 在处理模式空间中的数据时，可以使用保持空间来临时保存一些行
| 命令         |   描述                      |
|:------------|:---------------------------|
| h           |  将模式空间的内容复制到保持空间 |
| H           |  将模式空间的内容附加到保持空间 |
| g           |  将保持空间的内容复制到模式空间 |
| G           |  将保持空间的内容附加到模式空间 |
| x           |  将模式空间和保持空间中的内容互换 |

sed -n 'h;n;H;x;s/\n/, /;/Paulo/!b Print; s/^/- /; :Print;p' books2.txt

# 基本命令

## SED的基本命令主要包括: DELETE, WRITE, APPEND, CHANGE, INSERT, TRANSLATE, QUIT, READ, EXECUTE

## 删除命令(d)

### 语法格式
[address1 [, address2]] d
address1和address2分别表示需要删除行数的起始位置和结束位置, 可以是数字行寻址和文本行寻址，都是可选的

注： DELETE是用来进行删除指定的行数的，值得注意的是，该命令只会删除模式空间中的指定的行，该行不会发送到输出流，原生的内容不会改变
sed 'd' books.txt
如果未设置address1和address2， 即表示在模式空间中删除文件所有行，所以没有输出

### 删除数字寻址匹配的行
sed '4d' books.txt

### 逗号操作符(,), 表示删除指定行数范围的行(N, M d)
sed '1, 4d' books.txt

### 删除文本寻址匹配的行
sed '/The Pilgrimage/ d' books.txt

### 移除两个文本匹配之间的所有行
sed '/The Alchemist/, /The Pilgrimage/ d' books.txt

注： 如果address2行数在address1之前，那么就删除从address1匹配到行数到文本结束

## 文件写入命令(w)

### 语法格式
[address1 [, address2]]w file
address1和address2代表的意思与DELETE命令一致

w - 写命令
file - 存储文件内容的文件名
注： 使用file时需要注意，当file不存在是，会创建新的文件；当该文件存在时，会覆盖该文件的内容

### 实例: 复制文件，相当于`cp`操作
sed -n 'w books.bak' books.txt

使用上述命令会创建一个`books.bak`的副本文件，可以使用下述命令来查看两个文件的不同之处

diff books.bak books.txt
echo $?

### 复制文件中的某些行
sed -n '2~2 w junk.txt' books.txt

注： 从第2行开始，每隔一行复制到junk.txt文件中(复制偶数行到junk.txt文件中)

### 存储每位作者的书到单独的文件中
sed -n -e '/Martin/ w Martin.txt' -e '/Paulo/ w Paulo.txt' -e '/Tolkien/ w Tolkien.txt' books.txt
sed -n -e '
/Martin/ w Martin1.txt
/Paulo/ w Paulo1.txt
/Tolkien/ w Tolkien1.txt
' books.txt

## 追加命令(a)

### 语法格式
[address] a Append text

### 实例：在第4行追加

