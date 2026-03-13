# 论文下载与总结小工具

根据 bib 文件从 Semantic Scholar 获取论文信息、下载开放获取 PDF，并用配置的 LLM API 对每篇论文做四步总结，输出到同一份 Markdown 中。

## 功能

- **Bib 解析**：默认使用当前目录下的 `citation.bib`，解析其中的条目并提取 Semantic Scholar CorpusID（从 `url` 中的 `CorpusID:xxxxx`）。
- **论文获取与下载**：通过 Semantic Scholar API 获取元数据与摘要；若有开放获取 PDF 则下载到当次运行的输出目录。
- **统一输出目录**：每次运行会新建一个带时间戳的文件夹（如 `papers_20250313_143022`），避免覆盖。
- **四步总结**：对每篇论文按以下四个维度调用 API 总结：
  - 一句话总结：这篇论文到底做了一件什么事？
  - 研究背景：在这个领域，大家之前遇到了什么痛点或难题？
  - 核心创新：作者提出了什么新方法来解决这个难题？
  - 主要结论：实验/研究证明了什么？效果如何？
- **统一 Markdown**：同一 bib 下所有论文的总结写入**同一份** Markdown 文件，文件名带时间戳（如 `summaries_20250313_143022.md`），避免重名覆盖。

## 环境与依赖

推荐使用 **uv** 创建虚拟环境并安装依赖（项目根目录下执行）：

```bash
# 创建虚拟环境（默认 .venv）
uv venv

# 根据 pyproject.toml 安装依赖
uv sync
```

激活虚拟环境后运行脚本：

- **Windows:** `.venv\Scripts\activate`
- **macOS/Linux:** `source .venv/bin/activate`

也可使用 pip（需先激活虚拟环境）：

```bash
pip install -r requirements_paper_tool.txt
```

需已配置 `api_config.env`（与现有项目一致），用于总结接口：

- `OPENAI_COMPATIBLE_API_KEY`
- `OPENAI_COMPATIBLE_BASE_URL`
- `OPENAI_COMPATIBLE_MODEL`

## 使用方法

在项目根目录（即 `citation.bib` 所在目录）下执行：

```bash
# 使用默认 citation.bib，输出到当前目录下新建的 papers_YYYYMMDD_HHMMSS
python run_paper_download_and_summary.py

# 指定 bib 文件
python run_paper_download_and_summary.py --bib path/to/other.bib

# 指定输出根目录（论文 PDF 与总结 MD 都会放在该目录下）
python run_paper_download_and_summary.py --out-dir ./my_output

# 可选：Semantic Scholar API Key（提高限流）
python run_paper_download_and_summary.py --semantic-api-key YOUR_KEY
```

## 输出说明

- **文件夹**：每次运行默认创建 `papers_YYYYMMDD_HHMMSS`，其下存放：
  - 下载的 PDF（若有）
  - `summaries_YYYYMMDD_HHMMSS.md`：本批论文的四步总结汇总。
- **Markdown 结构**：每篇论文一个二级标题 `## [citation_key] 标题`，下面为四步总结内容。

## 说明

- 仅当 bib 条目的 `url` 中含有 `CorpusID:数字` 时才会请求 Semantic Scholar 并尝试下载与总结；无 CorpusID 的条目会在 MD 中标记为“未提供 CorpusID”。
- 若某篇论文无开放获取 PDF，仍会用**摘要**进行四步总结。
- 总结 API 使用 `api_config_loader` 读取的 OpenAI 兼容接口（如阿里云 DashScope）。
