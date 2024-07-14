# `Http`域名加载

在xml文件夹中加入’`network_security_config.xml`文件，其中内容为

```xml
<?xml version="1.0" encoding="utf-8"?>  
<network-security-config xmlns:tools="http://schemas.android.com/tools">
<network-security-config>  
    <base-config cleartextTrafficPermitted="true">  
        <domain-config cleartextTrafficPermitted="true">  
        <domain includeSubdomains="true">p1.music.126.net</domain>  
    </domain-config>  
    </base-config>  
</network-security-config>
```

再在注册表中加入

```xml
android:networkSecurityConfig="@xml/network_security_config"
```

记得在’`network_security_config.xml`中加的是域名，不是文件名。比如`p1.music.126.net/WxDbgJRBVWj1p9kdDqTFbg==/109951169746442412.jpg`

是文件名。`p1.music.126.net`才是域名。