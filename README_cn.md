# go-ethernet-ip

一个用 Go 语言实现的 EtherNet/IP 协议库，用于与 Allen-Bradley PLC 等工业自动化设备进行通信。

## 功能特性

- **TCP 连接管理**：支持 EtherNet/IP 设备的 TCP 连接和会话管理
- **标签读写**：支持读取和写入 PLC 标签数据
- **标签组操作**：支持多标签批量读写，提高效率
- **数据类型支持**：支持多种数据类型（BOOL, SINT, INT, DINT, LINT, REAL, LREAL, STRING 等）
- **低级 API**：提供底层 EtherNet/IP 服务接口（ListIdentity, ListServices, ListInterface 等）
- **连接模式**：支持未连接和已连接两种消息发送模式

## 安装

```bash
go get github.com/loki-os/go-ethernet-ip
```

## 快速开始

### 基本用法

```go
package main

import (
    "log"
    go_ethernet_ip "github.com/loki-os/go-ethernet-ip"
)

func main() {
    // 创建 TCP 连接（默认端口 44818）
    conn, err := go_ethernet_ip.NewTCP("192.168.1.10", nil)
    if err != nil {
        log.Fatalln("无法解析主机:", err)
    }

    // 建立连接
    err = conn.Connect()
    if err != nil {
        log.Fatalln("无法连接到主机:", err)
    }

    // 获取所有标签
    tags, err := conn.AllTags()
    if err != nil {
        log.Fatalln("无法获取标签:", err)
    }

    // 读取指定标签
    targetTag := tags["tagName"]
    err = targetTag.Read()
    if err != nil {
        log.Fatalln("读取标签失败:", err)
    }

    // 获取标签值
    intValue := targetTag.Int32()
    stringValue := targetTag.String()
    log.Printf("Int32: %d, String: %s\n", intValue, stringValue)

    // 写入标签值
    targetTag.SetInt32(123)
    err = targetTag.Write()
    if err != nil {
        log.Fatalln("写入标签失败:", err)
    }
}
```

### 标签组批量操作

```go
import (
    "sync"
    go_ethernet_ip "github.com/loki-os/go-ethernet-ip"
)

// 创建标签组
lock := new(sync.Mutex)
tagGroup := go_ethernet_ip.NewTagGroup(lock)

// 添加标签到组
tag1 := tags["tagName1"]
tag2 := tags["tagName2"]
tagGroup.Add(tag1)
tagGroup.Add(tag2)

// 设置标签值
tag1.SetInt32(123)
tag2.SetString("hello")

// 批量写入
err := tagGroup.Write()
if err != nil {
    log.Fatalln("批量写入失败:", err)
}

// 批量读取
err = tagGroup.Read()
if err != nil {
    log.Fatalln("批量读取失败:", err)
}
```

### 使用 Forward Open 建立连接模式

```go
// 建立连接（Forward Open）
err := conn.ForwardOpen()
if err != nil {
    log.Fatalln("Forward Open 失败:", err)
}

// 初始化标签（连接模式下）
var tag = new(go_ethernet_ip.Tag)
conn.InitializeTag("OP.UDT_Alarm.DINT_065_096", tag)
log.Println("Name:", tag.Name())
log.Println("Value:", tag.GetValue())
```

### 低级 API 使用

```go
// 获取设备身份信息
identities, err := conn.ListIdentity()
if err != nil {
    log.Fatalln(err)
}
log.Println(identities)

// 获取接口信息
interfaces, err := conn.ListInterface()
if err != nil {
    log.Fatalln(err)
}
log.Println(interfaces)

// 获取服务列表
services, err := conn.ListServices()
if err != nil {
    log.Fatalln(err)
}
log.Println(services)
```

## 配置选项

```go
// 自定义配置
cfg := &go_ethernet_ip.Config{
    TCPPort:     0xAF12,  // 默认 44818
    UDPPort:     0xAF12,  // 默认 44818
    Slot:        0,       // PLC 插槽号
    TimeTick:    3,
    TimeTickOut: 250,
}

conn, err := go_ethernet_ip.NewTCP("192.168.1.10", cfg)
```

## 支持的数据类型

| 类型 | 说明 | 取值方法 |
|------|------|----------|
| BOOL | 布尔型 | `tag.Bool()` |
| SINT | 8位有符号整数 | `tag.Int8()` |
| USINT | 8位无符号整数 | `tag.UInt8()` |
| INT | 16位有符号整数 | `tag.Int16()` |
| UINT | 16位无符号整数 | `tag.UInt16()` |
| DINT | 32位有符号整数 | `tag.Int32()` |
| UDINT | 32位无符号整数 | `tag.UInt32()` |
| LINT | 64位有符号整数 | `tag.Int64()` |
| ULINT | 64位无符号整数 | `tag.UInt64()` |
| REAL | 32位浮点数 | `tag.Float32()` |
| LREAL | 64位浮点数 | `tag.Float64()` |
| STRING | 字符串 | `tag.String()` |

通用取值方法：`tag.GetValue()` 会根据标签类型返回对应的值。

## 项目结构

```
go-ethernet-ip/
├── bufferx/         # 二进制缓冲区操作工具
├── command/         # 命令处理
├── messages/        # EtherNet/IP 消息实现
│   ├── listIdentity/    # 列出设备身份
│   ├── listInterface/   # 列出接口
│   ├── listServices/    # 列出服务
│   ├── packet/          # 数据包封装
│   ├── registerSession/ # 会话注册
│   ├── sendRRData/      # 请求/响应数据传输
│   ├── sendUnitData/    # 单元数据传输
│   └── unRegisterSession/ # 会话注销
├── path/            # CIP 路径构建
├── types/           # 数据类型定义
├── utils/           # 工具函数
├── config.go        # 配置定义
├── context.go       # 上下文管理
├── example.go       # 使用示例
├── request.go       # 请求处理
├── tag.go           # 标签操作
└── tcp.go           # TCP 连接管理
```

## 协议参考

本项目基于 EtherNet/IP 协议规范实现，参考文档：`1756-pm020_-en-p.pdf`（EtherNet/IP Communication Modules in Logix5000 Control Systems）

## 注意事项

1. **数据类型限制**：当前版本主要支持 Int32 和 String 类型的便捷读写，其他类型需要通过相应的类型方法获取
2. **连接模式**：Forward Open 功能可能需要根据具体设备进行调整
3. **线程安全**：标签操作使用了互斥锁保护，标签组操作需要传入外部锁
4. **数组标签**：支持数组类型标签，如 `wotag[0]`、`wwwtag[1,0,1]` 等

## 许可证

WTFPL (Do What The F*ck You Want To Public License)

```
            DO WHAT THE FUCK YOU WANT TO PUBLIC LICENSE
                    Version 2, December 2004

 Copyright (C) 2004 Sam Hocevar <sam@hocevar.net>

 Everyone is permitted to copy and distribute verbatim or modified
 copies of this license document, and changing it is allowed as long
 as the name is changed.

            DO WHAT THE FUCK YOU WANT TO PUBLIC LICENSE
   TERMS AND CONDITIONS FOR COPYING, DISTRIBUTION AND MODIFICATION

  0. You just DO WHAT THE FUCK YOU WANT TO.
```

## 贡献

欢迎提交 Issue 和 Pull Request！

## 相关链接

- [EtherNet/IP Specification](https://www.odva.org/technology-standards/ethernet-ip/)
- [Go Documentation](https://golang.org/doc/)
