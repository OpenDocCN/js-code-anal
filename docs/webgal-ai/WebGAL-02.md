# WebGal源码解析 2

# `/packages/webgal/src/Components/Stage/MainStage/useSetEffects.ts`

这段代码定义了两个函数，一个是 `setStageObjectEffects`，另一个是 `setStageEffects`。这两个函数都是用来设置游戏中的舞台效果。

`setStageObjectEffects` 函数接收一个 `IStageState` 对象，这个对象包含游戏中的所有舞台对象和它们所产生的特效。这个函数的作用是设置舞台对象和它们所产生的特效。它通过调用 `setTimeout(() => {`, 10) 来延迟执行，这会在 10 毫秒之后执行 `setStageEffects` 函数。

`setStageEffects` 函数接收一个 `IEffect` 数组，这个数组包含游戏中的所有已应用过的特效。这个函数的作用是设置游戏中的舞台和它们所产生的特效。它通过调用 `setTimeout(() => {`, 10) 来延迟执行，这会在 10 毫秒之后执行 `setStageObjectEffects` 函数。

这两个函数都有使用 `WebGAL` 库，它是一个用于游戏开发的库，提供了对游戏中的对象、地图等对象的操作。


```js
import { baseTransform, IEffect, IStageState, ITransform } from '@/store/stageInterface';

import { WebGAL } from '@/Core/WebGAL';

export function setStageObjectEffects(stageState: IStageState) {
  const effects = stageState.effects;
  setTimeout(() => {
    setStageEffects(effects);
  }, 10);
}

export function setStageEffects(effects: IEffect[]) {
  const stageObjects = WebGAL.gameplay.pixiStage?.getAllStageObj() ?? [];
  for (const stageObj of stageObjects) {
    const key = stageObj.key;
    const effect = effects.find((effect) => effect.target === key);
    const lockedStageTargets = WebGAL.gameplay.pixiStage?.getAllLockedObject() ?? [];
    if (!lockedStageTargets.includes(key)) {
      if (effect) {
        // logger.debug('应用effects', key);
        const targetPixiContainer = WebGAL.gameplay.pixiStage?.getStageObjByKey(key);
        if (targetPixiContainer) {
          const container = targetPixiContainer.pixiContainer;
          Object.assign(container, convertTransform(effect.transform));
        }
      } else {
        const targetPixiContainer = WebGAL.gameplay.pixiStage?.getStageObjByKey(key);
        if (targetPixiContainer) {
          const container = targetPixiContainer.pixiContainer;
          Object.assign(container, convertTransform(baseTransform));
        }
      }
    }
  }
}

```

这段代码定义了一个名为 `convertTransform` 的函数，它接受一个 `ISTRANSFORM` 类型的参数 `transform`。函数的作用是将传入的 `transform` 对象进行转换，如果输入参数 `transform` 为 `undefined`，则返回一个空对象 `{}`。

如果 `transform` 对象不为 `undefined`，则函数会对 `transform` 进行解构，获取其 `position` 属性的 `x` 和 `y` 坐标，并将它们添加到新的 `rest` 对象中(原本 `transform` 对象中没有 `x` 和 `y` 属性)，最终返回新的 `transform` 对象。

通俗来说，这段代码的作用是将一个 `ISTRANSFORM` 类型的对象 `transform` 转换成一个新的 `transform` 对象，其中新对象包含了 `transform` 的 `position` 属性(即 `x` 和 `y` 坐标)。


```js
function convertTransform(transform: ITransform | undefined) {
  if (!transform) {
    return {};
  }
  const { position, pivot, ...rest } = transform;
  return { ...rest, x: position.x, y: position.y };
}

```

# `/packages/webgal/src/Components/Stage/MainStage/useSetFigure.ts`



This is a JavaScript code that creates a 2D animation in a Live2D game. It uses the Live2D Motion API to create animations and注册它们到场景中。同时，它还处理了一些特殊情况，如进入和退出场景时调用进出场动画，以及移除不在状态表中的立绘。

具体来说，代码首先根据传入的figKey查找相应的场景和图形，并创建一个 entering 动画。接着，它会判断当前图形是否存在于预设动画中，如果是，则执行进入动画；如果不是，则进入退出动画。在进入退出动画执行完成后，代码会检查当前图形是否存在于状态表中，如果是，则移除它；如果不是，则创建一个进入/退出场景动画，并将该图形加入状态表中。

此外，代码还实现了一个 removeFig 函数，用于移除场景中的单个图形，该函数会根据图形的关键帧信息在场景中查找并在必要时移除它。


```js
import { IEffect, IStageState } from '@/store/stageInterface';
import { useEffect } from 'react';
import { logger } from '@/Core/util/etc/logger';
import { generateUniversalSoftInAnimationObj } from '@/Core/controller/stage/pixi/animations/universalSoftIn';
import { IStageObject } from '@/Core/controller/stage/pixi/PixiController';
import { generateUniversalSoftOffAnimationObj } from '@/Core/controller/stage/pixi/animations/universalSoftOff';

import { getEnterExitAnimation } from '@/Core/Modules/animationFunctions';
import { WebGAL } from '@/Core/WebGAL';

export function useSetFigure(stageState: IStageState) {
  const { figNameLeft, figName, figNameRight, freeFigure, live2dMotion } = stageState;

  /**
   * 设置立绘
   */
  useEffect(() => {
    /**
     * 特殊处理：中间立绘
     */
    const thisFigKey = 'fig-center';
    const softInAniKey = 'fig-center-softin';
    if (figName !== '') {
      const currentFigCenter = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
      if (currentFigCenter) {
        if (currentFigCenter.sourceUrl !== figName) {
          removeFig(currentFigCenter, softInAniKey, stageState.effects);
        }
      }
      WebGAL.gameplay.pixiStage?.addFigure(thisFigKey, figName, 'center');
      const regex = /.json$/;
      if (regex.test(figName)) {
        addLive2dFigure(thisFigKey, figName, 'center', live2dMotion.find((e) => e.target === thisFigKey)?.motion ?? '');
      }
      logger.debug('中立绘已重设');
      const { duration, animation } = getEnterExitAnimation(thisFigKey, 'enter');
      WebGAL.gameplay.pixiStage!.registerPresetAnimation(animation, softInAniKey, thisFigKey, stageState.effects);
      setTimeout(() => WebGAL.gameplay.pixiStage!.removeAnimationWithSetEffects(softInAniKey), duration);
    } else {
      logger.debug('移除中立绘');
      const currentFigCenter = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
      if (currentFigCenter) {
        if (currentFigCenter.sourceUrl !== figName) {
          removeFig(currentFigCenter, softInAniKey, stageState.effects);
        }
      }
    }
  }, [figName]);

  useEffect(() => {
    /**
     * 特殊处理：左侧立绘
     */
    const thisFigKey = 'fig-left';
    const softInAniKey = 'fig-left-softin';
    if (figNameLeft !== '') {
      const currentFigLeft = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
      if (currentFigLeft) {
        if (currentFigLeft.sourceUrl !== figNameLeft) {
          removeFig(currentFigLeft, softInAniKey, stageState.effects);
        }
      }
      WebGAL.gameplay.pixiStage?.addFigure(thisFigKey, figNameLeft, 'left');
      const regex = /.json$/;
      if (regex.test(figNameLeft)) {
        addLive2dFigure(
          thisFigKey,
          figNameLeft,
          'left',
          live2dMotion.find((e) => e.target === thisFigKey)?.motion ?? '',
        );
      }
      logger.debug('左立绘已重设');
      const { duration, animation } = getEnterExitAnimation(thisFigKey, 'enter');
      WebGAL.gameplay.pixiStage!.registerPresetAnimation(animation, softInAniKey, thisFigKey, stageState.effects);
      setTimeout(() => WebGAL.gameplay.pixiStage!.removeAnimationWithSetEffects(softInAniKey), duration);
    } else {
      logger.debug('移除左立绘');
      const currentFigLeft = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
      if (currentFigLeft) {
        if (currentFigLeft.sourceUrl !== figNameLeft) {
          removeFig(currentFigLeft, softInAniKey, stageState.effects);
        }
      }
    }
  }, [figNameLeft]);

  useEffect(() => {
    /**
     * 特殊处理：右侧立绘
     */
    const thisFigKey = 'fig-right';
    const softInAniKey = 'fig-right-softin';
    if (figNameRight !== '') {
      const currentFigRight = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
      if (currentFigRight) {
        if (currentFigRight.sourceUrl !== figNameRight) {
          removeFig(currentFigRight, softInAniKey, stageState.effects);
        }
      }
      WebGAL.gameplay.pixiStage?.addFigure(thisFigKey, figNameRight, 'right');
      const regex = /.json$/;
      if (regex.test(figNameRight)) {
        addLive2dFigure(
          thisFigKey,
          figNameRight,
          'right',
          live2dMotion.find((e) => e.target === thisFigKey)?.motion ?? '',
        );
      }
      logger.debug('右立绘已重设');
      const { duration, animation } = getEnterExitAnimation(thisFigKey, 'enter');
      WebGAL.gameplay.pixiStage!.registerPresetAnimation(animation, softInAniKey, thisFigKey, stageState.effects);
      setTimeout(() => WebGAL.gameplay.pixiStage!.removeAnimationWithSetEffects(softInAniKey), duration);
    } else {
      const currentFigRight = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
      if (currentFigRight) {
        if (currentFigRight.sourceUrl !== figNameRight) {
          removeFig(currentFigRight, softInAniKey, stageState.effects);
        }
      }
    }
  }, [figNameRight]);

  useEffect(() => {
    // 自由立绘
    for (const fig of freeFigure) {
      /**
       * 特殊处理：自由立绘
       */
      const thisFigKey = `${fig.key}`;
      const softInAniKey = `${fig.key}-softin`;
      /**
       * 非空
       */
      if (fig.name !== '') {
        const currentFigThisKey = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
        if (currentFigThisKey) {
          if (currentFigThisKey.sourceUrl !== fig.name) {
            removeFig(currentFigThisKey, softInAniKey, stageState.effects);
            WebGAL.gameplay.pixiStage?.addFigure(thisFigKey, fig.name, fig.basePosition);
            logger.debug(`${fig.key}立绘已重设`);
            const { duration, animation } = getEnterExitAnimation(thisFigKey, 'enter');
            WebGAL.gameplay.pixiStage!.registerPresetAnimation(animation, softInAniKey, thisFigKey, stageState.effects);
            setTimeout(() => WebGAL.gameplay.pixiStage!.removeAnimationWithSetEffects(softInAniKey), duration);
          }
        } else {
          WebGAL.gameplay.pixiStage?.addFigure(thisFigKey, fig.name, fig.basePosition);
          const regex = /.json$/;
          if (regex.test(fig.name)) {
            addLive2dFigure(
              thisFigKey,
              fig.name,
              fig.basePosition,
              live2dMotion.find((e) => e.target === thisFigKey)?.motion ?? '',
            );
          }
          logger.debug(`${fig.key}立绘已重设`);
          const { duration, animation } = getEnterExitAnimation(thisFigKey, 'enter');
          WebGAL.gameplay.pixiStage!.registerPresetAnimation(animation, softInAniKey, thisFigKey, stageState.effects);
          setTimeout(() => WebGAL.gameplay.pixiStage!.removeAnimationWithSetEffects(softInAniKey), duration);
        }
      } else {
        const currentFigThisKey = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisFigKey);
        if (currentFigThisKey) {
          if (currentFigThisKey.sourceUrl !== fig.name) {
            removeFig(currentFigThisKey, softInAniKey, stageState.effects);
          }
        }
      }
    }

    /**
     * 移除不在状态表中的立绘
     */
    const currentFigures = WebGAL.gameplay.pixiStage?.getFigureObjects();
    if (currentFigures) {
      for (const existFigure of currentFigures) {
        if (
          existFigure.key === 'fig-left' ||
          existFigure.key === 'fig-center' ||
          existFigure.key === 'fig-right' ||
          existFigure.key.endsWith('-off')
        ) {
          // 什么也不做
        } else {
          const existKey = existFigure.key;
          const existFigInState = freeFigure.findIndex((fig) => fig.key === existKey);
          if (existFigInState < 0) {
            const softInAniKey = `${existFigure.key}-softin`;
            removeFig(existFigure, softInAniKey, stageState.effects);
          }
        }
      }
    }
  }, [freeFigure]);
}

```

