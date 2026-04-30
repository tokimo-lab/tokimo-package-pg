# tokimo-package-pg

预编译的 PostgreSQL + pgvector，用于嵌入 Tauri 桌面应用。

## 产物

每次构建产出两个平台的可移植 zip 包：

| 平台 | 架构 | 文件名 |
|---|---|---|
| Windows | x64 (MSVC) | `pg-{版本}-windows-x64.zip` |
| macOS | arm64 (Apple Silicon) | `pg-{版本}-macos-arm64.zip` |

zip 解压后是可独立运行的 PostgreSQL 安装目录，包含 `bin/`、`lib/`、`share/`，可嵌入 Tauri sidecar。

## CI 参数

在 [Actions](https://github.com/tokimo-lab/tokimo-package-pg/actions) 页面手动触发，可配置：

| 参数 | 默认值 | 说明 |
|---|---|---|
| `pg_version` | `REL_18_3` | PostgreSQL 源码 git tag |
| `pgvector_version` | `v0.8.2` | pgvector 源码 git tag |
| `ssl` | `false` | 是否编译 OpenSSL 支持 |

每次构建自动以 `pg{版本}-vec{版本}` 格式打 tag 并发布 Release。

## 已验证

CI 中每平台均执行：

1. `initdb` 初始化数据库集群
2. `pg_ctl start` 启动 PostgreSQL
3. `CREATE EXTENSION vector` 加载 pgvector
4. 向量插入、L2 距离查询、IVFFlat 索引创建
5. `pg_ctl stop` 正常关闭

## Tauri 集成

```
tauri-app/
  src-tauri/
    binaries/
      pg-windows-x64/    ← 解压到此
        bin/postgres.exe
        bin/initdb.exe
        lib/
        share/
```

在 Tauri `tauri.conf.json` 中配置为 external binary，通过 `Command::new_sidecar` 调用 `initdb`、`postgres` 等命令。

## 本地构建

```bash
# 需要 meson, ninja, C 编译器

# 编译 PG
git clone --depth 1 --branch REL_18_3 https://github.com/postgres/postgres.git
cd postgres
meson setup build --prefix=/path/to/install --buildtype=release -Dssl=none
ninja -C build
ninja -C build install

# 编译 pgvector
git clone --depth 1 --branch v0.8.2 https://github.com/pgvector/pgvector.git
cd pgvector
export PATH=/path/to/install/bin:$PATH
make
make install
```
