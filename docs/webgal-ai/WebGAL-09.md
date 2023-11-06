# WebGal源码解析 9

# `/packages/webgal/src/translations/zh-cn.ts`

This is a description of a JSON object for a game. It includes information about the game's settings, such as volume levels, language options, and game options. The object has a `title` property that includes the game's main title and subtitle, as well as options for load/saving and extra features. The `gaming` section includes information about the game's saves and how to load/play it. The `extra` property includes additional options and settings for the game.


```js
const zhCn = {
  // 通用
  common: {
    yes: '是',
    no: '否',
  },

  menu: {
    options: {
      title: '选项',
      pages: {
        system: {
          title: '系统',
          options: {
            autoSpeed: {
              title: '自动播放速度',
              options: {
                slow: '慢',
                medium: '中',
                fast: '快',
              },
            },
            language: {
              title: '语言',
            },
            resetData: {
              title: '清除或还原数据',
              options: {
                clearGameSave: '清除所有存档',
                resetSettings: '还原默认设置',
                clearAll: '清除所有数据',
              },
              dialogs: {
                clearGameSave: '确定要清除存档吗',
                resetSettings: '确定要还原默认设置吗',
                clearAll: '确定要清除所有数据吗',
              },
            },
            gameSave: {
              title: '导入或导出存档与选项',
              options: {
                export: '导出存档与选项',
                import: '导入存档与选项',
              },
              dialogs: {
                import: {
                  title: '确定要导入存档与选项吗',
                  tip: '导入存档',
                  error: '存档解析失败',
                },
              },
            },
            about: {
              title: '关于 WebGAL',
              subTitle: 'WebGAL：开源的网页端视觉小说引擎',
              version: '版本号',
              source: '源代码仓库',
              contributors: '贡献者',
              website: '网站',
            },
          },
        },
        display: {
          title: '显示',
          options: {
            textSpeed: {
              title: '文字显示速度',
              options: {
                slow: '慢',
                medium: '中',
                fast: '快',
              },
            },
            textSize: {
              title: '文本大小',
              options: {
                small: '小',
                medium: '中',
                large: '大',
              },
            },
            textFont: {
              title: '文本字体',
              options: {
                siYuanSimSun: '霞鹜文楷',
                SimHei: '黑体',
              },
            },
            textPreview: {
              title: '文本显示预览',
              text: '现在预览的是文本框字体大小和播放速度的情况，您可以根据您的观感调整上面的选项。',
            },
          },
        },
        sound: {
          title: '音频',
          options: {
            volumeMain: { title: '主音量' },
            vocalVolume: { title: '语音音量' },
            bgmVolume: { title: '背景音乐音量' },
            seVolume: { title: '音效音量' },
            uiSeVolume: { title: '用户界面音效音量' },
          },
        },
        // language: {
        //   title: '语言',
        //   options: {
        //   },
        // },
      },
    },
    saving: {
      title: '存档',
      isOverwrite: '是否覆盖存档？',
    },
    loadSaving: {
      title: '读档',
    },
    title: {
      title: '标题',
      options: {
        load: '',
        extra: '鉴赏模式',
      },
    },
    exit: {
      title: '返回',
    },
  },

  title: {
    start: {
      title: '开始游戏',
      subtitle: 'START',
    },
    continue: {
      title: '继续游戏',
      subtitle: 'CONTINUE',
    },
    options: {
      title: '游戏选项',
      subtitle: 'OPTIONS',
    },
    load: {
      title: '读取存档',
      subtitle: 'LOAD',
    },
    extra: {
      title: '鉴赏模式',
      subtitle: 'EXTRA',
    },
  },

  gaming: {
    noSaving: '暂无存档',
    buttons: {
      hide: '隐藏',
      show: '显示',
      backlog: '回想',
      replay: '重播',
      auto: '自动',
      forward: '快进',
      quicklySave: '快速存档',
      quicklyLoad: '快速读档',
      save: '存档',
      load: '读档',
      options: '选项',
      title: '标题',
    },
  },

  extra: {
    title: '鉴赏模式',
  },
};

```

该代码是一个 ES export 语句，表示导出一个名为 "zhCn" 的函数或模块。

在 JavaScript 中，export 语句有以下两种用法：

1. 导出函数或模块

导出函数或模块时，使用 export 语句可以在函数或模块外部使用它们。例如，可以在其他模块中使用以下代码导出一个名为 "zhCn" 的函数：

```jsjavascript
// 导出函数
export function zhCn(arg1, arg2) {
 // 在这里编写代码
}
```

```jsjavascript
// 导出模块
export default function zhCn(arg1, arg2) {
 // 在这里编写代码
}
```

2. 修改父模块

修改父模块时，使用 export 语句可以在当前模块中使用父模块中的函数或模块。父模块中的函数或模块可以通过相对路径访问。例如，在以下情况下，可以使用父模块中的一个名为 "util" 的函数：

```jsjavascript
// 导出函数
import util from 'util';

// 在这里使用 util.min() 函数
const result = util.min(1, 2);
```

在这两种情况下，export 语句都用于告诉其他模块如何使用导出的函数或模块。


```js
export default zhCn;

```

# `/packages/webgal/src/types/angular-expressions.d.ts`

这段代码定义了一个名为 'angular-expressions' 的模块，其中包含一个名为 'compile' 的函数。这个函数接受一个字符串参数 'src'，以及一个可选的参数 'lexerOptions'（参数 'lexerOptions' 的类型未知）。

这个函数返回一个函数，这个函数接受一个字符串参数 'expression'，并且返回一个数值或者是布尔值。这个函数的作用是编译一个 Express 的路由表达式，将其转换为可执行的代码。


```js
declare module 'angular-expressions' {
  export function compile(src: string, lexerOptions?: any): () => number | boolean;
}

```

# `/packages/webgal/src/types/debugProtocol.ts`

这段代码定义了一个名为 `DebugCommand` 的枚举类型，用于表示命令的类型，包括跳转、同步客户端和同步编辑器等操作。接着定义了一个名为 `IDebugMessage` 的接口类型，包含命令类型、场景消息和舞台状态等属性。最后，通过 `import` 语句引入了 `IStageState` 接口类型，未做任何定义，但可以推测它可能是用于在舞台和编辑器之间同步状态的接口。


```js
import { IStageState } from '@/store/stageInterface';

export enum DebugCommand {
  // 跳转
  JUMP,
  // 同步自客户端
  SYNCFC,
  // 同步自编辑器
  SYNCFE,
}

export interface IDebugMessage {
  command: DebugCommand;
  sceneMsg: {
    sentence: number;
    scene: string;
  };
  stageSyncMsg: IStageState;
}

```