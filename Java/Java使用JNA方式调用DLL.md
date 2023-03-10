## 注意

Java使用JNA方式调用DLL时，需保证JDK位数与DLL位数相同，即，如果你本地JDK版本是64位的，那么必须编译64位版本DLL，才能使用JNA方式来调用

## 引包

~~~xml
<dependency>
    <groupId>net.java.dev.jna</groupId>
    <artifactId>jna</artifactId>
    <version>5.5.0</version>
</dependency>
~~~

## DLL接口

DLL声明接口格式如下

~~~c++
    function CreditTransABC(strIn, StrOut:pChar):Integer; stdcall;
~~~
Java声明对应接口如下。其中Java接口类中方法名必须与DLL中一致

~~~java
public interface IHiBankLibrary extends Library {

    IHiBankLibrary INSTANCE = Native.loadLibrary("E:\\JavaDemo\\jna-demo\\data\\HiBankW64.dll", IHiBankLibrary.class);

    int CreditTransABC(String strIn, byte[] strOut);
}
~~~

## 调用 DLL接口

~~~java
public class HiBankJnaExecutor {

    public void creditTransABC() {
        byte[] strOut = new byte[1024];

        IHiBankLibrary.INSTANCE.CreditTransABC("1", strOut);
        String result = new String(strOut, StandardCharsets.UTF_8).trim();
        System.out.println(result);
    }
}
~~~

## 测试

~~~java
    @Test
    public void CreditTransABC() {
        new HiBankJnaExecutor().creditTransABC();
    }
~~~



---

GitHub：[https://github.com/dkbnull/JavaDemo](https://github.com/dkbnull/JavaDemo)

Gitee：[https://gitee.com/dkbnull/JavaDemo](https://gitee.com/dkbnull/JavaDemo)

CSDN：[https://blog.csdn.net/dkbnull/article/details/105037245](https://blog.csdn.net/dkbnull/article/details/105037245)

---