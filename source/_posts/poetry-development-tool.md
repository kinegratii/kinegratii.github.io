---
title: poetry 开发工具
date: 2018-09-18 21:44:35
categories: 编程
tags:
- 工具
---

[poetry](https://poetry.eustace.io/) 是一个 Python 依赖管理和构建、发布的工具。

在 Python 中，对于初学者来说，打包系统和依赖管理是非常复杂和难懂的。即使对于经验丰富的开发者，一个项目总是要同时创建多个文件： `setup.py` ,`requirements.txt`,`setup.cfg` , `MANIFEST.in` ，还有最新的 `Pipfile`。

基于此， poetry 将所有的配置都放置在一个 toml 文件中，这些配置包括：依赖管理、构建、打包、发布。

poetry 的灵感来自于其他语言的一些工具： composer(PHP) 和 cargo (Rust) 。

<!-- more -->



## TOML文件与配置

TOML 的全称是 Tom’s Obvious, Minimal Language 。TOML是前GitHub CEO， Tom Preston-Werner，于2013年创建的语言，其目标是成为一个小规模的易于使用的语义化配置文件格式。TOML被设计为可以无二义性的转换为一个哈希表(Hash table)。

```toml
title = "TOML 例子"

[owner]
name = "Tom Preston-Werner"
organization = "GitHub"
bio = "GitHub Cofounder & CEO\nLikes tater tots and beer."
dob = 1979-05-27T07:32:00Z

[database]
server = "192.168.1.1"
ports = [ 8001, 8001, 8002 ]
connection_max = 5000
enabled = true
```



在 Python 中可以使用 [toml](https://github.com/uiri/toml) 库读取和写入配置，该库使用非常方面，拥有和 json 一样的接口。

- `toml.load(f, _dict=dict)`
- `toml.loads(s, _dict=dict)`
- `toml.dump(o, f)`
- `toml.dumps(o)`



在 Python 项目中，可以在根目录 `pyproject.toml` 定义相关配置，该名称由 [PEP 518](https://www.python.org/dev/peps/pep-0518/#toml) 所规定，可用于一般的开发工具，不仅仅是 poetry 。

## 命令



 poetry 由一系列的命令所组成，这些命令能够覆盖项目开发的流程。

| 名称    | 功能                                                       | 示例                       |
| ------- | ---------------------------------------------------------- | -------------------------- |
| new     | 创建一个项目脚手架，包含基本结构、pyproject.toml 文件      |                            |
| init    | 基于已有的项目代码创建 pyproject.toml 文件，支持交互式填写 |                            |
| install | 安装依赖库                                                 |                            |
| update  | 更新依赖库                                                 |                            |
| add     | 添加一个或多个依赖库                                       | `poetry add six pytz`      |
| remove  | 移除依赖库                                                 |                            |
| show    | 查看具体依赖库信息，支持显示树形依赖链                     | `poetry show --tree`       |
| build   | 构建 tar.gz 或 wheel 包                                    |                            |
| publish | 发布到 PyPI                                                |                            |
| run     | 运行脚本和代码                                             | `poetry run python app.py` |
|         |                                                            |                            |

备注：

- 和 npm 一样， poetry 将依赖库分为 `main` 和 `dev` 两种渠道，使用 `poetry add -D pyinstaller` 添加为 dev 依赖包



示例：add 命令

```
F:\ArgonApp>poetry add -D pyinstaller
Using version ^3.4 for pyinstaller

Updating dependencies
Resolving dependencies...


Package operations: 4 installs, 0 updates, 0 removals

Writing lock file

  - Installing altgraph (0.16.1)
  - Installing macholib (1.11)
  - Installing pefile (2018.8.8)
  - Installing pyinstaller (3.4)
  
```



查看树形依赖链

```
pyecharts 0.5.11 Python echarts, make charting easier
|-- future *
|-- jinja2 *
|   `-- markupsafe >=0.23
|-- jupyter-echarts-pypkg 0.1.2
|   |-- lml 0.0.2
|   `-- pyecharts-jupyter-installer 0.0.3
|-- lml 0.0.2
|-- pillow *
`-- pyecharts-javascripthon 0.0.6
    `-- javascripthon >=0.10
        |-- dukpy *
        |-- macropy3 1.1.0b2
        `-- setuptools *
pyinstaller 3.4 PyInstaller bundles a Python application and all its dependencies into a single package.
|-- altgraph *
|-- macholib >=1.8
|   `-- altgraph >=0.15
|-- pefile >=2017.8.1
|   `-- future *
`-- setuptools *
pypiwin32 223
`-- pywin32 >=223
pywin32 223 Python for Window Extensions
pywin32-ctypes 0.2.0
```



