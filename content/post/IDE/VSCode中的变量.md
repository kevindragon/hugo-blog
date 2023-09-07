---
title: "VSCode中的变量"
date: "2023-09-07T13:08:18+08:00"
tags: ["VSCode", "Visual Studio Code"]
categories: [编辑器]
---

Visual Studio Code 支持调试和任务配置文件中的变量替换以及某些选择设置。 launch.json 和tasks.json 文件中的某些键和值字符串支持使用${variableName} 语法进行变量替换。

## **预定义变量**

### **支持以下预定义变量**

**${userHome}** - 用户主文件夹的路径

**${workspaceFolder}** - 在 VS Code 中打开的文件夹的路径

**${workspaceFolderBasename}** - 在 VS Code 中打开的文件夹的名称，不带任何斜杠 (/)

**${file}** - 当前打开的文件

**${fileWorkspaceFolder}** - 当前打开文件的工作区文件夹

**${relativeFile}** - 相对于workspaceFolder当前打开的文件

**${relativeFileDirname}** - 当前打开的文件相对于workspaceFolder的目录名

**${fileBasename}** - 当前打开的文件的基本名称

**${fileBasenameNoExtension}** - 当前打开的文件的基本名称，没有文件扩展名

**${fileExtname}** - 当前打开的文件的扩展名

**${fileDirname}** - 当前打开文件的文件夹路径

**${fileDirnameBasename}** - 当前打开文件的文件夹名称

**${cwd}** - VS Code 启动时任务运行器的当前工作目录

**${lineNumber}** - 活动文件中当前选定的行号

**${selectedText}** - 活动文件中当前选定的文本

**${execPath}** - 正在运行的 VS Code 可执行文件的路径

**${defaultBuildTask}** - 默认构建任务的名称

**${pathSeparator}** - 操作系统用于分隔文件路径中组件的字符

#### **预定义变量示例**

假设您有以下需求：

1. 位于 `/home/your-username/your-project/folder/file.ext` 的文件在编辑器中打开；
2. 目录 `/home/your-username/your-project` 作为根工作区打开。

**因此，每个变量将具有以下值：**

**${userHome}** - /home/your-username

**${workspaceFolder}** - /home/your-username/your-project

**${workspaceFolderBasename}** - your-project

**${file}** - /home/your-username/your-project/folder/file.ext

**${fileWorkspaceFolder}** - /home/your-username/your-project

**${relativeFile}** - folder/file.ext

**${relativeFileDirname}** - folder

**${fileBasename}** - file.ext

**${fileBasenameNoExtension}** - file

**${fileDirname}** - /home/your-username/your-project/folder

**${fileExtname}** - .ext

**${lineNumber}** - 光标所在的行号

**${selectedText}** - 在代码编辑器中选择的文本

**${execPath}** - Code.exe 的位置

**${pathSeparator}** - 在 macOS 或 Linux 上是 `/` ，在 Windows 上是 `\`

> 提示：在 `task.json` 和 `launch.json` 中可以使用IntelliSense来获得所有预定义的变量
>

### 每个工作目录的变量作用范围

通过将根文件夹的名称附加到变量（用冒号分隔），可以访问工作区的同级根文件夹。如果没有根文件夹名称，则变量的作域为使用它的同一文件夹。

例如，在一个包含多个项目的 `[multi root workspace](https://code.visualstudio.com/docs/editor/multi-root-workspaces)` 的工作区中（包含了`Server`和`Client`项目），`${workspaceFolder:Client}`指的是Client根目录的路径。

## 环境变量

可以通过 `${env:Name}` 语法使用环境变量，例如： `${env:USERNAME}`

```json
{
  "type": "node",
  "request": "launch",
  "name": "Launch Program",
  "program": "${workspaceFolder}/app.js",
  "cwd": "${workspaceFolder}",
  "args": ["${env:USERNAME}"]
}
```

## 配置变量

通过 `${config:Name}` 语法使用VSCode的配置变量（settings），例如访问字体大小 `${config:editor.fontSize}`

## 命令变量

通过`${command:commandID}`语法引用一个命令

一个命令变量所在位置，会被指定的变量运行的结果给替换掉，结果需要是string类型。结果可以从一个简单的非UI计算的命令来实现，一些复杂的功能可以基于扩展的API来实现。如果命令的结果不是字符串类型，可能会发生一些意想不到的结果，结果必须是字符串格式。

这个功能的一个例子是Node.js的调试扩展，此扩展提供了一个交互式的命令`extension.pickNodeProcess` ，来用从运行的Node.js进程不选择一个进程。这个命令返回一个所选的进程ID。这个命令可以在 `Attach by Process Id` 启动配置当中使用：

```json
{
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach by Process ID",
      "processId": "${command:extension.pickNodeProcess}"
    }
  ]
}
```

在 `launch.json` 配置中使用命令变量时，当前的配置对象参数会以对象的形式传递给命令。这样的话，命令就可以知道运行的上下文参数了。

## 输入变量

命令变量已经很强大了，但它们缺乏为特定用例配置正在运行的命令的机制。例如，不可能将提示消息或默认值传递给通用的“用户输入提示”。这种情况就可以通过输入变量来解决了，语法是 `${input:variableId}` 。 `variableID` 指向`launch.json`和`tasks.json`当中的对象，其中指定了额外的配置属性，输入变量不支持嵌套定义。

以下示例展示了使用输入变量的`tasks.json`的结构：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "task name",
      "command": "${input:variableID}"
      // ...
    }
  ],
  "inputs": [
    {
      "id": "variableID",
      "type": "type of input variable"
      // type specific configuration attributes
    }
  ]
}
```

目前VS Code支持三种类型的输入变量：

- **promptString**: 显示一个输入框以从用户处获取字符串类型的内容。
- **pickString**: 显示一个快速选择下拉列表，让用户从多个选项中选择。
- **command**: 执行任意命令。

每种类型都需要额外的配置属性：

`promptString`:

- **description**: 在快速输入中显示，为输入提供上下文。
- **default**: 如果用户没有输入其他内容，将使用的默认值。
- **password**: 设置为 true 以在密码提示下输入，不会显示输入的值。

`pickString`:

- **description**: 在快速选择中显示，为输入提供上下文。
- **options**: 用户可选择的一系列选项。
- **default**: 如果用户没有输入其他内容，将使用的默认值。它必须是选项值之一。

选项可以是字符串值，也可以是包含标签和值的对象。下拉列表将显示：**label: value**

`command`:

- **command**: 在变量插值上运行的命令。
- **args**: 可选的命令参数。

下面示例的 `tasks.json` 展示了使用 `inputs` 来运行Angular CLI：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "ng g",
      "type": "shell",
      "command": "ng",
      "args": ["g", "${input:componentType}", "${input:componentName}"]
    }
  ],
  "inputs": [
    {
      "type": "pickString",
      "id": "componentType",
      "description": "What type of component do you want to create?",
      "options": [
        "component",
        "directive",
        "pipe",
        "service",
        "class",
        "guard",
        "interface",
        "enum"
      ],
      "default": "component"
    },
    {
      "type": "promptString",
      "id": "componentName",
      "description": "Name your component.",
      "default": "my-new-component"
    }
  ]
}
```

