---
title: NMS协议包识别引擎
date: 2017-04-02 15:33:14
categories: 技术研究
tags:
- Python
- 设计模式
---

基于工厂模式的包识别引擎设计，和一般工厂模式不同是：

- 匹配规则查询表中是一个树形结构，既有精确规则，又有模糊规则，匹配的过程需要多次查询的操作，从精确到模糊直至无法匹配。
- 匹配规则查询表能够支持添加新规则操作

<!-- more -->

## 1 问题描述

**A NMS基础协议**

NMS是服务器与硬件设备(具体为RTU或者网关)工作的通信协议，协议格式描述如下：

| 网关地址 | 节点地址 | 源类型 | 包序号 | 包类型 | 负载长度 | 负载 |
| ------ | ------ | ------ | ------ | ------ | ------ | ------ |
|  gateway_mac | rtu_mac | source_type | packet_id | packet_type | payload_length | payload |
| 8 | 8 | 1 | 4 | 1 | 1 | - |

前23字节为包头部，各个字段描述如下：

- 网关地址：64位长整型，
- 节点地址：64位长整型，
- 源类型：表示哪一种设备，主要有 Gateway GPRS-RTU Mesh-RTU 三种类型的设备。
- 包序号：32位整数
- 包类型：该包的类型。
- 负载长度：整数0-255，表示后面的负载长度，实际中负载长度小于255

源类型取值定义如下：

| 设备分类 | 设备类型取值 | 描述 |
| ------ | ------ | ------ |
| 网关 | 0x03 | 自组网网关 |
| | 0x06 | 以太网网关 |
| Mesh-RTU | 0x01 | 自组网RTU，通过自组网关与服务器相连 |
| GPRS-RTU | 0x02 | GPRS-RTU，与服务器直连 |
| web服务器 | 0x04 | 由服务器或者上层应用发起 |


**B 应用协议**

NMS可承载多种设备应用协议，比如Modbus协议、DTU低功耗设备协议、远程抄表协议等，每个应用协议使用1字节的整数标识，称之为应用类型(app_id)。**由于一些类型的包与上层应用无关，所以在设计中，NMS协议头部没有专门的应用类型字段。** 它的规则如下：

- 应用类型存储在payload中，并且位置不一定相同，但由包类型决定，即由packet_type和payload两个参数可计算出具体应用类型数值
- 在应用协议扩展过程中，属于统一应用协议的设备（网关或节点）既可以使用现有的设备类型取值，也可申请新的设备类型取值。这取决于具体应用场景，比如Modbus协议中，网关设备只起数据转化的功能，可以部署通用网关，设备类型取值为0x03，远程抄表协议中，网关也能响应服务器的动作，申请新的设备类型取值。

在上述描述中，设备类型(source_type)和设备分类(source_catalog)的概念是不一样的。加入上层应用协议后，它们的关系，

| 设备分类 | 设备分类取值 | 设备类型取值 | 应用协议 | 描述 |
| ------ | ------ | ------ | ------ | ------ |
| 网关 | gateway |  0x03 | 通用 | 自组网网关 |
| | | 0x06 |通用 | 以太网网关 |
| | | 0x07 | 抄表应用(0x01) | 网关 |
| | | ... |... | ... |
| Mesh-RTU | mesh_rtu | 0x01 | 通用 | 自组网RTU，通过自组网关与服务器相连 |
| GPRS-RTU | gprs_rtu | 0x02 | 通用 | GPRS-RTU，与服务器直连 |
|  | | 0x08 | 报警 |  |
| | | ... |... | ... |
| web服务器 | - | 0x04 | - | 由服务器或者上层应用发起 |




以具体到NMS协议， 包类型与应用类型的对应关系（部分）如下：

| 包类型 | 取值 | 是否应用协议相关 | 是否设备相关 | 描述 |
| ------ | ------ | ------ | ------ |
| 数据包 | 0x01 | 是 | 无 | 设备自动上报的数据 |
| 心跳包 | 0x04 | 否 | 设备分类相关 | 维持在线状态 |
| 时间同步包 | 0x06 | 否 | 否 | 由服务器发送 |




设计基于树结构的包结构体系，并实现给定一段符合格式NMS的包二进制字节数组，识别所对应的包种类，返回实例化的包对象。

