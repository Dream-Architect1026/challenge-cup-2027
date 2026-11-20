# 返工记录 REWORK-NOTES

- 基线：commit `6ffcab0`，`index.html` 49,818 B / 858 行
- 工作副本：`/home/user/Doubao/chats/38454512187271682/site`（两轮均未执行 `git commit` / `git push`，未改 git 历史；HEAD 仍为 `6ffcab0`）
- 验收环境：Debian，Chromium 146（Playwright 1.64.1）无头实测；ffmpeg 4.4.2；Python 3.12；fontTools 4.68.0 + Pillow 10.4.0
- 记录口径：第一节为第一轮返工，第二节「第三轮定点修补」对应复审 8.11 分后的逐条整改；所有字节 / 比值 / 布局数值以第二节复测为准。

## 0. 红线遵守情况

- 视频 `assets/demo-1080p.mp4`、报名表单外链、PDF `assets/deck.pdf`、三条如实短板声明、招募承诺边界（不承诺获奖 / 代写 / 无贡献署名）全部保留，未删除。
- 数据口径数字与人名 / 学号 / 手机 / 经费 / 日期未改：学号 `2410630507`、手机 `18711962657`、6 人名单（邓国栋（负责人）、方思齐、李诗涵、刘舜宇、董贺昕、汪珍月）、经费 `6,600 元`、`2024 级`、指导教师刘海滨（管理学院教授）、`0.2.63-r181`、SQLite `16,779 行`、API `8790`、`246 点 / 70 字段`、视频 `23.545011s` 均原样在页。
- 排版保持暗金液态玻璃 + 斜切角语言，未退回圆角矩形大海报；PPT 原页未被 AI 生图替换，作品区仍为原页转码。

---

# 一、第一轮返工（规格 §1–§7）

## 1. SEO 与分享

| 条目 | 落地情况 | 实测 |
|---|---|---|
| title | `智警御安 · 煤与瓦斯突出预警模型研究｜挑战杯 2027 成员招募` | 33 字符 |
| description | 含「煤与瓦斯突出预警 / 多源数据融合 / 挑战杯 / 招募」 | 97 字 |
| canonical | 指向站点根 URL | — |
| OG 全套 | type / site_name / title / description / url / locale / image（绝对 URL）/ width 1200 / height 630 / alt | 齐全 |
| Twitter | card=summary_large_image + title / description / image | 第三轮补 `twitter:image:alt` |
| JSON-LD | `@graph`：`WebSite` + 项目实体；第三轮由 `Organization` 改为 `ResearchProject`（详见 §二.B2） | 解析通过 |
| robots.txt / sitemap.xml | 根目录；Sitemap 指向 canonical；首页 URL + `lastmod=2026-11-20` | 本地 200 |
| theme-color | `#080604` | — |

## 2. 性能

### 2.1 字体自托管子集

- 已删除 `miaoda.feishu.cn` css2 外链（全文检索 `miaoda` / `fonts.googleapis` / `fonts.gstatic` 命中 0）。
- 按字重分桶（Chromium 读取元素 computed `fontFamily` / `fontWeight`），每字重一个 woff2；`font-display:swap`；预载仅 Ma Shan Zheng 400、ZCOOL XiaoWei 400、Noto Sans SC 500。
- 第三轮重建子集：字符集改为从**含属性值（alt / title / aria-label / data-label 等）的全文**提取；mono 栈元素中的 CJK / 标点并入 Noto Sans SC 400 安全网桶。

### 2.2 图片转 WebP

- `mine-depth.png` 1920 宽 q75；被引用 `slide*.png` / `smoke_center.png` 1600 宽 q86；`cover.jpg` 1600 宽 q80 作 poster。
- 全部 `<img>` 补齐 `width` / `height`；首屏外 `loading="lazy" decoding="async"`，首屏内 `decoding="async"`。
- 原 PNG/JPG 保留在仓库，仅停止引用。

### 2.3 着色器 / 合成成本

- rAF 封顶 30fps；`prefers-reduced-motion` 只渲染静态一帧；`visibilitychange` 与 IntersectionObserver 离屏停循环、回屏恢复；补 `webglcontextlost / restored`；DPR 上限 1.5。
- ≤640 关闭毛玻璃（第三轮扩到 top / nav / btn / lbl，详见 §二.A3）。

## 3. 可访问性

