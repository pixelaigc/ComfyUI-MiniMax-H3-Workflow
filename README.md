# MiniMax H3 首尾帧视频工作流 · ComfyUI

将首帧、尾帧和文字描述组合成视频，并解码音频、合成带音轨的视频输出。模型加载、Turbo LoRA、采样、分辨率计算和音视频解码封装在一个子流程中，常用参数可直接在外层面板调整。

**工作流文件：[video_minimax_h3_t2v_0909.json](./video_minimax_h3_t2v_0909.json)**

> 此工作流支持首尾帧视频、仅首帧图生视频和文生视频三种用法。当前保存的配置连接了首帧、尾帧两个图像输入。

## 总控面板图解

下图标注了首尾帧接口、提示词、模型选择、LoRA、采样步数、时长、分辨率及保存视频节点，并附三种生成模式的切换方法。点击图片可查看大图。

[![MiniMax H3 总控面板使用指南](./assets/minimax-h3-control-panel-guide.png)](./assets/minimax-h3-control-panel-guide.png)

> 图解展示的是另一组示例设置（6 步、目标 5 秒，并外露 `strength`）；本仓库 JSON 保存的是 8 步、目标 6 秒，`strength` 在子流程内部。图解用于说明参数含义，实际默认值以工作流和下方参数表为准。

## 功能

- 首尾帧图像输入，配合提示词描述动作、镜头和声音。
- 外层直接选择基础模型、文本编码器、视频 VAE、音频 VAE 和 Turbo LoRA。
- 外层调整步数、目标时长、随机种子、宽高比和百万像素。
- 内置分辨率计算，宽高对齐至 32 的倍数。
- 24 fps 视频输出，自动对齐模型所需帧数。
- 视频与音频分别解码后合成，由 `SaveVideo` 保存。

## 环境与依赖

需要支持 MiniMax H3 和子图功能的 ComfyUI。JSON 记录的前端版本是 **1.47.11**，这是导出环境信息，不代表已验证的最低版本。

