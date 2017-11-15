ZDNS
====

[![Build Status](https://travis-ci.org/zmap/zdns.svg?branch=master)](https://travis-ci.org/zmap/zdns)
[![Go Report Card](https://goreportcard.com/badge/github.com/zmap/zdns)](https://goreportcard.com/report/github.com/zmap/zdns)

ZDNS is a command-line utility that provides high-speed DNS lookups. For
example, the following will perform MX lookups and a secondary A lookup for the
IPs of MX servers for the domains in the Alexa Top Million:

	cat top1m-alexa.csv  | zdns alookup  --ipv4-lookup --alexa

ZDNS is written in golang and is primarily based on https://github.com/miekg/dns.

Install
=======

ZDNS can be installed by running:

	go get github.com/zmap/zdns/zdns


Usage
=====

ZDNS provides several types of modules.

Raw DNS Modules
---------------

The `A`, `AAAA`, `ANY`, `AXFR`, `CAA`, `CNAME`, `DMARC`, `MX`, `NS`, `PTR`, `TXT`,
`SOA`, and `SPF` modules provide the raw DNS response in JSON form, similar to dig.

For example, the command:
----------
       echo "2,baidu.com" | ./zdns alookup --ipv4-lookup --alexa
returns:
```json

{
	"name": "baidu.com",
	"class": "IN",
	"alexa_rank": 2,
	"status": "NOERROR",
	"timestamp": "2017-11-15T02:10:31-05:00",
	"data": {
		"ipv4_addresses": ["123.125.114.144","220.181.57.217","111.13.101.208"]
	}
}
```
----------

	echo "censys.io" | zdns A

returns:
```json
{
  "name": "censys.io",
  "class": "IN",
  "status": "NOERROR",
  "data": {
    "answers": [
      {
        "ttl": 300,
        "type": "A",
        "class": "IN",
        "name": "censys.io",
        "data": "216.239.38.21"
      }
    ],
    "additionals": [
      {
        "ttl": 34563,
        "type": "A",
        "class": "IN",
        "name": "ns-cloud-e1.googledomains.com",
        "data": "216.239.32.110"
      },
    ],
    "authorities": [
      {
        "ttl": 53110,
        "type": "NS",
        "class": "IN",
        "name": "censys.io",
        "data": "ns-cloud-e1.googledomains.com."
      },
    ],
    "protocol": "udp"
  }
}
```

Lookup Modules
--------------

Raw DNS responses frequently do not provide the data you _want_. For example,
an MX response may not include the associated A records in the additionals
section requiring an additional lookup. To address this gap and provide a
friendlier interface, we also provide several _lookup_ modules: `alookup` and
`mxlookup`.

`mxlookup` will additionally do an A lookup for the IP addresses that
correspond with an exchange record. `alookup` acts similar to nslookup and will
follow CNAME records.

For example,

	echo "censys.io" | ./zdns mxlookup --ipv4-lookup

returns:
```json
{
  "name": "censys.io",
  "status": "NOERROR",
  "data": {
    "exchanges": [
      {
        "name": "aspmx.l.google.com",
        "type": "MX",
        "class": "IN",
        "preference": 1,
        "ipv4_addresses": [
          "74.125.28.26"
        ],
        "ttl": 288
      },
      {
        "name": "alt1.aspmx.l.google.com",
        "type": "MX",
        "class": "IN",
        "preference": 5,
        "ipv4_addresses": [
          "64.233.182.26"
        ],
        "ttl": 288
      }
    ]
  }
}
```

Local Recursion
---------------

ZDNS can either operate against a recursive resolver (e.g., an organizational
DNS server) [default behavior] or can perform its own recursion internally. To
perform local recursion, run zdns with the `--iterative` flag. When this flag
is used, ZDNS will round-robin between the published root servers (e.g.,
198.41.0.4). In iterative mode, you can control the size of the local cache by
specifying `--cache-size` and the timeout for individual iterations by setting
`--iteration-timeout`. The `--timeout` flag controls the timeout of the entire
resolution for a given input (i.e., the sum of all iterative steps).

Running ZDNS
------------

By default, ZDNS will operate with 1,000 light-weight go routines. If you're
not careful, this will overwhelm many upstream DNS providers. We suggest that
users coordinate with local network administrators before performing any scans.
You can control the number of concurrent connections with the `--threads` and
`--go-processes` command line arguments. Alternate name servers can be
specified with `--name-servers`. ZDNS will rotate through these servers when
making requests.

Unsupported Types
-----------------

If zdns encounters a record type it does not support it will generate an output
record with the `type` field set correctly and a representation of the
underlying data structure in the `unparsed_rr` field. Do not rely on the
presence or structure of this field. This field (and its existence) may change
at any time as we expand support for additional record types. If you find
yourself using this field, please consider submitting a pull-request adding
parser support.

License
=======

ZDNS Copyright 2016 Regents of the University of Michigan

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the
License at http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See LICENSE for the specific
language governing permissions and limitations under the License.


 DNS域名解析中A、AAAA、CNAME、MX、NS、TXT、SRV、SOA、PTR各项记录的作用 2016-09-02 17:03:35
分类： 网络与安全

	域名注册完成后首先需要做域名解析，域名解析就是把域名指向网站所在服务器的IP，让人们通过注册的域名可以访问到网站。IP地址是网络上标识服务器的数字地址，为了方便记忆，使用域名来代替IP地址。域名解析就是域名到IP地址的转换过程，域名的解析工作由DNS服务器完成。DNS服务器会把域名解析到一个IP地址，然后在此IP地址的主机上将一个子目录与域名绑定。域名解析时会添加解析记录，这些记录有：A记录、AAAA记录、CNAME记录、MX记录、NS记录、TXT记录、SRV记录、URL转发。



1. DNS域名解析中添加的各项解析记录


	A记录： 将域名指向一个IPv4地址（例如：100.100.100.100），需要增加A记录


	CNAME记录： 如果将域名指向一个域名，实现与被指向域名相同的访问效果，需要增加CNAME记录。这个域名一般是主机服务商提供的一个域名


	MX记录： 建立电子邮箱服务，将指向邮件服务器地址，需要设置MX记录。建立邮箱时，一般会根据邮箱服务商提供的MX记录填写此记录


	NS记录： 域名解析服务器记录，如果要将子域名指定某个域名服务器来解析，需要设置NS记录


	TXT记录： 可任意填写，可为空。一般做一些验证记录时会使用此项，如：做SPF（反垃圾邮件）记录


	AAAA记录： 将主机名（或域名）指向一个IPv6地址（例如：ff03:0:0:0:0:0:0:c1），需要添加AAAA记录


	SRV记录： 添加服务记录服务器服务记录时会添加此项，SRV记录了哪台计算机提供了哪个服务。格式为：服务的名字.协议的类型（例如：_example-server._tcp）。


	SOA记录： SOA叫做起始授权机构记录，NS用于标识多台域名解析服务器，SOA记录用于在众多NS记录中那一台是主服务器


	PTR记录： PTR记录是A记录的逆向记录，又称做IP反查记录或指针记录，负责将IP反向解析为域名


	显性URL转发记录： 将域名指向一个http(s)协议地址，访问域名时，自动跳转至目标地址。例如：将www.liuht.cn显性转发到www.itbilu.com后，访问www.liuht.cn时，地址栏显示的地址为：www.itbilu.com。


	隐性UR转发记录L： 将域名指向一个http(s)协议地址，访问域名时，自动跳转至目标地址，隐性转发会隐藏真实的目标地址。例如：将www.liuht.cn显性转发到www.itbilu.com后，访问www.liuht.cn时，地址栏显示的地址仍然是：www.liuht.cn。

2. DNS解析中一些问题


	2.1 A记录与CNAME记录


	A记录是把一个域名解析到一个IP地址，而CNAME记录是把域名解析到另外一个域名，而这个域名最终会指向一个A记录，在功能实现在上A记录与CNAME记录没有区别。


	CNAME记录在做IP地址变更时要比A记录方便。CNAME记录允许将多个名字映射到同一台计算机，当有多个域名需要指向同一服务器IP，此时可以将一个域名做A记录指向服务器IP，然后将其他的域名做别名(即：CNAME)到A记录的域名上。当服务器IP地址变更时，只需要更改A记录的那个域名到新IP上，其它做别名的域名会自动更改到新的IP地址上，而不必对每个域名做更改。


	2.2 A记录与AAAA记录


	二者都是指向一个IP地址，但对应的IP版本不同。A记录指向IPv4地址，AAAA记录指向IPv6地址。AAAA记录是A记录的升级版本。


	2.3 IPv4与IPv6


	IPv4，是互联网协议（Internet Protocol，IP）的第四版，也是第一个被广泛使用的版本，是构成现今互联网技术的基础协议。IPv4 的下一个版本就是IPv6，在将来将取代目前被广泛使用的IPv4。


	IPv4中规定IP地址长度为32位（按TCP/IP参考模型划分) ，即有2^32-1个地址。IPv6的提出最早是为了解决，随着互联网的迅速发展IPv4地址空间将被耗尽的问题。为了扩大地址空间，IPv6将IP地址的长度由32位增加到了128位。在IPv6的设计过程中除了一劳永逸地解决了地址短缺问题以外，还解决了IPv4中的其它问题，如：端到端IP连接、服务质量（QoS）、安全性、多播、移动性、即插即用等。


	2.4 TTL值


	TTL－生存时间（Time To Live），表示解析记录在DNS服务器中的缓存时间，TTL的时间长度单位是秒，一般为3600秒。比如：在访问www.itbilu.com时，如果在DNS服务器的缓存中没有该记录，就会向某个NS服务器发出请求，获得该记录后，该记录会在DNS服务器上保存TTL的时间长度，在TTL有效期内访问www.itbilu.com，DNS服务器会直接缓存中返回刚才的记录。

下面就简要的介绍下 DNS 的 SOA记录吧：


	在任何 DNS 记录文件（Domain Name System (DNS) Zone file）中， 都是以SOA（Start of Authority）记录开始。SOA 资源记录表明此 DNS 名称服务器是为该 DNS 域中的数据的信息的最佳来源。SOA 记录与 NS 记录的区别:简单讲，NS记录表示域名服务器记录，用来指定该域名由哪个DNS服务器来进行解析;SOA记录设置一些数据版本和更新以及过期时间的信息.


	下面用我的 DNS 的 SOA 记录为例来说明其结构：


		The SOA record is:

Primary nameserver: ns51.domaincontrol.com

Hostmaster E-mail address: dns.jomax.net

Serial #: 2010123100

Refresh: 28800

Retry: 7200

Expire: 604800   1 weeks

Default TTL: 86400
	


	源主机（Primary nameserver）：


	DNS记录文件所在的主机位置。


	联系邮箱（Hostmaster E-mail address）：


	记录主机管理员的联系方式，其中第一个点表示的是@。


	序列号（Serial）：


	格式为yyyymmddnn，nn代表这一天是第几次修改。辅名字服务器通过比较这个序列号是否加载一份新的区数据拷贝。


	refresh（刷新）：


	告诉该区的辅名字服务器相隔多久检查该区的数据是否是最新的。


	retry（重试）：


	如果辅名字服务器超过刷新间隔时间后无法访问主服务器，那么它就开始隔一段时间重试连接一次。这个时间通常比刷新时间短，但也不一定非要这样。


	expire（过期或期满）：


	如果在期满时间内辅名字服务器还不能和主服务器连接上，辅名字服务器就使用这个我失效。这就意味着辅名字服务器将停止关于该区的回答，因为这些区数据太旧了，没有用了。设置时间要比刷新和重试时间长很多，以周为单位是较合理的。


	否定缓存TTL（生存期）：


	这个值对来自这个区的权威名字服务器的否定响应都适用。


	一个Microsoft DNS服务器的SOA记录的数据结构如下：
