https://blog.csdn.net/weiliao_/article/details/129315762

Quectel_Linux_USB_Serial_Option_Driver_20210205.tgz 里面最相近的V3.3.1 可以模仿修改（直接相近的文件覆盖openwrt内核文件build_dir/linux-ar71xx_generic/linux-3.3.8/drivers/usb/serial/）

copy driver to lede/build_dir/target-mipsel_24kc_musl/linux-ramips_mt76x8/linux-5.4.266/drivers/usb/serial

/home/darren/openwrt/lede/build_dir/target-mipsel_24kc_musl/linux-ramips_mt76x8/linux-5.4.266/drivers/usb/serial

build_dir/linux-ar71xx_generic/linux-3.3.8/drivers/usb/serial/

root@OpenWrt:/etc/firewall.user.
iptables -t mangle -N XRAY
iptables -t mangle -A XRAY -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A XRAY -d 100.64.0.0/10 -j RETURN
iptables -t mangle -A XRAY -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A XRAY -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A XRAY -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A XRAY -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A XRAY -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A XRAY -d 240.0.0.0/4 -j RETURN
iptables -t mangle -A XRAY -d 255.255.255.255/32 -j RETURN


iptables -t mangle -A XRAY -d 192.168.0.0/16 -p tcp ! --dport 53 -j RETURN
iptables -t mangle -A XRAY -d 192.168.0.0/16 -p udp ! --dport 53 -j RETURN

iptables -t mangle -A XRAY -p tcp -j TPROXY --on-port 12345 --tproxy-mark 1
iptables -t mangle -A XRAY -p udp -j TPROXY --on-port 12345 --tproxy-mark 1
iptables -t mangle -A PREROUTING -j XRAY

iptables -t mangle -N XRAY_SELF
iptables -t mangle -A XRAY_SELF -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A XRAY_SELF -d 100.64.0.0/10 -j RETURN
iptables -t mangle -A XRAY_SELF -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A XRAY_SELF -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A XRAY_SELF -d 172.16.0.0/12 -j RETURN
iptables -t mangle -A XRAY_SELF -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A XRAY_SELF -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A XRAY_SELF -d 240.0.0.0/4 -j RETURN
iptables -t mangle -A XRAY_SELF -d 255.255.255.255/32 -j RETURN



iptables -t mangle -A XRAY_SELF -d 192.168.0.0/16 -p tcp ! --dport 53 -j RETURN
iptables -t mangle -A XRAY_SELF -d 192.168.0.0/16 -p udp ! --dport 53 -j RETURN
iptables -t mangle -A XRAY_SELF -m mark --mark 2 -j RETURN
iptables -t mangle -A XRAY_SELF -p tcp -j MARK --set-mark 1
iptables -t mangle -A XRAY_SELF -p udp -j MARK --set-mark 1
iptables -t mangle -A OUTPUT -j XRAY_SELF

root@OpenWrt:/etc/rc.local
sysctl -w net.ipv4.ip_forward=1
ip rule add fwmark 1 table 100
ip route add local 0.0.0.0/0 dev lo table 100

exit 0

root@OpenWrt:/etc/config/xray

config xray 'enabled'
        option enabled '1'

config xray 'config'
        option confdir '/etc/xray'
        list conffiles '/etc/xray/config.json'
        option datadir '/usr/share/v2ray'
        option dialer ''
        option format 'json'



root@OpenWrt:/etc/xray/config.json
{
        "log": {
                "dnsLog" : false,
                "loglevel": "none"

        },
        "inbounds": [
                {
                        "tag": "all-in",
                        "port": 12345,
                        "protocol": "dokodemo-door",
                        "settings": {
                                "network": "tcp,udp",
                                "followRedirect": true
                        },
                        "sniffing": {
                                "enabled": true,
                                "destOverride": [
                                        "http",
                                        "tls",
                                        "quic"
                                ],
                                "routeOnly": true
                        },
                        "streamSettings": {
                                "sockopt": {
                                        "tproxy": "tproxy"
                                }
                        }
                }
        ],
        "outbounds": [
                {
                        "tag": "proxy",
                        "protocol": "vless",
                        "settings": {
                                "vnext": [
                                        {
                                                "address": "server ip",
                                                "port": 443,
                                                "users": [
                                                        {
                                                                "id": "my id",
                                                                "security": "auto",
                                                                "encryption": "none",
                                                                "flow": "xtls-rprx-vision"
                                                        }
                                                ]
                                        }
                                ]
                        },
                        "streamSettings": {
                                "network": "tcp",
                                "security": "reality",
                                "realitySettings": {
                                        "show": false,
                                        "fingerprint": "chrome",
                                        "serverName": "www.amazon.com",
                                        "publicKey": "my key",
                                        "shortId": "888888",
                                        "spiderX": ""
                                },
                                "sockopt": {
                                        "mark": 2
                                }
                        },
                        "mux": {
                                "enabled": false,
                                "concurrency": -1
                        }
                },
                {
                        "tag": "dns-out",
                        "protocol": "dns",
                        "settings": {
                                "address": "8.8.8.8"
                        },
                        "proxySettings": {
                                "tag": "proxy"
                        },
                        "streamSettings": {
                                "sockopt": {
                                        "mark": 2
                                }
                        }
                }
        ],
        "dns": {
                "servers": [
                        "8.8.8.8",
                        "1.1.1.1",
                        "https+local://doh.dns.sb/dns-query"
                ]
        },
        "routing": {
                "domainStrategy": "IPIfNonMatch",
                "rules": [
                        {
                                "type": "field",
                                "inboundTag": [
                                        "all-in"
                                ],
                                "port": 53,
                                "outboundTag": "dns-out"
                        },
                        {
                                "type": "field",
                                "ip": [
                                        "8.8.8.8",
                                        "1.1.1.1"
                                ],
                                "outboundTag": "proxy"
                        }
                ]
        }
}
