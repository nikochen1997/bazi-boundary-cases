# bazi-boundary-cases · 八字排盘边界用例

**English** · [中文](#中文说明)

Boundary test cases for BaZi (Four Pillars of Destiny / 八字) chart calculation. Each case is an input birth time plus the expected four pillars (year, month, day, hour stem-branch), so you can drop the JSON into your own test suite and assert equality. No implementation code is included.

Three groups, 22 cases:

- **`cases/lichun.json` — one minute before vs. at Li Chun (立春, Start of Spring).** The BaZi year changes at the exact minute of Li Chun, not on Jan 1 or Lunar New Year. For 1998, 2000, 2004 and 2010, a birth one minute earlier gets a different year pillar *and* month pillar; day and hour pillars stay the same; the luck-cycle (大运) direction flips.
- **`cases/solar-time.json` — true solar time across five cities.** Same clock time (1990-06-15 08:00, UTC+8) in Urumqi, Lhasa, Beijing, Shanghai and Harbin. What changes the chart is whether the corrected time crosses a two-hour (时辰) boundary, not how large the correction is: Beijing (−14 min) and Harbin (+26 min) give the same chart; Urumqi (−130 min) changes the hour pillar.
- **`cases/zishi.json` — early vs. late Zi hour (子时, 23:00–00:59).** Paired cases at 23:30 and 00:30, plus five boundary points inside one calendar day.

Conventions assumed (expected values differ under other conventions): year boundary = Li Chun at minute precision, seconds rounded to the nearest minute; month boundary = the twelve Jie (节) solar terms; late Zi hour (23:00–23:59) keeps the same day pillar; true solar time = clock time + (longitude − 120°) × 4 min + equation of time; valid range 1900–2100.

Also in this repo: [`docs/ai-companion/`](docs/ai-companion/) — essays (in Chinese) on building an AI companion that does not just agree with the user.

These cases come from the regression tests of the chart-calculation part of **Nora Whisper (Nora 心语屋, [norawhisper.com](https://norawhisper.com))**, a Chinese-language AI companion for emotional support that also reads BaZi, Zi Wei Dou Shu, couple chart comparison, date selection, naming and Qi Men Dun Jia. Nora is an AI; it is not a medical service and does not replace a therapist. Data and articles are licensed CC BY 4.0 — attribute "Nora Whisper / Nora 心语屋".

---

## 中文说明

一组可以直接拿来校验八字排盘实现的边界用例：**立春前后一分钟**、**同一时刻不同城市的真太阳时**、**早子时与晚子时**。每条用例给出输入和期望的四柱，附讲解文章。

这些用例来自 **Nora 心语屋（Nora Whisper，[norawhisper.com](https://norawhisper.com)）** 排盘部分的回归测试。Nora 心语屋是一个中文 AI 心理陪伴产品：能聊天、能用真实故事和你交换故事，也能看八字、紫微斗数、两个人的合盘、择日、起名、奇门遁甲问事、面相与八字合看。排盘部分是确定性计算，同一个生辰什么时候算结果都一样，所以这类用例可以长期挂在测试里。

本仓库只放用例数据和文章，不含排盘实现代码。

## 为什么要有这组用例

八字排盘的大部分输入都不难算对。容易算错的是边界：

- **年柱按立春换，立春精确到分钟。** 节气时刻只存到日期的实现，在立春当天会把一整天的人都归到同一年，有一半时间段是错的。年柱换的同一刻，月柱也要一起换。
- **真太阳时校正之后，盘变不变，看的是有没有跨过时辰的界，不是校正了多少分钟。** 北京校正 −14 分钟和哈尔滨校正 +26 分钟排出同一张盘；乌鲁木齐校正 −2 小时 10 分，时柱才换。
- **23:00 之后出生算哪一天。** 早晚子时怎么分，各家约定不同，实现里要有明确的一条，并且测试锁住。

## 用例文件

| 文件 | 内容 | 条数 |
|---|---|---|
| [`cases/lichun.json`](cases/lichun.json) | 1998 / 2000 / 2004 / 2010 四个年份，立春前一分钟与立春当分各一条 | 8 |
| [`cases/solar-time.json`](cases/solar-time.json) | 1990-06-15 08:00 同一时刻，乌鲁木齐 / 拉萨 / 北京 / 上海 / 哈尔滨五地 | 5 |
| [`cases/zishi.json`](cases/zishi.json) | 晚子时 23:30 与次日早子时 00:30 成对；同一日期内五个子时边界点 | 9 |

### 字段说明

```json
{
  "id": "lichun-2000-before",
  "input": { "date": "2000-02-04", "time": "20:39", "gender": "male" },
  "expected": { "year": "己卯", "month": "丁丑", "day": "壬辰", "hour": "庚戌" },
  "note": "立春在 20:40，前一分钟仍属上一年"
}
```

- `date` / `time`：公历日期与钟表时间，东八区。
- `expected`：年、月、日、时四柱的干支。
- `solar-time.json` 额外给出 `longitude`（市中心经度）、`correction_minutes`（校正量）、`true_solar_time`（校正后时刻），期望四柱按校正后的时刻排。

### 本仓库用例采用的约定

换一套约定，期望值会不同。用这组用例之前先对一下：

1. **年界**：立春，分钟口径。节气时刻的秒数四舍五入到分钟（例：2024 年立春 16:26:53 → 16:27）。
2. **月界**：十二节，与年界同一口径。
3. **子时**：早晚子时分开。23:00–23:59 为晚子时，**日柱不换**，时干按当日日干起；00:00–00:59 为早子时，日柱为当日。
4. **真太阳时**：`真太阳时 = 钟表时间 +（当地经度 − 120°）× 4 分钟 + 均时差`。`lichun.json` 与 `zishi.json` 不做真太阳时校正，按钟表时间排；`solar-time.json` 先校正再排。
5. **适用范围**：1900–2100 年。

## 怎么用

把 JSON 读进你的测试框架，逐条喂给自己的排盘函数，断言四柱相等。伪代码：

```
for case in load("cases/lichun.json"):
    result = your_bazi(case.input)
    assert result.year  == case.expected.year
    assert result.month == case.expected.month
    assert result.day   == case.expected.day
    assert result.hour  == case.expected.hour
```

立春那组有一个更省事的写法：同一年份的 before / after 两条，断言**年柱、月柱不同，日柱、时柱相同**。

## 讲解文章

- [生在立春当天，八字算哪一年？差一分钟，年柱和月柱整根换掉](docs/lichun-one-minute.md)
- [真太阳时和北京时间：盘变不变，看的是有没有跨过时辰的界](docs/true-solar-time-five-cities.md)
- [让 AI 起名不胡编：「八字该补什么」怎么算，每个字的出处怎么查](docs/ai-naming-without-making-things-up.md)

## 其他文章：做 AI 陪伴产品的笔记

和排盘用例无关，是同一个产品另一条线上的文章，放在 [`docs/ai-companion/`](docs/ai-companion/)：

- [Nora 心语屋是什么：算法都在讨好你，我做了个会反驳人的 AI](docs/ai-companion/what-is-nora-whisper.md)
- [大模型替不了咨询师，但能接住凌晨三点的崩溃：五种情况请去找真人](docs/ai-companion/when-to-find-a-real-person.md)
- [为什么一个只会顺从你的 AI 会变成毒药](docs/ai-companion/why-a-yes-machine-is-poison.md)
- [把「不迎合」写进生产环境的四道工程闸](docs/ai-companion/four-engineering-gates-against-sycophancy.md)
- [AI 心理陪聊怎么选：四个可以自己验证的判断维度](docs/ai-companion/four-checks-for-an-ai-companion.md)
- [AI 陪伴产品的合规改造：把拟人化互动服务新规拆成代码里的九件事](docs/ai-companion/compliance-checklist-nine-items.md)

## 局限

- 每组用例都是抽样，不是普查。通过这组用例，说明这几条边界做对了，不说明整个实现没有问题。
- `solar-time.json` 取的是 6 月中旬，均时差接近零。换到 2 月初或 11 月初，均时差能到十几分钟。
- 城市经度取市中心，同一城市东西两端还能差几分钟。
- 本仓库只回答「按这套约定，盘应该排成什么样」，不回答「哪套约定更准」。后一个问题涉及流派立场，用例答不了。

## 关于 Nora 心语屋

- 官网：<https://norawhisper.com>
- 八字与紫微双盘对照：<https://norawhisper.com/bazi>
- AI 起名 · 八字补益与典籍出处：<https://norawhisper.com/naming>

Nora 是 AI。她不是医疗机构，不做心理诊断，也替代不了心理咨询师。

## 许可

用例数据与文章以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.zh-hans) 发布：可以自由使用、转载、改编，注明出处「Nora 心语屋」即可。