该函数名为 `removeFig`，其作用是移除一个舞台对象（StageObject）中的动画，并为该动画指定一个进入的停止键（enterTikerKey）和一个或多个效果（effects）。

函数内部首先尝试使用 `WebGAL.gameplay.pixiStage.removeAnimationWithSetEffects` 方法移除舞台对象中的动画，该方法可以同时设置进入和退出效果。如果舞台对象进入的动画存在，则函数会尝试立即关闭立绘，否则将舞台对象从舞台上移除并重新开始动画。

接下来，函数会使用 `WebGAL.gameplay.isFast` 获取当前是否处于快速模式。如果是快速模式，则函数会立即关闭立绘并移除舞台对象。否则，函数会将进入停止键设置为该舞台对象的键，并使用 `WebGAL.gameplay.pixiStage.removeStageObjectByKey` 方法移除舞台对象。

最后，函数会尝试使用 `getEnterExitAnimation` 函数获取舞台对象进入或退出动画的信息，并使用 `WebGAL.gameplay.pixiStage!.registerPresetAnimation` 方法将进入停止键指定的动画注册为预设动画，指定进入停止键和舞台对象。然后，函数会使用 `setTimeout` 函数在一定时间内延迟移除进入停止键的动画和舞台对象，即等待进入停止键指定的动画完成之后，移除舞台对象并重新开始动画。


```js
function removeFig(figObj: IStageObject, enterTikerKey: string, effects: IEffect[]) {
  WebGAL.gameplay.pixiStage?.removeAnimationWithSetEffects(enterTikerKey);
  // 快进，跳过退出动画
  if (WebGAL.gameplay.isFast) {
    logger.info('快速模式，立刻关闭立绘');
    WebGAL.gameplay.pixiStage?.removeStageObjectByKey(figObj.key);
    return;
  }
  const oldFigKey = figObj.key;
  figObj.key = figObj.key + '-off';
  WebGAL.gameplay.pixiStage?.removeStageObjectByKey(oldFigKey);
  const figKey = figObj.key;
  const leaveKey = figKey + '-softoff';
  const { duration, animation } = getEnterExitAnimation(figKey, 'exit');
  WebGAL.gameplay.pixiStage!.registerPresetAnimation(animation, leaveKey, figKey, effects);
  setTimeout(() => {
    WebGAL.gameplay.pixiStage?.removeAnimation(leaveKey);
    WebGAL.gameplay.pixiStage?.removeStageObjectByKey(figKey);
  }, duration);
}

```

这是一个 JavaScript 函数，名为 `addLive2dFigure`。它接受任意数量的参数 `args`，并返回一个 `Promise`。

函数的作用是执行 `WebGAL.gameplay.pixiStage?.addLive2dFigure.apply(args)` 方法，该方法将 `args` 数组中的内容应用到游戏场景的 `Live2D` 图形对象上。如果 `Live2D` 对象不存在，函数将不会创建它。

具体来说，这个函数的实现依赖于 `WebGAL.gameplay.pixiStage` 和 `Live2D` 两个库。函数中使用的 `addLive2dFigure` 方法来自 `WebGAL.gameplay.pixiStage` 库，它是一个字符串，表示要添加的 Live2D 图层 figure 的 ID。函数的参数是一个任意数量的参数 `args`，它们将被传递给 `apply` 方法，该方法将 `args` 数组中的每个元素应用到 `Live2D` 对象上。


```js
/**
 * 如果要使用 Live2D，取消这里的注释
 * @param args
 */
function addLive2dFigure(...args: any[]) {
  // return WebGAL.gameplay.pixiStage?.addLive2dFigure.apply(args);
}

```

# `/packages/webgal/src/Components/UI/getTextSize.ts`

这段代码定义了一个名为 `getTextSize` 的函数，它接受一个参数 `size`，表示想要知道文本的大小小。函数内部使用了一个 `switch` 语句，它根据 `size` 的值来返回一个常量。

当 `size` 的值为0时，函数返回150；当 `size` 的值为1时，函数返回205；当 `size` 的值为2时，函数返回280。如果 `size` 的值不匹配任何一个预定义的值，函数将返回205。

这段代码的主要目的是提供一个可以处理不同文本大小的函数，它可以在不同的情况下提供不同的返回值。


```js
export function getTextSize(size: number) {
  switch (size) {
    case 0:
      return 150;
    case 1:
      return 205;
    case 2:
      return 280;
    default:
      return 205;
  }
}

```

# `/packages/webgal/src/Components/UI/DevPanel/devFunctions/getPixiSscreenshot.ts`

该函数使用了 PIXI.js 库中的一个名为 `getPixiSscreenshot` 的函数，它将在网页上下载一张绘制有 PIXI 图案的截图并将其保存为 PNG 格式的图片。

具体来说，该函数执行以下步骤：

