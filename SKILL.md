---
name: create-character-four-grid
description: 将用户提供的一张人物全身照生成为同一人物的四宫格参考图：正面半身、侧面半身、正面全身与背面全身，第三格默认模糊脸部，保留原图背景和动作姿态。用于“人物四宫格”“人生四格”“人物多视角参考图”等请求。
---

# 人物四宫格

用户只需提供一张人物全身照，即可直接生成一张完整四宫格。这里的“人生四格”指人物多视角参考图，不是人生阶段故事。沿用下列默认设置，用户明确提出的变更优先，无需再次确认已明确的需求。

## 运行环境适配

本技能是与平台和模型无关的生成规范，不依赖 Codex、特定工具名、API key 或命令行程序。调用宿主 agent 已连接且用户可用的图片生成/编辑工具；使用其真实参数，不假设其他平台也有同名工具。

- 需要能查看用户照片，并通过参考图生成或图片编辑保留人物身份与服装；只有文生图能力时不能保证完成本任务，应说明缺少参考图能力。
- 参考图支持本地路径、附件 ID、上传素材 ID 或可访问 URL 时，按宿主工具接受的方式提供。私有或临时的本地路径不能当作公开 URL。
- 优先同时传入本次人物照片与排版参考图，并标明角色。工具仅支持单张参考时，优先保留人物照片，使用下面的数值规范描述构图；说明排版参考未能作为图像输入，不把参考人物当成目标人物。
- 先检查宿主是否具备裁切、局部模糊与拼图能力。支持精确合成时优先走分格准备与合成流程，最终画布采用 26:45 和 40:60 分排。只能让模型直接生成整图时，使用其支持的最接近比例，不虚构尺寸或参数，也不声称布局可精确保证。
- `agents/openai.yaml` 只是可选的宿主显示元数据；不识别它的 agent 可忽略，执行规范以本文件为准。用户无需使用特定的 `$` 调用语法，自然语言调用也适用。

## 默认成品

按从左到右、从上到下的顺序排布：

| 位置 | 视角与取景 | 脸部 |
| --- | --- | --- |
| 第一格·左上 | 正面半身，从头顶到腰部 | 清晰，保持原照身份特征 |
| 第二格·右上 | 侧面半身，默认面朝画面左侧 | 清晰，保持原照身份特征 |
| 第三格·左下 | 正面全身，从头顶到鞋底 | 默认强高斯模糊，仅覆盖脸部，五官不可辨认 |
| 第四格·右下 | 背面全身，从头顶到鞋底 | 自然背对镜头 |

- 同一人物、同一套服装，保留原照的发色、发型、体型、肤色、衣服、鞋子、已有配饰和动作姿态。不沿用任何历史示例人物的外观。
- 写实摄影，默认保留原图背景、场景元素、色调、光线方向与时间氛围，不擅自替换为白底或摄影棚。四格都呈现同一地点，半身格可对原背景自然裁切；人物转身时保持相机朝向与背景空间关系一致。仅在用户明确要求换背景时更换。保留原图动作、手持物及其与身体的接触关系，不统一改成普通站姿。
- 排版与上排人物比例按随技能附带的 [排版参考图](references/layout-reference.png) 执行。此图只控制构图、人物占比与取景，不提供身份、服装、背景、动作或文字。
- 整体宽高比约 26:45（以随附蓝色毛衣人物四宫格为准）；两列各占 50%；横向分隔线在总高度约 40% 处，下排约 60%。上排每格宽高比约 0.72:1，下排每格约 0.48:1。用细分隔线，四格边缘对齐，不做四格等高布局。
- 上排为紧凑的头部至腰部特写，不是缩小的站姿照。头顶位于上排格子顶部约 2–4%，头顶至下巴约占该格高度 35–40%；头肩与躯干填满画面，下边缘在腰部附近，不展示臀部和大腿。正面格肩部宽度约占格宽 85–95%；侧面格使用与正面格一致的头部大小和取景尺度，头顶及下巴高度基本对齐。不为了展示手持物或完整手臂而缩小人物。
- 下排保持完整全身，头顶留白约为下排高度 2–4%，鞋底靠近底部但完整可见。
- 无文字、编号、水印或额外装饰。各格保留完整头顶；下排保留完整手部和鞋子。上排允许手臂、手持物随腰部取景自然出框，不为此拉远镜头。
- 默认选择模糊脸，保留完整头部。若用户明确选择“没有头部”，第三格采用头部在画面外的颈下取景，保留身体到鞋底，画面自然，不表现断颈、伤口或悬空头部。

## 布局不稳定时的执行策略

