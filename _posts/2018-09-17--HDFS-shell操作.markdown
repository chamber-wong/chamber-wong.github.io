---
layout:     post
title:      "HDFS的shell操作详解"
subtitle:   ""
date:       2018-09-11 12:00:00
author:     "Chamber"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - hadoop
    - HDFS
---
<!-- MarkdownTOC -->

- HDFS的shell操作
    - 基本语法格式
    - 参数列表
        - appendToFile
        - cat
        - checksum
        - chgrp
        - chmod
        - chown
        - copyFromLocal
        - copyToLocal
        - count
        - cp
        - createSnapshot
        - deleteSnapshot
        - df
        - du
        - dus
        - expunge
        - find
        - get
        - getfacl
        - getfattr
        - getmerge
        - help
        - ls
        - lsr
        - mkdir
        - moveFromLocal
        - moveToLocal
        - mv
        - put
        - rm
        - rmdir
        - rmr
        - setfacl
        - setfattr
        - setrep
        - stat
        - tail
        - test
        - text
        - touchz
        - truncate
        - usage

<!-- /MarkdownTOC -->

# HDFS的shell操作

## 基本语法格式

```
hadoop fs <args>
```

所有FS shell命令都将路径URI作为参数。

## 参数列表

### appendToFile  

>  将文件添加到块后

```shell
$ hadoop fs -appendToFile <localsrc> ... <dst>
```

- 语法说明

```shell
$ hadoop fs -appendToFile localfile /user/hadoop/hadoopfile
$ hadoop fs -appendToFile localfile1 localfile2 /user/hadoop/hadoopfile
$ hadoop fs -appendToFile localfile hdfs://nn.example.com/hadoop/hadoopfile
$ hadoop fs -appendToFile - hdfs://nn.example.com/hadoop/hadoopfile Reads the input from stdin.
```

**执行成功返回0,执行失败返回-1 **

### cat
> 查看文件内容

```shell
$ hadoop fs -cat [-ignoreCrc] URI [URI ...]
```

参数:

- -ignoreCrc 禁用verification验证

```shell
$ hadoop fs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
$ hadoop fs -cat file:///file3 /user/hadoop/file4
```

**执行成功返回0,执行失败返回-1 **

### checksum

>  返回文件的校验和信息

```shell
$ hadoop fs -checksum URI
```

Example:

```shell
$ hadoop fs -checksum hdfs://nn1.example.com/file1
$ hadoop fs -checksum file:///etc/hosts
```



### chgrp

>  更改所属组。用户必须是文件的所有者，否则必须是超级用户。

```shell
$ hadoop fs -chgrp [-R] GROUP URI [URI ...]
```

选项:

- -R 选项将通过目录结构**递归**地进行更改

### chmod

>  更改文件的权限。使用-R，通过目录结构递归更改。用户必须是文件的所有者，否则必须是超级用户。

```shell
$  hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]
```

选项:

- -R 选项将通过目录结构**递归**地进行更改



### chown

>  Change the owner of files. The user must be a super-user. 更改所有者

```shell
$ hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```

选项:

- -R 选项将通过目录结构**递归**地进行更改



### copyFromLocal

>  与fs -put类似,但是**源文件仅限于本地文件**

```shell
$ hadoop fs -copyFromLocal <localsrc> URI
```

选项:

- `-p`：保留访问和修改时间，所有权和权限。（假设权限可以跨文件系统传播）
- `-f`：覆盖目标（如果已存在）。
- `-l`：允许DataNode懒惰地将文件持久保存到磁盘，**小心使用**。
- `-d`：使用后缀`._COPYING_`跳过创建临时文件。





### copyToLocal

>  与get命令类似，但目标仅限于本地文件引用

```shell
$ hadoop fs -copyToLocal [-ignorecrc] [-crc] URI <localdst>
```



### count

>  计算与指定文件模式匹配的路径下的目录，文件和字节数。文件大小和使用情况

```shell
$ hadoop fs -count [-q] [-h] [-v] [-x] [-t [<storage type>]] [-u] <paths>
```

选项:

- -count 默认输入列为:DIR_COUNT，FILE_COUNT，CONTENT_SIZE，PATHNAME

- -q：输出列为：QUOTA，REMAINING_QUOTA，SPACE_QUOTA，REMAINING_SPACE_QUOTA，DIR_COUNT，FILE_COUNT，CONTENT_SIZE，PATHNAME
- -u : 的输出列为：QUOTA，REMAINING_QUOTA，SPACE_QUOTA，REMAINING_SPACE_QUOTA，PATHNAME
- -t : 选项显示每种存储类型的配额和使用情况。如果未给出-u或-q选项，则忽略-t选项。可以在-t选项中使用的可能参数列表（除参数“”之外不区分大小写）：“”，“all”，“ram_disk”，“ssd”，“disk”或“archive”。
- -h : 选项以人类可读格式显示大小。
- -v  选项显示标题行。