1. 从 `document.getElementById('pixiCanvas')!` 中获取到 `pixiCanvas` 元素的引用，因为 `pixiCanvas` 元素是包含在 `document.getElementById` 事件中的 `id` 参数中的。
2. 将 `pixiCanvas` 元素转换为一个 `HTMLCanvasElement` 对象。
3. 使用 `canvas.toBlob` 方法将 `pixiCanvas` 元素的内容（可能是 PNG 图像数据）转换为一个二进制缓冲区（Blob）。
4. 如果缓冲区不为空，执行以下操作：
	* 使用 `document.createElement('a')` 创建一个新的 `a` 元素。
	* 使用 `document.body.append` 方法将 `a` 元素添加到 `document.body` 对象上。
	* 使用 `a.download` 属性设置生成的图片的文件名，设置为 `screenshot`。
	* 使用 `a.href` 属性设置生成的图片的 URL，设置为 `URL.createObjectURL(b)`，其中 `b` 是缓冲区中的内容。
	* 执行 `a.click` 事件，打开生成的图片。
	* 删除 `a` 元素。

该函数的作用是使用 PIXI.js 库中的 `toBlob` 方法将 `pixiCanvas` 中的图形保存为 PNG 格式，并在网页上下载一张包含 PIXI 图案的截图。


```js
export function getPixiSscreenshot() {
  const canvas: HTMLCanvasElement = document.getElementById('pixiCanvas')! as HTMLCanvasElement;
  canvas.toBlob((b) => {
    if (b) {
      const a = document.createElement('a');
      document.body.append(a);
      a.download = 'screenshot';
      a.href = URL.createObjectURL(b);
      a.click();
      a.remove();
    }
  }, 'image/png');
}

```

# `/packages/webgal/src/Components/UI/Menu/MenuPanel/menuPanelInterface.ts`

这段代码定义了一个名为IMenuPanel的接口，用于定义菜单页面上按钮的参数。

IMenuPanel接口有以下属性：

- clickFunc: 点击事件触发的函数，可以定义为buttonClickFunc;
- buttonOnClassName: 按钮激活时的className，可以定义为onButtonClickClassName;
- tagColor: 标签颜色，可以定义为onButtonTagColor;
- iconColor: 图标颜色，可以定义为onButtonIconColor;
- tagName: 标签显示名称，可以定义为onButtonText;
- iconName: 图标名称，可以定义为onButtonIconName。

IMenuPanel接口可以被用于开发一个菜单面板，其中包含按钮等菜单元素，每个按钮可以定义自己的样式和事件。


```js
/**
 * @interface IMenuPanel Menu页面的按钮的参数接口
 */
export interface IMenuPanel {
  clickFunc?: any; // 点击事件触发的函数
  buttonOnClassName?: string; // 按钮激活（在当前按钮对应页面）时的className
  tagColor?: string; // 标签颜色
  iconColor?: string; // 图标颜色
  tagName?: string; // 标签显示名称
  iconName: string; // 图标名称
}

```

# `/packages/webgal/src/Components/UI/Menu/Options/OptionInterface.ts`

这段代码定义了一个名为INormalButton的接口，它表示了一个普通的按钮，包括文本列表、功能列表和当前状态等属性。接着，定义了一个ISlider的接口，表示了一个滑块，包括滑块的唯一ID、接收滑块变化的回调函数、滑块的初始值等属性。最后，这两个接口被用来创建一个带有这些属性的对象，并在页面上显示一个带有这些属性的按钮和滑块组件。


```js
/**
 * @interface INormalButton 普通按钮的参数
 */
import { ChangeEvent } from 'react';

export interface INormalButton {
  textList: Array<string>;
  functionList: Array<any>;
  currentChecked: number;
}

export interface ISlider {
  uniqueID: string;
  onChange: (event: ChangeEvent<HTMLInputElement>) => void;
  initValue: number;
}

```

# `/packages/webgal/src/config/info.ts`

该代码定义了一个名为 `__INFO` 的对象，其中包含一个名为 `version` 的键，其值为 `'WebGAL 4.4.5.3'`。该对象还包括一个名为 `contributors` 的键，其包含一个由多个对象组成的数组，每个对象都包含一个 `username` 键和一个 `link` 键，用于表示该对象的贡献者信息。这些对象的数量和内容取决于后面导入的外部依赖关系或者已经定义好的常量。


```js
export const __INFO = {
  version: 'WebGAL 4.4.5.3',
  contributors: [
    { username: 'Mahiru', link: 'https://github.com/MakinoharaShoko' },
    { username: 'Hoshinokinya', link: 'https://github.com/hshqwq' },
    { username: 'Junbo Xiong', link: 'https://github.com/C6H5-NO2' },
    { username: 'lykl', link: 'https://github.com/lykl' },
    { username: 'SakuraSnow', link: 'https://github.com/sliyoxn' },
    { username: 'bcqsd', link: 'https://github.com/bcqsd' },
    { username: 'Yuji Sakai', link: 'https://github.com/generalfreed' },
    { username: 'Iara', link: 'https://github.com/labiker' },
    { username: '22', link: 'https://github.com/nini22P' },
    { username: '德布罗煜', link: 'https://github.com/ch1ny' },
    { username: 'Mike Zhou', link: 'https://github.com/mikezzb' },
    { username: 'Murasame0721', link: 'https://github.com/Murasame0721' },
    { username: 'loliko', link: 'https://github.com/loliko114514' },
    { username: 'IdrilK', link: 'https://github.com/IdrilK' },
    { username: 'callofblood', link: 'https://github.com/callofblood' },
    { username: 'lyle', link: 'https://github.com/lylelove' },
  ],
};

```

# `/packages/webgal/src/config/language.ts`

这段代码是一个JavaScript配置文件，它允许你在应用程序中配置多种语言的翻译。它主要实现了以下功能：

1. 配置了多种语言的翻译图层。
2. 允许你关闭某些窗口或对话框。
3. 允许你添加想要翻译的语言。

具体来说，这段代码可以让你在一个配置文件中定义多种语言的翻译，然后在你的应用程序中使用它们。你只需指定要翻译的语言名称，而不必担心具体的语言细节。当需要关闭窗口或对话框时，你可以通过添加"//"来指明。


```js
/**
 * You can config the languages display in this file.
 * If you want close someone, please add "//" forward that line.
 * If you want add language, please add the language English abbreviation name into language and languages,
 * also you need to code the name of it show.
 */

import en from '@/translations/en';
import jp from '@/translations/jp';
import zhCn from '@/translations/zh-cn';
import fr from '@/translations/fr';
/*
  Import your translation configs here;
  example:
  import myLang from '@/translations/filename of your config file';
```

以上代码定义了一个枚举类型language，该枚举类型有四个成员：zhCn、en、jp和fr。

接着，定义了一个const类型的变量languages，该变量使用了Record类型的初始化方式，用于存储了一个Map类型的对象。该Map对象包含四个键值对：{zhCn: '中文'、en: 'English'、jp: '日本語'和fr: 'Français'}。

最后，通过在const类型的变量languages上添加了一个斜杠，将Map类型的对象转化为一个字符串类型的枚举类型，这样就完成了对language枚举类型的定义。


```js
*/

export enum language {
  zhCn,
  en,
  jp,
  fr,
}

const languages: Record<string, string> = {
  zhCn: '中文',
  en: 'English',
  jp: '日本語',
  fr: 'Français',
};

```

这段代码定义了一个名为 `i18nTranslationResources` 的常量对象，它将不同语言的翻译资源存储在一个 object 中，每种语言的资源对象是一个 `Record` 对象，键为语言代码，值为一个包含翻译信息的对象。

具体来说，这个 object 包含以下几个键值对：

* `en`：键为英文，值为一个包含翻译信息的对象。
* `zhCn`：键为中文，值为一个包含翻译信息的对象。
* `jp`：键为日文，值为一个包含翻译信息的对象。
* `fr`：键为法语，值为一个包含翻译信息的对象。

此外，还有一个名为 `defaultLanguage` 的常量变量，它的值为 `language.zhCn`，即默认语言为中文。

最后，还有一个名为 `languages` 的常量变量，它的值为上面定义的 `i18nTranslationResources` 对象。


```js
export const i18nTranslationResources: Record<string, { translation: Record<string, any> }> = {
  en: { translation: en },
  zhCn: { translation: zhCn },
  jp: { translation: jp },
  fr: { translation: fr },
};

export const defaultLanguage: language = language.zhCn;

export default languages;

```

# `/packages/webgal/src/Core/initializeScript.ts`

这段代码是一个JavaScript文件，名为“initialization.js”，在游戏引擎初始化时会被执行。

它主要实现了以下功能：

1. 导入一些 utility 函数和库，包括 logger，infoFetcher，assetSetter，sceneFetcher，sceneParser，bindExtraFunc，webSocketFunc，以及 PixiController 和 PixiStage，axios库等。

