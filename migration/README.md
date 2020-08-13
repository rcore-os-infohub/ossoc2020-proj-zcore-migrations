# 可以迁移的程序

* GNU Make
* Rust 工具链
* Nginx
* fish
* GCC

## 仓库目录

* `bin/`：`fish`, `nginx`, `make`
* `lib/`：所有工具依赖的库
* `root/`：musl-rust 工具链
* `x86_64-linux-musl`：musl-gcc 工具链

## 使用方法

```bash
cp -d bin/* path_to_zcore/rootfs/bin
cp -d lib/* path_to_zcore/rootfs/lib
cp -rd root path_to_zcore/rootfs/
```

经过一定修改后即可在 zCore 目录中运行，如：

```bash
cargo run -p linux-loader /bin/fish
cargo run -p linux-loader /bin/nginx
cargo run -p linux-loader /root/.cargo/bin/rustc
cargo run -p linux-loader /bin/gcc
```

## 目前进度

### GNU Make

可以直接运行，但是编写 `Makefile` 并运行*可能*需要 `sys_fork`（未测试）

### Rust 工具链

不需要网络的部分运行不报错，但是也没有输出（正在解决）

### Nginx

使用方法：

```bash
mkdir -p rootfs/var/lib/nginx/logs
touch rootfs/var/lib/nginx/logs/error.log
cargo run -p linux-loader /bin/nginx
```

缺少系统调用 `SCHED_GETAFFINITY`

### fish

使用方法：

1. 注释掉报错的 `linux-object/src/fs/file.rs` 173 行

2. 修改 `linux-syscall/src/lib.rs`，在 250 行 `Sys::DUP2` 前插入如下内容：

   ```rust
   Sys::DUP => self.sys_dup2(a0.into(), a1.into()),
   ```

3. 修改 `linux-loader/src/main.rs` 第 22 行，将数组最后添加 `, "HOME=/root".into()`

4. 使用如下命令运行：

   ```bash
   cargo run -p linux-loader /bin/fish
   ```

   即可看到缺少系统调用 `SOCKET`

### GCC

GCC 5.3.0 重新编译完成！已放到 [x86_64-linux-musl](x86_64-linux-musl) 目录内