Example:

```shell
hadoop fs -count hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -count -q hdfs://nn1.example.com/file1
hadoop fs -count -q -h hdfs://nn1.example.com/file1
hadoop fs -count -q -h -v hdfs://nn1.example.com/file1
hadoop fs -count -u hdfs://nn1.example.com/file1
hadoop fs -count -u -h hdfs://nn1.example.com/file1
hadoop fs -count -u -h -v hdfs://nn1.example.com/file1
```



### cp

>  将文件从源复制到目标。此命令也允许多个源，在这种情况下，目标必须是目录。

```shell
$ hadoop fs -cp [-f] [-p | -p[topax]] URI [URI ...] <dest>
```

选项:

- -f : 如果目标已经存在,则覆盖目标

- -p选项将保留文件属性[topx]（时间戳，所有权，权限，ACL，XAttr）

Example:

```shell
$ hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2
$ hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir
```

Returns 0 on success and -1 on error.

### createSnapshot

>  创建快照

### deleteSnapshot

> 删除快照

### df

>  显示可用空间

```shell
$ hadoop fs -df [-h] URI [URI ...]
```

参数:

- -h : 以人类可读(“human-readable”)的方式显示(e.g 64.0m instead of 67108864)

Example:

```shell
$ hadoop dfs -df /user/hadoop/dir1
```



### du

>  显示文件和目录的大小

```shell
$ hadoop fs -du [-s] [-h] [-x] URI [URI ...]
```

参数:

- -s  显示文件长度的摘要.加上改参数将从上一级目录开始查看
- -h  以人类可读的方式显示
- -x  

Example:

```shell

$ hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 dfs://nn.example.com/user/hadoop/dir1
```



### dus

>  显示摘要信息,等同于du -s

```shell
$ hadoop fs -dus <args>
```

### expunge

>  将文件扔进回收站中

```shell
$ hadoop fs -expunge
```



### find

>  查找与指定表达式匹配的所有文件，并将选定的操作应用于它们。如果未指定*路径*，则默认为当前工作目录。如果未指定表达式，则默认为-print。

```shell
$ hadoop fs -find <path> ... <expression> ...
```

参数:

- -name pattern 
  -iname pattern

  如果文件的基名与使用标准文件系统通配符的模式匹配，则求值为true。如果使用-iname，则匹配不区分大小写。

- -print 
  -print0

  始终评估为true。导致将当前路径名写入标准输出。如果使用-print0表达式，则附加ASCII NULL字符。

Example:

```shell
hadoop fs -find / -name test -print
```





### get

>  将文件复制到本地文件系统。可以使用-ignorecrc选项复制CRC校验失败的文件。可以使用-crc选项复制文件和CRC。

```shell
$ hadoop fs -get [-ignorecrc] [-crc] [-p] [-f] <src> <localdst>
```

参数:

- `-p`：保留访问和修改时间，所有权和权限。（假设权限可以跨文件系统传播）
- `-f`：覆盖目标（如果已存在）。
- `-ignorecrc`：对下载的文件进行跳过CRC校验。
- `-crc`：为下载的文件写入CRC校验和。

Example:

```shell
$ hadoop fs -get / user / hadoop / file localfile
$ hadoop fs -get hdfs：//nn.example.com/user/hadoop/file localfile
```



### getfacl

>  显示文件和目录的访问控制列表（ACL）。如果目录具有默认ACL，则getfacl还会显示默认ACL。

```shell
$ hadoop fs -getfacl [-R] <path>
```

选项：

- -R：递归列出所有文件和目录的ACL。
- *path*：要列出的文件或目录。

Example:

```shell
hadoop fs -getfacl / file
hadoop fs -getfacl -R / dir
```





### getfattr

>  显示文件或目录的扩展属性名称和值（如果有）。

```shell
$ hadoop fs -getfattr [-R] -n name | -d [-e en] <path>
```

选项：

- -R：递归列出所有文件和目录的属性。
- -n name：转储指定的扩展属性值。
- -d：转储与pathname关联的所有扩展属性值。
- -e *encoding*：检索后*对代码*值进行编码。有效编码为“text”，“hex”和“base64”。编码为文本字符串的值用双引号（“）括起来，编码为十六进制和base64的值分别以0x和0为前缀。
- *path*：文件或目录。

Example:

```shell
$ hadoop fs -getfattr -d / file
$ hadoop fs -getfattr -R -n user.myAttr / dir
```



### getmerge

>  将源目录和目标文件作为输入，并将src中的文件连接到目标本地文件。可选地，-nl可以设置为在每个文件的末尾添加换行符（LF）。-skip-empty-file可用于在空文件的情况下避免不需要的换行符。

```shell
$ hadoop fs -getmerge [-nl] <src> <localdst>
```