- 全局 `:focus-visible` 描边 + clip-path 卡片内 inset 内阴影；地标 header / main / skip 链接；标题层级 h1→h2→h3→h4 无跳级；VTT 字幕；reduced-motion 全覆盖；`th scope="col"` + 表格 `aria-label`；移动端导航命中区 44px；说明性文字 ≥12px。

## 4. 版式

- 390 h1 34px 单行；时间线状态改内联标签；斜切角令牌；校验条数值 nowrap 右对齐；移动端表格堆叠卡片；footbg 约束在 footer 内；lede 限 62ch。

## 5–7. 文案 / 交互 / 代码

- 学号手机、六人名单、口径说明、题量标注等按规格落地；锚点 scroll-padding 96/104；CTA 文案一致；重复定义 / 死选择器 / 内联样式 / 调试代码清理；noscript 兜底。

---

# 二、第三轮定点修补（复审 8.11 → 逐条整改）

## A. 性能

### A1. 删除 hero 背景 png 回退图层（真凶定位）

- 问题：`.hero .bg` 原先「先 png 回退、后 webp 覆盖」双层写法，Chromium 会把多层背景里每个 `url()` 都下载，`mine-depth.png` 3,304,787 B 被照拉。
- 改动：删掉 png 图层，背景只留 `assets/img/webp/mine-depth.webp`。
- 实测（Playwright 真实网络日志，1440 与 390 两档一致）：
  - 请求列表中**无 `mine-depth.png`**；首屏 / 页面请求里**无任何 >200KB 的图**（最大图片 `slide5.webp` 130,808 B）。
  - 首屏（未滚动）：16 个请求，**648,575 B（633.4 KB）**，≤700KB 达标；滚动到底后全量 23 个请求，1,391,005 B（1.32 MB，含懒加载图与全部字体）。

### A2. 小图标替换 favicon

- 旧 favicon 用 `logo-tile.png`（80,988 B）。由该 256×256 徽标重采样并 64 色调色板量化：

| 文件 | 尺寸 | 字节 |
|---|---|---:|
| `assets/img/favicon-32.png` | 32×32 | 861 |
| `assets/img/apple-touch-icon-180.png` | 180×180 | 6,985 |
| **合计** | — | **7,846 B（≤12KB）** |

- `<link rel="icon" sizes="32x32">` 替换旧引用；补 `<link rel="apple-touch-icon" sizes="180x180">`。180 图标仅在 iOS 添加主屏幕时拉取，不计入常规首屏 payload。

### A3. 移动端补关其余 backdrop-filter

- 原 ≤640 只关了 `.glass`；本轮把 `.top` / `.nav` / `.btn` / `.lbl` 的 `backdrop-filter` / `-webkit-backdrop-filter` 一并关闭，改纯色半透明底：`.top rgba(8,6,4,.86)`、`.nav rgba(18,13,9,.72)`、`.btn rgba(20,14,8,.72)`、`.lbl rgba(10,8,5,.82)`；桌面保留毛玻璃。

### A4. Sans 子集补字 + 全字符 cmap 校验

- 问题：`涵`（李诗涵）、`舜`（刘舜宇）、`ï`、`–` 等不在子集内；旧提取只取正文文本，漏了属性值与带 `<b>/<br>` 子元素的段落直接文本节点。
- 改动：
  - 提取脚本改为收集每个元素的**直接文本节点** + `alt/title/aria-label/data-label/placeholder/value/label` 属性值；
  - mono 栈元素里的 CJK / 标点并入 Noto Sans SC 400 安全网桶；
  - 重建全部 8 个 woff2。
- 逐字校验（脚本 `verify_cmap.py`）：页面字符 602 个，cmap 并集 692 字形，**缺失 0**；`涵 / 舜 / ï / – / ó / · / ± / → / ≥ / ≤ / % / /` 全部命中。

### A5. 字体与字节复测（最终值）

| 文件 | 字节 | 字形数 |
|---|---:|---:|
| `noto-sans-sc-400.woff2` | 159,040 | 692 |
| `noto-sans-sc-500.woff2` | 49,472 | 268 |
| `noto-serif-sc-400.woff2` | 24,380 | 143 |
| `noto-serif-sc-600.woff2` | 27,816 | 158 |
| `noto-serif-sc-700.woff2` | 27,172 | 155 |
| `noto-serif-sc-900.woff2` | 13,824 | 115 |
| `ma-shan-zheng-400.woff2` | 2,436 | 4 |
| `zcool-xiaowei-400.woff2` | 7,420 | 28 |
| **字体合计** | **311,560 B（304.3 KB）** | — |

