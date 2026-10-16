# ACT 新增 SPC 新渠道需求评估报告

> 需求来源：ACT 需新增 SPC（Special 特殊渠道）新渠道支持；涉及 COT-ACT 城市总部 COT 覆盖门店数接口变更，并评估手机端（企微端）是否需对三个二级店型（出行 / 加油充电站 / 校园店）做相应调整，以支持其加入 ACT 档期。
> 本报告基于知识库既有事实，逐项标注【知识库实证】/【推断】/【待确认】，凡知识库未覆盖处显式标注缺口。

---

## 一、需求理解

1. **接口侧**：COT-ACT 城市总部「COT 覆盖门店数」接口新增 3 个字段，用于把「COT 覆盖门店数」按 SPC 渠道的 3 个新二级店型拆分：
   | 新增字段 | 中文名 | 对应二级店型 |
   |---|---|---|
   | `cot_store_travelling_num` | COT覆盖门店数-出行 | 出行（Travelling） |
   | `cot_store_powerstation_num` | COT覆盖门店数-加油/充电站 | 加油/充电站（PowerStation） |
   | `cot_store_school_num` | COT覆盖门店数-校园店 | 校园店（SchoolS） |

2. **手机端侧**：评估企微端是否需要针对「出行 / 加油充电站 / 校园店」三个店型做调整，使这些店型的门店能够加入 ACT 档期——推测落点为 **COT 拜访表单中 ACT 执行表单的显示**。

---

## 二、需求涉及的既有概念（知识库锚点）

### 2.1 SPC 渠道与三个二级店型【知识库实证】

SPC = Special 特殊渠道，为 **2026Q4 新增**渠道（`execution-king-kpi-and-dashboard.md:45`「SPC 渠道为 2026Q4 新增」）。其二级/三级店型 taxonomy（`channel.md:67-76`、`store-channel-taxonomy.md:40-42`）：

| 一级渠道 | 二级店型 | 三级店型（store_channel_code） |
|---|---|---|
| Special 特殊渠道 | **出行 Travelling** | Airport 飞机场售点 / TrainStation 火车站售点 / Freeway 高速服务区售点 / ScenicSpots 景区售点 |
| Special 特殊渠道 | **加油/充电站 PowerStation** | GasStation 加油站 / ChargingStation 充电站 |
| Special 特殊渠道 | **校园店 SchoolS** | SchoolSuper 校园内标准超市 / SchoolMini 校园内小型超市 / SchoolTT 校园内TT店 / SchoolService 校园供应 |

2026 渠道重构来源（`proj-store-definition.md:27`）：NC → 特殊渠道 + 种子渠道、Horeca → Special、**新增充电站 / 校园店等店型**；商圈环境升为店型判断首要字段。

### 2.2 城市总部 COT-ACT 接口现状【知识库实证 + 待确认】

知识库中城市总部下游接口共 3 个（`downstream-city-hq-2026.md`），其中「城市总部（活动）」接口即 **`chain_act`**（api_key `chain_act`），是城市总部数据给 ACT 的活动侧接口：

| 项 | 现状 |
|---|---|
| 网关/鉴权 | `GET apim[-qa].mars-ad.net/customer-locust/api`，`app_key=7ab147fc…` + `api_key=chain_act` |
| 处理引擎 | **D1（内置/数据框架）**，配置 `{"df_key":"di_otr_city_hq_act_view_dfk"}` |
| 数据源 | `customer_sequoia.di_otr_city_hq_act_view`（VIEW）→ 基表 `di_otr_city_hq_output`（**17,409 行**） |
| 文档字段数 | 21 |
| 返回字段（文档口径） | `code`/`name`/`level`/`chain_code1/2`/`chain_name1/2`/`parentid`/`katype`/`rtm_channel`… |
| 可用性 | ⚠️ QA 实测 **500**（OAuth 依赖不可达）；`app_definition.status` 全为 0 |

> **【待确认·关键缺口】**：知识库对 `chain_act` 仅记录 21 个字段，**未记录任何 `cot_store_*_num` 类「COT 覆盖门店数」细分字段**。需求引用的「COT-ACT 接口文档标绿字段」是**外部文档，尚未入库**——现有字段族（若已存在其他店型的 `cot_store_*_num`）及其命名/口径需要从该文档补齐后方可定稿。本报告后续对比以「接口需按店型新增 COT 覆盖门店数细分」为前提推断。

