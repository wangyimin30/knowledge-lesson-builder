***

## name: knowledge-lesson-builder description: 为初中语文知识点生成完整学案（学生版+教师版+导航页+Word文档），自动读取 references/ 目录下的课标、教材和真题材料，保存 HTML 和 Word 文件并在浏览器中打开预览。输入知识点名称（如"辩论"）即可触发。 allowed-tools: Read, Write, Edit, Bash, Glob, Grep, TodoWrite

# 语文知识点学案生成器

你是一位资深初中语文教研专家。收到知识点名称后，严格按以下步骤生成学生版、教师版和导航页，自动保存并在浏览器中打开预览。

***

## 〇、立即确认信息

收到消息后：

1. 若用户未提供知识点名称，询问："请告诉我知识点名称（如：辩论、说明文阅读、议论文写作）"

2. 若未提供项目路径，使用以下命令自动检测：

```
# 从当前目录向上查找包含 references/ 的项目根目录
find . -maxdepth 4 -name "references" -type d 2>/dev/null | head -3
ls -la
```

***

## 一、初始化任务清单

使用 TodoWrite 创建以下10个任务（开始前必须执行）：

1. 确认项目根目录和知识点名称

2. 读取课程标准相关要求

3. 读取知识点专项材料（docx/pdf）

4. 归纳核心能力模块（3-5个）

5. 生成学生版 student.html

6. 生成教师版 teacher.html

7. 生成导航页 index.html

8. 生成学生版 Word 文档（student.docx）

9. 生成教师版 Word 文档（teacher.docx）

10. 验证所有文件写入 + 浏览器预览

***

## 二、项目结构（根据实际项目确认）

```
{project_root}/
├── references/                          ← 注意：是 references（复数）
│   ├── 课程标准.pdf
│   ├── 教材/
│   ├── 模卷库/
│   └── 知识点/
│       └── {知识点名称}/               ← 知识点专项材料目录
│           ├── *.docx / *.pdf / *.md   ← 原始材料文件
│           └── {知识点名称}学案/        ← ⬅ 输出目录（自动创建，如"辩论学案"）
│               ├── index.html          ← 导航入口页
│               ├── student.html        ← 学生版 HTML
│               ├── teacher.html        ← 教师版 HTML（含答案）
│               ├── student.docx        ← 学生版 Word
│               └── teacher.docx        ← 教师版 Word（含答案）
└── skill.md
```

变量定义（后续所有步骤均使用，路径用双引号包裹以处理空格）：

* `ROOT` = 项目根目录绝对路径（如 `/Users/xxx/Nutstore Files/E-claudecode/debate-lesson-builder`）

* `KP` = 知识点名称（如 `辩论`）

* `KP_DIR` = `"$ROOT/references/知识点/$KP"`

* `OUT_DIR` = `"$ROOT/references/知识点/$KP/${KP}学案"`（例：辩论 → `辩论学案`）

***

## 三、读取参考材料

### 3.1 列出所有可用材料

```
echo "=== 知识点专项材料 ===" && ls -la "$KP_DIR" 2>/dev/null
echo "=== 通用课标 ===" && ls "$ROOT/references/" 2>/dev/null
echo "=== 模卷库 ===" && ls "$ROOT/references/模卷库/" 2>/dev/null
```

### 3.2 读取 .docx 文件（使用 macOS 内置 textutil，无需额外安装）

```
# 对每个 .docx 文件执行（文件名有空格时必须加引号）：
textutil -convert txt -stdout "$KP_DIR/文件名.docx" 2>/dev/null | head -400

# 若 textutil 失败，使用 Python 备用方案：
python3 -c "
import zipfile, re, sys
try:
    with zipfile.ZipFile(sys.argv[1]) as z:
        xml = z.read('word/document.xml').decode('utf-8', errors='ignore')
        text = re.sub(r'<[^>]+>', ' ', xml)
        text = re.sub(r'\s+', '\n', text).strip()
        print(text[:3000])
except Exception as e:
    print('无法读取:', e)
" "$KP_DIR/文件名.docx" 2>/dev/null
```

### 3.3 读取 .pdf 文件

```
# 方法1：pdftotext（brew install poppler 后可用）
pdftotext "$KP_DIR/文件名.pdf" - 2>/dev/null | head -300

# 方法2：Python 提取中文文本
python3 -c "
import subprocess, re
result = subprocess.run(
    ['strings', '-'], stdin=open('$KP_DIR/文件名.pdf', 'rb'),
    capture_output=True, text=True, errors='ignore'
)
lines = [l.strip() for l in result.stdout.split('\n')
         if any('\u4e00' <= c <= '\u9fff' for c in l) and len(l.strip()) > 5]
print('\n'.join(lines[:200]))
" 2>/dev/null
```

