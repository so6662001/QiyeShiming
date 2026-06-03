# 钢贸提货单实名签收系统 · PRD 汇总与接口字段表

> 本文为产品需求汇总（PRD），整合前述各文档要点，并给出核心接口的出入参字段表，供研发对接。
> 字段类型说明：`S`=字符串，`N`=数值，`I`=整数，`B`=布尔，`T`=时间(ISO8601)，`E`=枚举，`A`=数组，`O`=对象。

---

## 1. 产品概述

| 项 | 内容 |
|---|---|
| 产品名 | 钢贸提货单实名签收系统 |
| 目标用户 | 钢贸企业（卖方）及其客户（买方企业、经办人、驾驶员） |
| 核心价值 | 极简实名 + 提货码授权 + 出库即确认 + 锁价留货 + 全程电子签名存证 |
| 端 | 经办人移动端、仓管核验端、业务员/平台后台 |
| 关键约束 | 实名一次性成本 ≤ 0.5 元/人；客户端核心操作 ≤ 2 步；法律可出证 |

---

## 2. 角色与核心用例

| 角色 | 核心用例 |
|---|---|
| 业务员 | 开户、签服务协议、代录企业档案、生成留货单/结算单、出证 |
| 经办人 | 个人实名、确认留货单、发起提货指令、查看回执、核对结算/提异议 |
| 驾驶员 | 凭提货码到仓提货 |
| 仓管 | 核验提货码+车牌、过磅、出库放行 |

---

## 3. 功能需求清单（FR）

| 编号 | 模块 | 需求 | 优先级 |
|---|---|---|---|
| FR-01 | 开户 | 业务员录入企业档案，发起服务协议签署（法人意愿认证/电子章） | P0 |
| FR-02 | 实名 | 个人实名（身份证OCR+三要素/人脸），签发数字证书，结果复用 | P0 |
| FR-03 | 留货单 | 生成留货单（锁价/数量/单价/定金/有效期），经办人在线确认即签署存证 | P0 |
| FR-04 | 留货逾期 | 到期提醒；调价/抵扣定金/解除三分支处理，通知送达+存证 | P1 |
| FR-05 | 提货指令 | 经办人发起「提货指令暨收货确认」，生成一次性提货码 | P0 |
| FR-06 | 提货码 | 绑定车牌/数量/时段，动态刷新，撤销，防套用 | P0 |
| FR-07 | 仓库核验 | 提货码+车牌核验，过磅录入，出库放行+出库时间戳 | P0 |
| FR-08 | 出库确认 | 驶离即交付确认，归集现场证据 | P0 |
| FR-09 | 结算 | 生成结算单，多渠道送达可举证，异议期自动确认 | P0 |
| FR-10 | 异议 | 买方提异议（暂停计时）→ 人工对账 → 重新确认 | P1 |
| FR-11 | 风控 | 按货值/设备/经办人/车牌分级触发短信或人脸 | P1 |
| FR-12 | 存证出证 | 全证据链聚合、哈希上链、一键出具存证报告 | P0 |
| FR-13 | 通知送达 | 站内信/短信/邮件多渠道，记录送达与已读 | P0 |

---

## 4. 非功能需求（NFR）

| 编号 | 指标 | 目标 |
|---|---|---|
| NFR-01 | 实名成功率 | ≥ 98% |
| NFR-02 | 提货码核验响应 | ≤ 1s |
| NFR-03 | 出库到存证完成 | ≤ 30s（异步） |
| NFR-04 | 系统可用性 | ≥ 99.9% |
| NFR-05 | 证据可验证性 | 100% 可重算哈希比对 |
| NFR-06 | 敏感信息 | 加密存储，人脸不落地/脱敏 |

---

## 5. 核心接口出入参字段表

> 通用返回包：`{ code:I, message:S, data:O }`，下表仅列 `data` 及业务入参。所有写接口需带鉴权头 `Authorization`。

### 5.1 个人实名认证 `POST /realname/verify`
**入参**
| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| operatorId | S | 是 | 经办人ID |
| name | S | 是 | 姓名 |
| idCard | S | 是 | 身份证号（加密传输） |
| mobile | S | 是 | 手机号 |
| method | E | 是 | `two`二要素 / `carrier3`运营商三要素 / `face`人脸 |
| faceToken | S | 否 | 人脸核身令牌（method=face 时） |

**出参**
| 字段 | 类型 | 说明 |
|---|---|---|
| verified | B | 是否通过 |
| certId | S | 签发的数字证书ID |
| realnameRecordId | S | 实名记录ID（已存证） |
| evidenceHash | S | 实名记录哈希 |

### 5.2 留货单确认 `POST /holdnote/{id}/confirm`
**入参**
| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| operatorId | S | 是 | 经办人ID |
| willAuth | E | 是 | 意愿认证方式 `sms`/`face` |
| authCode | S | 是 | 短信验证码或人脸令牌 |
| agreementVersion | S | 是 | 适用服务协议版本 |

**出参**
| 字段 | 类型 | 说明 |
|---|---|---|
| holdNoteId | S | 留货单ID |
| status | E | `signed` 已签署锁价 |
| lockedUnitPrice | N | 锁定单价 |
| validUntil | T | 提货有效期 |
| signEvidenceId | S | 签署存证ID |
| timestamp | T | 可信时间戳 |
| chainTx | S | 上链交易号 |