### 2.3 ACT 档期与门店快照【知识库实证】

- **档期**：`ActMasterDo` 含 `period_start`（活动开始档期）/`period_end`（活动结束档期），`activityType` 枚举含「全国档期活动 / 客户定制活动 / 区域档期活动 / 区域客户定制活动」（`campaign-exchange/data-model.md:63-94`）。
- **档期动作（ACT）**：拜访动作四类之一（`ge-106-a-visit-journey.html:543`），即业务员在拜访中执行 ACT 档期任务。
- **门店快照**：`act` 库 354 表含 `act_stores` / `cot_stores` / `act_cot_stores` / `cot_store_all` / `mdp_store_his`（staging 0 行、生产有数，见 `campaign-center-deep-dive.md:52`；`OQ-MK3`）。
- **门店范围圈选**：活动门店范围可用 57 个门店属性圈选，**已含 `channel_code/name`（一级）、`channel_level2_code/name`（二级）、`store_channel_code/name`（三级）**（`campaign-center-deep-dive.md:103`）→ 三个新店型在「数据层圈选维度」上已具备。
- **ACT 门店主数据接口**：门店中心 1.5（api_key `store_act`）join `act_store_consignee`，糖巧固定/非固定/不可拜访/线路外，38→41 字段（`downstream-store-master-2026.md:27`）。

### 2.4 相关既有业务口径【知识库实证】

- **SPC 校园高校拓店达成率**：`二级门店类型=校园店 且 商圈环境=校园-高校校园内`（`execution-king-kpi-and-dashboard.md:43`）。
- **加油站与出行不计算巧克力分销**：执行为王 SPC 达成率说明中「加油站与出行不计算巧克力分销」（`execution-king-kpi-and-dashboard.md:45`）→ 影响 ACT 档期内巧克力分销类执行项在这两类店型的口径。
- **完美门店特殊渠道报表路由**：`zo_special_detail` 含 加油站 / 高速服务区售点 / 校园店（3 类）/ 线下喜铺；而 2026 bySKU 报表路由「取消 O2O 前置仓、火车站售点、飞机场售点、连锁加油站、独立加油站」——即**火车站/飞机场售点、加油站已被从部分报表路由移出或归入特殊渠道报表**，店型在不同报表/执行域的可见性不一致，需在 ACT 档期圈选中显式对齐。

---

## 三、接口层面改动点 + 改动逻辑前后对比

### 3.1 改动逻辑

「COT 覆盖门店数」从**笼统总数**细化为**按 SPC 二级店型拆分的分项计数**，使城市总部级别的 ACT 档期能够按店型判断「该城市总部下 COT 覆盖了多少出行 / 加油充电站 / 校园店门店」，从而决定这三类店型能否/如何加入档期。

### 3.2 前后对比

| 维度 | 改前（现状） | 改后（目标） |
|---|---|---|
| 字段 | 城市总部基础属性（`code`/`name`/`level`/`katype`/`rtm_channel` 等） | 保留原有字段，**新增 3 个**分项覆盖数 |
| 覆盖门店数口径 | 未按新店型拆分（KB 无 `cot_store_*_num` 记录） | 按 3 个 SPC 二级店型拆分：出行 / 加油充电站 / 校园店 |
| 新字段 1 | — | `cot_store_travelling_num`（出行 = Airport+TrainStation+Freeway+ScenicSpots） |
| 新字段 2 | — | `cot_store_powerstation_num`（加油/充电站 = GasStation+ChargingStation） |
| 新字段 3 | — | `cot_store_school_num`（校园店 = SchoolSuper+SchoolMini+SchoolTT+SchoolService） |
| 数据源 | `di_otr_city_hq_act_view` → `di_otr_city_hq_output`（17,409 行） | 需在该视图/基表**新增按 `channel_level2_code ∈ {Travelling, PowerStation, SchoolS}` 计数的列**（或视图内 `CASE WHEN` 派生） |
| 下游消费 | ACT 侧读取城市总部数据 | ACT 侧按店型分项读取，用于档期门店范围/执行项判断 |