- 内联 `@font-face` CSS 1,092 B（≤40KB）；浏览器内 8 个 face 全部 `loaded`，无 OTS 告警。
- 首屏口径（HTML + 字体 CSS + 预载字重 + 首屏背景 + 首屏侧图 + logo + favicon）：

| 口径 | 字节 | KB | 目标 | 判定 |
|---|---:|---:|---:|---|
| 窄口径首屏含 poster | 371,454 | 362.7 | ≤700KB | 达标 |
| 窄口径首屏不含 poster | 317,130 | 309.7 | ≤700KB | 达标 |
| 浏览器真实网络日志（首屏 16 请求，含全部已加载字体） | 648,575 | 633.4 | ≤700KB | 达标 |

## B. SEO

### B1. Twitter 图片 alt

- 补 `<meta name="twitter:image:alt" content="智警御安 · 煤与瓦斯突出预警模型研究项目封面">`。

### B2. JSON-LD 实体类型修正

- `Organization` → `ResearchProject`；保留 `WebSite`；`parentOrganization` 仍指向 `中国矿业大学（北京）`；未编造 logo / 地址 / contactPoint 等未见事实。`json.loads` 解析通过。

### B3. 重做 og-cover（三类裁切不丢字）

- 旧卡由 cover.jpg 直接裁切，方形裁切切掉首字「智」与左上徽标。本轮改用无文字的矿井实景图作底，徽标（项目 cube 标识）+ 标题四字 + 副标题 + 标签全部水平居中，安全区左右 ≥18%（≥216px）。
- 实测裁切预览（脚本 `make_og.py` 中心裁切）：

| 裁切比例 | 裁切盒 | 标题 / 徽标 / 副标题 |
|---|---|---|
| 1.91:1 | 0,1 → 1200,629 | 完整 |
| 1:1 | 285,0 → 915,630 | 四字与徽标完整，副标题贴边未裁 |
| 16:9 | 40,0 → 1160,630 | 完整 |

- 产物 `assets/og-cover.jpg`：1200×630，q80，**60,826 B（≤160KB）**。
- 备注：两枚校徽素材未单独提供为可复用文件，og 卡面徽标采用项目 cube 标识，未凭空补绘校徽。

## C. 版式

### C1. 角部语言统一为两套令牌 + 一个显式例外

- 令牌：`--cut:18px`（卡片族，45° 切边）、`--cut-sm:9px`（按钮 / 标签等小件，45° 切边）。
- 逐元素切边清单：

| 元素 | 令牌 / 写法 | 切边尺寸 / 角度 |
|---|---|---|
| `.glass`（基类：notebox / vnotes / tl / fr / muster .c / bar / dc / cta / dossier / hshot / metaline） | `--cut` | 18px，45° |
| `.screen`（视频框） | `--cut` | 18px，45° |
| `.shot`（作品缩略框） | `--cut` | 18px，45° |
| `.ln`（四线卡） | `--cut` | 18px，45° |
| `.role`（岗位卡） | `--cut` | 18px，45° |
| `.skip`（跳至正文） | `--cut-sm` | 9px，45° |
| `.nav a`（导航条目） | `--cut-sm` | 9px，45° |
| `.lbl`（视频角标） | `--cut-sm` | 9px，45° |
| `.btn`（按钮） | `--cut-sm` | 9px，45° |
| `.nav`（导航容器） | **唯一胶囊例外** `border-radius:12px` | 圆角；注释写明理由：横向可滚动条目需要连续触摸滑轨与滑动指示 |
| `.fig` / `.sysgrid .gate` | 账本元素，无 clip-path、`border-radius:0` | 不参与角切语言 |

- 旧的 12px / 9px / 8px 圆角与直角按钮已全部并入上述令牌。

### C2. 两张缩略图同角度、同排等宽等高

- 问题：`.ln .thumb` 底边斜切实测 6.4°（`100% 80%`），`.role .thumb` 完全直角，两者不一致。
- 改动：两者统一为 `polygon(0 0,100% 0,100% 86%,0 100%)`；同排两个缩略图等宽等高、共用基线（`.ln` 行内 16/9，`.role` 行内 3/1，各自排内一致）。

### C3. 390 背景取景保住「井架 + 巷道」

