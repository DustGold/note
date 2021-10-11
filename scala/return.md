![1615543198606](C:%5Cnote%5Cscala%5Creturn.assets%5C1615543198606.png)

![1615543210904](C:%5Cnote%5Cscala%5Creturn.assets%5C1615543210904.png)

![1615543224856](C:%5Cnote%5Cscala%5Creturn.assets%5C1615543224856.png)

![1615543243105](C:%5Cnote%5Cscala%5Creturn.assets%5C1615543243105.png)

<font color="red">如果函数明确使用return关键字，那么函数返回就不能使用自行推断了，这时要明确写成： 返回类型= ，当然你什么都不写，即使有return 返回值为（）</font>

什么都不写  ==   : Unit=

<font color="red">如果函数明确声明无返回值（声明Unit）,那么函数体中即使使用return关键字也不会有返回值</font>

<font color="red">如果明确函数无返回值或不确定返回值类型，那么返回值类型可以省略（或声明为Any）</font>

![1615550958967](C:%5Cnote%5Cscala%5Creturn.assets%5C1615550958967.png)

<font color="red">Scala函数的形参，在声明参数时，直接赋初始值（默认值），这时调用函数时，如果没有指定实参，则会使用默认值。如果指定了实参，则实参会覆盖默认值</font>

```scala
def sayOk(str:String = "jack"): Unit ={//str形参的默认值jack
    printfln(str)
}
```

![Screenshot_20210312_204231_tv.danmaku.bili](C:%5Cnote%5Cscala%5Creturn.assets%5CScreenshot_20210312_204231_tv.danmaku.bili.jpg)

<font color="red">scala函数的形参默认是val的，因此不能在函数中进行修改</font>



<font color="red">递归函数未执行之前是无法推断出来结果类型，在使用时必须有明确的返回值类型</font>

```scala
def f8(n:Int)={	//错误，递归不能使用类型推断，在使用时必须有明确的返回值类型
	.....

	f8(n-1)

}
```



