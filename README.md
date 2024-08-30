
## 开心一刻


有一天螃蟹出门，不小心撞倒了泥鳅泥鳅很生气地说：你是不是瞎啊！螃蟹说：不是啊，我是螃蟹


![开心一刻](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829231857375-1982663522.gif)
## 概述


[maven\-shade\-plugin](https://github.com) 官网已经介绍的很详细了，我给大家简单翻译一下



> This plugin provides the capability to package the artifact in an uber\-jar, including its dependencies and to *shade* \- i.e. rename \- the packages of some of the dependencies.


这段话简明扼要的概述了 maven\-shade\-plugin 的功能


1. 能够将项目连同其依赖，一并打包到一个 `uber-jar` 中


uber\-jar 就是一个超级 jar，不仅包含我们的工程代码，还包括依赖的 jar，和 `spring-boot-maven-plugin` 类似
2. 能够对依赖 jar 中的包名进行重命名


这个功能就有意思了，后面我们详说


maven\-shade\-plugin 必须和 Maven 构建生命周期的 package 阶段绑定，那么当 Maven 执行 `mvn package` 时会自动触发 maven\-shade\-plugin；使用很简单，在 `pom.xml` 添加该插件依赖即可



```
<plugin>
    <groupId>org.apache.maven.pluginsgroupId>
    <artifactId>maven-shade-pluginartifactId>
    <version>3.6.0version>
    <executions>
        <execution>
            
            <phase>packagephase>
            <goals>
                <goal>shadegoal>
            goals>

            <configuration>
                
            configuration>
        execution>
    executions>
plugin>

```

`phase` 和 `goal` 按如上固定配置，`configuration` 才是我们自由发挥的平台；有了基本了解后，我们再结合官方提供的 `Examples` 来看看 maven\-shade\-plugin 具体能干啥


## 选择打包内容


假设我们有项目 `maven-shade-plugin-demo`，其项目结构如下


![项目结构](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829231923764-520281583.png)
如果不做任何剔除，可以按如下配置进行全打包



```
<dependencies>
    <dependency>
        <groupId>cn.hutoolgroupId>
        <artifactId>hutool-allartifactId>
        <version>5.8.26version>
    dependency>
dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.pluginsgroupId>
            <artifactId>maven-shade-pluginartifactId>
            <version>3.6.0version>
            <executions>
                <execution>
                    
                    <phase>packagephase>
                    <goals>
                        <goal>shadegoal>
                    goals>
                    <configuration>
                        
                    configuration>
                execution>
            executions>
        plugin>
    plugins>
build>

```

执行 `mvn package` 后，我们会看到两个包


![全打包](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829231945907-669210017.png)
`maven-shade-plugin-demo-1.0-SNAPSHOT.jar` 就是 uber\-jar；解压可看其结构


![uber-jar结构](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232023636-1232700971.png)
不仅包括 `package`、还包括各种配置文件、元文件，统统打包进 uber\-jar；而 `original-maven-shade-plugin-demo-1.0-SNAPSHOT.jar` 则是不包括依赖 jar 的原始项目包；如果我们比较细心的话，会发现打包的时候告警了


![全打包告警](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232045043-1439522625.png)
意思是说 hutool jar 包中有 `META-INF/MANIFEST.MF`，而 `maven-shade-plugin-demo` 打包成 jar 后也包含 `META-INF/MANIFEST.MF`，两者重复了，只会将其中一个复制进 uber jar；默认情况下，是将我们项目的 jar 中的 `META-INF/MANIFEST.MF` 复制进 uber jar


![默认用项目的MANIFEST](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232106088-671033566.png)
那如果我们想保留 hutool 下的 MANIFEST.MF，而去掉 maven\-shade\-plugin\-demo 中的 MANIFEST.MF，该如何处理呢？只需要微调下 `configuration`



```
<configuration>
    <filters>
        <filter>
            <artifact>com.qsl:maven-shade-plugin-demoartifact>
            <excludes>
                <exclude>META-INF/*.MFexclude>
            excludes>
        filter>
    filters>
configuration>

```

此时 uber jar 中的 MANIFEST.MF 就来自 hutool jar 了


![换成hutool下的MANIFEST](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232123918-1971242697.png)
回到前面的 `configuration` 配置，我们需要明白其每个子标签的含义


1. `filter`：过滤器，可以配置多个
2. `artifact`：复合标识符，用来匹配 jar，简单点说，就是匹配 jar 的 `匹配规则`


按 Maven 的坐标：groupId:artifactId\[\[:type]:classifier] 进行配置，`groupId:artifactId` 必配，`[[:type]:classifier]` 选配；支持通配符 `*` 和 `?`，例如：`*:*`（相当于匹配上所有jar）
3. `exclude`：排除项，也就是不会复制进 uber\-jar；支持通配符配置
4. `include`：包含项，也就是**只有**这些会被复制进 uber\-jar；支持通配符配置


我们实战下，假设我们项目结构如下所示


![明细配置项目结构](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232142530-1142400784.png)
`configuration` 配置如下



```
<configuration>
    <filters>
        <filter>
            <artifact>com.qsl:maven-shade-plugin-demoartifact>
            <excludes>
                <exclude>com/qsl/test/**exclude>
                <exclude>com/qsl/Entry.classexclude>
            excludes>
        filter>
        <filter>
            <artifact>cn.hutool:hutool-allartifact>
            <includes>
                <include>cn/hutool/Hutool.classinclude>
                <include>cn/hutool/json/**include>
            includes>
        filter>
    filters>
configuration>

```

执行 `mvn package` 后，uber\-jar 内部结构你们能想到吗？我们来看看实际结果


![明细配置后uber-jar结构](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232202422-2119625187.png)
是不是和跟你们想的一样？


除了手动指定 `filter` 外，此插件还支持自动移除项目中没有使用到的依赖类，以此来最小化 uber jar 的体积；configuration 配置如下



```
<configuration>
    <minimizeJar>trueminimizeJar>
configuration>

```

我们在 `StringUtil` 中引入 hutool 的 StrUtil（相当于项目依赖了 StrUtil）



```
package com.qsl.util;

import cn.hutool.core.util.StrUtil;

/**
 * @author: 青石路
 */
public class StringUtil {

    public static boolean isBlank(String str) {
        return StrUtil.isBlank(str);
    }
}

```

然后打包，uber\-jar 内部结构如下所示


![最小依赖](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232224213-134425516.png)
从 maven\-shade\-plugin 1\.6 开始，`minimizeJar` 会保留 `filter` 中 `include` 配置的类，但是要注意：



> inlcude 默认会排除所有不在 include 配置中的类


这就会导致问题，我们来看个案例，我们引入 `logback` 依赖，但代码中未用到它，而我们又想将其下的 class 复制进 uber\-jar，另外我们还想将 hutool 的 `cn/hutool/json` 包下的全部类都复制进 uber\-jar，并且开启 `minimizeJar`，是不是按如下配置？



```
<dependencies>
    <dependency>
        <groupId>cn.hutoolgroupId>
        <artifactId>hutool-allartifactId>
        <version>5.8.26version>
    dependency>
    <dependency>
        <groupId>ch.qos.logbackgroupId>
        <artifactId>logback-classicartifactId>
        <version>1.3.14version>
    dependency>
dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.pluginsgroupId>
            <artifactId>maven-shade-pluginartifactId>
            <version>3.6.0version>
            <executions>
                <execution>
                    
                    <phase>packagephase>
                    <goals>
                        <goal>shadegoal>
                    goals>
                    <configuration>
                        <minimizeJar>trueminimizeJar>
                        <filters>
                            <filter>
                                <artifact>ch.qos.logback:logback-classicartifact>
                                <includes>
                                    <include>**include>
                                includes>
                            filter>
                            <filter>
                                <artifact>cn.hutool:hutool-allartifact>
                                <includes>
                                    <include>cn/hutool/json/**include>
                                includes>
                            filter>
                        filters>
                    configuration>
                execution>
            executions>
        plugin>
    plugins>
build>

```

打包后看 uber\-jar 目录结构


![最小依赖_include](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232246506-1360993955.png)
hutool 的 `core` 包没有复制进来，这是因为我们对 hutool 配置了 `include` ，默认把最小依赖的 `core` 包给排除掉了，那怎么办呢？插件提供了配置 `false` 来处理此种情况，它会覆盖 `include` 默认排除行为



```
<filter>
    <artifact>cn.hutool:hutool-allartifact>
    <excludeDefaults>falseexcludeDefaults>
    <includes>
        <include>cn/hutool/json/**include>
    includes>
filter>

```

这样配置之后，既能包含 hutool 的 json 包，又能包含最小依赖的 core 包



> false 通常配合 true 来使用，不然
> 
> 
> 
> ```
> <configuration>
>     <filters>
>         <filter>
>             <artifact>cn.hutool:hutool-allartifact>
>             <excludeDefaults>falseexcludeDefaults>
>             <includes>
>                 <include>cn/hutool/json/**include>
>             includes>
>         filter>
>     filters>
> configuration>
> 
> ```
> 
> 这么配置有何意义？


## 重定位 class


如果 uber\-jar 被其他项目依赖，而我们的 uber\-jar 又是保留了依赖 jar 的 class 的全类名，那么就可能类重复而导致类加载冲突；比如项目A依赖了我们的 `maven-shade-plugin-demo`，还依赖了 B.jar，两个 jar 中都存在 `cn.hutool.core.util.StrUtil.class`，但 api 完全不一样，根据 `双亲委派模型`，只会成功加载其中某个 `cn.hutool.core.util.StrUtil.class`，那么另一个的 api 则使用不了。为了解决这个问题，插件提供了重定位功能，通过创建 class 字节码的私有副本，按新配置的 package，打包进 uber\-jar


我们来看个案例，假设我们只需要 hutool 的 core 包，将其下所有的 class 按 `com.qsl.core` 包打包进 uber\-jar，可以按如下配置



```
<dependencies>
    <dependency>
        <groupId>cn.hutoolgroupId>
        <artifactId>hutool-allartifactId>
        <version>5.8.26version>
    dependency>
dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.pluginsgroupId>
            <artifactId>maven-shade-pluginartifactId>
            <version>3.6.0version>
            <executions>
                <execution>
                    
                    <phase>packagephase>
                    <goals>
                        <goal>shadegoal>
                    goals>
                    <configuration>
                        <relocations>
                            <relocation>
                                <pattern>cn.hutool.corepattern>
                                <shadedPattern>com.qsl.coreshadedPattern>
                            relocation>
                        relocations>
                        <filters>
                            <filter>
                                <artifact>cn.hutool:hutool-allartifact>
                                <includes>
                                    <include>cn/hutool/core/**include>
                                includes>
                            filter>
                        filters>
                    configuration>
                execution>
            executions>
        plugin>
    plugins>
build>

```

打包后 uber\-jar 目录结构如下


![relocate](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232422472-685216006.png)
我们来看下 uber\-jar 中的 StringUtil.class


![StringUtil中的StrUtil路径](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232440389-251899.png)
依赖的 `StrUtil` 也被正确调整了，是不是很牛皮？


![有点东西](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/96accfdd9c23489b82449a1c626d778e~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6Z2S55-z6Lev:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMTk5OTM1NzAyMTAwNTU4MCJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1725550305&x-orig-sign=aucYeF5SDJb1MGj0vPnUYskVNmU%3D)
此时项目A 依赖 B.jar 的同时，又依赖我们的 `maven-shade-plugin-demo`，就不会类重名了（package 不一致了）


`relocation` 同样支持 `exclude` 和 `include`



```
<configuration>
    <relocations>
        <relocation>
            <pattern>cn.hutool.corepattern>
            <shadedPattern>com.qsl.coreshadedPattern>
            
            <excludes>
                <exclude>cn.hutool.core.util.ObjUtilexclude>
                
                <exclude>cn.hutool.core.date.**exclude>
            excludes>
        relocation>
        <relocation>
            <pattern>cn.hutool.jsonpattern>
            <shadedPattern>com.qsl.jsonshadedPattern>
            
            <includes>
                <include>cn.hutool.json.JSONUtilinclude>
                
                <include>cn.hutool.json.xml.**include>
            includes>
        relocation>
    relocations>
    <filters>
        <filter>
            <artifact>cn.hutool:hutool-allartifact>
            <includes>
                <include>cn/hutool/core/**include>
                <include>cn/hutool/json/**include>
            includes>
        filter>
    filters>
configuration>

```

此时 uber\-jar 的目录结构是怎样的？你们自己去试！


## 生成附属包


前面已经介绍过，打包后会生成两个包


![全打包](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829231945907-669210017.png)
但 `original` 开头的那个明显不是按 Maven 坐标命名的，所以它是不能够 `install` 到本地或者远程仓库的；如果需要将两个 jar 都 `install` 到仓库中，那么就需要用到插件的 `Attaching the Shaded Artifact` （生成附属包）功能



```
<configuration>
    <relocations>
        <relocation>
            <pattern>cn.hutool.corepattern>
            <shadedPattern>com.qsl.coreshadedPattern>
            
            <excludes>
                <exclude>cn.hutool.core.util.ObjUtilexclude>
                
                <exclude>cn.hutool.core.date.**exclude>
            excludes>
        relocation>
        <relocation>
            <pattern>cn.hutool.jsonpattern>
            <shadedPattern>com.qsl.jsonshadedPattern>
            
            <includes>
                <include>cn.hutool.json.JSONUtilinclude>
                
                <include>cn.hutool.json.xml.**include>
            includes>
        relocation>
    relocations>
    <filters>
        <filter>
            <artifact>cn.hutool:hutool-allartifact>
            <includes>
                <include>cn/hutool/core/**include>
                <include>cn/hutool/json/**include>
            includes>
        filter>
    filters>
    <shadedArtifactAttached>trueshadedArtifactAttached>
    <shadedClassifierName>qslshadedClassifierName>
configuration>

```

部署到仓库的 jar 如下


![生成附属包](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232554026-675932245.png)
## 可执行 JAR


这个就比较简单了，我们直接看配置



```
<configuration>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.qsl.EntrymainClass>
        transformer>
    transformers>
configuration>

```

如上配置会将 `Main-Class` 写进 uber\-jar 的 MANIFEST.MF，还可以通过 `manifestEntries` 自定义属性



```
<configuration>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <manifestEntries>
                <mainClass>com.qsl.EntrymainClass>
                <Build-Author>qslBuild-Author>
            manifestEntries>
        transformer>
    transformers>
configuration>

```

打包之后，uber\-jar 的 MANIFEST.MF 内容如下


![可执行jar](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232616314-1098086027.png)
## 资源转换器


[Resource Transformers](https://github.com) 已经介绍的很详细了，我就不一一介绍了，挑几个个人认为比较重要的简单讲一下


### ServicesResourceTransformer


合并  `META-INF/services/` 下的文件，并对文件中的 class 进行重定向；我们来看个例子，hutool 下有文件 `cn.hutool.aop.proxy.ProxyFactory`


![services_proxyFactory](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232639964-1857674738.png)
我们也自定义一个


![自定义QslFactory](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232655351-520555118.png)
configuration 配置如下



```
<configuration>
    <relocations>
        <relocation>
            <pattern>cn.hutool.aoppattern>
            <shadedPattern>com.qsl.aopshadedPattern>
        relocation>
    relocations>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
    transformers>
configuration>

```

打包后，hutool 与 uber\-jar 的 cn.hutool.aop.proxy.ProxyFactory 文件内容差异如下


![services_合并后前后对比](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232712495-657100479.png)
如果不配置 `ServicesResourceTransformer`，结果是怎样，你们自己去试


### AppendingTransformer


将多个同名文件的内容合并追加到一起（不配置的情况下会覆盖，最终文件内容只是其中某个文件的内容），configuration 配置如下



```
<configuration>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
            <resource>META-INF/spring.factoriesresource>
        transformer>
    transformers>
configuration>

```

打包后文件内容合并如下


![Append转换器](https://img2024.cnblogs.com/blog/747662/202408/747662-20240829232734731-389970735.png)
`XmlAppendingTransformer`、`ResourceBundleAppendingTransformer` 功能类似，只是针对的文件内容格式略微有点特殊，就不演示了，你们自行去测试


## 同包同名 class 共存


回到我们的主题，如果我们项目依赖的 jar 中出现了同名的 class （包名和类名均相同），根据 `双亲委派模型`，只会加载其中某一个 class，虽然两个 class 同名了，但功能完全不一样，另一个未被加载的 class 的功能则用不了，如果想同时使用这两个同名 class 的功能，我们该如何处理？



> [甲方扔给两个存在包名与类名均相同的Jar包，要在工程中同时使用怎么办？](https://github.com):[蓝猫机场](https://fenfang.org)


文中给出了几种解决方案（注意看评论区），最高效最实用的当属 `maven-shade-plugin`；假设我们项目依赖的 A.jar 和 B.jar 都存在 `com.qsl.Hello.class`，我们可以新建一个项目，名字叫 `qsl-a`，没有任何代码，仅仅依赖 A.jar，然后利用 maven\-shade\-plugin 的 `Relocating Classes` 功能对 A.jar 中存在重名的 class 进行重定向，例如



```
<configuration>
    <relocations>
        <relocation>
            <pattern>com.qslpattern>
            <shadedPattern>com.qslashadedPattern>
        relocation>
        <includes>
            <include>com.qsl.Helloinclude>
        includes>
    relocations>
configuration>

```

然后打包得到 uber jar（qsl\-a.jar），项目依赖从 A.jar 更改成 qsl\-a.jar，B.jar 依赖继续保留，那么项目中可用的 Hello.class 就包括



> com.qsl.Hello（B.jar）
> 
> 
> com.qsla.Hello（qsl\-a.jar）


问题是不是就得到解决了？更实际的案例，敬请期待我下篇博客


## 总结


1. maven\-shade\-plugin 的输入目标是 `项目原始jar` 以及 `项目依赖的所有jar`，而输出目标是 `uber-jar`，所以 maven\-shade\-plugin 的规则对 `项目原始jar` 是无效的
2. `minimizeJar` 针对的只是 `class`，其他类型的文件不受此约束
3. 同 class 共存问题，可以利用 maven\-shade\-plugin 的 Relocating Classes 功能，将其中一个或多个 jar 重新打包成新的 jar，保证类名相同但包名不同，然后项目依赖新的 jar，变相解决了同 class 共存问题
4. 示例项目：[maven\-shade\-plugin\-demo](https://github.com)


