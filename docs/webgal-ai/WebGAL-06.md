# WebGal源码解析 6

# `/packages/webgal/src/Core/gameScripts/showVars.ts`

This is a JavaScript method that performs a simple conversation. The method takes in a sentence as an input and returns an IPerform object.

The IPerform object contains the following properties:

- `performName`: The name of the performance.
- `duration`: The duration of the performance, measured in characters.
- `isHoldOn`: A boolean indicating whether the performance is being held (cancelled) or not.
- `stopFunction`: A function to be called when the performance is being cancelled.
- `blockingNext`: A boolean indicating whether to block the next performer to appear, or allow all.
- `blockingAuto`: A boolean indicating whether to block the next performer to appear, always.
- `stopTimeout`: An optional property for the timeout for the stop function.

The method uses the `webgalStore` object to perform an action, which is not defined in the code provided. It is recommended to replace `webgalStore` with the actual implementation for any environment that requires this store.


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import styles from '../../Components/Stage/TextBox/textbox.module.scss';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';
import { logger } from '@/Core/util/etc/logger';
import { getRandomPerformName } from '@/Core/Modules/perform/performController';
import { PERFORM_CONFIG } from '@/Core/config/config';

/**
 * 进行普通对话的显示
 * @param sentence 语句
 * @return {IPerform} 执行的演出
 */