运行示例：

![run-input-example.gif](/img/IDE/VSCode/run-input-example.gif)

下面展示了如何在一个调试配置中使用命令类型的用户输入变量，让用户可以在指定目录的所有测试当中选择测试用例。这里假设某个扩展提供了`extension.mochaSupport.testPicker` 命令，该命令可以通过指定目录找到所有的测试用例，并显示一个选择框让用户可以选择其中一个。命令输入的参数由命令本身定义：

```json
{
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Run specific test",
      "program": "${workspaceFolder}/${input:pickTest}"
    }
  ],
  "inputs": [
    {
      "id": "pickTest",
      "type": "command",
      "command": "extension.mochaSupport.testPicker",
      "args": {
        "testFolder": "/out/tests"
      }
    }
  ]
}
```

命令输入也可以与任务一起使用。在这个例子中，使用了内置的“终止任务”命令。它可以接受一个参数来终止所有任务。

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Terminate All Tasks",
      "command": "echo ${input:terminate}",
      "type": "shell",
      "problemMatcher": []
    }
  ],
  "inputs": [
    {
      "id": "terminate",
      "type": "command",
      "command": "workbench.action.tasks.terminate",
      "args": "terminateAll"
    }
  ]
}
```

## 常见问题

### 调试配置或任务中变量替换的详细信息

调试配置或任务中的变量替换是一个两阶段的过程：

- 在第一阶段，所有变量都会被求值为字符串结果。如果一个变量出现多次，它只会被求值一次。
- 在第二次通过中，所有变量都用第一次通过的结果进行替换。

这导致的一个后果是，对变量（例如，在扩展中实现的基于命令的变量）的求值无法访问调试配置或任务中的其他替换变量。它只能看到原始变量。这意味着变量不能相互依赖（这确保了隔离，使替换对求值顺序具有鲁棒性）。

### 用户和工作区设置中是否支持变量替换？

在 `settings.json` 中少数几个预定义的变量是支持替换的，例如： `cwd`, `env`, `shell` and `shellArgs` 的值。一些配置项如 `window.title` 有它自己的固定内容：

`"window.title": "${dirty}${activeEditorShort}${separator}${rootName}${separator}${appName}"`

### 为什么 ${workspaceRoot} 没有文档

变量`${workspaceRoot}` 将会被废弃，推荐使用`${workspaceFolder}` 共更好的支持多根工作区。

### 为什么 tasks.json 中的变量没有被解析？

不是 `tasks.json` 中的所有值都支持变量替换。具体来说，只有 `command`、`args` 和 `options` 支持变量替换。输入部分中的`inputs`不会被解析，因为不支持输入变量的嵌套。

### 我如何知道一个变量的实际值？

检查变量运行时值的一个简单方法是创建一个 VS Code 任务，将变量值输出到控制台。例如，要看到`${workspaceFolder}`的解析值，您可以在 `tasks.json` 中创建并运行（**Terminal** > **Run Task**）以下简单的"echo"任务：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "echo",
      "type": "shell",
      "command": "echo ${workspaceFolder}"
    }
  ]
}
```

## 公众号

欢迎关注我的公众号，同步更新

![木木小小孩](/img/qrcode_for_gh.jpg)