2. 初始化游戏引擎的相关变量和设置，包括游戏信息，核心初始化函数，场景加载函数，场景解析函数，以及场景预处理函数等。

3. 创建一个 bindExtraFunc 函数，用来绑定一些方便使用的函数。

4. 导入 config/info 的 __INFO 常量，用来在游戏中输出一些信息。

5. 导入 PixiController 和 PixiStage，用来在游戏引擎中创建 Pixi 控制器并渲染场景。


```js
/**
 * @file 引擎初始化时会执行的脚本，包括获取游戏信息，初始化运行时变量，初始化用户数据存储
 */
import { logger } from './util/etc/logger';
import { infoFetcher } from './util/coreInitialFunction/infoFetcher';
import { assetSetter, fileType } from './util/gameAssetsAccess/assetSetter';
import { sceneFetcher } from './controller/scene/sceneFetcher';
import { sceneParser } from './parser/sceneParser';
import { bindExtraFunc } from '@/Core/util/coreInitialFunction/bindExtraFunc';
import { webSocketFunc } from '@/Core/util/syncWithEditor/webSocketFunc';
import uniqWith from 'lodash/uniqWith';
import { scenePrefetcher } from './util/prefetcher/scenePrefetcher';
import PixiStage from '@/Core/controller/stage/pixi/PixiController';
import axios from 'axios';
import { __INFO } from '@/config/info';
```

这段代码是一个 JavaScript 导出函数，它使用了一个名为 `WebGAL` 的库，并尝试从用户的用户代理（userAgent）中提取关于设备的信息。

首先，它判断设备是否为 iOS 终端，通过检查用户代理中是否包含某些特定的字符串来判断。然后，它调用了一个名为 `initializeScript` 的函数，这个函数会在浏览器启动时被调用，并在页面上打印一些日志信息。

`initializeScript` 函数的作用取决于它的调用者的需求。通常情况下，这个函数会被用来设置一些全局性的初始化逻辑，例如激活强制缩放，以确保在调整窗口大小时能够正确地重新计算宽高。

在这里，判断是否为 iOS 终端的代码，如果设备的确为 iOS 终端，则会弹出一个警告框，告知用户需要在横屏设备上获得最佳体验。这个警告框在 iOS 设备上默认是打开的，因此它并不会阻止用户在横屏设备上访问网站。


```js
import { WebGAL } from '@/Core/WebGAL';

const u = navigator.userAgent;
export const isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); // 判断是否是 iOS终端

/**
 * 引擎初始化函数
 */
export const initializeScript = (): void => {
  // 打印初始log信息
  logger.info(__INFO.version);
  logger.info('Github: https://github.com/MakinoharaShoko/WebGAL ');
  logger.info('Made with ❤ by MakinoharaShoko');
  // 激活强制缩放
  // 在调整窗口大小时重新计算宽高，设计稿按照 1600*900。
  if (isIOS) {
    /**
     * iOS
     */
    alert(
      `iOS 用户请横屏使用以获得最佳体验
```

这段代码是一个JavaScript函数，它使用了多个APIs和库来创建一个iOS游戏。以下是它的主要部分，我将对其进行逐行解释：

```js
// 在此处获得iOS用户推荐的最佳体验。
| Please use landscape mode on iOS for the best experience|

```

这是游戏的标题，告诉用户在iOS设备上使用竖屏模式可以获得更好的游戏体验。

```js
// 在此处引入Pixi的样式文件。
 .css

```

这引入了Pixi的样式文件，以便在游戏中的UI元素上应用样式。

```js
// 这里获取用户动画。
 .getUserAnimation()

```

这使用getUserAnimation()函数从用户获取动画信息，以便在游戏中将动画应用到玩家身上。

```js
// 这里加载游戏信息。
 .loadStyle('./game/userStyleSheet.css');
 // 这里从游戏文件夹中获取配置文件。
 .getUserAnimation();
 // 这里从游戏文件夹中获取启动场景。
 .infoFetcher('./game/config.txt');
 // 这里从游戏文件夹中获取当前场景。
 .loadStyle('./game/scene/start.txt');
 // 这里从游戏文件夹中加载当前场景。
 .sceneFetcher('./game/config.txt').then((rawScene) => {
   WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, 'start.txt', './game/config.txt');
   // 这里将应用到场景中的静态资源复制到游戏中的运行时。
   copyAssetTemplatesToGame(rawScene, './game/config.txt');
   // 这里启用场景预加载。
   WebGAL.sceneManager.settledScenes.push(rawScene);
   // 这里去除重复场景，以便正确加载场景。
   WebGAL.sceneManager.sortScenesByKey().then((scenes) => {
     scenes.sort((a, b) => scenes.b.index - scenes.a.index);
     scenes.sort((a, b) => scenes.a.length - scenes.b.length);
     scenes.sort((a, b) => scenes.b.index - scenes.a.index);
   });
   // 这里应用场景初始化逻辑。
   runGame();
 });

```

在这段代码中，游戏获取了一个用户动画，然后加载了一个样式表，接着从游戏文件夹中读取了一个配置文件，然后加载了一个启动场景。在启动场景中，静态资源被复制到游戏中的运行时，场景被预加载。最后，游戏中的运行时被初始化，然后开始运行游戏。


```js
| Please use landscape mode on iOS for the best experience
| iOS ユーザーは横画面での使用をお勧めします`,
    );
  }

  // 获得 userAnimation
  loadStyle('./game/userStyleSheet.css');
  // 获得 user Animation
  getUserAnimation();
  // 获取游戏信息
  infoFetcher('./game/config.txt');
  // 获取start场景
  const sceneUrl: string = assetSetter('start.txt', fileType.scene);
  // 场景写入到运行时
  sceneFetcher(sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, 'start.txt', sceneUrl);
    // 开始场景的预加载
    const subSceneList = WebGAL.sceneManager.sceneData.currentScene.subSceneList;
    WebGAL.sceneManager.settledScenes.push(sceneUrl); // 放入已加载场景列表，避免递归加载相同场景
    const subSceneListUniq = uniqWith(subSceneList); // 去重
    scenePrefetcher(subSceneListUniq);
  });
  /**
   * 启动Pixi
   */
  WebGAL.gameplay.pixiStage = new PixiStage();

  /**
   * iOS 设备 卸载所有 Service Worker
   */
  // if ('serviceWorker' in navigator && isIOS) {
  //   navigator.serviceWorker.getRegistrations().then((registrations) => {
  //     for (const registration of registrations) {
  //       registration.unregister().then(() => {
  //         logger.info('已卸载 Service Worker');
  //       });
  //     }
  //   });
  // }

  /**
   * 绑定工具函数
   */
  bindExtraFunc();
  webSocketFunc();
};

```

这两函数脚本的作用是通过 `loadStyle` 函数加载 external CSS 文件，然后将包含该样式表的 `link` 元素添加到页面 `<head>` 中的第一个元素，从而为游戏应用程序添加样式表。具体来说，`loadStyle` 函数接收一个 `url` 参数，它是一个字符串，表示 CSS 文件的 URL。函数返回一个包含 `link` 元素的 `document.createElement` 对象。这个 `link` 元素会将一个 `text/css` 类型的标签，一个 `rel` 属性的 `link` 元素，和一个 `href` 属性的 `link` 元素添加到页面的 `<head>` 元素中。

而 `getUserAnimation` 函数则是调用 `axios` 库的一个 `get` 请求，从 `./game/animation/animationTable.json` 中获取游戏中的动画数据，并将这些数据存储在一个数组中。然后，它再调用另一个 `get` 请求，从 `./game/animation/${animationName}.json` 中获取与每个动画名称对应的动画数据。如果第一个请求返回的数据有更新，它就会在用户动画列表中添加一个新的动画，并将其存储到 `userAnimation` 变量中。最后，它将 `userAnimation` 对象传递给 `WebGAL.animationManager.addAnimation` 方法，以便在游戏启动时加载它。


```js
function loadStyle(url: string) {
  const link = document.createElement('link');
  link.type = 'text/css';
  link.rel = 'stylesheet';
  link.href = url;
  const head = document.getElementsByTagName('head')[0];
  head.appendChild(link);
}

function getUserAnimation() {
  axios.get('./game/animation/animationTable.json').then((res) => {
    const animations: Array<string> = res.data;
    for (const animationName of animations) {
      axios.get(`./game/animation/${animationName}.json`).then((res) => {
        if (res.data) {
          const userAnimation = {
            name: animationName,
            effects: res.data,
          };
          WebGAL.animationManager.addAnimation(userAnimation);
        }
      });
    }
  });
}

```

# `/packages/webgal/src/Core/WebGAL.ts`