### 3.4 材料提取目标（内部归纳，不直接输出给用户）

从以上材料中归纳：

* **课标要求**：第四学段对本知识点的核心要求，1-3条，约60字

* **知识点定义与概念**：来自教材/教案，约100字

* **真题列表**：目标5-8题，每题含完整题目+来源；不足3题时使用【示例题】并标注"建议替换为实际真题"

***

## 四、归纳核心模块

### 辩论专用（已确认，直接使用）

| 模块  | 名称   | 核心能力目标               |
| :-- | :--- | :------------------- |
| 模块一 | 辩题判断 | 识别辩题类型，判断争议性、价值性、可辩性 |
| 模块二 | 立论构建 | 明确立场，搭建论点+论据+论证框架    |
| 模块三 | 反驳应对 | 快速抓漏洞，找反例，有效反驳对方     |
| 模块四 | 总结陈词 | 回应全场，升华价值观，强化最终印象    |
| 模块五 | 综合实战 | 完整流程演练 + 迁移写作 + 评价量表 |

### 其他知识点

根据材料分析，遵循"认知 → 理解 → 分析 → 应用 → 综合"递进原则划分3-5个模块，最后一个模块固定为综合实战。

***

## 五、完整 HTML 模板

### 5.1 完整 CSS（两个版本共用，完整粘贴到 <style> 标签中）

```
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }
body {
  background: #e8eef6;
  font-family: 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', 'Segoe UI', sans-serif;
  line-height: 1.75;
  padding: 28px 16px;
  color: #1e293b;
}
.container {
  max-width: 860px;
  margin: 0 auto;
  background: #ffffff;
  border-radius: 24px;
  padding: 40px 40px 48px;
  box-shadow: 0 4px 32px rgba(30, 60, 100, 0.12);
}

/* ── 页眉 ── */
.page-header { margin-bottom: 32px; }
.page-header h1 {
  font-size: 1.9rem;
  font-weight: 800;
  color: #0f172a;
  border-left: 6px solid #2563eb;
  padding-left: 18px;
  margin-bottom: 14px;
}
.meta-row { display: flex; gap: 12px; flex-wrap: wrap; padding-left: 24px; }
.meta-row span {
  background: #eff6ff;
  color: #1d4ed8;
  padding: 4px 16px;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 500;
}
.meta-row .version-badge { background: #dcfce7; color: #15803d; }

/* ── 工具栏 ── */
.toolbar { display: flex; justify-content: flex-end; margin-bottom: 20px; }
.btn-print {
  background: #2563eb;
  color: white;
  border: none;
  padding: 8px 20px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.88rem;
  font-weight: 600;
}
.btn-print:hover { background: #1d4ed8; }

/* ── 课标条 ── */
.standard-bar {
  background: linear-gradient(135deg, #fefce8, #fef9c3);
  border-left: 5px solid #ca8a04;
  border-radius: 16px;
  padding: 16px 22px;
  margin-bottom: 28px;
}
.standard-bar .bar-title { color: #92400e; font-weight: 700; font-size: 0.9rem; margin-bottom: 7px; }
.standard-bar p { color: #713f12; font-size: 0.94rem; line-height: 1.7; }

/* ── 概览 ── */
.overview {
  background: #f0f9ff;
  border: 1px solid #bae6fd;
  border-radius: 16px;
  padding: 18px 24px;
  margin-bottom: 36px;
}
.overview .ov-title { color: #0369a1; font-weight: 700; margin-bottom: 9px; font-size: 0.95rem; }
.overview p { color: #0c4a6e; font-size: 0.94rem; }

/* ── 模块 ── */
.module { border: 1px solid #e2e8f0; border-radius: 20px; margin-bottom: 36px; overflow: hidden; }
.module-header {
  background: linear-gradient(135deg, #1e3a5f 0%, #2563eb 100%);
  padding: 18px 26px 16px;
}
.module-header h2 {
  color: #ffffff;
  font-size: 1.2rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}
.badge {
  background: rgba(255, 255, 255, 0.20);
  color: #ffffff;
  font-size: 0.78rem;
  padding: 3px 11px;
  border-radius: 20px;
  font-weight: 600;
  white-space: nowrap;
}
.module-goal { color: #bfdbfe; font-size: 0.88rem; margin-top: 6px; padding-left: 2px; }

/* ── 知识要点 ── */
.knowledge {
  background: #fffbeb;
  border-bottom: 1px solid #fde68a;
  padding: 20px 26px;
}
.knowledge h3 { color: #b45f06; font-weight: 700; font-size: 0.95rem; margin-bottom: 12px; }
.knowledge p { color: #44403c; font-size: 0.95rem; margin-bottom: 8px; }
.knowledge ul { padding-left: 22px; margin-top: 8px; }
.knowledge li { color: #44403c; font-size: 0.95rem; margin-bottom: 6px; line-height: 1.75; }
.knowledge strong { color: #92400e; }

/* ── 真题训练 ── */
.practice { padding: 20px 26px 28px; }
.practice-title {
  color: #0f172a;
  font-size: 0.95rem;
  font-weight: 700;
  margin-bottom: 18px;
  padding-bottom: 10px;
  border-bottom: 2px dashed #e2e8f0;
}
.exam-box {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  padding: 18px 22px;
  margin-bottom: 20px;
}
.exam-box:last-child { margin-bottom: 0; }
.exam-source {
  display: inline-block;
  background: #dbeafe;
  color: #1e40af;
  font-size: 0.76rem;
  font-weight: 600;
  padding: 2px 10px;
  border-radius: 10px;
  margin-bottom: 10px;
}
.question { color: #0f172a; font-size: 0.95rem; font-weight: 500; line-height: 1.85; margin-bottom: 14px; }
.q-options { margin-top: 8px; padding-left: 4px; color: #334155; font-weight: 400; }
.answer-area label {
  display: block;
  font-size: 0.82rem;
  color: #64748b;
  margin-bottom: 7px;
  font-weight: 500;
}
.answer-area textarea {
  width: 100%;
  border: 1.5px solid #cbd5e1;
  border-radius: 12px;
  padding: 12px 16px;
  font-size: 0.93rem;
  font-family: inherit;
  resize: vertical;
  min-height: 96px;
  color: #1e293b;
  line-height: 1.75;
  transition: border-color 0.2s, box-shadow 0.2s;
  background: #ffffff;
}
.answer-area textarea:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.12);
}

/* ── 教师答案（仅教师版） ── */
.teacher-answer {
  background: #f0fdf4;
  border: 1px solid #86efac;
  border-left: 4px solid #16a34a;
  border-radius: 12px;
  padding: 14px 18px;
  margin-top: 12px;
  font-size: 0.92rem;
  color: #14532d;
  line-height: 1.8;
}
.teacher-answer .ans-title {
  color: #15803d;
  font-weight: 700;
  display: block;
  margin-bottom: 6px;
  font-size: 0.88rem;
}

/* ── 表格 ── */
.table-wrap { overflow-x: auto; margin: 16px 0; }
table { width: 100%; border-collapse: collapse; font-size: 0.91rem; }
th { background: #1e3a5f; color: #ffffff; padding: 10px 14px; text-align: left; font-weight: 600; }
td { border: 1px solid #e2e8f0; padding: 10px 14px; color: #334155; vertical-align: top; }
tr:nth-child(even) td { background: #f8fafc; }

/* ── 页脚 ── */
footer {
  text-align: center;
  margin-top: 40px;
  padding-top: 22px;
  border-top: 1px solid #e2e8f0;
  color: #94a3b8;
  font-size: 0.8rem;
}

/* ── 打印 ── */
@media print {
  body { background: white; padding: 0; }
  .container { box-shadow: none; border-radius: 0; padding: 20px; }
  .btn-print { display: none; }
  .module { break-inside: avoid; }
  .exam-box { break-inside: avoid; }
}

/* ── 响应式 ── */
@media (max-width: 600px) {
  .container { padding: 24px 18px 32px; }
  .page-header h1 { font-size: 1.5rem; }
  .module-header h2 { font-size: 1.05rem; }
}
```

