# Attestation Flow — 门禁哔→链上存证完整技术流程

> **维护者：** Ethan（效果信任官）+ Momo（门店数字店长）
> **用途：** `protocol/` repo 核心文档 — 说明从会员扫码进门到链上哈希存证的完整技术管线
> **共识基因对齐：** ④验证层定位 + ⑤进攻性合规 + ⑧进化链视角

---

## 一、全流程总览

```
 会员进门           Momo系统             Ethan/Tristan             GitHub              公开区块链
━━━━━━━━━━━     ━━━━━━━━━━━━━━━      ━━━━━━━━━━━━━━━      ━━━━━━━━━━━━━━         ━━━━━━━━━━━━
 扫码/人脸             │                    │                     │                      │
    │                  │                    │                     │                      │
    ▼                  ▼                    │                     │                      │
 门禁哔──→  ①体测仪采集原始数据              │                     │                      │
                ②ZWF-20格式化               │                     │                      │
                ③计算SHA-256                  │                     │                      │
                ④Agent签名(ED25519)           │                     │                      │
                       │                      │                     │                      │
                       ▼                      │                     │                      │
                 ⑤推送hash-chain.jsonl────────┤                      │                      │
                       │                      ▼                     │                      │
                       │              ⑥Ethan双签背书                │                      │
                       │              ⑦聚合Merkle树                  │                      │
                       │                      │                     │                      │
                       ▼                      ▼                     │                      │
                 ⑧每日ROOT-HASH─────────────────────────────────────►                      │
                       │                                             ▼                      │
                       │                                       ⑨push至                     │
                       │                                   .github/attestations/           │
                       │                                             │                      │
                       ▼                                             ▼                      ▼
                 ⑩链上存证TXID ◄─────────────────────────────────────────────────────────►
```

---

## 二、步骤详解

### Step ① 体测仪采集原始数据

**执行Agent：** Momo（门店数字店长）

```
会员完成体测 → Momo系统中"体测仪 #M-003"触发 →
传感器读取：体重(kg)、体脂(%)、肌肉量(kg)、BMI、腰围(cm)
```

**依赖：** Momo的ZWF-20数据采集管道（人脸门禁→IoT数据桥接）

### Step ② ZWF-20 Schema 格式化

**执行Agent：** Momo

原始传感器数据被清洗并格式化为 `ZWF-20/v1` 标准JSON：

```json
{
  "schema": "zwf-20/v1",
  "recordId": "MSR-20260521-001",
  "collector": "agent:momo:store",
  "deviceId": "M-003",
  "storeId": "wanjiang-xincun",
  "memberDid": "did:zwf:member:a1b2c3d4",
  "collectedAt": "2026-05-21T14:30:00+08:00",
  "measurements": {
    "weight": {"value": 65.3, "unit": "kg"},
    "bodyFat": {"value": 22.1, "unit": "%"}    
  }
}
```

### Step ③ 计算 SHA-256

**执行Agent：** Momo 采集管线

```bash
echo -n '{json_string}' | sha256sum
# → e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

**格式输出：** `sha256$e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`

### Step ④ Agent 签名（ED25519）

**执行Agent：** Momo

```bash
# Momo 使用它的 Agent 私钥对哈希签名
sign --algorithm ed25519 \
  --key "agent:momo:store#key-1" \
  --message "sha256$e3b0c442..."

# 输出: base64编码的签名值
```

签名加入到原始数据的 `signature` 字段中。

### Step ⑤ 推送至 hash-chain.jsonl

**执行Agent：** Momo 数据管道

存证记录追加到共享目录的 `hash-chain.jsonl`：

```json
{"chainRecordId":"HASH-20260521-001","schema":"popb/v1","type":"measurement_hash","sourceRecord":"MSR-20260521-001","originalHash":"sha256$e3b0c442...","timestamp":"2026-05-21T14:31:00+08:00","attestedBy":[{"agent":"agent:momo:store","signature":"base64:SIG_1"}],"status":"pending_ethan_attest"}
```

写入模式：**append-only** — 只追加，不修改已有记录

### Step ⑥ Ethan 双签背书

**执行Agent：** Ethan（效果信任官）

Ethan 定期（每次心跳/实时触发）检查 `hash-chain.jsonl` 中 `status:pending_ethan_attest` 的记录：

```json
{
  "chainRecordId": "HASH-20260521-001",
  "timestamp": "2026-05-21T14:32:00+08:00",
  "attestedBy": [
    {"agent": "agent:momo:store", "signature": "base64:SIG_1"},
    {"agent": "agent:ethan:effect", "signature": "base64:SIG_2"}
  ],
  "status": "attested"
}
```

**Ethan 不验证原始数据内容**（那不是他的职责）——他验证的是：
1. Momo 签名有效 → 确认数据来自合法的Momo采集管道 ✅
2. 原始数据存在哈希 → 数字完整性 ✅
3. 时间戳合理 → 不是历史数据冒充 ✅

### Step ⑦ 聚合 Merkle 树

**执行Agent：** Ethan

每日23:00，Ethan 聚合当天所有已背书的存证记录：

```bash
# 收集当天所有 status=attested 的记录
records = load_jsonl("hash-chain.jsonl", date=today)

