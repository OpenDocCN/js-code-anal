# WebGal源码解析 3

# `/packages/webgal/src/Core/controller/gamePlay/stopAllPerform.ts`

这段代码是一个 JavaScript 函数，名为 `stopAllPerform`，属于一个名为 `WebGAL` 的类中。函数的作用是清除游戏中的所有演出，并阻止每个演出在定时器中。

具体来说，这段代码做以下几件事情：

1. 导入 `logger` 函数，从 `@/Core/util/etc/logger` 包中导入。
2. 导入 `WebGAL` 类，从 `@/Core/WebGAL` 包中导入。
3. 定义 `stopAllPerform` 函数，该函数内部有些 `warn` 级别的警告信息，说明这个函数是在做一些危险的操作。
4. `for`  loop 遍历 `WebGAL.gameplay.performController.performList` 数组，其中的 `i` 用来计数当前演出在数组中的位置。
5. 对于 `i` 遍历到的演出，执行以下操作：
	* 调用 `e.stopFunction()`，停止当前演出的运行。
	* 调用 `clearTimeout(e.stopTimeout as unknown as number)`，清除定时器中安排的停止函数。注意，这个 `e.stopTimeout` 参数是一个 `string` 类型的未知值，可能是一个定时器 ID，也可能是另一个参数，总之不是普通的 `timeout` 对象。
	* 移除 `WebGAL.gameplay.performController.performList.splice(i, 1)`，从 `performList` 数组中删除当前演出，使得数组中的元素个数不变。
	* 减去 1，因为已经去除了当前演出，所以不再计数。

这段代码的作用是清除游戏中的所有演出，并阻止每个演出在定时器中，从而保证游戏进程的安全稳定。


```js
import { logger } from '@/Core/util/etc/logger';

import { WebGAL } from '@/Core/WebGAL';

export const stopAllPerform = () => {
  logger.warn('清除所有演出');
  for (let i = 0; i < WebGAL.gameplay.performController.performList.length; i++) {
    const e = WebGAL.gameplay.performController.performList[i];
    e.stopFunction();
    clearTimeout(e.stopTimeout as unknown as number);
    WebGAL.gameplay.performController.performList.splice(i, 1);
    i--;
  }
};

```

# `/packages/webgal/src/Core/controller/scene/callScene.ts`

这段代码是一个 JavaScript 函数，名为 `callScene`，它接受两个参数 `sceneUrl` 和 `sceneName`，分别表示要加载的场景 URL 和场景名称。函数的作用是先将本场景压入场景栈，然后使用 `sceneFetcher` 将场景从 URL 加载到运行时，接着使用 `sceneParser` 将场景解析为 DOM，将场景写入到运行时，并将其入栈。在场景写入完成后，将场景从运行时取出来，并从场景栈中弹出本场景，避免递归加载相同场景。


```js
import { sceneFetcher } from './sceneFetcher';
import { sceneParser } from '../../parser/sceneParser';
import { logger } from '../../util/etc/logger';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';
import uniqWith from 'lodash/uniqWith';
import { scenePrefetcher } from '@/Core/util/prefetcher/scenePrefetcher';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 调用场景
 * @param sceneUrl 场景路径
 * @param sceneName 场景名称
 */
export const callScene = (sceneUrl: string, sceneName: string) => {
  // 先将本场景压入场景栈
  WebGAL.sceneManager.sceneData.sceneStack.push({
    sceneName: WebGAL.sceneManager.sceneData.currentScene.sceneName,
    sceneUrl: WebGAL.sceneManager.sceneData.currentScene.sceneUrl,
    continueLine: WebGAL.sceneManager.sceneData.currentSentenceId,
  });
  // 场景写入到运行时
  sceneFetcher(sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, sceneName, sceneUrl);
    WebGAL.sceneManager.sceneData.currentSentenceId = 0;
    // 开始场景的预加载
    const subSceneList = WebGAL.sceneManager.sceneData.currentScene.subSceneList;
    WebGAL.sceneManager.settledScenes.push(sceneUrl); // 放入已加载场景列表，避免递归加载相同场景
    const subSceneListUniq = uniqWith(subSceneList); // 去重
    scenePrefetcher(subSceneListUniq);
    logger.debug('现在调用场景，调用结果：', WebGAL.sceneManager.sceneData);
    nextSentence();
  });
};

```

# `/packages/webgal/src/Core/controller/scene/changeScene.ts`

该代码是一个 JavaScript 函数，名为 `changeScene`。它接受两个参数 `sceneUrl` 和 `sceneName`，用于指定要切换到的场景。函数返回一个 Promise 对象，其 resolve 状态为已加载的场景。

具体来说，函数内部首先使用 `sceneFetcher` 将场景 URL 解析为 `rawScene`，然后使用 `sceneParser` 将 `rawScene` 解析为 `sceneObject`。接着，使用 `nextSentence` 函数将场景名称添加到已加载的场景列表中，并使用 `scenePrefetcher` 将场景预加载。最后，调用 `logger.debug` 函数输出切换后的场景信息，然后调用 `nextSentence` 函数继续游戏流程。


```js
import { sceneFetcher } from './sceneFetcher';
import { sceneParser } from '../../parser/sceneParser';
import { logger } from '../../util/etc/logger';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';
import uniqWith from 'lodash/uniqWith';
import { scenePrefetcher } from '@/Core/util/prefetcher/scenePrefetcher';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 切换场景
 * @param sceneUrl 场景路径
 * @param sceneName 场景名称
 */
export const changeScene = (sceneUrl: string, sceneName: string) => {
  // 场景写入到运行时
  sceneFetcher(sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, sceneName, sceneUrl);
    WebGAL.sceneManager.sceneData.currentSentenceId = 0;
    // 开始场景的预加载
    const subSceneList = WebGAL.sceneManager.sceneData.currentScene.subSceneList;
    WebGAL.sceneManager.settledScenes.push(sceneUrl); // 放入已加载场景列表，避免递归加载相同场景
    const subSceneListUniq = uniqWith(subSceneList); // 去重
    scenePrefetcher(subSceneListUniq);
    logger.debug('现在切换场景，切换后的结果：', WebGAL.sceneManager.sceneData);
    nextSentence();
  });
};

```

# `/packages/webgal/src/Core/controller/scene/restoreScene.ts`

这段代码是一个JavaScript函数，名为`restoreScene`，它接受一个参数`entry`，表示要恢复的场景。函数内部使用了多个其他函数和模块，如`sceneFetcher`、`sceneParser`、`logger`、`nextSentence`、`WebGAL`等。

函数的作用是读取传入的一个场景URL，并使用`sceneFetcher`将场景文件加载到内存中。然后，使用`sceneParser`将场景文件解析成JavaScript语法，以便后续在WebGAL中使用。接着，将解析后的场景数据存储到`WebGAL`中的场景管理器中，并使用`nextSentence`更新场景进度。最后，输出当前场景的信息，以便下一次调用时使用。


```js
import { sceneFetcher } from './sceneFetcher';
import { sceneParser } from '../../parser/sceneParser';
import { logger } from '../../util/etc/logger';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';
import { ISceneEntry } from '@/Core/Modules/scene';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 恢复场景
 * @param entry 场景入口
 */
export const restoreScene = (entry: ISceneEntry) => {
  // 场景写入到运行时
  sceneFetcher(entry.sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, entry.sceneName, entry.sceneUrl);
    WebGAL.sceneManager.sceneData.currentSentenceId = entry.continueLine + 1; // 重设场景
    logger.debug('现在恢复场景，恢复后场景：', WebGAL.sceneManager.sceneData.currentScene);
    nextSentence();
  });
};

```

# `/packages/webgal/src/Core/controller/scene/sceneFetcher.ts`