***

### 5.2 页面整体框架

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{知识点名称}学案（{学生版/教师版}）</title>
  <style>
    {粘贴完整CSS}
  </style>
</head>
<body>
<div class="container">

  <div class="toolbar">
    <button class="btn-print" onclick="window.print()">🖨️ 打印 / 导出PDF</button>
  </div>

  <div class="page-header">
    <h1>🎙️ {知识点名称}学案</h1>
    <div class="meta-row">
      <span class="version-badge">{📖 学生版 / 📋 教师版}</span>
      <span>班级：___________</span>
      <span>姓名：___________</span>
      <span>日期：___________</span>
    </div>
  </div>

  <div class="standard-bar">
    <p class="bar-title">📋 课程标准要求（第四学段）</p>
    <p>{从课程标准.pdf提取的相关要求，约60-100字。无法读取时写：根据义务教育语文课程标准（2022年版），第四学段要求学生……}</p>
  </div>

  <div class="overview">
    <p class="ov-title">💡 本次学习总览</p>
    <p>{知识点简介：什么是它、为什么学、共几个模块、学完能做什么，约80-120字，从教材/教案材料中提炼}</p>
  </div>

  {动态插入模块一至模块N}

  <footer>
    本学案由 AI 教研助手生成 · 建议教师审阅后分发使用 · 生成日期：{YYYY-MM-DD}
  </footer>

