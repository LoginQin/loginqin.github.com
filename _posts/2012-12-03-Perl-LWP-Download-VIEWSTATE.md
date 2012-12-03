---
layout: post-md
title: PERL LWP模块下载碰到ASPX , __VIEWSTATE,及注意事项
---
今天在下载环球会展网的数据, 页面是ASPX的, 并且用到了网页保存数据, 即__VIEWSTATE状态BASE+64编码解码.
APSX的网页用_viewstate保存状态很恶心, 整个网页就会很大, 先不讨论这个.
记录一下我的下载过程:
##下载步骤
1. 页数的参数是
   
    Page$3

2. 我用URL::Escape转义了, **错就错在这里, 因为UserAgent会再将%转义一次, 实际上不用转了!!!**
  
    uri_escape("Page$3"); # Page%243

3. 由于是ASPX页面, 并且用__viewstate保存页面信息, 加上cookie,都一定要全部传过服务器, 关于COOKIE可以这样设置UserAgent
        sub new_userAgent {
        my $ua = LWP::UserAgent->new(keep_alive=>1);
        $ua->cookie_jar(HTTP::Cookies->new('file'=>'cookie.lwp','autosave'=>1));
        $ua->timeout(100);
        #$ua->env_proxy();
        $ua->agent('Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.12 (KHTML, like Gecko) Chrome/9.0.576.0 Safari/534.12');
        return $ua; 
        }

4. 用正则匹配到以下__EVENTVALIDATION, __VIEWSTATEENCRYPTED, __VIEWSTATE, 构造参数传值
       my $parm = {
       '__EVENTTARGET' => 'GridView1'
       , '__EVENTARGUMENT' => $page
       ,'__VIEWSTATE' => $state
       , '__VIEWSTATEENCRYPTED' => ''
       ,'__EVENTVALIDATION'=> $dation 
       };
5. POST到服务器

    my $res2 = $ua->post($url, $parm, 'Content-Type'=>'application/x-www-form-urlencoded');

*如果不放心可以加上Content-type, 保证和原来的FORM表单一致, 实际上不用*

##常见错误:

>“/”应用程序中的服务器错误。
>
> 回发或回调参数无效。在配置中使用 <pages enableEventValidation="true"/> 或在页面中使用 <%@ Page EnableEventValidation="true" %> 启用了事件验证。出于安全目的，此功能验证回发或回调事件的参数是否来源于最初呈现这些事件的服务器控件。如果数据有效并且是预期的，则使用 ClientScriptManager.RegisterForEventValidation 方法来注册回发或回调数据以进行验证。

原 因: __viewstate等参数传递成功了! 但是其他参数错误, 我的是Page%243错误, 应该是Page$3, 在PERL下是 Page\$3, **注意不需要escape, LWP会帮转的**

> 此页的状态信息无效，可能已损坏。

>   [FormatException: Base-64 字符串中的无效字符。]
    System.Convert.FromBase64String(String s) +0
    System.Web.UI.ObjectStateFormatter.Deserialize(String inputString) +77
    System.Web.UI.ObjectStateFormatter.System.Web.UI.IStateFormatter.Deserialize(String serializedState) +4
    System.Web.UI.Util.DeserializeWithAssert(IStateFormatter formatter, String serializedState) +37
    System.Web.UI.HiddenFieldPageStatePersister.Load() +113
>  [ViewStateException: 无效的视图状态。
	Client IP: 171.37.35.xx
	Port: 39848
	User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.12 (KHTML, like Gecko) Chrome/9.0.576.0 Safari/534.12

	ViewState: AFwUmwTsP8psmjWC2Bycupw4Xn7ix6Kha6vtBfQB2pxH6+38+9A3KPYPmXlnl8jVeFCEh4WWNBurX8SBtssFRAnTfPMwQwrNCIZ37Pig60G/1Sz5e50UqnpwXdz4CDGVKfp4yKkN5rcnk6lpWaTpaCgpWSUjiN8e8Wa+ScNyQwMGaNXp5KHdJ0n/ofsc+53SO0lm9DGA2kcO4hPczO6kBzEsq/e1nVfx1T4kYcjeqbxiVxhhmEQCG50drXIlPC+7JRDpSVtoSHJro5SRT2Z80Ek9NRPXncOv8Km5YhHEK4sgQAYf3j0D2Vh7EWP+UzsulfgCnlEUmXIh5QtJITTL4Yn26Uf3s6po8WK+oUwqTyUg+MJJab7s3Lz79Zl78Gw4H310q4knWt9xj5nAJ+mwqvDi3qZ5tFlLmL35ombAWHZ5Mkbi3TFWZGnTUoeeVLikHsQ6Rjy5ZcnUaUFR7dBwHUDIJoPMR3MP6Eg1wBD5OZSPPG9hGWQbQcON2VNjsKXkCvKAOmzreLbTX2OabYTyUsVawoLfFYdDNZlyGrjvWSuLZGFM1w5O3CGypF3ZjTA0HRxnOt8GIivwb/oyH6aLWktsTK7hYizQFqr7UafYbcyPmgQ4XYQc+tD3EAHS1Qg4t264bvWLvyzo3Z/10dYzeTkaMcdG0CeSKbCcELiJC+7ihxL6SDlrJGHmC8+oM4OEU9kUE0hCNGYnxhxBH0aOcQnRFJyJudbO0I4LoeXWBwowbZoY46aw5lZG6hq18O+J/p8QL24b+mFFL/KPHzv+yt4yStdP20it59y9aaFh3W1Iw1fvBwkRhkl/9DaYK2aqRV3...]

原 因: \___EVENTVALIDATION, \___VIEWSTATEENCRYPTED, \__VIEWSTATE , 三个参数可能某个或多个错误, 匹配出来的数据, 请不要escape任何参数 

##有用的工具
   1. Firefox + FireBug   无需多言
   2. Firefox + live HTTP headers  => 不仅可以查看GET/POST数据, 还可以构造请求, 重新提交参数
   3. WireShark => 抓包工具, 由于Perl提交的参数我没找到很好的办法查看, 只好抓包看请求, 这才发现我的escape多余了, 而PERL最后的提交又转了一次

   *哈, 没想到逼我出绝招, 使用WireShark终极武器来研究*

##Perl下载总结
   1. Perl的LWP, 不需要显示的去帮escape()

   2. 下载的网页根据charset=编码, 用$res->decoded_content, 为utf8

   3. 不能直接操作$res->decoded\_content进行替换等, 必须转存再替换, 例如下面的是错误的,

     $res->decoded_content ~= s/charset=gb2312/charset=utf8/;

   4. 直接将$res->decoded\_content写入文件, 不管你open ">:utf8",还是">:gbk", 统一utf8存
    
   5. 如果想保存成GBK, 可以这样:
    
    my $data = $res->decoded_content;  
    open FILE, ">html.html";  
    print FILE encode('gbk', $data);  
    close(FILE);  
 