这段代码使用了来自axios库的`axios.get()`函数来获取一个场景文件的字符串表示形式。场景文件通常以.json或.xml等格式保存，其中包含了场景的描述信息。

具体来说，这段代码定义了一个名为`sceneFetcher`的函数，该函数接受一个场景文件的路径参数。函数内部使用`axios.get()`函数来获取场景文件，并使用该函数返回一个 Promise 对象。

当场景文件获取成功后，函数内部的回调函数会被调用，并且在回调函数中使用`response.data.toString()`方法将获取到的响应数据转换为字符串形式。然后，`resolve`函数将被调用，并且将转换后的场景字符串作为参数传递给`resolve()`函数，从而最终返回该场景字符串。

由于使用了axios库，所以`import axios from 'axios';`语句用于导入该库并声明`axios`变量。


```js
import axios from 'axios';

/**
 * 原始场景文件获取函数
 * @param sceneUrl 场景文件路径
 */
export const sceneFetcher = (sceneUrl: string) => {
  return new Promise<string>((resolve) => {
    axios.get(sceneUrl).then((response) => {
      const rawScene: string = response.data.toString();
      resolve(rawScene);
    });
  });
};

```

# `/packages/webgal/src/Core/controller/scene/sceneInterface.ts`

这段代码定义了一个枚举类型`commandType`，用于描述游戏中的各种指令类型。

枚举类型`commandType`中定义了以下16个枚举值：

- `say`: 对话指令
- `changeBg`: 更改背景指令
- `changeFigure`: 更改立绘指令
- `bgm`: 更改背景音乐指令
- `video`: 播放视频指令
- `pixi`: pixi演出指令
- `pixiInit`: pixi初始化指令
- `intro`: 黑屏文字演示指令
- `miniAvatar`: 小头像指令
- `changeScene`: 切换场景指令
- `choose`: 分支选择指令
- `end`: 结束游戏指令
- `setComplexAnimation`: 动画演出指令
- `setFilter`: 设置效果指令
- `label`: 标签指令
- `jumpLabel`: 跳转标签指令
- `chooseLabel`: 选择标签指令
- `setVar`: 设置变量指令
- `if`: 条件跳转指令
- `callScene`: 调用场景指令
- `showVars`: 显示变量指令
- `unlockCg`: 解锁背景音乐指令
- `unlockBgm`: 解锁背景音乐指令
- `filmMode`: 设置电影模式指令
- `setTextbox`: 在对话框中输出信息指令
- `setAnimation`: 设置动画指令
- `playEffect`: 播放特效指令
- `setTempAnimation`: 设置临时动画指令
- `comment`: 添加注释指令
- `setTransform`: 设置转换指令
- `setTransition`: 设置过渡指令


```js
/**
 * 语句类型
 */
import { fileType } from '@/Core/util/gameAssetsAccess/assetSetter';
import { ISceneEntry } from '@/Core/Modules/scene';

export enum commandType {
  say, // 对话
  changeBg, // 更改背景
  changeFigure, // 更改立绘
  bgm, // 更改背景音乐
  video, // 播放视频
  pixi, // pixi演出
  pixiInit, // pixi初始化
  intro, // 黑屏文字演示
  miniAvatar, // 小头像
  changeScene, // 切换场景
  choose, // 分支选择
  end, // 结束游戏
  setComplexAnimation, // 动画演出
  setFilter, // 设置效果
  label, // 标签
  jumpLabel, // 跳转标签
  chooseLabel, // 选择标签
  setVar, // 设置变量
  if, // 条件跳转
  callScene, // 调用场景
  showVars,
  unlockCg,
  unlockBgm,
  filmMode,
  setTextbox,
  setAnimation,
  playEffect,
  setTempAnimation,
  comment,
  setTransform,
  setTransition,
}

```

这段代码定义了两个接口：arg和IAsset。

arg接口定义了一个包含两个属性的接口，包括一个string类型的键和一个可以是string、boolean或number类型的值。

IAsset接口定义了一个包含多个属性的接口，包括一个string类型的资源和三个fileType类型的属性，分别是资源名称、资源类型和资源url。此外，还定义了一个number类型的属性lineNumber，用于表示触发资源语句的行号。


```js
/**
 * 单个参数接口
 * @interface arg
 */
export interface arg {
  key: string; // 参数键
  value: string | boolean | number; // 参数值
}

/**
 * 资源接口
 * @interface IAsset
 */
export interface IAsset {
  name: string; // 资源名称
  type: fileType; // 资源类型
  url: string; // 资源url
  lineNumber: number; // 触发资源语句的行号
}

```



这段代码定义了一个名为`ISentence`的接口，用于表示一条单条语句的信息。

该接口包含以下字段：

* `command: commandType`：表示该语句的操作类型，可以是`CommandType.移动`、`CommandType.攻击`等。
* `commandRaw: string`：该语句的原始命令内容，方便调试。
* `content: string`：该语句的具体内容。
* `args: Array<arg>`：该语句所携带的参数列表。
* `sentenceAssets: Array<IAsset>`：该语句所携带的资源列表，用于在游戏引擎中加载需要使用的资源。
* `subScene: Array<string>`：该语句包含的子场景列表，可以用于游戏内场景之间的跳转。

同时，该接口还提供了一些方便的方法，如`ISentence.hasEffect`用于判断一个语句是否对当前场景产生影响，`ISentence.generateSpawnPoint`用于生成一个可挂载的生成器位置。


```js
/**
 * 单条语句接口
 * @interface ISentence
 */
export interface ISentence {
  command: commandType; // 语句类型
  commandRaw: string; // 命令的原始内容，方便调试
  content: string; // 语句内容
  args: Array<arg>; // 参数列表
  sentenceAssets: Array<IAsset>; // 语句携带的资源列表
  subScene: Array<string>; // 语句包含子场景列表
}

/**
 * 场景接口
 * @interface IScene
 */
```

这段代码定义了一个接口IScene，它包含了场景名称、场景URL、语句列表、资源列表、子场景列表这六个属性。

接下来定义了一个ISceneData接口，它继承自IScene接口，并添加了两个属性：当前场景ID和场景栈。

最后，export了两个接口：IScene和ISceneData。IScene接口定义了ISceneData接口中所有的属性，ISceneData接口则是对IScene接口的补充，它包括了当前场景数据这一关键属性。


```js
export interface IScene {
  sceneName: string; // 场景名称
  sceneUrl: string; // 场景url
  sentenceList: Array<ISentence>; // 语句列表
  assetsList: Array<IAsset>; // 资源列表
  subSceneList: Array<string>; // 子场景的url列表
}

/**
 * 当前的场景数据
 * @interface ISceneData
 */
export interface ISceneData {
  currentSentenceId: number; // 当前语句ID
  sceneStack: Array<ISceneEntry>; // 场景栈
  currentScene: IScene; // 当前场景数据
}

```

这段代码定义了一个名为 "parsedCommand" 的接口，用于表示经过处理的命令接口。该接口包含两个属性： "type" 和 "additionalArgs"。

具体来说，"type" 表示命令接口的类型，例如 "GET"、"POST"、"PUT" 等。而 "additionalArgs" 则是一个或多个附加给命令接口的参数，这些参数在执行命令时可以被传递给命令对象。

这段代码的作用是定义了一个用于表示命令接口的接口，方便在应用程序中使用和处理命令接口。


```js
/**
 * 处理后的命令接口
 * @interface parsedCommand
 */
export interface parsedCommand {
  type: commandType;
  additionalArgs: Array<arg>;
}

```

# `/packages/webgal/src/Core/controller/stage/playBgm.ts`

这段代码涉及到多个模块，需要分开解释。

