# Teaching页面设置说明

## 概述
Teaching页面已经简化，移除了复杂的课程子界面，现在只展示课程信息和syllabus下载链接。

## 当前结构
- **Current & Upcoming Courses**: 当前和即将教授的课程
- **Past Courses**: 过去教授的课程
- **Office Hours & Contact**: 办公时间和联系方式
- **Teaching Philosophy**: 教学理念
- **Additional Resources**: 额外资源

## 添加Syllabus PDF文件

### 1. 准备PDF文件
为每个课程准备一个syllabus PDF文件，建议命名为：
- `cs335-syllabus.pdf` - CS335课程大纲
- `cs101-syllabus.pdf` - CS101课程大纲
- `hpc-syllabus.pdf` - HPC课程大纲
- `ai-syllabus.pdf` - AI/ML课程大纲

### 2. 上传文件
将PDF文件上传到对应的目录：
- `files/cs335/cs335-syllabus.pdf`
- `files/cs101/cs101-syllabus.pdf`
- `files/hpc/hpc-syllabus.pdf`
- `files/ai/ai-syllabus.pdf`

### 3. 更新课程信息
在 `_pages/teaching.md` 中：
- 更新课程描述
- 调整学期信息
- 修改课程编号（如CS 4XX）
- 添加或删除课程

## 文件结构
```
files/
├── cs335/
│   ├── README.md
│   └── cs335-syllabus.pdf (需要添加)
├── cs101/
│   ├── README.md
│   └── cs101-syllabus.pdf (需要添加)
├── hpc/
│   ├── README.md
│   └── hpc-syllabus.pdf (需要添加)
└── ai/
    ├── README.md
    └── ai-syllabus.pdf (需要添加)
```

## 注意事项
- 确保PDF文件大小合理（建议小于10MB）
- 文件名要与teaching.md中的链接一致
- 可以随时添加新课程或移除旧课程
- 保持页面简洁，避免过度复杂化

## 自定义
如果需要进一步自定义：
- 修改 `_pages/teaching.md` 中的课程描述
- 调整CSS样式（如果需要）
- 添加更多课程或资源链接 