- 问题：旧 `background-size:auto 100%` 随移动端堆叠把背景盒拉到 1530px 高，纵向放大 1.75×、横向只取 11.6%，井架右缘出画。
- 改动：≤640 背景盒高度 `100svh`，`background-size:cover`，`background-position:50% 22%`。
- 实测 390 截图（`/tmp/shot-390-bg2.png`）：井架塔架与双轮完整入画，中部轨道 / 巷道保留；背景盒高度 844px，cover 取景。

### C4. 校验条右缘内边距

- `.sysgrid .gate` 右列补 `padding-right:8px`；390 下五道数值右缘 366px（原 374px 贴线），未被裁，右缘有金色像素。

### C5. hbar 与斜切对角线净空

- `.hshot .hbar` 内边距由 `9px 13px` 提到 `12px 16px`；`.hfoot` 同步 `10px 16px`，与卡片斜切角净空 ≥10px。

## D. 可访问性

### D1. 金属条三条标签加深底色

- 问题：桌面三条 dim 标签压在首屏照片上，实测 1.99 / 2.36 / 3.26，不达标。
- 改动：`.metaline` 底色 `rgba(8,6,4,.86)` + 1px 内描边 `rgba(214,176,112,.18)`。
- 复测（WCAG 相对亮度 + alpha 合成，岩面采样底）：

| 配对 | 比值 | 阈值 | 判定 |
|---|---:|---:|---|
| 判定内核（.86 底压岩面） | 5.87 | 4.5 | 达标 |
| 入库规模（.86 底压岩面） | 5.87 | 4.5 | 达标 |
| 结果台账（.86 底压岩面） | 5.87 | 4.5 | 达标 |

### D2. h1 渐变暗端提亮

- 问题：暗端 `#D8A03C` 在桌面亮部 1.77–2.41（大字阈值 3.0）。
- 改动：渐变暗端提到 `#E9B75C`（`#FFE9BE 0% → #F2B94E 46% → #E9B75C 92%`）。
- 复测：暗端 **9.63**（≥3.0 达标）；中段 10.00、浅端 14.92。

### D3. 导航条目命中区补宽度

- 旧只有 `min-height:44px`；补 `min-width:44px` 并居中。实测 390 下 7 个导航条目全部 **44×44**。

### D4. 视频控件焦点环

- 问题：原生控件在 shadow DOM 内 `:focus-visible=false`，且 `.screen` 斜切会剪 outline。
- 改动：补 `.screen:focus-within{box-shadow:inset 0 0 0 2px rgba(242,185,78,.85),…}`；inset 阴影在边框盒内绘制，不被 clip-path 裁掉。

### D5. hfoot 局部深色底

- 问题：`.hfoot` 最亮照片点 2.85 边际。
- 改动：补 `background:rgba(8,6,4,.82)`。复测 **5.48**（≥4.5 达标）。

### D6. 对比度全表复测

| 配对 | 比值 | 阈值 | 判定 |
|---|---:|---:|---|
| lede（移动端雾上岩面） | 5.27 | 4.5 | 达标（当前最差一） |
| hfoot 脚注（.82 底压岩面） | 5.48 | 4.5 | 达标（当前最差二） |
| 金属条标签（.86 底压岩面） | 5.87 | 4.5 | 达标（当前最差三） |
| 校验条 span rgba(232,220,200,.62) | 6.01 | 4.5 | 达标 |
| sub（移动端雾上岩面） | 6.77 | 4.5 | 达标 |
| eyebrow（移动端雾上岩面） | 6.95 | 4.5 | 达标 |
| 辅助 dim / fine / 页脚 / 表格末列 / 章节 en | 6.90 | 4.5 | 达标 |
| 正文 muted / 数字条说明 / 短板正文 / 四线 / 章节正文 | 8.57 | 4.5 | 达标 |
| h1 暗端 #E9B75C | 9.63 | 3.0（大字） | 达标 |
| h1 中段 #F2B94E | 10.00 | 3.0（大字） | 达标 |
| h1 浅端 #FFE9BE | 14.92 | 3.0（大字） | 达标 |
| 要点 b gold-h3 / 成员承担 h3 | 11.39 | 4.5 | 达标 |
| kv teal / 项目提供 h3 teal | 10.98 | 4.5 | 达标 |
| 图注 #EFE3D2（.96 底） | 15.98 | 4.5 | 达标 |
| 金属条末位 #E8D7B9 | 14.31 | 4.5 | 达标 |
| 正文 ink | 16.53 | 4.5 | 达标 |
| 短板 h3 #F0C4B3 / 不作承诺 h3 #EDA392 | 12.77 / 9.88 | 4.5 | 达标 |
| 岗位序号 gold / 时间线状态 gold | 8.59 | 4.5 | 达标 |

