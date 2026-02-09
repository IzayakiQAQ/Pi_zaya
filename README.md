# kb_chat

一个轻量的「可检索知识库（BM25）+ DeepSeek 对话 UI（Streamlit）」原型，用来在你自己的论文/笔记（Markdown）里检索并对话。

## 给合作者试用（每个人用自己的 PDF/目录，本地跑）

你提到的需求：
- 合作者自己管理/存放 PDF（用他自己的目录，不用你的硬盘路径）
- 你更新代码后，合作者能尽快用到最新版

结论：推荐做法是 **合作者在自己电脑上跑一份 kb_chat**。这样他读写的就是他自己的硬盘；你更新后他 `git pull` 即可拿到最新版。

### 快速启动（Windows）

推荐用一键脚本（会自动创建 `.venv`、安装依赖、启动 Streamlit）：
- 双击 `run.cmd`
- 或在 PowerShell 运行 `./run.ps1`

默认只允许本机访问（更安全）：`127.0.0.1:8501`  
如果你想同一局域网其它设备也能访问，把环境变量设成：
- `$env:KB_STREAMLIT_ADDR="0.0.0.0"`

端口也可改：
- `$env:KB_STREAMLIT_PORT="8501"`

### 合作者第一次使用（复制给他）

```powershell
git clone https://github.com/LittlePyx/Pi_zaya.git
cd Pi_zaya
$env:DEEPSEEK_API_KEY="他的key"
.\run.cmd
```

### 目录设置（每个人独立）

在页面左侧「设置」里选择：
- `PDF 路径`：你的 PDF 根目录
- `DB 路径 / MD 路径`：你的知识库目录

这些偏好会写在本机的 `user_prefs.json`（已在 `.gitignore` 中忽略，不会影响拉取更新）。

### 如何拿到最新版

你更新代码并推送后，合作者只要再次运行 `run.cmd` / `run.ps1` 即可。

`run.ps1` 会在启动前自动 `git pull`（前提：当前目录是 git repo 且系统有 git）。

如果你不想用 git，也可以用 OneDrive/网盘/共享盘同步整个文件夹，但要注意同步冲突。

## 手动安装/运行（可选）

如果你不想用 `run.cmd`/`run.ps1`，也可以手动装依赖并启动。

### 1) 安装依赖

如果你没有管理员权限，建议用 `--user` 安装到当前用户目录：

```powershell
cd F:\research-papers\2026\Jan\else\kb_chat
python -m pip install --user -r requirements.txt
```

### 2) 配置 DeepSeek

建议只用环境变量，不要把 key 写进代码文件里。

PowerShell：

```powershell
$env:DEEPSEEK_API_KEY="你的key"
$env:DEEPSEEK_BASE_URL="https://api.deepseek.com/v1"
$env:DEEPSEEK_MODEL="deepseek-chat"
```

CMD：

```bat
set DEEPSEEK_API_KEY=你的key
set DEEPSEEK_BASE_URL=https://api.deepseek.com/v1
set DEEPSEEK_MODEL=deepseek-chat
```

> 可选：如果你把转换脚本（默认会自动找上级目录的 `test2.py`）挪到了别处，可以设置：
>
> ```powershell
> $env:KB_PDF_CONVERTER="D:\\path\\to\\test2.py"
> ```

### 3) 建库（把 Markdown 喂进去）

建议把“最终版 md”喂进去，默认会跳过 `temp/` 这类页级临时文件，避免重复。

```powershell
python ingest.py --src F:\research-papers\2026\Jan\else\tosave_md_strict --db .\db --incremental --prune
```

### 4) 启动 UI

```powershell
python -m streamlit run app.py
```

## 常见问题

### 1) 合作者想用自己的目录，但访问不了你的 `192.168.x.x:8501`

这是正常的：`192.168.*` 是内网地址，只能同一局域网访问。A 方案（本地跑）不依赖你的内网地址，合作者跑起来后直接访问他自己本机的 `127.0.0.1:8501` 即可。

### 2) 运行时提示 401 / key 无效

检查是否设置了正确的环境变量（PowerShell）：
- `$env:DEEPSEEK_API_KEY="你的key"`
