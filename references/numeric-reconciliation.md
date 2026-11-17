# 数字对账：报告里的数自己会打架

判断项大多靠数字撑着。把这些数字互相碰一遍，比找外部证据更快。

## 1. 明细累加 vs 报告汇总

把报告自己给的明细全部加总，和它给的汇总比：

- 明细之和 **大于** 汇总 → 口径不同（或汇总是拍的）
- 明细之和 **小于** 汇总 → 漏项
- 差出来的部分必须让报告解释；解释不了，汇总就是编的

实测过的典型：九个封禁 IP 的请求数相加 = 1192，报告称「可疑请求 656」、总请求 1883——三个数互相不兼容，等于自认六成流量都可疑。

## 2. 计数器 vs 物理上限

- `kernel.pid_max` 默认 4194304。当前 PID（`systemctl show -p MainPID <unit>` 或 `cat /proc/sys/kernel/ns_last_pid`）若没绕回，**开机以来 fork 的进程总数上限就是这个 PID**
- 「服务重启了 N 次」要求每次至少 fork 一个进程：`N × 1 ≤ 当前 PID`，否则数字不成立
- 两个相邻服务（如 nginx 与应用）的 PID 差，就是它们启动之间系统 fork 过的进程数——只差几千，说明系统很安静，与「疯狂重启」矛盾
- systemd 默认 `StartLimitBurst=5` / `StartLimitIntervalSec=10s`，连挂五次就锁 failed。能无限重启说明限速被显式关掉，报告必须交代
- **交叉核**：`systemctl show -p NRestarts <unit>` 是 systemd 计数器（daemon-reload／重启会清零），`journalctl -u <unit> | grep -c 'Started'` 才是真启动条数。两者差一个数量级，说明报告读错了字段

## 3. 频率换算

「高频扫描」这类判断，把次数除以时间窗：

- 277 次 / 12 小时 ≈ 23 次/小时——这是正常监控或爬虫，不是高频
- 阈值也反算一遍：50 次/5 分钟 = 10 次/分钟，一个带图文的页面加载就可能超，会误伤真人

## 4. 自称的数字

- 报告自称「约 8KB」，实际 12,670 字节 = 12.4KB
- 磁盘写「58%」，原始是 24G/40G = 60%
- 单独看不重要，但它是**「这个字段不是实测」**的证据，用来给整份报告的可信度定级

## 5. nginx 日志字段偏移（取错一列，全盘结论都错）

combined 格式的 `[24/Oct/2026:10:56:12 +0800]` 被空格拆成两列，所以：

- `$4` = `[24/Oct/2026:10:56:12`（日期时间），`$5` = `+0800]`
- **状态码是 `$9`，不是 `$7`；响应字节是 `$10`，不是 `$8`**
- URI 是 `$7`；UA 用 `awk -F'"' '{print $6}'`

按 common 格式的偏移取值，会把 URI 当状态码，统计全错而且看不出来。

## 6. 只读核验命令

```bash
# 重启真伪
systemctl show <unit> -p NRestarts -p MainPID -p Restart -p StartLimitBurst
cat /proc/sys/kernel/pid_max; cat /proc/sys/kernel/ns_last_pid
journalctl -u <unit> --since today --no-pager | grep -c 'Started'

# 请求口径（combined 格式）
NG=/var/log/nginx/access.log; D=$(date +%d/%b/%Y)
grep -c "\[$D:" "$NG"                                      # 今日总请求
awk -v t="[$D:" 'index($0,t) && $9+0>=400' "$NG" | wc -l  # 4xx+5xx
awk -v t="[$D:" 'index($0,t){b[$1]+=$10} END{for(i in b) printf "%14d %s\n", b[i], i}' "$NG" | sort -rn | head -15

# 按分钟切片找流量尖峰
awk -v t="[$D:" 'index($0,t){split($4,a,":"); c[a[2]":"a[3]]++} END{for(k in c) print k, c[k]}' "$NG" | sort
```

统计段改完先拿一份**合成日志**跑一遍再交，口径错了比不给更糟。
