使用 **warp-reg**，注册warp，导出wireguard配置

```bash
curl -sLo warp-reg https://github.com/badafans/warp-reg/releases/download/v1.0/main-linux-amd64 && chmod +x warp-reg && ./warp-reg && rm warp-reg
```

使用 **api.zeroteam.top**，获取warp账号
```bash
curl -sLo /root/warp "https://api.zeroteam.top/warp?format=xray" > /dev/null && grep -Eo --color=never '"2606:4700:[0-9a-f:]+/128"|"secretKey":"[0-9a-zA-Z\/+]+="|"reserved":\[[0-9]+(,[0-9]+){2}\]' warp && rm warp
```

打开 **wireguard.json**，复制"private_key"的值，粘贴到"secretKey": "",处，复制"reserved"的值，粘贴到"reserved":[0, 0, 0],处， 复制"peer_public_key"的值，粘贴到"publicKey: "",处

```json
    "outbounds": [
        {
            "protocol": "wireguard",
            "settings": {
                "secretKey": "",
                "address": [
                    "172.16.0.2/32"
                ],
                "peers": [
                    {
                        "publicKey": "",
                        "allowedIPs": [
                            "0.0.0.0/0"
                        ],
                        "endpoint": "engage.cloudflareclient.com:2408"
                    }
                ],
                "reserved":[0, 0, 0],
                "mtu": 1280
            },
            "tag": "wireguard"
        }
    ]
```

编辑 **<http://ip:port/xui/setting>**，按需增加"routing"和"outbounds"的内容（注意检查json语法）， 选择保存配置 重启面板，访问ip.sb查看是否为Cloudflare的IP

```json
    "routing": {
        "domainStrategy": "IPIfNonMatch",
        "rules": [
            {
                "inboundTag": [
                    "api"
                ],
                "outboundTag": "api",
                "type": "field"
            },
            {
                "type": "field",
                "domain": [
                    "www.gstatic.com"
                ],
                "outboundTag": "direct"
            },
            {
                "type": "field",
                "domain": [
                    "ip.sb",
                    "geosite:openai",
                    "geosite:geolocation-cn",
                    "geosite:cn"
                ],
                "outboundTag": "wireguard"
            },
            {
                "type": "field",
                "ip": [
                    "geoip:cn"
                ],
                "outboundTag": "wireguard"
            }
        ]
    }
```

**模板** 配置示例

```json
{
        "log": {
            "loglevel": "info"
        },
        "api": {
            "services": [
                "HandlerService",
                "LoggerService",
                "StatsService"
            ],
            "tag": "api"
        },
        "inbounds": [
            {
                "listen": "127.0.0.1",
                "port": 62789,
                "protocol": "dokodemo-door",
                "settings": {
                    "address": "127.0.0.1"
                },
                "tag": "api"
            }
        ],
        "outbounds": [
            {
                "protocol": "freedom",
                "settings": {}
            },
            {
                "protocol": "blackhole",
                "settings": {
                    "response": {
                        "type": "http"
                    }
                },
                "tag": "blocked"
            },
            {
                "protocol": "wireguard",
                "settings": {
                    "secretKey": "",
                    "address": [
                        "172.16.0.2/32"
                    ],
                    "network": "tcp,udp",
                    "peers": [
                        {
                            "publicKey": "",
                            "allowedIPs": [
                                "0.0.0.0/0"
                            ],
                            "endpoint": "engage.cloudflareclient.com:2408"
                        }
                    ],
                    "reserved": [
                        209,
                        123,
                        109
                    ],
                    "mtu": 1280
                },
                "tag": "wireguard"
            }
        ],
        "policy": {
            "system": {
                "statsInboundDownlink": true,
                "statsInboundUplink": true,
                "statsOutboundDownlink": true,
                "statsOutboundUplink": true
            },
            "levels": {
                "0": {
                    "handshake": 2,
                    "connIdle": 120,
                    "uplinkOnly": 1,
                    "downlinkOnly": 1
                }
            }
        },
        "routing": {
            "domainStrategy": "IPIfNonMatch",
            "rules": [
                {
                    "inboundTag": [
                        "api"
                    ],
                    "outboundTag": "api",
                    "type": "field"
                },
                {
                    "type": "field",
                    "domain": [
                        "www.gstatic.com"
                    ],
                    "outboundTag": "direct"
                },
                {
                    "type": "field",
                    "domain": [
                        "ip.sb",
                        "geosite:openai",
                        "geosite:geolocation-cn",
                        "geosite:cn"
                    ],
                    "outboundTag": "wireguard"
                },
                {
                    "type": "field",
                    "ip": [
                        "geoip:cn"
                    ],
                    "outboundTag": "wireguard"
                }
            ]
        },
        "stats": {}
    }
```
