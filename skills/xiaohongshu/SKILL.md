---
name: xiaohongshu
description: |
  小红书内容工具。支持搜索、获取推荐、帖子详情、发布内容等功能。
  使用前确保 MCP 服务已启动。
---

# 小红书技能

## 前置要求

1. MCP 服务运行中：`~/.mioclaw/extensions/xiaohongshu/scripts/start-mcp.sh`
2. 已登录账号：`~/.mioclaw/extensions/xiaohongshu/scripts/status.sh`

## 使用方式

```bash
# 进入技能目录
cd ~/.mioclaw/extensions/xiaohongshu/scripts

# 检查登录状态
./status.sh

# 搜索内容
./search.sh "关键词"

# 获取推荐
./recommend.sh

# 获取帖子详情 (需要 feed_id 和 xsec_token)
./post-detail.sh <feed_id> <xsec_token>

# 发表评论
./comment.sh <feed_id> <xsec_token> "评论内容"

# 获取用户主页
./user-profile.sh <user_id>

# 热点跟踪
./track-topic.sh "话题" --limit 10
```

## MCP 工具直接调用

```bash
# 通用 MCP 调用
./mcp-call.sh <tool_name> [json_args]

# 示例
./mcp-call.sh search_feeds '{"keyword":"AI"}'
./mcp-call.sh list_feeds '{}'
./mcp-call.sh publish_content '{"title":"标题","content":"正文","images":["图片URL"]}'
```

## 可用 MCP 工具

| 工具 | 描述 |
|------|------|
| check_login_status | 检查登录状态 |
| search_feeds | 搜索内容 |
| list_feeds | 获取首页推荐 |
| get_feed_detail | 获取帖子详情 |
| post_comment_to_feed | 发表评论 |
| reply_comment_in_feed | 回复评论 |
| user_profile | 获取用户主页 |
| like_feed | 点赞/取消 |
| favorite_feed | 收藏/取消 |
| publish_content | 发布图文笔记 |
| publish_with_video | 发布视频笔记 |

## 发布笔记

```bash
# 发布图文
./mcp-call.sh publish_content '{
  "title": "我的第一篇笔记",
  "content": "这是笔记正文内容",
  "images": ["https://example.com/image1.jpg"]
}'

# 注意：图片需要先上传到小红书支持的空间
```
