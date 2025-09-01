# ECH (Encrypted Client Hello) 功能支持

## 概述

Txray 现已支持 TLS 配置中的 `echForceQuery` 参数，该参数用于控制 ECH (Encrypted Client Hello) 的强制查询行为。

## 支持的协议

以下协议现在支持 `echForceQuery` 参数：

1. **VLESS** - 支持 TLS 和 XTLS 传输
2. **VMessAEAD** - 支持 TLS 传输
3. **VMess** - 支持 TLS 传输
4. **Trojan** - 支持 TLS 传输

## 参数值

`echForceQuery` 参数支持以下三种值：

- `"full"` - 完全强制查询
- `"half"` - 半强制查询
- `"none"` - 不强制查询（默认值）

## 使用方法

### 1. 通过命令行界面添加节点

在添加节点时，当选择 TLS 安全传输时，系统会提示您选择 ECH 强制查询模式：

```
ECH强制查询（echForceQuery）?
[0] 
[1] full
[2] half
[3] none
```

### 2. 通过链接导入

支持在节点链接中包含 `echForceQuery` 参数：

- **VLESS**: `vless://id@address:port?security=tls&sni=example.com&echForceQuery=full#remarks`
- **VMessAEAD**: `vmess://id@address:port?security=tls&sni=example.com&echForceQuery=half#remarks`
- **VMess**: 在 JSON 配置中包含 `"echForceQuery": "none"`
- **Trojan**: `trojan://password@address:port?sni=example.com&echForceQuery=full#remarks`

## 生成的配置示例

当启用 ECH 功能时，生成的 Xray 配置将包含以下 TLS 设置：

```json
{
  "serverName": "example.com",
  "allowInsecure": false,
  "alpn": ["h2", "http/1.1"],
  "echForceQuery": "full"
}
```

## 技术实现

### 字段定义

在 `core/protocols/field/field.go` 中添加了：

```go
EchForceQuery = NilStrField("echForceQuery") // ECH强制查询，可选值 full/half/none (default)
```

### 协议支持

1. **VLESS**: 使用 `url.Values` 存储参数，通过 `GetValue(field.EchForceQuery)` 获取
2. **VMessAEAD**: 使用 `url.Values` 存储参数，通过 `GetValue(field.EchForceQuery)` 获取
3. **VMess**: 在结构体中添加 `EchForceQuery string` 字段
4. **Trojan**: 使用 `url.Values` 存储参数，通过 `EchForceQuery()` 方法获取

### 配置生成

在 `xray/config.go` 中，所有支持 TLS 的协议都会检查 `echForceQuery` 参数，如果存在则添加到 `tlsSettings` 中。

## 注意事项

1. `echForceQuery` 参数仅在启用 TLS 安全传输时有效
2. 默认值为空字符串，表示不设置该参数
3. 该功能需要 Xray-core 支持 ECH 功能
4. 不同的 `echForceQuery` 值可能影响连接的安全性和性能

## 版本要求

- Txray: 支持 ECH 功能的版本
- Xray-core: 支持 ECH 功能的版本