## 2 基本识别过程

基于(packet_type, source_type, app_id)的识别引擎主框架代码如下。

packet.py

```python

class PacketBase(object):
    packet_type = None
    source_catalog = None
    source_type = None
    app_related = False
    app_id = None

    def __init__(self, gateway_mac, rtu_mac, source_type, packet_id, packet_type, payload_length, payload, **kwargs):
        self.gateway_mac = gateway_mac
        self.rtu_mac = rtu_mac
        self.source_type = source_type or self.source_type
        self.packet_id = packet_id
        self.packet_type = packet_type or self.packet_type
        self.payload_length = payload_length
        self.payload = payload

```
engine.py

```python
import struct

class AppProtocolBase(object):
    app_id = None

class Engine(object):
    def __init__(self):
        self._packet_lookup = {} # 包类对象查询表，为(packet_type, source_type, app_id)到包类对象的映射
        self._protocol_lookup = {}

    def register_protocol(self, protocol_class):
        pass

    def add_lookup_item(self, packet_type, source_catalog, source_type, app_id, packet_class):
        # 添加匹配规则
        pass

    def identify(self, packet_binary):
        header, payload = packet_binary[:23], packet_binary[23:]
        gateway_mac, rtu_mac, source_type, packet_id, packet_type, payload_length = struct.unpack('>QQBIBB', header)

        cls = self._identify_class(packet_type, source_type, payload)
        if cls:
            return cls(gateway_mac, rtu_mac, source_type, packet_id, packet_type, payload_length, payload)
    def identify_class(self, packet_type, source_type, payload):
        # 输入为包索引(packet_type, source_type, app_id)
        # 在查询过程中，可能需要经过精确到模糊的查找过程，并不是简单的 dict.get 调用
        pass
```

## 3 查询表(lookup)

### 3.1 构建基本查询表

包索引即为查询表中的键

packet.py

```python
# 构建

class DataPacket(PacketBase):
    packet_type = 0x01
    app_related = True

class GatewayHeartbeatPacket(PacketBase):
    packet_type = 0x04
    source_catalog = 'gateway'

class MeshRTUHeartbeatPacket(PacketBase):
    packet_type = 0x04
    source_catalog = 'mesh_rtu'

class GPRSHeartbeatPacket(PacketBase):
    packet_type = 0x04
    source_catalog = 'gprs_rtu'

class TimeSyncPacket(PacketBase):
    packet_type = 0x06
    source_catalog = 0x04  


def build_lookup(*args):
    packet_lookup = {}
    app_related_packet_type_set = {}
    for packet_class in args:
        packet_lookup[(packet_class.packet_type, packet_class.source_type or packet_class.source_type, None)] = packet_class
        if packet_class.app_related:
            app_related_packet_type_set.add(packet_class.packet_type)
    return packet_lookup, app_related_packet_type_set

# 两个查询表计算过程
# PACKET_LOOKUP, APP_RELATED_PACKET_TYPE_SET = build_lookup(
#    DataPacket,
#    GatewayHeartbeatPacket,
#    GPRSHeartbeatPacket,
#    MeshRTUHeartbeatPacket,
#    TimeSyncPacket
# )

# 为了下面示例方面，直接给出最后结果
PACKET_LOOKUP =  {
  (0x01, None, None): DataPacket,
  (0x04, 'gateway', None): GatewayHeartbeatPacket,
  (0x04, 'gprs_rtu', None): GPRSHeartbeatPacket,
  (0x04, 'mesh_rtu', None): MeshRTUHeartbeatPacket,
  (0x06, 0x04, None): TimeSyncPacket
}

# 所有与应用相关的包类型集合
APP_RELATED_PACKET_TYPE_SET = { 0x01 }

SOURCE_TYPE_CATALOG_LOOKUP = {
    0x01: 'mesh_rtu',
    0x02: 'gprs_rtu',
    0x03: 'gateway',
    0x04: 'gateway'
}

SOURCE_CATALOG_SET = {'gateway', 'mesh_rtu', 'gprs_rtu'}
```

### 3.2 基于应用协议扩展识别规则查询表

注册应用协议实际为上述查询表增加了更加精确的匹配规则，当本应用协议的包类对象无法使用才会使用基本的包类对象。在添加匹配规则时：