1. `import { webgalStore } from '@/store/store';` - 引入了`webgalStore`模块，可能用于与WebGame后台存储有关的功能。
2. `import { setStage } from '@/store/stageReducer';` - 引入了`setStage`模块，可能用于与后台存储有关的功能。
3. `import { logger } from '@/Core/util/etc/logger';` - 引入了`logger`模块，可能用于与日志有关的功能。
4. `/**` - 从一个位置开始，后面可能还有注释。
5. `export const eraseBgm = () => {...};` - 定义了一个名为`eraseBgm`的函数，这可能是某个后台任务的名称。
6. `logger.debug(`停止bgm`);` - 在logger模块中，将一个日志消息`停止bgm`输出到日志。
7. `let VocalControl: any = document.getElementById('currentBgm');` - 在`eraseBgm`函数中，获取了一个与当前舞台相关的元素（可能是ID为`currentBgm`的音频控制组件）。
8. `if (VocalControl !== null) {` - 如果`VocalControl`不为空，那么执行以下操作。
9. `VocalControl.currentTime = 0;` - 设置`VocalControl`的当前播放时间（或值）为0。
10. `if (!VocalControl.paused) VocalControl.pause();` - 如果`VocalControl`没有被暂停，那么暂停它。
11. `}` - 如果以上代码都执行完毕，那么结束函数。


```js
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';
import { logger } from '@/Core/util/etc/logger';

// /**
//  * 停止bgm
//  */
// export const eraseBgm = () => {
//   logger.debug(`停止bgm`);
//   // 停止之前的bgm
//   let VocalControl: any = document.getElementById('currentBgm');
//   if (VocalControl !== null) {
//     VocalControl.currentTime = 0;
//     if (!VocalControl.paused) VocalControl.pause();
//   }
```

该代码是一个 JavaScript 函数，它使用 webgalStore 状态库来获取舞台状态并设置 bgm 音源。函数接受三个参数，url、enter 和 volume，分别表示 bgm 文件的路径、淡入时间和音量。函数内部先检查传入的参数是否为空，然后根据参数设置对应的舞台状态。接着，函数使用 webgalStore 的 dispatch 方法来更新舞台状态，然后使用 HTML5 的 Audio 元素来播放 bgm 文件。


```js
//   // 获得舞台状态并设置
//   webgalStore.dispatch(setStage({key: 'bgm', value: ''}));
// };

/**
 * 播放bgm
 * @param url bgm路径
 * @param enter 淡入时间（单位毫秒）
 * @param volume 背景音乐 音量调整（0 - 100）
 */
export function playBgm(url: string, enter = 0, volume = 100): void {
  logger.info('playing bgm' + url);
  if (url === '') {
    const lastSrc = webgalStore.getState().stage.bgm.src;
    webgalStore.dispatch(setStage({ key: 'bgm', value: { src: lastSrc, enter: -enter, volume: volume } }));
  } else {
    webgalStore.dispatch(setStage({ key: 'bgm', value: { src: url, enter: enter, volume: volume } }));
  }
  const audioElement = document.getElementById('currentBgm') as HTMLAudioElement;
  if (audioElement.src) {
    audioElement?.play();
  }
}

```

# `/packages/webgal/src/Core/controller/stage/resetStage.ts`

这段代码是一个 Redux 舞台 Reducer，作用是处理舞台场景和变量状态。主要包含以下几个方法：

1. `resetStage`：清空舞台运行时状态，包括清除 backlog、场景和变量，并重新获取。

2. `setStage`：设置舞台初始状态。

3. `cloneDeep`：对传入的对象进行深度克隆。

4. `WebGAL`：导入自 `@/Core/WebGAL` 模块，用于与 WebGAL 游戏服务器通信。

`resetStage` 方法首先判断是否设置了清除 backlog，如果是，则创建一个空 backlog 并清空它。接着，分别执行清空场景和变量的操作，最后清空舞台状态表并设置初始状态。

`setStage` 方法接受一个初始状态对象，然后将其与舞台初始状态进行比较，如果没有设置初始状态，则将 `GameVar` 设置为当前场景和变量的值。

`cloneDeep` 方法用于对传入的对象进行深度克隆，以便在需要时进行依赖拦截。

`WebGAL` 模块导入来自 `@/Core/WebGAL` 模块，用于与 WebGAL 游戏服务器通信。


```js
import { initState, resetStageState, setStage } from '@/store/stageReducer';
import { webgalStore } from '@/store/store';
import cloneDeep from 'lodash/cloneDeep';
import { WebGAL } from '@/Core/WebGAL';

export const resetStage = (resetBacklog: boolean, resetSceneAndVar = true) => {
  /**
   * 清空运行时
   */
  if (resetBacklog) {
    WebGAL.backlogManager.makeBacklogEmpty();
  }
  // 清空sceneData，并重新获取
  if (resetSceneAndVar) {
    WebGAL.sceneManager.resetScene();
  }

  // 清空所有演出和timeOut
  WebGAL.gameplay.performController.removeAllPerform();
  WebGAL.gameplay.resetGamePlay();

  // 清空舞台状态表
  const initSceneDataCopy = cloneDeep(initState);
  const currentVars = webgalStore.getState().stage.GameVar;
  webgalStore.dispatch(resetStageState(initSceneDataCopy));
  if (!resetSceneAndVar) {
    webgalStore.dispatch(setStage({ key: 'GameVar', value: currentVars }));
  }
};

```

# `/packages/webgal/src/Core/controller/stage/setVolume.ts`

这段代码是一个 JavaScript 函数，使用了 `@/store/store` 模块中的 `logger` 类，用于设置电脑的音量。具体来说，它实现了以下功能：

1. 读取 `userDataState` 对象中的 `optionData` 属性，这个属性里可能包含 `volumeMain` 和 `vocalVolume` 两个属性，分别表示主音量和 vocal 音量。
2. 根据 `optionData` 中的 volume 主音量，计算出背景音量和 vocal 音量。
3. 使用 `logger.debug` 方法输出设置的背景音量和 vocal 音量。
4. 设置背景音量的元素（如果已经存在的话），将值设置为 `bgmVolume` 对象中的值；设置 vocal 音量的元素（如果已经存在的话），将值设置为 `vocalVolume` 对象中的值。

设置电脑音量的函数，主要用于在需要改变音量时提供用户一个便利。


```js
import { logger } from '../../util/etc/logger';
import { webgalStore } from '@/store/store';

/**
 * 设置音量
 */
export const setVolume = () => {
  const userDataState = webgalStore.getState().userData;
  const mainVol = userDataState.optionData.volumeMain;
  const vocalVol = mainVol * 0.01 * userDataState.optionData.vocalVolume * 0.01;
  const bgmVol = mainVol * 0.01 * userDataState.optionData.bgmVolume * 0.01;
  logger.debug(`设置背景音量：${bgmVol},语音音量：${vocalVol}`);
  // const bgmElement: any = document.getElementById('currentBgm');
  // if (bgmElement) {
  //   bgmElement.volume = bgmVol.toString();
  // }
  // const vocalElement: any = document.getElementById('currentVocal');
  // if (vocalElement) {
  //   vocalElement.volume = vocalVol.toString();
  // }
};

```

# `/packages/webgal/src/Core/controller/stage/pixi/PixiController.ts`

这段代码是一个 JavaScript 应用程序，使用了 Pixi.js 和 WebGALPixiContainer，其中包含了两个主要模块：StageReducer 和 StageInterface。

1. StageReducer 模块

这个模块可能包含了游戏中的多个状态，以及如何更新这些状态。具体来说，它可能包含了以下信息：

- 游戏中的玩家状态(如玩家所处的地图、目标、角色等)
- 游戏中的敌人状态(如敌人当前的状态、位置等)
- 游戏中的道具状态(如游戏中的魔法、技能等)
- 游戏中的动画状态(如角色的动画、特效等)

