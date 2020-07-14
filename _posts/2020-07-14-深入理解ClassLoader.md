# 深入理解ClassLoader
## 什么是ClassLoader
ClassLoader是负责将 Class 的字节码形式转换成内存形式的 Class 对象。
ClassLoader是帮助JVM加载指定文件目录下的Class文件。

## ClassLoader类型有那些

- Bootstrap ClassLoader 启动类加载器，最顶层的加载类，由C++实现，负责加载%JAVA_HOME%/lib目录中或-Xbootclasspath中参数指定的路径中的，并且是虚拟机识别的（按名称）类库
- Extention ClassLoader 扩展类加载器，由启动类加载器加载，负责加载目录  %JRE_HOME%/lib/ext目录中或-Djava.ext.dirs中参数指定的路径中的jar包和class文件
- Application ClassLoader 应用类加载器，也称为系统类加载器。负责加载当前应用classpath下的所有类
- 自定义类加载器


## ClassLoader怎么加载到JVM
通过双亲委派模型，加载Class文件转换成Class对象到JVM中。






### 双亲委派模型（类树型结构）

双亲委派模型 就是 ClassLoader 加载Class文件。


#### 双亲委派模型图

![](https://raw.githubusercontent.com/kakaCat/kakacat.github.io/master/img/ClassLoader_tree_model.png)


#### 双亲委派模型ClassLoader与文件系统关系图
![](https://raw.githubusercontent.com/kakaCat/kakacat.github.io/master/img/ClassLoader_IO_file.png)

### 双亲委派模型加载顺序和优点

#### 双亲委派模型加载顺序
1. 先让父类ClassLoader去加载
2. 一直查询到最顶层
3. 如果父类加载不到Class文件，则由自己的ClassLoader加载
4. 最终加载到内存中

#### 双亲委派模型优点
- 避免Class文件的重复加载
	- 父类ClassLoader 加载子类需要的Class文件，不需要自己再次加载Class文件
- 安全保证 防止JAVA的核心API被篡改
	- 例如 String.class 核心类，被非法串改
- 隔离性功能 不同包同名类区分
	- 例如tomcat 通过不同子加载器加载Class，防止tomcat核心Class文件和其他应用文件相同问题
	

### JAVA代码实现双亲委派模型


#### ClassLoader类结构


![](https://raw.githubusercontent.com/kakaCat/kakacat.github.io/master/img/ClassLoader_IO_file.png)


#### 实现AppClassLoader和ExtClassLoader委托 要从sun.misc.Launcher聊起

##### Launcher
````
public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }

        try {
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }

        Thread.currentThread().setContextClassLoader(this.loader);
        String var2 = System.getProperty("java.security.manager");
        if (var2 != null) {
            SecurityManager var3 = null;
            if (!"".equals(var2) && !"default".equals(var2)) {
                try {
                    var3 = (SecurityManager)this.loader.loadClass(var2).newInstance();
                } catch (IllegalAccessException var5) {
                } catch (InstantiationException var6) {
                } catch (ClassNotFoundException var7) {
                } catch (ClassCastException var8) {
                }
            } else {
                var3 = new SecurityManager();
            }

            if (var3 == null) {
                throw new InternalError("Could not create SecurityManager: " + var2);
            }

            System.setSecurityManager(var3);
        }

    }

````
##### 图解Launcher 代码实现图
![](https://raw.githubusercontent.com/kakaCat/kakacat.github.io/master/img/ClassLoader_Class_app_ext.png)

#### 双亲委派模型具体实现

##### java.lang.ClassLoader$loadClass
````
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

````

##### 双亲委派模型代码实现图

![](https://raw.githubusercontent.com/kakaCat/kakacat.github.io/master/img/ClassLoader_loader.png)


### 双亲委派模型带来的问题

- 父加载器无法访问到子类加载器加载的Class对象
	- 场景 核心包为三方jar提供接口，具体实现需要对应三方服务商实现。程序直接加载核心包的接口，并调用三方实现类。这个场景违反了（双亲委派模型）


- 不同jar包中的有相同路径同名Class文件（钻石依赖）
	- 场景 A对象由B对象和C对象组装，B对象由D（v1）对象组装,C对象由D（v2）对象组装。

![](https://raw.githubusercontent.com/kakaCat/kakacat.github.io/master/img/diamond.png)


JAVA的双亲委派模型并不是强制规范,所以JAVA提供了contextClassLoader解决以上问题。

#### java.lang.Thread.contextClassLoader

通过线程中共享内存的方式解决Class对象加载问题。应用线程可以获取所有的Class对象。