Example:

```shell
$ hadoop fs -getmerge -nl / src /opt/output.txt
$ hadoop fs -getmerge -nl /src/file1.txt /src/file2.txt /output.txt
```



### help

```shell
$ hadoop fs -help
```

### ls

>  返回文件的校验和信息,查看目录或信息

```shell
$ hadoop fs -ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] <args>
```

选项：

- -C：仅显示文件和目录的路径。
- -d：目录列为纯文件。
- -h：以人类可读的方式格式化文件大小（例如64.0m而不是67108864）。
- -q：打印？而不是不可打印的字符。
- -R：递归列出遇到的子目录。
- -t：按修改时间排序输出（最近的第一个）。
- -S：按文件大小排序输出。
- -r：反转排序顺序。
- -u：使用访问时间而不是修改时间进行显示和排序。

Example:

```shell
$ hadoop fs -ls / user / hadoop / file1
```



### lsr

> ls的递归版本

```shell
$ hadoop fs -lsr <args>
```



### mkdir

> 将路径uri作为参数并创建目录。

```shell
$ hadoop fs -mkdir [-p] <paths>
```

选项：

- -p选项行为与Unix mkdir -p非常相似，沿路径创建父目录

Example:

```shell
$ hadoop fs -mkdir / user / hadoop / dir1 / user / hadoop / dir2
$ hadoop fs -mkdir hdfs：//nn1.example.com/user/hadoop/dir 
$ hdfs：//nn2.example.com/user/hadoop/dir
```



### moveFromLocal

>  与put命令类似，只是在复制后删除了源localsrc。

```shell
$ hadoop fs -moveFromLocal <localsrc> <dst>
```



### moveToLocal

```shell
$ hadoop fs -moveToLocal [-crc] <src> <dst>
```



### mv

> 将文件从源移动到目标。此命令也允许多个源，在这种情况下，目标需要是目录。**不允许跨文件系统移动文件**。

```shell
$ hadoop fs -mv URI [URI ...] <dest>
```

Example:

```shell
$ hadoop fs -mv / user / hadoop / file1 / user / hadoop / file2
$ hadoop fs -mv hdfs：//nn.example.com/file1 hdfs：//nn.example.com/file2 hdfs：//nn.example.com/file3 hdfs：//nn.example.com/dir1
```



### put

>  将单个src或多个srcs从本地文件系统复制到目标文件系统。如果源设置为“ - ”，还从stdin读取输入并写入目标文件系统
>
> 如果文件已存在，则复制失败，除非给出-f标志。

```shell
$ hadoop fs -put [-f] [-p] [-l] [-d] [ - | <localsrc1> .. ]. <dst>
```

选项：

- `-p`：保留访问和修改时间，所有权和权限。（假设权限可以跨文件系统传播）
- `-f`：覆盖目标（如果已存在）。
- `-l`：允许DataNode懒惰地将文件持久保存到磁盘，强制复制因子为1.此标志将导致持久性降低。小心使用。
- `-d`：使用后缀`._COPYING_`跳过创建临时文件。

Example:

```shell
$ hadoop fs -put localfile / user / hadoop / hadoopfile
$ hadoop fs -put -f localfile1 localfile2 / user / hadoop / hadoopdir
$ hadoop fs -put -d localfile hdfs：//nn.example.com/hadoop/hadoopfile
$ hadoop fs -put - hdfs：//nn.example.com/hadoop/hadoopfile从stdin读取输入。
```



### rm

>  删除指定为args的文件。
>
> 如果启用了垃圾箱，则文件系统会将已删除的文件移动到垃圾箱目录

```shell
$ hadoop fs -rm [-f] [-r |-R] [-skipTrash] [-safely] URI [URI ...]
```

选项：

- 如果文件不存在，-f选项将不显示诊断消息或修改退出状态以反映错误。
- -R选项以递归方式删除目录及其下的任何内容。
- -r选项等效于-R。
- -skipTrash选项将绕过垃圾桶（如果已启用），并立即删除指定的文件。当需要从超配额目录中删除文件时，这非常有用

Example:

```shell
$ hadoop fs -rm hdfs：//nn.example.com/file / user / hadoop / emptydir
```



### rmdir

>  删除目录。

```shell
$ hadoop fs -rmdir [--ignore-fail-on-non-empty] URI [URI ...]
```

选项：

- `--ignore-fail-on-non-empty`：使用通配符时，如果目录仍包含文件，请不要失败。



### rmr

> 删除的递归版本。
>
> **注意：**不推荐使用此命令。而是使用`hadoop fs -rm -r`

```shell
$ hadoop fs -rmr [-skipTrash] URI [URI ...]
```



### setfacl

> 设置文件和目录的访问控制列表（ACL）。