### 3.3 字段命名与口径提示【推断·待确认】

- **命名不对称**：出行/加油充电站用英文二级店型（`travelling`/`powerstation`），校园店用 `school`（而非 `schools`）。若沿用接口既有命名规范（其余 `cot_store_*_num` 字段），建议对齐，否则需在接口文档中显式登记本次三个字段的命名与映射，避免后续漂移。
- **三级店型归并口径**：每个 `num` 是「二级店型」计数，需明确其三级子店型集合（见 2.1），以及是否含 `SchoolService`（校园供应）这类边界店型。
- **覆盖模式口径**：沿用 BR-SD-106 的「覆盖门店数」语义需明确过滤条件（糖巧：有效且覆盖模式含 糖巧固定/非固定/不可拜访；冰淇淋：有拜访要求/无拜访要求）——新字段是否同样按品类口径拆分需确认。

---

## 四、企微端（手机端）功能改动点

### 4.1 结论【推断，需确认】

**需要调整，且落点确为「COT 拜访表单中 ACT 执行表单的显示」，但数据层圈选已基本就绪，主要缺口在表单显示条件与执行项口径。**

### 4.2 逐项评估

| # | 手机端关注点 | 现状【知识库实证】 | 是否需要改动 | 说明 |
|---|---|---|---|---|
| 1 | **店型维度是否可被档期圈选** | 门店范围圈选 57 字段已含 `channel_level2_code/name`（二级）、`store_channel_code/name`（三级） | ✅ 数据层已具备 | 三个二级店型（Travelling/PowerStation/SchoolS）及三级店型均在既有值类型内 |
| 2 | **拜访表单中 ACT 执行表单的显示条件** | 档期动作（ACT）为拜访动作四类之一；COT 表单显示条件 400 条**无校园/出行/加油站关键字命中**（`2026-09-15-cot-2026-prd-deep-analysis.md:126`，F74） | ⚠️ **需扩展** | 需让「出行/加油充电站/校园店」门店触发档期动作时，手机端展示对应 ACT 执行表单（或沿用既有档期动作表单，仅圈选范围扩展） |
| 3 | **执行表单数据回流** | 拜访执行表单由 `store-fts`（表单服务）处理上传（离店消息）（`field-ops-microservices-analysis.md:96`） | ⚠️ 视档期执行项而定 | 若三个店型的执行项（如巧克力分销）与既有店型一致，回流链路可复用；否则需新增执行项/字段 |
| 4 | **巧克力分销类执行项口径** | 「加油站与出行不计算巧克力分销」 | ⚠️ **需在档期执行项中排除** | 出行/加油站门店进入档期后，巧克力分销类执行项不应纳入计算，需在表单显示或达标口径中做店型过滤 |
| 5 | **店型在报表/执行域的可见性** | 火车站/飞机场售点、加油站已被 2026 bySKU 报表路由取消（归特殊渠道报表） | ⚠️ 需对齐 | 保证这三个店型在 ACT 档期圈选、拜访执行、回流报表三处口径一致 |

### 4.3 手机端改动清单（建议）

1. **档期动作（ACT）显示条件扩展**：将 ACT 执行表单的触发条件从「既有店型集合」扩展至含 Travelling / PowerStation / SchoolS（或其三级子店型）。
2. **执行项店型过滤**：加油站/出行门店在档期内的巧克力分销类执行项按「不计算」处理（隐藏或不计分）。
3. **店型归属展示**：若执行表单需区分「出行 vs 加油充电站 vs 校园店」标签，需在表单头展示二级/三级店型（数据已在门店主数据中）。

---

## 五、技术实现方案

### 5.1 接口侧（COT-ACT 城市总部 COT 覆盖门店数接口）

1. **数据源加列**（推荐）：在 `customer_sequoia.di_otr_city_hq_output` 基表或 `di_otr_city_hq_act_view` 视图中，按 `channel_level2_code` 对门店数做 `CASE WHEN` 归并，产出 3 个计数列：
   - `cot_store_travelling_num = COUNT(channel_level2_code = 'Travelling')`
   - `cot_store_powerstation_num = COUNT(channel_level2_code = 'PowerStation')`
   - `cot_store_school_num = COUNT(channel_level2_code = 'SchoolS')`
   （三级子店型按 2.1 归并入二级；计数口径与 BR-SD-106 覆盖门店数口径对齐。）
