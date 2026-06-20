# 数据更新工具 (Data Updater)

## 概述

本文档描述如何为张雪峰 Skill 更新高考数据，尤其是 2026 年新数据的导入流程。

## 数据更新日历

| 时间 | 事件 | 需更新的数据 |
|------|------|------------|
| 6月初 | 高考结束 | 更新高考模式（确认新高考省份） |
| 6月23-26日 | 各省公布分数线+一分一段表 | **立即更新** `data/yifenyiduan/` 和 `data/scores/` |
| 6月底-7月初 | 志愿填报开始 | 确保所有数据到位 |
| 7月中旬 | 录取开始 | 可开始收集 2026 录取数据用于明年 |
| 7月底 | 各省公布投档线 | 更新 `data/scores/` 加入 2026 年数据 |

## 数据源（按优先级）

### 一级数据源（官方，最权威）

1. **各省教育考试院官网**
   - 河南省：http://www.heao.gov.cn/
   - 四川省：http://www.sceea.cn/
   - 广东省：https://eea.gd.gov.cn/
   - 北京市：https://www.bjeea.cn/
   - 山东省：http://www.sdzk.cn/
   - 完整列表见 `data/sources.json`

2. **阳光高考平台 (教育部)**
   - https://gaokao.chsi.com.cn/
   - 权威但更新稍慢

### 二级数据源（聚合平台，方便批量获取）

1. **掌上高考** - https://www.eol.cn/e_html/gk/fsx/index.shtml
2. **中国教育在线** - https://gkcx.eol.cn/
3. **各省教育厅官方微信公众号**

### 三级数据源（第三方，需交叉验证）

1. 夸克高考 - 数据更新快，但需要交叉验证
2. 蝶变志愿 - 数据较全，偶有误差

## 标准数据格式

### 一分一段表格式 (yifenyiduan/{province}_{year}.json)

```json
{
  "province": "henan",
  "year": 2026,
  "category": "理科",
  "total_examinees": 1400000,
  "source": "河南省教育考试院",
  "source_url": "http://www.heao.gov.cn/...",
  "updated_at": "2026-06-25",
  "segments": [
    {"score": 750, "count": 3, "cumulative": 3},
    {"score": 749, "count": 2, "cumulative": 5}
  ]
}
```

### 录取数据格式 (scores/{province}_{year}_{batch}.json)

```json
{
  "province": "henan",
  "year": 2026,
  "batch": "本科一批",
  "source": "河南省教育考试院",
  "source_url": "http://www.heao.gov.cn/...",
  "updated_at": "2026-07-25",
  "schools": [
    {
      "code": "10001",
      "name": "北京大学",
      "category": "理科",
      "min_score": 698,
      "min_rank": 95,
      "avg_score": 701,
      "enrollment": 45,
      "enrollment_change": 0,
      "change_note": "与上年持平"
    }
  ]
}
```

### 省控线格式 (policies/cutoff_scores.json)

```json
{
  "year": 2026,
  "provinces": {
    "henan": {
      "理科": {"一本": 514, "二本": 409, "专科": 200},
      "文科": {"一本": 527, "二本": 445, "专科": 200}
    }
  }
}
```

## 2026 年数据导入检查清单

- [ ] 确认 2026 年新高考省份名单（预计新增：河南等）
- [ ] 更新 `data/policies/gaokao_reform.json`
- [ ] 收集各省一分一段表 → 填充 `data/yifenyiduan/`
- [ ] 收集各省省控线 → 填充 `data/policies/cutoff_scores.json`
- [ ] 收集各省投档线 → 填充 `data/scores/`
- [ ] 更新《从就业看专业》数据集（薪资、就业率 2025-2026 版）
- [ ] 更新「冲稳保」阈值（基于最新数据微调）
- [ ] 更新「防撞车」热点方向
- [ ] 运行 3 个测试案例验证数据一致性

## 自动化工具

### 从掌上高考抓取省控线

```bash
# 使用 WebFetch 工具
python -c "
url = 'https://www.eol.cn/e_html/gk/fsx/index.shtml'
# 抓取并解析各省分数线
"
```

### 解析一分一段表

一分一段表通常是 PDF 或图片格式，需要：
1. 下载 PDF → 使用 OCR 或手动解析
2. 或从第三方聚合网站（如掌上高考）获取结构化的 HTML 表格数据
3. 转换为标准 JSON 格式

### 数据分析：识别异常值

```python
# 快速检查数据一致性
# 1. 同一学校相邻年份位次波动超过 20% → 标记为异常
# 2. 同一省份同批次学校录取位次与学校层次明显不匹配 → 标记检查
# 3. 新高考省份第一年数据 → 标注波动警告
```

## 维护责任人

- 每年 6-7 月：集中更新高考数据
- 每季度：更新一次就业数据（基于招聘平台报告）
- 每年 12 月：更新 AI 替代风险评估