```shell
$ hadoop fs -setfacl [-R] [-b |-k -m |-x <acl_spec> <path>] |[--set <acl_spec> <path>]
```

选项：

- -b：删除除基本ACL条目之外的所有条目。保留用户，组和其他条目以与权限位兼容。
- -k：删除默认ACL。
- -R：递归地对所有文件和目录应用操作。
- -m：修改ACL。新条目将添加到ACL，并保留现有条目。
- -x：删除指定的ACL条目。保留其他ACL条目。
- `--set`：完全替换ACL，丢弃所有现有条目。所述*acl_spec*必须包括用户，组条目和其他用于与权限位兼容性。
- *acl_spec*：以逗号分隔的ACL条目列表。
- *path*：要修改的文件或目录。

Example:

```shell
$ hadoop fs -setfacl -m user：hadoop：rw- / file
$ hadoop fs -setfacl -x user：hadoop / file
$ hadoop fs -setfacl -b / file
$ hadoop fs -setfacl -k / dir
$ hadoop fs -setfacl --set user :: rw-，user：hadoop：rw-，group :: r - ，other :: r-- / file
$ hadoop fs -setfacl -R -m user：hadoop：rx / dir
$ hadoop fs -setfacl -m default：user：hadoop：rx / dir
```





### setfattr

> 设置文件或目录的扩展属性名称和值。

```shell
$  hadoop fs -setfattr -n name [-v value] | -x name <path>
```

选项：

- -n name：扩展属性名称。
- -v value：扩展属性值。该值有三种不同的编码方法。如果参数用双引号括起来，那么值就是引号内的字符串。如果参数的前缀为0x或0X，则将其视为十六进制数。如果参数以0或0S开头，则将其视为base64编码。
- -x name：删除扩展属性。
- *path*：文件或目录。

Example:

```shell
$ hadoop fs -setfattr -n user.myAttr -v myValue / file
$ hadoop fs -setfattr -n user.noValue / file
$ hadoop fs -setfattr -x user.myAttr / file
```



### setrep

>  更改文件的副本数。如果*path*是目录，则命令以递归方式更改以*path为*根的目录树下的所有文件的复制因子

```shell
$ hadoop fs -setrep [-R] [-w] <numReplicas> <path>
```

选项：

- -w标志请求命令等待复制完成。这可能需要很长时间。
- 接受-R标志是为了向后兼容。它没有效果。

Example:

```shell
hadoop fs -setrep -w 3 / user / hadoop / dir1
```



### stat

>  以指定格式打印有关<path>的文件/目录的统计信息。
>
> 格式接受八进制（％a）和符号（％A），文件大小（字节）（％b），类型（％F），所有者组名（％g），名称（％n），块大小（％o）的权限），复制（％r），所有者的用户名（％u），访问日期（％x，％X）和修改日期（％y，％Y）。
>
> ％x和％y将UTC日期显示为“yyyy-MM-dd HH：mm：ss”，％X和％Y显示自1970年1月1日UTC以来的毫秒数。如果未指定格式，则默认使用％y

```shell
$ hadoop fs -stat [format] <path> ...
```

Example:

```shell
hadoop fs -stat“type：％F perm：％a％u：％g size：％b mtime：％y atime：％x name：％n”/ file
```



### tail

>  显示文件的最后一千字节到stdout。

```shell
$ hadoop fs -stat [format] <path> ...
```

选项：

- -f选项将在文件增长时输出附加数据，如在Unix中一样。

Example:

```shell
$ hadoop fs -tail路径名
```



### test

>  返回文件的校验和信息

```shell
$ hadoop fs -test -[defsz] URI
```

选项：

- -d：f路径是目录，返回0。
- -e：如果路径存在，则返回0。
- -f：如果路径是文件，则返回0。
- -s：如果路径不为空，则返回0。
- -r：如果路径存在且授予读权限，则返回0。
- -w：如果路径存在且授予写入权限，则返回0。
- -z：如果文件长度为零，则返回0。

Example:

```shell
$ hadoop fs -test -e filename
```



### text

>  获取源文件并以文本格式输出文件。允许的格式是zip和TextRecordInputStream

```shell
$ hadoop fs -text <src>
```



### touchz

>  创建一个零长度的文件。如果文件存在非零长度，则返回错误

```shell
$ hadoop fs -touchz URI [URI ...]
```

Example:

```shell
$ hadoop fs -touchz pathname
```



### truncate

>  返回文件的校验和信息

```shell
$ hadoop fs -truncate [-w] <length> <paths>
```

Example:

```shell

```



### usage

> 将与指定文件模式匹配的所有文件截断为指定的长度。

```shell
$ hadoop fs -usage command
```

Example:

```shell
$ hadoop fs -truncate 55 / user / hadoop / file1 / user / hadoop / file2
$ hadoop fs -truncate -w 127 hdfs：//nn1.example.com/user/hadoop/file1
```

