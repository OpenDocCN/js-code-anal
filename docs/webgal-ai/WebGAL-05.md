# WebGal源码解析 5

# `/packages/webgal/src/Core/gameScripts/changeSceneScript.ts`

这段代码定义了一个名为 `changeSceneScript` 的函数，它接受一个 ISentence 对象作为参数。这个函数的作用是在场景结束后将场景状态持久化并将其存储在一个后台服务中，以便在场景加载时从后台服务中恢复场景状态。

函数内部首先通过 `split` 方法将输入的 ISentence 对象的内容按照 / 路径分割成一个场景名数组，然后使用 `console.log` 打印出场景名数组最后一个元素，即场景名称。接着使用一个名为 `changeScene` 的函数从后台服务中获取指定的场景，并将其存储到传入的 `sentence.content` 变量中。

函数的返回值定义了一个 IPerform 对象，它包含了一些与场景相关的信息。例如，该对象包含 `performName`、`duration`、`isHoldOn` 和 `stopFunction` 等与表演相关的信息，以及一些与场景加载相关的信息，如 `stopTimeout` 等。这些信息在后面可能会被用于控制表演的执行时间、停止操作等。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { changeScene } from '../controller/scene/changeScene';

/**
 * 切换场景。在场景结束后不会回到父场景。
 * @param sentence
 */
export const changeSceneScript = (sentence: ISentence): IPerform => {
  const sceneNameArray: Array<string> = sentence.content.split('/');
  const sceneName = sceneNameArray[sceneNameArray.length - 1];
  changeScene(sentence.content, sceneName);
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: true,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/comment.ts`

以上代码的作用是定义了一个名为 "comment" 的函数，用于在场景中添加一个注释。

具体来说，它使用了一个具体的导出语句 `import { ISentence } from '@/Core/controller/scene/sceneInterface';` 从一个名为 "@/Core/controller/scene/sceneInterface" 的模块中导入了 "ISentence" 和 "IPerform" 类型。

接着，它使用了一个名为 "logger" 的模块中的 "debug" 函数来输出注释信息，这里的作用是在函数内部做一些调试工作，记录下脚本内注释的内容。

最后，它定义了一个返回值，它是一个实现了 `IPerform` 接口的对象，包含了许多与该接口相关的属性和方法。具体来说，它的 `performName` 属性设置为 "none"，表示这个注释脚本不会在场景中显示任何名字；它的 `duration` 属性设置为 0，表示这个注释脚本不会对场景的渲染速度产生任何影响；它的 `isHoldOn` 属性设置为 false，表示这个注释脚本不会被挂起；它的 `stopFunction` 属性设置为一个空函数，表示当该注释脚本挂起时，调用这个函数并不会有任何效果；它的 `blockingNext` 属性设置为 false，表示这个注释脚本不会被阻塞；它的 `blockingAuto` 属性设置为 true，表示这个注释脚本会被自动挂起。最后，它的 `stopTimeout` 属性设置为一个 undefined 值，表示暂时不用这个属性。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { logger } from '@/Core/util/etc/logger';

/**
 * 注释，打LOG
 * @param sentence
 */
export const comment = (sentence: ISentence): IPerform => {
  logger.debug(`脚本内注释${sentence.content}`);
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/end.ts`

这段代码定义了一个函数，接收一个ISentence类型的参数sentence，并执行以下操作：

1. 导入所需的模块：ISentence、IPerform、assetSetter、sceneFetcher、sceneParser、resetStage、webgalStore和setVisibility。

2. 加载游戏中的纹理：通过调用fileType获取文件类型，然后使用assetSetter的getTexture方法加载纹理，并将其存储在数组中。

3. 获取游戏中的场景：调用sceneFetcher的getScene方法， passing the ISentence instance and the texture array, and the scene parser will return a Scene对象。

4. 对场景进行解析：调用sceneParser的parseScene方法，对场景进行解析，并返回一个可用的scene对象。

5. 调用舞台重置函数：调用resetStage的()方法，进行舞台重置。

6. 调用WebGL纹理：通过调用WebGAL.getInstance()方法获取WebGL对象，然后调用playBgm()方法来播放背景音乐。

7. 将GUI可见性设置为true：使用setVisibility()方法将GUI可见性设置为true。

8. 将ISentence实例作为参数传递给playBgm()方法：调用playBgm()方法，并将ISentence实例作为参数传递。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { assetSetter, fileType } from '@/Core/util/gameAssetsAccess/assetSetter';
import { sceneFetcher } from '@/Core/controller/scene/sceneFetcher';
import { sceneParser } from '@/Core/parser/sceneParser';
import { resetStage } from '@/Core/controller/stage/resetStage';
import { webgalStore } from '@/store/store';
import { setVisibility } from '@/store/GUIReducer';
import { playBgm } from '@/Core/controller/stage/playBgm';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 结束游戏
 * @param sentence
 */
```

这段代码定义了一个名为 `end` 的函数，它接受一个名为 `sentence` 的参数，并返回一个名为 `Perform` 的接口。

首先，函数内使用了 `resetStage` 函数，它会清除当前场景，并返回 `true`，表示已经执行成功。

接着，函数内使用 `dispatch` 函数来向 WebGAL 存储库发送一个名为 `setVisibility` 的 action，该 action 设置了一个名为 `showTitle` 的组件的可见性为 `true`。

然后，函数内使用 `playBgm` 函数来播放音频背景，它需要一个 `webgalStore` 的状态对象作为参数。

最后，函数内返回了一个包含表演名称、持续时间、是否正在挂起以及停止函数等属性的对象，这些属性可以用于其他地方。


```js
export const end = (sentence: ISentence): IPerform => {
  resetStage(true);
  const dispatch = webgalStore.dispatch;
  // 重新获取初始场景
  const sceneUrl: string = assetSetter('start.txt', fileType.scene);
  // 场景写入到运行时
  sceneFetcher(sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, 'start.txt', sceneUrl);
  });
  dispatch(setVisibility({ component: 'showTitle', visibility: true }));
  playBgm(webgalStore.getState().GUI.titleBgm);
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/filmMode.ts`

这段代码定义了一个名为 `filmMode` 的函数，它接受一个参数 `sentence`。函数的作用是根据 `sentence` 的内容决定是哪种拍摄模式。

具体来说，如果 `sentence` 的内容既不是 'none' 也不是空，那么函数会将 `webgalStore` 中的一个名为 `enableFilm` 的状态设置为 `sentence` 的内容，这里 `sentence` 的内容将成为 String 类型。

否则，函数会将 `webgalStore` 中的另一个名为 `enableFilm` 的状态设置为空，这里 `sentence` 的内容将成为一个字符串。

最终，函数返回一个 an object，它包含以下内容：

- `performName`：表演名称，这里为 'none'。
- `duration`：持续时间，这里为 0。
- `isHoldOn`：是否保持抑制，这里为 false。
- `stopFunction`：停止函数，这里为空函数。
- `blockingNext`：下一个块是否自动执行，这里为 false。
- `blockingAuto`：是否自动停止，这里为 true。
- `stopTimeout`：停止时间，这里为 undefined。

注意， `stopTimeout` 目前还没有被使用，当需要的时候，可以给它赋值，让函数在一定的时间后自动停止。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';

/**
 * 语句执行的模板代码
 * @param sentence
 */
export const filmMode = (sentence: ISentence): IPerform => {
  if (sentence.content !== '' && sentence.content !== 'none') {
    webgalStore.dispatch(setStage({ key: 'enableFilm', value: sentence.content }));
  } else {
    webgalStore.dispatch(setStage({ key: 'enableFilm', value: '' }));
  }
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/jumpLabel.ts`

这段代码定义了一个名为 "jumpLabel" 的函数，它接受一个 ISentence 对象作为参数。这个函数的作用是跳转到指定标签，然后返回一个 IPerform 对象，用于在游戏中执行相应的表演动作。

具体来说，这段代码执行以下步骤：

