---
name: story-pipeline
version: 1.0.0
description: 故事生成→分镜→图片生成的完整流水线。当用户说"生成故事并画图"、"创作故事并制作图片"时使用。
---

# Story Pipeline - 故事创作完整流水线

把故事生成、分镜、图片生成串联成完整流程。

## 工作流程

```
用户输入主题
    ↓
1. story-generator 生成故事
    ↓
用户确认故事
    ↓
2. story-grid 生成分镜提示词
    ↓
用户确认格数
    ↓
3. image-bed 逐格生成图片
    ↓
自动上传S3
    ↓
输出：完整故事 + 分镜 + S3图片链接
```

---

## Step 1: 生成故事

使用 `story-generator` skill 生成故事。

**输出产物：** 完整故事文本

**保存路径：** `media/story/<故事名>.md`

```markdown
# 《故事标题》

[故事正文]

---

**故事结构**：
- 起：
- 承：
- 转：
- 合：

**建议格数**：X 格

**背景音乐**：《曲目名》
```

---

## Step 2: 分镜

使用 `story-grid` skill 生成分镜提示词。

**输出产物：** 每格的中文描述 + 英文生图提示词

---

## Step 3: 生成图片

对每一格：
1. 读取分镜提示词
2. 调用 image-bed 生成图片
3. 自动上传到 S3
4. 记录返回的 S3 URL

**输出产物：** 每格的 S3 图片链接

---

## 最终输出格式

```markdown
# 《故事标题》

## 故事正文
[...]

## 分镜

### 第1格：<场景名>
![Image](https://s3.xxx.com/...)

### 第2格：<场景名>
![Image](https://s3.xxx.com/...)

...

## 背景音乐
《曲目名》

---

**生成时间**：YYYY-MM-DD HH:mm
```

---

## 技术实现

### 图片生成

```bash
python3 ~/.agents/skills/image-bed/generate_image.py \
  "<英文提示词>" \
  "<输出路径>.png" \
  1120 1440 \
  --upload
```

### 保存结果

```python
{
    "local_path": "/path/to/image.png",
    "url": "https://s3.xxx.com/..."  # 上传后返回
}
```

---

## 注意事项

1. **先确认再下一步** — 每步都要等用户确认
2. **保存中间结果** — 每步完成后保存到本地文件
3. **S3链接持久化** — 把S3链接写入最终文档
4. **图片命名** — `故事名_第X格.png`

---

## 状态文件

流水线过程中维护状态文件：`media/story/.pipeline-state.json`

```json
{
  "story_title": "故事标题",
  "story_content": "...",
  "frames": [
    {
      "index": 1,
      "title": "场景名",
      "prompt": "英文提示词",
      "image_path": "/path/to/local.png",
      "s3_url": "https://s3.xxx.com/..."
    }
  ],
  "music": "背景音乐",
  "created_at": "2026-03-19 08:30"
}
```
