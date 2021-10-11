![1616226419410](C:%5Cnote%5Cjava%5CUntitled%5Cimg%5C1616226419410.png)

![1616226540265](C:%5Cnote%5Cjava%5CUntitled%5Cimg%5C1616226540265.png)

<font color="red">小心nextLine()会吸附int变量后的空格</font>

![1616226652736](C:%5Cnote%5Cjava%5CUntitled%5Cimg%5C1616226652736.png)

//next()和nextLine()的区别详解
/*next()方法在读取内容时，会过滤掉有效字符前面的无效字符，对输入有效字符之前遇到的空格键、Tab键或Enter键等结束符，next()方法会自动将其过滤掉；只有在读取到有效字符之后，next()方法才将其后的空格键、Tab键或Enter键等视为结束符；所以next()方法不能得到带空格的字符串。
 */

/*nextLine()方法字面上有扫描一整行的意思，它的结束符只能是Enter键，即nextLine()方法返回的是Enter键之前没有被读取的所有字符，它是可以得到带空格的字符串的。
 */