这段代码使用了JavaScript二次打包库（Webpack）导入了@/Core/webgalCore库，并创建了一个名为WebGAL的实例。WebGAL作为一个WebGALCore实例，可以用来进行互联网内容分析、对比、以及相关材料的抓取等。


```js
import { WebgalCore } from '@/Core/webgalCore';

export const WebGAL = new WebgalCore();

```

# `/packages/webgal/src/Core/webgalCore.ts`

该代码定义了一个名为 "WebgalCore" 的类，包含了以下模块：

- BacklogManager：用于处理游戏中的排行榜和日志。
- mitt：用于在游戏循环中捕获和处理事件。
- SceneManager：用于管理游戏中的场景。
- AnimationManager：用于管理游戏中的动画。
- Gameplay：用于处理游戏玩法逻辑。

在类的构造函数中，调用了场景管理器、排行榜管理器和动画管理器的构造函数，并创建了游戏玩法的实例。还创建了两个字符串类型的变量 gameName 和 gameKey，用于在游戏中存储游戏名称和键，以及一个名为 eventBus 的 mitt 对象，用于在游戏中处理事件。


```js
import { BacklogManager } from '@/Core/Modules/backlog';
import mitt from 'mitt';
import { SceneManager } from '@/Core/Modules/scene';
import { AnimationManager } from '@/Core/Modules/animations';
import { Gameplay } from './Modules/gamePlay';

export class WebgalCore {
  public sceneManager = new SceneManager();
  public backlogManager = new BacklogManager(this.sceneManager);
  public animationManager = new AnimationManager();
  public gameplay = new Gameplay();
  public gameName = '';
  public gameKey = '';
  public eventBus = mitt();
}

```

# `/packages/webgal/src/Core/config/config.ts`

这两行代码定义了两个对象，分别 export const SYSTEM_CONFIG = { backlog_size: 200 } 和 export const PERFORM_CONFIG = { textInitialDelay: 80 }。其中，SYSTEM_CONFIG 的作用是存储后端服务的后端处理能力设置，而 PERFORM_CONFIG 的作用是设置一个文本转写任务的延迟时间。

后端服务的后端处理能力设置

SYSTEM_CONFIG 中的后端处理能力设置为 200，这意味着后端将会有 200 个请求队列，用于处理各种请求。这个设置可以帮助您在分布式系统中的后端服务中处理高负载的情况。

文本转写任务的延迟时间设置

PERFORM_CONFIG 中的文本转写任务的延迟时间为 80，这意味着在将请求发送给后端之前，允许有一个最大 80 毫秒的延迟时间。这个设置可以帮助您在将请求发送给后端之前进行必要的延迟处理，以提高您的应用程序的性能。


```js
export const SYSTEM_CONFIG = {
  backlog_size: 200,
};
export const PERFORM_CONFIG = {
  // 不能小于50
  textInitialDelay: 80,
};

```

# `/packages/webgal/src/Core/controller/gamePlay/autoPlay.ts`

这段代码的作用是设置一个包含“autoplay”按钮的UI组件中的“on”属性（也就是开关）的激活状态。具体来说，它通过使用`logger`库中的`setButton`函数来设置按钮的样式，并在样式中定义了一个名为`Button_ControlPanel_auto`的CSS类。当`on`为`true`时，将按钮的样式设置为`styles.button_on`，当`on`为`false`时，将按钮的样式设置为`styles.singleButton`。然后，这个样式类会被应用到包含“autoplay”按钮的UI组件上。


```js
// import {logger} from '../../util/logger';
import styles from '../../../Components/UI/BottomControlPanel/bottomControlPanel.module.scss';
import { webgalStore } from '@/store/store';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 设置 autoplay 按钮的激活与否
 * @param on
 */
const setButton = (on: boolean) => {
  const autoIcon = document.getElementById('Button_ControlPanel_auto');
  if (autoIcon) {
    if (on) {
      autoIcon.className = styles.button_on;
    } else autoIcon.className = styles.singleButton;
  }
};

```

这段代码定义了一个名为 `stopAuto` 的函数，用于停止自动播放。函数内部使用了 WebGAL.gameplay 对象，这是 Adobe Flash 游戏引擎的一个内置模块，用于控制游戏中的自动播放功能。

具体来说，这段代码的作用是关闭自动播放，即在游戏正在自动播放时停止游戏自动播放的功能。具体实现方式如下：

1. 通过 `WebGAL.gameplay.isAuto` 属性，将游戏中的自动播放设置为 false，这会取消自动播放的设置。

2. 通过 `setButton` 函数(可能是用于游戏界面中的按钮)，将按钮的状态设置为 false，这也会取消游戏界面的自动播放功能。

3. 通过 `if (WebGAL.gameplay.autoInterval !== null)` 条件判断，如果游戏中的自动播放设置为非 null，则执行以下操作：

  - 通过 `clearInterval` 函数，清除游戏中的自动播放定时器。这可以防止游戏中的自动播放在用户手动停止自动播放之前停止。

  - 通过 `WebGAL.gameplay.autoInterval = null` 设置，将游戏中的自动播放定时器设置为 null，这可以防止游戏中的自动播放在自动播放设置过期后继续自动播放。

4. 通过 `if (WebGAL.gameplay.autoTimeout !== null)` 条件判断，如果游戏中的自动播放设置为非 null，则执行以下操作：

  - 通过 `clearTimeout` 函数，清除游戏中的自动播放定时器。这可以防止游戏中的自动播放在自动播放设置过期后继续自动播放。

  - 通过 `WebGAL.gameplay.autoTimeout = null` 设置，将游戏中的自动播放定时器设置为 null，这可以防止游戏中的自动播放在自动播放设置过期后继续自动播放。


```js
/**
 * 停止自动播放
 */
export const stopAuto = () => {
  WebGAL.gameplay.isAuto = false;
  setButton(false);
  if (WebGAL.gameplay.autoInterval !== null) {
    clearInterval(WebGAL.gameplay.autoInterval);
    WebGAL.gameplay.autoInterval = null;
  }
  if (WebGAL.gameplay.autoTimeout !== null) {
    clearTimeout(WebGAL.gameplay.autoTimeout);
    WebGAL.gameplay.autoTimeout = null;
  }
};

```

这段代码定义了一个名为 `switchAuto` 的函数，用于切换网页游戏中的自动播放状态。

函数内部，首先检查当前的游戏是否处于自动播放状态，如果是，则调用 `stopAuto()` 函数停止自动播放，否则创建一个新的自动播放状态，并设置 `WebGAL.gameplay.isAuto` 为 `true`，以及设置一个定时器，在每隔 100 毫秒时调用 `autoPlay()` 函数来开始自动播放。

通过调用 `switchAuto()` 函数，用户可以手动切换网页游戏的自动播放状态。


```js
/**
 * 切换自动播放状态
 */
export const switchAuto = () => {
  // 现在正在自动播放
  if (WebGAL.gameplay.isAuto) {
    stopAuto();
  } else {
    // 当前不在自动播放
    WebGAL.gameplay.isAuto = true;
    setButton(true);
    WebGAL.gameplay.autoInterval = setInterval(autoPlay, 100);
  }
};

```

这段代码定义了一个名为 `autoNextSentence` 的函数，它会执行一次 `nextSentence` 函数并设置 `WebGAL.gameplay.autoTimeout` 状态为 `null`。

接下来，定义了一个名为 `autoPlay` 的函数，它会执行一次 `autoNextSentence` 函数，然后根据 `WebGAL.gameplay.autoSpeed` 设置一个延迟 `delay`，最后根据延迟设置 `autoPlayDelay` 变量。

然后，代码中创建了一个 `isBlockingAuto` 变量，初始值为 `false`，接着遍历 `WebGAL.gameplay.performController.performList` 列表，如果当前元素 `e` 在 `blockingAuto` 属性上为 `true`，则将 `isBlockingAuto` 设置为 `true`。

接下来，代码会判断 `isBlockingAuto` 的值，如果为 `true`，则提前结束执行，否则继续执行。最后，代码设置了一个定时器 `autoNextSentence`，延迟为 `autoPlayDelay`。当 `WebGAL.gameplay.autoTimeout` 被设置为 `null` 时，将 `autoNextSentence` 中的 `nextSentence` 函数执行一次，并且设置 `WebGAL.gameplay.autoTimeout` 状态为 `null`。


