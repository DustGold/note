### linux命令--umask



**本文转载自 https://www.cnblogs.com/sench/p/8933638.html**

## 一、umask介绍

在linux系统中，我们创建一个新的文件或者目录的时候，这些新的文件或目录都会有默认的访问权限，umask命令与文件和目录的默认访问权限有关。若用户创建一个文件，则文件的默认访问权限为 `-rw-rw-rw-` ，创建目录的默认权限`drwxrwxrwx` ，而umask值则表明了需要从默认权限中去掉哪些权限来成为最终的默认权限值。

## 二、umask值的含义

可以使用命令 `umask` 来查看umask值

```
hadoop@sench-pc:~$ umask 
0002
12
```

可以看到umask值为0002，其中第一个0与特殊权限有关，可以暂时不用理会，后三位002则与普通权限(rwx)有关，其中002中第一个0与用户(user)权限有关，表示从用户权限减0，也就是权限不变，所以文件的创建者的权限是默认权限(rw)，第二个0与组权限（group）有关，表示从组的权限减0，所以群组的权限也保持默认权限（rw），最后一位2则与系统中其他用户（others）的权限有关，由于w=2，所以需要从其他用户默认权限（rw）减去2，也就是去掉写（w）权限，则其他人的权限为rw - w = r，则创建文件的最终默认权限为 `-rw-rw-r--` 。同理，目录的默认权限为 `drwxrwxrwx` ，则d rwx rwx rwx - 002 = (d rwx rwx rwx) - (- — --- -w-) = d rwx rwx r-x，所以用户创建目录的默认访问权限为 `drwxrwxr-x` 。我们通过下面的例子验证一下：

```
hadoop@sench-pc:~$ umask 
hadoop@sench-pc:~$ touch test.txt
hadoop@sench-pc:~$ ls -l test.txt 
-rw-rw-r-- 1 hadoop hadoop 0 4月  24 20:31 test.txt
hadoop@sench-pc:~$ mkdir test
hadoop@sench-pc:~$ ls -al test
总用量 8
drwxrwxr-x  2 hadoop hadoop 4096 4月  24 20:32 .
drwxr-xr-x 52 hadoop hadoop 4096 4月  24 20:32 ..
123456789
```

可以看到文件test.txt的权限为 `-rw-rw-r--` ，目录test的权限为 `drwxrwxr-x` ( `.` 代表当前目录，也就是test目录的属性）。

umask 命令显示的为umask的数字值，还可以使用命令 umask -S 来显示umask的符号值：

```
hadoop@sench-pc:~$ umask -S
u=rwx,g=rwx,o=rx
12
```

可以看出(rwx rwx rwx) - (rwx rwx r-x) = (— --- -w-) = 002 。

## 三、更改umask值

可以通过命令 `umask 值` 的方式来更改umask值，比如我要把umask值改为027，则使用命令 `umask 027` 即可。改成027后，用户权限不变，群组权限减掉2，也就是去掉写（w）权限，其他用户减7，也就是去掉读写执行权限(rwx)，所以其他用户没有访问权限。

```
hadoop@sench-pc:~$ umask 027
hadoop@sench-pc:~$ umask
hadoop@sench-pc:~$ touch test.txt
hadoop@sench-pc:~$ ls -l test.txt
-rw-r----- 1 hadoop hadoop 0 4月  24 20:49 test.txt
hadoop@sench-pc:~$ mkdir test
hadoop@sench-pc:~$ ls -al test
总用量 8
drwxr-x---  2 hadoop hadoop 4096 4月  24 20:49 .
drwxr-xr-x 52 hadoop hadoop 4096 4月  24 20:49 ..
12345678910
```

可以看到文件的默认访问权限变为了 `-rw-r-----` ，目录test的默认访问权限变为了 `drwxr-x---` 。这种方式并不能永久改变umask值，只是改变了当前会话的umask值，打开一个新的terminal输入umask命令，可以看到umask值仍是默认的002。要想永久改变umask值，则可以修改文件/etc/bashrc，在文件中添加一行 `umask 027` 。

## 四、总结

当我们想改变创建文件和目录时的默认访问权限，则可以通过umask命令来实现。