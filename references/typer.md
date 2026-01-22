# Typer 详细参考

## 目录
- [基础结构](#基础结构)
- [命令与子命令](#命令与子命令)
- [参数类型](#参数类型)
- [选项配置](#选项配置)
- [回调与钩子](#回调与钩子)
- [错误处理](#错误处理)

---

## 基础结构

### 单命令脚本

```python
import typer

def main(name: str):
    print(f"Hello {name}")

if __name__ == "__main__":
    typer.run(main)
```

### 多命令应用

```python
import typer

app = typer.Typer()

@app.command()
def hello(name: str):
    print(f"Hello {name}")

@app.command()
def goodbye(name: str):
    print(f"Goodbye {name}")

if __name__ == "__main__":
    app()
```

---

## 命令与子命令

### 子应用（命令组）

```python
import typer

app = typer.Typer()
users_app = typer.Typer()
app.add_typer(users_app, name="users")

@users_app.command()
def create(name: str):
    print(f"Creating user: {name}")

@users_app.command()
def delete(name: str):
    print(f"Deleting user: {name}")

# 使用: python main.py users create alice
```

### 命令帮助文本

```python
@app.command(help="创建新用户")
def create(name: str):
    """
    创建新用户。

    NAME: 用户名称
    """
    print(f"Creating {name}")
```

---

## 参数类型

### Argument（位置参数）

```python
from typing import Annotated
import typer

@app.command()
def greet(
    name: Annotated[str, typer.Argument(help="用户名")],
    times: Annotated[int, typer.Argument()] = 1,
):
    for _ in range(times):
        print(f"Hello {name}")

# 使用: python main.py alice 3
```

### Option（命名选项）

```python
from typing import Annotated
import typer

@app.command()
def greet(
    name: Annotated[str, typer.Option("--name", "-n", help="用户名")],
    formal: Annotated[bool, typer.Option("--formal", "-f")] = False,
):
    greeting = "Good day" if formal else "Hello"
    print(f"{greeting} {name}")

# 使用: python main.py --name alice -f
```

### 常用类型

| 类型 | 示例 | 说明 |
|------|------|------|
| `str` | `name: str` | 字符串 |
| `int` | `count: int` | 整数 |
| `float` | `rate: float` | 浮点数 |
| `bool` | `--verbose` | 布尔标志 |
| `Path` | `file: Path` | 文件路径 |
| `list[str]` | `--item` | 可重复选项 |
| `Enum` | `--color` | 枚举选择 |

### 枚举选项

```python
from enum import Enum
from typing import Annotated
import typer

class Color(str, Enum):
    red = "red"
    green = "green"
    blue = "blue"

@app.command()
def paint(
    color: Annotated[Color, typer.Option(help="选择颜色")]
):
    print(f"Painting in {color.value}")

# 使用: python main.py --color red
```

### 文件路径

```python
from pathlib import Path
from typing import Annotated
import typer

@app.command()
def process(
    input_file: Annotated[Path, typer.Argument(
        exists=True,
        file_okay=True,
        dir_okay=False,
        readable=True,
        help="输入文件"
    )],
    output_dir: Annotated[Path, typer.Option(
        exists=True,
        file_okay=False,
        dir_okay=True,
        writable=True,
    )] = Path("."),
):
    print(f"Processing {input_file} -> {output_dir}")
```

---

## 选项配置

### 提示输入

```python
@app.command()
def login(
    username: Annotated[str, typer.Option(prompt=True)],
    password: Annotated[str, typer.Option(prompt=True, hide_input=True)],
):
    print(f"Logging in as {username}")
```

### 确认提示

```python
@app.command()
def delete(
    force: Annotated[bool, typer.Option(
        prompt="确定要删除吗？",
        help="强制删除"
    )] = False,
):
    if force:
        print("Deleted!")
```

### 环境变量

```python
@app.command()
def connect(
    host: Annotated[str, typer.Option(envvar="DB_HOST")] = "localhost",
    port: Annotated[int, typer.Option(envvar="DB_PORT")] = 5432,
):
    print(f"Connecting to {host}:{port}")
```

---

## 回调与钩子

### 全局选项（callback）

```python
import typer

app = typer.Typer()
state = {"verbose": False}

@app.callback()
def main(
    verbose: Annotated[bool, typer.Option("--verbose", "-v")] = False,
):
    """应用描述文字"""
    state["verbose"] = verbose

@app.command()
def run():
    if state["verbose"]:
        print("Verbose mode")
    print("Running...")
```

### 版本选项

```python
from typing import Optional
from typing import Annotated
import typer

__version__ = "1.0.0"

def version_callback(value: bool):
    if value:
        print(f"Version: {__version__}")
        raise typer.Exit()

@app.callback()
def main(
    version: Annotated[Optional[bool], typer.Option(
        "--version", "-V",
        callback=version_callback,
        is_eager=True,
    )] = None,
):
    pass
```

---

## 错误处理

### 退出码

```python
import typer

@app.command()
def process(file: Path):
    if not file.exists():
        print(f"Error: {file} not found", err=True)
        raise typer.Exit(code=1)
    print(f"Processing {file}")
```

### 中止执行

```python
@app.command()
def dangerous():
    confirm = typer.confirm("这是危险操作，确定继续？")
    if not confirm:
        raise typer.Abort()
    print("执行中...")
```

### 异常处理

```python
@app.command()
def fetch(url: str):
    try:
        # ... fetch logic
        pass
    except Exception as e:
        print(f"Error: {e}", err=True)
        raise typer.Exit(code=1)
```
