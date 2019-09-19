在当前目录下编译所有的.java文件，并存放到bin目录下

```shell
javac -d bin src/module-info.java src/com/itranswarp/sample/*.java
```

把bin目录下的所有class文件先打包成jar，在打包的时候，注意传入--main-class参数，让这个jar包能自己定位main方法所在的类

```shell
jar --create --file hello.jar --main-class com.itranswarp.sample.Main -C bin .
```

使用JDK自带的jmod命令把一个jar包转换成模块

```shell
jmod create --class-path hello.jar hello.jmod
```

要运行一个jar，我们使用java -jar xxx.jar命令。要运行一个模块，我们只需要指定模块名

```shell
java --module-path hello.jmod --module hello.world
```

结果是一个错误

```shell
Error occurred during initialization of boot layer
java.lang.module.FindException: JMOD format not supported at execution time: hello.jmod
```

原因是.jmod不能被放入--module-path中。换成.jar就没问题了

```shell
java --module-path hello.jar --module hello.world
```

打包JRE

```shell
jlink --module-path hello.jmod --add-modules java.base,java.xml,hello.world --output jre/
```

我们在--module-path参数指定了我们自己的模块hello.jmod，然后，在--add-modules参数中指定了我们用到的3个模块java.base、java.xml和hello.world，用,分隔。最后，在--output参数指定输出目录。

现在，在当前目录下，我们可以找到jre目录，这是一个完整的并且带有我们自己hello.jmod模块的JRE。试试直接运行这个JRE

```shell
jre/bin/java --module hello.world
```

class的这些访问权限只在一个模块内有效，模块和模块之间，例如，a模块要访问b模块的某个class，必要条件是b模块明确地导出了可以访问的包。
只有它声明的导出的包，外部代码才被允许访问。换句话说，如果外部代码想要访问我们的hello.world模块中的com.itranswarp.sample.Greeting类，我们必须将其导出

```java
module hello.world {
    exports com.itranswarp.sample;
    requires java.base;
    requires java.xml;

}
```


