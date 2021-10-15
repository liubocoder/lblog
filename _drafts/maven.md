#maven构建工具的使用

##org.apache.maven.plugins插件的使用

org.apache.maven.plugins是一系列的插件

常用的插件及示例配置代码

1. maven-compiler-plugin 基础编译运行插件
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>1.8</source> <!-- 源代码使用的JDK版本 -->
        <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->
        <encoding>UTF-8</encoding><!-- 字符集编码 -->
        <skipTests>true</skipTests><!-- 跳过测试 -->
    </configuration>
</plugin>
```

2. maven-jar-plugin
```
<!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <classesDirectory>target/classes/</classesDirectory>
        <archive>
            <addMavenDescriptor>false</addMavenDescriptor>
            <manifest>
                <mainClass>[自己的mainclass，没有则不添加这项]</mainClass>
                <!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
                <useUniqueVersions>false</useUniqueVersions>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
            </manifest>
            <manifestEntries>
                <Class-Path>.</Class-Path>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>

```

3. maven-assembly-plugin 自定义打包
```
<!-- 打包外部依赖 -->
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <!-- 指定assembly插件
        <descriptors>
            <descriptor>src/main/resources/assembly/package.xml</descriptor>
        </descriptors>
        -->
        <archive>
            <manifest>
                <mainClass>[main class]</mainClass>
            </manifest>
            <manifestEntries>
                <Class-Path>.</Class-Path>
            </manifestEntries>
        </archive>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>

```
```
<!-- assembly插件示例 -->
<?xml version="1.0" encoding="UTF-8"?>
<assembly>
    <id>full</id>
    <!-- 最终打包成一个用于发布的zip文件 -->
    <formats>
        <format>zip</format>
    </formats>

    <!-- 把依赖jar包打包进Zip压缩文件的lib目录下 -->
    <dependencySets>
        <dependencySet>
            <!--不使用项目的artifact，第三方jar不要解压，打包进zip文件的lib目录-->
            <useProjectArtifact>false</useProjectArtifact>

            <!-- 第三方jar打包进Zip文件的lib目录下， -->
            <!-- 注意此目录要与maven-jar-plugin中classpathPrefix指定的目录相同, -->
            <!-- 不然这些依赖的jar包加载到ClassPath的时候会找不到-->
            <outputDirectory>lib</outputDirectory>

            <!-- 第三方jar不要解压-->
            <!--<unpack>false</unpack>-->
        </dependencySet>
    </dependencySets>

    <!-- 文件设置，你想把哪些文件包含进去，或者把某些文件排除掉，都是在这里配置-->
    <fileSets>
        <!-- 把项目自己编译出来的可执行jar，打包进zip文件的根目录 -->
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory></outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>

        <!--
        把项目readme说明文档，打包进zip文件根目录下
        (这里针对目录document/readme.txt文件)
        ${projet.document.directory}是pom.xml中自己配置的
         -->
        <!--<fileSet>
            <directoryl>${projet.document.directory}</directoryl>
            <outputDirectory></outputDirectory>
            <includes>
                <include>readme.*</include>
            </includes>
        </fileSet>-->

        <!--
        把项目相关的说明文档(除了readme文档)，
        打包进zip文件根目录下的document目录
        (这里针对document/exclode.txt文件)
        ${project.document.directory}是在pom.xml中自己配置的
        -->
        <fileSet>
            <directory>${project.document.directory}</directory>
            <outputDirectory>document</outputDirectory>
            <excludes>
                <exclude>readme.*</exclude>
            </excludes>
        </fileSet>

        <!--
        把项目的脚本文件目录(src/main/scripts )中的启动脚本文件，
        打包进zip文件的根目录
        (这里针对的是src/scripts/execute/include-file.sh文件)
        ${project.script.execute.directory}
        -->
        <fileSet>
            <directory>${project.script.execute.directory}</directory>
            <outputDirectory></outputDirectory>
            <includes>
                <include>*</include>
            </includes>
        </fileSet>

    </fileSets>
</assembly>
```

##参考资料

[pom.xml文件结构](https://blog.csdn.net/weixin_38569499/article/details/91456988)

[maven常用插件](https://www.cnblogs.com/april-chen/p/10414857.html)

[Apache maven插件](https://maven.apache.org/plugins/index.html)