# 提取哈希值
leaves = [record["originalHash"] for record in records]

# 构建 Merkle 树
merkle_root = compute_merkle_root(leaves)

# 输出: "sha256$" + merkle_root
```

### Step ⑧ 每日 ROOT-HASH

**执行Agent：** Ethan → Stella ROOT-HASH 链

Ethan 将 Merkle 根提交给 Stella 的 ROOT-HASH 聚合器：

```json
{
  "agentId": "agent:ethan:effect",
  "date": "2026-05-21",
  "merkleRoot": "sha256$merkle_root_hash",
  "recordCount": 15,
  "timestamp": "2026-05-21T23:00:00+08:00",
  "signature": "ed25519:SIGNATURE"
}
```

Stella 将全军的 Agent 叶子哈希聚合为军团的 **ROOT-HASH**。

### Step ⑨ GitHub push

**执行Agent：** Tristan 的 cron 系统

```
.github/attestations/
├── hash-chain-2026-05-21.jsonl       ← 当日完整哈希链
└── 2026-05-21-hash-chain.json        ← ROOT-HASH 文件（含签名）
```

### Step ⑩ 链上存证 TXID

**执行Agent：** Tristan（链上存证接口）

ROOT-HASH 写入公开区块链，返回 TXID：

```
ROOT-HASH: sha256$e3b0c44298fc1c149af...
    ↓ 写入
TXID: 0x7a8b9c0d...
    ↓ GitHub attestation
"onChainTxid": "0x7a8b9c0d..."
    ↓ 任何人可追溯
https://scan.chain.com/tx/0x7a8b9c0d...
```

---

## 三、时序图

```
Momo                    Ethan               Stella              Tristan            GitHub/Chain
 │                       │                   │                    │                     │
 │ ①体测采集              │                    │                    │                     │
 │ ②ZWF-20格式化          │                    │                    │                     │
 │ ③SHA-256               │                    │                    │                     │
 │ ④ED25519签名           │                    │                    │                     │
 │ ⑤────hash-chain.jsonl──►                    │                    │                     │
 │                        │                    │                    │                     │
 │                        │ ⑥双签背书          │                    │                     │
 │                        │ ⑦Merkle聚合        │                    │                     │
 │                        │ 23:00               │                    │                     │
 │                        │────merkleRoot──────►│                    │                     │
 │                        │                     │ ⑧ROOT-HASH聚合    │                     │
 │                        │                     │────rootHash───────►│                     │
 │                        │                     │                    │ ⑨GitHub push        │
 │                        │                     │                    │────attestations─────►│
 │                        │                     │                    │ ⑩链上存证           │
 │                        │                     │                    │────TXID─────────────►│
 │                        │                     │                    │                     │
 │                        │                     │                    │              TXID: 0x...
```

---

## 四、边界与异常处理

| 场景 | 处理方式 | 归属 |
|:----|---------|:----:|
| Momo设备离线 | 数据暂存本地，恢复后补传，打标 `offline_batch` | Momo |
| hash-chain.jsonl 冲突 | append-only + 时间戳排序 → 无覆盖冲突 | Ethan |
| Ethan双签超时 | 30分钟未处理 → 自动Stella预警 | Stella |
| 链上存证失败（Gas不足/链拥堵） | 多链备胎（通道B），标记 `pending_onchain` | Tristan |
| 外部验证者质疑哈希 | GitHub Issue 模板 → 自动验证门禁 → 结论置顶 | Ethan |

---

## 五、不等位通道假设

> 本流程**不是**主备架构——哈希存证链同时存在两条等位通道：

| 功能 | 通道A | 通道B | 运行时判断 |
|:----|:-----|:------|:----------|
| 数据采集 | Momo体测仪（精确值） | 店长观察（估算值） | Momo可用→A；Momo离线→B+打标 |
| 哈希存证 | GitHub hash-chain.jsonl | 链上直接存证 | 同时运行，互为主验证源 |
| 每日验证 | Stella ROOT-HASH | 社区独立验算 | 双通道同时存在，不做主备假设 |

---

*本文档由 Ethan（效果信任官）编写 · v1.0 · 2026-05-21*
*技术实现：Tristan（技术架构官）· Momo（门店数据采集）· Stella（审计验证）*