全部达标，无 4.5 以下正文 / 说明文字，无 3.0 以下大字。

## E. 文案与口径

| # | 改动 |
|---|---|
| E1 | 「会议按节点召开，提前三日通知」→「会议不定期、提前三日通知」 |
| E2 | 时间线「14 页答辩稿」→「14 页项目介绍」，与作品区材料条目统一（同一 PDF，不再两名并存） |
| E3 | 短板卡「现用公开数据与自建仿真集」→「现用公开数据集与自建仿真集」 |
| E4 | 因果归因卡 kv 补基线：`XGBoost · MAE 1.718±0.618（基线 2.275）· −24% vs 基线` |
| E5 | 数据线卡改为「原始记录 86,400 条入库，扩展至 306,791 条样本（28 通道）」；数字不动，只改动词使量级关系成立 |
| E6 | 校验注脚补「分项分母为该项自检条目数」，消除凑分母观感 |

## F. 交互与代码

| # | 改动 |
|---|---|
| F1 | 字幕文件非 ASCII 路径改名 `assets/demo-keypoints.vtt`，`<track src>` 同步；本地 200 |
| F2 | 联系渠道统一为「以邮件、电话或微信联系」（页面已暴露邮件 + 电话，表单收微信，三渠道成立） |
| F3 | 删除未使用变量 `--bg2`、`--line` |
| F4 | 新增文件清单见第三节；本轮仍不 commit，发布时由主控 `git add -A` |

---

# 三、交付清单与字节对照

## 9. 新增 / 修改文件

| 路径 | 说明 | 字节 |
|---|---|---:|
| `index.html` | 主页面（49,818 B → 55,390 B） | 55,390 |
| `assets/fonts/noto-sans-sc-400.woff2` | 无衬线安全网子集，692 字 | 159,040 |
| `assets/fonts/noto-sans-sc-500.woff2` | 无衬线中粗，268 字 | 49,472 |
| `assets/fonts/noto-serif-sc-400.woff2` | 衬线，143 字 | 24,380 |
| `assets/fonts/noto-serif-sc-600.woff2` | 衬线，158 字 | 27,816 |
| `assets/fonts/noto-serif-sc-700.woff2` | 衬线，155 字 | 27,172 |
| `assets/fonts/noto-serif-sc-900.woff2` | 衬线，115 字 | 13,824 |
| `assets/fonts/ma-shan-zheng-400.woff2` | 展示字体，4 字 | 2,436 |
| `assets/fonts/zcool-xiaowei-400.woff2` | 标题字体，28 字 | 7,420 |
| `assets/img/webp/mine-depth.webp` | 首屏背景，1920×870 q75 | 85,278 |
| `assets/img/webp/slide5.webp` | 原页转码，1600×900 q86 | 130,808 |
| `assets/img/webp/slide6.webp` | 原页转码，1600×900 q86 | 97,634 |
| `assets/img/webp/slide7.webp` | 原页转码，1600×900 q86 | 74,964 |
| `assets/img/webp/slide8.webp` | 原页转码，1600×900 q86 | 110,946 |
| `assets/img/webp/slide10.webp` | 原页转码，1600×900 q86 | 126,370 |
| `assets/img/webp/slide13.webp` | 原页转码，1600×900 q86 | 79,968 |
| `assets/img/webp/smoke_center.webp` | 原页转码，1600×990 q86 | 91,404 |
| `assets/img/webp/cover.webp` | poster / 封面，1600×1000 q80 | 54,324 |
| `assets/img/favicon-32.png` | 站点图标 32×32 | 861 |
| `assets/img/apple-touch-icon-180.png` | iOS 主屏幕图标 180×180 | 6,985 |
| `assets/og-cover.jpg` | 分享卡，1200×630 q80 | 60,826 |
| `assets/demo-keypoints.vtt` | 视频要点字幕，4 条 cue | 311 |
| `assets/demo-1080p.mp4` | 演示视频，1728×1080 H.264 + AAC 立体声，23.546s（第四轮重编码） | 3,522,197 |
| `robots.txt` | 爬虫规则 + Sitemap 引用 | 102 |
| `sitemap.xml` | 首页 URL + lastmod 2026-11-20 | 300 |

