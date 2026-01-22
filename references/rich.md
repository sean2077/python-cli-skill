# Rich 详细参考

## 目录
- [Console 基础](#console-基础)
- [样式与颜色](#样式与颜色)
- [表格](#表格)
- [面板](#面板)
- [进度条](#进度条)
- [日志集成](#日志集成)
- [其他组件](#其他组件)

---

## Console 基础

### 创建 Console

```python
from rich.console import Console

console = Console()

# 基础打印
console.print("Hello, World!")

# 带样式打印
console.print("Hello", style="bold red")

# 打印到 stderr
console.print("Error!", style="red", stderr=True)
```

### print vs log

```python
# print: 普通输出
console.print("Processing data...")

# log: 带时间戳和调用位置，适合调试
console.log("Starting process")
console.log("Data loaded", log_locals=True)  # 显示局部变量
```

### 输出控制

```python
# 禁用颜色（如重定向到文件）
console = Console(force_terminal=False)

# 指定宽度
console = Console(width=120)

# 记录输出
console = Console(record=True)
console.print("Hello")
html = console.export_html()  # 导出为 HTML
```

---

## 样式与颜色

### 内联样式（markup）

```python
console.print("[bold]粗体[/bold] [italic]斜体[/italic]")
console.print("[red]红色[/red] [green]绿色[/green] [blue]蓝色[/blue]")
console.print("[bold red on white]粗体红字白底[/bold red on white]")
console.print("[link=https://example.com]点击链接[/link]")
```

### 常用样式

| 样式 | 说明 |
|------|------|
| `bold` | 粗体 |
| `italic` | 斜体 |
| `underline` | 下划线 |
| `strike` | 删除线 |
| `dim` | 暗淡 |
| `reverse` | 反色 |

### 常用颜色

| 颜色 | 说明 |
|------|------|
| `red`, `green`, `blue` | 基础色 |
| `cyan`, `magenta`, `yellow` | 辅助色 |
| `white`, `black` | 黑白 |
| `bright_red`, `bright_green` | 亮色 |
| `#ff5733` | 十六进制 |
| `rgb(255,87,51)` | RGB |

### Style 对象

```python
from rich.style import Style

error_style = Style(color="red", bold=True)
success_style = Style(color="green")

console.print("Error!", style=error_style)
console.print("Success!", style=success_style)
```

---

## 表格

### 基础表格

```python
from rich.table import Table

table = Table(title="用户列表")
table.add_column("ID", justify="right", style="cyan")
table.add_column("名称", style="magenta")
table.add_column("状态", justify="center")

table.add_row("1", "Alice", "[green]Active[/green]")
table.add_row("2", "Bob", "[red]Inactive[/red]")
table.add_row("3", "Charlie", "[yellow]Pending[/yellow]")

console.print(table)
```

### 表格样式

```python
table = Table(
    title="报表",
    title_style="bold magenta",
    header_style="bold cyan",
    border_style="blue",
    show_lines=True,  # 显示行分隔线
    expand=True,      # 扩展到终端宽度
)
```

### 列配置

```python
table.add_column(
    "数值",
    justify="right",      # left, center, right
    style="green",
    no_wrap=True,         # 禁止换行
    width=20,             # 固定宽度
    min_width=10,         # 最小宽度
    max_width=30,         # 最大宽度
    ratio=1,              # 比例宽度
)
```

---

## 面板

### 基础面板

```python
from rich.panel import Panel

console.print(Panel("Hello, World!"))
console.print(Panel("带标题", title="Info"))
console.print(Panel("带副标题", title="Main", subtitle="v1.0"))
```

### 面板样式

```python
panel = Panel(
    "内容文本",
    title="标题",
    title_align="left",    # left, center, right
    border_style="green",
    padding=(1, 2),        # (上下, 左右)
    expand=False,          # 不扩展到终端宽度
)
console.print(panel)
```

### 嵌套内容

```python
from rich.panel import Panel
from rich.table import Table

table = Table()
table.add_column("Name")
table.add_row("Alice")

console.print(Panel(table, title="用户表"))
```

---

## 进度条

### 简单进度（track）

```python
from rich.progress import track
import time

for item in track(range(100), description="处理中..."):
    time.sleep(0.01)
```

### 自定义进度条

```python
from rich.progress import (
    Progress,
    SpinnerColumn,
    TextColumn,
    BarColumn,
    TaskProgressColumn,
    TimeRemainingColumn,
    TimeElapsedColumn,
)

with Progress(
    SpinnerColumn(),
    TextColumn("[bold blue]{task.description}"),
    BarColumn(),
    TaskProgressColumn(),
    TimeRemainingColumn(),
) as progress:
    task1 = progress.add_task("[cyan]下载...", total=1000)
    task2 = progress.add_task("[green]处理...", total=1000)

    while not progress.finished:
        progress.update(task1, advance=5)
        progress.update(task2, advance=3)
        time.sleep(0.01)
```

### 不确定进度

```python
with Progress() as progress:
    task = progress.add_task("[cyan]加载中...", total=None)
    # total=None 显示不确定进度（旋转动画）
    time.sleep(2)
```

### 手动控制

```python
progress = Progress()
progress.start()

task = progress.add_task("处理", total=100)
for i in range(100):
    progress.update(task, advance=1, description=f"处理 {i}")
    time.sleep(0.01)

progress.stop()
```

---

## 日志集成

### 标准库 logging 集成

```python
import logging
from rich.logging import RichHandler

logging.basicConfig(
    level=logging.DEBUG,
    format="%(message)s",
    datefmt="[%X]",
    handlers=[RichHandler(rich_tracebacks=True)]
)

log = logging.getLogger("myapp")

log.debug("调试信息")
log.info("普通信息")
log.warning("警告信息")
log.error("错误信息")
log.exception("异常信息")  # 会显示完整 traceback
```

### RichHandler 配置

```python
handler = RichHandler(
    show_time=True,
    show_level=True,
    show_path=True,
    rich_tracebacks=True,
    tracebacks_show_locals=True,
    markup=True,  # 允许在日志中使用 rich markup
)
```

### 与 Console 配合

```python
from rich.console import Console
from rich.logging import RichHandler

console = Console(stderr=True)
handler = RichHandler(console=console)

logging.basicConfig(handlers=[handler])
```

---

## 其他组件

### 语法高亮

```python
from rich.syntax import Syntax

code = '''
def hello(name: str) -> str:
    return f"Hello {name}"
'''

syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console.print(syntax)

# 从文件读取
syntax = Syntax.from_path("main.py", theme="monokai")
console.print(syntax)
```

### Markdown 渲染

```python
from rich.markdown import Markdown

md = """
# 标题

这是一段**粗体**和*斜体*文字。

- 列表项 1
- 列表项 2
"""

console.print(Markdown(md))
```

### 树形结构

```python
from rich.tree import Tree

tree = Tree("[bold]项目结构")
src = tree.add("[blue]src")
src.add("main.py")
src.add("utils.py")
tests = tree.add("[green]tests")
tests.add("test_main.py")

console.print(tree)
```

### 状态指示器

```python
from rich.status import Status

with console.status("[bold green]处理中...") as status:
    time.sleep(2)
    status.update("[bold blue]第二阶段...")
    time.sleep(2)
```

### 分组输出

```python
from rich.console import Group
from rich.panel import Panel

group = Group(
    Panel("第一块"),
    Panel("第二块"),
)
console.print(Panel(group, title="组合"))
```
