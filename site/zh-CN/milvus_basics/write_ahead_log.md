---
id: write_ahead_log.md
title: Write Ahead Log
sidebar_label: Write Ahead Log
---

# 预写式日志

![wal_structure](../../../assets/wal/wal_workflow.jpg)

预写式日志首先把用户的插入和删除请求记入日志文件，然后由后台线程写入系统。一旦将用户请求成功写入日志，服务端即会返回成功。开启该功能可以增强数据的可靠性，并减少对客户端的阻塞。

## 数据可靠性

预写式日志能保证修改请求的原子性。所有返回成功的请求都会被完整地写入系统。对于因系统意外退出或者链接意外断开而没有响应的请求，操作只可能全部成功或者全部失败。操作是否成功可以通过调用其它接口来确认。此外，在系统重启时，日志中还未被应用到系统状态的请求将被重新执行。

## 缓冲区设置

预写式日志使用的缓冲区大小由系统参数 `wal.buffer_size` 决定。为保证预写式日志的写入性能，建议把缓冲区大小设为单批次导入数据量大小的 2 倍以上。

<div class="alert info">
    关于如何设置系统参数 <code>wal.buffer_size</code>，请见 <a href="server_config.md">Milvus 配置</a>。
</div>

## 旧日志删除

Milvus 会自动删除那些已经应用到系统的日志。