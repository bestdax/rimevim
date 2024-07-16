# 利用Karabiner实现Squirrel输入法在Vim中的中英文切换

在使用Vim进行文本编辑时，频繁地在中英文之间切换会大大影响编辑效率。Squirrel输入法（鼠须管）是一款优秀的中文输入法，通过与Karabiner配合使用，我们可以实现更加便捷的中英文切换，提升工作效率。本文将介绍如何在macOS上配置Squirrel输入法和Karabiner，实现Vim中的快捷切换。

## 失败的几个方法

- im_select及其Vim插件，这个已经失效了，可以切换输入法，但是不会重置Squirrel的状态(系统自带的输入法可以)
- InputSourceSelector与im_select一样
- 用applescript的keycode来输出shift键，无法切换中英文

## 前提条件

- macOS 操作系统
- 安装了Vim或者neovim编辑器
- 安装了Squirrel输入法
- 安装了Karabiner-Elements

## 安装与配置

### 安装Squirrel输入法

1. 从[Squirrel输入法的GitHub页面](https://github.com/rime/squirrel)下载最新版本的Squirrel输入法。
2. 打开下载的安装包，按照提示完成安装。
3. 安装完成后，可以在系统偏好设置中添加Squirrel输入法。

### 安装Karabiner-Elements

1. 从[Karabiner-Elements的官网](https://karabiner-elements.pqrs.org)下载并安装最新版本的Karabiner-Elements。
2. 打开Karabiner-Elements，按照提示完成初始设置。

### 配置Karabiner-Elements

- 利用Karabiner的简单映射，将`right_command`键改为`right_control`键，这个不是必须的。因为我平时只用到左`command`键，右`command`对我来说是多余的。
- 利用Karabiner的复杂映射，将`caps_lock`大写锁定键在终端下映射为`escape`键和`right_control`键，这样在插入模式用中文输入时按下`caps_lock`键即可回到`NORMAL`模式并切换到英文输入模式。因为我只用终端，所以这里的只写了系统自带的Terminal。如果是用的其他的GUI版本的Vim，请用`osascript -e 'id of app "AppName"'`来查看相应的APP的id号。下面的代码还实现了在终端下长按`caps_lock`键锁定大写的功能。
```json
{
    "description": "Remap Caps Lock: short press for Esc+Ctrl in Terminal, long press for Caps Lock",
    "manipulators": [
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "^com\\.apple\\.Terminal$"
                    ],
                    "type": "frontmost_application_if"
                }
            ],
            "from": {
                "key_code": "caps_lock",
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                }
            },
            "parameters": {
                "basic.simultaneous_threshold_milliseconds": 500
            },
            "to_if_alone": [
                {
                    "key_code": "escape"
                },
                {
                    "key_code": "right_control"
                }
            ],
            "to_if_held_down": [
                {
                    "key_code": "caps_lock"
                }
            ],
            "type": "basic"
        }
    ]
}
```

- 利用Karabiner的复杂映射将`fd`同时按下时输出`gi`和`right_control`，前面的`gi`回到上次从插入模式跳出的地方，`right_control`用来切换成中文模式。
```json
{
    "description": "fd to gi right_control",
    "manipulators": [
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "^com\\.apple\\.Terminal$"
                    ],
                    "type": "frontmost_application_if"
                }
            ],
            "from": {
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                },
                "simultaneous": [
                    {
                        "key_code": "f"
                    },
                    {
                        "key_code": "d"
                    }
                ]
            },
            "to": [
                {
                    "key_code": "g"
                },
                {
                    "key_code": "i"
                },
                {
                    "key_code": "right_control"
                }
            ],
            "type": "basic"
        }
    ]
}
```

- 跟上面类似，将`jk`映射为`a`和`right_control`，在当前位置后面继续插入并切换为中文。
```json
{
    "description": "jk to a right_ctrl",
    "manipulators": [
        {
            "conditions": [
                {
                    "bundle_identifiers": [
                        "^com\\.apple\\.Terminal$"
                    ],
                    "type": "frontmost_application_if"
                }
            ],
            "from": {
                "modifiers": {
                    "optional": [
                        "any"
                    ]
                },
                "simultaneous": [
                    {
                        "key_code": "j"
                    },
                    {
                        "key_code": "k"
                    }
                ]
            },
            "to": [
                {
                    "key_code": "a"
                },
                {
                    "key_code": "right_control"
                }
            ],
            "type": "basic"
        }
    ]
}
```

### 配置Squirrel输入法

将`control`键设为Squirrel的中英文切换键。我以前一直是用`Shift`键来切换的，但是这个有点麻烦，输入大写字母的时候容易切换中英文状态。在`~/Library/Rime/`文件夹中找到`default.custom.yaml`文件，如果没有就新建一个。已经有配置的朋友可以只修改设置`Control_R`和`Control_L`的部分即可。
```yaml
patch:
  menu:
    page_size: 5
  schema_list:
    - schema: wubi86
    - schema: luna_pinyin
    - schema: easy_en

  switcher:
    caption: 切换
    hotkeys:
      - Control+grave
      - Control+s
      - Control+space
  ascii_composer:
    good_old_caps_lock: true # true: 在保持 cap 键原有的特征， false: 点击不会切换大小写
    switch_key:
      Caps_Lock: commit_code # 如果想用 cap 键切换中英文输入，就修改为上面三种的任一一种，否则用 noop
      # Shift_L: inline_ascii
      # Shift_R: commit_code
      Control_L: inline_ascii
      Control_R: commit_code
```

## 说明
我才刚刚使用这个方法来切换中英文状态，自己还有点不习惯，可能以后会根据需求再进行调整。目前的这个方法能够解决一些问题，但是不是智能动态的根据Vim的状态来自动切换的，这个我目前没有办法实现。
