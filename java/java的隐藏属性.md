# Java的隐藏属性

<font color="blue">Java中绝对没有所谓的属性重写，但是我们可以定义与父类相同的属性名的属性。具体细节如下。</font>

```java
public class FieldHidden {
	public static void main(String[] args) {
		Sup sub1 = new Sub();//父类指针指向子类对象
		Sub sub2 = new Sub();
		System.out.println(sub1.num); //100
		System.out.println(sub2.num); //10
	}
}
class Sub extends Sup{//子类
	int num = 10;
	String str = "sub";
	public int getNum() {
		return this.num;
	}
	public void add() {
		num+=10;
	}
}
class Sup{//父类
	int num = 100;
	String str = "sup";
	public int getNum() {
		return this.num;
	}
	public void add() {
		num+=100;
	}
}

```

 从上面实例可以看出：

1、成员变量不能像方法一样被重写

2、当子类定义一个跟父类相同名字的字段，子类就定义了一个新的字段。

3、当我们使用子类指针访问子类对象，就可以访问到子类中与父类属性名相同的属性变量；当我们使用父类指针访问父类对象时，就可以访问到父类中与子类属性名相同的属性变量；

```java
public class FieldHidden {
	public static void main(String[] args) {
		Sup sub1 = new Sub();//父类指针指向子类对象
		sub1.add();
		System.out.println(sub1.num);//100
		System.out.println(((Sub)sub1).num);//20
	}
}
class Sub extends Sup{//子类
	int num = 10;
	String str = "sub";
	public int getNum() {
		return this.num;
	}
	public void add() {
		num+=10;
	}
}
class Sup{//父类
	int num = 100;
	String str = "sup";
	public int getNum() {
		return this.num;
	}
	public void add() {
		num+=100;
	}
}

```

当我们调用sub1.add()时，我们调用的是子类本身的方法，因为动态绑定原理。即jvm会调用对象本身的方法。（具体虚实函数，动态绑定知识请自行百度，本文不再赘述）。

即

1、sub1.add()调用的是子类（Sub）的add();故其sum增加的也应该是子类（Sub）中的sum。

2、sub1.sum 因为sum是属性，没有动态绑定原理，则显示的是父类(Sup)的sum。

3、((Sub)sub1).num 将sub1的指针先强制转换为子类(Sub)的，再打印sum属性，即打印子类(Sub)的属性。