- 支持本地或宿主精确拼图时，优先按 [分格生成与合成](references/split-generation.md) 准备四个格子，再合成一张最终图。第一格尽量裁切原图，第三格尽量复用原图并仅模糊脸；只生成确实缺少的视角。四张单格可以作为中间素材，最终仍只交付一张四宫格。
- 只有整图生图能力时，可用文末的整图提示词尝试一次。若上排变成全身、人物明显过小或分排接近 50:50，判为布局失败；有合成工具就切换分格流程，不反复依赖加长提示词重抽整图。没有合成能力则说明限制，不能宣称已精确复现参考比例。
- 单参考图工具始终优先输入当前人物照片；排版图由 agent 自己查看并提取数值，不为满足双参考要求而牺牲身份和动作来源，也不把两张图随意拼成一张条件图导致人物串入。
- 精确合成只能控制格子尺寸与位置；人物动作、身份和上排取景仍须逐格检查，不能把“拼图比例正确”视为全部合格。

## 原图动作锁定

四格是同一人物在同一瞬间的不同视角与取景，不是四个重新摆姿势的人物。除用户明确要求改动作外，人物照片是唯一动作来源；排版参考里的直立姿势、垂手、腿形和道具都不能迁移到目标人物。

- 生成前从原图提取具体姿态：头部倾斜与表情，肩线与躯干倾斜，双臂是否交叉及上下遮挡顺序，手部位置与接触对象，左右腿各自的弯曲方向、承重腿、脚尖方向和脚跟是否抬起。只描述可观察的信息；不能确认解剖左右时用“原图画面左侧的腿”等视图坐标，避免猜错或镜像。
- 第一格是原图上半身的同姿态近景；第三格是原图全身同姿态版本，仅脸部按要求模糊。保持原图可见的躯干角度、头部姿态、肩线、手臂、手、腿、脚与重心，不为“正面”二字强行端正身体。
- 第二格和第四格将同一姿态整体转到侧面或背面：保留关节弯曲、抱臂关系、承重侧及道具所在的解剖侧，按新视角合理推断遮挡。背面看不到正面的手、道具或前臂是正常遮挡，不得为展示它们而放下双臂、换手、换腿或改变姿势。单张图不能确定的隐藏细节只合理补全，不声称精确还原。
- 原图若是抱臂、叉腰、单腿弯曲、交叉腿、倚靠或坐姿，默认保留该动作；不要套用“自然站立”“双臂放松”“双腿伸直”等通用口令。原图本身为普通站姿时才沿用它。
- 原图动作优先于排版参考的人物轮廓和占比。上排可以自然裁掉腿或手持物，但画面内不能改变动作；若少量调整人物占比才能避免关键抱臂等动作被裁坏，应优先保留动作。
- 工具支持时，第一格优先由原图上半身裁切放大获得，第三格优先由原图保留全身并仅模糊脸部获得，再与生成的侧面、背面排入四格。若只能一次生成整张四宫格，必须将具体姿态要求写入每个格子的描述，不能只写一句“动作一致”。不要求为了满足工具不存在的能力而使用额外服务。

## 执行

1. 检查当前上传图，明确哪张是人物来源、哪张仅是排版参考。以实际画面判断，不依赖附件排序或扩展名。只有缺少可用人物照片、存在多个候选人物而无法判断等阻碍生成的情况才询问。
2. 使用宿主的图片查看能力确认输入。HEIC 无法显示时，用环境已有的解码工具转换至工作目录中的 PNG/JPEG，并验证可解码、有有效像素尺寸；不要依赖特定操作系统命令。缺少转换能力时请求可读取的图片格式，不把空文件交给生成器。
3. 查看随技能附带的 `references/layout-reference.png`，将它作为仅用于排版和上排人物尺度的辅助参考，按上面的运行环境适配规则调用宿主图片生成/编辑工具。不以程序绘图代替生成人物，不擅自新增付费服务或上传到未获用户授权的其他服务。
4. 以人物照片为身份、服装、背景与动作来源。先写出本次输入的具体姿态，再把它加入生成提示词和四格描述中。生成提示词应明确列出本次照片的主要场景元素和光照特征，并要求四格保留这些元素。附加参考图仅控制布局和视角，不能把参考人物的服装、脸、动作或风格混进成品。侧面、背面是依据单张照片的合理生成，不声称是实拍还原。
5. 按布局执行策略选择分格合成或整图生成。允许为合成准备四张中间单格，原图可直接复用或裁切，不要求每格重新生图。最终交付一张四宫格，不额外交付多个版本，除非用户要求。
6. 检查结果：恰好四格；整体宽高比约 26:45、上下排约 40:60；上排头顶留白少、头部占格高约 35–40%，裁到腰部而非臀腿，正侧面头部尺度一致；顺序正确；第一格和第三格与原图的动作一致，侧背面是该动作的合理转视角，而非重新摆姿势；四格人物、服装一致，原图背景、关键场景元素和光照氛围得到保留；第三格位于左下且脸部充分模糊；另外两张可见脸保持清晰；全身格鞋子完整；没有多余肢体或文字。动作检查需逐项对照抱臂/手位、躯干倾斜、弯腿、承重侧和脚跟状态；任何关键动作被改掉，即使脸和衣服正确也不通过。若有明显偏差，只针对该问题修正，并保留其他约束；工具无法保留时说明限制，不把不合格结果声称为已保持原动作。
7. 展示成品，简短说明第三格已处理。若用户要求保存或环境要求交付文件，将最终成品复制至相应输出目录并提供链接。生成工具不可用时明确说明，不假称已经生成。