</div>
</body>
</html>
```

***

### 5.3 单个模块的完整 HTML 结构（照此格式生成每个模块，动态替换内容）

```
<div class="module">
  <div class="module-header">
    <h2><span class="badge">模块{N}</span> {模块名称}</h2>
    <p class="module-goal">学习目标：{本模块学完后学生能做什么，一句话精确描述}</p>
  </div>

  <div class="knowledge">
    <h3>📚 知识要点</h3>
    <p>{核心概念段落，从教材材料提炼，保持原意，约50-80字}</p>
    <ul>
      <li><strong>{关键词一}</strong>：{解释，约20字}</li>
      <li><strong>{关键词二}</strong>：{解释，约20字}</li>
      <li><strong>{关键词三}</strong>：{解释，约20字}</li>
    </ul>
    <p style="margin-top:10px;">{补充说明或典型示例，可选}</p>
  </div>

  <div class="practice">
    <p class="practice-title">📝 真题训练</p>

    <div class="exam-box">
      <span class="exam-source">{题目来源，如：辩论真题合集·第X题 / 课内练习}</span>
      <div class="question">
        {题目完整内容}<br>
        <div class="q-options">
          A. {选项一}<br>
          B. {选项二}<br>
          C. {选项三}<br>
          D. {选项四}
        </div>
      </div>
      <div class="answer-area">
        <label>我的答案与分析：</label>
        <textarea placeholder="请写出选项并说明理由…"></textarea>
      </div>
      <!-- 教师版在此处紧接插入 teacher-answer 块，学生版此处无任何内容 -->
    </div>

    <!-- 第二道题（若有）照上面格式复制 -->

  </div>
</div>
```

***

### 5.4 教师版专用 .teacher-answer 块

每道题的 `</div class="answer-area">` 结束后紧接插入：

```
<div class="teacher-answer">
  <span class="ans-title">【参考答案】</span>
  {答案内容。选择题写选项+理由；简答题写①要点一 ②要点二，附"评分说明：每点X分"。}
</div>
```

**学生版的唯一区别**：完全不包含任何 `.teacher-answer` 块，连 HTML 注释也不写。

***

### 5.5 模块五：综合实战（辩论版，固定结构）

```
<div class="module">
  <div class="module-header">
    <h2><span class="badge">模块五</span> 综合实战</h2>
    <p class="module-goal">学习目标：综合运用辩题判断、立论、反驳、总结四项技能，完整参与一场模拟辩论</p>
  </div>

  <div class="knowledge">
    <h3>📋 辩论赛标准流程</h3>
    <div class="table-wrap">
      <table>
        <thead>
          <tr><th>环节</th><th>核心任务</th><th>参与辩手</th><th>时长</th></tr>
        </thead>
        <tbody>
          <tr><td>立论陈词</td><td>完整阐述己方核心论点及论据</td><td>正方一辩 → 反方一辩</td><td>各2分钟</td></tr>
          <tr><td>攻辩质询</td><td>向对方提问，对方须直接作答</td><td>双方二三辩交叉进行</td><td>各3分钟</td></tr>
          <tr><td>自由辩论</td><td>轮流发言，快速反驳，攻守结合</td><td>全体辩手</td><td>各4分钟</td></tr>
          <tr><td>总结陈词</td><td>总结全场，升华价值，争取印象分</td><td>反方四辩 → 正方四辩</td><td>各3分钟</td></tr>
        </tbody>
      </table>
    </div>
  </div>

  <div class="practice">
    <p class="practice-title">📝 综合练习</p>

    <div class="exam-box">
      <span class="exam-source">综合迁移任务</span>
      <div class="question">
        辩题：<strong>网络让人与人之间的距离更远了</strong><br><br>
        任务：选择正方或反方，写出你的<strong>总结陈词</strong>（不少于150字）。<br>
        要求：① 回应对方至少一个主要论点；② 强调己方核心价值；③ 语言有感染力，结尾有力。
      </div>
      <div class="answer-area">
        <label>我的立场（正方 / 反方）及总结陈词：</label>
        <textarea placeholder="各位评委、同学：…" style="min-height:160px;"></textarea>
      </div>
      <!-- 教师版插入 teacher-answer -->
    </div>

    <div class="exam-box" style="margin-top:4px;">
      <span class="exam-source">⭐ 优秀辩手评价量表（满分50分）</span>
      <div class="table-wrap" style="margin-top:12px;">
        <table>
          <thead>
            <tr><th>评价维度</th><th>分值</th><th>评分标准</th><th>自评</th><th>组评</th></tr>
          </thead>
          <tbody>
            <tr><td>语言表达</td><td>10分</td><td>普通话标准，口齿清晰，语速适中，有感染力</td><td>　/10</td><td>　/10</td></tr>
            <tr><td>逻辑反驳</td><td>10分</td><td>快速抓漏洞，反驳有理有据，逻辑严密</td><td>　/10</td><td>　/10</td></tr>
            <tr><td>论据充分</td><td>10分</td><td>事实、数据、经典案例，具有说服力</td><td>　/10</td><td>　/10</td></tr>
            <tr><td>团队配合</td><td>10分</td><td>尊重队友，观点一致，衔接流畅自然</td><td>　/10</td><td>　/10</td></tr>
            <tr><td>仪态风度</td><td>10分</td><td>举止得体，尊重对手，自信大方，情绪稳定</td><td>　/10</td><td>　/10</td></tr>
          </tbody>
        </table>
      </div>
    </div>

  </div>