额外安装 [ComfyUI-MiniMax-H3-Turbo](https://github.com/Larryvrh/ComfyUI-MiniMax-H3-Turbo)，它提供本工作流使用的 `MiniMaxH3TurboLoRA` 节点。

在 ComfyUI 目录执行以下命令，安装后重启：

```bash
git clone https://github.com/Larryvrh/ComfyUI-MiniMax-H3-Turbo custom_nodes/ComfyUI-MiniMax-H3-Turbo
```

`MiniMaxH3ImageToVideo`、`ResolutionSelector`、`ComfyMathExpression` 已在本次核对的本机 ComfyUI 核心代码中提供。若这些节点缺失，请检查 ComfyUI 版本和启动日志。

本文件没有记录显卡型号、显存峰值、后端提交版本和运行耗时，因此不提供最低显存或速度承诺。

## 模型下载与目录

以下为 JSON 当前选择的文件。基础模型链接来自工作流元数据，LoRA 来源来自节点包说明。

| 用途 | 模型文件 / 下载入口 | 存放目录 |
| --- | --- | --- |
| 基础模型 | [minimax_h3_fl2va_pruned_int8_convrot.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_fl2va_pruned_int8_convrot.safetensors) | `ComfyUI/models/diffusion_models/` |
| 文本编码器 | [qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) | `ComfyUI/models/text_encoders/` |
| 视频 VAE | [minimax_h3_video_vae_fp16.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors) | `ComfyUI/models/vae/` |
| 音频 VAE | [minimax_h3_audio_vae_fp32.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors) | `ComfyUI/models/vae/` |
| Turbo LoRA | `minimax_h3_turbo_4step_ema_ckpt850.safetensors`，见 [LoRA 模型仓库](https://huggingface.co/larryvrh/MiniMax-H3-Turbo-Lora) | `ComfyUI/models/loras/` |

准备好文件后刷新模型列表或重启，在总控面板确认各模型选项。如下载的是其他检查点，请按对应模型说明配置，不应将不同版本视为等效默认值。

## 快速开始

1. 下载本仓库的工作流 JSON，拖入 ComfyUI 画布。
2. 安装所需节点，准备模型，确认没有缺失节点或模型报错。
3. 在左侧两个图像加载节点上传图片：上方接 `first_frame`，下方接 `last_frame`。
4. 在 `Image to Video (MiniMax H3)` 面板填写提示词，描述首尾帧之间的动作、镜头变化与声音。
5. 确认 LoRA、步数、时长和分辨率，点击运行。
6. 在右侧 `SaveVideo` 查看输出。

**JSON 不包含图片本体。** 文件保存了 `HPmuyFwbYAALgM9.jfif` 和 `HPmu4ASbQAAVr8D.jfif` 两个本地素材文件名，使用前需要重新上传或选择图片。

当前外层提示词为海怪掀船、泰坦踩踏场景，可以直接替换。编辑外层总控面板即可；子流程内部保存的旧示例文字不是当前外层输入。

## 三种输入模式

通过控制首帧、尾帧图像是否输入，可切换生成方式：

| 生成模式 | 首帧 `first_frame` | 尾帧 `last_frame` | 用法 |
| --- | --- | --- | --- |
| 首尾帧视频 | 输入 | 输入 | 以两张图片约束视频的起始与结束画面 |
| 图生视频 | 输入 | 绕过 / 不输入 | 仅以首帧图片为起点，结合提示词生成视频 |
| 文生视频 | 绕过 / 不输入 | 绕过 / 不输入 | 不使用图像条件，仅通过提示词生成视频 |

**仅保留首帧、绕过尾帧图像输入，就是图生视频；同时绕过首帧和尾帧图像输入，就是文生视频。**

这里的“绕过”指让对应图片不再传入子流程。可以断开相应的图像输入连线；使用节点绕过操作时，也应确认该接口没有继续收到图片。切换为文生视频后，无需准备首尾帧素材。

## 总控参数

| 参数 | 当前保存值 | 说明 |
| --- | --- | --- |
| `prompt` | 中文海怪 / 泰坦场景 | 内容、动作、镜头和声音描述 |
| `noise_seed` | `614289519754737` | 噪声种子；复现还需保持其他输入与环境一致 |
| `lora_name` | `minimax_h3_turbo_4step_ema_ckpt850.safetensors` | Turbo LoRA 文件 |
| `steps` | `8` | 实际采样步数；文件名中的 `4step` 不会自动将其改为 4 |
| `duration` | `6` | 目标秒数，实际帧数会对齐 |
| `aspect_ratio` | `16:9 (Widescreen)` | 目标画面比例 |
| `megapixels` | `0.4` | 当前计算分辨率为 `864 × 480` |
| `unet_name` / `clip_name` | 见模型表 | 基础模型与文本编码器 |
| `vae_name` / `audio_vae` | 见模型表 | 视频与音频 VAE |

### 子流程内部参数

| 设置 | 当前值 |
| --- | --- |
| 采样器 | `res_multistep` |
| 调度器 | `simple` |
| 降噪强度 | `1.0` |
| LoRA 强度 | `1.0` |
| LoRA `low_vram` | `false`（关闭） |
| 分辨率倍数 `multiple` | `32` |
| 视频帧率 | `24 fps` |
| 视频位深 | `8` |

这些设置需进入子流程修改。本工作流使用普通 `KSamplerSelect` 选择 `res_multistep`，没有使用节点包中的专用 Turbo Sampler。

### 分辨率与时长

分辨率节点按 `megapixels × 1024 × 1024` 计算目标面积，再将宽高分别舍入到 32 的倍数。实际比例和像素面积会略有偏差。

以下尺寸依据本次核对的节点算法计算，不代表所有尺寸都已完成生成测试：

| 宽高比 | 百万像素 | 计算分辨率 |
| --- | --- | --- |
| 16:9 | 0.2 | 608 × 352 |
| 16:9 | 0.4 | 864 × 480 |
| 16:9 | 0.6 | 1056 × 608 |
| 16:9 | 0.8 | 1216 × 672 |

时长换算表达式如下，`a` 为目标秒数：

```text
max(5, round(a * 24)) + (5 - (max(5, round(a * 24)) % 17)) % 17
```

帧数向上对齐至 `17k + 5`。默认 **6 秒会换算为 158 帧，按 24 fps 约为 6.58 秒**，输出时长不会严格等于输入值。

## 输出位置

`SaveVideo` 当前文件名前缀为 `video/MiniMax_H3_h9`，格式和编码器均为 `auto`。

使用默认输出目录时，视频保存在 `ComfyUI/output/video/` 下。如启动时自定义了输出目录，以实际配置为准。扩展名和编码由保存节点自动选择。

## 常见问题

**导入后有缺失节点**  
检查是否安装 Turbo 节点包；若缺失 H3、分辨率或数学表达式节点，检查 ComfyUI 版本及启动时的导入错误。

**模型列表找不到文件**  
检查存放目录、文件名，刷新列表或重启后重新选择。JSON 不会打包模型权重。

**图片不存在**  
在两个图像加载节点重新上传首帧、尾帧。作者的本地文件名无法替代实际图片。

**显存不足**  
降低 `megapixels` 或缩短 `duration`。也可进入子流程尝试开启 LoRA 的 `low_vram`；根据本机节点包说明，该设置可能影响量化模型的画面锐度，需比较实际结果。

**输出时长比设置值长**  
这是帧数对齐造成的，详见上方公式。

**声音异常**  
检查音频 VAE、ComfyUI 和 Turbo 节点版本是否匹配。此文件使用普通采样器路径，跨版本兼容性需要实际验证。

## 分享与说明来源

将本 README 和 JSON 放在仓库同一目录即可。预览图、示例视频与首尾帧素材可另行添加，模型权重通过上方来源下载。

本说明依据工作流 JSON 和本机节点源码整理，编写时未重新执行完整视频生成。模型与自定义节点的使用条款以各自项目为准；工作流仓库的许可证由作者另行指定。