这些信息可能会动态地变化，并且每个状态都需要在游戏中进行更新。

2. StageInterface 模块

这个模块可能包含了与游戏中的状态进行交互的方法。具体来说，它可能包含了以下信息：

- 创建游戏中的新状态
- 更新游戏中的状态
- 获取游戏中的状态信息

这些信息可以帮助游戏开发者更好地管理游戏中的状态和状态之间的转换。

此外，还 import 了 Pixi 和 uuid 模块，以及 WebGALPixiContainer 组件，可能与游戏中的图形渲染和 UUID 相关。同时，还 import 了 Live2DModel 和 SoundManager 模块，如果要在游戏中使用 Live2D 组件，那么这个模块的 import 可能会被取消。


```js
import * as PIXI from 'pixi.js';
import { v4 as uuid } from 'uuid';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';
import cloneDeep from 'lodash/cloneDeep';
import { IEffect, IFigureAssociatedAnimation } from '@/store/stageInterface';
import { logger } from '@/Core/util/etc/logger';
import { isIOS } from '@/Core/initializeScript';
import { WebGALPixiContainer } from '@/Core/controller/stage/pixi/WebGALPixiContainer';
// import { Live2DModel, SoundManager } from 'pixi-live2d-display'; // 如果要使用 Live2D，取消这里的注释

export interface IAnimationObject {
  setStartState: Function;
  setEndState: Function;
  tickerFunc: PIXI.TickerCallback<number>;
  getEndFilterEffect?: Function;
}

```

这两段代码定义了两个接口：`IStageAnimationObject` 和 `IStageObject`。

`IStageAnimationObject` 接口描述了一个舞台动画对象，它具有唯一标识 `uuid`、与动画对象作用目标相关的 `key` 和 `targetKey`、动画对象的类型 `type`（可以是 'common' 或 'preset'）以及一个 `animationObject`。

`IStageObject` 接口描述了一个舞台对象，它具有唯一标识 `uuid`、与动画对象作用目标相关的 `key` 和 `pixiContainer`（ Pixi 是一个流行的 2D 图形库，它提供了一个用于创建和操作 2D 图形的高级 API）以及一个与动画相关的源 URL `sourceUrl`。


```js
interface IStageAnimationObject {
  // 唯一标识
  uuid: string;
  // 一般与作用目标有关
  key: string;
  targetKey?: string;
  type: 'common' | 'preset';
  animationObject: IAnimationObject;
}

export interface IStageObject {
  // 唯一标识
  uuid: string;
  // 一般与作用目标有关
  key: string;
  pixiContainer: PIXI.Container;
  // 相关的源 url
  sourceUrl: string;
}

```

This is a JavaScript class called `Pixi. loader`. It appears to be a loading manager for the Pixi.js library, which is a 2D drawing library for the web.

The `Pixi. loader` class appears to handle loading and caching of assets, as well as updating the current cache and animation state. It has the following methods:

* `load(assetUrl: string, callback: (result) => void)`: This method loads an asset and invokes the callback with the loaded data and a success/error message. The loaded data can be retrieved by calling `.resources` on the loaded asset.
* `cacheGC()`: This method clears the texture cache used by Pixi.js.
* `loadAsset(assetUrl: string, callback: () => void)`: This is a wrapper for the `load` method that tries to retrieve the texture at the given URL. It is intended to be used in place of `load` with an empty callback, like this:
```js
this.loadQueue.push({ url, callback });
this.callLoader();
```
* `callLoader()`: This method calls the `load` method for each asset in the queue. It tries to retrieve the texture at the URL specified in the queue, and if it is not found, it pushes the current asset to the queue again.
* `updateCurrentEffects(currentEffects: PIXI.Effect[])`: This method updates the current animation state by calling the `updateCurrentEffects` method on each effect object in the currentEffects array.
* `updateScreen()`: This method updates the current screen based on the FPS count, it will update the frame duration, and the texture/image used for the screen.
* `render()`: This method is called when `render` is called on the container. It will attempt to make the app responsive by updating the screen at a consistent FPS, and will also try to optimize the rendering loop by using the `.createScene` method to create a new scene for high performance.

It also has some utility methods like `.splitEffectIntoFrames(effect: PIXI.Effect)`, `.addCount(count: number)` and `.cacheTexture(textureUrl: string)`

It is not clear where this class is used and how it is intended to be used in the Pixi.js app development process.