1. 导入两个依赖的模块：来自 "@/Core/controller/scene/sceneInterface" 的 ISentence 和来自 "@/Core/Modules/perform/performInterface" 的 IPerform。
2. 引入一个叫做 "jmp" 的游戏脚本模块。
3. 定义一个名为 "jumpLabel" 的函数，它接受一个 ISentence 对象作为参数。
4. 在函数内部，使用 IPerform 的 "jmp" 方法来执行从当前场景开始跳转到指定标签的操作，并返回一个 IPerform 对象。
5. 这个函数的 "performName" 属性设置为 "none"，表示在表演中不会输出任何信息。
6. "duration" 属性设置为 0，表示这个表演动作不会被暂停或延迟。
7. "isHoldOn" 属性设置为 false，表示这个表演动作不会被锁定在时间轴上。
8. "stopFunction" 属性设置为一个空函数，表示当这个表演动作停止时，会调用这个函数来处理取消操作。
9. "blockingNext" 属性设置为 false，表示这个表演动作不会被阻塞等待后续操作。
10. "blockingAuto" 属性设置为 true，表示这个表演动作会一直保持激活状态，直到被手动停止。
11. "stopTimeout" 属性设置为 undefined，表示暂时不需要设置停止时间。

总之，这段代码定义了一个可以跳转到指定标签的表演动作，并可以在游戏中使用。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { jmp } from '@/Core/gameScripts/function/jmp';

/**
 * 跳转到指定标签
 * @param sentence
 */
