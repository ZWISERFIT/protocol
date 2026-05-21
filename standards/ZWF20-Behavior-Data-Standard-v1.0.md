# ZWF-20 行为数据资产标准 v1.0

> **状态:** ACTIVE | **发布日期:** 2026-05-21
> **性质:** 开源标准 | **许可:** 标准文本 CC BY 4.0 | 数据格式 PDL v1.0
> **对标:** Ocean Protocol / Evidation / WHOOP / Flatiron Health | **创新:** 首个实体健身行为数据 RWA 标准

---

## 概述

ZWF-20 是全球首个面向实体健身场景的行为数据资产标准。它定义了：
1. **采集什么** — 行为数据字段与格式
2. **如何归类** — 事件类型与质量等级
3. **隐私边界** — 什么永不离开门店
4. **如何对接** — API 与 JSON Schema

---

## 一、数据模型

### 1.1 核心事件类型

| 事件类型 | event_type | 描述 | 触发条件 |
|---------|-----------|------|---------|
| 入场签到 | `checkin` | 会员进入门店 | 门禁验证通过 |
| 离场签退 | `checkout` | 会员离开门店 | 门禁验证通过 |
| 训练开始 | `workout_start` | 开始使用器械/区域 | IoT 传感器检测 |
| 训练结束 | `workout_end` | 结束训练 | 器械释放 / 区域离开 |
| 身体指标 | `body_metric` | InBody 测量记录 | InBody 设备完成测量 |
| 设备使用 | `equipment_use` | 单次器械使用记录 | 器械传感器 |

### 1.2 数据字段定义

```
ZWF-20 Behavior Event
├── event_id            UUID v4
├── member_did          did:zwf:{hash}
├── event_type          enum(checkin|checkout|workout_start|workout_end|body_metric|equipment_use)
├── timestamp           Unix ms
├── location_id         ZWF门店编码
├── duration_seconds    (workout/equipment 类型必填)
├── intensity           enum(low|medium|high)  # 基于心率/运动量推算
├── body_metrics        (body_metric 类型必填)
│   ├── weight_kg
│   ├── body_fat_pct
│   ├── muscle_mass_kg
│   └── bmi
├── equipment           (equipment_use 类型必填)
│   ├── equipment_id
│   ├── equipment_type  enum(treadmill|bike|rower|free_weight|machine|cable)
│   └── settings        器械参数（重量/速度/坡度）
└── privacy_level       enum(L0|L1|L2|L3|L4)  # 默认 L2
```

### 1.3 JSON Schema（示例）

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://zwf.dev/schemas/zwf20-event-v1.json",
  "title": "ZWF-20 Behavior Data Event",
  "type": "object",
  "required": ["event_id", "member_did", "event_type", "timestamp"],
  "properties": {
    "event_id": {"type": "string", "format": "uuid"},
    "member_did": {"type": "string", "pattern": "^did:zwf:[a-f0-9]{64}$"},
    "event_type": {
      "type": "string",
      "enum": ["checkin", "checkout", "workout_start", "workout_end", "body_metric", "equipment_use"]
    },
    "timestamp": {"type": "integer", "minimum": 1577836800000},
    "location_id": {"type": "string"},
    "duration_seconds": {"type": "integer", "minimum": 0},
    "intensity": {"type": "string", "enum": ["low", "medium", "high"]},
    "privacy_level": {"type": "string", "enum": ["L0", "L1", "L2", "L3", "L4"]}
  },
  "additionalProperties": false
}
```

> 完整 JSON Schema 文件参见 `schemas/zwf20-event-v1.json`

---

## 二、脱敏规范（脱敏铁律）

### 2.1 数据出境规则

```
店内原始数据 ──→ 边缘计算节点 ──→ 脱敏行为事件 ──→ 受托人保管库
                    │
                    ├── 原始视频 ──→ 处理后删除（30秒内）
                    ├── 人脸向量 ──→ 匹配后删除
                    ├── PII  ──→ 替换为 DID
                    └── 精确GPS ──→ 替换为门店 location_id
