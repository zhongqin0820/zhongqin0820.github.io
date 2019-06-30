---
title: 基于Hexo博文生成器的更新时间及其它相关问题
date: 2019-06-05 13:45:06
updated: 2019-06-07 15:20:06
categories:
- 其它

tags:
- 修复问题
---
# 前言
使用更新时间作为博客的排序方式，如果对之前文章进行修改，每次都要手动修改元数据中的`updated`字段。使用IDEA自带的live template或者Sublime中的自定义插件而不需要安装第三方插件就可以较方便的解决这个问题。

以及解决关于每次使用`hexo new`命令都生成对应文章资源文件夹的问题（修改根目录下`_config.yml`中的`post_asset_folder: false`即可避免）。

<!-- more -->
# 解决方法
## IDEA自带的live template
- [图文教程参考链接](https://www.cnblogs.com/chenfangzhi/p/liveTemplate.html)
- 文字描述
    - `Preferences/Settings` -> `Editor` -> `Live Templates`
    - 找到自己希望添加的模板组，如：Markdown，选中后，右上角`+`号新建新的Live Templates。或直接新建模板组（右上角`+`新建模板组，输入组名即可）。
    - 模板字段说明
        - `Abbreviation`：模板名称，相当于快捷键，尽量简略。
        - `Description`：模板描述
        - `Templates text`：正文段，即通过`Abbreviation`替换得到的内容，注意右侧内容
            - `Edit variables`：设置在正文字段定义的变量`$variable$`。
        - 注意，一定要指定生效的文档类型，否则模板替换无法生效。点击最下方`Define`，找不到要指定的特定文件类型就选`Other`。
    - 定义完后，在相应文件中的空白行输入模板名称后，按`tab箭`（可由`Options`字段指定）进行替换。

> 需要注意的指定内容的格式问题。下面给出一个基于Hexo博文生成器的更新时间定义的实例。

### Demo
- Abbreviation：`update`
- Description：`update the date up to the system time`
- Templates text：
```txt
updated: $date$ $time$
```

- Edit variables（注意时间格式的定义）：

| Name | Expression | Default value |
| :--------: | :--------: | :--------: |
| `date`   | `date("yyyy-MM-dd")` | `date("yyyy-MM-dd")` |
| `time`  | `time("HH:mm:ss")` | `time("HH:mm:ss")` |

- No applicable contexts. [Define]().
    - 选择Other后变为：`Applicable in Other`. [Change]().

## Sublime中的自定义插件
> Sublime也可以通过设计模板替换，但是考虑到直接用插件的方式进行替换更方便，因此选用这种方式。

通过`Tools → New Plugin`新建插件命名为`addCurrentTime.py`，并通过`Preference → Key Bindings - User`绑定插件的快捷键为`"add_current_time"`。

- `addCurrentTime.py`代码

```py
import sublime
import sublime_plugin
import datetime

class AddCurrentTimeCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        self.view.run_command("insert_snippet",
            {
                "contents": "updated: %s " % datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            }
        )
```

- `"command": "add_current_time"`文本

```json
{ "keys": ["alt+t"], "command": "add_current_time" }
```

## 取消文章资源文件夹
- 修改`_config.yml`下`post_asset_folder: false`即可避免。
- 其它相关字段说明可以访问[官方文档](https://hexo.io/zh-cn/docs/asset-folders.html)。

# TODO
- [ ] 2019/06/05: 使用`hexo new`能够分别指定文件名（文件名由下划线代替而不是短破折号）以及文章的标题