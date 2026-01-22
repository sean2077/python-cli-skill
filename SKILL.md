---
name: python-cli
description: 使用 typer + rich 构建现代 Python CLI 应用。当需要创建命令行脚本、添加 CLI 参数/选项、美化终端输出、显示进度条、集成日志、显示表格/面板时使用此 skill。
---

# Python CLI (typer + rich)

## 依赖

```bash
pip install typer rich
```

## 单脚本模板

适用于：工具脚本、一次性任务、简单自动化

```python
#!/usr/bin/env python3
"""简短描述脚本功能"""
from pathlib import Path
from typing import Annotated

import typer
from rich.console import Console

console = Console()
err_console = Console(stderr=True)

def main(
    input_file: Annotated[Path, typer.Argument(help="输入文件", exists=True)],
    output: Annotated[Path, typer.Option("-o", "--output", help="输出文件")] = None,
    verbose: Annotated[bool, typer.Option("-v", "--verbose", help="详细输出")] = False,
):
    """脚本主功能描述"""
    if verbose:
        console.log(f"处理: {input_file}")

    # 业务逻辑
    try:
        result = process(input_file)
        console.print(f"[green]完成[/]")
    except Exception as e:
        err_console.print(f"[red]错误:[/] {e}")
        raise typer.Exit(1)

if __name__ == "__main__":
    typer.run(main)
```

## 多命令脚本模板

适用于：多个相关操作、CRUD 类工具

```python
#!/usr/bin/env python3
"""工具描述"""
from pathlib import Path
from typing import Annotated, Optional

import typer
from rich.console import Console
from rich.table import Table

app = typer.Typer(help="工具描述")
console = Console()

@app.command()
def list():
    """列出所有项目"""
    table = Table(title="项目列表")
    table.add_column("ID", style="cyan")
    table.add_column("名称")
    # ...
    console.print(table)

@app.command()
def add(name: Annotated[str, typer.Argument(help="名称")]):
    """添加新项目"""
    console.print(f"[green]已添加:[/] {name}")

@app.command()
def remove(
    name: Annotated[str, typer.Argument(help="名称")],
    force: Annotated[bool, typer.Option("--force", "-f", help="强制删除")] = False,
):
    """删除项目"""
    if not force and not typer.confirm(f"确定删除 {name}?"):
        raise typer.Abort()
    console.print(f"[yellow]已删除:[/] {name}")

if __name__ == "__main__":
    app()
```

## 复杂项目结构

适用于：大型 CLI 工具、需要分模块、可安装包

```
mycli/
├── pyproject.toml
├── src/mycli/
│   ├── __init__.py
│   ├── __main__.py      # python -m mycli 入口
│   ├── cli.py           # 命令定义
│   ├── core.py          # 业务逻辑
│   └── utils/
│       └── console.py   # 共享 Console
```

**`pyproject.toml`** (关键部分):
```toml
[project.scripts]
mycli = "mycli.cli:app"
```

**`__main__.py`**:
```python
from .cli import app
app()
```

**`utils/console.py`**:
```python
from rich.console import Console
console = Console()
err_console = Console(stderr=True)
```

## 常用模式速查

### Typer

| 需求 | 代码 |
|------|------|
| 位置参数 | `name: Annotated[str, typer.Argument()]` |
| 命名选项 | `name: Annotated[str, typer.Option("--name", "-n")]` |
| 布尔标志 | `verbose: Annotated[bool, typer.Option("-v", "--verbose")] = False` |
| 文件路径 | `file: Annotated[Path, typer.Argument(exists=True)]` |
| 密码输入 | `typer.Option(prompt=True, hide_input=True)` |
| 确认提示 | `typer.confirm("确定?")` |
| 子命令组 | `app.add_typer(sub_app, name="sub")` |
| 退出码 | `raise typer.Exit(code=1)` |

详见 [references/typer.md](references/typer.md)

### Rich

| 需求 | 代码 |
|------|------|
| 彩色输出 | `console.print("[bold red]Error[/]")` |
| 调试日志 | `console.log("msg", log_locals=True)` |
| 表格 | `Table()` + `add_column()` + `add_row()` |
| 面板 | `Panel("content", title="Title")` |
| 简单进度 | `for x in track(items): ...` |
| 复杂进度 | `Progress()` + columns |
| 日志集成 | `RichHandler(rich_tracebacks=True)` |

详见 [references/rich.md](references/rich.md)

## 常用组合

### 带进度的批处理

```python
from rich.progress import track

for file in track(files, description="处理中..."):
    process(file)
```

### 错误处理

```python
try:
    result = process(data)
    console.print(Panel(result, title="[green]Success"))
except Exception as e:
    err_console.print(f"[red]Error:[/] {e}")
    raise typer.Exit(1)
```