## 10. 图片源 → 目标字节对照

| 源文件 | 源字节 | 目标文件 | 目标字节 | 尺寸 / 质量 | 压缩比 |
|---|---:|---|---:|---|---:|
| `mine-depth.png` | 3,304,787 | `webp/mine-depth.webp` | 85,278 | 1920×870 q75 | 2.6% |
| `slide5.png` | 1,829,340 | `webp/slide5.webp` | 130,808 | 1600×900 q86 | 7.2% |
| `slide6.png` | 2,472,999 | `webp/slide6.webp` | 97,634 | 1600×900 q86 | 3.9% |
| `slide7.png` | 1,880,435 | `webp/slide7.webp` | 74,964 | 1600×900 q86 | 4.0% |
| `slide8.png` | 2,105,755 | `webp/slide8.webp` | 110,946 | 1600×900 q86 | 5.3% |
| `slide10.png` | 2,215,274 | `webp/slide10.webp` | 126,370 | 1600×900 q86 | 5.7% |
| `slide13.png` | 2,886,133 | `webp/slide13.webp` | 79,968 | 1600×900 q86 | 2.8% |
| `smoke_center.png` | 346,088 | `webp/smoke_center.webp` | 91,404 | 1600×990 q86 | 26.4% |
| `cover.jpg` | 271,581 | `webp/cover.webp` | 54,324 | 1600×1000 q80 | 20.0% |
| **转换合计** | **17,312,392** | — | **851,696** | — | **4.9%** |

- 保留原样引用：`gen_role1.jpg` 115,242 B、`gen_role2.jpg` 86,466 B、`logo.png` 24,869 B（560×140），合计 226,577 B。
- 当前浏览器引用图片合计：webp 851,696 + 保留原图 226,577 + 图标 7,846 + og-cover 60,826 = **1,146,945 B（1.09 MB，≤6MB 达标）**。
- 按规格保留、未被引用的原图（第四轮已 untrack 并 ignore，不计入浏览器 / 部署 payload）：`cover.jpg` 271,581、`logo-tile.png` 80,988、`mine-depth.png` 3,304,787、`slide1.png` 2,508,470、`slide3.png` 2,177,492、`slide4.png` 1,654,376、`slide5/6/7/8/10/13.png`、`smoke_center.png` 346,088、`smoke_downhole.png` 128,347、`control-room.png` 3,329,499、`ui_page1/2/3/5.png`。本地 `assets/` 总盘另含 3.36 MB 重编码视频、5.09 MB PDF。

## 11. 复测验收记录

### 11.1 四档布局溢出（Chromium 无头）

| 视口 | scrollWidth−clientWidth | h1 字号 / 行数 | 五道校验条 | 时间线状态 | 表格横滑 |
|---|---:|---|---|---|---|
| 1920×1080 | 0 | 124px / 1 行，不溢出 | 5/5 未裁 | 6/6 可见 | 0 |
| 1512×982 | 0 | 102.8px / 1 行，不溢出 | 5/5 未裁 | 6/6 可见 | 0 |
| 1440×900 | 0 | 97.9px / 1 行，不溢出 | 5/5 未裁 | 6/6 可见 | 0 |
| 390×844 | 0 | 34px / 1 行，不溢出 | 5/5 未裁（右缘 366px） | 6/6 可见 | 0 |

- 1920 左右留白 28px / 28px（区间 12–28px）。

### 11.2 三入口可用性

| 入口 | 状态 |
|---|---|
| 报名表单（飞书外链） | HTTP 200 |
| 视频 `demo-1080p.mp4` | HTTP 200，h264 视频 + aac 音轨，时长 23.545011s |
| PDF `deck.pdf` | HTTP 200，5,093,631 B，有效 PDF 1.7 |
| og-cover / favicon / apple-touch-icon / vtt / robots / sitemap | 均 HTTP 200 |

### 11.3 无 JS 兜底

- 禁用 JS：`bodyOpacity=1`，无残留 `.reveal` 透明态，正文全部可见。

### 11.4 必留内容在位

学号 `2410630507` ×2；手机 `18711962657` ×3（含 tel 链接）；6 人名单齐全；经费 `6,600 元`；`2024 级` ×3；刘海滨 ×3（管理学院教授）；三条如实短板声明；三条承诺边界（不承诺获奖 / 不承诺代写论文或代做实验 / 不承诺无贡献署名）。