```js
export const autoNextSentence = () => {
  nextSentence();
  WebGAL.gameplay.autoTimeout = null;
};

/**
 * 自动播放的执行函数
 */
const autoPlay = () => {
  const delay = webgalStore.getState().userData.optionData.autoSpeed;
  const autoPlayDelay = 750 - 250 * delay;
  let isBlockingAuto = false;
  WebGAL.gameplay.performController.performList.forEach((e) => {
    if (e.blockingAuto())
      // 阻塞且没有结束的演出
      isBlockingAuto = true;
  });
  if (isBlockingAuto) {
    // 有阻塞，提前结束
    return;
  }
  // nextSentence();
  if (WebGAL.gameplay.autoTimeout === null) {
    WebGAL.gameplay.autoTimeout = setTimeout(autoNextSentence, autoPlayDelay);
  }
};

```

# `/packages/webgal/src/Core/controller/gamePlay/backToTitle.ts`

这段代码是一个高阶组件，它将一系列与游戏相关的状态操作组合在一起。主要目的是在玩家退出游戏时，将应用程序状态重置为标题界面。

具体来说，这段代码会执行以下操作：

1. 停止游戏中的所有自动执行任务。
2. 停止游戏中的自动播放功能。
3. 停止游戏中的快速跳过功能。
4. 将应用程序状态中的 visibility 设置为 true，以显示标题界面。
5. 将应用程序状态中的 GUI.titleBg 设置为当前标题背景。

这段代码还包含一个 backToTitle 方法，该方法会通过调用上述操作来执行重置操作。


```js
import { webgalStore } from '@/store/store';
import { setVisibility } from '@/store/GUIReducer';
import { stopAllPerform } from '@/Core/controller/gamePlay/stopAllPerform';
import { stopAuto } from '@/Core/controller/gamePlay/autoPlay';
import { stopFast } from '@/Core/controller/gamePlay/fastSkip';
import { setEbg } from '@/Core/util/setEbg';

export const backToTitle = () => {
  const dispatch = webgalStore.dispatch;
  stopAllPerform();
  stopAuto();
  stopFast();
  // 重新打开标题界面
  dispatch(setVisibility({ component: 'showTitle', visibility: true }));
  /**
   * 重设为标题背景
   */

  setEbg(webgalStore.getState().GUI.titleBg);
};

```

# `/packages/webgal/src/Core/controller/gamePlay/fastSkip.ts`

这段代码是一个 JavaScript 片段，它定义了一个名为 `setButton` 的函数。

这个函数的作用是切换一个名为 `Button_ControlPanel_fast` 的元素的激活状态。这个元素是一个按钮，它被用来控制一个自动播放的功能。

函数接受一个布尔值 `on`，表示是否要切换按钮的激活状态。如果 `on` 是 `true`，那么函数会点击按钮，按钮的样式会设置为与 `styles.button_on` 样式相同；如果 `on` 是 `false`，那么函数会点击按钮，按钮的样式会设置为与 `styles.singleButton` 样式相同。

整个函数中，使用了两个对象：`stopAuto` 和 `nextSentence`。`stopAuto` 对象的作用是停止自动播放功能，而 `nextSentence` 对象的作用是控制下一句话的输出。


```js
// 切换自动播放状态
import { stopAuto } from './autoPlay';
import styles from '../../../Components/UI/BottomControlPanel/bottomControlPanel.module.scss';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 设置 fast 按钮的激活与否
 * @param on
 */
const setButton = (on: boolean) => {
  const autoIcon = document.getElementById('Button_ControlPanel_fast');
  if (autoIcon) {
    if (on) {
      autoIcon.className = styles.button_on;
    } else autoIcon.className = styles.singleButton;
  }
};

```

这段代码是一个ES6模块中的一个函数，它的作用是停止快进模式。

代码首先检查是否处于快进模式，如果不是，就返回。如果处于快进模式，则执行以下操作：

1. 设置WebGAL游戏中的fastInterval为false。
2. 清除interval对象中现有的 fastInterval定时器。
3. 设置fastInterval为null，将其从游戏中的fastInterval数组中删除。

通过这个函数，用户可以暂停游戏的快进模式，使得游戏中的时间显示更加准确。


```js
/**
 * 停止快进模式
 */
export const stopFast = () => {
  if (!isFast()) {
    return;
  }
  WebGAL.gameplay.isFast = false;
  setButton(false);
  if (WebGAL.gameplay.fastInterval !== null) {
    clearInterval(WebGAL.gameplay.fastInterval);
    WebGAL.gameplay.fastInterval = null;
  }
};

```

这段代码是一个 JavaScript 函数，用于在游戏中的快速进阶模式下进行操作。具体来说，它的作用是：

1. 如果游戏当前处于快速进阶模式，则直接返回，不会执行函数体中的代码。
2. 如果游戏当前不处于快速进阶模式，则将游戏中的快速进阶模式设置为 true，并将 `WebGAL.gameplay.fastInterval` 设置为每秒一次调用 `nextSentence` 函数。`nextSentence` 函数用于在快速进阶模式下连续发送两句话。


```js
/**
 * 开启快进
 */
export const startFast = () => {
  if (isFast()) {
    return;
  }
  WebGAL.gameplay.isFast = true;
  setButton(true);
  WebGAL.gameplay.fastInterval = setInterval(() => {
    nextSentence();
  }, 100);
};

// 判断是否是快进模式
```

这段代码定义了两个函数，一个是`isFast`，另一个是`stopAll`。

`isFast`函数的作用是判断当前的游戏是否处于快速模式。如果快速模式为真，则返回`true`，否则返回`false`。

`stopAll`函数的作用是停止快速模式和自动播放。它调用`stopFast`函数和`stopAuto`函数，分别停止快速模式和自动播放。


```js
export const isFast = function () {
  return WebGAL.gameplay.isFast;
};

/**
 * 停止快进模式与自动播放
 */
export const stopAll = () => {
  stopFast();
  stopAuto();
};

/**
 * 切换快进模式
 */
```

这段代码定义了一个名为 `switchFast` 的函数，它通过一个条件语句来判断是否处于快速移动模式，并根据情况执行不同的操作。

目前，快速移动模式已经被禁用，因此如果 `WebGAL.gameplay.isFast` 的值为 `true`，则函数将执行 `stopFast()` 函数，停止快速移动。否则，如果 `WebGAL.gameplay.isFast` 的值为 `false`，则函数将执行 `startFast()` 函数，重新启动快速移动。

具体来说，`switchFast` 函数会在以下两种情况下之一执行：

1. 当快速移动模式已启用时，停止快速移动并停止执行 `stopFast()` 函数。
2. 当快速移动模式禁用时，启动快速移动并停止执行 `startFast()` 函数。

快速移动函数的启用由 `WebGAL.gameplay.isFast` 属性控制，该属性在 `true` 时快速移动模式启用，在 `false` 时禁用。


```js
export const switchFast = () => {
  // 现在正在快进
  if (WebGAL.gameplay.isFast) {
    stopFast();
  } else {
    // 当前不在快进
    startFast();
  }
};

```

# `/packages/webgal/src/Core/controller/gamePlay/nextSentence.ts`

This is a JavaScript function that may be used in a game like Fortnite or League of Legends. It appears to perform actions for a player's character in a game, such as a促成演出、清除状态或跳过演出等。

It is written in JavaScript, and it appears to use a library called "WebGAL" (Web Game Authoring Library). It is backed up by a "backlogManager" and has a "webgalStore" that is used to store information about the game world.

The function takes an array of IRunPerform objects, which represent each演出 in the game. It checks if the演出 is being held back, and if it is, it performs various actions such as clearing the hold, stopping the演出， and updating the game world.

It appears to have multiple global variables, including a boolean flag called "allSettled" that determines whether the game is settled and a boolean flag called "isGoNext". It also has a function called "scriptExecutor" that seems to execute scripts that are stored in the "scripts" folder.

Overall, it looks like a versatile function that can be used to handle a wide range of演出 in a game.