export const showVars = (sentence: ISentence): IPerform => {
  const stageState = webgalStore.getState().stage;
  const userDataState = webgalStore.getState().userData;
  const dispatch = webgalStore.dispatch;
  // 设置文本显示
  dispatch(setStage({ key: 'showText', value: JSON.stringify(stageState.GameVar) }));
  dispatch(setStage({ key: 'showName', value: '展示变量' }));
  logger.debug('展示变量：', stageState.GameVar);
  setTimeout(() => {
    const textElements = document.querySelectorAll('.' + styles.TextBox_textElement_start);
    const textArray = [...textElements];
    textArray.forEach((e) => {
      e.className = styles.TextBox_textElement;
    });
  }, 0);
  const performInitName: string = getRandomPerformName();
  const textDelay = PERFORM_CONFIG.textInitialDelay - 20 * userDataState.optionData.textSpeed;
  const endDelay = 750 - userDataState.optionData.textSpeed * 250;
  return {
    performName: performInitName,
    duration: sentence.content.length * textDelay + endDelay,
    isHoldOn: false,
    stopFunction: () => {
      const textElements = document.querySelectorAll('.' + styles.TextBox_textElement);
      const textArray = [...textElements];
      textArray.forEach((e) => {
        e.className = styles.TextBox_textElement_Settled;
      });
    },
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/template.ts`

这段代码定义了一个名为 `template` 的函数，它会根据传入的 `sentence` 参数生成一个 `IPerform` 对象，用于控制句子的执行。

具体来说，这个 `template` 函数的作用如下：

1. 返回一个包含 `performName`、`duration`、`isHoldOn`、`stopFunction` 和 `blockingNext`、`blockingAuto`、`stopTimeout` 的对象。这些对象用于描述要执行的句子的性能特征。
2. 通过 `IPerform` 的 `performName` 属性，指定执行的动作，此处未做具体赋值。
3. 通过 `duration` 属性，指定句子需要持续的时间，单位为毫秒。
4. 通过 `isHoldOn` 属性，指定是否要将句子暂停，如果是，则不会播放音频，此处未做具体赋值。
5. 通过 `stopFunction` 属性，指定当句子停止时，要执行的函数，此处未做具体赋值。
6. 通过 `blockingNext` 属性的 `$》` 符号，指定句子是否可以继续执行下一条语句，此处未做具体赋值。
7. 通过 `blockingAuto` 属性的 `$》` 符号，指定句子是否可以继续执行下一条语句，此处未做具体赋值。
8. 通过 `stopTimeout` 属性，指定句子停止的时间，单位为毫秒。目前未设置此属性，因此会在句子播放过程中根据 `blockingAuto` 属性进行自动清除。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';

/**
 * 语句执行的模板代码
 * @param sentence
 */
export const template = (sentence: ISentence): IPerform => {
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

# `/packages/webgal/src/Core/gameScripts/unlockBgm.ts`

这段代码的作用是实现了一个解锁背景音乐的功能，用于某个场景的评价。

具体来说，这段代码：

1. 导入了需要用到的三个模块：ISentence，IPerform，以及一个名为webgalStore的store。
2. 导入了本地存储库localforage，以及一个logger用于记录日志信息。
3. 引入了@/Core库中的一个名为WebGAL的类，以及一个名为logger的函数。
4. 定义了一个名为unlockBgm的方法，它接收一个ISentence类型的参数sentence，然后执行一系列操作来解锁BGM。
5. 在unlockBgm方法的内部，首先获取要解锁的BGM的URL，然后根据需要设置一些属性和状态，如名称和系列等。
6. 使用logger记录了一些日志信息，用于调试解锁BGM的过程。
7. 调用WebGAL中的dispatch函数，将解锁BGM的请求发送到store中。
8. 使用localforage存储BGM的下载状态，并在下载成功后将其保存到本地。
9. 如果下载失败，设置一些默认的停止函数，以便在BGM下载失败时进行回退。
10. 将unlockBgm方法的performName设置为'none'，以便在子组件中使用时可以默认使用。
11. 将unlockBgm方法的duration设置为0，以便在子组件中可以默认禁止动画效果。
12. 将unlockBgm方法的isHoldOn设置为false，以便在子组件中可以设置为true来保持BGM的自动播放。
13. 将unlockBgm方法的stopFunction设置为一个空函数，以便在子组件中提供一个默认的停止按钮。
14. 将unlockBgm方法的stopTimeout设置为一个未定义的值，以便在子组件中可以设置具体的停止时间。
15. 在unlockBgm方法的stopTimer属性中，暂时不用，后面会交给自动清除。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import { unlockBgmInUserData } from '@/store/userDataReducer';
import localforage from 'localforage';
import { logger } from '@/Core/util/etc/logger';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 解锁bgm
 * @param sentence
 */
export const unlockBgm = (sentence: ISentence): IPerform => {
  const url = sentence.content;
  let name = sentence.content;
  let series = 'default';
  sentence.args.forEach((e) => {
    if (e.key === 'name') {
      name = e.value.toString();
    }
    if (e.key === 'series') {
      series = e.value.toString();
    }
  });
  logger.info(`解锁BGM：${name}，路径：${url}，所属系列：${series}`);
  webgalStore.dispatch(unlockBgmInUserData({ name, url, series }));
  const userDataState = webgalStore.getState().userData;
  localforage.setItem(WebGAL.gameKey, userDataState).then(() => {});
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

# `/packages/webgal/src/Core/gameScripts/unlockCg.ts`

这段代码定义了一个名为 `unlockCg` 的函数，它接受一个 `ISentence` 类型的参数 `sentence`。这个函数的作用是解锁一个指定的 `IPerform`，使得该 `IPerform` 可以被用于生成游戏世界中的图形元素。

具体来说，这个函数首先获取传入 `sentence` 的内容URL，然后根据给定的信息（例如名称、系列）进行相应的解析和处理。接着，它会调用一个名为 `webgalStore.dispatch` 的方法，传递一个包含解锁信息的 `unlockCgInUserData` 对象。这个对象包括了解锁信息的名称、URL和系列等信息，函数会将其存储在 `webgalStore.state.userData` 中。

最后，函数使用 `localforage.setItem` 方法将 `WebGAL.gameKey` 和 `unlockCgInUserData` 组成的键值对存储在 `localforage` 中的话。这样，当程序运行结束时，即使服务器没有关闭，也不会丢失解锁信息。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import { unlockCgInUserData } from '@/store/userDataReducer';
import { logger } from '@/Core/util/etc/logger';
import localforage from 'localforage';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 解锁cg
 * @param sentence
 */
export const unlockCg = (sentence: ISentence): IPerform => {
  const url = sentence.content;
  let name = sentence.content;
  let series = 'default';
  sentence.args.forEach((e) => {
    if (e.key === 'name') {
      name = e.value.toString();
    }
    if (e.key === 'series') {
      series = e.value.toString();
    }
  });
  logger.info(`解锁CG：${name}，路径：${url}，所属系列：${series}`);
  webgalStore.dispatch(unlockCgInUserData({ name, url, series }));
  const userDataState = webgalStore.getState().userData;
  localforage.setItem(WebGAL.gameKey, userDataState).then(() => {});
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

# `/packages/webgal/src/Core/gameScripts/function/conentsCash.ts`

这两段代码定义了两个独立的类 `FigureConentsCash` 和 `VoiceConentsCash`，旨在实现将文本内容存储在现金中的目的。每个类都有一个私有属性 `_content`，用于存储要存储的文本内容。

每个类都有一个 `push` 方法，用于将所提供的文本内容添加到现金中，并返回该内容的字符串表示。例如，在 `FigureConentsCash` 中，您可以调用 `push` 方法将文本字符串添加到现金中，并从现金中返回该内容的字符串表示。类似地，在 `VoiceConentsCash` 中，您也可以调用 `push` 方法将文本字符串添加到现金中，并从现金中返回该内容的字符串表示。

每个类还有一个 `pop` 方法，用于从现金中删除一个字符串并返回其内容。例如，在 `FigureContsCash` 中，您可以调用 `pop` 方法从现金中删除字符串，并从该字符串中返回其内容。类似地，在 `VoiceConentsCash` 中，您也可以调用 `pop` 方法从现金中删除字符串，并从该字符串中返回其内容。

通过使用这些类，您可以将文本内容存储为现金，并从现金中提取有用的信息。


```js
export class FigureConentsCash {
  private _content = '';

  // public constructor() {}
  public push(sentenceContent: string) {
    this._content = sentenceContent;
  }
  public pop(): string {
    const constContent = this._content;
    this._content = '';
    return constContent;
  }
}

export class VoiceConentsCash {
  private _content = '';

  // public constructor() {}
  public push(sentenceContent: string) {
    this._content = sentenceContent;
  }
  public pop(): string {
    const constContent = this._content;
    this._content = '';
    return constContent;
  }
}

```

这段代码的作用是创建两个 `FigureConentsCash` 和 `VoiceConentsCash` 的实例。 `FigureConentsCash` 和 `VoiceConentsCash` 都是从同一个 `Cash` 类中导出的不同实例，但它们提供不同的功能。 `FigureConentsCash` 负责绘制现金的图形，而 `VoiceConentsCash` 则负责将文本转换为现金的声音效果。


```js
export const figureCash = new FigureConentsCash();
export const voiceCash = new VoiceConentsCash();

```

# `/packages/webgal/src/Core/gameScripts/function/generateTransformAnimationObj.ts`

这段代码定义了一个名为 `generateTransformAnimationObj` 的函数，它接受四个参数：

1. `target`：需要应用变换的对象，例如一个字符串或一个包含字符串的数组。
2. `applyFrame`：需要应用的变换帧，它的格式为 `ITransformFrame` 类型。这个类型包含了动画的起始时间、持续时间和强度等信息。
3. `duration`：动画的持续时间，可以是数字、字符串或布尔值。如果是数字，那么它将直接作为 `applyFrame` 的 `duration` 属性的值。如果是字符串，那么它将被解析为持续时间（秒或毫秒）。如果是布尔值，那么它将被设置为 `true`，从而立即应用变换，而不会等待动画的完成。如果这个参数为 `null`，那么它将被忽略。
4. `transform`：需要应用的变换效果，可以是字符串、数字或布尔值。如果这个参数是一个字符串，那么它将被解析为一个效果的 ID，而不是具体的变换效果。如果这个参数是一个数字，那么它将是一个数字 ID，对应一个定义在 `@/store/stageInterface` 中的 `ITransform` 类型。如果这个参数是一个布尔值，那么它将被设置为 `true`，从而立即应用变换，而不会等待动画的完成。

函数的实现是首先查找 `target` 对应的变换效果，然后创建一个 `AnimationFrame` 对象，其中 `duration` 属性根据传入的参数进行了设置。如果 `duration` 是一个数字，那么直接将 `applyFrame` 的 `duration` 属性设置为这个数字。如果 `duration` 是一个字符串，那么将 `applyFrame` 中的 `duration` 属性设置为这个字符串。如果 `transform` 是一个字符串，那么将 `applyFrame` 中的 `transform` 属性设置为这个字符串。如果 `transform` 是一个数字，那么将 `applyFrame` 中的 `transform` 属性设置为这个数字。最后，返回动画对象数组 `animationObj`。


```js
import { ITransform } from '@/store/stageInterface';
import { webgalStore } from '@/store/store';

type AnimationFrame = ITransform & { duration: number };
type AnimationObj = Array<AnimationFrame>;

export function generateTransformAnimationObj(
  target: string,
  applyFrame: AnimationFrame,
  duration: number | string | boolean | null,
): AnimationObj {
  let animationObj;
  // 获取那个 target 的当前变换
  const transformState = webgalStore.getState().stage.effects;
  const targetEffect = transformState.find((effect) => effect.target === target);
  applyFrame.duration = 500;
  if (duration && typeof duration === 'number') {
    applyFrame.duration = duration;
  }
  animationObj = [applyFrame];
  // 找到 effect
  if (targetEffect) {
    const effectWithDuration = { ...targetEffect!.transform!, duration: 0 };
    animationObj.unshift(effectWithDuration);
  } else {
    // 应用默认effect，也就是最终的 effect 的 alpha = 0 版本
    const effectWithDuration = { ...applyFrame, alpha: 0, duration: 0 };
    animationObj.unshift(effectWithDuration);
  }
  return animationObj;
}

```

# `/packages/webgal/src/Core/gameScripts/function/jmp.ts`

该代码定义了一个名为 "jmp" 的函数，用于在游戏中的场景中查找并移除标签。以下是函数的实现细节：

1. 导入了两个函数模块：`commandType` 和 `nextSentence`。
2. 引入了 `WebGAL` 模块。
3. 定义了 `jmp` 函数，它接收一个标签名称作为参数。
4. 在函数内部，首先定义了一个变量 `currentLine`，用于跟踪当前场景中的句子编号。
5. 然后使用 `forEach` 方法，遍历当前场景中所有可能的标签。
6. 对于每个标签，使用条件语句检查是否满足标签名称和标签内容的条件。如果是，就设置 `result` 变量为该标签的句子编号。
7. 最后，使用 `setTimeout` 函数，延迟 1 秒后，将场景中的第二个句子编号设置为 `result`。

该函数的作用是在游戏中，根据标签名称在场景中查找并移除标签，使得游戏更加顺畅和有趣。


```js
import { commandType } from '@/Core/controller/scene/sceneInterface';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';

import { WebGAL } from '@/Core/WebGAL';

export const jmp = (labelName: string) => {
  // 在当前场景中找到指定的标签。
  const currentLine = WebGAL.sceneManager.sceneData.currentSentenceId;
  let result = currentLine;
  WebGAL.sceneManager.sceneData.currentScene.sentenceList.forEach((sentence, index) => {
    if (sentence.command === commandType.label && sentence.content === labelName && index >= currentLine) {
      result = index;
    }
  });
  WebGAL.sceneManager.sceneData.currentSentenceId = result;
  setTimeout(nextSentence, 1);
};

```

# `/packages/webgal/src/Core/gameScripts/function/strIf.ts`

这段代码使用了 Angular Expressions 的语法来定义一个名为 strIf 的函数，它会接受一个字符串参数 s。这个函数返回一个布尔值，具体取决于字符串 s 是否为真。

strIf 函数的核心部分是一个条件表达式，使用了 logical AND 和 logical OR 的组合来检查字符串是否为真。如果条件表达式的值为 true，函数返回真；否则返回 false。

接下来，函数使用了 Angular Expressions 的 compile 函数来 evaluate字符串表达式，并返回其结果。在编译字符串表达式时，Angular Expressions 会自动截取字符串中的所有非空字符，并将截取的结果返回。

最终，strIf 函数返回的结果是一个布尔值，表示给定字符串 s 的真假情况。


```js
import { compile } from 'angular-expressions';

export function strIf(s: string) {
  const res = compile(s);
  return res();
}

```

# `/packages/webgal/src/Core/gameScripts/function/vocalAnimation.ts`

该代码使用了@/Core/WebGAL包中的AudioContext类，创建了一个名为"audioContextWrapper"的接口对象。

这个接口对象有以下属性和方法：

- audioContext：一个AudioContext对象，用于在音频三中创建音频上下文。
- source：一个MediaElementAudioSourceNode对象，用于在音频三中获取音频源。
- analyser：一个AnalyserNode对象，用于在音频三中分析音频数据。
- dataArray：一个Uint8Array对象，用于保存音频数据。
- audioLevelInterval：一个返回类型为typeof setInterval的函数，用于设置音频中的音量。
- blinkTimerID：一个返回类型为typeof setTimeout的函数，用于设置音频中的自定义延迟时间。
- maxAudioLevel：一个用于存储最大音量的数字。

该代码还使用setInterval和setTimeout函数，分别用于设置音频中的音量每隔多少毫秒变化一次，以及设置自定义的延迟时间。这些函数都接受一个参数，即函数的执行次数，这些执行次数都被无限次调用，直到被明确地取消。


```js
import { WebGAL } from '@/Core/WebGAL';

interface IAudioContextWrapper {
  audioContext: AudioContext;
  source: MediaElementAudioSourceNode | null;
  analyser: AnalyserNode | undefined;
  dataArray: Uint8Array | undefined;
  audioLevelInterval: ReturnType<typeof setInterval>;
  blinkTimerID: ReturnType<typeof setTimeout>;
  maxAudioLevel: number;
}

// Initialize the object based on the interface
export const audioContextWrapper: IAudioContextWrapper = {
  audioContext: new AudioContext(),
  source: null,
  analyser: undefined,
  dataArray: undefined,
  audioLevelInterval: setInterval(() => {}, 0), // dummy interval
  blinkTimerID: setTimeout(() => {}, 0), // dummy timeout
  maxAudioLevel: 0,
};

```

这是一个 JavaScript 函数，分别定义了两个全局函数 `updateThresholds` 和 `performBlinkAnimation`。

1. `updateThresholds` 函数的作用是更新音频的阈值。函数接受一个参数 `audioLevel`，首先将其与 `audioContextWrapper.maxAudioLevel` 进行比较，然后将其与 `audioContextWrapper.maxAudioLevel` 中的最大值进行 `Math.max()` 运算，最后返回一个新的对象，包含两个属性：`OPEN_THRESHOLD` 和 `HALF_OPEN_THRESHOLD`，分别表示打开阈值和半 open 阈值。函数内部使用了 `WebGAL.gameplay.pixiStage` 类的 `performBlinkAnimation` 方法来实现动画效果，同时也使用了 `setTimeout()` 函数来设置动画的结束时间和延迟。

2. `performBlinkAnimation` 函数的作用是在游戏中的某个动画参数上进行动画效果的实现。函数接受一个包含参数 `params` 的对象，包含四个属性：`key`、`animationItem`、`pos` 和 `animationEndTime`。函数内部首先判断是否已经设置了动画结束时间，如果没有，则调用 `WebGAL.gameplay.pixiStage.performBlinkAnimation` 方法，传递参数 `params.key`、`params.animationItem` 和 `params.pos`，并设置 `isBlinking` 为 `false`，同时设置 `blinkTimerID` 为 `setTimeout()` 函数，在 200ms 之后执行 `blink()` 函数，其中 `blink()` 函数的作用是在动画结束后开启动画效果，并且在动画结束时延迟 200ms，之后再次每隔 300ms 延迟，直至动画结束。如果已经设置了动画结束时间，则立即执行 `blink()` 函数，其中 `blink()` 函数的作用是在动画时间内每隔 300ms 延迟，直至动画结束。函数内部使用了 `setTimeout()` 函数来设置动画的延迟时间。


```js
export const updateThresholds = (audioLevel: number) => {
  audioContextWrapper.maxAudioLevel = Math.max(audioLevel, audioContextWrapper.maxAudioLevel);
  return {
    OPEN_THRESHOLD: audioContextWrapper.maxAudioLevel * 0.75,
    HALF_OPEN_THRESHOLD: audioContextWrapper.maxAudioLevel * 0.5,
  };
};

export const performBlinkAnimation = (params: {
  key: string;
  animationItem: any;
  pos: string;
  animationEndTime: number;
}) => {
  let isBlinking = false;

  function blink() {
    if (isBlinking || (params.animationEndTime && Date.now() > params.animationEndTime)) return;
    isBlinking = true;
    WebGAL.gameplay.pixiStage?.performBlinkAnimation(params.key, params.animationItem, 'closed', params.pos);
    audioContextWrapper.blinkTimerID = setTimeout(() => {
      WebGAL.gameplay.pixiStage?.performBlinkAnimation(params.key, params.animationItem, 'open', params.pos);
      isBlinking = false;
      const nextBlinkTime = Math.random() * 300 + 3500;
      audioContextWrapper.blinkTimerID = setTimeout(blink, nextBlinkTime);
    }, 200);
  }
  blink();
};

```

这是一个 JavaScript 函数，名为 `performMouthAnimation`，它实现了嘴巴动画效果的制作。

该函数接收一个包含以下属性的参数：

* `audioLevel`：音频水平值，取值范围为0-1，值越大，嘴巴越张开。
* `OPEN_THRESHOLD`：张嘴打开阈值，高于该值时嘴巴打开，低于该值时嘴巴关闭。
* `HALF_OPEN_THRESHOLD`：张嘴半开阈值，高于该值时半张嘴，低于该值时全张嘴。
* `currentMouthValue`：当前嘴巴的位置，取值范围为0-1，值越大，嘴巴张开的程度越大。
* `lerpSpeed`：嘴部运动的步长，单位为毫秒。
* `key`：动画的快捷键，用于在动画期间使用键盘上的对应功能键。
* `animationItem`：嘴部动画的 DOM 元素。
* `pos`：嘴部的初始位置，可以是字符串或数字。

函数的主要逻辑如下：

1. 根据 `audioLevel` 属性的值，判断嘴巴是张开、半张开还是关闭。
2. 根据 `HALF_OPEN_THRESHOLD` 和 `OPEN_THRESHOLD` 属性的值，进一步缩小嘴巴张开的程度。
3. 根据 `currentMouthValue` 属性的值，计算嘴部运动的步长。
4. 根据 `animationItem` 属性的值，在动画期间执行嘴部动画。
5. 如果 `key` 属性的值为指定的快捷键，则使用对应的功能键调节嘴部动画的速度。
6. 如果 `pos` 属性的值为字符串，则将其作为动画的初始位置，否则将其作为动画的结束位置。

由于 `WebGAL.gameplay.pixiStage` 是一个尚未注册的模块，因此无法确定该模块的实现细节。


```js
// Updated getAudioLevel function
export const getAudioLevel = (analyser: AnalyserNode, dataArray: Uint8Array, bufferLength: number): number => {
  analyser.getByteFrequencyData(dataArray);
  let sum = 0;
  for (let i = 0; i < bufferLength; i++) {
    sum += dataArray[i];
  }
  return sum / bufferLength;
};

export const performMouthAnimation = (params: {
  audioLevel: number;
  OPEN_THRESHOLD: number;
  HALF_OPEN_THRESHOLD: number;
  currentMouthValue: number;
  lerpSpeed: number;
  key: string;
  animationItem: any;
  pos: string;
}) => {
  const { audioLevel, OPEN_THRESHOLD, HALF_OPEN_THRESHOLD, currentMouthValue, lerpSpeed, key, animationItem, pos } =
    params;

  let targetValue;
  if (audioLevel > OPEN_THRESHOLD) {
    targetValue = 1; // open
  } else if (audioLevel > HALF_OPEN_THRESHOLD) {
    targetValue = 0.5; // half_open
  } else {
    targetValue = 0; // closed
  }
  // Lerp
  const mouthValue = currentMouthValue + (targetValue - currentMouthValue) * lerpSpeed;

  let mouthState;
  if (mouthValue > 0.75) {
    mouthState = 'open';
  } else if (mouthValue > 0.25) {
    mouthState = 'half_open';
  } else {
    mouthState = 'closed';
  }
  if (animationItem !== undefined) {
    WebGAL.gameplay.pixiStage?.performMouthSyncAnimation(key, animationItem, mouthState, pos);
  }
};

```

# `/packages/webgal/src/Core/gameScripts/pixiPerformScripts/cherryBlossoms.ts`

这是一个关于 cherry blossoms(樱花)的物理引擎，主要实现了在给定的 stage width 和 stage height 内，以随机位置和速度下落、衰减和随机飘动的 cherry blossoms。同时，也支持在同一画面上多次呈现同一种 cherry blossoms，并对其进行数量控制。

该 cherry blossoms 引擎使用的是一个 Bunny 对象，其中包含一个 Bunny 本身，以及一个用于存储 Bunny List 中所有 cherry blossoms 的数组。Bunny List 中每个 cherry blossom 对象，包含一个其自身的 x、y 和 dropSpeed 属性，以及一个用于控制 rotation 的 acc 属性。此外，还包括一个用于控制花是否从树上落下来的 dropSpeed 属性，以及一个用于控制花飘动速度的 randomNumber 属性。

当玩家点击区域时，引擎会从 Bunny List 中随机选择一个或多个 cherry blossom，对其进行下落，并在下落过程中衰减其 Open(开)状态的速度。同时，也会根据玩家的操作(如使用方向键控制 cherry blossom 的左右移动和旋转)来调整 cherry blossom 的位置和旋转角度。

如果同一场景中 cherry blossom 数量超过了 200 个，引擎会将其中的每个 cherry blossom 移动到屏幕的左下角，并将其从场景中移除。此外，为了控制场景中 cherry blossom 数量，引擎还会在每次更新时根据 randomNumber 属性的值对 Bunny List 中的 cherry blossom 进行数量调整。

最后，该 cherry blossoms 引擎还支持使用一个樱花效果来渲染场景中的 cherry blossoms，同时，场景中的其他元素，如树和地板，也可以使用不同的效果来渲染。


```js
import * as PIXI from 'pixi.js';
import { registerPerform } from '@/Core/util/pixiPerformManager/pixiPerformManager';

import { WebGAL } from '@/Core/WebGAL';

const pixicherryBlossoms = (cherryBlossomsSpeed: number) => {
  // アニメーション パラメータ
  // 倍率を設定
  // 动画参数
  // 设置缩放的系数
  const scalePreset = 0.15;

  const effectsContainer = WebGAL!.gameplay!.pixiStage!.effectsContainer;
  const app = WebGAL!.gameplay!.pixiStage!.currentApp!;
  const container = new PIXI.Container();
  effectsContainer.addChild(container);
  // テクスチャを作成
  // 创建纹理
  const texture = PIXI.Texture.from('./game/tex/cherryBlossoms.png');
  // コンテナを中央に移動
  // 将容器移到中心
  container.x = app.screen.width / 2;
  container.y = app.screen.height / 2;
  container.pivot.x = container.width / 2;
  container.pivot.y = container.height / 2;
  // ズームを調整
  // 调整缩放
  container.scale.x = 1;
  container.scale.y = 1;
  // container.rotation = -0.2;
  const bunnyList: any = [];
  // アニメーションの更新を監視
  // 监听动画更新
  function tickerFn(delta: number) {
    // 桜の位置を制御するために使用される長さと幅を取得します
    // 获取长宽，用于控制花出现位置
    const stageWidth = 2560;
    const stageHeight = 1440;
    // オブジェクトを作成
    // 创建对象
    const bunny = new PIXI.Sprite(texture);
    let scaleRand = 0.25;

    bunny.scale.x = scalePreset * scaleRand;
    bunny.scale.y = scalePreset * scaleRand;
    // アンカーポイントを設定
    // 设置锚点
    bunny.anchor.set(0.5);
    // ランダムな桜の位置
    // 随机花位置
    bunny.x = Math.random() * stageWidth - 0.5 * stageWidth;
    bunny.y = 0 - 0.5 * stageHeight;
    // @ts-ignore
    bunny['dropSpeed'] = Math.random() * 5;
    // @ts-ignore
    bunny['acc'] = Math.random();
    container.addChild(bunny);
    bunnyList.push(bunny);

    let count = 0;
    for (const e of bunnyList) {
      count++;
      const randomNumber = Math.random();
      e['dropSpeed'] = e['acc'] * 0.01 + e['dropSpeed'];
      e.y += delta * cherryBlossomsSpeed * e['dropSpeed'] * 0.3 + 0.7;
      const addX = count % 2 === 0;
      if (addX) {
        e.x += delta * randomNumber * 0.5;
        e.rotation += delta * randomNumber * 0.03;
      } else {
        e.x -= delta * randomNumber * 0.5;
        e.rotation -= delta * randomNumber * 0.03;
      }
    }
    // 同じ画面上の桜の数を制御します
    // 控制同屏花数
    if (bunnyList.length >= 200) {
      bunnyList.unshift();
      container.removeChild(container.children[0]);
    }
  }
  WebGAL!.gameplay!.pixiStage!.registerAnimation(
    { setStartState: () => {}, setEndState: () => {}, tickerFunc: tickerFn },
    'cherryBlossoms-Ticker',
  );
  return { container, tickerKey: 'cherryBlossoms-Ticker' };
};

```

该代码创建了一个名为 "cherryBlossoms" 的注册函数，该函数接收一个参数 "3"，然后使用 "pixisherryBlossoms" 函数并传入参数 "3" 作为该函数的第二个参数，将 "pixisherryBlossoms" 函数绑定到 "cherryBlossoms" 名称上。这样，当 "cherryBlossoms" 函数被调用时，它将调用 "pixisherryBlossoms" 函数中的第三个参数（即 "3"）。


```js
registerPerform('cherryBlossoms', () => pixicherryBlossoms(3));

```

# `/packages/webgal/src/Core/gameScripts/pixiPerformScripts/rain.ts`

This is a JavaScript passage that shows how to create a simple rain effect for a game. The passage is divided into several parts that describe the different steps involved in creating the rain, such as setting up the stage, creating雪花， defining the animation, and adding the雪花 to the container.

The code uses the PIXI.js library to handle the graphics and animation. The rain animation is created using the `@keyframes` rule and the `rain-Ticker` key. The animation starts from the original position of the雪花 and adds a random drop speed to each雪花， which is determined by the animation's `tick` function.

The code also defines a `scaleRand` variable that is used to determine the scale of the雪花. This is based on the random number generated by `Math.random()` and is used to multiply the雪花的一致大小。

Overall, the code provides a good starting point for creating a simple rain effect for a game.


```js
import * as PIXI from 'pixi.js';
import { registerPerform } from '@/Core/util/pixiPerformManager/pixiPerformManager';

import { WebGAL } from '@/Core/WebGAL';

const pixiRain = (rainSpeed: number, number: number) => {
  // 动画参数
  // 设置缩放的系数
  const scalePreset = 0.48;

  const effectsContainer = WebGAL.gameplay.pixiStage!.effectsContainer!;
  const app = WebGAL.gameplay.pixiStage!.currentApp!;
  const container = new PIXI.Container();
  effectsContainer.addChild(container);
  // 创建纹理
  const texture = PIXI.Texture.from('./game/tex/raindrop.png');
  // 将容器移到中心
  container.x = app.screen.width / 2;
  container.y = app.screen.height / 2;
  container.pivot.x = container.width / 2;
  container.pivot.y = container.height / 2;
  // 调整缩放
  container.scale.x = 1;
  container.scale.y = 1;
  // container.rotation = -0.2;
  const bunnyList: any = [];
  // 监听动画更新
  function ticker(delta: number) {
    // 获取长宽，用于控制雪花出现位置
    const stageWidth = 2560;
    const stageHeight = 1440;
    for (let i = 0; i < number; i++) {
      // 创建对象
      const bunny = new PIXI.Sprite(texture);
      // 随机雨点大小
      let scaleRand = Math.random();
      if (scaleRand <= 0.5) {
        scaleRand = 0.5;
      }
      bunny.scale.x = scalePreset * scaleRand;
      bunny.scale.y = scalePreset * scaleRand;
      // 设置锚点
      bunny.anchor.set(0.5);
      // 随机雪花位置
      bunny.x = Math.random() * stageWidth - 0.5 * stageWidth;
      bunny.y = 0 - 0.5 * stageHeight;
      // @ts-ignore
      bunny['dropSpeed'] = Math.random() * 2;
      // @ts-ignore
      bunny['acc'] = Math.random();
      bunny['alpha'] = Math.random();
      if (bunny['alpha'] >= 0.5) {
        bunny['alpha'] = 0.5;
      }
      if (bunny['alpha'] <= 0.2) {
        bunny['alpha'] = 0.2;
      }
      container.addChild(bunny);
      // 控制每片雨点
      bunnyList.push(bunny);
    }
    // 雨点落下
    for (const e of bunnyList) {
      e['dropSpeed'] = e['acc'] * 0.01 + e['dropSpeed'];
      e.y += delta * rainSpeed * e['dropSpeed'] * 1.1 + 3;
    }
    // 控制同屏雨点数
    if (bunnyList.length >= 2500) {
      bunnyList.unshift();
      container.removeChild(container.children[0]);
    }
  }
  WebGAL.gameplay.pixiStage?.registerAnimation(
    { setStartState: () => {}, setEndState: () => {}, tickerFunc: ticker },
    'rain-Ticker',
  );
  return { container, tickerKey: 'rain-Ticker' };
};

```

这段代码使用了JavaScript中的registerPerform函数，它接受一个回调函数作为参数，并在该函数中执行一组代码。在这段代码中，我们传入了名为rain的函数，它使用了pixiRain函数来创建一个雨滴效果。

注册Perform函数的作用是为该函数提供一个名字，当需要执行该函数时，JavaScript引擎会自动调用该名字，并将一些参数传递给函数。在这段代码中，我们传递给了Perform函数一个雨滴参数6和10，这可能会在未来的Perform函数中被使用。


```js
registerPerform('rain', () => pixiRain(6, 10));

```

# `/packages/webgal/src/Core/gameScripts/pixiPerformScripts/snow.ts`

This is a JavaScript example that demonstrates how to create a snow scene using the PIXI.js library. The snow scene consists of several elements, including雪花、控制雪花飘落的强度、控制雪花掉落的速度、以及用于显示雪花效果的容器和动画。雪花通过一条飘落的动态路径来到达容器，并依据不同的初始位置和速度进行飘落。雪花掉落的强度和速度也是可以调整的，从而可以更好地控制雪花的效果。


```js
import * as PIXI from 'pixi.js';
import { registerPerform } from '@/Core/util/pixiPerformManager/pixiPerformManager';

import { WebGAL } from '@/Core/WebGAL';

const pixiSnow = (snowSpeed: number) => {
  // 动画参数
  // 设置缩放的系数
  const scalePreset = 0.144;

  const effectsContainer = WebGAL.gameplay.pixiStage!.effectsContainer!;
  const app = WebGAL.gameplay.pixiStage!.currentApp!;
  const container = new PIXI.Container();
  effectsContainer.addChild(container);
  // 创建纹理
  const texture = PIXI.Texture.from('./game/tex/snowFlake_min.png');
  // 将容器移到中心
  container.x = app.screen.width / 2;
  container.y = app.screen.height / 2;
  container.pivot.x = container.width / 2;
  container.pivot.y = container.height / 2;
  // 调整缩放
  container.scale.x = 1;
  container.scale.y = 1;
  // container.rotation = -0.2;
  const bunnyList: any = [];
  // 监听动画更新
  function tickerFn(delta: number) {
    // 获取长宽，用于控制雪花出现位置
    const stageWidth = 2560;
    const stageHeight = 1440;
    // 创建对象
    const bunny = new PIXI.Sprite(texture);
    // 随机雪花大小
    let scaleRand = Math.random();
    if (scaleRand <= 0.5) {
      scaleRand = 0.5;
    }
    bunny.scale.x = scalePreset * scaleRand;
    bunny.scale.y = scalePreset * scaleRand;
    // 设置锚点
    bunny.anchor.set(0.5);
    // 随机雪花位置
    bunny.x = Math.random() * stageWidth - 0.5 * stageWidth;
    bunny.y = 0 - 0.5 * stageHeight;
    // @ts-ignore
    bunny['dropSpeed'] = Math.random() * 2;
    // @ts-ignore
    bunny['acc'] = Math.random();
    container.addChild(bunny);
    // 控制每片雪花
    bunnyList.push(bunny);
    let count = 0; // 用于判断雪花往左还是往右飘，是2的倍数则往左
    for (const e of bunnyList) {
      count++;
      const randomNumber = Math.random();
      e['dropSpeed'] = e['acc'] * 0.01 + e['dropSpeed'];
      e.y += delta * snowSpeed * e['dropSpeed'] * 0.3 + 0.7;
      const addX = count % 2 === 0;
      if (addX) {
        e.x += delta * randomNumber * 0.5;
        e.rotation += delta * randomNumber * 0.03;
      } else {
        e.x -= delta * randomNumber * 0.5;
        e.rotation -= delta * randomNumber * 0.03;
      }
    }
    // 控制同屏雪花数
    if (bunnyList.length >= 500) {
      bunnyList.unshift();
      container.removeChild(container.children[0]);
    }
  }
  WebGAL.gameplay.pixiStage?.registerAnimation(
    { setStartState: () => {}, setEndState: () => {}, tickerFunc: tickerFn },
    'snow-Ticker',
  );
  return { container, tickerKey: 'snow-Ticker' };
};

```

这段代码创建了一个名为 "snow" 的 PNG 图片，并在 JavaScript 上下文中执行一个名为 "pixiSnow" 的函数。

具体来说，代码执行以下操作：

1. 将 PNG 图片 "3" 加载到内存中，并将其存储在变量 "png" 中。
2. 创建一个名为 "snow" 的画布对象，并将其绘制成包含 PNG 图片的图形。
3. 将 "png" 图片的三个通道(即红色、绿色和蓝色)转换为 PNG 图片对象，并将其绘制成雪地中白色像素的图形。
4. 将画布对象("snow")添加到上下文栈中，以便在必要时可以访问它。

最终的结果是，在代码执行完毕后，将会在内存中保存一张名为 "snow" 的 PNG 图片，它包含了一个雪地中白色像素的图形。


```js
registerPerform('snow', () => pixiSnow(3));

```

# `/packages/webgal/src/Core/Modules/animationFunctions.ts`

这段代码定义了一个名为 `getAnimationObject` 的函数，它接受两个参数：`animationName` 和 `target`，并传入一个参数 `duration`。函数的作用是：

1. 查找并获取名为 `animationName` 的动画对象。
2. 如果已找到动画对象，则检查它是否与传入的目标匹配。
3. 如果匹配，那么根据传入的 `duration` 设置动画对象的 `duration` 属性，并使用 `cloneDeep` 方法创建一个克隆副本。
4. 将克隆副本设置为原对象，并输出一个包含所有克隆副本的效果的 `WebGAL.animationManager.getState().stage.effects` 数组。
5. 如果未找到匹配的动画对象，那么返回 `null`。

该函数主要用于在 PIXI 动画中加载并使用自定义动画。它允许您创建自己的动画效果，并将其应用到目标动画对象上，从而使代码更加易于理解和维护。


```js
import { generateUniversalSoftInAnimationObj } from '@/Core/controller/stage/pixi/animations/universalSoftIn';
import { logger } from '@/Core/util/etc/logger';
import { generateUniversalSoftOffAnimationObj } from '@/Core/controller/stage/pixi/animations/universalSoftOff';
import { webgalStore } from '@/store/store';
import cloneDeep from 'lodash/cloneDeep';
import { baseTransform } from '@/store/stageInterface';
import { generateTimelineObj } from '@/Core/controller/stage/pixi/animations/timeline';
import { WebGAL } from '@/Core/WebGAL';

export function getAnimationObject(animationName: string, target: string, duration: number) {
  const effect = WebGAL.animationManager.getAnimations().find((ani) => ani.name === animationName);
  if (effect) {
    const mappedEffects = effect.effects.map((effect) => {
      const targetSetEffect = webgalStore.getState().stage.effects.find((e) => e.target === target);
      const newEffect = cloneDeep({ ...(targetSetEffect?.transform ?? baseTransform), duration: 0 });
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



This is a JavaScript function that generates animations for entering and exiting a fade-in/out transition. The function takes in an object with a `type` property that determines whether to use a fade-in or fade-out transition.

The function also takes an optional object with a `duration` property that specifies the duration of the animation, and an `isBg` property that sets whether the background should animate as well.

The function then returns an object with the `duration` and `animation` properties, which are determined by either generating a fade-in or fade-out animation based on the `type` and `isBg` properties, respectively.

The `generateUniversalSoftInAnimationObj` function generates a universal soft-in animation object, while the `getAnimationObject` function retrieves an animation object for a given animation name. The `getAnimateDuration` function returns the duration of an animation in the specified unit (e.g. milliseconds).

Overall, this function provides a simple way to generate animations for fade-in and fade-out transitions, and can be useful for generating soft-in and soft-out animations in a variety of contexts.


```js
export function getAnimateDuration(animationName: string) {
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

export function getEnterExitAnimation(
  target: string,
  type: 'enter' | 'exit',
  isBg = false,
): {
  duration: number;
  animation: {
    setStartState: () => void;
    tickerFunc: (delta: number) => void;
    setEndState: () => void;
  } | null;
} {
  if (type === 'enter') {
    let duration = 500;
    if (isBg) {
      duration = 1500;
    }
    // 走默认动画
    let animation: {
      setStartState: () => void;
      tickerFunc: (delta: number) => void;
      setEndState: () => void;
    } | null = generateUniversalSoftInAnimationObj(target, duration);
    const animarionName = WebGAL.animationManager.nextEnterAnimationName.get(target);
    if (animarionName) {
      logger.debug('取代默认进入动画', target);
      animation = getAnimationObject(animarionName, target, getAnimateDuration(animarionName));
      duration = getAnimateDuration(animarionName);
      // 用后重置
      WebGAL.animationManager.nextEnterAnimationName.delete(target);
    }
    return { duration, animation };
  } else {
    let duration = 750;
    if (isBg) {
      duration = 1500;
    }
    // 走默认动画
    let animation: {
      setStartState: () => void;
      tickerFunc: (delta: number) => void;
      setEndState: () => void;
    } | null = generateUniversalSoftOffAnimationObj(target, duration);
    const animarionName = WebGAL.animationManager.nextExitAnimationName.get(target);
    if (animarionName) {
      logger.debug('取代默认退出动画', target);
      animation = getAnimationObject(animarionName, target, getAnimateDuration(animarionName));
      duration = getAnimateDuration(animarionName);
      // 用后重置
      WebGAL.animationManager.nextExitAnimationName.delete(target);
    }
    return { duration, animation };
  }
}

```

# `/packages/webgal/src/Core/Modules/animations.ts`

这段代码定义了一个名为 `AnimationManager` 的类，该类用于实现用户动画效果。

首先，它导入了 `ISTransform` 接口，该接口定义了动画的数据结构，然后定义了一个名为 `IUserAnimation` 的接口，该接口定义了用户动画的数据结构，包括动画的名称和效果等。

接着，该类创建了两个名为 `nextEnterAnimationName` 和 `nextExitAnimationName` 的 `Map` 对象，用于存储待 enter 和 exit 的动画名称，每个名称都是一个字符串类型的键值对。

然后，该类创建了一个名为 `animations` 的数组，用于存储所有的用户动画。

接下来，该类实现了 `addAnimation` 方法，用于添加一个用户动画，该方法接收一个 `IUserAnimation` 类型的参数，将其添加到 `animations` 数组中。

最后，该类实现了 `getAnimations` 方法，用于返回所有的用户动画，该方法返回一个数组，该数组包含所有已添加的用户动画。


```js
import { ITransform } from '@/store/stageInterface';

export interface IUserAnimation {
  name: string;
  effects: Array<ITransform & { duration: number }>;
}

export class AnimationManager {
  public nextEnterAnimationName: Map<string, string> = new Map();
  public nextExitAnimationName: Map<string, string> = new Map();
  private animations: Array<IUserAnimation> = [];

  public addAnimation(animation: IUserAnimation) {
    this.animations.push(animation);
  }
  public getAnimations() {
    return this.animations;
  }
}

```

# `/packages/webgal/src/Core/Modules/backlog.ts`

此代码定义了一个名为IBacklogItem的接口，它表示一个阶段的备份状态。它还引入了来自不同存储库的对象，如IStageState，ISaveScene和cloneDeep。

IBacklogItem组件有一个currentStageState属性和一个saveScene属性，这些属性都来自IStageState对象。这个对象在当前备份状态下保存了该场景的所有更改，包括场景的元数据、场景中的对象和属性、以及场景的布局。它还包含一个saveScene方法，用于将场景保存到文件中。

此外，此代码引入了一个名为cloneDeep的函数，该函数用于从其父对象中克隆一个深层次的对象。这个函数将在saveScene方法中用于复制整个场景到备份文件中。


```js
/**
 * 当前的backlog
 */
import { IStageState } from '@/store/stageInterface';
import { webgalStore } from '@/store/store';
import { ISaveScene } from '@/store/userDataInterface';
import cloneDeep from 'lodash/cloneDeep';

import { SYSTEM_CONFIG } from '@/Core/config/config';
import { SceneManager } from '@/Core/Modules/scene';

export interface IBacklogItem {
  currentStageState: IStageState;
  saveScene: ISaveScene;
}

```

This is a TypeScript class that wraps a `BacklogManager` class that manages the backup of a game's state at different stages. The `BacklogManager` class has several private methods, including `isSaveBacklogNext`, `backlog`, `makeBacklogEmpty`, `insertBacklogItem`, and `saveCurrentStateToBacklog`.

The `isSaveBacklogNext` property is a boolean that determines whether to save a backup at the next frame or not. The `backlog` property is an array of `IBacklogItem` objects that represent the items to be saved in the backup. The `makeBacklogEmpty` method is used to clear the `backlog` array by removing any existing elements and returning an empty array. The `insertBacklogItem` method is used to add a new element to the `backlog` array.

The `saveCurrentStateToBacklog` method is used to save the current state of the game to the backup. It creates a new `BacklogElement` object that contains the current state of the game and tells the `getBacklog` method to push it onto the `backlog` array. The `getBacklog` method is used to retrieve the existing elements of the `backlog` array and clear any elements that have already been added.


```js
export class BacklogManager {
  public isSaveBacklogNext = false;
  private backlog: Array<IBacklogItem> = [];

  private readonly sceneManager: SceneManager;

  public constructor(sceneManager: SceneManager) {
    this.sceneManager = sceneManager;
  }

  public getBacklog() {
    return this.backlog;
  }
  public makeBacklogEmpty() {
    this.backlog.splice(0, this.backlog.length); // 清空backlog
  }
  public insertBacklogItem(item: IBacklogItem) {
    this.backlog.push(item);
  }
  public saveCurrentStateToBacklog() {
    // 存一下 Backlog
    const currentStageState = webgalStore.getState().stage;
    const stageStateToBacklog = cloneDeep(currentStageState);
    stageStateToBacklog.PerformList.forEach((ele) => {
      ele.script.args.forEach((argelement) => {
        if (argelement.key === 'concat') {
          argelement.value = false;
          ele.script.content = stageStateToBacklog.showText;
        }
      });
    });
    const backlogElement: IBacklogItem = {
      currentStageState: stageStateToBacklog,
      saveScene: {
        currentSentenceId: this.sceneManager.sceneData.currentSentenceId, // 当前语句ID
        sceneStack: cloneDeep(this.sceneManager.sceneData.sceneStack), // 场景栈
        sceneName: this.sceneManager.sceneData.currentScene.sceneName, // 场景名称
        sceneUrl: this.sceneManager.sceneData.currentScene.sceneUrl, // 场景url
      },
    };
    this.getBacklog().push(backlogElement);

    // 清除超出长度的部分
    while (this.getBacklog().length > SYSTEM_CONFIG.backlog_size) {
      this.getBacklog().shift();
    }
  }
}

```

# `/packages/webgal/src/Core/Modules/gamePlay.ts`

这段代码定义了一个名为`Gameplay`的类，用于控制游戏的速度和定时器。

* `isAuto`是一个布尔值，表示游戏是否自动控制速度，初始值为false。
* `isFast`也是一个布尔值，表示游戏是否使用快速模式，初始值为false。
* `autoInterval`是一个返回类型为`setInterval`的函数，用于设置自动控制速度的定时器。初始值为`null`。
* `fastInterval`是一个返回类型为`setInterval`的函数，用于设置快速模式下的定时器。初始值为`null`。
* `autoTimeout`是一个返回类型为`setTimeout`的函数，用于设置自动控制速度的超时时间。初始值为`null`。
* `pixiStage`是一个返回类型为`PixiStage`的函数，用于获取游戏舞台实例。初始值为`null`。
* `performController`是一个引用，指向一个名为`PerformController`的类实例。
* `resetGamePlay`是一个方法，用于重置游戏玩法，该方法会清除所有定时器，并设置游戏为自动模式。


```js
import PixiStage from '@/Core/controller/stage/pixi/PixiController';
import { PerformController } from '@/Core/Modules/perform/performController';

/**
 * 游戏运行时变量
 */
export class Gameplay {
  public isAuto = false;
  public isFast = false;
  public autoInterval: ReturnType<typeof setInterval> | null = null;
  public fastInterval: ReturnType<typeof setInterval> | null = null;
  public autoTimeout: ReturnType<typeof setTimeout> | null = null;
  public pixiStage: PixiStage | null = null;
  public performController = new PerformController();
  public resetGamePlay() {
    this.performController.timeoutList = [];
    this.isAuto = false;
    this.isFast = false;
    const autoInterval = this.autoInterval;
    if (autoInterval !== null) clearInterval(autoInterval);
    this.autoInterval = null;
    const fastInterval = this.fastInterval;
    if (fastInterval !== null) clearInterval(fastInterval);
    this.fastInterval = null;
    const autoTimeout = this.autoTimeout;
    if (autoTimeout !== null) clearInterval(autoTimeout);
    this.autoTimeout = null;
  }
}

```

# `/packages/webgal/src/Core/Modules/scene.ts`

这段代码主要作用是初始化一个场景数据对象（ISceneData），并为每个场景分配一个空场景对象。

具体来说，代码首先导入了来自`@/Core/controller/scene/sceneInterface`的一个名为`ISceneData`的接口类型。接着，引入了`lodash/cloneDeep`的`cloneDeep`函数，用于复制场景对象以确保深层次的子对象也得以复制。

然后，定义了一个名为`ISceneEntry`的接口类型，包含场景名称、场景URL和继续原场景的行号。

接下来，定义了一个`initSceneData`函数，该函数接受一个包含初始场景数据的对象。函数内部，使用了嵌套的`set`函数，将`ISceneData`接口的各个属性都设置为初始值。最后，返回`initSceneData`函数的值。


```js
import { ISceneData } from '@/Core/controller/scene/sceneInterface';
import cloneDeep from 'lodash/cloneDeep';

export interface ISceneEntry {
  sceneName: string; // 场景名称
  sceneUrl: string; // 场景url
  continueLine: number; // 继续原场景的行号
}

/**
 * 初始化场景数据
 */
export const initSceneData = {
  currentSentenceId: 0, // 当前语句ID
  sceneStack: [],
  // 初始场景，没有数据
  currentScene: {
    sceneName: '', // 场景名称
    sceneUrl: '', // 场景url
    sentenceList: [], // 语句列表
    assetsList: [], // 资源列表
    subSceneList: [], // 子场景列表
  },
};

```

这段代码定义了一个名为 `SceneManager` 的类，用于管理场景数据。这个类有以下几个属性和方法：

1. `settledScenes`：场景的定居数据，是一个数组，用于存储已经定居的场景名称。
2. `settledAssets`：场景的定居数据，是一个数组，用于存储已经定居的资产名称。
3. `sceneData`：场景的数据，是一个实现了 `ISceneData` 接口的对象，用于存储场景的数据。这个对象可能是通过调用 `initSceneData` 方法生成的，或者是在场景加载时从磁盘或其他来源加载得到的。
4. `resetScene`：重置场景的方法，这个方法会被场景加载时调用，然后将场景数据设置为初始状态。具体来说，会将 `sceneData.currentSentenceId` 设置为 0，将 `sceneData.sceneStack` 设置为空数组，将 `sceneData.currentScene` 设置为初始场景的克隆。

场景加载时，可以使用 `SceneManager` 类来加载场景数据。比如，可以这样加载场景：

```js
const sceneManager = new SceneManager();
sceneManager.resetScene();
const scene = sceneManager.sceneData;
```

在这个例子中，`SceneManager` 类调用 `resetScene` 方法来重置场景，然后场景数据被设置为初始状态。接着，场景经理使用 `sceneData` 对象来获取场景，并将其存储在一个变量中。


```js
export class SceneManager {
  public settledScenes: Array<string> = [];
  public settledAssets: Array<string> = [];
  public sceneData: ISceneData = cloneDeep(initSceneData);

  public resetScene() {
    this.sceneData.currentSentenceId = 0;
    this.sceneData.sceneStack = [];
    this.sceneData.currentScene = cloneDeep(initSceneData.currentScene);
  }
}

```

# `/packages/webgal/src/Core/Modules/perform/performController.ts`

这段代码包含了一个名为 `getRandomPerformName` 的函数，它使用 `Math.random()` 库从指定位置随机提取一个字符串，这个字符串长度为10个字符。它最终返回生成的随机演出名称，并将其返回。

该函数的作用是用于生成一个用于表示随机演出的名称，例如演出名称、艺术家名称等。它将被用于随机演出系统中，当需要随机生成演出名称时，将从演出列表中选择一个演出，并在演出名称中包含随机生成的名称。

此外，该段代码还引入了几个其他模块和函数：

- `IPerform`：用于在演出过程中执行的业务逻辑。
- `ISentence`：用于在场景中创建和操作语句的类。
- `webgalStore`：用于在 Vuex 中存储演出列表的数据的模块。
- `cloneDeep`:`lodash` 库中的一个方法，用于对对象进行克隆并递归地复制所有的属性。
- `resetStageState`：用于在演出开始时重置舞台状态的函数。
- `nextSentence`：用于在游戏中下一句台词的类。
- `IRunPerform`：用于在演出过程中执行的业务逻辑的类。

由于该函数没有其他的作用，因此它的作用就是生成一个随机的演出名称，并将其返回。


```js
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { webgalStore } from '@/store/store';
import cloneDeep from 'lodash/cloneDeep';
import { resetStageState } from '@/store/stageReducer';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';
import { IRunPerform } from '@/store/stageInterface';

/**
 * 获取随机演出名称
 */
export const getRandomPerformName = (): string => {
  return Math.random().toString().substring(0, 10);
};

```

This is a JavaScript class called `PerformManager`. It appears to be managing a list of演出， known as `PerformList`, and also has a list of timeout delays, known as `timeoutList`. It performs two main functions: `startPerform` and `endPerform`.

`startPerform` is used to start a new演出。 It takes a `name` argument and performs the演出。 If the演出被暂停（通过调用 `stopPerform` 或 `blockingAuto` 方法），会继续等待 100ms 后继续执行。

`endPerform` 是演出结束后的回调函数。它会遍历 `PerformList`，检查每个演出是否被阻塞（即 `blockingAuto` 状态为 `true`）。如果是，则清除阻塞并继续等待 100ms。否则，恢复演出状态。

此外，还有一个 `removeAllPerform` 方法，用于移除所有演出。还有一个 `getTime` 方法，用于获取演出开始和结束的时间戳。


```js
export class PerformController {
  public performList: Array<IPerform> = [];
  public timeoutList: Array<ReturnType<typeof setTimeout>> = [];

  public arrangeNewPerform(perform: IPerform, script: ISentence, syncPerformState = true) {
    // 语句不执行演出
    if (perform.performName === 'none') {
      return;
    }
    // 同步演出状态
    if (syncPerformState) {
      const stageState = webgalStore.getState().stage;
      const newStageState = cloneDeep(stageState);
      newStageState.PerformList.push({ id: perform.performName, isHoldOn: perform.isHoldOn, script: script });
      webgalStore.dispatch(resetStageState(newStageState));
    }

    // 时间到后自动清理演出
    perform.stopTimeout = setTimeout(() => {
      // perform.stopFunction();
      // perform.isOver = true;
      if (!perform.isHoldOn) {
        // 如果不是保持演出，清除
        this.unmountPerform(perform.performName);
        if (perform.goNextWhenOver) {
          // nextSentence();
          this.goNextWhenOver();
        }
      }
    }, perform.duration);

    this.performList.push(perform);
  }

  public unmountPerform(name: string, force = false) {
    if (!force) {
      for (let i = 0; i < this.performList.length; i++) {
        const e = this.performList[i];
        if (!e.isHoldOn && e.performName === name) {
          e.stopFunction();
          clearTimeout(e.stopTimeout as unknown as number);
          this.performList.splice(i, 1);
          i--;
        }
      }
    } else {
      for (let i = 0; i < this.performList.length; i++) {
        const e = this.performList[i];
        if (e.performName === name) {
          e.stopFunction();
          clearTimeout(e.stopTimeout as unknown as number);
          this.performList.splice(i, 1);
          i--;
          /**
           * 从状态表里清除演出
           */
          this.erasePerformFromState(name);
        }
      }
    }
  }

  public erasePerformFromState(name: string) {
    const stageState = webgalStore.getState().stage;
    const newStageState = cloneDeep(stageState);
    for (let i = 0; i < newStageState.PerformList.length; i++) {
      const e2: IRunPerform = newStageState.PerformList[i];
      if (e2.id === name) {
        newStageState.PerformList.splice(i, 1);
        i--;
      }
    }
    webgalStore.dispatch(resetStageState(newStageState));
  }

  public removeAllPerform() {
    for (const e of this.performList) {
      e.stopFunction();
    }
    this.performList = [];
    for (const e of this.timeoutList) {
      clearTimeout(e);
    }
  }

  private goNextWhenOver() {
    let isBlockingAuto = false;
    this.performList.forEach((e) => {
      if (e.blockingAuto())
        // 阻塞且没有结束的演出
        isBlockingAuto = true;
    });
    if (isBlockingAuto) {
      // 有阻塞，提前结束
      setTimeout(this.goNextWhenOver, 100);
    } else {
      nextSentence();
    }
  }
}

```

# `/packages/webgal/src/Core/Modules/perform/performInterface.ts`

以上代码定义了一个名为 IPerform 的接口，用于演出控制。该接口包含以下属性：

- performName: 演出名称，可以是字符串，也可以是包含特殊字符组成的字符串，用于在演出开始时手动清除演出，如果没有标识，则代表不是保持演出，给予一个随机字符串。
- duration: 演出持续时间，单位为毫秒，持续时间到后自动回收演出。
- isHoldOn: 演出是否保持类型。
- stopFunction: 卸载演出的函数。
- blockingNext: 用于判断是否要阻塞游戏流程 nextSentence 的结果，返回一个布尔值。
- blockingAuto: 用于判断是否要阻塞自动模式，返回一个布尔值。
- stopTimeout: 若演出需要停止，设置的 timeout 时间，会优先于 goNextWhenOver。
- goNextWhenOver: 当自动模式下一次演出开始时，是否继续演出。
- arrangePerformPromise: 对于延迟触发的演出，使用 Promise，但并未定义具体的 Promise 实现。
- skipNextCollect: 用于判断是否允许延迟演出，返回一个布尔值。

该接口的定义对于演出控制具有一种统一的方式，使得在控制演出时，不同的演出可以采用统一的接口进行控制。


```js
/**
 * 描述演出的接口，主要用于控制演出，而不是执行（在演出开始时被调用演出的执行器返回）
 * @interface IPerform
 */
export interface IPerform {
  // 演出名称，用于在后面手动清除演出，如果没有标识，则代表不是保持演出，给予一个随机字符串
  performName: string;
  // 持续时间，单位为ms，持续时间到后自动回收演出
  duration: number;
  // 演出是不是一个保持类型的演出
  isHoldOn: boolean;
  // 卸载演出的函数
  stopFunction: () => void;
  // 演出是否阻塞游戏流程继续（一个函数，返回 boolean类型的结果，判断要不要阻塞）
  blockingNext: () => boolean;
  // 演出是否阻塞自动模式（一个函数，返回 boolean类型的结果，判断要不要阻塞）
  blockingAuto: () => boolean;
  // 自动回收使用的 Timeout
  stopTimeout: undefined | ReturnType<typeof setTimeout>;
  // 演出结束后转到下一句
  goNextWhenOver?: boolean;
  // 对于延迟触发的演出，使用 Promise
  arrangePerformPromise?: Promise<IPerform>;
  // 跳过由 nextSentence 函数引发的演出回收
  skipNextCollect?: boolean;
}

```

这段代码定义了一个演出列表的数据结构以及一些演出相关的信息。

首先，定义了一个名为perform的演出对象，包含演出名称、持续时间、当前状态(isOver 和 isHoldOn)、停止函数以及阻塞下一个演出的函数(blockingNext 和 blockingAuto)和使用变量stopFunction来存储停止演出时需要执行的函数。

其次，定义了一个名为initPerform的演出列表的初始化函数，其中初始化了演出列表的所有对象。

initPerform的作用是初始化演出列表，根据函数内部设置的值，如果当前演出列表仅包含不阻塞自动的演出，则允许下一个演出自动开始。否则，所有演出都会被阻塞，当前演出列表也不会响应blockingNext和blockingAuto函数。当所有的演出都完成时，演出列表也会被移出。


```js
// next之后，可以被打断的演出会被打断，不能被打断的演出会继续，阻塞next的演出会阻止next被响应。
// 被打断或执行完毕的演出会移出演出列表
// 只有所有演出都完成，或者仅存在不阻塞auto的演出，才允许auto

/**
 * 初始化的演出
 */
export const initPerform: IPerform = {
  performName: '',
  duration: 100,
  // isOver: false,
  isHoldOn: false,
  stopFunction: () => {},
  blockingNext: () => false,
  blockingAuto: () => true,
  stopTimeout: undefined,
};

```

# `/packages/webgal/src/Core/parser/sceneParser.ts`

这段代码的作用是执行一系列游戏场景操作，包括从控制器层、游戏逻辑层、特效层等不同方面改动游戏场景。

具体来说，这段代码会执行以下操作：

1. 导入`IScene`、`logger`、`assetsPrefetcher`、`assetSetter`、`SceneParser`。这些库和模块可能会在游戏开发中用到，具体作用不明。

2. 导入`commandType`、`ISentence`。这些数据类型可能与游戏中的命令和对话有关，具体作用也不明。

3. 导入`intro`、`changeBg`、`changeFigure`、`miniAvatar`、`changeSceneScript`、`choose`、`end`、`bgm`。这些函数或类可能与游戏中的界面、特效、角色等有关，具体作用也不明。

4. 从`@/Core/util/prefetcher/assetsPrefetcher`中载入游戏资源。

5. 从`@/Core/util/gameAssetsAccess/assetSetter`中载入游戏资源。

6. 解析游戏场景，可能是从`webgal-parser`库中进行解析。

7. 根据需要，执行`intro`函数，初始化游戏界面。

8. 执行`changeBg`函数，改变游戏背景。

9. 执行`changeFigure`函数，改变游戏中的角色或场景等。

10. 如果选择了某个场景，使用`changeSceneScript`函数，改变场景脚本。

11. 如果需要，使用`miniAvatar`函数，改变游戏中的角色造型。

12. 执行`changeFigure`函数，改变游戏中的场景。

13. 执行`changeSceneScript`函数，改变场景脚本。

14. 执行`choose`函数，可能是在选择场景。

15. 执行`end`函数，关闭游戏窗口。

16. 执行`bgm`函数，开启游戏背景音乐。

17. 在游戏过程中，使用`logger`函数，输出日志信息。

这段代码的具体作用和使用场景可能会根据游戏开发的需求而有所不同。


```js
import { IScene } from '../controller/scene/sceneInterface';
import { logger } from '../util/etc/logger';
import { assetsPrefetcher } from '@/Core/util/prefetcher/assetsPrefetcher';
import { assetSetter } from '@/Core/util/gameAssetsAccess/assetSetter';
import SceneParser from 'webgal-parser';

import { commandType, ISentence } from '@/Core/controller/scene/sceneInterface';
import { intro } from '@/Core/gameScripts/intro';
import { changeBg } from '@/Core/gameScripts/changeBg';
import { changeFigure } from '@/Core/gameScripts/changeFigure';
import { miniAvatar } from '@/Core/gameScripts/miniAvatar';
import { changeSceneScript } from '@/Core/gameScripts/changeSceneScript';
import { choose } from '@/Core/gameScripts/choose';
import { end } from '../gameScripts/end';
import { bgm } from '@/Core/gameScripts/bgm';
```

这段代码的作用是引入了多个游戏脚本中的类和函数，包括 playVideo、setComplexAnimation、setFilter、pixiInit、pixi、label、jumpLabel、setVar、showVars、unlockCg、unlockBgm、say 和 filmMode，以及 callSceneScript 和 setTextbox。这些类和函数可以用于游戏中播放视频、设置复杂的动画、过滤屏幕、设置文本框、解锁背景音乐和电影模式等功能。具体来说，这些代码可以被用来开发一个游戏，或者将现有的游戏进行优化和改进。


```js
import { playVideo } from '@/Core/gameScripts/playVideo';
import { setComplexAnimation } from '@/Core/gameScripts/setComplexAnimation';
import { setFilter } from '@/Core/gameScripts/setFilter';
import { pixiInit } from '../gameScripts/pixiInit';
import { pixi } from '@/Core/gameScripts/pixi';
import { label } from '@/Core/gameScripts/label';
import { jumpLabel } from '../gameScripts/jumpLabel';
import { setVar } from '../gameScripts/setVar';
import { showVars } from '../gameScripts/showVars';
import { unlockCg } from '@/Core/gameScripts/unlockCg';
import { unlockBgm } from '@/Core/gameScripts/unlockBgm';
import { say } from '../gameScripts/say';
import { filmMode } from '@/Core/gameScripts/filmMode';
import { callSceneScript } from '@/Core/gameScripts/callSceneScript';
import { setTextbox } from '@/Core/gameScripts/setTextbox';
```

It appears that you are a script for a game or application that generates a JavaScript file that contains scripts for various commands that the game or application can use. The scripts are written in a language that appears to be similar to JavaScript, but with some differences in syntax and function naming conventions.

The scripts are organized into different categories, such as "commands" and "utilities," and each category contains a set of scripts with a specific function or script type. For example, the "commands" category contains scripts for actions like "say" and "unlockCg," while the "utilities" category contains scripts for things like "setTransform" and "setAnimation."

It's important to note that these scripts are generated dynamically based on some kind of input or selection criteria, and each script may have different effects or behaviors depending on how it's used.


```js
import { setAnimation } from '@/Core/gameScripts/setAnimation';
import { playEffect } from '@/Core/gameScripts/playEffect';
import { setTempAnimation } from '@/Core/gameScripts/setTempAnimation';
import { comment } from '@/Core/gameScripts/comment';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { setTransform } from '@/Core/gameScripts/setTransform';
import { setTransition } from '@/Core/gameScripts/setTransition';

interface IConfigInterface {
  scriptString: string;
  scriptType: commandType;
  scriptFunction: (sentence: ISentence) => IPerform;
}

export const SCRIPT_CONFIG: IConfigInterface[] = [
  { scriptString: 'intro', scriptType: commandType.intro, scriptFunction: intro },
  { scriptString: 'changeBg', scriptType: commandType.changeBg, scriptFunction: changeBg },
  { scriptString: 'changeFigure', scriptType: commandType.changeFigure, scriptFunction: changeFigure },
  { scriptString: 'miniAvatar', scriptType: commandType.miniAvatar, scriptFunction: miniAvatar },
  { scriptString: 'changeScene', scriptType: commandType.changeScene, scriptFunction: changeSceneScript },
  { scriptString: 'choose', scriptType: commandType.choose, scriptFunction: choose },
  { scriptString: 'end', scriptType: commandType.end, scriptFunction: end },
  { scriptString: 'bgm', scriptType: commandType.bgm, scriptFunction: bgm },
  { scriptString: 'playVideo', scriptType: commandType.video, scriptFunction: playVideo },
  {
    scriptString: 'setComplexAnimation',
    scriptType: commandType.setComplexAnimation,
    scriptFunction: setComplexAnimation,
  },
  { scriptString: 'setFilter', scriptType: commandType.setFilter, scriptFunction: setFilter },
  { scriptString: 'pixiInit', scriptType: commandType.pixiInit, scriptFunction: pixiInit },
  { scriptString: 'pixiPerform', scriptType: commandType.pixi, scriptFunction: pixi },
  { scriptString: 'label', scriptType: commandType.label, scriptFunction: label },
  { scriptString: 'jumpLabel', scriptType: commandType.jumpLabel, scriptFunction: jumpLabel },
  { scriptString: 'setVar', scriptType: commandType.setVar, scriptFunction: setVar },
  { scriptString: 'callScene', scriptType: commandType.callScene, scriptFunction: changeSceneScript },
  { scriptString: 'showVars', scriptType: commandType.showVars, scriptFunction: showVars },
  { scriptString: 'unlockCg', scriptType: commandType.unlockCg, scriptFunction: unlockCg },
  { scriptString: 'unlockBgm', scriptType: commandType.unlockBgm, scriptFunction: unlockBgm },
  { scriptString: 'say', scriptType: commandType.say, scriptFunction: say },
  { scriptString: 'filmMode', scriptType: commandType.filmMode, scriptFunction: filmMode },
  { scriptString: 'callScene', scriptType: commandType.callScene, scriptFunction: callSceneScript },
  { scriptString: 'setTextbox', scriptType: commandType.setTextbox, scriptFunction: setTextbox },
  { scriptString: 'setAnimation', scriptType: commandType.setAnimation, scriptFunction: setAnimation },
  { scriptString: 'playEffect', scriptType: commandType.playEffect, scriptFunction: playEffect },
  { scriptString: 'setTempAnimation', scriptType: commandType.setTempAnimation, scriptFunction: setTempAnimation },
  { scriptString: '__commment', scriptType: commandType.comment, scriptFunction: comment },
  { scriptString: 'setTransform', scriptType: commandType.setTransform, scriptFunction: setTransform },
  { scriptString: 'setTransition', scriptType: commandType.setTransition, scriptFunction: setTransition },
];
```

以上代码定义了一个名为 `ADD_NEXT_ARG_LIST` 的数组，该数组包含了 13 个命令类型。这些命令类型包括：

- `commandType.bgm`：用于在游戏启动时加载一个背景音乐。
- `commandType.pixi`：用于在 PIXI 库中绘制图形。
- `commandType.pixiInit`：用于初始化 PIXI 库。
- `commandType.label`：用于在游戏界面上显示标签。
- `commandType.if`：用于分支到条件代码块。
- `commandType.miniAvatar`：用于在游戏界面上显示小型 Avatar。
- `commandType.setVar`：用于在游戏中设置变量。
- `commandType.unlockBgm`：用于解锁背景音乐。
- `commandType.unlockCg`：用于解锁动画效果。
- `commandType.filmMode`：用于设置游戏模式。
- `commandType.playEffect`：用于在游戏中添加特效。
- `commandType.comment`：用于显示游戏中的注释。
- `commandType.setTransition`：用于设置游戏中的平滑过渡。

这些命令类型可以用于开发各种游戏或应用程序。


```js
export const ADD_NEXT_ARG_LIST = [
  commandType.bgm,
  commandType.pixi,
  commandType.pixiInit,
  commandType.label,
  commandType.if,
  commandType.miniAvatar,
  commandType.setVar,
  commandType.unlockBgm,
  commandType.unlockCg,
  commandType.filmMode,
  commandType.playEffect,
  commandType.comment,
  commandType.setTransition,
];

```

这段代码定义了一个名为 `WebgalParser` 的函数，它接受一个 `rawScene`、一个 `sceneName` 和一个 `sceneUrl` 参数。这个函数使用 `SceneParser` 类来解析场景，并将解析结果返回。

`SceneParser` 类接受一个 `assetsPrefetcher`、一个 `assetSetter` 和一个 `ADD_NEXT_ARG_LIST` 参数。这些参数将在解析场景时使用，具体作用不明。

函数内部，首先调用 `WebgalParser.parse` 方法，将 `rawScene`、`sceneName` 和 `sceneUrl` 传入，得到一个 `parsedScene` 变量。然后，调用 `logger.info` 函数，输出 `parsedScene` 的信息，包括场景名称和解析结果。最后，返回 `parsedScene`，即解析后的场景。

整个函数的作用是定义了一个名为 `WebgalParser` 的函数，接受三个参数，并使用 `SceneParser` 类来解析场景。返回解析后的场景。


```js
/**
 * 场景解析器
 * @param rawScene 原始场景
 * @param sceneName 场景名称
 * @param sceneUrl 场景url
 * @return {IScene} 解析后的场景
 */
export const WebgalParser = new SceneParser(assetsPrefetcher, assetSetter, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

export const sceneParser = (rawScene: string, sceneName: string, sceneUrl: string): IScene => {
  const parsedScene = WebgalParser.parse(rawScene, sceneName, sceneUrl);
  logger.info(`解析场景：${sceneName}，数据为：`, parsedScene);
  return parsedScene;
};

```