```js
// export interface IRegisterTickerOpr {
//   tickerGeneratorFn: (targetKey: string, duration: number) => PIXI.TickerCallback<number>;
//   key: string;
//   target: string;
//   duration: number;
// }

// @ts-ignore
window.PIXI = PIXI;

export default class PixiStage {
  /**
   * 当前的 PIXI App
   */
  public currentApp: PIXI.Application | null = null;
  public readonly effectsContainer: PIXI.Container;
  public frameDuration = 16.67;
  private readonly figureContainer: PIXI.Container;
  private figureObjects: Array<IStageObject> = [];
  private readonly backgroundContainer: PIXI.Container;
  private backgroundObjects: Array<IStageObject> = [];

  // 注册到 Ticker 上的函数
  private stageAnimations: Array<IStageAnimationObject> = [];
  private assetLoader = new PIXI.Loader();
  private loadQueue: { url: string; callback: () => void }[] = [];

  // 锁定变换对象（对象可能正在执行动画，不能应用变换）
  private lockTransformTarget: Array<string> = [];
  private stageWidth = 2560;
  private stageHeight = 1440;
  /**
   * 暂时没用上，以后可能用
   * @private
   */
  private MAX_TEX_COUNT = 10;

  public constructor() {
    const app = new PIXI.Application({
      backgroundAlpha: 0,
      preserveDrawingBuffer: true,
    });
    // @ts-ignore

    window.PIXIapp = this; // @ts-ignore
    window.__PIXI_APP__ = app;
    // 清空原节点
    const pixiContainer = document.getElementById('pixiContianer');
    if (pixiContainer) {
      pixiContainer.innerHTML = '';
      pixiContainer.appendChild(app.view);
    }

    // 设置样式
    app.renderer.view.style.position = 'absolute';
    app.renderer.view.style.display = 'block';
    app.renderer.view.id = 'pixiCanvas';
    // @ts-ignore
    app.renderer.autoResize = true;
    const appRoot = document.getElementById('root');
    if (appRoot) {
      app.renderer.resize(appRoot.clientWidth, appRoot.clientHeight);
    }
    if (isIOS) {
      app.renderer.view.style.zIndex = '-5';
    }

    // 设置可排序
    app.stage.sortableChildren = true;

    // 添加 3 个 Container 用于做渲染
    this.effectsContainer = new PIXI.Container();
    this.effectsContainer.zIndex = 3;
    this.figureContainer = new PIXI.Container();
    this.figureContainer.zIndex = 2;
    this.backgroundContainer = new PIXI.Container();
    this.backgroundContainer.zIndex = 0;
    app.stage.addChild(this.effectsContainer, this.figureContainer, this.backgroundContainer);
    this.currentApp = app;
    // 每 5s 获取帧率
    const update = () => {
      this.updateFps();
      setTimeout(update, 10000);
    };
    update();
  }

  public getFigureObjects() {
    return this.figureObjects;
  }

  public getAllLockedObject() {
    return this.lockTransformTarget;
  }

  /**
   * 注册动画
   * @param animationObject
   * @param key
   * @param target
   */
  public registerAnimation(animationObject: IAnimationObject | null, key: string, target = 'default') {
    if (!animationObject) return;
    this.stageAnimations.push({ uuid: uuid(), animationObject, key: key, targetKey: target, type: 'common' });
    // 上锁
    this.lockStageObject(target);
    animationObject.setStartState();
    this.currentApp?.ticker.add(animationObject.tickerFunc);
  }

  /**
   * 注册预设动画
   * @param animationObject
   * @param key
   * @param target
   * @param currentEffects
   */
  // eslint-disable-next-line max-params
  public registerPresetAnimation(
    animationObject: IAnimationObject | null,
    key: string,
    target = 'default',
    currentEffects: IEffect[],
  ) {
    if (!animationObject) return;
    const effect = currentEffects.find((effect) => effect.target === target);
    if (effect) {
      const targetPixiContainer = this.getStageObjByKey(target);
      if (targetPixiContainer) {
        const container = targetPixiContainer.pixiContainer;
        Object.assign(container, effect.transform);
      }
      return;
    }
    this.stageAnimations.push({ uuid: uuid(), animationObject, key: key, targetKey: target, type: 'preset' });
    // 上锁
    this.lockStageObject(target);
    animationObject.setStartState();
    this.currentApp?.ticker.add(animationObject.tickerFunc);
  }

  public stopPresetAnimationOnTarget(target: string) {
    const targetPresetAnimations = this.stageAnimations.find((e) => e.targetKey === target && e.type === 'preset');
    if (targetPresetAnimations) {
      this.removeAnimation(targetPresetAnimations.key);
    }
  }

  /**
   * 移除动画
   * @param key
   */
  public removeAnimation(key: string) {
    const index = this.stageAnimations.findIndex((e) => e.key === key);
    if (index >= 0) {
      const thisTickerFunc = this.stageAnimations[index];
      this.currentApp?.ticker.remove(thisTickerFunc.animationObject.tickerFunc);
      thisTickerFunc.animationObject.setEndState();
      this.unlockStageObject(thisTickerFunc.targetKey ?? 'default');
      this.stageAnimations.splice(index, 1);
    }
  }

  public removeAnimationWithSetEffects(key: string) {
    const index = this.stageAnimations.findIndex((e) => e.key === key);
    if (index >= 0) {
      const thisTickerFunc = this.stageAnimations[index];
      this.currentApp?.ticker.remove(thisTickerFunc.animationObject.tickerFunc);
      thisTickerFunc.animationObject.setEndState();
      const webgalFilters = thisTickerFunc.animationObject.getEndFilterEffect?.() ?? {};
      this.unlockStageObject(thisTickerFunc.targetKey ?? 'default');
      if (thisTickerFunc.targetKey) {
        const target = this.getStageObjByKey(thisTickerFunc.targetKey);
        if (target) {
          const targetTransform = {
            alpha: target.pixiContainer.alpha,
            scale: {
              x: target.pixiContainer.scale.x,
              y: target.pixiContainer.scale.y,
            },
            pivot: {
              x: target.pixiContainer.pivot.x,
              y: target.pixiContainer.pivot.y,
            },
            position: {
              x: target.pixiContainer.x,
              y: target.pixiContainer.y,
            },
            rotation: target.pixiContainer.rotation,
            // @ts-ignore
            blur: target.pixiContainer.blur,
            ...webgalFilters,
          };
          const prevEffects = webgalStore.getState().stage.effects;
          const newEffects = cloneDeep(prevEffects);
          let effect: IEffect = {
            target: thisTickerFunc.targetKey,
            transform: targetTransform,
          };
          const index = newEffects.findIndex((e) => e.target === thisTickerFunc.targetKey);
          if (index >= 0) {
            effect = newEffects[index];
            effect.transform = targetTransform;
            newEffects[index] = effect;
          } else {
            newEffects.push(effect);
          }
          updateCurrentEffects(newEffects);
        }
      }
      this.stageAnimations.splice(index, 1);
    }
  }

  // eslint-disable-next-line max-params
  public performMouthSyncAnimation(
    key: string,
    targetAnimation: IFigureAssociatedAnimation,
    mouthState: string,
    presetPosition: string,
  ) {
    const currentFigure = this.getStageObjByKey(key)?.pixiContainer as WebGALPixiContainer;

    if (!currentFigure) {
      return;
    }

    const mouthTextureUrls: any = {
      open: targetAnimation.mouthAnimation.open,
      half_open: targetAnimation.mouthAnimation.halfOpen,
      closed: targetAnimation.mouthAnimation.close,
    };

    // Load mouth texture (reuse if already loaded)
    this.loadAsset(mouthTextureUrls[mouthState], () => {
      const texture = this.assetLoader.resources[mouthTextureUrls[mouthState]].texture;
      if (!texture) {
        return;
      }
      const originalWidth = texture.width;
      const originalHeight = texture.height;
      const scaleX = this.stageWidth / originalWidth;
      const scaleY = this.stageHeight / originalHeight;
      const targetScale = Math.min(scaleX, scaleY);
      const figureSprite = new PIXI.Sprite(texture);
      figureSprite.scale.x = targetScale;
      figureSprite.scale.y = targetScale;
      figureSprite.anchor.set(0.5);
      figureSprite.position.y = this.stageHeight / 2;
      const targetWidth = originalWidth * targetScale;
      const targetHeight = originalHeight * targetScale;
      currentFigure.setBaseY(this.stageHeight / 2);
      if (targetHeight < this.stageHeight) {
        currentFigure.setBaseY(this.stageHeight / 2 + this.stageHeight - targetHeight / 2);
      }
      if (presetPosition === 'center') {
        currentFigure.setBaseX(this.stageWidth / 2);
      }
      if (presetPosition === 'left') {
        currentFigure.setBaseX(targetWidth / 2);
      }
      if (presetPosition === 'right') {
        currentFigure.setBaseX(this.stageWidth - targetWidth / 2);
      }
      currentFigure.pivot.set(0, this.stageHeight / 2);
      currentFigure.addChild(figureSprite);
    });
  }

  // eslint-disable-next-line max-params
  public performBlinkAnimation(
    key: string,
    targetAnimation: IFigureAssociatedAnimation,
    blinkState: string,
    presetPosition: string,
  ) {
    const currentFigure = this.getStageObjByKey(key)?.pixiContainer as WebGALPixiContainer;

    if (!currentFigure) {
      return;
    }
    const blinkTextureUrls: any = {
      open: targetAnimation.blinkAnimation.open,
      closed: targetAnimation.blinkAnimation.close,
    };

    // Load eye texture (reuse if already loaded)
    this.loadAsset(blinkTextureUrls[blinkState], () => {
      const texture = this.assetLoader.resources[blinkTextureUrls[blinkState]].texture;

      if (!texture) {
        return;
      }

      const originalWidth = texture.width;
      const originalHeight = texture.height;
      const scaleX = this.stageWidth / originalWidth;
      const scaleY = this.stageHeight / originalHeight;
      const targetScale = Math.min(scaleX, scaleY);
      const figureSprite = new PIXI.Sprite(texture);
      figureSprite.scale.x = targetScale;
      figureSprite.scale.y = targetScale;
      figureSprite.anchor.set(0.5);
      figureSprite.position.y = this.stageHeight / 2;
      const targetWidth = originalWidth * targetScale;
      const targetHeight = originalHeight * targetScale;
      currentFigure.setBaseY(this.stageHeight / 2);
      if (targetHeight < this.stageHeight) {
        currentFigure.setBaseY(this.stageHeight / 2 + this.stageHeight - targetHeight / 2);
      }
      if (presetPosition === 'center') {
        currentFigure.setBaseX(this.stageWidth / 2);
      }
      if (presetPosition === 'left') {
        currentFigure.setBaseX(targetWidth / 2);
      }
      if (presetPosition === 'right') {
        currentFigure.setBaseX(this.stageWidth - targetWidth / 2);
      }
      currentFigure.pivot.set(0, this.stageHeight / 2);
      currentFigure.addChild(figureSprite);
    });
  }

  /**
   * 添加背景
   * @param key 背景的标识，一般和背景类型有关
   * @param url 背景图片url
   */
  public addBg(key: string, url: string) {
    // const loader = this.assetLoader;
    const loader = this.assetLoader;
    // 准备用于存放这个背景的 Container
    const thisBgContainer = new WebGALPixiContainer();

    // 是否有相同 key 的背景
    const setBgIndex = this.backgroundObjects.findIndex((e) => e.key === key);
    const isBgSet = setBgIndex >= 0;

    // 已经有一个这个 key 的背景存在了
    if (isBgSet) {
      // 挤占
      this.removeStageObjectByKey(key);
    }

    // 挂载
    this.backgroundContainer.addChild(thisBgContainer);
    const bgUuid = uuid();
    this.backgroundObjects.push({ uuid: bgUuid, key: key, pixiContainer: thisBgContainer, sourceUrl: url });

    // 完成图片加载后执行的函数
    const setup = () => {
      const texture = loader.resources?.[url]?.texture;
      if (texture && this.getStageObjByUuid(bgUuid)) {
        /**
         * 重设大小
         */
        const originalWidth = texture.width;
        const originalHeight = texture.height;
        const scaleX = this.stageWidth / originalWidth;
        const scaleY = this.stageHeight / originalHeight;
        const targetScale = Math.max(scaleX, scaleY);
        const bgSprite = new PIXI.Sprite(texture);
        bgSprite.scale.x = targetScale;
        bgSprite.scale.y = targetScale;
        bgSprite.anchor.set(0.5);
        bgSprite.position.y = this.stageHeight / 2;
        thisBgContainer.setBaseX(this.stageWidth / 2);
        thisBgContainer.setBaseY(this.stageHeight / 2);
        thisBgContainer.pivot.set(0, this.stageHeight / 2);

        // 挂载
        thisBgContainer.addChild(bgSprite);
      }
    };

    /**
     * 加载器部分
     */
    const resourses = Object.keys(loader.resources);
    this.cacheGC();
    if (!resourses.includes(url)) {
      this.loadAsset(url, setup);
    } else {
      // 复用
      setup();
    }
  }

  /**
   * 添加立绘
   * @param key 立绘的标识，一般和立绘位置有关
   * @param url 立绘图片url
   * @param presetPosition
   */
  public addFigure(key: string, url: string, presetPosition: 'left' | 'center' | 'right' = 'center') {
    const loader = this.assetLoader;
    // 准备用于存放这个立绘的 Container
    const thisFigureContainer = new WebGALPixiContainer();

    // 是否有相同 key 的立绘
    const setFigIndex = this.figureObjects.findIndex((e) => e.key === key);
    const isFigSet = setFigIndex >= 0;

    // 已经有一个这个 key 的立绘存在了
    if (isFigSet) {
      this.removeStageObjectByKey(key);
    }

    // 挂载
    this.figureContainer.addChild(thisFigureContainer);
    const figureUuid = uuid();
    this.figureObjects.push({ uuid: figureUuid, key: key, pixiContainer: thisFigureContainer, sourceUrl: url });

    // 完成图片加载后执行的函数
    const setup = () => {
      const texture = loader.resources?.[url]?.texture;
      if (texture && this.getStageObjByUuid(figureUuid)) {
        /**
         * 重设大小
         */
        const originalWidth = texture.width;
        const originalHeight = texture.height;
        const scaleX = this.stageWidth / originalWidth;
        const scaleY = this.stageHeight / originalHeight;
        const targetScale = Math.min(scaleX, scaleY);
        const figureSprite = new PIXI.Sprite(texture);
        figureSprite.scale.x = targetScale;
        figureSprite.scale.y = targetScale;
        figureSprite.anchor.set(0.5);
        figureSprite.position.y = this.stageHeight / 2;
        const targetWidth = originalWidth * targetScale;
        const targetHeight = originalHeight * targetScale;
        thisFigureContainer.setBaseY(this.stageHeight / 2);
        if (targetHeight < this.stageHeight) {
          thisFigureContainer.setBaseY(this.stageHeight / 2 + this.stageHeight - targetHeight / 2);
        }
        if (presetPosition === 'center') {
          thisFigureContainer.setBaseX(this.stageWidth / 2);
        }
        if (presetPosition === 'left') {
          thisFigureContainer.setBaseX(targetWidth / 2);
        }
        if (presetPosition === 'right') {
          thisFigureContainer.setBaseX(this.stageWidth - targetWidth / 2);
        }
        thisFigureContainer.pivot.set(0, this.stageHeight / 2);
        thisFigureContainer.addChild(figureSprite);
      }
    };

    /**
     * 加载器部分
     */
    const resourses = Object.keys(loader.resources);
    this.cacheGC();
    if (!resourses.includes(url)) {
      this.loadAsset(url, setup);
    } else {
      // 复用
      setup();
    }
  }

  /**
   * Live2d立绘，如果要使用 Live2D，取消这里的注释
   * @param jsonPath
   */
  // eslint-disable-next-line max-params
  // public addLive2dFigure(key: string, jsonPath: string, pos: string, motion: string) {
  //   let stageWidth = this.stageWidth;
  //   let stageHeight = this.stageHeight;
  //   logger.debug('Using motion:', motion);
  //
  //   figureCash.push(jsonPath);
  //
  //   const loader = this.assetLoader;
  //   // 准备用于存放这个立绘的 Container
  //   const thisFigureContainer = new WebGALPixiContainer();
  //
  //   // 是否有相同 key 的立绘
  //   const setFigIndex = this.figureObjects.findIndex((e) => e.key === key);
  //   const isFigSet = setFigIndex >= 0;
  //
  //   // 已经有一个这个 key 的立绘存在了
  //   if (isFigSet) {
  //     this.removeStageObjectByKey(key);
  //   }
  //
  //   // 挂载
  //   this.figureContainer.addChild(thisFigureContainer);
  //   this.figureObjects.push({ uuid: uuid(), key: key, pixiContainer: thisFigureContainer, sourceUrl: jsonPath });
  //
  //   const setup = () => {
  //     if (thisFigureContainer) {
  //       (async function () {
  //         const models = await Promise.all([Live2DModel.from(jsonPath)]);
  //
  //         models.forEach((model) => {
  //           const scaleX = stageWidth / model.width;
  //           const scaleY = stageHeight / model.height;
  //           const targetScale = Math.min(scaleX, scaleY) * 1.5;
  //           const targetWidth = model.width * targetScale;
  //           // const targetHeight = model.height * targetScale;
  //
  //           model.scale.set(targetScale);
  //           model.anchor.set(0.5);
  //           model.position.x = stageWidth / 2;
  //           model.position.y = stageHeight / 1.2;
  //
  //           if (pos === 'left') {
  //             model.position.x = targetWidth / 2;
  //           }
  //           if (pos === 'right') {
  //             model.position.x = stageWidth - targetWidth / 2;
  //           }
  //
  //           let category_name = motion;
  //           let animation_index = 0;
  //           let priority_number = 3;
  //           // var audio_link = voiceCash.pop();
  //
  //           // model.motion(category_name, animation_index, priority_number,location.href + audio_link);
  //           model.motion(category_name, animation_index, priority_number);
  //
  //           // lip-sync is still a problem and you can not.
  //           SoundManager.volume = 0;
  //           thisFigureContainer.addChild(model);
  //         });
  //       })();
  //     }
  //   };
  //
  //   /**
  //    * 加载器部分
  //    */
  //   const resourses = Object.keys(loader.resources);
  //   this.cacheGC();
  //   if (!resourses.includes(jsonPath)) {
  //     this.loadAsset(jsonPath, setup);
  //   } else {
  //     // 复用
  //     setup();
  //   }
  // }

  /**
   * 根据 key 获取舞台上的对象
   * @param key
   */
  public getStageObjByKey(key: string) {
    return [...this.figureObjects, ...this.backgroundObjects].find((e) => e.key === key);
  }

  public getStageObjByUuid(objUuid: string) {
    return [...this.figureObjects, ...this.backgroundObjects].find((e) => e.uuid === objUuid);
  }

  public getAllStageObj() {
    return [...this.figureObjects, ...this.backgroundObjects];
  }

  /**
   * 根据 key 删除舞台上的对象
   * @param key
   */
  public removeStageObjectByKey(key: string) {
    const indexFig = this.figureObjects.findIndex((e) => e.key === key);
    const indexBg = this.backgroundObjects.findIndex((e) => e.key === key);
    if (indexFig >= 0) {
      const bgSprite = this.figureObjects[indexFig];
      bgSprite.pixiContainer.destroy();
      this.figureContainer.removeChild(bgSprite.pixiContainer);
      this.figureObjects.splice(indexFig, 1);
    }
    if (indexBg >= 0) {
      const bgSprite = this.backgroundObjects[indexBg];
      bgSprite.pixiContainer.destroy();
      this.backgroundContainer.removeChild(bgSprite.pixiContainer);
      this.backgroundObjects.splice(indexBg, 1);
    }
    // /**
    //  * 删掉相关 Effects，因为已经移除了
    //  */
    // const prevEffects = webgalStore.getState().stage.effects;
    // const newEffects = __.cloneDeep(prevEffects);
    // const index = newEffects.findIndex((e) => e.target === key);
    // if (index >= 0) {
    //   newEffects.splice(index, 1);
    // }
    // updateCurrentEffects(newEffects);
  }

  public cacheGC() {
    PIXI.utils.clearTextureCache();
  }

  private loadAsset(url: string, callback: () => void) {
    /**
     * Loader 复用疑似有问题，转而采用先前的单独方式
     */
    this.loadQueue.push({ url, callback });
    /**
     * 尝试启动加载
     */
    this.callLoader();
  }

  private callLoader() {
    if (!this.assetLoader.loading) {
      const front = this.loadQueue.shift();
      if (front) {
        try {
          if (this.assetLoader.resources[front.url]) {
            front.callback();
            this.callLoader();
          } else {
            this.assetLoader.add(front.url).load(() => {
              front.callback();
              this.callLoader();
            });
          }
        } catch (error) {
          logger.fatal('PIXI Loader 故障', error);
          front.callback();
          // this.assetLoader.reset(); // 暂时先不用重置
          this.callLoader();
        }
      }
    }
  }

  private updateFps() {
    getScreenFps?.(120).then((fps) => {
      this.frameDuration = 1000 / (fps as number);
      // logger.info('当前帧率', fps);
    });
  }

  private lockStageObject(targetName: string) {
    this.lockTransformTarget.push(targetName);
  }

  private unlockStageObject(targetName: string) {
    const index = this.lockTransformTarget.findIndex((name) => name === targetName);
    if (index >= 0) this.lockTransformTarget.splice(index, 1);
  }
}

```