```js
import { scriptExecutor } from './scriptExecutor';
import { logger } from '../../util/etc/logger';
import { webgalStore } from '@/store/store';
import { resetStageState } from '@/store/stageReducer';
import cloneDeep from 'lodash/cloneDeep';
import { IBacklogItem } from '@/Core/Modules/backlog';

import { SYSTEM_CONFIG } from '@/Core/config/config';
import { WebGAL } from '@/Core/WebGAL';
import { IRunPerform } from '@/store/stageInterface';

/**
 * 进行下一句
 */
export const nextSentence = () => {
  /**
   * 发送 “发生点击下一句” 事件。
   */
  WebGAL.eventBus.emit('__NEXT');

  // 如果当前显示标题，那么不进行下一句
  const GUIState = webgalStore.getState().GUI;
  if (GUIState.showTitle) {
    return;
  }

  // 第一步，检查是否存在 blockNext 的演出
  let isBlockingNext = false;
  WebGAL.gameplay.performController.performList.forEach((e) => {
    if (e.blockingNext())
      // 阻塞且没有结束的演出
      isBlockingNext = true;
  });
  if (isBlockingNext) {
    // 有阻塞，提前结束
    logger.warn('next 被阻塞！');
    return;
  }

  // 检查是否处于演出完成状态，不是则结束所有普通演出（保持演出不算做普通演出）
  let allSettled = true;
  WebGAL.gameplay.performController.performList.forEach((e) => {
    if (!e.isHoldOn && !e.skipNextCollect) allSettled = false;
  });
  if (allSettled) {
    // 所有普通演出已经结束
    // if (WebGAL.backlogManager.isSaveBacklogNext) {
    //   WebGAL.backlogManager.isSaveBacklogNext = false;
    // }
    // 清除状态表的演出序列（因为这时候已经准备进行下一句了）
    const stageState = webgalStore.getState().stage;
    const newStageState = cloneDeep(stageState);
    for (let i = 0; i < newStageState.PerformList.length; i++) {
      const e: IRunPerform = newStageState.PerformList[i];
      if (!e.isHoldOn) {
        newStageState.PerformList.splice(i, 1);
        i--;
      }
    }
    webgalStore.dispatch(resetStageState(newStageState));
    scriptExecutor();
    return;
  }

  // 不处于 allSettled 状态，清除所有普通演出，强制进入settled。
  logger.warn('提前结束被触发，现在清除普通演出');
  let isGoNext = false;
  for (let i = 0; i < WebGAL.gameplay.performController.performList.length; i++) {
    const e = WebGAL.gameplay.performController.performList[i];
    if (!e.isHoldOn) {
      if (e.goNextWhenOver) {
        isGoNext = true;
      }
      if (!e.skipNextCollect) {
        e.stopFunction();
        clearTimeout(e.stopTimeout as unknown as number);
        WebGAL.gameplay.performController.performList.splice(i, 1);
        i--;
      }
    }
  }
  if (isGoNext) {
    nextSentence();
  }
};

```

# `/packages/webgal/src/Core/controller/gamePlay/runScript.ts`

这段代码定义了一个函数 `scriptFunction`，它接受一个 `sentence` 对象作为参数，并返回一个 `IPerform` 对象。这个函数的作用是在场景中调用一个自定义的表演，并在调用完成后自动卸载演出。

具体来说，这段代码可以拆分成以下几个部分：

1. 导入了一些必要的模块和函数。
2. 定义了一个类型 `scriptFunction`，它表示一个接受 `sentence` 对象并返回 `IPerform` 对象的函数。
3. 引入了两个函数：`say` 和 `initPerform`，它们分别从游戏脚本中导入而来，用于输出文本信息和执行表演。
4. 通过 `SCRIPT_CONFIG` 函数导入了一个名为 `WebGAL` 的类，这个类可能用于与游戏引擎的交互。
5. 通过 `ISentence` 和 `IPerform` 接口，与场景接口中的语句执行和表演相关联。
6. 最后，定义了一个 `scriptFunction` 函数，它的作用是在场景中执行一个自定义的表演，并返回表演的 `IPerform` 对象。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { say } from '../../gameScripts/say';
import { initPerform, IPerform } from '@/Core/Modules/perform/performInterface';

import { WebGAL } from '@/Core/WebGAL';
import { SCRIPT_CONFIG } from '@/Core/parser/sceneParser';

/**
 * 规范函数的类型
 * @type {(sentence: ISentence) => IPerform}
 */
type scriptFunction = (sentence: ISentence) => IPerform;

/**
 * 语句调用器，真正执行语句的调用，并自动将演出在指定时间卸载
 * @param script 调用的语句
 */
```

这段代码定义了一个名为 `runScript` 的函数接口，它接受一个名为 `script` 的参数，该参数是一个表示 ISentence 对象的脚本字符串。

函数接口中，我们创建了一个名为 `perform` 的变量，它是一个实现了 IPerform 接口的变量，我们创建了一个名为 `funcToRun` 的变量，它是一个实现了 scriptFunction 接口的变量，该接口定义了脚本应该执行的函数。

接下来，我们使用 SCRIPT_CONFIG 数组来根据传入的脚本类型查找函数映射，如果找到了函数映射，我们就在 perform 变量上绑定函数；否则，我们将默认函数 assign 给 perform 变量。

最后，我们根据 perform 变量上绑定的函数，调用其函数并传入参数字符串，我们使用 ARRANGE_PERFORM_PROMISE 函数来通知游戏执行器新的表演，并在完成表演后通知游戏执行器。


```js
export const runScript = (script: ISentence) => {
  let perform: IPerform = initPerform;
  let funcToRun: scriptFunction = say; // 默认是say

  // 建立语句类型到执行函数的映射
  const scriptToFuncMap = new Map();
  SCRIPT_CONFIG.forEach((e) => {
    scriptToFuncMap.set(e.scriptType, e.scriptFunction);
  });

  // 根据脚本类型切换函数
  if (scriptToFuncMap.has(script.command)) {
    funcToRun = scriptToFuncMap.get(script.command) as scriptFunction;
  }

  // 调用脚本对应的函数
  perform = funcToRun(script);

  if (perform.arrangePerformPromise) {
    perform.arrangePerformPromise.then((resolovedPerform) =>
      WebGAL.gameplay.performController.arrangeNewPerform(resolovedPerform, script),
    );
  } else {
    WebGAL.gameplay.performController.arrangeNewPerform(perform, script);
  }
};

```

# `/packages/webgal/src/Core/controller/gamePlay/scriptExecutor.ts`

这段代码的作用是定义了一个名为 `cloneDeep` 的函数，并从多个模块中导入了一些不同类型的控制器和状态。具体来说，它从 `@/Core/controller/scene/sceneInterface` 模块中导入 `commandType` 和 `ISentence` 类型；从 `@/Core/store/stageInterface` 和 `@/Core/store/store` 模块中导入 `IStageState` 和 `restoreScene` 函数；从 `@/Core/gameScripts/setVar` 和 `@/Core/gameScripts/function/strIf` 模块中导入 `strIf` 和 `nextSentence` 函数；从 `@/Core/Modules/scene` 和 `@/Core/Modules/backlog` 模块中导入 `ISceneEntry` 和 `IBacklogItem` 接口；从 `@/Core/config/config` 和 `@/Core/WebGAL` 模块中导入 `SYSTEM_CONFIG` 和 `WebGAL` 类。

`cloneDeep` 函数的作用是接收一个深度克隆的参数，返回一个新的对象，这个新的对象具有原始对象的属性和状态。通过调用 `nextSentence` 函数，将原始场景的状态应用到新的场景中，并使用 `strIf` 函数检查给定的字符串是否为真，从而决定是否使用场景的下一帧。最后，将 `IBacklogItem` 对象的状态与 `ISceneEntry` 对象的状态进行比较，如果它们的状态不同，将 `IBacklogItem` 对象的状态存储到 `ISceneEntry` 对象的状态中。


```js
import { commandType, ISentence } from '@/Core/controller/scene/sceneInterface';
import { runScript } from './runScript';
import { logger } from '../../util/etc/logger';
import { IStageState } from '@/store/stageInterface';
import { restoreScene } from '../scene/restoreScene';
import { webgalStore } from '@/store/store';
import { getValueFromState } from '@/Core/gameScripts/setVar';
import { strIf } from '@/Core/gameScripts/function/strIf';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';
import cloneDeep from 'lodash/cloneDeep';
import { ISceneEntry } from '@/Core/Modules/scene';
import { IBacklogItem } from '@/Core/Modules/backlog';
import { SYSTEM_CONFIG } from '@/Core/config/config';
import { WebGAL } from '@/Core/WebGAL';

```

This is a JavaScript script that executes a macro function called `sayMacro`. The `sayMacro` function takes a single argument `currentScript` and returns.

The `sayMacro` function contains the following steps:

1. If the `currentScript` is not a macro, it returns immediately.
2. It searches for the first argument that has a `next` key and a value. If找到了下一个参数，它设置一个 flag `isNext` to `true`.
3. 对于每个搜索到的下一个参数，它检查该参数是否有一个名为 `notend` 的key，并将其值设置为 `true`。如果当前脚本有多个命令行参数，它将检查当前脚本是否包含一个名为 `notend` 的命令。
4. 如果 `isNext` 为 `true`，它将调用 `runScript` 函数并传递 `runToSentence` 参数。
5. 如果 `isNext` 为 `false`，它将执行 `scriptExecutor` 函数。
6. 最后，它调用 `setTimeout` 函数，该函数在 `0` 毫秒内执行 `sayMacro` 函数并保存当前舞台状态。

The `sayMacro` function uses the `WebGAL` and `logger` module for BacklogManager and scene管理。


```js
/**
 * 语句执行器
 * 执行语句，同步场景状态，并根据情况立即执行下一句或者加入backlog
 */
