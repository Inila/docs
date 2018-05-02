# SNMP(Simple Network Management Protocol)

中文意思是"简单网络管理协议"。 SNMP是一种简单网络管理协议，它属于TCP/IP五层协议中的应用层协议，用于网络管理的协议。 SNMP主要用于网络设备的管理

## SNMP 常用的OID

| Index          | .1.3.6.1.2.1.31.1.1.1.6   |
| HighSpeed      | .1.3.6.1.2.1.31.1.1.1.15  |
| IfInUcastPkts  | .1.3.6.1.2.1.2.2.1.11     |
| IfOutUcastPkts | .1.3.6.1.2.1.2.2.1.17     | 
| Alias          | .1.3.6.1.2.1.31.1.1.1.18  |
| Status         | .1.3.6.1.2.1.2.2.1.8      |
| IfHCOutOctets  | .1.3.6.1.2.1.31.1.1.1.10  |
| IfHCInOctets   | .1.3.6.1.2.1.31.1.1.1.6   |

IfHCOutOctets 和 IfHCInOctets 表示流经交换机端口的字节(Byte)数,是一个计数器类型,HC表示是64位长度计数器,若是32位长度OID则当带宽超过4G时,计数器会清0,重新计数,造成带宽数据计算错误