export const jumpLabel = (sentence: ISentence): IPerform => {
  jmp(sentence.content);
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/label.ts`

这段代码定义了一个名为 `label` 的函数，接受一个 ISentence 类型的参数 `sentence`。该函数返回一个 IPerform 类型的结果，用于对表演进行控制。

具体来说，这个函数不做任何实际操作，它返回的 IPerform 对象有以下几个属性：

* performName: 'none'，表示这个表演没有名字。
* duration: 0，表示这个表演没有持续时间限制。
* isHoldOn: false，表示这个表演没有被抓住（即，表演者可以自由停止或继续）。
* stopFunction: () => {}, 表示这个表演的停止函数是一个空函数，不会执行任何逻辑。
* blockingNext: () => false, 表示这个表演不会阻止下一个表演的执行。
* blockingAuto: () => true, 表示这个表演会阻止当前表演的执行，直到下一个表演开始。
* stopTimeout: undefined, 表示这个表演的停止定时器还没有被设置。

总之，这个函数只是定义了一个表演，它没有做任何实际的事情，只是返回了一个 IPerform 类型的对象，用于控制表演的执行。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';

/**
 * 标签代码，什么也不做
 * @param sentence
 */
export const label = (sentence: ISentence): IPerform => {
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/miniAvatar.ts`

这段代码定义了一个名为 `miniAvatar` 的特效组件，用于在场景中显示小头像。

在组件导入中，使用了来自不同模块的 `ISentence`、`IPerform` 和 `webgalStore` 类。

在组件内部，首先检查给出的句子内容，如果为空或省略，则设置为空字符串。然后将内容存储到 `webgalStore` 中，使用 `setStage` 方法将舞台状态中设置为 `miniAvatar` 的 key 的值设置为内容。

通过 `IPerform` 接口，定义了该组件的表演名称、持续时间、挂接位置、停止函数等属性。还定义了 `stopTimeout` 属性，用于在一定时间内自动停止表演。

最后，将定义的组件内容导出，使其可以在场景中使用。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';

/**
 * 显示小头像
 * @param sentence
 */
export const miniAvatar = (sentence: ISentence): IPerform => {
  let content = sentence.content;
  if (sentence.content === 'none' || sentence.content === '') {
    content = '';
  }
  webgalStore.dispatch(setStage({ key: 'miniAvatar', value: content }));
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: true,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/pixi.ts`

这段代码定义了一个名为 `pixi` 的函数，它接受一个 `ISentence` 类型的参数 `sentence`。

`pixi` 函数的作用是将传入的 `sentence` 对象转换成能够在 Pixi 舞台上表演的一段演出，并返回一个 `IPerform` 类型的结果。

在这个函数中，我们通过 `WebGAL` 库的 `gameplay.performController` 类来控制 Pixi 舞台上的一段演出。首先，我们获取演出名称，将其作为 `performName`，然后使用 `duration` 属性设置演出时长，使用 `isOver` 属性设置演出是否结束，使用 `isHoldOn` 属性设置演出是否正在被保持，使用 `stopFunction` 属性设置当演出正常结束时执行的函数，使用 `blockingNext` 属性设置阻止下一个演出发生，使用 `blockingAuto` 属性设置阻止自动阻止下一个演出发生，使用 `stopTimeout` 属性设置自动清除演出计时器的时间。

接下来，我们通过 `call` 方法调用 `sentence.content`，即演出内容，并获取其结果。如果结果符合我们的预期，我们继续执行演出，否则我们执行演出并记录结果。

最后，我们使用 `logger.warn` 函数输出一条警告信息，告知演出已卸载。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { logger } from '@/Core/util/etc/logger';
import { IResult, call } from '../util/pixiPerformManager/pixiPerformManager';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 运行一段pixi演出
 * @param sentence
 */
export const pixi = (sentence: ISentence): IPerform => {
  const pixiPerformName = 'PixiPerform' + sentence.content;
  WebGAL.gameplay.performController.performList.forEach((e) => {
    if (e.performName === pixiPerformName) {
      return {
        performName: 'none',
        duration: 0,
        isOver: false,
        isHoldOn: true,
        stopFunction: () => {},
        blockingNext: () => false,
        blockingAuto: () => false,
        stopTimeout: undefined, // 暂时不用，后面会交给自动清除
      };
    }
  });
  const res: IResult = call(sentence.content);
  const { container, tickerKey } = res;

  return {
    performName: pixiPerformName,
    duration: 0,
    isHoldOn: true,
    stopFunction: () => {
      logger.warn('现在正在卸载pixi演出');
      container.destroy({ texture: true, baseTexture: true });
      WebGAL.gameplay.pixiStage?.effectsContainer.removeChild(container);
      WebGAL.gameplay.pixiStage?.removeAnimation(tickerKey);
    },
    blockingNext: () => false,
    blockingAuto: () => false,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/pixiInit.ts`

`pixiInit`函数用于初始化Pixi游戏引擎，根据传入的`sentence`参数，在游戏执行时执行相应的Pixi操作。

具体来说，它主要完成以下几件事情：

1. 卸载演出：如果当前执行的演出与要初始化的演出相同，就卸载该演出并清除相关的时间定时器。

2. 从状态表中清除演出：从`webgalStore`中读取当前的状态，并清空`PerformList`数组，即清除所有演出，同时保留已经创建但还没有执行的演出。

3. 从`PerformList`中移除已有的演出：遍历`PerformList`数组，如果演出已经存在于其中，则将其从数组中移除。

4. 初始化演出列表：根据要初始化的演出创建或更新`PerformList`数组。

最后，它将`performName`设置为`none`,`duration`设置为`0`,`isHoldOn`设置为`false`,`stopFunction`设置为`()`,`blockingNext`设置为`false`,`blockingAuto`设置为`true`,`stopTimeout`设置为`undefined`，这些设置将在演出执行时决定Pixi的渲染行为。


```js
import { commandType, ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { logger } from '@/Core/util/etc/logger';
import { webgalStore } from '@/store/store';
import { resetStageState } from '@/store/stageReducer';
import cloneDeep from 'lodash/cloneDeep';

import { WebGAL } from '@/Core/WebGAL';
import { IRunPerform } from '@/store/stageInterface';

/**
 * 初始化pixi
 * @param sentence
 */
export const pixiInit = (sentence: ISentence): IPerform => {
  WebGAL.gameplay.performController.performList.forEach((e) => {
    if (e.performName.match(/PixiPerform/)) {
      logger.warn('pixi 被脚本重新初始化', e.performName);
      /**
       * 卸载演出
       */
      for (let i = 0; i < WebGAL.gameplay.performController.performList.length; i++) {
        const e2 = WebGAL.gameplay.performController.performList[i];
        if (e2.performName === e.performName) {
          e2.stopFunction();
          clearTimeout(e2.stopTimeout as unknown as number);
          WebGAL.gameplay.performController.performList.splice(i, 1);
          i--;
        }
      }
      /**
       * 从状态表里清除演出
       */
      const stageState = webgalStore.getState().stage;
      const newStageState = cloneDeep(stageState);
      for (let i = 0; i < newStageState.PerformList.length; i++) {
        const e2: IRunPerform = newStageState.PerformList[i];
        if (e2.script.command === commandType.pixi) {
          newStageState.PerformList.splice(i, 1);
          i--;
        }
      }
      webgalStore.dispatch(resetStageState(newStageState));
    }
  });
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/playEffect.ts`

This is a JavaScript script that defines an HTML5 audio element and performs an audio analysis for the given audio URL. It uses the WebGAL library to perform the analysis and utilizes the `WebGAL` object to store the data.

The script first gets the volume argument from the `optionData` property of the `userData` object in the `WebGAL.gameplay.performController`. If the volume argument is a number between 0 and 100, it multiplies it by the `seVolume` property of the `userData` object and multiplies it by 0.01 to scale it down to the main volume.

The `currentTime` property of the `audio` element is then set to 0 to start the audio playing from the beginning. The `perform` object is then initialized with the `performInitName` property, the duration of the analysis (in milliseconds), and the looping state (a boolean value).

The `blockingNext` and `blockingAuto` properties are set to `false` to allow the audio to start playing without being held back, and the `stopFunction` property is set to an empty function that does nothing when called.

The `stopTimeout` property is set to `undefined`, meaning it will be handled by the `WebGAL.gameplay.performController` if it is called.

The `perFORM.then` method is called with the `perform` object defined earlier. This method resolves the `perform` object, which is then passed to the `performInit` method of the `WebGAL.gameplay.performController`.

The `perFORM.then` method is also called with the `perFORM.then` method defined earlier. This method is passed to the `window.WebGAL.require.validatePerform@4` method, which validates the `perform` object and returns a promise.

The `window.WebGAL.require.validatePerform@4` method is then called with the `perform` object defined earlier, passing in the `validatePerform` callback, which is the `perFORM.then` method defined earlier.

The returned promise is then resolved, and the ` validatePerform` callback is passed to the `WebGAL.gameplay.performController` object to start


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { logger } from '@/Core/util/etc/logger';
import { RootState, webgalStore } from '@/store/store';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { useSelector } from 'react-redux';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 播放一段效果音
 * @param sentence 语句
 */
export const playEffect = (sentence: ISentence): IPerform => {
  logger.debug('play SE');
  // 如果有ID，这里被覆写，一般用于循环的情况
  // 有循环参数且有 ID，就循环
  let performInitName = 'effect-sound';
  // 清除先前的效果音
  WebGAL.gameplay.performController.unmountPerform(performInitName, true);
  let url = sentence.content;
  let isLoop = false;
  // 清除带 id 的效果音
  if (getSentenceArgByKey(sentence, 'id')) {
    const id = getSentenceArgByKey(sentence, 'id');
    performInitName = `effect-sound-${id}`;
    WebGAL.gameplay.performController.unmountPerform(performInitName, true);
    isLoop = true;
  }
  let isOver = false;
  return {
    performName: 'none',
    blockingAuto(): boolean {
      return false;
    },
    blockingNext(): boolean {
      return false;
    },
    isHoldOn: false,
    stopFunction(): void {},
    stopTimeout: undefined,

    duration: 1000 * 60 * 60,
    arrangePerformPromise: new Promise((resolve) => {
      // 播放效果音
      setTimeout(() => {
        const volumeArg = getSentenceArgByKey(sentence, 'volume');
        let seElement = document.createElement('audio');
        seElement.src = url;
        if (isLoop) {
          seElement.loop = true;
        }
        const userDataState = webgalStore.getState().userData;
        const mainVol = userDataState.optionData.volumeMain;
        // Work when volumeArg is a number between 0 and 100
        const volume = typeof volumeArg === 'number' && volumeArg >= 0 && volumeArg <= 100 ? volumeArg : 100;
        const seVol = mainVol * 0.01 * (userDataState.optionData?.seVolume ?? 100) * 0.01 * volume * 0.01;
        seElement.volume = seVol;
        seElement.currentTime = 0;
        const perform = {
          performName: performInitName,
          duration: 1000 * 60 * 60,
          isHoldOn: isLoop,
          skipNextCollect: true,
          stopFunction: () => {
            // 演出已经结束了，所以不用播放效果音了
            seElement.oncanplay = () => {};
            seElement.pause();
          },
          blockingNext: () => false,
          blockingAuto: () => {
            return !isOver;
          },
          stopTimeout: undefined, // 暂时不用，后面会交给自动清除
        };
        resolve(perform);
        seElement.oncanplay = () => {
          seElement?.play();
        };
        seElement.onended = () => {
          for (const e of WebGAL.gameplay.performController.performList) {
            if (e.performName === performInitName) {
              isOver = true;
              e.stopFunction();
              WebGAL.gameplay.performController.unmountPerform(e.performName);
            }
          }
        };
      }, 1);
    }),
  };
};

```

# `/packages/webgal/src/Core/gameScripts/playVocal.ts`

该代码集是一个名为 `sceneController` 的控制器类，从 `@/Core/controller/scene/sceneInterface` 包中导入了一个名为 `ISentence` 的类。这个类的作用是用于与游戏中的动作相关操作的服务。

此外，代码还从 `@/Core/util/etc/logger` 包中导入了一个名为 `logger` 的函数，以及从 `@/store/stageReducer` 和 `@/store/stageInterface` 包中导入了一些与舞台状态相关的类和函数。

另外，代码还从 `@/Core/gameScripts/function/vocalAnimation` 包中导入了一个名为 `audioContextWrapper` 的函数，从 `@/Core/util/match` 包中导入了一个名为 `match` 的函数，以及从 `@/Core/WebGAL` 包中导入了一个名为 `WebGAL` 的类。

最后，代码还从 `@/store/base` 包中导入了一个名为 `setStage` 的函数，从 `@/store/stageReducer` 中导入了一个名为 `getSentenceArgByKey` 的函数，从 `@/Core/gameScripts/function/vocalAnimation` 中导入了一个名为 `performBlinkAnimation` 的函数，从 `@/Core/gameScripts/function/vocalAnimation` 中导入了一个名为 `performMouthAnimation` 的函数，以及从 `@/store/stageInterface` 中导入了一个名为 `updateThresholds` 的函数。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { logger } from '@/Core/util/etc/logger';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { IStageState } from '@/store/stageInterface';
import {
  audioContextWrapper,
  getAudioLevel,
  performBlinkAnimation,
  performMouthAnimation,
  updateThresholds,
} from '@/Core/gameScripts/function/vocalAnimation';
import { match } from '../util/match';
import { WebGAL } from '@/Core/WebGAL';

```

This is a JavaScript code that creates a vocal control in a vocal sample. It uses the WebGAL.gameplay.performController to control the vocal performance.

The code creates a MediaElement for the vocal control and connects it to the audioContextWrapper, which is a wrapper for the audioContext. It then sets a timeout for the vocal control to open and a timeout for it to close.

When the vocal control opens, it creates a performance for the vocal sample and connects it to the audioContextWrapper. The code also sets a timeout for the vocal sample to perform and updates the threshold values for the vocal sample based on the current position of the sample.

When the vocal sample finishes, the code blinks the eyes and updates the animation item.

Overall, the code tries to create a simple and interactive vocal control for the vocal sample.


```js
/**
 * 播放一段语音
 * @param sentence 语句
 */
export const playVocal = (sentence: ISentence) => {
  logger.debug('play vocal');
  const performInitName = 'vocal-play';
  const url = getSentenceArgByKey(sentence, 'vocal'); // 获取语音的url
  const volume = getSentenceArgByKey(sentence, 'volume'); // 获取语音的音量比
  let currentStageState: IStageState;
  currentStageState = webgalStore.getState().stage;
  let pos = '';
  let key = '';
  const freeFigure = currentStageState.freeFigure;
  const figureAssociatedAnimation = currentStageState.figureAssociatedAnimation;
  let bufferLength = 0;
  let currentMouthValue = 0;
  const lerpSpeed = 1;

  // 先停止之前的语音
  let VocalControl: any = document.getElementById('currentVocal');
  WebGAL.gameplay.performController.unmountPerform('vocal-play', true);
  if (VocalControl !== null) {
    VocalControl.currentTime = 0;
    VocalControl.pause();
  }

  for (const e of sentence.args) {
    if (e.value === true) {
      match(e.key)
        ._('left', () => {
          pos = 'left';
        })
        ._('right', () => {
          pos = 'right';
        })
        .__('center', () => {
          pos = 'center';
        });
    }
    if (e.key === 'figureId') {
      key = `${e.value.toString()}`;
    }
  }

  // 获得舞台状态
  webgalStore.dispatch(setStage({ key: 'vocal', value: url }));

  let isOver = false;

  /**
   * 嘴型同步
   */

  return {
    arrangePerformPromise: new Promise((resolve) => {
      // 播放语音
      setTimeout(() => {
        let VocalControl: any = document.getElementById('currentVocal');
        // 设置语音音量
        typeof volume === 'number' && volume >= 0 && volume <= 100
          ? webgalStore.dispatch(setStage({ key: 'vocalVolume', value: volume }))
          : webgalStore.dispatch(setStage({ key: 'vocalVolume', value: 100 }));
        // 设置语音
        if (VocalControl !== null) {
          VocalControl.currentTime = 0;
          // 播放并作为一个特别演出加入
          const perform = {
            performName: performInitName,
            duration: 1000 * 60 * 60,
            isOver: false,
            isHoldOn: false,
            stopFunction: () => {
              // 演出已经结束了，所以不用播放语音了
              VocalControl.oncanplay = () => {};
              clearInterval(audioContextWrapper.audioLevelInterval);
              VocalControl.pause();
              key = key ? key : `fig-${pos}`;
              const animationItem = figureAssociatedAnimation.find((tid) => tid.targetId === key);
              performMouthAnimation({
                audioLevel: 0,
                OPEN_THRESHOLD: 1,
                HALF_OPEN_THRESHOLD: 1,
                currentMouthValue,
                lerpSpeed,
                key,
                animationItem,
                pos,
              });
              clearTimeout(audioContextWrapper.blinkTimerID);
            },
            blockingNext: () => false,
            blockingAuto: () => {
              return !isOver;
            },
            skipNextCollect: true,
            stopTimeout: undefined, // 暂时不用，后面会交给自动清除
          };
          WebGAL.gameplay.performController.arrangeNewPerform(perform, sentence, false);
          VocalControl.oncanplay = () => {
            key = key ? key : `fig-${pos}`;
            const animationItem = figureAssociatedAnimation.find((tid) => tid.targetId === key);
            if (animationItem) {
              let maxAudioLevel = 0;

              const foundFigure = freeFigure.find((figure) => figure.key === key);

              if (foundFigure) {
                pos = foundFigure.basePosition;
              }

              if (!audioContextWrapper.audioContext) {
                let audioContext: AudioContext | null;
                audioContext = new AudioContext();
                audioContextWrapper.analyser = audioContext.createAnalyser();
                audioContextWrapper.analyser.fftSize = 256;
                audioContextWrapper.dataArray = new Uint8Array(audioContextWrapper.analyser.frequencyBinCount);
              }

              if (!audioContextWrapper.analyser) {
                audioContextWrapper.analyser = audioContextWrapper.audioContext.createAnalyser();
                audioContextWrapper.analyser.fftSize = 256;
              }

              bufferLength = audioContextWrapper.analyser.frequencyBinCount;
              audioContextWrapper.dataArray = new Uint8Array(bufferLength);
              let vocalControl = document.getElementById('currentVocal') as HTMLMediaElement;

              if (!audioContextWrapper.source) {
                audioContextWrapper.source = audioContextWrapper.audioContext.createMediaElementSource(vocalControl);
                audioContextWrapper.source.connect(audioContextWrapper.analyser);
              }

              audioContextWrapper.analyser.connect(audioContextWrapper.audioContext.destination);

              // Lip-snc Animation
              audioContextWrapper.audioLevelInterval = setInterval(() => {
                const audioLevel = getAudioLevel(
                  audioContextWrapper.analyser!,
                  audioContextWrapper.dataArray!,
                  bufferLength,
                );
                const { OPEN_THRESHOLD, HALF_OPEN_THRESHOLD } = updateThresholds(audioLevel);

                performMouthAnimation({
                  audioLevel,
                  OPEN_THRESHOLD,
                  HALF_OPEN_THRESHOLD,
                  currentMouthValue,
                  lerpSpeed,
                  key,
                  animationItem,
                  pos,
                });
              }, 50);

              // blinkAnimation
              let animationEndTime: number;

              // 10sec
              animationEndTime = Date.now() + 10000;
              performBlinkAnimation({ key, animationItem, pos, animationEndTime });

              // 10sec
              setTimeout(() => {
                clearTimeout(audioContextWrapper.blinkTimerID);
              }, 10000);
            }

            VocalControl?.play();
          };
          VocalControl.onended = () => {
            for (const e of WebGAL.gameplay.performController.performList) {
              if (e.performName === performInitName) {
                isOver = true;
                e.stopFunction();
                WebGAL.gameplay.performController.unmountPerform(e.performName);
              }
            }
          };
        }
      }, 1);
    }),
  };
};

```

# `/packages/webgal/src/Core/gameScripts/say.ts`

这段代码的主要作用是创建一个普通对话的场景，并执行相应的演出。具体来说，它会执行以下操作：

1. 导入所需的模块和样式：从 '@/Core/controller/scene/sceneInterface' 导入 ISentence 模块，从 '@/Core/Modules/perform/performInterface' 导入 IPerform 模块，从 styles1 和 styles2 导入相关的样式；
2. 导入自定义样式：从 '../../Components/Stage/TextBox/textbox.module.scss' 和 '../../Components/Stage/TextBox/textboxFilm.module.scss' 导入自定义样式，分别对两个文本框组件的样式进行定义；
3. 导入 playVocal 函数：从 './playVocal' 导入 playVocal 函数，用于播放 Vocaloid 虚拟歌手的角色发音；
4. 导入 webgalStore 变量：从 '@/store/store' 导入 webgalStore 变量，用于在场景中使用 WebGAL 游戏引擎；
5. 导入 setStage 函数：从 '@/store/stageReducer' 导入 setStage 函数，用于在场景中创建或切换不同的舞台；
6. 导入 useTextDelay 函数：从 '@/hooks/useTextOptions' 导入 useTextDelay 函数，用于延迟显示文本内容；
7. 导入 getRandomPerformName 函数：从 '@/Core/Modules/perform/performController' 导入 getRandomPerformName 函数，用于生成随机的表演名称；
8. 导入 performController 变量：从 '@/Core/Modules/perform/performInterface' 导入 performController 变量，用于在场景中执行相应的演出；
9. 导入 getSentenceArgByKey 函数：从 '@/Core/util/getSentenceArg' 导入 getSentenceArgByKey 函数，用于从句子中提取出相应的 arguments；
10. 导入 textSize 函数：从 '@/store/userDataInterface' 导入 textSize 函数，用于设置文本的大小；
11. 导入 WebGAL 实例：从 '@/Core/Core.scss' 导入 WebGAL 实例，用于在场景中使用 WebGAL 游戏引擎；
12. 创建 textbox 变量：使用自定义样式创建一个 textbox 组件实例，并将其存储在场景中的 textbox 变量中；
13. 创建 perform 变量：使用 performController 函数创建一个演出实例，并将其存储在场景中的 perform 变量中；
14. 创建演出 变量：使用 setStage 函数创建一个演出实例，并将其存储在场景中的 currentStage 变量中；
15. 发送演出：调用 performController 函数发送演出，将演出实例作为参数传递给 setStage 函数，并使用 currentStage 变量作为场景舞台进行切换；
16. 播放虚拟歌手：调用 playVocal 函数播放虚拟歌手的角色发音，将需要发音的文本内容作为参数传递给 playVocal 函数；
17. 挂载演出：调用 performController 函数挂载演出，将演出实例作为参数传递给 setStage 函数，并使用 currentStage 变量作为场景舞台进行切换。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import styles1 from '../../Components/Stage/TextBox/textbox.module.scss';
import styles2 from '../../Components/Stage/TextBox/textboxFilm.module.scss';
import { playVocal } from './playVocal';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';
import { useTextDelay } from '@/hooks/useTextOptions';
import { getRandomPerformName, PerformController } from '@/Core/Modules/perform/performController';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { textSize } from '@/store/userDataInterface';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 进行普通对话的显示
 * @param sentence 语句
 * @return {IPerform} 执行的演出
 */
```

This is a React component called `PerformNotend`. It is used to perform a speech or text-to-speech operation with a pre-recorded or manually entered text.

It is composed of the following elements:

* `Dispatch`: This is a built-in React hook that allows you to dispatch actions to the component system.
* `useStage`: This hook is provided by the `react-os` library, and it allows you to access the current stage of a performance.
* `useDispatch`: This hook is also provided by the `react-os` library, and it allows you to dispatch actions using the `useDispatch` hook.
* `useEffect`: This hook is provided by React, and it allows you to perform side effects in your component.
* `useState`: This hook is provided by React, and it allows you to store a state value in your component.
* `textToSpeech`: This is a built-in JavaScript API that enables text-to-speech functionality.
* `performance`: This is a built-in JavaScript API that provides access to the performance of the user's device.
* `createTextDecoder`: This is a custom function that creates a text decoder from a given text-to-speech engine.
* `vocal`: This is a boolean that indicates whether to perform the text-to-speech operation using the 'vocal' or 'auto' voice.
* `setTextSize`: This is a decimal that allows you to control the font size of the text being read out.
* `styles`: This is an object that contains styles applied to elements in the component.
* `speaker`: This is a boolean that indicates whether to perform the text-to-speech operation using a pre-recorded or text-to-speech-enabled device.
* `clear`: This is a boolean that indicates whether to clear the text that has been read out before it is read out again.
* `optionData`: This is an object that contains any additional data that you want to pass to the `textToSpeech` function.
* `isNotend`: This is a boolean that indicates whether the last voice交代已经结束，如果没有，则不需要等待
* `endDelay`: This is an optional value that is used if `isNotend` is `true`. It is the number of milliseconds between the end of the last voice交代和 the start of the performance.

This component uses several other React hooks and libraries, including `react-os`, `react-hook-es`, `react-text-loop`, and `react-webfont-2`.


```js
export const say = (sentence: ISentence): IPerform => {
  const stageState = webgalStore.getState().stage;
  const styles = stageState.enableFilm === '' ? styles1 : styles2;
  const userDataState = webgalStore.getState().userData;
  const dispatch = webgalStore.dispatch;
  let dialogKey = Math.random().toString(); // 生成一个随机的key
  let dialogToShow = sentence.content; // 获取对话内容
  const isConcat = getSentenceArgByKey(sentence, 'concat'); // 是否是继承语句
  const isNotend = getSentenceArgByKey(sentence, 'notend') as boolean; // 是否有 notend 参数
  const speaker = getSentenceArgByKey(sentence, 'speaker'); // 获取说话者
  const clear = getSentenceArgByKey(sentence, 'clear'); // 是否清除说话者
  const vocal = getSentenceArgByKey(sentence, 'vocal'); // 是否播放语音

  // 如果是concat，那么就继承上一句的key，并且继承上一句对话。
  if (isConcat) {
    dialogKey = stageState.currentDialogKey;
    dialogToShow = stageState.showText + dialogToShow;
    dispatch(setStage({ key: 'currentConcatDialogPrev', value: stageState.showText }));
  } else {
    dispatch(setStage({ key: 'currentConcatDialogPrev', value: '' }));
  }

  // 设置文本显示
  dispatch(setStage({ key: 'showText', value: dialogToShow }));
  // 清除语音
  dispatch(setStage({ key: 'vocal', value: '' }));
  WebGAL.gameplay.performController.unmountPerform('vocal-play', true);
  // 设置key
  dispatch(setStage({ key: 'currentDialogKey', value: dialogKey }));
  // 计算延迟
  const textDelay = useTextDelay(userDataState.optionData.textSpeed);
  // 本句延迟
  const sentenceDelay = textDelay * sentence.content.length;

  for (const e of sentence.args) {
    if (e.key === 'fontSize') {
      switch (e.value) {
        case 'default':
          dispatch(setStage({ key: 'showTextSize', value: -1 }));
          break;
        case 'small':
          dispatch(setStage({ key: 'showTextSize', value: textSize.small }));
          break;
        case 'medium':
          dispatch(setStage({ key: 'showTextSize', value: textSize.medium }));
          break;
        case 'large':
          dispatch(setStage({ key: 'showTextSize', value: textSize.large }));
          break;
      }
    }
  }

  // 设置显示的角色名称
  let showName: string | number | boolean = stageState.showName; // 先默认继承
  if (speaker !== null) {
    showName = speaker;
  }
  if (clear) {
    showName = '';
  }
  dispatch(setStage({ key: 'showName', value: showName }));

  // 播放一段语音
  if (vocal) {
    playVocal(sentence);
  }

  const performInitName: string = getRandomPerformName();
  let endDelay = 750 - userDataState.optionData.textSpeed * 250;
  // 如果有 notend 参数，那么就不需要等待
  if (isNotend) {
    endDelay = 0;
  }

  return {
    performName: performInitName,
    duration: sentenceDelay + endDelay,
    isHoldOn: false,
    stopFunction: () => {
      const textElements = document.querySelectorAll('.' + styles.TextBox_textElement_start);
      const textArray = [...textElements];
      textArray.forEach((e) => {
        e.className = styles.TextBox_textElement_Settled;
      });
    },
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
    goNextWhenOver: isNotend,
  };
};

```

# `/packages/webgal/src/Core/gameScripts/setAnimation.ts`

This is a JavaScript function that allows you to set a background animation for a given sentence.

The function takes in a `sentence` argument, which is an instance of the `ISentence` interface from a module called `AnimationFunctions`. This interface has a `content` property that is a string representing the name of the animation that should be displayed.

The function returns an instance of the `IPerform` interface from a module called `Store`, which has a `setAnimation` method that takes a `sentence` argument and returns an instance of the `Perform` interface.

The `setAnimation` method takes several arguments:

* `sentence`: the sentence object that was passed in.
* `startDialogKey`: the key of the current dialog in the animation. This is useful for debug purposes.
* `animationName`: the name of the animation that should be displayed.
* `animationDuration`: the duration of the animation in seconds.
* `target`: the string that represents the target for the animation. This should be a string that is passed down by the `AnimationFunctions` module.

The method uses the `WebGAL` library to perform the animation and returns an instance of the `Perform` interface that has a `performName` property for the animation to be displayed.


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { IAnimationObject } from '@/Core/controller/stage/pixi/PixiController';
import { logger } from '@/Core/util/etc/logger';
import { webgalStore } from '@/store/store';

import { getAnimateDuration, getAnimationObject } from '@/Core/Modules/animationFunctions';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 设置背景动画
 * @param sentence
 */
export const setAnimation = (sentence: ISentence): IPerform => {
  const startDialogKey = webgalStore.getState().stage.currentDialogKey;
  const animationName = sentence.content;
  const animationDuration = getAnimateDuration(animationName);
  const target = (getSentenceArgByKey(sentence, 'target') ?? 0) as string;
  const key = `${target}-${animationName}-${animationDuration}`;
  let stopFunction;
  setTimeout(() => {
    WebGAL.gameplay.pixiStage?.stopPresetAnimationOnTarget(target);
    const animationObj: IAnimationObject | null = getAnimationObject(animationName, target, animationDuration);
    if (animationObj) {
      logger.debug(`动画${animationName}作用在${target}`, animationDuration);
      WebGAL.gameplay.pixiStage?.registerAnimation(animationObj, key, target);
    }
  }, 0);
  stopFunction = () => {
    setTimeout(() => {
      const endDialogKey = webgalStore.getState().stage.currentDialogKey;
      const isHasNext = startDialogKey !== endDialogKey;
      WebGAL.gameplay.pixiStage?.removeAnimationWithSetEffects(key);
    }, 0);
  };

  return {
    performName: key,
    duration: animationDuration,
    isHoldOn: false,
    stopFunction,
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/setComplexAnimation.ts`

This is a JavaScript method that performs a complex animation. It takes in a sentence as an argument and returns an IPerform object.

The purpose of this method is to enable setting the background animation of a game scene to the effect of choosing a sentence in the game, which will be displayed as a video in the UI.

To implement this, the method follows these steps:

1. It retrieves the current animation state from the WebGAL store.
2. It extracts the animation name and duration from the sentence passed.
3. It gets the animation object defined by the `getAnimationObject` function.
4. If the animation object is retrieved successfully, it logs the animation name and duration.
5. It gets the animation target from the sentence passed.
6. It generates a unique animation name based on the animation target and duration.
7. It creates the animation function by passing the animation object and animation name to the `getAnimationObject` function.
8. It creates a stop function that is called when the animation is ended.
9. It sets the performing name, duration, and isHoldOn properties.
10. It sets the stopTimeout property if not already set.
11. It calls the animation function with the animation target and animation duration.
12. It returns an IPerform object with the performing name, duration, isHoldOn, stopFunction, and blockingNext properties.

This method can be used in various Pixi and web-based games, where the background animation can be easily added to the game with the effects of a selected sentence.


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { webgalAnimations } from '@/Core/controller/stage/pixi/animations';
import { IAnimationObject } from '@/Core/controller/stage/pixi/PixiController';
import { logger } from '@/Core/util/etc/logger';
import { webgalStore } from '@/store/store';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 设置背景动画
 * @param sentence
 */
export const setComplexAnimation = (sentence: ISentence): IPerform => {
  const startDialogKey = webgalStore.getState().stage.currentDialogKey;
  const animationName = sentence.content;
  const animationDuration = (getSentenceArgByKey(sentence, 'duration') ?? 0) as number;
  const target = (getSentenceArgByKey(sentence, 'target') ?? 0) as string;
  const key = `${target}-${animationName}-${animationDuration}`;
  const animationFunction: Function | null = getAnimationObject(animationName);
  let stopFunction: () => void = () => {};
  if (animationFunction) {
    logger.debug(`动画${animationName}作用在${target}`, animationDuration);
    const animationObj: IAnimationObject = animationFunction(target, animationDuration);
    WebGAL.gameplay.pixiStage?.stopPresetAnimationOnTarget(target);
    WebGAL.gameplay.pixiStage?.registerAnimation(animationObj, key, target);
    stopFunction = () => {
      const endDialogKey = webgalStore.getState().stage.currentDialogKey;
      const isHasNext = startDialogKey !== endDialogKey;
      WebGAL.gameplay.pixiStage?.removeAnimationWithSetEffects(key);
    };
  }
  return {
    performName: key,
    duration: animationDuration,
    isHoldOn: false,
    stopFunction,
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

该函数的作用是获取给定动画名称的动画对象（也称为动画剪辑或动画实例）。它返回一个函数，如果找到了动画对象，则返回该函数的`animationGenerateFunc`属性；否则，返回`null`。

具体来说，函数首先使用`webgalAnimations`数组中的`find`方法来查找给定动画名称的动画对象。如果找到了动画对象，函数将返回该动画对象的`animationGenerateFunc`属性，这个函数返回一个动画对象，它包含一个可以调用该函数的实例，可以通过设置`animationName`属性来设置要缓存的动画名称。如果找到了动画对象，函数将返回`null`，否则将返回`null`。


```js
function getAnimationObject(animationName: string): Function | null {
  const result = webgalAnimations.find((e) => e.name === animationName);
  logger.debug('装载动画', result);
  if (result) {
    return result.animationGenerateFunc;
  }
  return null;
}

```

# `/packages/webgal/src/Core/gameScripts/setFilter.ts`

这段代码定义了一个名为 `setFilter` 的函数，它接受一个 `ISentence` 类型的参数 `sentence`。

通过这个函数，我们可以将背景效果设置为在指定的 `ISentence` 对象上应用一种名为 "none" 的效果，其持续时间为0，即不显示任何效果，停止函数是一个空函数，因为没有执行任何特定操作。

更具体地说，这个函数的作用是：

1. 通过 `performName` 属性，设置名为 "none" 的效果，作为默认效果。
2. 通过 `duration` 属性，设置该效果持续的时间，为0，表示不会显示任何效果。
3. 通过 `isHoldOn` 属性，设置该效果是否保持显示，为false，表示不会显示任何效果。
4. 通过 `stopFunction` 属性，设置该效果在停止时执行的函数，为空函数，因为没有执行任何特定操作。
5. 通过 `blockingNext` 属性，设置该效果在后续事件继续时是否阻止它们，为false，表示不会阻止后续事件。
6. 通过 `blockingAuto` 属性，设置该效果在停止时是否自动清除，为true，表示背景将自动清除。
7. 通过 `stopTimeout` 属性，设置背景效果停止的时间，如果没有设置该属性，则默认情况下为undefined。

总之，这个函数可以用来设置一个简单的背景效果，在指定的 `ISentence` 对象上应用。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';

/**
 * 设置背景效果
 * @param sentence
 */
export const setFilter = (sentence: ISentence): IPerform => {
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/setTempAnimation.ts`

该代码的主要作用是创建一个临时动画对象，并将其存储到WebGAL中。

具体来说，代码会根据传入的句子对象（ISentence对象），从`@/Core/Modules/animationFunctions`中获取动画对象，然后根据传入的时间（动画对象的持续时间，例如毫秒数）生成一个临时动画对象，最后将其存储到WebGAL中的`/scene/animations/`路径下，以便在场景中使用。

此外，代码还使用了`baseTransform`函数对传入的句子对象进行转换，以便在生成临时动画对象时对其进行动画效果的处理。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { IAnimationObject } from '@/Core/controller/stage/pixi/PixiController';
import { logger } from '@/Core/util/etc/logger';
import { webgalStore } from '@/store/store';
import { generateTimelineObj } from '@/Core/controller/stage/pixi/animations/timeline';
import cloneDeep from 'lodash/cloneDeep';
import { baseTransform } from '@/store/stageInterface';
import { IUserAnimation } from '../Modules/animations';
import { getAnimateDuration, getAnimationObject } from '@/Core/Modules/animationFunctions';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 设置临时动画
 * @param sentence
 */
```

根据提供的代码，我们可以看到它是一个用于在 WebGL 上添加动画效果的函数，即 `ISentence)。它接受一个 `sentence` 参数，它是需要添加动画效果的文本。函数返回一个 `IPerform` 对象，它包含添加动画效果的一些信息，如 `performName`、`duration`、`isHoldOn`、`stopFunction` 等。

具体来说，函数在添加动画效果时会执行以下操作：

1. 从 `webgalStore` 中获取当前对话框的 ID，以及需要添加的动画效果的名称和持续时间。
2. 从 `sentence` 中提取动画效果的 JSON 数据，并将其解析为 `IAnimationObject` 对象。
3. 创建一个新的动画效果对象，包含需要添加的动画效果名称、持续时间等。
4. 使用 `WebGAL.animationManager` 类的 `addAnimation` 方法将动画效果添加到游戏进程的动画列表中。
5. 使用 `setTimeout` 函数来设置动画效果的停止时间，当停止时间到达时，使用 `WebGAL.gameplay.pixiStage?.stopPresetAnimationOnTarget` 方法停止当前动画在指定的目标上。
6. 返回一个 `IPerform` 对象，其中包含添加动画效果的一些信息。

对于使用哪种方法来添加动画效果，它取决于应用程序的设置。可以提供更多的上下文和选项，让用户可以根据需要选择不同的添加方式。


```js
export const setTempAnimation = (sentence: ISentence): IPerform => {
  const startDialogKey = webgalStore.getState().stage.currentDialogKey;
  const animationName = (Math.random() * 10).toString(16);
  const animationString = sentence.content;
  let animationObj;
  try {
    animationObj = JSON.parse(animationString);
  } catch (e) {
    animationObj = [];
  }
  const newAnimation: IUserAnimation = { name: animationName, effects: animationObj };
  WebGAL.animationManager.addAnimation(newAnimation);
  const animationDuration = getAnimateDuration(animationName);
  const target = (getSentenceArgByKey(sentence, 'target') ?? 0) as string;
  const key = `${target}-${animationName}-${animationDuration}`;
  let stopFunction = () => {};
  setTimeout(() => {
    WebGAL.gameplay.pixiStage?.stopPresetAnimationOnTarget(target);
    const animationObj: IAnimationObject | null = getAnimationObject(animationName, target, animationDuration);
    if (animationObj) {
      logger.debug(`动画${animationName}作用在${target}`, animationDuration);
      WebGAL.gameplay.pixiStage?.registerAnimation(animationObj, key, target);
    }
  }, 0);
  stopFunction = () => {
    setTimeout(() => {
      const endDialogKey = webgalStore.getState().stage.currentDialogKey;
      const isHasNext = startDialogKey !== endDialogKey;
      WebGAL.gameplay.pixiStage?.removeAnimationWithSetEffects(key);
    }, 0);
  };

  return {
    performName: key,
    duration: animationDuration,
    isHoldOn: false,
    stopFunction,
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/setTextbox.ts`

该代码是一个 JavaScript 函数，名为 `setTextbox`，属于一个名为 `sceneInterface` 的控制器类别的子类。

该函数接收一个参数 `sentence`，它是一个 ISentence 类型的数据，表示当前场景中的一个句子对象。这个函数的作用是根据 `sentence.content` 的值，决定是否禁用文本框，并返回一个 IPerform 类型的数据，表示表演内容的描述。

具体来说，如果 `sentence.content` 的值为 'hide'，则调用一个名为 `setStage` 的 action，将 `isDisableTextbox` 的键设置为 `true`，并将此状态通知给 reducer，以便在状态中更新对应的值。如果 `sentence.content` 的值为 'show'，则调用一个名为 `setStage` 的 action，将 `isDisableTextbox` 的键设置为 `false`，并将此状态通知给 reducer，以便在状态中更新对应的值。

另外，函数还返回一个 IPerform 类型的数据，包括 performName、duration、isHoldOn、stopFunction 和 stopTimeout 等属性。这些属性的含义如下：

* performName：表演名称，这里目前没有使用；
* duration：表演持续时间，这里目前没有使用；
* isHoldOn：是否抓住，这里目前没有使用；
* stopFunction：停止函数，这里目前没有使用；
* stopTimeout：停止超时时间，这里目前没有使用。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';

/**
 * 语句执行的模板代码
 * @param sentence
 */
export function setTextbox(sentence: ISentence): IPerform {
  if (sentence.content === 'hide') {
    webgalStore.dispatch(setStage({ key: 'isDisableTextbox', value: true }));
  } else {
    webgalStore.dispatch(setStage({ key: 'isDisableTextbox', value: false }));
  }
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
}

```

# `/packages/webgal/src/Core/gameScripts/setTransform.ts`

这段代码的主要作用是创建一个 `ISentence` 对象，然后使用它来设置一个 `IPerform` 对象的动作。首先，从 `@/Core/controller/scene/sceneInterface` 中导入 `ISentence` 和 `IPerform` 对象。接着，从 `@/Core/Modules/perform/performInterface` 中导入 `getSentenceArgByKey` 函数，用于从 `ISentence` 对象中获取特定的参数。然后，从 `@/Core/controller/stage/pixi/PixiController` 中导入 `IAnimationObject` 对象，从 `@/Core/store/webgal` 对象中导入 `logger` 函数，从 `@/Core/controller/stage/pixi/animations/timeline` 中导入 `generateTimelineObj` 函数。

代码中使用了一个名为 `cloneDeep` 的函数，它是 `lodash` 库中的一个方法，用于从指定对象中克隆一个deep副本。此外，还使用了一个名为 `baseTransform` 的函数，它是 `@/store/stageInterface` 中的一个方法，用于将一个Transform对象转换为基本的变换对象。最后，从 `@/Core/gameScripts/function/generateTransformAnimationObj` 中导入 `generateTransformAnimationObj` 函数，从 `@/Core/controller/stage/pixi/animations/timeline` 中导入 `IUserAnimation` 对象，生成一个Transform动画对象。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { IAnimationObject } from '@/Core/controller/stage/pixi/PixiController';
import { logger } from '@/Core/util/etc/logger';
import { webgalStore } from '@/store/store';
import { generateTimelineObj } from '@/Core/controller/stage/pixi/animations/timeline';
import cloneDeep from 'lodash/cloneDeep';
import { baseTransform, ITransform } from '@/store/stageInterface';
import { IUserAnimation } from '../Modules/animations';
import { generateTransformAnimationObj } from '@/Core/gameScripts/function/generateTransformAnimationObj';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 设置变换
 * @param sentence
 */
```

动画数据存储为 JSON 格式的动画数据对象，每个对象包含以下属性：

- name: 动画的名称，用于在游戏内显示动画的提示信息。
- effects: 动画效果列表，是一个包含多个效果的对象。
 - duration: 动画的持续时间，以秒为单位。
 - timeout: 动画的延迟时间，以秒为单位。

这里的效果列表，指的是所有的动画效果，不包含目标(target)为null的效果。

使用正则表达式将 JSON 格式的数据解析为 Transform 对象，通过 `generateTransformAnimationObj` 函数生成动画对象，如果解析失败则返回一个空数组 `[]`。

生成动画对象后，将动画对象存储到 `animationManager` 实例的 `addAnimation` 方法中，然后根据动画名称获取动画的持续时间(duration)，最终返回动画对象作为 `animationManager` 实例的 `addAnimation` 方法的第一个参数，这样就完成了动画数据的设置。


```js
export const setTransform = (sentence: ISentence): IPerform => {
  const startDialogKey = webgalStore.getState().stage.currentDialogKey;
  const animationName = (Math.random() * 10).toString(16);
  const animationString = sentence.content;
  let animationObj: (ITransform & {
    duration: number;
  })[];
  const duration = getSentenceArgByKey(sentence, 'duration');
  const target = (getSentenceArgByKey(sentence, 'target') ?? 0) as string;
  try {
    const frame = JSON.parse(animationString) as ITransform & { duration: number };
    animationObj = generateTransformAnimationObj(target, frame, duration);
  } catch (e) {
    // 解析都错误了，歇逼吧
    animationObj = [];
  }

  const newAnimation: IUserAnimation = { name: animationName, effects: animationObj };
  WebGAL.animationManager.addAnimation(newAnimation);
  const animationDuration = getAnimateDuration(animationName);

  const key = `${target}-${animationName}-${animationDuration}`;
  let stopFunction = () => {};
  setTimeout(() => {
    WebGAL.gameplay.pixiStage?.stopPresetAnimationOnTarget(target);
    const animationObj: IAnimationObject | null = getAnimationObject(animationName, target, animationDuration);
    if (animationObj) {
      logger.debug(`动画${animationName}作用在${target}`, animationDuration);
      WebGAL.gameplay.pixiStage?.registerAnimation(animationObj, key, target);
    }
  }, 0);
  stopFunction = () => {
    setTimeout(() => {
      const endDialogKey = webgalStore.getState().stage.currentDialogKey;
      const isHasNext = startDialogKey !== endDialogKey;
      WebGAL.gameplay.pixiStage?.removeAnimationWithSetEffects(key);
    }, 0);
  };

  return {
    performName: key,
    duration: animationDuration,
    isHoldOn: false,
    stopFunction,
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

该函数的作用是获取一个给定名称的动画对象，该动画对象与给定的目标元素发生交互，具有指定持续时间。它返回一个包含动画对象对象属性的对象，或者返回一个空对象。

具体来说，它首先从内存中的动画系统中查找给定名称的动画对象。如果找到了动画对象，该函数将其封装为一个新的对象，该对象包含原始对象的属性和新的动画系统 duration。然后，它将新的对象从内存中删除 duration 属性，并将其设置为动画系统 duration 的值除以 1000。接下来，该函数将封装后的对象传递给一个名为 generateTimelineObj 的函数，该函数将在给定的目标元素上生成动画对象。如果 getAnimationObject 在尝试查找给定名称的动画对象时出现任何错误，它将返回一个空对象，否则它将返回动画对象。


```js
function getAnimationObject(animationName: string, target: string, duration: number) {
  const effect = WebGAL.animationManager.getAnimations().find((ani) => ani.name === animationName);
  if (effect) {
    const mappedEffects = effect.effects.map((effect) => {
      const newEffect = cloneDeep({ ...baseTransform, duration: 0 });
      Object.assign(newEffect, effect);
      newEffect.duration = effect.duration / 1000;
      return newEffect;
    });
    logger.debug('装载自定义动画', mappedEffects);
    return generateTimelineObj(mappedEffects, target, duration);
  }
  return null;
}

```

该函数的作用是获取给定动画名称的动画效果持续时间。它首先使用 `WebGAL.animationManager.getAnimations()` 获取所有与给定动画名称相同的动画效果。如果找到了动画效果，它将获取该动画效果的持续时间并将其存储在 `duration` 变量中。如果找不到动画效果，它将返回 0。函数的返回值是动画效果的持续时间。


```js
function getAnimateDuration(animationName: string) {
  const effect = WebGAL.animationManager.getAnimations().find((ani) => ani.name === animationName);
  if (effect) {
    let duration = 0;
    effect.effects.forEach((e) => {
      duration += e.duration;
    });
    return duration;
  }
  return 0;
}

```

# `/packages/webgal/src/Core/gameScripts/setTransition.ts`

这段代码定义了一个名为 `setTransition` 的函数，它接受一个参数 `sentence`，表示一个序列动作（sentence）。这个函数的作用是设置进入和退出转场效果，并返回一个 `IPerform` 对象。

具体来说，这个函数首先根据传入的 `sentence` 对象，查找并设置进入和退出转场效果的关键帧。然后，使用 `WebGAL` 中的 `animationManager` 类，设置对应的进入和退出动画。最后，返回一个关于 `performName`、`duration`、`isHoldOn`、`stopFunction`、`blockingNext`、`blockingAuto` 和 `stopTimeout` 的 `IPerform` 对象。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import cloneDeep from 'lodash/cloneDeep';
import { updateCurrentEffects } from '@/Core/controller/stage/pixi/PixiController';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { setStage } from '@/store/stageReducer';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 设置转场效果
 * @param sentence
 */
export const setTransition = (sentence: ISentence): IPerform => {
  // 根据参数设置指定位置
  let key = '';
  for (const e of sentence.args) {
    if (e.key === 'target') {
      key = e.value.toString();
    }
  }
  if (getSentenceArgByKey(sentence, 'enter')) {
    WebGAL.animationManager.nextEnterAnimationName.set(key, getSentenceArgByKey(sentence, 'enter')!.toString());
  }
  if (getSentenceArgByKey(sentence, 'exit')) {
    WebGAL.animationManager.nextExitAnimationName.set(key + '-off', getSentenceArgByKey(sentence, 'exit')!.toString());
  }
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => false,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/setVar.ts`

这是一个Python中的webgal组件，用于在webgal中执行脚本以获取游戏变量的值并设置游戏变量。

它包含以下功能：

- `performName`：脚本执行时显示的名称，目前为空。
- `duration`：脚本在webgal中持续的时间，单位是毫秒。
- `isHoldOn`：脚本是否被挂起，目前为false。
- `stopFunction`：脚本停止时执行的函数，目前为空。
- `blockingNext`：脚本下一帧是否可以继续执行，目前为false。
- `blockingAuto`：脚本是否可以挂起，目前为true。
- `stopTimeout`：脚本停止的有效时间，单位是毫秒。
- `compile`：脚本编译成字符串的函数，用于在webgal中执行脚本。
- `expand`：函数字符串展开成字符串的函数，用于在webgal中执行展开的脚本。
- `setStageVar`：设置游戏变量值的函数，用于在webgal中执行设置游戏变量值的操作。
- `logger.debug`：用于输出调试信息的函数，可以输出一个消息和一个对象，对象包含消息信息和函数参数。

它的`setStageVar`函数可以用于设置游戏变量的值，例如：
```jscss
const valExp = sentence.content.split(/=/)[1];
const valExp2 = valExp.split(/([+\-*\/())/).map(val => {
 if (val.match(/[a-zA-Z]/)) {
   return getValueFromState(val).toString();
 } else {
   return val;
 }
});
const exp = compile(valExp2);
const result = exp();
webgalStore.dispatch(setStageVar({ key, value: result }));
```
`getValueFromState`函数用于从游戏状态中获取变量值，并将其转换为字符串。`compile`函数用于将表达式编译成字符串，以便在webgal中执行。`expand`函数用于展开字符串，`logger.debug`函数用于输出调试信息。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import { setStageVar } from '@/store/stageReducer';
import { logger } from '@/Core/util/etc/logger';
import { compile } from 'angular-expressions';

/**
 * 设置变量
 * @param sentence
 */
export const setVar = (sentence: ISentence): IPerform => {
  // 先把表达式拆分为变量名和赋值语句
  if (sentence.content.match(/=/)) {
    const key = sentence.content.split(/=/)[0];
    const valExp = sentence.content.split(/=/)[1];
    // 如果包含加减乘除号，则运算
    if (valExp.match(/[+\-*\/()]/)) {
      // 先取出运算表达式中的变量
      const valExpArr = valExp.split(/([+\-*\/()])/g);
      // 将变量替换为变量的值，然后合成表达式字符串
      const valExp2 = valExpArr
        .map((e) => {
          if (e.match(/[a-zA-Z]/)) {
            return getValueFromState(e).toString();
          } else return e;
        })
        .reduce((pre, curr) => pre + curr, '');
      const exp = compile(valExp2);
      const result = exp();
      webgalStore.dispatch(setStageVar({ key, value: result }));
    } else if (valExp.match(/true|false/)) {
      if (valExp.match(/true/)) {
        webgalStore.dispatch(setStageVar({ key, value: true }));
      }
      if (valExp.match(/false/)) {
        webgalStore.dispatch(setStageVar({ key, value: false }));
      }
    } else {
      if (!isNaN(Number(valExp))) {
        webgalStore.dispatch(setStageVar({ key, value: Number(valExp) }));
      } else webgalStore.dispatch(setStageVar({ key, value: valExp }));
    }
    logger.debug('设置变量：', { key, value: webgalStore.getState().stage.GameVar[key] });
  }
  return {
    performName: 'none',
    duration: 0,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

这段代码定义了一个名为 `getValueFromState` 的函数，它接受一个名为 `key` 的字符串参数。函数返回一个数值、字符串或布尔值，根据给定的 key 从 webgalStore 的游戏状态中获取相应的值。

函数首先定义了一个名为 `ret` 的变量，初始值为 0。接下来，它使用 if 语句判断给定的 key 是否存在于 `webgalStore.getState().stage.GameVar` 对象中。如果是，函数将 `key` 的值返回给 `ret`，否则不会做任何操作。

最后，函数返回 `ret` 的值。这个函数可以被用来从游戏状态中获取变量值，例如 `webgalStore.getState().stage.GameVar.hasOwnProperty(key)` 将会返回 `true`，因为 `webgalStore.getState().stage.GameVar` 对象中可能存在名为 `key` 的属性。


```js
export function getValueFromState(key: string) {
  let ret: number | string | boolean = 0;
  if (webgalStore.getState().stage.GameVar.hasOwnProperty(key)) {
    ret = webgalStore.getState().stage.GameVar[key];
  }
  return ret;
}

```