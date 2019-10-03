### mybatis-generator介绍
Mybatis-Generator可以帮我们去生成 Model、Dao、Mapper 和映射的 sql，节省不少时间
### 1. 导入mybatis-generator插件
```
<project ...>
     ...
     <build>
       ...
       <plugins>
        ...
        <plugin>
          <groupId>org.mybatis.generator</groupId>
          <artifactId>mybatis-generator-maven-plugin</artifactId>
          <version>1.3.7</version>
          <executions>
            <execution>
              <id>Generate MyBatis Artifacts</id>
              <goals>
                <goal>generate</goal>
              </goals>
            </execution>
          </executions>
          <dependencies>
            <dependency>
              <groupId>org.hsqldb</groupId>
              <artifactId>hsqldb</artifactId>
              <version>2.3.4</version>
            </dependency>
          </dependencies>
        </plugin>
        ...
      </plugins>
      ...
    </build>
    ...
  </project>
```
### 2. 配置generatorConfig.xml文件
在src/main/resources下创建generatorConfig.xml文件，内容如下，具体数据库连接和包路径自行修改
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<!-- 配置生成器 -->
<generatorConfiguration>

    <!--id:必选，上下文id，用于在生成错误时提示-->
    <context id="mysql" targetRuntime="MyBatis3">

        <!-- 生成的Java文件的编码 -->
        <property name="javaFileEncoding" value="UTF-8"/>

        <!-- 对注释进行控制 -->
        <commentGenerator>
            <!-- suppressDate是去掉生成日期那行注释 -->
            <property name="suppressDate" value="true"/>
            <!-- suppressAllComments是去掉所有的注解 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--jdbc的数据库连接 -->
        <jdbcConnection
                driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://127.0.0.1:3306/test?useUnicode=true"
                userId="root"
                password="admin">
        </jdbcConnection>

        <!-- java类型处理器
        用于处理DB中的类型到Java中的类型，默认使用JavaTypeResolverDefaultImpl；
        注意一点，默认会先尝试使用Integer，Long，Short等来对应DECIMAL和 NUMERIC数据类型；
        -->
        <javaTypeResolver type="org.mybatis.generator.internal.types.JavaTypeResolverDefaultImpl">
            <!--
                true：使用BigDecimal对应DECIMAL和 NUMERIC数据类型
                false：默认,
                    scale>0;length>18：使用BigDecimal;
                    scale=0;length[10,18]：使用Long；
                    scale=0;length[5,9]：使用Integer；
                    scale=0;length<5：使用Short；
             -->
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>


        <!-- java模型创建器，是必须要的元素
            负责：1，key类（见context的defaultModelType）；2，java类；3，查询类
            targetPackage：生成的类要放的包，真实的包受enableSubPackages属性控制；
            targetProject：目标项目，指定一个存在的目录下，生成的内容会放到指定目录中，如果目录不存在，MBG不会自动建目录
         -->
        <javaModelGenerator targetPackage="cn.utry.bo" targetProject="src/main/java">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="true"/>
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>

        <!-- 生成SQL map的XML文件生成器，
            注意，在Mybatis3之后，我们可以使用mapper.xml文件+Mapper接口（或者不用mapper接口），
                或者只使用Mapper接口+Annotation，
                所以，如果 javaClientGenerator配置中配置了需要生成XML的话，这个元素就必须配置
            targetPackage/targetProject:同javaModelGenerator
         -->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>


        <!-- 对于mybatis来说，即生成Mapper接口，注意，如果没有配置该元素，那么默认不会生成Mapper接口
            targetPackage/targetProject:同javaModelGenerator
            type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）：
                1，ANNOTATEDMAPPER：会生成使用Mapper接口+Annotation的方式创建（SQL生成在annotation中），不会生成对应的XML；
                2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中；
                3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
            注意，如果context是MyBatis3Simple：只支持ANNOTATEDMAPPER和XMLMAPPER
        -->
        <javaClientGenerator targetPackage="cn.utry.dao" type="XMLMAPPER" targetProject="src/main/java">
            <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        <!-- 选择一个table来生成相关文件，可以有一个或多个table，必须要有table元素
            tableName（必要）：要生成对象的表名；
            domainObjectName 给表对应的 model 起名字
            注意：大小写敏感问题。
         -->
        <table schema="general" tableName="t_student" domainObjectName="Student"
               enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false"
               enableSelectByExample="false" selectByExampleQueryId="false"/>

        <!--<table schema="general" tableName="user_info" domainObjectName="UserInfo"
               enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false"
               enableSelectByExample="false" selectByExampleQueryId="false"/>-->
        <!--用来修改表中某个列的属性,一个table元素中可以有多个columnOverride元素哈.
            property属性来指定列要生成的属性名称.
         -->
        <!--<columnOverride column="username" property="userName" />-->


        <!--<table tableName="person" domainObjectName="Person"/>-->
        <!--<table tableName="department" domainObjectName="Depart"/>-->

    </context>

</generatorConfiguration>
```
``注意：`` 加主键，不然只能生成insert语句
### 3. 运行
``1：``  mvn mybatis-generator:generate

``2：``  mvn -Dmybatis.generator.overwrite=true mybatis-generator:generate

``3：``  点击Edit Configurations，点击弹出窗口左上角的+号，新增Maven.为当前配置配置一个名称，这里命名为"generator",然后在 “Command line” 选项中输入“mybatis-generator:generate -e”，点击Apply
在项目右侧导航栏中选择Run Configurations下的generator即可
###git地址
https://github.com/shizhenshuang123/mvn-Mybatis-Generator-Demo
###gradle版本
https://blog.csdn.net/inke88/article/details/74766432