该函数的作用是更新当前存在于后端的数据库中的Effect条目。它接收一个名为`newEffects`的数组，这个数组包含了当前时间间隔内需要进行更新的Effect条目。

如果不存在需要更新的Effect条目，函数会等待50毫秒，然后从数据库中读取当前后端中与该函数的`key`相同的数据，并将新的Effect条目添加到该数据中。接着，函数会将数据库中与该函数的`key`相同的数据更新为新的Effect条目，并将其添加到后端的数据库中。

函数的实现还使用了JavaScript中的`cloneDeep`函数，这个函数可以用来对对象进行克隆，但是在这个函数中，它被用来在更新Effect条目时创建一个新的克隆对象，以便在更新时将后端的Effect条目与新的Effect条目进行比较。


```js
export function updateCurrentEffects(newEffects: IEffect[]) {
  // /**
  //  * 更新当前 backlog 条目的 effects 记录
  //  */
  // if (!notUpdateBacklogEffects)
  //   setTimeout(() => {
  //     const backlog = RUNTIME_CURRENT_BACKLOG[RUNTIME_CURRENT_BACKLOG.length - 1];
  //     if (backlog) {
  //       const newBacklogItem = cloneDeep(backlog);
  //       const backlog_effects = newBacklogItem.currentStageState.effects;
  //       while (backlog_effects.length > 0) {
  //         backlog_effects.pop();
  //       }
  //       backlog_effects.push(...newEffects);
  //       RUNTIME_CURRENT_BACKLOG.pop();
  //       RUNTIME_CURRENT_BACKLOG.push(newBacklogItem);
  //     }
  //   }, 50);

  webgalStore.dispatch(setStage({ key: 'effects', value: newEffects }));
}

```

