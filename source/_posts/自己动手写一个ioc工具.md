title: 自己动手写一个ioc工具
date: 2015-01-02 17:00:33
tags: [java]
category: 开发
description: IOC是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。本文介绍了如何使用Java编写一个简单的IOC工具。
---

### 前言
可能会有人问，你为什么又重复造轮子呢，已经有spring可以用啦。我倒不认为重复造轮子是不好的，知道怎么造轮子，并且把轮子造出来对解决开发中遇见的问题是非常有帮助的，并且能够应用到实际项目中页是很有成就感的一件事情。我本人就是在重复造轮子中慢慢成长的。不废话了，进入正题。

引用维基百科对控制反转的解释：
> 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

今天我们要使用的方法便是**依赖注入**中的**基于注解**和**基于set**方式。

### 分析实现方式

#### 定义注解类
首先，我们定义两个注解`@Bean`和`@Resource`，前者用于标注一个对象需要容器管理，而后者用于标注所依赖的对象。注解`@Bean`的值不允许为空，`@Resource`的值可以为空(则使用字段名称)。至于关于注解的语法，请自行查询相关资料。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Bean {
    
    String value();
    
}

@Target({FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Resource {
    
    String value() default "";
    
}
```

<!-- more -->
#### 扫描指定包下的类

```java
public class Scanner {
    
    private ClassLoader classLoader = null;

    /**
	 *  扫描包下面的类
	 * @param packageName
     */
    public Set<Class<?>> scanPackage(String packageName) {
        classLoader = Thread.currentThread().getContextClassLoader();
        //是否循环搜索包
        boolean recursive = true;
        //存放扫描到的类
        Set<Class<?>> classes = new LinkedHashSet<Class<?>>();

        //将包名转换为文件路径
        String packageDirName = packageName.replace('.', '/');

        try {
            Enumeration<URL> dirs = classLoader.getResources(packageDirName);
            
            while (dirs.hasMoreElements()) {
                URL url = dirs.nextElement();
                String protocol = url.getProtocol();
                if ("file".equals(protocol)) {
                    // 获取包的物理路径
                    String filePath = URLDecoder.decode(url.getFile(), "UTF-8");
                    // 以文件的方式扫描整个包下的文件 并添加到集合中
                    findAndAddClassesInPackageByFile(packageName, filePath,
                            recursive, classes);
                }
            }
            return classes;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
	  * 以文件的形式来获取包下的所有Class
	  * @param packageName 包名
	  * @param packagePath 包的物理路径
	  * @param recursive 是否递归扫描
	  * @param classes 类集合
     */
    private void findAndAddClassesInPackageByFile(String packageName,
                                                        String packagePath,
                                                        final boolean recursive, Set<Class<?>> classes) {
        // 获取此包的目录 建立一个File
        File dir = new File(packagePath);
        // 如果不存在或者 也不是目录就直接返回
        if (!dir.exists() || !dir.isDirectory()) {
            System.out.println("用户定义包名 " + packageName + " 下没有任何文件");
            return;
        }
        // 如果存在 就获取包下的所有文件 包括目录
        File[] dirFiles = dir.listFiles(new FileFilter() {
            // 自定义过滤规则 如果可以循环(包含子目录) 或则是以.class结尾的文件(编译好的java类文件)
            public boolean accept(File file) {
                return (recursive && file.isDirectory())
                        || (file.getName().endsWith(".class"));
            }
        });
        // 循环所有文件
        for (File file : dirFiles) {
            // 如果是目录 则递归继续扫描
            if (file.isDirectory()) {
                findAndAddClassesInPackageByFile(packageName + "."
	                                - file.getName(), file.getAbsolutePath(), recursive,
                        classes);
            } else {
                // 如果是java类文件 去掉后面的.class 只留下类名
                String className = file.getName().substring(0,
                        file.getName().length() - 6);
                try {
                    // 添加到集合中去
                    classes.add(classLoader.loadClass(packageName + '.' + className));
                } catch (ClassNotFoundException e) {
                    System.out.println("添加用户自定义视图类错误 找不到此类的.class文件");
                    e.printStackTrace();
                }
            }
        }
    }
}
```


这段代码一部分是spring中的，我就直接拿来用了，理解什么意思就行。扫描完成后会获得一个Set集合，存放着所有的Class对象。

##### 分析注解及依创建对象，注入依赖

1. 遍历类集合，如果检测到有`@Bean`注解则实例化对象存放到Map中，然后继续扫描该类下的所有field，如果发现`@Resource`注解则记录依赖值Map中。

```java
/**
  * 扫描注解,创建对象,记录依赖关系
  * @param classes 类集合
  */
 private void createBeansAndScanDependencies(Set<Class<?>> classes) {

    Iterator<Class<?>> iterator = classes.iterator();
    while (iterator.hasNext()) {
        Class<?> item = iterator.next();
        Bean annotation = item.getAnnotation(Bean.class);
        if (annotation != null) {
            String beanName = annotation.value();
            try {
                this.beans.put(annotation.value(), item.newInstance());
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }

            /*
            记录依赖关系
             */
            Field[] fields = item.getDeclaredFields();

            for (int i = 0; i < fields.length; i++) {
                Field field = fields[i];
                Resource fieldAnnotation = field.getAnnotation(Resource.class);
                if (fieldAnnotation != null) {
                    //获取依赖的bean的名称,如果为null,则使用字段名称
                    String resourceName = fieldAnnotation.value();
                    if (StringUtil.isEmpty(resourceName)) {
                        resourceName = field.getName();
                    }

                    this.dependencies.put(beanName.concat(".").concat(field.getName()), resourceName);
                }

            }
        }
    }
 }
```

遍历依赖关系Map，进行依赖注入。

```java
 /**
  * 扫描依赖关系并注入bean
  */
private void injectBeans() {

    Iterator<Map.Entry<String, String>> iterator = dependencies.entrySet().iterator();
    while (iterator.hasNext()) {
        Map.Entry<String, String> item = iterator.next();
        String key = item.getKey();
        String value = item.getValue();//依赖对象的值
        String[] split = key.split("\\.");//数组第一个值表示bean对象名称,第二个值为字段属性名称
        try {
            PropertyUtils.setProperty(beans.get(split[0]), split[1], beans.get(value));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
    }
}
```

再提供一个函数用于从工厂中获取Bean对象。

```java
/**
 * 获取bean对象
 * @param name beanName
 * @return
 */
public Object getBean(String name) {

    return beans.get(name);
}
```


### 测试

在com.mlongbo.sunflower.ioc.bean包中定义两个Bean:


```java
@Bean("jack")
public class Person {
    private String name = "jack";

    public String getName() {
        return name;
    }
}

@Bean("hi")
public class Hi {
    @Resource("jack")
    private Person person;

    /**
     * Say Hello
     */
    public void sayHello() {

        System.out.println("Hello " + person.getName());
        
    }

    public void setPerson(Person person) {
        this.person = person;
    }
}
```



最后再写个例子单元测试下。

```java
@Test
public void test() {

    Set<Class<?>> classes = new Scanner().scanPackage("com.mlongbo.sunflower.ioc.bean");

    BeanContext.me().init(classes);

    Hi hi = (Hi) BeanContext.me().getBean("hi");
    hi.sayHello();
}
```


至此，大功告成～ 如果有错误和需要完善的地方还请指正。完整代码请查看[GitHub仓库](https://github.com/lenbo-ma/sunflower-ioc)
