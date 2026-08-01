---
title: "Notes on Camera Calibration"
date: 2026-08-01
categories: [Computational Optics]
tags: [camera calibration, bundle adjustment, python]
layout: single
author_profile: true
---

这里写正文，支持标准 Markdown

## 1. Introduction
相机标定是三维重建基础……

### 公式支持（LaTeX）
$$
\boldsymbol{p} = K[R|t]\boldsymbol{P}
$$

## 代码块
```python
import cv2
# 相机标定代码

## 头部 Front Matter 字段说明（`---`包裹区域）
- `title`：网页显示标题
- `date`：发布时间，建议和文件名日期保持一致
- `categories`：大分类，一个或多个
- `tags`：关键词标签
- `layout: single`：固定，Minimal Mistakes标准博文布局
- `author_profile: true` 侧边栏保留你的个人信息（和主页统一）

# 3、两种发布模式
## 方式A：公开博文（所有人可见）
直接放入 `_posts/`，push 到github，等待1分钟自动编译上线。

## 方式B：草稿（不想立刻发布）
新建文件夹 `_drafts/`
草稿文件**不需要日期前缀**，例如 `camera-notes.md`
本地预览草稿命令（如果你本地装了jekyll）
```bash
bundle exec jekyll serve --drafts