这段代码定义了一个函数 `getScreenFps`，其作用是获取屏幕的 FPS(每秒帧数)，并返回一个 Promise 对象。

该函数内部包含一个内部函数 `nextFrame`，该函数用于在目标帧数帧数下获取动画帧，并返回一个 Promise 对象。如果目标帧数小于1，函数会抛出一个错误。

函数内部还有一个参数 `targetCount`，表示经过 `targetCount` 帧之后返回的结果。如果 `targetCount` 小于1，函数不会返回任何值。

函数内部还有一个 `count`，用于记录动画帧数，初始值为0。每次动画帧到达时，`count` 会自增，并判断是否达到了目标帧数。如果 `count` 已经等于目标帧数，函数会返回目标帧数每秒的 FPS。函数内部还包含一个 `log` 函数，用于在动画帧到达时通知结果。

由于 `nextFrame` 函数可能不支持某些浏览器，因此在 `getScreenFps` 函数内部会先使用 `window.requestAnimationFrame` 和 `window.webkitRequestAnimationFrame` 和 `window.mozRequestAnimationFrame` 函数来获取动画帧。如果这些函数不支持动画帧，函数内部会抛出一个错误并返回。


```js
/**
 * @param {number} targetCount 不小于1的整数，表示经过targetCount帧之后返回结果
 * @return {Promise<number>}
 */
const getScreenFps = (() => {
  // 先做一下兼容性处理
  const nextFrame = [
    window.requestAnimationFrame,
    // @ts-ignore
    window.webkitRequestAnimationFrame,
    // @ts-ignore
    window.mozRequestAnimationFrame,
  ].find((fn) => fn);
  if (!nextFrame) {
    console.error('requestAnimationFrame is not supported!');
    return;
  }
  return (targetCount = 60) => {
    // 判断参数是否合规
    if (targetCount < 1) throw new Error('targetCount cannot be less than 1.');
    const beginDate = Date.now();
    let count = 0;
    return new Promise((resolve) => {
      (function log() {
        nextFrame(() => {
          if (++count >= targetCount) {
            const diffDate = Date.now() - beginDate;
            const fps = (count / diffDate) * 1000;
            return resolve(fps);
          }
          log();
        });
      })();
    });
  };
})();

```

# `/packages/webgal/src/Core/controller/stage/pixi/WebGALPixiContainer.ts`

This is a TypeScript class that represents a PIXI filter called "rgbFilm".

The "rgbFilm" filter can be either a simple Gaussian blur filter, or a more advanced godray filter that applies a multi-pass image process to the传入 image.

The "getOrCreateGodrayFilter" method is used to create a godray filter if one does not already exist, and the "setGodrayFilm" method is used to set the value of the "rgbFilm" property to a color with the specified RGB values.

The "getGodrayFilm" property returns the index of the color with the specified RGB values, if it is set, otherwise it returns 0.

The "setColorPrimary" property sets the color that will be used as the primary color for the animation, this can be a hex color or a color from an image.