- 应用类型：应用类型不能与已有的冲突
- 包类型：只能注册那些与应用相关的类型的包(类型存储在 `APP_RELATED_PACKET_TYPE_SET`)，因为与应用无关的包没有存储应用类型字段，识别时无法分发。
- 源类型和源分类
    - 使用已有的设备分类，对应于若干种设备
    - 使用已有的设备类型
    - 使用新的设备类型，不能和已有的相冲突，同时必须指定设备分类


实现过程如下：

```python

import copy
from .packet import PACKET_LOOKUP, APP_RELATED_PACKET_TYPE_SET, SOURCE_CATALOG_SET, SOURCE_TYPE_CATALOG_LOOKUP

class Engine(object):
    def __init__(self):
        self._packet_lookup = copy.copy(PACKET_LOOKUP) # 包类对象查询表，为(packet_type, source_type, app_id)到包类对象的映射
        self._source_type_catalog_lookup = copy.copy(SOURCE_TYPE_CATALOG_LOOKUP)
        self._protocol_lookup = {}

    def register_protocol(self, protocol_class):
        if protocol_class.app_id in self._protocol_lookup:
            raise ValueError('The app_id value {} in the {} has conflicted!'.format(protocol_class.app_id, protocol_class.__name__))
        self._protocol_lookup[protocol_class.app_id] = protocol_class
        for packet_class in protocol_class.packet_lookup:
            self.add_lookup_item(packet_class.packet_type, protocol_class.source_catalog, protocol_class.source_type, protocol_class.app_id, protocol_class)

    def add_lookup_item(self, packet_type, source_catalog, source_type, app_id, packet_class):
        if packet_type not in APP_RELATED_PACKET_TYPE_SET:
            raise ValueError('Protocol Register with packet_type {} is not supported!'.format(packet_type))
        if source_type in self._source_type_catalog_lookup:
            self._packet_lookup[(packet_type, source_type, protocol_class.app_id)] = packet_class
        else:
            if source_catalog in SOURCE_CATALOG_SET:
                self._packet_lookup[(packet_type, source_catalog, protocol_class.app_id)] = packet_class
            else:
                raise ValueError('Invalid source_catalog')

```

### 3.3 使用方法和测试案例

```python

class HydrantDataPacket(DataPacket):
    pass

class HydrantProtocol(AppProtocolBase):
    app_id = 0x0A
    packet_lookup = { HydrantDataPacket }

engine = Engine()

engine.register_protocol(HydrantProtocol)

payload = struct.pack('>BBBBB', 0x0A, 3, 4, 5, 6)
test_binary = struct.pack('>QQBIBB', 70971071088567232, 70971071088567240, 13, 0x01, 5) + payload

packet = engine.indentfity(test_binary)

assert isinstance(packet, HydrantProtocol) == True

```


## 4 识别匹配

- 查询 (packet_tpye, source_type, None)
    - 是：计算app_id，查询(packet_tpye, source_catalog, app_id)
        - 是：使用(packet_tpye, source_catalog, app_id)
        - 否：使用(packet_tpye, source_type, None)
    - 否：当前packet_tpye是否有app_id
        - 是：计算app_id，查询 (packet_tpye, source_catalog, app_id)
          - 是：使用 (packet_tpye, source_catalog, app_id)
          - 否：无法识别
        - 否：查询 (packet_tpye, source_catalog, None)
          - 是：使用(packet_tpye, source_catalog, None)
          - 否：无法识别

```python
class Engine(object):

  def identify_class(self, packet_type, source_type, payload)
      cls = self._packet_lookup.get((packet_type, source_type, None))
         if cls:
             source_type_or_catalog = source_type
         else:
             source_catalog = self._source_type_catalog_lookup.get(source_type)
             cls = self._packet_lookup.get((packet_type, source_catalog, None))
             if cls:
                 source_type_or_catalog = source_catalog
             else:
                 source_type_or_catalog = None
         if packet_type in APP_PACKET_TYPE_LOOKUP and cls:
             app_id = cls.parse_app_id()
             t_cls = self._packet_lookup.get((packet_type, source_type_or_catalog, app_id))
             if t_cls:
                 cls = t_cls
         return cls
```