</div>
```

***

## 六、辩论答案库（教师版直接使用，无需改动）

| 编号  | 考查能力 | 题目摘要                 | 参考答案                                                      |
| :-- | :--- | :------------------- | :-------------------------------------------------------- |
| A01 | 辩题判断 | 范进是否值得同情             | 值得同情；范进是封建科举制度的受害者。                                       |
| A02 | 辩题筛选 | 选出适合做辩题的选项           | A和F；A有争议空间；F脱离学生生活（具体以真题选项为准）。                            |
| A03 | 辩题识别 | 事实问题能否辩论             | B；"太阳从东方升起"是事实判断，无争议空间，不适合作为辩题。                           |
| A04 | 反驳方法 | 反驳"逆境不利成长"（孙悟空反例）    | 火眼金睛恰在炼丹炉烈火中炼成——那正是逆境！被压五行山五百年才磨去野性成佛。逆境激发潜能，磨砺意志，蜕变来自锤炼。 |
| A05 | 总结陈词 | 反驳"以成败论英雄"（正方）       | 成败受客观因素影响（如周瑜）；以成败论英雄诱导不择手段；英雄的本质是精神而非结果。                 |
| A06 | 准备流程 | 辩论准备阶段"不必要"的步骤       | C（调查大众观点非必须）和D（分析对手非独立准备环节）。                              |
| A07 | 论据判断 | 哪个论据不能证明范进值得同情       | B；作者对人物的选择不能直接证明范进值不值得同情。                                 |
| A08 | 文学联系 | 哪部名著与辩论主题相关          | 《水浒传》；接受招安、水泊梁山失败；学习逻辑反驳技巧。                               |
| A09 | 材料选择 | 选A段还是B段反驳"时代局限论"     | 选A段。① 明确反驳"时代局限"并用王冕作反例；② 升华"做破壁者"，有现实意义。                 |
| A10 | 立论构建 | 《朝花夕拾》底色：讽刺vs温馨（讽刺方） | 《二十四孝图》鞭笞虚伪孝道，《五猖会》控诉封建教育——温馨只是表象，批判才是灵魂。                 |
| A11 | 综合辩论 | 碎片化阅读弊大于利（弊方）        | 破坏思维连贯性，丧失追问能力；削弱对复杂文本的耐心，认知浅表化；应回归深度阅读。                  |
| AM1 | 综合迁移 | 网络让人更疏远（正方示例）        | 当表情包代替微笑，我们失去了人性温度。网络越发达孤独感越强，请走出电子城堡，拥抱真实世界！             |

***

## 七、index.html 导航页（固定模板）

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{知识点名称} · 学案导航</title>
  <style>
    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: linear-gradient(135deg, #1e3a5f 0%, #2563eb 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
      padding: 20px;
    }
    .card {
      background: white;
      border-radius: 28px;
      padding: 48px 40px 40px;
      text-align: center;
      max-width: 400px;
      width: 100%;
      box-shadow: 0 24px 64px rgba(0, 0, 0, 0.28);
    }
    .icon { font-size: 3.2rem; margin-bottom: 18px; }
    h1 { font-size: 1.8rem; font-weight: 800; color: #0f172a; margin-bottom: 8px; }
    .subtitle { color: #64748b; font-size: 0.92rem; margin-bottom: 36px; }
    .btn {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 10px;
      width: 100%;
      padding: 16px 20px;
      border-radius: 16px;
      font-size: 1rem;
      font-weight: 700;
      text-decoration: none;
      margin-bottom: 14px;
      transition: transform 0.15s, box-shadow 0.15s;
      letter-spacing: 0.02em;
    }
    .btn:hover { transform: translateY(-2px); box-shadow: 0 10px 30px rgba(0,0,0,0.20); }
    .btn-student { background: linear-gradient(135deg, #2563eb, #60a5fa); color: white; }
    .btn-teacher { background: linear-gradient(135deg, #16a34a, #4ade80); color: white; }
    .divider { height: 1px; background: #e2e8f0; margin: 20px 0 16px; }
    .meta { font-size: 0.78rem; color: #94a3b8; }
  </style>
</head>
<body>
  <div class="card">
    <div class="icon">🎙️</div>
    <h1>{知识点名称}学案</h1>
    <p class="subtitle">请选择查看版本</p>
    <a href="student.html" class="btn btn-student">📖 学生版</a>
    <a href="teacher.html" class="btn btn-teacher">📋 教师版（含参考答案）</a>
    <div class="divider"></div>
    <p class="meta">AI 教研助手生成 · {YYYY-MM-DD}</p>
  </div>
</body>
</html>
```

