- 导入tomcat 8.5源码过程
1. 创建pom.xml
2. 创建catalina-home文件,并将conf与webapps目录复制进来
3. 配置启动参数
```
Main class设置为org.apache.catalina.startup.Bootstrap

添加VM options

-Dcatalina.home=/Users/xiaoxiaoxiang/IdeaProjects/apache-tomcat-8.5.57-src/catalina-home
-Dcatalina.base=/Users/xiaoxiaoxiang/IdeaProjects/apache-tomcat-8.5.57-src/catalina-home
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Djava.util.logging.config.file=/Users/xiaoxiaoxiang/IdeaProjects/apache-tomcat-8.5.57-src/catalina-home/conf/logging.properties

```
4. TestCookieFilter会报错,注释掉
5. 启动成功,访问`http://localhost:8080/`报错,原因:  
启动`org.apache.catalina.startup.Bootstrap`时候没有加载`org.apache.jasper.servlet.JasperInitializer`,
从而无法编译JSP.
解决办法是在tomcat的源码`org.apache.catalina.startup.ContextConfig#configureStart()`中手动将JSP解析器初始化:
```
context.addServletContainerInitializer(new JasperInitializer(), null);
```
6. 发现控制台日志里有形如 `è³å°æä¸ä¸ªJARè¢«`的乱码,-Dfile.encoding=UTF8或者GBK都不行,如下修改:
```
- org.apache.tomcat.util.res.StringManager
    public static final StringManager getManager(String packageName) {
        // 由于本机会出现形如： æå¡å¨æå»º的乱码,修改编码
//        return getManager(packageName, Locale.getDefault());
        return getManager(packageName, Locale.ENGLISH);
    }

- org.apache.jasper.compiler.Localizer

    public static String getMessage(String errCode) {
        String errMsg = errCode;
        try {
            if (bundle != null) {
                errMsg = bundle.getString(errCode);
                errMsg = new String(errMsg.getBytes("ISO-8859-1"), "UTF-8");
            }
        } catch (MissingResourceException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return errMsg;
    }

```
7. 访问http://localhost:8080/docs/会404,并且报错也有乱码