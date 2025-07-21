# Wind-KVStore

[English](./README.md)

![GitHub License](https://img.shields.io/badge/license-MIT-blue.svg)
![Rust Version](https://img.shields.io/badge/rust-1.65%2B-orange)

Wind-KVStore 是一个轻量级、高效且持久的键值存储引擎，采用 Rust 实现。项目结合了高性能存储引擎与用户友好的命令行界面，适用于需要本地持久化存储的场景。

## ✨ 功能特性

- **📁 持久化存储** - 数据安全写入磁盘，支持崩溃恢复
- **📝 预写日志(WAL)** - 确保操作原子性和持久性
- **⚡ 内存映射文件** - 提供高效的文件访问性能
- **🗂️ LRU 缓存** - 自动管理热点数据缓存
- **🔢 分页存储** - 支持溢出页处理大值数据
- **♻️ 空闲页管理** - 高效复用磁盘空间
- **🗜️ 数据库压缩** - 优化存储空间利用率
- **💻 交互式 Shell** - 提供直观的命令行操作界面

## 🚀 快速开始

### 前置要求

- Rust工具链([安装指南](https://www.rust-lang.org/tools/install))

### 安装与运行

```bash
# 克隆仓库
git clone https://github.com/starwindv/wind-kvstore
cd wind-kvstore

# 构建项目
cargo build --release

# 运行交互式 Shell
cargo run
```

## 💻 交互式 Shell 使用指南

### 启动界面
```
Welcome to Wind-KVStore!

██╗    ██╗    ██╗    ███╗   ██╗    ██████╗ 
██║    ██║    ██║    ████╗  ██║    ██╔══██╗
██║ █╗ ██║    ██║    ██╔██╗ ██║    ██║  ██║
██║███╗██║    ██║    ██║╚██╗██║    ██║  ██║
╚███╔███╔╝    ██║    ██║ ╚████║    ██████╔╝
 ╚══╝╚══╝     ╚═╝    ╚═╝  ╚═══╝    ╚═════╝ 

v0.0.1 [compiled 2023-10-15 14:30:00]
Type ".help;" for usage hints.

KVStore > 
```

### Shell操作示例

```
.open my_database.db;

PUT "username":"john_doe", "email":"john@example.com";

GET WHERE KEY="username";

DEL WHERE KEY="email";

COMPACT;

IDENTIFIER SET "UserDatabase";

IDENTIFIER GET;

.close;

.quit;
```
注意：当前不支持`--`等注释格式
### 📚 Shell 命令参考

| 命令 | 描述 | 示例 |
|------|------|------|
| `.open <path>` | 打开/创建数据库 | `.open data.db;` |
| `.close` | 关闭当前数据库 | `.close;` |
| `.help` | 查看帮助信息 | `.help;` |
| `.clear` | 清屏 | `.clear;` |
| `.title` | 显示标题信息 | `.title;` |
| `.quit` | 退出程序 | `.quit;` |
| `PUT "key":"value"` | 插入键值对 | `PUT "name":"Alice";` |
| `GET WHERE KEY="key"` | 查询键值 | `GET WHERE KEY="age";` |
| `DEL WHERE KEY="key"` | 删除键值 | `DEL WHERE KEY="temp";` |
| `COMPACT` | 压缩数据库 | `COMPACT;` |
| `IDENTIFIER SET "id"` | 设置数据库标识符 | `IDENTIFIER SET "AppDB";` |
| `IDENTIFIER GET` | 获取数据库标识符 | `IDENTIFIER GET;` |

## 📦 作为库使用

### 添加依赖

在 `Cargo.toml` 中添加：

```toml
[dependencies]
wind-kvstore = { git = "https://github.com/starwindv/wind-kvstore" }
```

### lib用法示例

```rust
use wind_kvstore::KVStore;
use anyhow::Result;

fn main() -> Result<()> {
    // 打开数据库
    let mut store = KVStore::open("app_data.db", Some("MyAppDB"))?;
    
    // 存储数据
    store.put(b"username", b"alice")?;
    
    // 检索数据
    if let Some(value) = store.get(b"username")? {
        println!("Username: {}", String::from_utf8_lossy(&value));
    }
    
    // 删除数据
    store.delete(b"username")?;
    
    // 设置数据库标识符
    store.set_identifier("UserDatabase")?;
    
    // 压缩数据库
    store.compact()?;
    
    // 关闭数据库
    store.close()?;
    
    Ok(())
}
```

### 核心 API

```rust
impl KVStore {
    /// 打开或创建数据库
    pub fn open<P: AsRef<Path>>(
        path: P, 
        db_identifier: Option<&str>
    ) -> Result<Self>;
    
    /// 存储键值对
    pub fn put(&mut self, key: &[u8], value: &[u8]) -> Result<()>;
    
    /// 检索键值
    pub fn get(&mut self, key: &[u8]) -> Result<Option<Vec<u8>>>;
    
    /// 删除键值
    pub fn delete(&mut self, key: &[u8]) -> Result<()>;
    
    /// 压缩数据库
    pub fn compact(&mut self) -> Result<()>;
    
    /// 获取数据库标识符
    pub fn get_identifier(&self) -> &str;
    
    /// 设置数据库标识符
    pub fn set_identifier(&mut self, identifier: &str) -> Result<()>;
    
    /// 关闭数据库
    pub fn close(mut self) -> Result<()>;
}
```

## 🏗️ 项目结构

```
.
├── build.rs            # 构建脚本（记录编译时间）
├── Cargo.toml          # 项目配置和依赖管理
├── README.md           # 项目文档
├── README_CN.md           # 项目中文文档
└── src
    ├── kvstore.rs      # 键值存储引擎核心实现
    ├── main.rs         # 程序入口点
    └── shell.rs        # 交互式命令行 Shell 实现
```

## ⚙️ 技术实现

### 存储架构
```
+-----------------------+
|      文件头 (128B)     |
+-----------------------+
|      页头 (16B)        |
+-----------------------+
|       键值数据          |
+-----------------------+
|      溢出页指针         |
+-----------------------+
|         ...           |
+-----------------------+
```

### 关键特性

1. **分页存储**  
   - 固定大小页面（默认 1KB）
   - 支持溢出页处理大值数据
   - 空闲页链表管理

2. **预写日志**  
   - 操作日志记录
   - 崩溃后自动恢复
   - 原子性操作保证

3. **缓存管理**  
   - LRU 缓存策略
   - 自动热点数据管理
   - 100KB 默认缓存大小

4. **空间优化**  
   - 自动空闲页回收
   - 在线数据库压缩
   - 高效存储布局

## 🤝 贡献指南

我们欢迎任何形式的贡献！
我们建议您在做出贡献时：
 - 更新相关文档
 - 提交清晰的 PR 描述

## 📜 许可证

本项目采用 [MIT 许可证](https://github.com/StarWindv/Wind-KVStore/LICENSE)