***

## 七点五、生成 Word 文档（student.docx / teacher.docx）

### 格式规范

* **正文**：小四字号（12pt）、黑色、宋体

* **标题/模块名**：小三字号（15pt）、黑色加粗、宋体

* **行距**：全文 1.5 倍行距

* **配色**：全黑白，无任何彩色填充或边框颜色

* **纸张**：A4，上下边距 2.54cm，左右边距 3.18cm

### 步骤

**第一步：确认 docx npm 包已安装**

```
npm list -g docx 2>/dev/null | grep docx || npm install -g docx
```

**第二步：生成 Node.js 脚本**

将以下脚本写入 `$OUT_DIR/gen_docx.js`，动态替换 `{知识点名称}` 和所有模块内容：

```
const {
  Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
  AlignmentType, BorderStyle, WidthType, LevelFormat, ShadingType
} = require('docx');
const fs = require('fs');

// ── 字号常量（单位：半点，1pt = 2半点）──
const BODY_SIZE  = 24; // 小四 12pt
const TITLE_SIZE = 30; // 小三 15pt
const LINE_SPACE = 360; // 1.5倍行距（240=单倍）

// ── 页面设置（A4，中文标准页边距）──
const PAGE = {
  size: { width: 11906, height: 16838 },
  margin: { top: 1440, bottom: 1440, left: 1814, right: 1814 }
};
const CONTENT_WIDTH = 11906 - 1814 - 1814; // = 8278 DXA

// ── 通用段落样式 ──
const bodySpacing = { line: LINE_SPACE, lineRule: 'auto', before: 0, after: 120 };
const titleSpacing = { line: LINE_SPACE, lineRule: 'auto', before: 240, after: 120 };

// ── 辅助函数 ──
function bodyPara(text, opts = {}) {
  return new Paragraph({
    spacing: bodySpacing,
    children: [new TextRun({
      text,
      font: '宋体',
      size: BODY_SIZE,
      color: '000000',
      ...opts
    })]
  });
}

function titlePara(text, level = 1) {
  return new Paragraph({
    spacing: titleSpacing,
    alignment: level === 0 ? AlignmentType.CENTER : AlignmentType.LEFT,
    children: [new TextRun({
      text,
      font: '宋体',
      size: TITLE_SIZE,
      bold: true,
      color: '000000'
    })]
  });
}

function sectionHeading(moduleNum, moduleName) {
  return new Paragraph({
    spacing: titleSpacing,
    children: [new TextRun({
      text: `【模块${moduleNum}】${moduleName}`,
      font: '宋体',
      size: TITLE_SIZE,
      bold: true,
      color: '000000'
    })]
  });
}

function divider() {
  return new Paragraph({
    spacing: { before: 60, after: 60 },
    border: { bottom: { style: BorderStyle.SINGLE, size: 4, color: '000000' } },
    children: []
  });
}

// ── 表格（黑白，无彩色）──
function makeTable(headers, rows, colWidths) {
  const border = { style: BorderStyle.SINGLE, size: 4, color: '000000' };
  const borders = { top: border, bottom: border, left: border, right: border };

  function makeCell(text, width, bold = false) {
    return new TableCell({
      borders,
      width: { size: width, type: WidthType.DXA },
      margins: { top: 80, bottom: 80, left: 120, right: 120 },
      children: [new Paragraph({
        spacing: { line: LINE_SPACE, lineRule: 'auto' },
        children: [new TextRun({ text, font: '宋体', size: BODY_SIZE, bold, color: '000000' })]
      })]
    });
  }

  return new Table({
    width: { size: CONTENT_WIDTH, type: WidthType.DXA },
    columnWidths: colWidths,
    rows: [
      new TableRow({
        tableHeader: true,
        children: headers.map((h, i) => makeCell(h, colWidths[i], true))
      }),
      ...rows.map(row => new TableRow({
        children: row.map((cell, i) => makeCell(cell, colWidths[i]))
      }))
    ]
  });
}

// ── 答题区（学生版用空行，教师版用答案文字）──
function answerBlock(isTeacher, answerText = '') {
  if (isTeacher) {
    return new Paragraph({
      spacing: bodySpacing,
      children: [new TextRun({
        text: `【参考答案】${answerText}`,
        font: '宋体',
        size: BODY_SIZE,
        color: '000000'
      })]
    });
  } else {
    // 学生版：空白答题区（用下划线段落模拟）
    return new Paragraph({
      spacing: { line: LINE_SPACE, lineRule: 'auto', before: 60, after: 60 },
      border: { bottom: { style: BorderStyle.SINGLE, size: 2, color: '888888' } },
      children: [new TextRun({ text: '答：', font: '宋体', size: BODY_SIZE, color: '000000' })]
    });
  }
}

// ════════════════════════════════════════
// 生成文档内容（动态替换为实际模块内容）
// 按以下结构生成 children 数组，复制并填充每个模块
// ════════════════════════════════════════
function buildChildren(isTeacher, kpName, modules) {
  const children = [];

  // 封面信息
  children.push(titlePara(`${kpName}学案（${isTeacher ? '教师版' : '学生版'}）`, 0));
  children.push(bodyPara(`班级：___________ 姓名：___________ 日期：___________`));
  children.push(divider());

  // 课标要求
  children.push(titlePara('课程标准要求（第四学段）'));
  children.push(bodyPara(modules.standard));
  children.push(bodyPara(''));

  // 知识点概览
  children.push(titlePara('本次学习总览'));
  children.push(bodyPara(modules.overview));
  children.push(bodyPara(''));

  // 各模块
  for (const mod of modules.list) {
    children.push(sectionHeading(mod.num, mod.name));
    children.push(bodyPara(`学习目标：${mod.goal}`));
    children.push(divider());

    // 知识要点
    children.push(new Paragraph({
      spacing: bodySpacing,
      children: [new TextRun({ text: '一、知识要点', font: '宋体', size: BODY_SIZE, bold: true, color: '000000' })]
    }));
    for (const point of mod.knowledge) {
      children.push(bodyPara(point));
    }
    children.push(bodyPara(''));

    // 真题训练
    children.push(new Paragraph({
      spacing: bodySpacing,
      children: [new TextRun({ text: '二、真题训练', font: '宋体', size: BODY_SIZE, bold: true, color: '000000' })]
    }));
    for (const q of mod.questions) {
      children.push(bodyPara(`【${q.source}】`));
      children.push(bodyPara(q.text));
      children.push(answerBlock(isTeacher, q.answer));
      children.push(bodyPara(''));
    }
  }

  // 综合实战流程表
  children.push(sectionHeading('五', '综合实战'));
  children.push(titlePara('辩论赛标准流程'));
  children.push(makeTable(
    ['环节', '核心任务', '参与辩手', '时长'],
    [
      ['立论陈词', '完整阐述己方核心论点及论据', '正方一辩 → 反方一辩', '各2分钟'],
      ['攻辩质询', '向对方提问，对方须直接作答', '双方二三辩交叉进行', '各3分钟'],
      ['自由辩论', '轮流发言，快速反驳，攻守结合', '全体辩手', '各4分钟'],
      ['总结陈词', '总结全场，升华价值，争取印象分', '反方四辩 → 正方四辩', '各3分钟'],
    ],
    [1800, 3000, 2200, 1278]
  ));
  children.push(bodyPara(''));
  children.push(titlePara('优秀辩手评价量表（满分50分）'));
  children.push(makeTable(
    ['评价维度', '分值', '评分标准', '自评', '组评'],
    [
      ['语言表达', '10分', '普通话标准，口齿清晰，语速适中，有感染力', '/10', '/10'],
      ['逻辑反驳', '10分', '快速抓漏洞，反驳有理有据，逻辑严密', '/10', '/10'],
      ['论据充分', '10分', '事实、数据、经典案例，具有说服力', '/10', '/10'],
      ['团队配合', '10分', '尊重队友，观点一致，衔接流畅自然', '/10', '/10'],
      ['仪态风度', '10分', '举止得体，尊重对手，自信大方，情绪稳定', '/10', '/10'],
    ],
    [1400, 800, 3200, 800, 800] // 注意：列宽之和 = 7000，需根据实际调整至 CONTENT_WIDTH
  ));

  return children;
}

// ════════════════════════════════════════
// 主程序：填充 modules 对象后运行
// ════════════════════════════════════════
// 在生成脚本时，将以下 modules 替换为从材料中提炼的实际内容
const KP_NAME = '{知识点名称}';
const modules = {
  standard: '{课标要求文字}',
  overview: '{知识点概览文字}',
  list: [
    {
      num: '一', name: '辩题判断',
      goal: '能识别辩题类型，判断争议性、价值性、可辩性',
      knowledge: [
        '好的辩题需满足三个条件：',
        '① 争议性：命题双方均有合理论证空间，无明确标准答案',
        '② 价值性：值得讨论，能引发深度思考',
        '③ 可辩性：贴近学生生活或认知，能找到具体论据支撑',
      ],
      questions: [
        {
          source: '辩论真题合集·第1题',
          text: '下列选项中，适合作为辩论赛辩题的是（  ）\nA. 网络让人与人之间的距离更远了\nB. 1加1等于几\nC. 中国的首都是北京\nD. 学校应该禁止使用手机',
          answer: 'A和D。A具有争议性；D是行动方案型，双方均有充分论据。B和C是有客观答案的事实问题，无辩论空间。'
        }
      ]
    }
    // 其余模块照此格式补充
  ]
};

async function main() {
  for (const isTeacher of [false, true]) {
    const doc = new Document({
      sections: [{
        properties: { page: PAGE },
        children: buildChildren(isTeacher, KP_NAME, modules)
      }]
    });
    const buffer = await Packer.toBuffer(doc);
    const filename = isTeacher ? 'teacher.docx' : 'student.docx';
    fs.writeFileSync(filename, buffer);
    console.log(`✅ 已生成：${filename}`);
  }
}

main().catch(console.error);
```

