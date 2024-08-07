# DNS-Unlock

## 使用的DNS服务(部分服务还在调试中,若出现问题请提交Is或者在群组中提交问题以便解决)

默认服务 [https://dns.afosne.icu/afosne](https://dns.afosne.icu/afosne)<br/>

#### 以下服务器目前仅支持Netflix 和 Disneyplus 等部分网站的加速.后续将持续更新服务

ANYCAST服务器[https://dns.afosne.icu/afosneanycast](https://dns.afosne.icu/afosneanycast)<br/>
澳大利亚服务器[https://dns.afosne.icu/afosneau](https://dns.afosne.icu/afosneau)<br/>
欧洲服务器[https://dns.afosne.icu/afosneeu](https://dns.afosne.icu/afosneeu)<br/>
香港服务器[https://dns.afosne.icu/afosnehk](https://dns.afosne.icu/afosnehk)<br/>
日本服务器[https://dns.afosne.icu/afosnejp](https://dns.afosne.icu/afosnejp)<br/>
韩国服务器[https://dns.afosne.icu/afosnekr](https://dns.afosne.icu/afosnekr)<br/>
新加坡服务器[https://dns.afosne.icu/afosnesg](https://dns.afosne.icu/afosnesg)<br/>
泰国服务器[https://dns.afosne.icu/afosneth](https://dns.afosne.icu/afosneth)<br/>
土耳其服务器[https://dns.afosne.icu/afosnetr](https://dns.afosne.icu/afosnetr)<br/>
台湾地区服务器[https://dns.afosne.icu/afosnetw](https://dns.afosne.icu/afosnetw)<br/>
美国服务器[https://dns.afosne.icu/afosneus](https://dns.afosne.icu/afosneus)<br/>
赞助服务器[https://dns.afosne.icu/afosnefreeany](https://dns.afosne.icu/afosnefreeany)<br/>

## 解锁用服务器探针

[https://status.afosne.icu/](https://status.afosne.icu/)

## 优化使用方式

### CLASH Verge可以加入js代码来提供解锁服务

```javascript
function main(content) {
  const isObject = (value) => {
    return value !== null && typeof value === 'object'
  }

  const mergeConfig = (existingConfig, newConfig) => {
    if (!isObject(existingConfig)) {
      existingConfig = {}
    }
    if (!isObject(newConfig)) {
      return existingConfig
    }
    return { ...existingConfig, ...newConfig }
  }

  const cnDnsList = [
    'https://1.12.12.12/dns-query',
    'https://223.5.5.5/dns-query',
  ]
  
  // 大部分的网络请求都会走这个里面的，这里目前是腾讯、阿里、和使用节点查询的1.0.0.1的dns
  const trustDnsList = [
    'https://dns.afosne.icu/afosne', 

  ]
  const afosneDns = 'https://dns.afosne.icu/afosne' 
  const afosneUrls = [
    '+.netflix.com',
    '+.nflxext.com',
    '+.disneyplus.com',

    'https://translate.google.com',
  ]
  const combinedUrls = afosneUrls.join(',');
  
  const dnsOptions = {
    'enable': true,
    'prefer-h3': true, // 如果DNS服务器支持DoH3会优先使用h3（本例子中只有阿里DNS支持）
    'default-nameserver': cnDnsList, // 用于解析其他DNS服务器、和节点的域名, 必须为IP, 可为加密DNS。注意这个只用来解析节点和其他的dns，其他网络请求不归他管
    'nameserver': trustDnsList, // 其他网络请求都归他管
    
    // 这个用于覆盖上面的 nameserver
    'nameserver-policy': {
      [combinedUrls]: afosneDns,
      'geosite:geolocation-!cn': trustDnsList,
      // 如果你有一些内网使用的DNS，应该定义在这里，多个域名用英文逗号分割
      // '+.公司域名.com, www.4399.com, +.baidu.com': '10.0.0.1'
    },
  }

  // GitHub加速前缀
  const githubPrefix = 'https://fastgh.lainbo.com/'

  // GEO数据GitHub资源原始下载地址
  const rawGeoxURLs = {
    geoip: 'https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip-lite.dat',
    geosite: 'https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geosite.dat',
    mmdb: 'https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/country-lite.mmdb',
  }

  // 生成带有加速前缀的GEO数据资源对象
  const accelURLs = Object.fromEntries(
    Object.entries(rawGeoxURLs).map(([key, githubUrl]) => [key, `${githubPrefix}${githubUrl}`]),
  )

  const otherOptions = {
    'unified-delay': true,
    'tcp-concurrent': true,
    'profile': {
      'store-selected': true,
      'store-fake-ip': true,
    },
    'sniffer': {
      enable: true,
      sniff: {
        TLS: {
          ports: [443, 8443],
        },
        HTTP: {
          'ports': [80, '8080-8880'],
          'override-destination': true,
        },
      },
    },
    'geodata-mode': true,
    'geo-auto-update': true,
    'geo-update-interval': 24,
    'geodata-loader': 'standard',
    'geox-url': accelURLs,
    'find-process-mode': 'strict',
  }
  content.dns = mergeConfig(content.dns, dnsOptions)
  return { ...content, ...otherOptions }
}
```

### singbox可以修改配置文件来解锁服务

```javascript
{
  "log": {
    "disabled": false,
    "level": "warn",
    "output": "/var/run/homeproxy/sing-box-c.log",
    "timestamp": true
  },
  "dns": {
    "servers": [
      {
        "tag": "default-dns",
        "address": "223.5.5.5",
        "detour": "direct-out"
      },
      {
        "tag": "system-dns",
        "address": "local",
        "detour": "direct-out"
      },
      {
        "tag": "block-dns",
        "address": "rcode://name_error"
      },
      {
        "tag": "afosne",
        "address": "https://dns.afosne.icu/afosne",
        "address_resolver": "default-dns",
        "address_strategy": "ipv4_only",
        "strategy": "ipv4_only",
        "client_subnet": "1.0.1.0"
      }
    ],
    "rules": [
      {
        "outbound": "any",
        "server": "default-dns"
      },
      {
        "query_type": "HTTPS",
        "server": "block-dns"
      },
      {
        "clash_mode": "direct",
        "server": "default-dns"
      },
      {
        "clash_mode": "global",
        "server": "afosne"
      },
      {
        "rule_set": "cnsite",
        "server": "default-dns"
      }
    ],
    "strategy": "ipv4_only",
    "disable_cache": false,
    "disable_expire": false,
    "independent_cache": false,
    "final": "afosne"
  },
  "inbounds": [
    {
      "type": "direct",
      "tag": "dns-in",
      "listen": "::",
      "listen_port": 5333
    },
    {
      "type": "mixed",
      "tag": "mixed-in",
      "listen": "::",
      "listen_port": 5330,
      "sniff": true,
      "sniff_override_destination": false,
      "set_system_proxy": false
    },
    {
      "type": "redirect",
      "tag": "redirect-in",
      "listen": "::",
      "listen_port": 5331,
      "sniff": true,
      "sniff_override_destination": false
    },
    {
      "type": "tproxy",
      "tag": "tproxy-in",
      "listen": "::",
      "listen_port": 5332,
      "network": "udp",
      "sniff": true,
      "sniff_override_destination": false
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct-out",
      "routing_mark": 100
    },
    {
      "type": "block",
      "tag": "block-out"
    },
    {
      "type": "dns",
      "tag": "dns-out"
    },
    {
      "type": "urltest",
      "tag": "自动选择",
      "outbounds": [
        "香港",
        "日本",
        "美国"
      ]
    },
    {
      "type": "selector",
      "tag": "手动选择",
      "outbounds": [
        "direct-out",
        "block-out",
        "自动选择",
        "香港",
        "日本",
        "美国"
      ],
      "default": "自动选择"
    },
    {
      "type": "selector",
      "tag": "GLOBAL",
      "outbounds": [
        "direct-out",
        "香港",
        "日本",
        "美国"
      ],
      "default": "手动选择"
    },
    {
      "type": "shadowsocks",
      "tag": "香港",
      "routing_mark": 100,
      "server": "abc.com",
      "server_port": 10001,
      "password": "fdc43e321a",
      "method": "aes-128-gcm"
    },
    {
      "type": "shadowsocks",
      "tag": "日本",
      "routing_mark": 100,
      "server": "abc.com",
      "server_port": 10002,
      "password": "fdc43e321a",
      "method": "aes-128-gcm"
    },
    {
      "type": "shadowsocks",
      "tag": "美国",
      "routing_mark": 100,
      "server": "abc.com",
      "server_port": 10003,
      "password": "fdc43e321a",
      "method": "aes-128-gcm"
    }
  ],
  "route": {
    "rules": [
      {
        "inbound": "dns-in",
        "outbound": "dns-out"
      },
      {
        "protocol": "dns",
        "outbound": "dns-out"
      },
      {
        "protocol": "quic",
        "outbound": "block-out"
      },
      {
        "clash_mode": "direct",
        "outbound": "direct-out"
      },
      {
        "clash_mode": "global",
        "outbound": "GLOBAL"
      },
      {
        "rule_set": [
          "cnip",
          "cnsite"
        ],
        "outbound": "direct-out"
      }
    ],
    "rule_set": [
      {
        "type": "remote",
        "tag": "cnip",
        "format": "binary",
        "url": "https://github.com/MetaCubeX/meta-rules-dat/raw/sing/geo-lite/geoip/cn.srs",
        "download_detour": "自动选择"
      },
      {
        "type": "remote",
        "tag": "cnsite",
        "format": "binary",
        "url": "https://github.com/MetaCubeX/meta-rules-dat/raw/sing/geo-lite/geosite/cn.srs",
        "download_detour": "自动选择"
      }
    ],
    "auto_detect_interface": true,
    "final": "手动选择"
  },
  "experimental": {
    "cache_file": {
      "enabled": true,
      "path": "/etc/homeproxy/cache.db"
    },
    "clash_api": {
      "external_controller": "192.168.2.1:9090",
      "external_ui": "/etc/homeproxy/ui/",
      "external_ui_download_detour": "自动选择",
      "default_mode": "rule"
    }
  }
}
```

## 目前该服务已经提供的加速服务有

服务加速

- [x] Netflix 
- [x] Disneyplus.com 
- [x] Hulu 
- [x] 谷歌翻译
- [x] OP.GG
- [x] Spotify 播放列表加载和音频流
- [x] Notion 笔记
- [x] Cloudflare worker 和 pages
- [x] Github Github raw
- [x] OPENAI

游戏加速

- [ ] 测试中

# 后期维护和优化

面板开发和用户自定义配置模板
满足各个区服之间的解锁和使用。
使用dns直接解锁登录Netflix，Disney，等网站会员。
建立负载均衡以满足大量用户的需求。

# 有问题可以进入群组讨论 [Dns_Unlock](https://t.me/Dns_Unlock)


