# Code Quality Management of Project Workflow

!!! success "Define"
    Code Quality: code style, code logic, code recommended (replace or upgrade).

:smile:
PEP8 是官方出品的 code style，不少公司也有自己的一套 style，除了平时 coding 时注意遵循外，好的格式化工具和检查工具也是保证代码质量的必要环节。还好现在 Python 生态很好，有不少工具都可以实现。

## Code Quality Tools

参数配置（环境变量）一般都是首先要解决的，随着发展 Python 在这些方面也越来越规范和统一，这也给开发带来了不少便捷，其中最重要的是 [PEP 621](https://peps.python.org/pep-0621/)，为统一配置项打下基础。除此之外还有利用其它已有配置选项为配置进一步缩减了复杂度（一处配置即可）—— `.gitignore`，一会儿就能看到很多工具都复用了此文件大大减少了配置流程。

此项目用到的工具包括：

- Project and virtualenv management with dependencies (pyproject.toml): [PDM](https://pdm.fming.dev/latest/)
- Formatter (code style): [black](https://github.com/psf/black)
- Linter (code style and static code logic and code recommend): [ruff](https://github.com/charliermarsh/ruff)
- Static type checker (static code logic): [mypy](https://mypy.readthedocs.io/en/stable/index.html)
- Test (dynamic code logic): [pytest](https://github.com/pytest-dev/pytest/)

!!! warning "注意"

    Python >= 3.11.1

```bash title="安装依赖"
pdm add -d black ruff
```
### Formatter - black

格式化工具较常用的有两个：black 是一个 pep8 风格的格式化工具，isort 是一个 可以排序 import 的工具。其中 isort 已经被 linter 工具 ruff (--fix) 所替代，而 black 的格式化功能还在完善中，期待之后的大统一，All in one~

!!! 其它的工具还有

    - [autopep8](https://github.com/hhatto/autopep8)
    - [yapf](https://github.com/google/yapf)

常用配置选项如下：

```toml title="pyproject.toml"
[tool.black]
line-length = 120

# 23.1.0 之后将自动推断 python 版本
# target-version = ["py311"]

# extend-exclude = '| migragte'
```

black 会自动加载根目录下的 pyproject.toml 中相关配置。当不设置 exclude 选项时，black 自动排除 `gitignore` 中配置项。

!!! Tip

    ``` toml
    exclude = '''
    /(
        | \.direnv
        | \.eggs
        | \.git
        | \.hg
        | \.mypy_cache
        | \.nox
        | \.tox
        | \.venv
        | venv
        | \.svn
        | \.ipynb_checkpoints
        | _build
        | buck-out
        | build
        | dist
        | __pypackages__
    )/
    '''
    # | 开头表示任意位置， ^/ 开头表示根目录下。
    ```

配置好后就可以使用 `pdm run black .` 检查。`--check` 可以

```bash
# check and format
pdm run black .
# or only check
pdm run black --check .
```

### Linter - ruff

利用静态源码分析(static source code analysis)检查代码风格、逻辑和可能的优化方案，来达到 clean code。[flake8](https://github.com/PyCQA/flake8) 是其中一个较为出名的 linter（实质是PyFlakes、pycodestyle、McCabe 的wrapper ），并有许多围绕其的 plugins。而目前 ruff 作为后起之秀大有望其项背的趋势，且更现代化、更便捷、更统一，但其还不太稳定，更新频繁且有肯能 breaking change。本文采用 ruff，两者的 linter 代号基本一致，切换应该不难。

常用的 linter 及代号：

|  代号   |                       原有库                        |                              功能                               |
| :-----: | :-------------------------------------------------: | :-------------------------------------------------------------: |
|  **F**  |                      pyflakes                       |                           code logic                            |
|  **E**  | [pycodestyle](https://github.com/PyCQA/pycodestyle) |          pep8 基础规范,<br /> 不包括 naming docstring           |
|  **W**  |                     pycodestyle                     |                          pep8 基础规范                          |
|    D    |                     pydocstyle                      |         `docstring conventions` <br />补充 pycodestyle          |
|    N    |                     pep8-naming                     |           `naming conventions` <br />补充 pycodestyle           |
|    A    |                   flake8-builtins                   |              是否覆盖了内置变量名<br />pep8-naming              |
|  **B**  |                   flake8-bugbear                    |   (opinionated) 补充 pyflakes<br /> 和 pycodestyle （较杂乱）   |
|   PIE   |                     flake8-pie                      | 没有必要的 pass 或多余 `**` 补充 pyflakes 和 pycodestyle 的杂项 |
| **C4**  |                flake8-comprehensions                |                recommend `comprehension` replace                |
|   DTZ   |                  flake8-datetimez                   |               datetime recommend `tzinfo` replace               |
| **PTH** |                 flake8-use-pathlib                  |                   recommend `pathlib` replace                   |
| **SIM** |                   flake8-simplify                   |              简化语句或调用 recommend code replace              |
| **UP**  |                      pyupgrade                      |                upgrade syntax for newer versions                |
|   ANN   |                 flake8-annotations                  |           函数是否使用 annotations，与 mypy 相辅相成            |
|   TCH   |                flake8-type-checking                 |        imports to move in or out of type-checking blocks        |
|   C90   |                       mccabe                        |                  `function complex` 复杂度检查                  |
|    Q    |                    flake8-quotes                    |                            引号相关                             |
|    S    |                    flake8-bandit                    |                     security 比如硬编码密码                     |
|    I    |                        isort                        |                          import sorted                          |
|   BLE   |                 flake8-blind-except                 |                           检查 except                           |
|   PT    |                 flake8-pytest-style                 |                           pytest 相关                           |
|    G    |                flake8-logging-format                |                    recomad logging use extra                    |

=== "Ruff 0.0.238+"

    ```bash
    # 查看所有支持的 Linter
    ruff linter
    # 查看某个规则的说明
    ruff rule D401
    ```

!!! failure "还不支持的 linter"

    - match case (Structural Pattern Matching)

常用的配置如下：

```toml title="pyproject.toml"
[tool.ruff]
line-length = 120
target-version = "py311"

select = [
    "F",   # pyflake
    "E",   # pycodestyle
    "W",   # pycodestyle
    "B",   # bugbear
    "N",   # naming
    "I",   # isort
    "UP",  # upgrade
    "A",   # builtins
    "S",   # bandit
    "SIM",
    "RUF",
    "PTH",
    "ANN",
]
ignore = ['D211', 'D213']
# include .gitignore
respect-gitignore = true


[tool.ruff.pydocstyle]
# Google-style docstrings
convention = "google"

[tool.ruff.flake8-builtins]
builtins-ignorelist = ["id"]
```
!!! note "持续关注"

    ruff 需要与 black 一同使用，因为 ruff 暂缓了相关 formatter 工作。

检查：

```bash title="RUN"
ruff .
# or
ruff check .
```

### Static type checking with mypy

Python 是强类型动态语言，需要运行时进行类型推导，这是他的优势但也是劣势。为了更好的在静态源码分析时发现问题，引入了 type hint，通过类型检查来增强编码质量和编辑提示。

type annotation ([PEP 3107](https://peps.python.org/pep-3107/))只定义了函数注解，type hint ([PEP 484](https://peps.python.org/pep-0484/))范围更大。

!!! tip

    mypy 是一个在编译时进行类型检查的工具。 pydantic 是一个 runtime type checking。

常用配置选项如下：

```toml title="pyproject.toml"
[tool.mypy]
# 如果不指定则按照 mypy 所依赖的环境
python-version = 3.11
```

!!! tip

    mypy 也可以从项目根目录（当前运行目录）的 pyproject.yoml 读取配置

##  When should they be used?

### coding

使用 vscode editor 时可以安装插件支持 format 和 lint。
或者使用命令行形式。

- [Black Formatter](https://marketplace.visualstudio.com/items?itemName=ms-python.black-formatter) 设置 python 的默认格式化工具即可。
- [Ruff](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff) 提示和自动修复
- mypy 当前环境安装 mypy 后设置如下，或者使用 [Mypy](https://marketplace.visualstudio.com/items?itemName=matangover.mypy) 插件。也可以直接使用 vscode 自带的 pyright（与 mypy 一样是静态类型检查器）
  ```json
    {
        "python.linting.mypyEnabled": true
    }

  ```

### committing

使用 pre-commit hooks，配置 `.pre-commit-config.yaml` 后运行 `pre-commit install` 安装 git hook 脚本。

!!! Warning

    本地项目需要纳入 git 管理，即项目目录下有 `.git` 文件夹。

之后再 `git commit` 都会先运行检查，如果不通过则不能提交。

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/psf/black
    rev: 23.1.0
    hooks:
      - id: black
        language_version: python3.11
  - repo: https://github.com/charliermarsh/ruff-pre-commit
    rev: v0.0.239
    hooks:
      - id: ruff
```
### checked in (CI pipeline)

确保项目库中的代码质量。

```yaml title=".github/workflows/ci.yaml"
name: CI
on: [push]

jobs:
  test:
    name: "Testing"
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.11"]
        pdm-version: ["2.4.2"]
        os: [ubuntu-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - name: Set up PDM (with Python)
        uses: pdm-project.setup-pdm@v3
        with:
          python-version: ${{ matrix.python-version }}
          version: ${{ matrix.pdm-version }}
      - name: Install dependencies
        run: |
          pdm sync -d -G test
      - name: Run Test
        run: |
          pdm run pytest --cov=./ --cov-report=xml tests
      # - name: Upload coverage to Codecov
      #   uses: codecov/codecov-action@v3

  code-quality:
    name: "Code Quality"
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.11"]
        pdm-version: ["2.4.2"]
        os: [ubuntu-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - name: Set up PDM (with Python)
        uses: pdm-project.setup-pdm@v3
        with:
          python-version: ${{ matrix.python-version }}
          version: ${{ matrix.pdm-version }}
      - name: Install dependencies
        run: |
          pdm sync -d
      - name: Run Black
        run: |
          pdm run -v black . --check --exclude=tests/
      - name: Run Ruff
        run: |
          pdm run -v ruff --format=github .
      - name: Run Mypy
        run: |
          pdm run -v mypy . --prety
      - name: Run Safety
        run: pdm run -v safety check
```

## Conclusion

- 编写代码时，使用 black 温和的格式化为符合 pep8 风格的代码，然后利用 ruff 进一步检查代码风格和逻辑，最后再使用 mypy 检查代码类型是否正确，可选的是使用 pytest 进行测试，可以进一步增强代码质量。
- 向仓库提交代码前，利用 pre-commit 再次检查，确保提交的代码符合规范。
- 仓库自我检查，利用 github action 检查仓库中的代码是否符合规范确保其它人也正确提交了代码。

## Next Steps

- mypy 可以检查的类型很多，可以查看文档学习更多的 type hint 技巧
- pytest 也是一项重要且复杂的工作，需要更深入的学习
- 可以利用 github action 做更多有趣的事儿，比如自动生成文档

## Reference

[使用 pyproject.toml 保证代码质量](https://zhu327.github.io/2023/01/09/%E4%BD%BF%E7%94%A8pyproject.toml%E4%BF%9D%E8%AF%81%E4%BB%A3%E7%A0%81%E8%B4%A8%E9%87%8F/)

[An open source Python project CI pipeline](https://brntn.me/blog/open-source-python-ci/)

[Python Code Quality](https://testdriven.io/blog/python-code-quality/)