**第三步：运行脚本**

```
cd "$OUT_DIR"
node gen_docx.js
```

**第四步：验证并清理**

```
ls -lh "$OUT_DIR/"*.docx
rm "$OUT_DIR/gen_docx.js"   # 删除临时脚本
```

> **注意**：生成脚本时，将 `modules` 对象中的所有占位符替换为从材料中提炼的实际内容，每个模块的 `list` 数组对应一个学案模块，`questions` 数组对应该模块下的真题，`answer` 字段教师版使用、学生版忽略。

***

## 八、保存文件（严格按顺序执行）

```
# 第一步：创建输出目录
mkdir -p "$OUT_DIR"

# 第二步：确认目录已创建
ls -la "$ROOT/references/知识点/$KP/"
```

然后使用 **Write 工具**（不是 Bash echo）依次写入三个文件：

1. `$OUT_DIR/index.html` — 导航页（§七的模板，替换知识点名称和日期）

2. `$OUT_DIR/student.html` — 学生版（所有模块，**不含任何** `.teacher-answer` 块）

3. `$OUT_DIR/teacher.html` — 教师版（所有模块，每道题下方含 `.teacher-answer` 块）

写入时注意：文件路径用绝对路径，文件名不含中文，确保文件内容完整（从 `<!DOCTYPE html>` 到 `</html>` 完整输出）。

