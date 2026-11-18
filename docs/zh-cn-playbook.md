# TikTok AI 图生视频与宠物热舞提示词工程实战手册（2026.11 实测版）

[English Playbook (../README.md)](../README.md) · **简体中文版** · [在线交互版手册 (GitHub Pages)](https://siuserxiaowei.github.io/ai-photo-to-video-prompt-recipes/) · [TikTok AI 爆款趋势雷达](https://angrydance.com/zh/trends)

> **维护者与利益相关声明**：本开源配方库由 [@siuserxiaowei](https://github.com/siuserxiaowei) 与 [Angry Dance 中文工作台](https://angrydance.com/zh) 工程团队共同维护并以 MIT 协议开源。库内所有提示词（Prompt）配方均可直接复制到你自己的 ComfyUI、fal.ai、可灵（Kling 2.5 Turbo）或 Google AI Studio（Nano Banana 2）原生接口中独立运行；文末及表格所附 `angrydance.com/zh` 链接为在线样片参考与免提示词一键同款入口。

---

## 一、为什么复杂整活梗直接用「单步图生视频（I2V）」失败率超过 70%？

在复现 TikTok、抖音与小红书近期爆火的 AI 视觉热梗（例如：**小狗顶着洗发泡泡在美容台打功夫**、**十几个迷你黄帽小人爬梯子洗你的真车**、**老爸半夜在家门口监控里跟孟加拉虎过招**）时，新手最常踩的大坑就是：**把手机相册里的日常照片直接丢进图生视频模型，再塞一段 80 字的场景大改提示词。**

我们在 2026 年 11 月针对 **480 组对照样本** 实测了 `Kling 2.5 Turbo`、`Hailuo 2.3 Fast`、`Muse Video` 单步直出与 **`Image-Edit -> I2V` 两步串联管线**（`Google Nano Banana 2` 改图定帧 + `Kling 2.5 Turbo` 驱动动作），得到如下实测数据：

### 单步直出 vs. 两步管线（`Image-Edit -> I2V`）实测出片率对比

| 玩法类型 | 代表热梗 | 单步 I2V 一次成片率 (`N=40`) | 两步管线成片率 (`N=40`) | 单步 I2V 最常见翻车现象 |
| :--- | :--- | :---: | :---: | :--- |
| **原场景肢体律动**（不改背景与道具） | 愤怒之舞、狗狗跳舞、猫咪热舞、傲娇哼跺脚 | **82.5%** | **90.0%** | 镜头乱推拉假装在动；被遮挡的前爪或手掌在第 3 秒糊成残影 |
| **跨场景+新增道具**（宠物店/游轮甲板/草原） | 小狗大战美容师、海洋之心、骑狮子 | **25.0%** *(75% 翻车)* | **92.5%** | 前 2.5 秒卧室墙面强行融化成瓷砖，主角五官与毛色严重漂移 |
| **微缩比例+多智能体物理**（迷你小人+真车） | 微缩玩具工人洗车、3D 盒装手办破盒走出 | **17.5%** *(82.5% 翻车)* | **87.5%** | 小人凭空悬浮或越洗越大，真车引擎盖像橡皮泥一样变形 |
| **特殊镜头光学模拟**（红外夜视监控/高空鱼眼） | 监控打老虎、百层无边泳池滑落整蛊 | **27.5%** *(72.5% 翻车)* | **90.0%** | 彩色自拍与黑白红外提示词冲突，老虎身体与人物躯干粘连 |
| **跨媒介双我合成写真**（铅笔素描+全彩真人） | 素描自我拥抱（Self Hug Sketch） | *不适用* | **95.0%** *(单步精修)* | 普通文生图直接把本人换脸成陌生欧美模特，失去「拥抱自己」的意义 |

### 核心工程铁律：空间合成（Spatial）与时间运动（Temporal）彻底解耦

视频扩散模型在 5 秒（120 帧）内的时空注意力预算是有限的：
- **凡是不改环境、只动肢体的舞蹈梗**：直接走 **单步 I2V**，用「主体锚定 + 明确动作动词 + 保持原貌约束 + 锁定固定机位」四段式结构即可稳出。
- **凡是涉及换场景、加泡沫/蓝宝石道具、改微缩比例、转红外夜视监控的整活梗**：必须拆成两步——先用 `Nano Banana 2` 生成一张保留原图人物/爱宠/爱车身份的 **第 0 帧完美合成图**，再把这张图送入 `Kling 2.5 Turbo` 只写纯粹的物理动作指令！

---

## 二、12 个实测爆款玩法速查表与中文样片入口

| 编号 | 玩法名称 | 管线架构 | 推荐模型组合 | 画幅 / 时长 | 在线实测样片与中文入口 |
| :---: | :--- | :--- | :--- | :---: | :--- |
| 01 | **愤怒之舞（Angry Dance）** | 单步 I2V | `Kling 2.5 Turbo` | `9:16` · 5s | [愤怒之舞生成器](https://angrydance.com/zh/effects/angry-dance) |
| 02 | **宠物狗狗跳舞（Dog Dance AI）** | 单步 I2V | `Kling 2.5` / `Hailuo 2.3` | `9:16` · 5s | [狗狗跳舞生成器](https://angrydance.com/zh/effects/dog-dance-ai) |
| 03 | **傲娇猫咪跳舞（Cat Dance AI）** | 单步 I2V | `Kling 2.5 Turbo` | `9:16` · 5s | [猫咪跳舞生成器](https://angrydance.com/zh/effects/cat-dance-ai) |
| 04 | **小狗大战美容师（Groomer vs Dog）** | **两步串联** | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [小狗大战美容师](https://angrydance.com/zh/effects/groomer-vs-dog-ai) |
| 05 | **微缩玩具工人洗车（Mini Car Wash）** | **两步串联** | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [微缩工人洗车特效](https://angrydance.com/zh/effects/mini-workers-car-wash-ai) |
| 06 | **海洋之心变装（Heart of the Ocean）** | **两步串联** | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [海洋之心搞笑滤镜](https://angrydance.com/zh/effects/heart-of-the-ocean-ai) |
| 07 | **家门口监控打老虎（Tiger Fight CCTV）** | **两步串联** | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [监控打老虎整蛊视频](https://angrydance.com/zh/effects/tiger-fight-cctv-ai) |
| 08 | **素描自我拥抱（Self Hug Sketch）** | 单步改图 | `Nano Banana 2` (1K–4K) | `9:16` / `4:5` | [素描自我拥抱写真](https://angrydance.com/zh/effects/self-hug-sketch-ai) |
| 09 | **3D 盒装手办破盒走出（Action Figure）** | **两步串联** | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [3D 手办破盒视频](https://angrydance.com/zh/effects/ai-action-figure-video) |
| 10 | **草原霸气骑狮子（Lion Riding AI）** | **两步串联** | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [骑狮子视频特效](https://angrydance.com/zh/effects/lion-riding-ai) |
| 11 | **傲娇「哼」跺脚舞（Tsundere Stomp）** | 单步 I2V | `Kling 2.5 Turbo` | `9:16` · 5s | [傲娇跺脚舞特效](https://angrydance.com/zh/effects/tsundere-hmph-stomp-ai) |
| 12 | **Give It All to Me 卡点律动** | 单步 I2V | `Kling 2.5` / `Muse Video` | `9:16` · 5s | [Give It All to Me 律动](https://angrydance.com/zh/effects/give-it-all-to-me-ai) |

> 想查看全部 **23 款已上线特效** 可直接访问 [Angry Dance 中文特效大厅](https://angrydance.com/zh/effects)，需要自己粘贴下方英文原版 Prompt 自由调参可使用 [多模型图生视频工作台](https://angrydance.com/zh/tools/image-to-video)。

---

## 三、精选高难爆款配方的分步拆解与可直接复制 Prompt

### 配方 A：愤怒之舞（The Angry Dance — 复活王牌魔性重踏舞）
- **选照要点**：半身至膝盖或全身照最佳，双手自然垂放不要插兜，脚底或下身留一点余量。
- **在线免提示词入口**：[`angrydance.com/zh/effects/angry-dance`](https://angrydance.com/zh/effects/angry-dance)

```text
The person in the photo performs the viral Angry Dance with intense comedic attitude: furrowed brows, dramatic furious scowl, pointing one finger sharply toward the camera on beat, clenching fists at chest level, and hitting heavy rhythmic foot stomps side to side. Keep the person's exact facial features, hairstyle, outfit, and background unchanged from the original photo. Locked static camera, crisp natural lighting, grounded foot contact.
```

---

### 配方 B：小狗大战美容师（Groomer vs Dog — 两步法防融化配方）
- **避坑重点**：千万不要拿卧室里的狗照片一步生视频！先用 Step 1 把狗狗放上不锈钢美容台并加上头顶泡沫尖尖，再用 Step 2 驱动后腿站立功夫招架。
- **在线免提示词入口**：[`angrydance.com/zh/effects/groomer-vs-dog-ai`](https://angrydance.com/zh/effects/groomer-vs-dog-ai)

```text
[Step 1 - Nano Banana 2 场景合成]
Place the exact pet from the uploaded photo upright on a stainless-steel pet grooming salon table with a fluffy peak of white shampoo foam on top of its head. Beside the table stands a surprised pet groomer in a waterproof apron holding a handheld shower sprayer. Bright realistic pet salon background with tiled walls, candid smartphone video aesthetic, preserving the pet's exact face and coat markings.

[Step 2 - Kling 2.5 Turbo 5s 视频驱动]
Comedic handheld phone video in the pet salon: as the groomer reaches toward the pet with the sprayer, the foam-topped pet suddenly pops up into a hilarious martial-arts stance on its hind legs, throwing quick playful kung-fu paw blocks and dodges while shampoo bubbles bounce slightly. The groomer leans back laughing in surprise. Natural handheld micro-shake, crisp fur and foam physics.
```

---

### 配方 C：微缩玩具工人洗车（Mini Toy Workers Car Wash — 锁死真车刚体结构）
- **避坑重点**：车主最在意「这必须是我的车，连轮毂和包围都不能变」。Step 1 明确声明 `Keep the exact vehicle, paint color, wheels... unchanged`，Step 2 强调 `Keep the car's geometry rigid` 即可完美锁死真车。
- **在线免提示词入口**：[`angrydance.com/zh/effects/mini-workers-car-wash-ai`](https://angrydance.com/zh/effects/mini-workers-car-wash-ai)

```text
[Step 1 - Nano Banana 2 微缩比例锚定]
Keep the exact vehicle, paint color, wheels, license-plate area, and parking environment from the uploaded photo unchanged. Add a bustling crew of 14 tiny 4-inch miniature workers wearing bright yellow hard hats and reflective vests climbing tiny silver ladders leaned against the bumper and hood, holding miniature pressure-washer wands and soapy sponges with patches of rich white car-wash foam on the hood and windshield. Photorealistic macro depth, natural shadows matching the scene.

[Step 2 - Kling 2.5 Turbo 5s 动态擦洗]
The crew of tiny yellow-helmet workers busily washes the full-size car: two workers on the hood spray fine white soap mist from mini hoses, others scrub the windshield in circular motions with tiny sponges, and workers climb steadily up and down the mini ladders along the fender. Keep the car's geometry, paint reflections, and background rigid and unchanged. Subtle handheld phone camera movement.
```

---

### 配方 D：素描自我拥抱（Self Hug Sketch — 3 天破 339 万播放的治愈爆款）
- **选照要点**：单人胸口以上清晰自拍或半身照，肩部轮廓不被杂物遮挡，表情自然放松。
- **在线免提示词入口**：[`angrydance.com/zh/effects/self-hug-sketch-ai`](https://angrydance.com/zh/effects/self-hug-sketch-ai)

```text
[Single-Pass Image-Edit - Nano Banana 2]
Create a poignant mixed-media portrait based on the uploaded photo: keep the original full-color photorealistic person in the foreground with their exact facial features, skin tone, hair, and clothing, eyes softly relaxed with a gentle comforted smile. Standing right behind them is a hand-drawn black-and-white graphite pencil sketch version of the exact same person on textured paper shading, wrapping both sketched arms warmly around the full-color person's shoulders in a tender self-hug. Seamless blend between realistic photography and expressive cross-hatched pencil linework.
```

---

### 配方 E：宠物后腿热舞（狗狗版 & 猫咪高冷摇摆版）
- **选照黄金四原则**：① 全身四爪入镜（留 10% 地板边距）；② 蹲下与宠物胸口齐平平视拍；③ 单图只留一只宠物；④ 避开深黑毛狗配黑地毯的低对比逆光环境。
- **在线免提示词入口**：[狗狗跳舞 AI](https://angrydance.com/zh/effects/dog-dance-ai) ｜ [猫咪跳舞 AI](https://angrydance.com/zh/effects/cat-dance-ai)

```text
[狗狗欢快版 - Direct I2V 5s]
The dog in the photo smoothly stands upright on its hind legs and dances happily to an upbeat funk rhythm, bouncing its hips left and right and waving both front paws playfully in the air with a joyful expression. Keep the dog's exact breed proportions, coat markings, collar, and original room background identical to the photo. Static camera, natural indoor lighting, no morphing.

[猫咪高冷版 - Direct I2V 5s]
The cat in the photo stands effortlessly upright on its hind legs and hits a smooth, rhythmic hip-sway dance from side to side, pulsing its front paws in time with the beat while keeping a deadpan, completely unbothered feline expression. Preserve the cat's exact fur pattern, whiskers, eye color, and surrounding room details from the photo. Static camera, steady lighting.
```

---

## 四、延伸工具与实时榜单

- **[Angry Dance 中文首页](https://angrydance.com/zh)**：聚合 `Kling 2.5 Turbo`、`Muse Video`、`Hailuo 2.3 Fast` 与 `Google Nano Banana 2`，支持一次性积分包（免包月订阅）、无水印下载与失败自动全额退还积分。
- **[2026 TikTok AI 爆款趋势雷达](https://angrydance.com/zh/trends)**：实时追踪 **556 个短视频视觉范式** 与 **348 个单图直出 AI 玩法** 的 24 小时播放增速榜。
- **[英文版完整 12 配方](../README.md)**：包含海洋之心、监控打老虎、3D 手办破盒、骑狮子、傲娇跺脚舞等全部 12 组完整英文 Prompt。
