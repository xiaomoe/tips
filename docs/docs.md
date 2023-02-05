# Code Documentation Management of Project Workflow

!!! success "Define"

    **Document Type**: user documents and development documents.

    **Document Format**: comment docstring markdown

## 安装依赖

- [mkdocs](https://github.com/mkdocs/mkdocs)  markdown doc 生成
- [mkdocs-material](https://squidfunk.github.io/mkdocs-material/customization/)  mkdocs material style (default google)
- mkdocstrings[python]  # 将 python 文档字符串生成文档
- mdx-include  # 可以使用 `\{\! path \!\}` 形式嵌套 md 文件

## 编写文档

```bash
# 创建 mkdocs 格式
mkdocs new .
# 会生成 docs/index.md 和 mkdocs.yml
```

### 配置

```yaml title="mkdocs.yml"
--8<-- "mkdocs.yml"
```

### md 文档

docs 文件夹下就是编写的 md 文档，编写后在 `mkdocs.yml` 文件的 `nav` 标签下引用即可。

### [PEP257-docstrings](https://peps.python.org/pep-0257/) 文档

在 yml 文件中加入 `plugins` 配置：
```yaml
plugins:
  - mkdocstrings
```

使用 docstring
在 docs/ 下的 markdown 文件中使用 `::: package.module` 自动引入文档字符串

引入一个 python 文件


配合插件

# TODO

## 启动

### 本地启动

```bash
mkdocs serve
```

### 部署

```bash
mkdocs build
# 创建 site/ 文件夹

# github 支持在仓库中创建 gh-pages 分支来提供在线文档，mkdocs 集成了操作，直接
mkdocs gh-deploy
# 该命令会自动build 并将结果推送到绑定的github 仓库下的 gh-pages 分支下
# 网址是：ttps://username.github.io/project-name/
```

## Reference

[Documenting Python Code and Projects](https://testdriven.io/blog/documenting-python/)

[Build Your Python Project Documentation With MkDocs](https://realpython.com/python-project-documentation-with-mkdocs/)
