---
name: python-cli
description: 使用 typer + rich 构建现代 Python CLI 应用。当需要创建命令行脚本、添加 CLI 参数/选项、美化终端输出、显示进度条、集成日志、显示表格/面板时使用此 skill。
---

# Python CLI (typer + rich)

## 依赖

```bash
pip install typer rich
```

## 快速开始

```python
import typer
from rich.console import Console
from typing import Annotated

app = typer.Typer()
console = Console()

@app.command()
def hello(
    name: Annotated[str, typer.Argument(help="用户名")],
    count: Annotated[int, typer.Option("--count", "-c")] = 1,
):
    """打招呼命令"""
    for _ in range(count):
        console.print(f"[bold green]Hello[/] {name}!")

if __name__ == "__main__":
    app()
```

## 常用模式速查

### Typer

| 需求 | 代码 |
|------|------|
| 位置参数 | `name: Annotated[str, typer.Argument()]` |
| 命名选项 | `--name: Annotated[str, typer.Option("--name", "-n")]` |
| 布尔标志 | `--verbose: Annotated[bool, typer.Option("--verbose", "-v")] = False` |
| 文件路径 | `file: Annotated[Path, typer.Argument(exists=True)]` |
| 密码输入 | `typer.Option(prompt=True, hide_input=True)` |
| 确认提示 | `typer.confirm("确定?")` |
| 版本号 | 用 `@app.callback()` + `--version` callback |
| 子命令 | `app.add_typer(sub_app, name="sub")` |
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
| 语法高亮 | `Syntax(code, "python")` |

详见 [references/rich.md](references/rich.md)

## 推荐项目结构

```
mycli/
├── __init__.py
├── __main__.py      # typer app 入口
├── cli.py           # 命令定义
├── core.py          # 业务逻辑
└── utils/
    └── console.py   # 共享 Console 实例
```

**`__main__.py`**:
```python
from .cli import app

if __name__ == "__main__":
    app()
```

**`utils/console.py`**:
```python
from rich.console import Console

console = Console()
err_console = Console(stderr=True)
```

## 常用组合示例

### 带进度的批处理

```python
from rich.progress import track

@app.command()
def process(files: list[Path]):
    for file in track(files, description="处理中..."):
        # process file
        pass
```

### 结果表格输出

```python
from rich.table import Table

@app.command()
def list_users():
    table = Table(title="用户列表")
    table.add_column("ID", style="cyan")
    table.add_column("名称")
    table.add_column("状态")

    for user in get_users():
        table.add_row(str(user.id), user.name, user.status)

    console.print(table)
```

### 错误处理

```python
@app.command()
def run(config: Path):
    if not config.exists():
        console.print(f"[red]Error:[/] {config} not found", stderr=True)
        raise typer.Exit(1)

    try:
        result = process(config)
        console.print(Panel(result, title="[green]Success"))
    except Exception as e:
        console.print_exception()
        raise typer.Exit(1)
```
