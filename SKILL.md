---
name: create-character-four-grid
description: 将用户提供的一张人物全身照生成为同一人物的四宫格参考图：正面半身、侧面半身、正面全身与背面全身，第三格默认模糊脸部，保留原图背景。用于“人物四宫格”“人生四格”“人物多视角参考图”等请求。
---

# 人物四宫格

用户只需提供一张人物全身照，即可直接生成一张完整四宫格。这里的“人生四格”指人物多视角参考图，不是人生阶段故事。沿用下列默认设置，用户明确提出的变更优先，无需再次确认已明确的需求。

## 运行环境适配

本技能是与平台和模型无关的生成规范，不依赖 Codex、特定工具名、API key 或命令行程序。调用宿主 agent 已连接且用户可用的图片生成/编辑工具；使用其真实参数，不假设其他平台也有同名工具。

- 需要能查看用户照片，并通过参考图生成或图片编辑保留人物身份与服装；只有文生图能力时不能保证完成本任务，应说明缺少参考图能力。
- 参考图支持本地路径、附件 ID、上传素材 ID 或可访问 URL 时，按宿主工具接受的方式提供。私有或临时的本地路径不能当作公开 URL。
- 优先同时传入本次人物照片与排版参考图，并标明角色。工具仅支持单张参考时，优先保留人物照片，使用下面的数值规范描述构图；说明排版参考未能作为图像输入，不把参考人物当成目标人物。
- 画布尺寸按工具支持的最接近 26:45 的竖向比例设置，并在提示词中保留 40:60 分排要求；不虚构工具支持的尺寸或参数。
- `agents/openai.yaml` 只是可选的宿主显示元数据；不识别它的 agent 可忽略，执行规范以本文件为准。用户无需使用特定的 `$` 调用语法，自然语言调用也适用。

## 默认成品

按从左到右、从上到下的顺序排布：

| 位置 | 视角与取景 | 脸部 |
| --- | --- | --- |
| 第一格·左上 | 正面半身，从头顶到腰部 | 清晰，保持原照身份特征 |
| 第二格·右上 | 侧面半身，默认面朝画面左侧 | 清晰，保持原照身份特征 |
| 第三格·左下 | 正面全身，从头顶到鞋底 | 默认强高斯模糊，仅覆盖脸部，五官不可辨认 |
| 第四格·右下 | 背面全身，从头顶到鞋底 | 自然背对镜头 |

- 同一人物、同一套服装，保留原照的发色、发型、体型、肤色、衣服、鞋子和已有配饰。不沿用任何历史示例人物的外观。
- 写实摄影，默认保留原图背景、场景元素、色调、光线方向与时间氛围，不擅自替换为白底或摄影棚。四格都呈现同一地点，半身格可对原背景自然裁切；人物转身时保持相机朝向与背景空间关系一致。仅在用户明确要求换背景时更换。自然站立，保留原图手持物和与其相符的姿势。
- 排版与上排人物比例按随技能附带的 [排版参考图](references/layout-reference.png) 执行。此图只控制构图、人物占比与取景，不提供身份、服装、背景或文字。
- 整体宽高比约 26:45（以随附蓝色毛衣人物四宫格为准）；两列各占 50%；横向分隔线在总高度约 40% 处，下排约 60%。上排每格宽高比约 0.72:1，下排每格约 0.48:1。用细分隔线，四格边缘对齐，不做四格等高布局。
- 上排为紧凑的头部至腰部特写，不是缩小的站姿照。头顶位于上排格子顶部约 2–4%，头顶至下巴约占该格高度 35–40%；头肩与躯干填满画面，下边缘在腰部附近，不展示臀部和大腿。正面格肩部宽度约占格宽 85–95%；侧面格使用与正面格一致的头部大小和取景尺度，头顶及下巴高度基本对齐。不为了展示手持物或完整手臂而缩小人物。
- 下排保持完整全身，头顶留白约为下排高度 2–4%，鞋底靠近底部但完整可见。
- 无文字、编号、水印或额外装饰。各格保留完整头顶；下排保留完整手部和鞋子。上排允许手臂、手持物随腰部取景自然出框，不为此拉远镜头。
- 默认选择模糊脸，保留完整头部。若用户明确选择“没有头部”，第三格采用头部在画面外的颈下取景，保留身体到鞋底，画面自然，不表现断颈、伤口或悬空头部。

## 执行