The "setAnimation" property sets the animation that will be applied to the sprite, this can be a number that represents the animation frame number or a string that represents the animation.


```js
import { OldFilmFilter } from '@pixi/filter-old-film';
import { DotFilter } from '@pixi/filter-dot';
import { ReflectionFilter } from '@pixi/filter-reflection';
import { GlitchFilter } from '@pixi/filter-glitch';
import { RGBSplitFilter } from '@pixi/filter-rgb-split';
import { GodrayFilter } from '@pixi/filter-godray';
import * as PIXI from 'pixi.js';

export class WebGALPixiContainer extends PIXI.Container {
  private baseX = 0;
  private baseY = 0;

  private containerFilters = new Map<string, PIXI.Filter>();

  public constructor() {
    super();
  }

  public get blur(): number {
    // @ts-ignore
    return this.getOrCreateBlurFilter().blur as number;
  }

  public set blur(value: number) {
    // @ts-ignore
    this.getOrCreateBlurFilter().blur = value;
  }

  public get x() {
    if (!super.position) {
      return 0;
    }
    return super.position.x - this.baseX;
  }

  public set x(value) {
    if (!super.position) {
      return;
    }
    super.position.x = value + this.baseX;
  }

  public get y() {
    if (!super.position) {
      return 0;
    }
    return super.position.y - this.baseY;
  }

  public set y(value) {
    if (!super.position) {
      return;
    }
    super.position.y = value + this.baseY;
  }

  public setBaseX(x: number) {
    const originalX = this.x;
    this.baseX = x;
    this.x = originalX;
  }

  public setBaseY(y: number) {
    const originalY = this.y;
    this.baseY = y;
    this.y = originalY;
  }

  private getOrCreateBlurFilter() {
    const blurFilterFromMap = this.containerFilters.get('blur');
    if (blurFilterFromMap) {
      return blurFilterFromMap;
    } else {
      const blurFilter = new PIXI.filters.BlurFilter();
      // 默认的 blur 是8，覆盖掉
      blurFilter.blur = 0;
      this.addFilter(blurFilter);
      this.containerFilters.set('blur', blurFilter);
      return blurFilter;
    }
  }

  /**
   * old film filter
   * @private
   */
  private getOrCreateOldFilmFilter(createMode = true) {
    const blurFilterFromMap = this.containerFilters.get('oldFilm');
    if (blurFilterFromMap) {
      return blurFilterFromMap;
    } else {
      if (createMode) {
        const oldFilm = new OldFilmFilter();
        this.addFilter(oldFilm);
        this.containerFilters.set('oldFilm', oldFilm);
        return oldFilm;
      } else return null;
    }
  }
  public get oldFilm(): number {
    if (this.getOrCreateOldFilmFilter(false)) return 1;
    return 0;
  }

  public set oldFilm(value: number) {
    /**
     * 如果是0，就移除这个滤镜
     */
    if (value === 0) {
      this.removeFilter('oldFilm');
    } else this.getOrCreateOldFilmFilter();
  }

  /**
   * dot film filter
   * @private
   */
  private getOrCreateDotFilter(createMode = true) {
    const blurFilterFromMap = this.containerFilters.get('dotFilm');
    if (blurFilterFromMap) {
      return blurFilterFromMap;
    } else {
      if (createMode) {
        const dotFilm = new DotFilter();
        this.addFilter(dotFilm);
        this.containerFilters.set('dotFilm', dotFilm);
        return dotFilm;
      } else return null;
    }
  }
  public get dotFilm(): number {
    if (this.getOrCreateDotFilter(false)) return 1;
    return 0;
  }

  public set dotFilm(value: number) {
    /**
     * 如果是0，就移除这个滤镜
     */
    if (value === 0) {
      this.removeFilter('dotFilm');
    } else this.getOrCreateDotFilter();
  }

  /**
   * reflection film filter
   * @private
   */
  private getOrCreateReflectionFilter(createMode = true) {
    const blurFilterFromMap = this.containerFilters.get('reflectionFilm');
    if (blurFilterFromMap) {
      return blurFilterFromMap;
    } else {
      if (createMode) {
        const reflectionFilm = new ReflectionFilter();
        this.addFilter(reflectionFilm);
        this.containerFilters.set('reflectionFilm', reflectionFilm);
        return reflectionFilm;
      } else return null;
    }
  }
  public get reflectionFilm(): number {
    if (this.getOrCreateReflectionFilter(false)) return 1;
    return 0;
  }

  public set reflectionFilm(value: number) {
    /**
     * 如果是0，就移除这个滤镜
     */
    if (value === 0) {
      this.removeFilter('reflectionFilm');
    } else this.getOrCreateReflectionFilter();
  }

  /**
   * glitchFilter film filter
   * @private
   */
  private getOrCreateGlitchFilter(createMode = true) {
    const blurFilterFromMap = this.containerFilters.get('glitchFilm');
    if (blurFilterFromMap) {
      return blurFilterFromMap;
    } else {
      if (createMode) {
        const glitchFilm = new GlitchFilter();
        this.addFilter(glitchFilm);
        this.containerFilters.set('glitchFilm', glitchFilm);
        return glitchFilm;
      } else return null;
    }
  }
  public get glitchFilm(): number {
    if (this.getOrCreateGlitchFilter(false)) return 1;
    return 0;
  }

  public set glitchFilm(value: number) {
    /**
     * 如果是0，就移除这个滤镜
     */
    if (value === 0) {
      this.removeFilter('glitchFilm');
    } else this.getOrCreateGlitchFilter();
  }

  /**
   * rgbSplitFilter film filter
   * @private
   */
  private getOrCreateRGBSplitFilter(createMode = true) {
    const blurFilterFromMap = this.containerFilters.get('rgbFilm');
    if (blurFilterFromMap) {
      return blurFilterFromMap;
    } else {
      if (createMode) {
        const rgbFilm = new RGBSplitFilter();
        this.addFilter(rgbFilm);
        this.containerFilters.set('rgbFilm', rgbFilm);
        return rgbFilm;
      }
    }
  }
  public get rgbFilm(): number {
    if (this.getOrCreateRGBSplitFilter(false)) return 1;
    return 0;
  }

  public set rgbFilm(value: number) {
    /**
     * 如果是0，就移除这个滤镜
     */
    if (value === 0) {
      this.removeFilter('rgbFilm');
    } else this.getOrCreateRGBSplitFilter();
  }

  /**
   * godrayFilter film filter
   * @private
   */
  private getOrCreateGodrayFilter(createMode = true) {
    const blurFilterFromMap = this.containerFilters.get('godrayFilm');
    if (blurFilterFromMap) {
      return blurFilterFromMap;
    } else {
      if (createMode) {
        const godrayFilm = new GodrayFilter();
        this.addFilter(godrayFilm);
        this.containerFilters.set('godrayFilm', godrayFilm);
        return godrayFilm;
      }
    }
  }
  public get godrayFilm(): number {
    if (this.getOrCreateGodrayFilter(false)) return 1;
    return 0;
  }

  public set godrayFilm(value: number) {
    /**
     * 如果是0，就移除这个滤镜
     */
    if (value === 0) {
      this.removeFilter('godrayFilm');
    } else this.getOrCreateGodrayFilter();
  }

  private addFilter(filter: PIXI.Filter) {
    if (this.filters) {
      this.filters.push(filter);
    } else {
      this.filters = [filter];
    }
  }

  private removeFilter(name: string) {
    const filter = this.containerFilters.get(name);
    if (filter) {
      const index = (this?.filters ?? []).findIndex((e) => e === filter);
      if (this.filters) {
        this.filters.splice(index, 1);
        this.containerFilters.delete(name);
      }
    }
  }
}

```