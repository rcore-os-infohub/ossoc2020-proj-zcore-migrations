# 可以迁移的程序

* GNU Make
* Rust 工具链
* Nginx
* fish
* GCC
* busybox (Force NOMMU)
* Git

## 仓库目录

* `bin/`：`fish`, `nginx`, `make`, `busybox`, `git`
* `lib/`：所有工具依赖的库
* `x86_64-linux-musl`：musl-gcc 工具链

## 使用方法

### `fish`, `nginx`, `gcc`

```bash
cp -d bin/* path_to_zcore/rootfs/bin
cp -d lib/* path_to_zcore/rootfs/lib
cp -rd x86_64-linux-musl/* path_to_zcore/rootfs
```

经过一定修改后即可在 zCore 目录中运行，如：

```bash
cargo run -p linux-loader /bin/fish
cargo run -p linux-loader /bin/nginx
cargo run -p linux-loader /bin/x86_64-linux-musl-gcc
```

### `rustc`

从 [x86\_64-unknown-linux-musl](https://static.rust-lang.org/dist/rust-1.45.2-x86_64-unknown-linux-musl.tar.gz) 下载 Rust 工具链安装包并解压，解压后：

```bash
cd rust-1.45.2-x86_64-unknown-linux-musl
./install.sh --prefix=path_to_zcore/rootfs
```

之后即可运行 rustc：

```bash
cargo run -p linux-loader /bin/rustc
```

## 目前进度

### GNU Make

LibOS 中，会因为 `sys_vfork` 而报段错误

QEMU 中，会报不支持用户态中断

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

GCC 5.3.0 重新编译完成！已放到 [x86\_64-linux-musl](x86_64-linux-musl) 目录内

使用方法：

直接使用如下命令运行，也可在 `rootfs/root/` 内创建 `*.c` 文件供 GCC 编译

```bash
cargo run -p linux-loader /bin/x86_64-linux-musl-gcc -pie -fpie
```

### rustc

使用方法：

```bash
cargo run -p linux-loader /bin/rustc
```

* LibOS 中可以正常输出帮助信息，编译则会报段错误，但可以产生编译中间的 `*.o` 文件
* QEMU 中会报 OOM (out of memory)，原因不明，但不是因为内存不足

### Git

使用方法：

```bash
cargo run -p linux-loader /bin/git
```

目前只能使用与网络无关的功能，如 `git init`