### 5.3 发起提货指令（暨收货确认） `POST /pickup/order`
**入参**
| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| operatorId | S | 是 | 经办人ID |
| holdNoteId | S | 否 | 关联留货单（如有） |
| warehouseId | S | 是 | 提货仓库 |
| goodsSpec | S | 是 | 品名/规格 |
| authQty | N | 是 | 授权提货数量上限 |
| plateNo | S | 是 | 绑定车牌号 |
| driverName | S | 否 | 驾驶员姓名（大额必填） |
| validFrom | T | 是 | 有效期起 |
| validTo | T | 是 | 有效期止 |
| willAuth | E | 是 | `sms`/`face` |
| authCode | S | 是 | 验证码/人脸令牌 |

**出参**
| 字段 | 类型 | 说明 |
|---|---|---|
| pickupOrderId | S | 提货单ID |
| pickupCode | S | 提货码（短码） |
| qrToken | S | 二维码令牌（动态刷新） |
| status | E | `issued` 已下指令 |
| signEvidenceId | S | 指令签署存证ID |
| timestamp | T | 可信时间戳 |

### 5.4 仓库核验提货码 `POST /warehouse/verify-code`
**入参**
| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| pickupCode | S | 是 | 提货码或二维码令牌 |
| plateNoDetected | S | 是 | 现场车牌识别结果 |
| warehouseId | S | 是 | 仓库 |
| operatorStaffId | S | 是 | 核验仓管ID |

**出参**
| 字段 | 类型 | 说明 |
|---|---|---|
| valid | B | 是否有效 |
| plateMatch | B | 车牌是否一致 |
| authQty | N | 授权数量上限 |
| enterpriseName | S | 买方企业 |
| goodsSpec | S | 品名规格 |
| reject | E | 拒绝原因 `expired`/`plate_mismatch`/`revoked`/`used`/null |

### 5.5 过磅录入 `POST /warehouse/weigh`
**入参**：`pickupOrderId:S`、`gross:N`毛重、`tare:N`皮重、`weighTime:T`、`photoUrl:S`
**出参**：`net:N`净重、`weighRecordId:S`、`overAuth:B`(是否超授权)

### 5.6 出库放行 `POST /warehouse/outbound`
**入参**：`pickupOrderId:S`、`outboundNo:S`、`gateCaptureUrl:S`
**出参**
| 字段 | 类型 | 说明 |
|---|---|---|
| outboundId | S | 出库记录ID |
| outboundTime | T | 出库时间戳（交付确认时点） |
| deliveryConfirmed | B | 出库即确认=true |
| evidenceId | S | 出库证据存证ID |

### 5.7 结算单送达 `POST /settlement/deliver`
**入参**：`settlementId:S`、`channels:A`(`["app","sms","email"]`)、`objectionDays:I`
**出参**
| 字段 | 类型 | 说明 |
|---|---|---|
| delivered | B | 是否送达成功 |
| deliverEvidence | A | 各渠道送达记录（含时间、回执） |
| objectionDeadline | T | 异议截止时间 |
| status | E | `in_objection` 异议期中 |

### 5.8 提出异议 `POST /settlement/{id}/objection`
**入参**：`operatorId:S`、`type:E`(`qty`/`spec`/`price`/`other`)、`claimQty:N`、`reason:S`、`attachments:A`
**出参**：`status:E`(`under_review`人工对账)、`paused:B`(计时已暂停)、`objectionId:S`

### 5.9 留货单逾期处理 `POST /holdnote/{id}/overdue`
**入参**
| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| action | E | 是 | `repricing`调价 / `forfeit`抵扣定金解除 / `terminate`解除 |
| newUnitPrice | N | 否 | action=repricing 时 |
| newValidUntil | T | 否 | 顺延有效期 |
| notifyChannels | A | 是 | 通知渠道 |

**出参**：`status:E`(`signed_new`/`forfeited`/`terminated`/`await_buyer_confirm`)、`notifyEvidence:A`、`evidenceId:S`

### 5.10 出具存证报告 `POST /evidence/report/{pickupOrderId}`
**出参**
| 字段 | 类型 | 说明 |
|---|---|---|
| reportUrl | S | 存证报告下载地址 |
| items | A | 证据项清单（类型/哈希/时间戳/上链号） |
| verifiable | B | 是否全部可重算验证 |

---

## 6. 关键枚举字典

| 枚举 | 取值 |
|---|---|
| 经办人账号状态 | 待实名 / 已实名授权 / 已停用 / 已失效 |
| 留货单状态 | 待确认 / 已签署 / 部分提货 / 已提完 / 逾期待处理 / 已扣定金解除 / 已解除 / 已取消 |
| 提货单状态 | 已下指令 / 已核验 / 已出库 / 待结算确认 / 已撤销 / 已过期 |
| 结算单状态 | 待送达 / 异议期中 / 异议处理 / 已确认 |
| 意愿认证方式 | sms / face |
| 实名方式 | two / carrier3 / face |

---

## 7. 验收要点（与法律时点对应）

1. 留货单确认产生经办人签名+时间戳+存证，锁价可追溯。
2. 提货指令暨收货确认一次签署，含出库即确认与异议期认可语。
3. 出库放行返回 `outboundTime` 作为交付确认时点，现场证据齐全。
4. 结算单送达返回可举证 `deliverEvidence`，异议期到期自动确认。
5. 任一 `pickupOrderId` 可一键出具可验证存证报告。

> 详细业务逻辑、流程图、状态机、合规依据见 `01`~`06` 文档；本 PRD 为对接索引与字段契约。