```

### 2.2 隐私边界矩阵

| 数据项 | 门店本地 | 受托人保管库 | 买家可查询 | 链上公开 | Git 仓库 |
|--------|:-------:|:----------:|:--------:|:------:|:------:|
| 原始视频流 | ✅ 临时 | ❌ | ❌ | ❌ | ❌ |
| 人脸特征向量 | ✅ 临时 | ❌ | ❌ | ❌ | ❌ |
| 会员姓名/手机 | ✅ | ❌ | ❌ | ❌ | ❌ |
| DID | ✅ | ✅ | L2+ | ✅ | ❌ |
| 脱敏行为事件 | N/A | ✅ | L2+ | ❌ | ❌ |
| 聚合统计 | N/A | ✅ | L0+ | ✅ | ✅ |
| 哈希存证 | N/A | ✅ | ✅ | ✅ | ✅ |
| DID公钥 | N/A | N/A | N/A | ✅ | ✅ |
| 授权记录(脱敏) | N/A | N/A | N/A | ✅ | ✅ |

---

## 三、数据质量等级

| 等级 | 验证方式 | 最低训练频次 | 最低入驻时长 | 使用场景 |
|:----:|---------|:----------:|:----------:|---------|
| **QA** | 门禁 + 人脸双重验证 | 60+ 次 | 6+ 月 | 精算级分析、临床研究 |
| **QB** | 门禁单验证 | 30+ 次 | 3+ 月 | 商业分析、产品开发 |
| **QC** | 门禁单验证 | <30 次 | 1+ 月 | 趋势分析、市场调研 |
| **QD** | 门禁单验证 | 不限 | 不限 | 人口统计学分析 |

---

## 四、防伪机制

ZWF-20 数据具有天然防伪属性，区别于自报数据：

| 防伪层级 | 机制 | 说明 |
|:------:|------|------|
| L1 | **门禁验证** | 物理到场 = 不可伪造。你不能远程"刷"入场记录。 |
| L2 | **人脸匹配** | 门禁人脸 × 会员档案 = 确认为本人（非代打卡） |
| L3 | **传感器交叉验证** | 门禁时间 × 器械使用时间 × 离场时间 = 时间一致性检查 |
| L4 | **行为模式分析** | 长期行为模式异常检测 → 标记可疑数据 |

**区别于 WHOOP/Oura/Apple Watch：** 可穿戴设备无法证明"是谁在戴"。门禁×人脸 = 本人确认。

---

## 五、API 规范

### 5.1 数据查询接口

```
GET /api/v1/data/query
Authorization: Bearer {license_token}
X-MPC-Signature: {mpc_signature}

Parameters:
  license_tier   L0|L1|L2|L3|L4
  event_types    comma-separated event types
  date_from      ISO 8601
  date_to        ISO 8601
  quality_level  QA|QB|QC|QD
  limit          max records per page

Response:
  {
    "events": [...],
    "page_token": "...",
    "total_count": 12345,
    "quality_summary": {"QA": 8000, "QB": 3000, ...}
  }
```

### 5.2 会员授权管理接口

```
POST /api/v1/auth/grant
  member_did, licensee_did, license_tier, purpose_hash
  → requires MPC signature from member

POST /api/v1/auth/revoke
  grant_id
  → requires MPC signature from member
```

---

## 六、贡献指南

ZWF-20 标准在 GitHub 上开放演进：

1. **新增事件类型:** 提交 RFC → Nova 评审 → Stella 合规审查 → 合并
2. **修改字段定义:** 需向后兼容。Breaking changes 仅在新主版本号中允许。
3. **新增隐私类别:** 需证明不违反脱敏铁律。

参见 `CONTRIBUTING.md` 了解完整流程。

---

## 七、版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-05-21 | 初始版本。6 种事件类型 + 4 级质量 + 4 层防伪 + API 规范 |

---

*💎 ZWF-20 v1.0 | ZWISERFIT Behavior Data Asset Standard*
*PDL v1.0 Licensed | Open for RFC contributions*
