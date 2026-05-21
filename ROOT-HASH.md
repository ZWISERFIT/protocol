# ROOT-HASH 格式规范 v1.0

> **维护者：** Stella（品类合规架构师）+ Ethan（效果信任官）
> **更新频率：** 每日 23:59 CST 自动生成
> **用途：** 全军团每日不可伪造审计证据根哈希
> **发布位置：** `.github/attestations/2026-05-21-hash-chain.json`
> **共识基因对齐：** ⑤进攻性合规 + ⑥大脑外化

---

## 一、ROOT-HASH 定义

ROOT-HASH 是全军团每日所有可审计产出的 Merkle 根哈希。每天 23:59 CST 自动生成，包含：

1. 当日所有 Agent 的审计报告哈希
2. 当日所有链上存证 TXID
3. 当日交叉验证事件证据
4. 前一日 ROOT-HASH（形成不可逆链）

## 二、数据结构

### 根节点格式

```json
{
  "rootHash": "sha256$e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "date": "2026-05-21",
  "timestamp": "2026-05-21T23:59:00+08:00",
  "previousRootHash": "sha256$a7ffc6f8bf1ed76651c14756f5c0c8f1d4c8c8c8c8c8c8c8c8c8c8c8c8c8c8c8c8",
  "agents": {
    "stella": {
      "reportHash": "sha256$...",
      "reportPath": ".github/attestations/2026-05-21-tonight.md"
    },
    "ethan": {
      "hashChainFile": ".github/attestations/2026-05-21-hash-chain.json",
      "hashChainRoot": "sha256$..."
    },
    "tristan": {
      "preflightHash": "sha256$...",
      "preflightPath": "preflight/2026-05-21.log"
    }
  },
  "crossValidationEvents": [
    {
      "type": "SOS_VERIFY",
      "executor": "Stella",
      "target": "Tristan",
      "finding": "SOS同步正常（非Syncthing盲区）",
      "evidenceHash": "sha256$..."
    }
  ],
  "onChainTxid": "0x7a8b9c0d...",
  "generator": "Stella ROOT-HASH cron v1.0",
  "signature": "pgp:SIGNATURE_ARMOR"
}
```

### 叶子节点格式（Agent 级别）

每个 Agent 的独立审计产出生成自己的叶子哈希：

```json
{
  "agentId": "agent:ethan:effect",
  "date": "2026-05-21",
  "deliverableHashes": [
    {
      "name": "PoPB协议文档",
      "path": "docs/01-pb-protocol.md",
      "hash": "sha256$abc123...",
      "generatedAt": "2026-05-21T17:45:00+08:00"
    },
    {
      "name": "哈希存证工作流",
      "path": ".github/workflows/hash-attest.yml",
      "hash": "sha256$def456...",
      "generatedAt": "2026-05-21T17:50:00+08:00"
    }
  ],
  "crossValidationStatus": "✅ Stella审计通过",
  "signature": "ed25519:SIGNATURE"
}
```

## 三、Merkle 树结构

```
                       ROOT-HASH
                      /         \
                     /           \
          Stella-leaf          Ethan-leaf      Tristan-leaf    ...
        (audit报告哈希)    (存证链根哈希)    (preflight哈希)
              |                  |                |
        2026-05-21         2026-05-21       2026-05-21
        tonight.md         hash-chain        preflight.log
```

## 四、生成流程

```
00:00 — 开始新一天 ROOT-HASH 链
     ↓
各 Agent 产生当日产出
     ↓
23:00 — Stella 发起 ROOT-HASH 聚合请求
     ↓
23:30 — 各 Agent 提交叶子哈希 + 签名
     ↓
23:45 — Stella 计算 Merkle 根 → ROOT-HASH
     ↓
23:50 — ROOT-HASH 写入链上存证（TXID）
     ↓
23:59 — GitHub push → .github/attestations/YYYY-MM-DD-hash-chain.json
     ↓
00:00 — ROOT-HASH 存入 memory/ 作为次日 previousRootHash
```

## 五、Ethan 负责的哈希子链

Ethan（效果信任官）每日需提交：

| # | 输入 | 哈希内容 | 生成时机 |
|:-:|------|---------|:-------:|
| 1 | 所有已交付的会员报告 | 报告文件 SHA-256 列表的 Merkle 根 | 23:00 |
| 2 | 已发送给 Baron 的脱敏案例 | 案例文件 SHA-256 列表 | 23:00 |
| 3 | 当日 PoPB 存证记录 | hash-chain.jsonl 当日所有记录的 Merkle 根 | 23:00 |
| 4 | Consensus-Check 自检记录 | 7项自检结果的哈希 | 23:00 |

## 六、安全约束

1. 任何 Agent 不得跳过自己当日的哈希提交——缺失则 ROOT-HASH 不完整
2. ROOT-HASH 的 TXID 必须写入链上——仅存 GitHub 等同于未存证
3. 签名密钥每日轮转（Stella 签发当日密钥，次日销毁）
4. ROOT-HASH 文件本身是公开可验证的——任何第三方可用相同输入计算并比对

---

*本规范由Stella（安全审计官）+ Ethan（效果信任官）联合定义 · v1.0 · 2026-05-21*
*对齐共识基因⑤进攻性合规：哈希链=进攻性合规的证据基础设施*