1. 检查当前上传图，明确哪张是人物来源、哪张仅是排版参考。以实际画面判断，不依赖附件排序或扩展名。只有缺少可用人物照片、存在多个候选人物而无法判断等阻碍生成的情况才询问。
2. 使用宿主的图片查看能力确认输入。HEIC 无法显示时，用环境已有的解码工具转换至工作目录中的 PNG/JPEG，并验证可解码、有有效像素尺寸；不要依赖特定操作系统命令。缺少转换能力时请求可读取的图片格式，不把空文件交给生成器。
3. 查看随技能附带的 `references/layout-reference.png`，将它作为仅用于排版和上排人物尺度的辅助参考，按上面的运行环境适配规则调用宿主图片生成/编辑工具。不以程序绘图代替生成人物，不擅自新增付费服务或上传到未获用户授权的其他服务。
4. 以人物照片为身份、服装与背景来源。生成提示词应明确列出本次照片的主要场景元素和光照特征，并要求四格保留这些元素。附加参考图仅控制布局和视角，不能把参考人物的服装、脸或风格混进成品。侧面、背面是依据单张照片的合理生成，不声称是实拍还原。
5. 一次生成一张包含四格的完整图片。用户没有要求时，不另生成四张独立图，也不增加其他版本。
6. 检查结果：恰好四格；整体宽高比约 26:45、上下排约 40:60；上排头顶留白少、头部占格高约 35–40%，裁到腰部而非臀腿，正侧面头部尺度一致；顺序正确；四格人物、服装一致，原图背景、关键场景元素和光照氛围得到保留；第三格位于左下且脸部充分模糊；另外两张可见脸保持清晰；全身格鞋子完整；没有多余肢体或文字。若有明显偏差，只针对该问题修正，并保留其他约束。
7. 展示成品，简短说明第三格已处理。若用户要求保存或环境要求交付文件，将最终成品复制至相应输出目录并提供链接。生成工具不可用时明确说明，不假称已经生成。

## 可复用生成提示词

按实际人物和用户变更替换方括号内容，不将方括号原样发送；附带的排版参考图默认作为辅助输入；它不是本次人物或背景来源。

```text
Use case: identity-preserve.
Create one photorealistic character reference sheet in a clean 2x2 grid. Overall canvas width:height approximately 26:45. Two equal-width columns; horizontal divider at 40 percent of total height, leaving 60 percent for the bottom row. Follow the supplied layout reference for panel geometry and especially the large subject scale in the top portraits.
The supplied person photo is the ONLY identity, outfit and background source: [brief description of this person's actual hair, clothing, shoes and accessories]. Preserve the recognizable face, hairstyle, skin tone, natural body proportions, clothes and shoes across all four views.
The bundled layout reference is ONLY for panel geometry, camera framing and subject scale. Do not transfer its person, blue sweater, brown trousers, folder, backdrop, color grade or text. The current user photo supplies identity, outfit and background.
Reading order:
TOP LEFT: tight frontal crown-to-waist portrait, clear face. Crown only 2–4 percent below the top edge; crown-to-chin occupies approximately 35–40 percent of this panel height; shoulder span approximately 85–95 percent of panel width. Crop at the waist, excluding hips and thighs. Match the close camera distance and large head scale of the layout reference.
TOP RIGHT: tight left-facing side-profile crown-to-waist portrait, clear face. Same head size and camera scale as TOP LEFT, crowns and chins aligned, crown-to-chin approximately 35–40 percent of panel height. Do not zoom out to show hands or props; they may naturally fall outside the waist crop.
BOTTOM LEFT: front full-body neutral standing pose, head to toe, arms relaxed. Strong soft Gaussian blur ONLY over the face, so facial features are unrecognizable. Keep hair, clothing, hands, body and shoes crisp; preserve existing held objects and the corresponding natural hand pose.
BOTTOM RIGHT: rear full-body neutral standing pose, head to toe, same person and outfit from behind.
Preserve the original location and background in ALL four panels: [describe the actual background elements, colors, lighting direction and atmosphere in this input photo]. Maintain the original scene geometry and camera-facing background while rotating the person for side and rear views. Crop the same environment naturally for the upper portraits. Keep original light direction, time-of-day atmosphere and realistic contact shadows. Do not replace the background with white, a studio or another location unless explicitly requested. Two equal columns, top row 40 percent of canvas height, bottom row 60 percent, matching the supplied layout reference. Thin light gray grid dividers. Minimal headroom in upper portraits as specified. Full-body panels have 2–4 percent headroom and complete shoes near the bottom edge, without clipping. Exactly four panels, one person per panel, no text, labels, numbers, watermark or added accessories. Infer side and rear details consistently from the supplied person photo.
```

用户指定第三格无头取景时，把第三格的描述整体替换为：

```text
BOTTOM LEFT: front-facing neck-down outfit view, framing starts below the chin with the head entirely outside the image and includes the complete body through the soles of the shoes. Natural photographic crop, no visible cut neck or injury. Preserve clothing, hands, body proportions and footwear.
```
