---
title: vscode python调试
tags: 
categories: Others
date: 2022-09-29 20:30:00
index_img: 
banner_img: 
math: true
---

## 调试文件设置

launch.json

**基本配置**

```json
{
    "name": "Python: Current File",
    "type": "python",
    "request": "launch",
    "program": "${file}",
}
```

**自定义配置**

`name`

出现在vscode调试下拉列表中的调试配置名称，可以根据名称选择不同的调试配置文件

`type`

标识要使用的调试器的类型，即python

`request`

指定开始调试的模式：

- `launch`：在指定的文件上启动调试器 `program`
- `attach`：将调试器附加到已经运行的进程。有关示例，请参阅[远程调试](https://code.visualstudio.com/docs/python/debugging#_remote-debugging)。

`program`

提供python程序入口模块的完全限定路径。推荐值为`${file}`，它使用编辑器中的活动文件。但是，对于具有多个文件的程序，可以指定程序的启动文件。例如：

```json
"program": "/Users/Me/Projects/PokemonGo-Bot/pokemongo_bot/event_handlers/__init__.py",
```

还可以依赖工作区根目录中的相对路径。例如，如果根是`/Users/Me/Projects/PokemonGo-Bot`那么你可以使用以下内容：

```json
"program": "${workspaceFolder}/pokemongo_bot/event_handlers/__init__.py",
```

`pythonPath`

指向Python解释器用于调试目的。如果未指定，则默认为`python.pythonPath`设置中标识的解释器，这相当于使用该值`${config:python.pythonPath}`。要使用不同的解释器，请改为指定其路径。

> 注意：使用conda环境时，需要使用ctrl+shift+p搜索Python：Select interpreter来指定python解释器

`args`

指定传递给Python程序的参数，例如：

```json
"args": [
    "--quiet", "--norepeat"
],
```

`cwd`

指定调试器的当前工作目录，它是代码中使用的任何相对路径的基础文件夹。如果省略，默认为`${workspaceFolder}`（在VS代码中打开的文件夹）。

作为一个例子，说`${workspaceFolder}`包含一个`py_code`文件夹包含`app.py`，和一个`data`文件夹包含`salaries.csv`。如果启动调试器`py_code/app.py`，则数据文件的相对路径根据以下值而变化`cwd`：

| CWD                            | 数据文件的相对路径     |
| ------------------------------ | ---------------------- |
| 省略或 `${workspaceFolder}`    | `data/salaries.csv`    |
| `${workspaceFolder}/py_code`） | `../data/salaries.csv` |
| `${workspaceFolder}/data`      | `salaries.csv`         |

`env`

为除调试器始终继承的系统环境变量之外的调试器进程设置可选的环境变量。例如：

```json
"env":{
    "CUDA_VISIBLE_DEVIECES":"0, 1",
    "DATA_DIR":"../dream/data"
},
```



> https://www.cnblogs.com/it-tsz/p/9022456.html