## 整图生成提示词（仅限无合成能力时）

按实际人物和用户变更替换方括号内容，不将方括号原样发送；支持双参考时附带排版图作为辅助输入；仅支持单参考时只传人物照，将下列提示词里的“supplied layout reference”改为“numeric layout specification”，删除需要模型直接查看排版图的语句，保留全部尺寸与人物占比要求。分格流程使用独立单格提示词，不使用下面的整图模板。

```text
Use case: identity-preserve.
Create one photorealistic character reference sheet in a clean 2x2 grid. Overall canvas width:height approximately 26:45. Two equal-width columns; horizontal divider at 40 percent of total height, leaving 60 percent for the bottom row. Follow the supplied layout reference for panel geometry and especially the large subject scale in the top portraits.
The supplied person photo is the ONLY identity, outfit, background and POSE source: [brief description of this person's actual hair, clothing, shoes and accessories]. Preserve the recognizable face, hairstyle, skin tone, natural body proportions, clothes and shoes across all four views.
POSE LOCK: [describe the actual input pose: head/torso tilt, shoulder line, arm overlap and hand contacts, bent versus supporting leg, foot placement, heel lift, held-object contact]. Preserve this exact joint configuration, balance and expression. The four panels show the same frozen pose, not four newly posed people. Never replace crossed arms with hanging arms, a bent leg with a straight leg, or asymmetric balance with a symmetrical stance. Preserve the same anatomical limb roles through rotation; do not mirror them.
The bundled layout reference is ONLY for panel geometry, camera framing and subject scale, NEVER for pose. Do not transfer its person, blue sweater, brown trousers, folder, backdrop, color grade or text. The current user photo supplies identity, outfit and background.
Reading order:
TOP LEFT: tight front crown-to-waist crop of the ORIGINAL POSE, clear face. Preserve source head/shoulder/torso angles, visible arm crossing and hand contacts: [input upper-body pose]. Crown only 2–4 percent below the top edge; crown-to-chin occupies approximately 35–40 percent of this panel height; shoulder span approximately 85–95 percent of panel width. Crop at the waist, excluding hips and thighs. Match the close camera distance and large head scale of the layout reference.
TOP RIGHT: tight left-facing side-profile crown-to-waist portrait of the SAME FROZEN POSE, clear face. Rigidly rotate the posed subject into profile while preserving joint bends and arm/hand contacts: [input upper-body pose as seen from the side]. Same head size and camera scale as TOP LEFT, crowns and chins aligned, crown-to-chin approximately 35–40 percent of panel height. Do not zoom out to show hands or props; they may naturally fall outside the waist crop.
BOTTOM LEFT: front full-body version of the ORIGINAL PHOTO POSE, head to toe. Preserve exactly [input arm/hand configuration, torso lean, bent and weight-bearing leg, foot placement and heel lift]. Do not straighten the legs, lower the arms or rebalance the body. Strong soft Gaussian blur ONLY over the face, so facial features are unrecognizable. Keep hair, clothing, hands, body and shoes crisp; preserve existing held objects and the corresponding natural hand pose.
BOTTOM RIGHT: rear full-body view, head to toe, same person, outfit and FROZEN POSE. Rotate the same posed subject 180 degrees, maintaining [input arm configuration, joint bends, supporting leg and heel lift]. Allow natural rear-view occlusion, do not lower crossed arms, switch hands or straighten the bent leg.
Preserve the original location and background in ALL four panels: [describe the actual background elements, colors, lighting direction and atmosphere in this input photo]. Maintain the original scene geometry and camera-facing background while rotating the person for side and rear views. Crop the same environment naturally for the upper portraits. Keep original light direction, time-of-day atmosphere and realistic contact shadows. Do not replace the background with white, a studio or another location unless explicitly requested. Two equal columns, top row 40 percent of canvas height, bottom row 60 percent, matching the supplied layout reference. Thin light gray grid dividers. Minimal headroom in upper portraits as specified. Full-body panels have 2–4 percent headroom and complete shoes near the bottom edge, without clipping. Exactly four panels, one person per panel, no text, labels, numbers, watermark or added accessories. Infer side and rear details consistently from the supplied person photo.
```

用户指定第三格无头取景时，把第三格的描述整体替换为：

```text
BOTTOM LEFT: front-facing neck-down outfit view, framing starts below the chin with the head entirely outside the image and includes the complete body through the soles of the shoes. Natural photographic crop, no visible cut neck or injury. Preserve the ORIGINAL POSE including arm/hand contacts, torso tilt, bent versus supporting leg and foot placement, as well as clothing, body proportions and footwear.
```
