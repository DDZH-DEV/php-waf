此代码是个人根据宝塔的防火墙规则写的php过滤版本，平常都是在nginx端做过滤规则，但有时会遇到一些买虚拟主机的客户，无法进行过滤，于是就有了这个东西。 

规则方面与宝塔应用市场中的  Nginx免费防火墙  互相兼容，可以互相替换，宝塔中的默认目录为
```/www/server/free_waf/rule```

从原理上来讲，你永远也无法知道你的代码存在什么样的漏洞，所以规则能替你抵挡大部分注入，但不是全部，从性能上来讲，大量的正则表达式会有一点性能损耗，能上服务器的尽量在nginx层做过滤 


每个项目的逻辑处理都不一样,不会写全部的逻辑,只贴关键性代码
```php
<?php
use rax\RaxWaf;  
//自己初始化

$config=[
        //开启防火墙
        'enable'=>true,
        //开启记录
        'log'=>true, 
        //拦截消息
        'deny_message'=>'php waf hit ! ',
        //注入多少次后屏蔽，每种程序的业务逻辑不一样，需要自己根据deny_num去写
        'deny_num'=>3,
    ]; 

//RaxWaf::check($ip,$url,$data)
//在一些socket应用中url可能是固定的，不需要检测,可以是 RaxWaf::check(当前请求的ip,要检测的数据) 
//返回当前请求是否需要拦截
$deny=RaxWaf::check($ip,$_SERVER['REQUEST_URI'],['GET'=>$_GET,'POST'=>$_POST,'COOKIE'=>$_COOKIE]);
``` 

当然也可以这样
```php
RaxWaf::$handle=function($ip){
	//默认只做了日志记录,在这里可以自定义拦截逻辑,比如记录ip,退出代码等
};
```

其他代码参考,比如在workerman中使用
https://github.com/9raxdev/think-workerman/blob/master/core/WebServer4.php#L32

>写在最后: 不同的程序可能存在的漏洞不相同，出现拦截不到的，应当立即去分析日志，根据日志去做拦截规则。

>如果有搞不定的可以加入群进行交流，当然规则方面也会在群里面优先分享 点击链接加入群聊【宝塔防火墙webshell查杀】：https://jq.qq.com/?_wv=1027&k=q77siQGT