***

## 九、验证 + 自动打开预览

```
# 验证三个文件均已生成且大小大于0
ls -lh "$OUT_DIR/"

# macOS：在默认浏览器中打开导航页
open "$OUT_DIR/index.html"
```

验证通过后，向用户输出以下报告：

```
✅ {知识点名称}学案已生成，浏览器已自动打开预览！

📁 文件位置：references/知识点/{知识点名称}/{知识点名称}学案/
   ├── index.html    ← 导航入口（已自动打开）
   ├── student.html  ← 学生版 HTML（{N}个模块，{M}道真题）
   ├── teacher.html  ← 教师版 HTML（含参考答案）
   ├── student.docx  ← 学生版 Word（可直接打印）
   └── teacher.docx  ← 教师版 Word（含参考答案）

💡 建议：先打开教师版核对答案，再将学生版 Word 打印分发。
```

***

## 十、非辩论知识点处理规则

| 项目     | 规则                                       |
| :----- | :--------------------------------------- |
| 真题来源   | 优先从 `模卷库/` 和知识点目录实际文件提取；不足3题时补充【示例题】并标注  |
| 答案生成   | 基于课标+教材要点式作答，格式：① 要点一 ② 要点二，附"评分说明：每点X分" |
| 模块五    | 固定为综合实战：综合写作/口语任务 + 针对本知识点定制评价量表         |
| 评价量表维度 | 根据知识点调整，如说明文阅读：信息筛选/方法辨析/顺序把握/语言品味/综合表达  |

***

## 十一、错误处理

| 情况                    | 处理方式                                             |
| :-------------------- | :----------------------------------------------- |
| `textutil` 无法读取 .docx | 自动切换 Python zipfile 方案；仍失败则提示并用内置知识框架，标注"建议补充材料" |
| 知识点目录不存在              | 提示用户确认路径，询问"是否用通用材料继续生成？"                        |
| 目录创建失败（权限/路径问题）       | 报告具体错误，建议手动创建 `references/知识点/{知识点名称}/学案/`       |
| 真题不足3题                | 自动补充示例题，题目前注明"【示例题·建议替换为实际真题】"                   |
| `open` 命令失败（非 macOS）  | 跳过自动打开步骤，直接告知完整文件路径                              |

***

## 立即开始

收到消息后：

* 若已提供知识点名称 → 直接从**步骤一（TodoWrite 创建任务清单）**开始执行，无需再次确认

* 若未提供 → 询问知识点名称后立即执行

* **必须自动创建目录并写入文件**，不能只在对话中显示 HTML 代码

