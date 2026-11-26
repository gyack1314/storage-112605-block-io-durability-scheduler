# 多租户块盘的期限调度与落盘屏障

设备扇区用整数编号。长度以扇区计。设备回执携带请求 id、动作类型、分段起点与长度（若适用）、以及该动作的 attempt。不同次尝试的回执可以独立到达。驱动可报告在写缓存中已接收，只有 flush 回执证明持久。

## 输入消息

每个 JSONL 文件是一段独立输入记录。每行包括整数秒点 at、消息名称 kind 与对象 record。at 代表中间件收到消息的时刻，同一秒内按行序提交。业务时间和对应回执引用写在 record 中。

- **device**　depth 是队深，max_span 是单条数据命令承载的扇区数。
- **tenant**　name、share 为该租户的最低相对服务份额。
- **io**　tenant/id、type（read 或 write）、start/count、deadline。写记录带 text，每个扇区一个字符。
- **flush**　tenant/id 划定本次持久范围；重用同一 id 重试，不改变它原来覆盖的写集合。
- **pump**　请求提交下一批可执行设备命令。
- **done**　id、type、attempt、result，数据命令附 start/count；成功读附 text。
- **cancel**　尝试取消 id，已提交时等设备事实。
- **restart**　主机调度器重启，设备请求事实仍可迟到。
- **inspect**　查询 id 或全局队列。

## 示例

- late-fragment-before-barrier.jsonl
- timeout-not-queue-release.jsonl

## 读取演练资料

运行环境为 Python 3.11，只使用标准库。在根目录运行以下命令可核对文件格式及读取接口。

    python3 scenario_reader.py examples
    python3 -m unittest discover -v

资料读取器会按文件统计各类参与方消息的数量。演练记录只描述各参与方交给系统的输入。