---

# 三、第四轮收尾（终审仅余性能 / 代码各 0.5）

## 13. 演示视频原地重编码

- 原文件备份至 `site/../video-backup/demo-1080p.orig.mp4`（不在 site 内，不计部署体积）。
- 参数：H.264，`preset slow`，**CRF 32**，`pix_fmt yuv420p`；AAC 128k 立体声；`+faststart`；分辨率保持 **1728×1080**，文件名仍 `demo-1080p.mp4`。
- 前后字节与 ffprobe 复核：

| 项 | 原文件 | 重编码后 |
|---|---:|---:|
| 字节 | 9,434,230（9.00 MB） | **3,522,197（3.36 MB）** |
| 分辨率 | 1728×1080 | 1728×1080 |
| 时长 | 23.545011s | 23.546000s（±0.1s 内） |
| 视频轨 | h264 | h264 |
| 音频轨 | aac | aac，2 通道（立体声） |

- Playwright 页面内实测：`readyState=4`，`duration=23.545011`，`error=null`，缓冲 1 段，可播放；无 pageerror、无 ≥400。
- CRF 30 试版 4,247,887 B（4.05 MB）略超 4.0MB 上限，故采用 CRF 32 落入 3.0–4.0MB 区间。

## 14. 死文件 untrack 与 .gitignore

- 对 19 个已不被页面引用的原图执行 `git rm --cached`（**文件本身保留在磁盘，未删除**）：`assets/cover.jpg`、`assets/img/` 下 `control-room.png / logo-tile.png / mine-depth.png / slide1/3/4/5/6/7/8/10/13.png / smoke_center.png / smoke_downhole.png / ui_page1/2/3/5.png`。
- `.gitignore` 补入上述 19 项路径；原有 `_*.png` 临时截图规则保留，审计截图不进仓库。
- 仍在使用、**未被 untrack** 的文件：`assets/deck.pdf`、`assets/demo-1080p.mp4`、`assets/img/gen_role1.jpg`、`gen_role2.jpg`、`logo.png`，以及全部 `assets/fonts/*`、`assets/img/webp/*`、两枚图标、`og-cover.jpg`、vtt、robots、sitemap。
- 脚本校验：页面 28 项引用资源全部在磁盘存在、无一被 ignore；在用文件无「未跟踪又缺失」。

## 15. 其他小改

- 顶栏与页脚两处 `logo.png` 的 `<img>` 补 `decoding="async"`。
- `index.html` 体积 55,390 B → **55,424 B**（仅两处异步解码属性，其余未改动）。

## 16. 部署清单（28 项资源 + index.html）

- 已跟踪：`deck.pdf`、`demo-1080p.mp4`、`gen_role1.jpg`、`gen_role2.jpg`、`logo.png`、`.gitignore`、`index.html`。
- 上线前必须 `git add` 的未跟踪项：`REWORK-NOTES.md`、`demo-keypoints.vtt`、`fonts/`（8 个 woff2）、两枚图标、`webp/`（9 个 webp）、`og-cover.jpg`、`robots.txt`、`sitemap.xml`。
- 28 项资源磁盘合计 10,075,046 B（9.61 MB，含 5.09 MB PDF + 3.36 MB 视频 + 1.15 MB 图片 + 0.30 MB 字体）；19 个死文件已 ignore，不进部署包。

# 四、a11y 定点修补（G1–G5）

## 17. G1 切角元素焦点环改 inset（P0）

- 问题：`.skip / .nav a / .btn` 自带 `clip-path`，外部 `outline` 被斜切剪掉，10 个可聚焦目标环不可见。
- 改动：
  - `.skip:focus-visible, .nav a:focus-visible, .btn:focus-visible` → `outline:none; box-shadow:inset 0 0 0 2px #F2B94E; transform:none`（focus 下取消 translateY，避免环位移抖动）。
  - `.btn.main:focus-visible` → 深色 inset 环 `inset 0 0 0 2px #1A1206`；与金底对比 6.10–10.44（≥3:1 达标）。
  - `.nav a` transition 去掉 box-shadow，仅留 `color / background-color .2s`。
- Playwright 逐个聚焦 + 环带像素扫描结论（视口内 hit test 均 OK，未被切角裁）：