export const scriptExecutor = () => {
  // 超过总语句数量，则从场景栈拿出一个需要继续的场景，然后继续流程。若场景栈清空，则停止流程
  if (
    WebGAL.sceneManager.sceneData.currentSentenceId >
    WebGAL.sceneManager.sceneData.currentScene.sentenceList.length - 1
  ) {
    if (WebGAL.sceneManager.sceneData.sceneStack.length !== 0) {
      const sceneToRestore: ISceneEntry | undefined = WebGAL.sceneManager.sceneData.sceneStack.pop();
      if (sceneToRestore !== undefined) {
        restoreScene(sceneToRestore);
      }
    }
    return;
  }
  const currentScript: ISentence =
    WebGAL.sceneManager.sceneData.currentScene.sentenceList[WebGAL.sceneManager.sceneData.currentSentenceId];
  // 判断这个脚本要不要执行
  let runThis: number | boolean = true;
  let isHasWhenArg = false;
  let whenValue = '';
  currentScript.args.forEach((e) => {
    if (e.key === 'when') {
      isHasWhenArg = true;
      whenValue = e.value.toString();
    }
  });
  // 如果语句有 when
  if (isHasWhenArg) {
    // 先把变量解析出来
    const valExpArr = whenValue.split(/([+\-*\/()><!]|>=|<=|==)/g);
    const valExp = valExpArr
      .map((e) => {
        if (e.match(/[a-zA-Z]/)) {
          if (e.match(/true/) || e.match(/false/)) {
            return e;
          }
          return getValueFromState(e).toString();
        } else return e;
      })
      .reduce((pre, curr) => pre + curr, '');
    runThis = strIf(valExp);
  }
  // 执行语句
  if (!runThis) {
    logger.warn('不满足条件，跳过本句！');
    WebGAL.sceneManager.sceneData.currentSentenceId++;
    nextSentence();
    return;
  }
  runScript(currentScript);
  let isNext = false; // 是否要进行下一句
  currentScript.args.forEach((e) => {
    // 判断是否有下一句的参数
    if (e.key === 'next' && e.value) {
      isNext = true;
    }
  });

  let isSaveBacklog = currentScript.command === commandType.say; // 是否在本句保存backlog（一般遇到对话保存）
  // 检查当前对话是否有 notend 参数
  currentScript.args.forEach((e) => {
    if (e.key === 'notend' && e.value === true) {
      isSaveBacklog = false;
    }
  });
  let currentStageState: IStageState;

  // 执行至指定 sentenceID
  // if (runToSentence >= 0 && runtime_currentSceneData.currentSentenceId < runToSentence) {
  //   runtime_currentSceneData.currentSentenceId++;
  //   scriptExecutor(runToSentence);
  //   return;
  // }

  // 执行“下一句”
  if (isNext) {
    WebGAL.sceneManager.sceneData.currentSentenceId++;
    scriptExecutor();
    return;
  }

  /**
   * 为了让 backlog 拿到连续执行了多条语句后正确的数据，放到下一个宏任务中执行（我也不知道为什么这样能正常，有能力的可以研究一下
   */
  setTimeout(() => {
    // 同步当前舞台数据
    currentStageState = webgalStore.getState().stage;
    logger.debug('本条语句执行结果', currentStageState);
    // 保存 backlog
    if (isSaveBacklog) {
      // WebGAL.backlogManager.isSaveBacklogNext = true;
      WebGAL.backlogManager.saveCurrentStateToBacklog();
    }
  }, 0);
  WebGAL.sceneManager.sceneData.currentSentenceId++;
};

```

# `/packages/webgal/src/Core/controller/gamePlay/startContinueGame.ts`

这段代码是一个JavaScript脚本，它的目的是创建一个包含游戏元素和功能的游戏舞台。它主要通过引入不同的模块和函数来实现这一目的。

1. `assetSetter` 函数：从 '../../util/gameAssetsAccess/assetSetter' 包中导入，负责设置游戏资产。
2. `sceneFetcher` 函数：从 '../scene/sceneFetcher' 包中导入，负责从网络获取游戏场景。
3. `sceneParser` 函数：从 '../../parser/sceneParser' 包中导入，负责解析游戏场景的JSON数据。
4. `resetStage` 函数：从 '@/Core/controller/stage/resetStage' 包中导入，负责重置游戏舞台。
5. `webgalStore` 变量：从 '@/store/store' 包中导入，用于存储 WebGL 的上下文信息。
6. `setVisibility` 函数：从 '@/store/GUIReducer' 包中导入，用于设置游戏元素的可见性。
7. `nextSentence` 函数：从 '@/Core/controller/gamePlay/nextSentence' 包中导入，用于显示游戏中的对话。
8. `setEbg` 函数：从 '@/Core/util/setEbg' 包中导入，用于设置游戏元素的背景颜色。
9. `restorePerform` 函数：从 '@/Core/controller/storage/jumpFromBacklog' 包中导入，用于恢复在后台存储的游戏表现。
10. `hasFastSaveRecord` 函数：从 '@/Core/controller/storage/fastSaveLoad' 包中导入，用于检查游戏是否保存过快照。
11. `loadFastSaveGame` 函数：从 '@/Core/controller/storage/fastSaveLoad' 包中导入，用于加载游戏快照。
12. `WebGAL` 类：从 '@/Core/Core/WebGAL' 包中导入，用于与 WebGL 相关的操作。

经过以上导入和设置，这个游戏舞台可以用于游戏元素的创建、加载和快照恢复等功能。


```js
import { assetSetter, fileType } from '../../util/gameAssetsAccess/assetSetter';
import { sceneFetcher } from '../scene/sceneFetcher';
import { sceneParser } from '../../parser/sceneParser';
import { resetStage } from '@/Core/controller/stage/resetStage';
import { webgalStore } from '@/store/store';
import { setVisibility } from '@/store/GUIReducer';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';
import { setEbg } from '@/Core/util/setEbg';
import { restorePerform } from '@/Core/controller/storage/jumpFromBacklog';

import { hasFastSaveRecord, loadFastSaveGame } from '@/Core/controller/storage/fastSaveLoad';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 从头开始游戏
 */
```

这段代码是一个JavaScript模块，它定义了两个方法：startGame()和continueGame()。

startGame()方法的作用是在游戏开始时执行一些操作，包括：

1. 使用调用函数中的resetStage(true)方法，清除游戏舞台；
2. 加载游戏场景，使用const sceneUrl: string = assetSetter('start.txt', fileType.scene)的参数指定一个以assets/src/start.txt为文件名的场景文件，并将其加载到游戏场景中；
3. 使用then()方法，异步取得加载的的场景，并将其存储到WebGAL.sceneManager.sceneData中；
4. 使用nextSentence()方法，等待场景加载完成，然后继续游戏；
5. 如果游戏没有开始，使用nextSentence()方法，开始游戏。

continueGame()方法的作用是在游戏继续时执行一些操作，包括：

1. 设置游戏的快速保存状态；
2. 如果游戏正在进行中或者已经加载了游戏场景，那么不执行游戏操作；
3. 如果游戏没有开始，那么根据游戏状态决定是否继续游戏，并使用nextSentence()方法继续游戏；
4. 如果游戏正在运行中，那么使用setEbg()方法更改游戏的背景颜色，使用showTitle组件隐藏游戏标题，显示无法访问的游戏区域；
5. 如果游戏没有开始，那么使用loadFastSaveGame()方法加载快速保存的游戏记录，并使用continueGame()方法继续游戏。


```js
export const startGame = () => {
  resetStage(true);

  // 重新获取初始场景
  const sceneUrl: string = assetSetter('start.txt', fileType.scene);
  // 场景写入到运行时
  sceneFetcher(sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, 'start.txt', sceneUrl);
    // 开始第一条语句
    nextSentence();
  });
  webgalStore.dispatch(setVisibility({ component: 'showTitle', visibility: false }));
};

export async function continueGame() {
  /**
   * 重设模糊背景
   */
  setEbg(webgalStore.getState().stage.bgName);
  // 当且仅当游戏未开始时使用快速存档
  // 当游戏开始后 使用原来的逻辑
  if ((await hasFastSaveRecord()) && WebGAL.sceneManager.sceneData.currentSentenceId === 0) {
    // 恢复记录
    await loadFastSaveGame();
    return;
  }
  if (
    WebGAL.sceneManager.sceneData.currentSentenceId === 0 &&
    WebGAL.sceneManager.sceneData.currentScene.sceneName === 'start.txt'
  ) {
    // 如果游戏没有开始，开始游戏
    nextSentence();
  } else {
    restorePerform();
  }
}

```