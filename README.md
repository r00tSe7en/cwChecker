# cwChecker

cdn & waf过滤

[+] 修改自 https://github.com/alwaystest18/cdnChecker （修改判断逻辑，添加判断条件）

## 背景

现有的一些识别cdn的工具存在如下问题：

- *仅根据cname或ip范围判断cdn，cname与ip范围不全导致遗漏*

- *输出字段较多，不方便直接与其他工具结合*

## 安装

```
.........
```

## 使用

速度测试：实际测试13361个域名耗时82s

参数说明

```
Usage of ./cdnChecker:
  -cf string         //cdn cname文件，默认为同目录cdn_cname
        cdn cname file (default "cdn_cname")
  -df string         //域名列表文件，注意为host部分，不要带http://
        domain list file
  -o string         //未使用cdn域名输出文件，如果不指定生成在同目录no_cdn_domains+时间.txt
        output domains that are not using cdn to file (default "no_cdn_domains202304040755.txt")
  -oc string     //使用cdn域名输出文件，如果不指定生成在同目录use_cdn_domains+时间.txt
        output domains that are using cdn to file (default "use_cdn_domains202304040755.txt")
  -od string     //域名:ip的格式，便于根据ip反查域名，如果不指定生成在同目录domain_info+时间.txt
        output domain info(domain:ip) to file (default "domain_info202304122222.txt")
  -oi string    //未使用cdn的ip输出文件，如果不指定生成在同目录no_cdn_ips+时间.txt
        output ips that are not using cdn to file (default "no_cdn_ips202304040755.txt")
  -r string       //dns服务器列表文件
        dns resolvers file
```

使用

```
$ cat domains.txt 
www.baidu.com        //使用cdn
www.qq.com         //使用cdn
www.alibabagroup.com    //使用cdn
aurora.tencent.com     //未使用cdn

$ ./cwChecker -df domains.txt -cf cdn_cname -r resolvers.txt -oc use_cdn_subdomains.txt -od subdomains_2_ips.txt -oi no_cdn_ip.txt -o no_cdn_domains.txt

```


**强烈推荐dns服务器列表使用自带的resolvers.txt（均为国内dns服务器且验证可用），如果服务器数量过少，大量的dns查询会导致timeout，影响查询准确度**

## 识别cdn思路

主要通过多个dns服务器节点获取域名解析ip，如果存在4个以上不同的ip段，则判断使用cdn，反之未使用cdn。

但是直接通过dns服务器查询会增加网络开销影响速度，因此先通过以下方法完成初步筛选：

1.通过 "github.com/projectdiscovery/dnsx" 自带的checkCdn方法

```
通过ip范围判断，主要为国外cdn厂商，对国内cdn识别效果不理想
此方法为调用nslookup获取ip，实测不如（2）的方式准确
```

2.**[+]增加**通过"github.com/projectdiscovery/cdncheck" 自带的CheckCDN、CheckWAF方法
```
再次通过ip范围判断，主要为国外cdn&waf厂商
```

3.**[+]增加**通过获取域名解析的ip数
```
大于4个ip则认为域名使用了cdn
```

4.存在A记录但不存在cname的域名直接判断未使用cdn
```
此方法会有些疏漏
```

5.存在cname的与cdn name列表对比，如果包含cdn cname列表则判断使用cdn



## 常见问题

- 结果中使用cdn域名列表与未使用cdn域名列表数量相加与实际测试域名数量不符？

​       答：对于无法获取解析ip的域名，程序会默认为域名无效过滤掉

- 判定站点使用了cdn，不在cdn的ip范围，不在cdn的cname范围，同时domaininfo文件中对应的也是一个ip

​       答：判定站点是否使用cdn时是参考多个dns服务器解析结果，而写入domaininfo文件获取的ip地址为单个dns服务器解析结果，个别域名会存在多服务器解析多个ip而单服务器仅解析一个ip的情况



## 感谢

https://github.com/xiaoyiios/chinacdndomianlist

https://github.com/u9sky/cdn-cname-domain

https://github.com/alwaystest18/cdnChecker