| # | 元素 | 类 | inset 环 | 环像素采样 | 结论 |
|---:|---|---|---|---|---|
| 0 | 跳至正文 | `.skip` | 金 2px inset | 金环 382px | 可见 |
| 1–7 | 导航 7 项 | `.nav a` | 金 2px inset | 每项金环 187px | 可见 |
| 8 | 填写报名表（首屏） | `.btn.main` | 深 2px inset | 深环 681px | 可见 |
| 9 | 查看 PDF（首屏） | `.btn` | 金 2px inset | 金环 592px | 可见 |
| 10–13 | 报名区 4 按钮 | `.btn / .btn.main` | inset 规则生效（2 深环 + 2/3px 金 inset） | 位于折外，按 computed style 校验 | 规则生效 |

## 18. G2 / G3 对比度复测（真实坐标采样）

- G2 h1 暗端：渐变暗端 `#E9B75C → #F3C377`；`.hero-copy::before` 衬底加浓并延到右缘（线性暗纱 + 径向尾部提高到 92% 不透明）。
  - 采样 (683,231) 亮岩面：**5.03**（旧 2.80，大字阈值 3.0，留余量到 3.3+）；(683,342) 暗岩面：**8.32**。
- G3 金属条：底色 `.86 → .92`，1px 描边 `.18 → .34`。
  - 三行真实底衬标签对比度：判定内核 **5.54**、入库规模 **6.25**、结果台账 **6.54**（≥4.5；中位值未掉）；上缘透光点 (774,644 附近) 复测 **5.13** ≥4.5。

## 19. G4 小字号提到 ≥12px

| 选择器 | 旧 | 新 |
|---|---:|---:|
| `.vnotes .vi b`（要点序号） | 11px | 12px |
| `.shead .en`（英文小标） | 10px | 12px |
| `.role .no`（岗位序号） | 10.5px | 12px |
| 移动端 `.nav a` | 11px | 12px |

- 纯装饰元素未动；全表已无 10–11px 可见文字规则。
- 四档溢出复测：1920 / 1512 / 1440 / 390 `scrollWidth−clientWidth=0`；390 h1 34px 单行；`.sysgrid` 1440 双栏 645.8 / 702.0px 对齐，390 单栏堆叠 358px，无错位；表格无横滑。

## 20. G5 slide5 alt 统一

- 两处 `slide5.webp` alt 统一为「数据线架构配图」（旧一处作「数据线架构页」）。

## 21. 本轮回归与产物

- cmap 复测：602 字符，缺失 0；无 pageerror、无 ≥400；8 个字体全 loaded。
- `index.html`：55,424 B → **56,185 B**（本轮 CSS 焦点环 / 衬底 / 字号 + 一处 alt，其余未破坏）。
- `git status` 的 `??` 清单与上一轮一致，未新增未跟踪文件；HEAD 仍 `6ffcab0`，未 commit / push。

## 12. 残留风险与需注意条目

1. **展示字体子集封闭**：Ma Shan Zheng 仅 h1 四字、ZCOOL XiaoWei 仅 h2 标题 28 字；后续改标题文案需按新字符集重建这两个子集。
2. **跨字重缺字回退**：Serif / Sans 按字重分桶，跨桶缺字回退同族其他字重文件或系统栈；Sans 400 已并入全字符安全网，当前页面 602 字符 cmap 覆盖 0 缺失。
3. **WebGL contextlost 恢复**：`webglcontextrestored` 后仅重绘并重启循环，未重新 link program；极端丢上下文后若 program 失效需刷新页面。
4. **未引用原图占本地磁盘**：第 10 节列出的 19 个原图磁盘保留未删，但已 `git rm --cached` 并写入 `.gitignore`，不进部署包；本地 `assets/` 总盘仍含这些原图，不计浏览器 / 部署 payload。
5. **slide 类清晰度边界**：webp 1600 宽 q86，页面显示宽度 ≤700px 定稿文字清晰；4K 屏放大到原尺寸看 PPT 小字可能略软，属质量 / 体积权衡。
6. **og 卡徽标**：两枚校徽无可复用单独素材，分享卡仅用项目 cube 标识，未补绘校徽；如后续拿到校徽 PNG 可再叠入居中安全区。
7. **环境回退**：`mine-depth.png` 等原图已不在 CSS 引用链中；极老旧不支持 WebP 的环境背景会落到 `--bg` 深色底，不影响正文可读。