2. **D1 配置透传**：`chain_act` 的 D1 配置（`di_otr_city_hq_act_view_dfk`）将新增列纳入返回字段；字段沿用现有命名规范（若已有其他店型 `cot_store_*_num`，三个新字段并入同一族）。
3. **接口文档更新**：在 COT-ACT 接口文档中登记 3 个新字段的英文名/中文名/二级店型映射/三级归并口径（本次为标绿字段，入库后同步到知识库 `downstream-city-hq-2026.md`）。

### 5.2 手机端（企微端）侧

1. **表单显示条件**：在 COT 拜访表单的档期动作（ACT）显示条件中，加入 Travelling / PowerStation / SchoolS 二级店型（或对应三级店型）作为触发条件。
2. **执行项口径**：加油站/出行门店的巧克力分销类执行项按业务口径过滤（不计算），与执行为王 SPC 达成率口径一致。
3. **回流复用**：执行表单回流沿用 `store-fts` 既有离店上传链路，不新增回流通道（除非出现新执行项字段）。

### 5.3 数据侧（act 库门店快照）

- `act` 库 `cot_store_all` / `cot_stores` / `act_cot_stores` 等快照表若需按店型圈选档期门店，依赖门店主数据 `channel_level2_code` 维度已下沉到快照——**需确认快照是否已携带二级/三级店型列**（OQ-MK3 已记录 staging 空、写入方待生产验证）。

---

## 六、风险与待确认项

| 编号 | 风险/缺口 | 说明 | 建议 |
|---|---|---|---|
| R1 | **COT-ACT 接口文档（标绿字段）未入库** | 需求引用文档在知识库无对应，`chain_act` 现有 21 字段不含 `cot_store_*_num` | 入库该接口文档，补齐字段族与命名规范后再定稿 |
| R2 | **`chain_act` QA 实测 500** | OAuth 依赖不可达，`app_definition.status` 全 0 | 与 5.1 同步解决接口可用性，否则新增字段无法联调验证 |
| R3 | **OQ-MK3：act 库 354 表 staging 空** | 门店快照写入方与刷新节奏待生产验证 | 确认快照含店型列，否则档期门店圈选缺维度 |
| R4 | **命名不对称** | `school` vs `travelling`/`powerstation` 命名风格不一致 | 对齐接口既有 `cot_store_*_num` 命名规范 |
| R5 | **覆盖门店数口径** | 新字段是否沿用 BR-SD-106 的覆盖模式/品类过滤口径未定 | 与业务确认计数口径（糖巧/冰淇淋/全量） |
| R6 | **加油站/出行巧克力分销** | 这两类店型「不计算巧克力分销」，影响档期执行项 | 手机端执行项与达标口径需显式排除 |

---

## 七、结论

- **接口层**：以 `chain_act` 接口（或需求所引 COT-ACT 城市总部覆盖门店数接口）为落点，在数据源视图新增 3 个按二级店型归并的计数列并透传，命名与口径需与既有字段族对齐；**前提是补齐未入库的 COT-ACT 接口文档与接口可用性**。
- **手机端**：需要调整，核心是**扩展 ACT 执行表单的显示条件以纳入出行/加油充电站/校园店**，并对加油站/出行门店的巧克力分销类执行项按「不计算」口径过滤；数据层圈选维度已具备，主要工作是表单触发条件与执行项口径。
- **数据侧**：确认 `act` 库快照携带二级/三级店型列，是档期门店圈选落地的前提。

> 关联文档：`downstream-city-hq-2026.md`（chain_act 现状）｜`channel.md` / `store-channel-taxonomy.md`（店型 taxonomy）｜`campaign-center-deep-dive.md`（act 库 + 57 圈选字段）｜`proj-store-definition.md`（2026 渠道重构）｜`execution-king-kpi-and-dashboard.md`（SPC 2026Q4 新增 + 巧克力分销口径）｜`门店店型定义变更影响评估报告.md`（店型变更下游传播）。
