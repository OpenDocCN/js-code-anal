# WebGal源码解析 4

# `/packages/webgal/src/Core/controller/stage/pixi/animations/index.ts`

这段代码定义了一个名为 `webgalAnimations` 的数组，该数组包含三个对象，每个对象都代表一个动画，并在对象中定义了一个 `animationGenerateFunc` 属性，该属性是一个函数，用于生成动画对象。

这个数组的目的是在 PIXI 动画库中定义一些可以用于生成不同类型动画的函数，以便在需要时进行调用，这样可以提高代码的可读性和可维护性。


```js
import { generateUniversalSoftInAnimationObj } from '@/Core/controller/stage/pixi/animations/universalSoftIn';
import { generateUniversalSoftOffAnimationObj } from '@/Core/controller/stage/pixi/animations/universalSoftOff';
import { generateTestblurAnimationObj } from '@/Core/controller/stage/pixi/animations/testblur';

export const webgalAnimations: Array<{ name: string; animationGenerateFunc: Function }> = [
  { name: 'universalSoftIn', animationGenerateFunc: generateUniversalSoftInAnimationObj },
  { name: 'universalSoftOff', animationGenerateFunc: generateUniversalSoftOffAnimationObj },
  { name: 'testblur', animationGenerateFunc: generateTestblurAnimationObj },
];

```

# `/packages/webgal/src/Core/controller/stage/pixi/animations/template.ts`

这段代码定义了一个名为 `generateTemplateAnimationObj` 的函数，用于创建动画模板。

该函数接受两个参数：

- `targetKey`：指作用目标精灵的实例的 `pixiContainer` 属性的索引，用于在动画中操控该精灵。
- `duration`：指动画模板的持续时间，单位是秒。

该函数返回一个包含以下属性的对象：

- `setStartState`：用于设置动画模板的初始状态的函数。此函数可以包含任何精灵的初始化代码，但请注意，这个函数不会被用于动画中。
- `setEndState`：用于设置动画模板的结束状态的函数。此函数可以包含任何精灵的结束状态的代码，但请注意，这个函数不会被用于动画中。
- `tickerFunc`：一个函数，用于在动画的每一帧执行。它接收一个 `delta` 参数，用于控制动画的速度。如果存在指定的精灵，该函数将在每一帧中更新该精灵的位置和状态。如果没有指定精灵，则不会更新任何精灵的状态。

这个函数的作用是为动画模板提供一个起点，让其在游戏中的游戏中更加灵活使用。具体来说，这个函数会在游戏中的每一帧中更新指定的精灵的位置和状态，从而实现动画的播放。


```js
import { WebGAL } from '@/Core/WebGAL';

/**
 * 动画创建模板
 * @param targetKey 作用目标
 * @param duration 持续时间
 */
export function generateTemplateAnimationObj(targetKey: string, duration: number) {
  const target = WebGAL.gameplay.pixiStage!.getStageObjByKey(targetKey);

  // 先设置一个通用的初态

  // TODO：通用初态设置
  /**
   * 在此书写为动画设置初态的操作
   */
  function setStartState() {}

  // TODO：通用终态设置
  /**
   * 在此书写为动画设置终态的操作
   */
  function setEndState() {}

  /**
   * 在此书写动画每一帧执行的函数
   * @param delta
   */
  function tickerFunc(delta: number) {
    if (target) {
      // 要操控的精灵
      const sprite = target.pixiContainer;
      // 每一帧的时间
      const baseDuration = WebGAL.gameplay.pixiStage!.frameDuration;

      /**
       * 在下面书写具体的动画
       */

      // 具体的操作......
    }
  }

  return {
    setStartState,
    setEndState,
    tickerFunc,
  };
}

```

# `/packages/webgal/src/Core/controller/stage/pixi/animations/testblur.ts`

这段代码定义了一个名为 `generateTestblurAnimationObj` 的函数，它接受一个目标（string）和一个持续时间（number）。这个函数的主要目的是生成一个模糊动画，这个动画会在给定的时间内从透明状态逐渐增强，并在达到目标后逐渐减弱。

函数内部，首先引入了 `WebGAL` 库，然后在函数体内部创建了一个 `WebGAL.gameplay.pixiStage` 对象，并获取了这个舞台对象的一个节点（target）。接下来，函数内部创建了一个通用初态（setStartState）和一个通用终态（setEndState），这些初态和终态设置用于设置模糊动画的起始和结束状态。在这些设置中，我们首先将目标对象的透明度设置为 0，然后设置其模糊度为 0。这样，当我们调用 `tick` 函数时，我们会根据目标对象当前的模糊度值和设定的增减比例来逐渐增强或减弱它的透明度，从而实现动画的模糊效果。

`tick` 函数内部，我们获取了当前活动的节点对象（target），然后获取了当前动画的时间间隔（duration）。接着，我们通过 `Math.random()` 函数生成一个随机数（delta），然后根据这个随机数和当前动画的时间间隔来更新目标对象的透明度和模糊度值。这里我们设定，在 `setStartState` 和 `setEndState` 中，透明度应该在 0 到 1 之间，模糊度应该在 0 到 5 之间。


```js
import { WebGAL } from '@/Core/WebGAL';

export function generateTestblurAnimationObj(targetKey: string, duration: number) {
  const target = WebGAL.gameplay.pixiStage!.getStageObjByKey(targetKey);

  // 先设置一个通用的初态

  // TODO：通用初态设置
  /**
   * 在此书写为动画设置初态的操作
   */
  function setStartState() {
    if (target) {
      target.pixiContainer.alpha = 0;
      // @ts-ignore
      target.pixiContainer.blur = 0;
    }
  }

  // TODO：通用终态设置
  /**
   * 在此书写为动画设置终态的操作
   */
  function setEndState() {
    if (target) {
      target.pixiContainer.alpha = 1;
      // @ts-ignore
      target.pixiContainer.blur = 5;
    }
  }

  /**
   * 在此书写动画每一帧执行的函数
   * @param delta
   */
  function tickerFunc(delta: number) {
    if (target) {
      const container = target.pixiContainer;
      const baseDuration = WebGAL.gameplay.pixiStage!.frameDuration;
      const currentAddOplityDelta = (duration / baseDuration) * delta;
      const increasement = 1 / currentAddOplityDelta;
      const decreasement = 5 / currentAddOplityDelta;
      if (container.alpha < 1) {
        container.alpha += increasement;
      }
      // @ts-ignore
      if (container.blur < 5) {
        // @ts-ignore
        container.blur += decreasement;
      }
    }
  }

  return {
    setStartState,
    setEndState,
    tickerFunc,
  };
}

```

# `/packages/webgal/src/Core/controller/stage/pixi/animations/timeline.ts`

This is a JavaScript class that defines a motion animation using GSAP. The animation has a simple setup, with a time-based rotation, blur, and scaling effect, as well as a requirement for the animation to fill the entire screen.

The animation has a single timeline, which is defined using GSAP's Timeline API. This timeline contains several timings for scaling, position, rotation, and blur. The animation starts from the original position, with a rotation of 0 degrees, and a scale of 1. The animation ends when the screen is fully saturated with color.

The animation works by updating the animation's position, rotation, blur, and scale based on the current time. This is done in a loop that executes once per frame.

The `setEndState` function is used to set the animation's end state. This function resets the animation's position, rotation, blur, and scale to their original values, and sets the animation to fill the entire screen.

The `getEndFilterEffect` function is used to retrieve the end filter effect of the animation, which is a combination of the required animation properties.

The example animation in the text above demonstrates how to use GSAP to create a simple, yet effected animation by setting the required properties for the animation and using the `getEndFilterEffect` function to retrieve the end effect of the animation.


```js
import { ITransform } from '@/store/stageInterface';
import { gsap } from 'gsap';
import { WebGAL } from '@/Core/WebGAL';

/**
 * 动画创建模板
 * @param timeline
 * @param targetKey 作用目标
 * @param duration 持续时间
 */
export function generateTimelineObj(
  timeline: Array<ITransform & { duration: number }>,
  targetKey: string,
  duration: number,
) {
  const target = WebGAL.gameplay.pixiStage!.getStageObjByKey(targetKey);
  let gsapTimeline1: gsap.core.Timeline | null = gsap.timeline();
  let gsapTimeline2: gsap.core.Timeline | null = gsap.timeline();
  let gsapTimeline3: gsap.core.Timeline | null = gsap.timeline();
  let gsapTimeline4: gsap.core.Timeline | null = gsap.timeline();
  let gsapTimeline5: gsap.core.Timeline | null = gsap.timeline();
  const gsapTimelines = [gsapTimeline1, gsapTimeline2, gsapTimeline3, gsapTimeline4, gsapTimeline5];
  for (const gsapEffect of timeline) {
    const gsapEffectDuration = gsapEffect.duration;
    if (target?.pixiContainer) {
      gsapTimeline1.to(target.pixiContainer, {
        alpha: gsapEffect.alpha,
        rotation: gsapEffect.rotation,
        blur: gsapEffect.blur,
        duration: gsapEffectDuration,
      });
      gsapTimeline2.to(target.pixiContainer.scale, {
        ...gsapEffect.scale,
        duration: gsapEffectDuration,
      });
      gsapTimeline3.to(target.pixiContainer, {
        ...gsapEffect.position,
        duration: gsapEffectDuration,
      });
      /**
       * filters
       */
      const { alpha, rotation, blur, duration, scale, position, pivot, ...rest } = gsapEffect;
      gsapTimeline4.to(target.pixiContainer, {
        ...rest,
        duration: gsapEffectDuration,
      });
    }
  }

  /**
   * 在此书写为动画设置初态的操作
   */
  function setStartState() {}

  /**
   * 在此书写为动画设置终态的操作
   */
  function setEndState() {
    for (const gsaptimeline of gsapTimelines) {
      if (gsaptimeline) {
        gsaptimeline.seek(duration);
        gsaptimeline.kill();
      }
    }
    gsapTimeline1 = null;
    gsapTimeline2 = null;
    gsapTimeline3 = null;
    gsapTimeline4 = null;
    gsapTimeline5 = null;
  }

  /**
   * 在此书写动画每一帧执行的函数
   * @param delta
   */
  function tickerFunc(delta: number) {}

  function getEndFilterEffect() {
    const gsapEffect = timeline[timeline.length - 1];
    const { alpha, rotation, blur, duration, scale, position, pivot, ...rest } = gsapEffect;
    return rest;
  }

  return {
    setStartState,
    setEndState,
    tickerFunc,
    getEndFilterEffect,
  };
}

```

# `/packages/webgal/src/Core/controller/stage/pixi/animations/universalSoftIn.ts`

该代码定义了一个名为 `generateUniversalSoftInAnimationObj` 的函数，它接受两个参数：`targetKey` 和 `duration`。函数的主要作用是生成一个通用软阴影在动画中效果的物体对象。

函数内部先创建一个通用的初始状态，然后设置通用状态。接着，定义了一个名为 `tick` 的函数，用于在每一帧执行动画。在 `tick` 函数中，先获取传入的 `target` 对象，然后通过 `target.pixiContainer.alpha` 属性来设置物体的透明度。通过 `WebGAL.gameplay.pixiStage!.frameDuration` 获取到每帧的动画时间，然后根据 `increasement` 和 `decreasement` 变量来控制物体的透明度变化速度。最后，在该函数中，如果 `target` 对象还在，则设置物体的初始透明度为 `0`，否则设置物体的透明度为 `1`。

函数返回了一个包含 `setStartState`、`setEndState` 和 `tickerFunc` 的对象，它们分别用于设置物体的起始状态、结束状态以及每一帧的动画执行函数。


```js
import { WebGAL } from '@/Core/WebGAL';

export function generateUniversalSoftInAnimationObj(targetKey: string, duration: number) {
  const target = WebGAL.gameplay.pixiStage!.getStageObjByKey(targetKey);

  // 先设置一个通用的初态

  // TODO：通用初态设置
  /**
   * 在此书写为动画设置初态的操作
   */
  function setStartState() {
    if (target) {
      target.pixiContainer.alpha = 0;
    }
  }

  // TODO：通用终态设置
  /**
   * 在此书写为动画设置终态的操作
   */
  function setEndState() {
    if (target) {
      target.pixiContainer.alpha = 1;
    }
  }

  /**
   * 在此书写动画每一帧执行的函数
   * @param delta
   */
  function tickerFunc(delta: number) {
    if (target) {
      const sprite = target.pixiContainer;
      const baseDuration = WebGAL.gameplay.pixiStage!.frameDuration;
      const currentAddOplityDelta = (duration / baseDuration) * delta;
      const increasement = 1 / currentAddOplityDelta;
      // const decreasement = 5 / currentAddOplityDelta;
      if (sprite.alpha < 1) {
        sprite.alpha += increasement;
      }
    }
  }

  return {
    setStartState,
    setEndState,
    tickerFunc,
  };
}

```

# `/packages/webgal/src/Core/controller/stage/pixi/animations/universalSoftOff.ts`

这段代码定义了一个名为 `generateUniversalSoftOffAnimationObj` 的函数，它接受一个名为 `targetKey` 的字符串参数和一个名为 `duration` 的数字参数。函数返回一个对象，该对象包含一个 `setStartState` 的函数、一个 `setEndState` 的函数和一个名为 `tickerFunc` 的函数。

`setStartState` 函数用于设置动画的起始状态， `setEndState` 函数用于设置动画的结束状态， `tickerFunc` 函数用于在动画每一帧执行的函数，它根据传入的 `delta` 值来控制 `target` 对象（即动画的目标）的透明度变化速度。

这段代码的作用是定义了一个名为 `generateUniversalSoftOffAnimationObj` 的函数，用于生成一个通用软阴影的动画对象。这个动画对象可以在游戏中的不同场景之间创建和应用，可以设置动画的起始和结束状态，并且每帧都可以更新动画的状态。


```js
import { WebGAL } from '@/Core/WebGAL';

export function generateUniversalSoftOffAnimationObj(targetKey: string, duration: number) {
  const target = WebGAL.gameplay.pixiStage!.getStageObjByKey(targetKey);

  // 先设置一个通用的初态

  /**
   * 在此书写为动画设置初态的操作
   */
  function setStartState() {}

  /**
   * 在此书写为动画设置终态的操作
   */
  function setEndState() {
    if (target) target.pixiContainer.alpha = 0;
  }

  /**
   * 在此书写动画每一帧执行的函数
   * @param delta
   */
  function tickerFunc(delta: number) {
    if (target) {
      const sprite = target.pixiContainer;
      const baseDuration = WebGAL.gameplay.pixiStage!.frameDuration;
      const currentAddOplityDelta = (duration / baseDuration) * delta;
      const decreasement = 1 / currentAddOplityDelta;
      if (sprite.alpha > 0) {
        sprite.alpha -= decreasement;
      }
    }
  }

  return {
    setStartState,
    setEndState,
    tickerFunc,
  };
}

```

# `/packages/webgal/src/Core/controller/storage/fastSaveLoad.ts`

这段代码的主要作用是初始化一个名为`fastSaveGameKey`的变量，以及定义了两个变量`isFastSaveKey`和`lock`。`fastSaveGameKey`和`isFastSaveKey`用于标识快速保存游戏的key，而`lock`变量则用于同步快速保存游戏的状态，当`lock`为真时，`fastSaveGameKey`和`isFastSaveKey`将不会被创建，只有在快速保存游戏处于激活状态时才会创建。

此外，还引入了两个函数`setFastSave`和`getStorageAsync`，以及一些常见的JS类库`lodash`和`WebGAL`。`setFastSave`用于设置快速保存游戏的key和激活状态，`getStorageAsync`用于从本地存储中获取快速保存游戏的状态，并调用`loadGame`和`generateCurrentStageData`函数来完成快速保存游戏的过程。`cloneDeep`库则用于对对象进行深拷贝。


```js
import { webgalStore } from '@/store/store';
import { setFastSave } from '@/store/userDataReducer';
import { getStorageAsync, setStorageAsync } from '@/Core/controller/storage/storageController';
import { ISaveData } from '@/store/userDataInterface';
import { loadGameFromStageData } from '@/Core/controller/storage/loadGame';
import { generateCurrentStageData } from '@/Core/controller/storage/saveGame';
import cloneDeep from 'lodash/cloneDeep';
import { WebGAL } from '@/Core/WebGAL';

export let fastSaveGameKey = '';
export let isFastSaveKey = '';
let lock = true;

export function initKey() {
  lock = false;
  fastSaveGameKey = `FastSaveKey-${WebGAL.gameName}-${WebGAL.gameKey}`;
  isFastSaveKey = `FastSaveActive-${WebGAL.gameName}-${WebGAL.gameKey}`;
}

```

这段代码定义了一个名为 `fastSaveGame` 的函数，用于在客户端保存游戏数据以进行紧急回避。

该函数的主要作用是：

1. 生成当前游戏阶段的 `saveData` 数据，并将其克隆成一个新的 `newSaveData` 数据。
2. 将新的 `newSaveData` 数据存储到本地存储的 `fastSaveGameKey` 键中，并设置 `isFastSaveKey` 为 `true`。
3. 将当前游戏阶段的 `runtime_currentSceneData.currentSentenceId` 存储为 `runtime_currentSceneData.currentSentenceId` 的 `JSON.stringify()` 版本，并将其作为参数传递给 `webgalStore.dispatch(setFastSave(newSaveData))` 函数。
4. 调用 `setStorageAsync()` 函数，将 `newSaveData` 数据存储到本地硬盘上。

该函数的作用是在紧急回避时快速保存游戏数据，以便在需要时可以重新加载游戏数据。由于该函数使用了 `localStorage` 和 `webgalStore` 存储数据，因此其影响可能会比正式的 `saveData` 数据存储方式更小，但是也可能会受到一些限制，例如本地存储空间的问题以及不同浏览器对该函数的执行的差异。


```js
/**
 * 用于紧急回避时的数据存储 & 快速保存
 */
export async function fastSaveGame() {
  const saveData: ISaveData = generateCurrentStageData(-1);
  const newSaveData = cloneDeep(saveData);
  // localStorage.setItem(fastSaveGameKey, JSON.stringify(newSaveData));
  // localStorage.setItem(isFastSaveKey, JSON.stringify(true));
  // localStorage.setItem('currentSentenceId', JSON.stringify(runtime_currentSceneData.currentSentenceId));
  // await localforage.setItem(fastSaveGameKey, newSaveData);
  // await localforage.setItem(isFastSaveKey, true);
  webgalStore.dispatch(setFastSave(newSaveData));
  await setStorageAsync();
}

```

这段代码是一个javascript文件，包含两个函数：hasFastSaveRecord 和 loadFastSaveGame。这两个函数的作用如下：

1. hasFastSaveRecord：
这个函数判断是否有无存储紧急回避时的数据。首先，它使用localforage.getItem(isFastSaveKey)方法获取一个与“isFastSaveKey”相关的存储键（可能是一个键值对）。然后，它使用getStorageAsync()方法获取一个存储库（如本地存储或远程存储）。最后，它使用webgalStore.getState().userData.quickSaveData判断快速保存数据是否存在于本地存储中。如果快速保存数据存在于本地存储中，那么函数返回true，否则返回false。

2. loadFastSaveGame：
这个函数加载紧急回避时的数据。它首先使用localforage.getItem(fastSaveGameKey)方法获取一个与“fastSaveGameKey”相关的存储键。然后，它使用getStorageAsync()方法获取一个存储库。如果获取到的存储键存在，它将调用loadGameFromStageData(loadFile)函数，使用从该存储键返回的ISaveData对象加载数据。

快速保存数据存储在localforage.storage.文件夹中，可能具有不同的键名。


```js
/**
 * 判断是否有无存储紧急回避时的数据
 */
export async function hasFastSaveRecord() {
  // return await localforage.getItem(isFastSaveKey);
  await getStorageAsync();
  return webgalStore.getState().userData.quickSaveData !== null;
}

/**
 * 加载紧急回避时的数据
 */
export async function loadFastSaveGame() {
  // 获得存档文件
  // const loadFile: ISaveData | null = await localforage.getItem(fastSaveGameKey);
  await getStorageAsync();
  const loadFile: ISaveData | null = webgalStore.getState().userData.quickSaveData;
  if (!loadFile) {
    return;
  }
  loadGameFromStageData(loadFile);
}

```

这段代码定义了一个名为 `removeFastSaveGameRecord` 的函数，它的作用是移除存储在 localStorage 和 webgalStore 中的快速保存游戏记录数据。

具体来说，它执行以下操作：

1. 通过调用 `webgalStore.dispatch(setFastSave(null))` 方法，将一个对象 `null` 传递给 `webgalStore.dispatch` 函数，这个对象代表一个空对象，代表移除所有快速保存的游戏记录数据。

2. 通过调用 `setStorageAsync()` 函数，将快速保存的游戏记录数据存储到 localStorage 中。

3. 通过调用 `localforage.setItem(isFastSaveKey, false)` 和 `localforage.setItem(fastSaveGameKey, null)` 方法，将快速保存的游戏记录数据存储到 webgalStore 中的 `fastSaveGameKey` 和 `isFastSaveKey` 对应的存储位置。

由于该函数的实现没有返回任何值，因此它的作用是移除快速保存的游戏记录数据，从而使得相关游戏无法被快速保存。


```js
/**
 * 移除紧急回避的数据
 */
export async function removeFastSaveGameRecord() {
  webgalStore.dispatch(setFastSave(null));
  await setStorageAsync();
  // await localforage.setItem(isFastSaveKey, false);
  // await localforage.setItem(fastSaveGameKey, null);
}

```

# `/packages/webgal/src/Core/controller/storage/jumpFromBacklog.ts`

这段代码的主要作用是创建一个WebGAL游戏引擎的子应用，用于在游戏执行过程中获取和编辑游戏中的场景、角色、音频等信息，并将它们以友好的格式存储到本地或在线数据库中。以下是这段代码的功能和模块：

1. 引入了需要用到的库和模块，包括Logger、SceneFetcher、SceneParser、IStageState、WebGAL store、stageReducer、GUI教育者、Controller和游戏引擎相关模块等。

2. 定义了场景类型（StageType）以及场景状态（StageState）。场景类型有五种：场景、主场景、教育场景、子场景和游戏场景。场景状态包含了舞台、角色、音频、动画和脚本等信息。

3. 导入并导入了SceneFetcher、SceneParser和WebGAL store，分别用于获取和解析游戏场景、场景数据和游戏数据。

4. 导入并导入了IStageState，用于在场景中保存和读取游戏中的信息，包括舞台状态、角色状态、音频等等。

5. 导入并导入了WebGAL store，用于在游戏执行过程中与WebGAL游戏服务器通信，包括登录、游戏更新和断开连接等操作。

6. 导入并导入了stageReducer，用于在场景reducer中处理游戏场景的变更，包括保存已编辑的场景和更新游戏场景状态。

7. 导入并导入了setVisibility，用于在游戏执行过程中设置游戏UI的可见性。

8. 导入并导入了runScript、stopAllPerform和cloneDeep，分别用于运行游戏脚本、停止游戏进程和克隆一个深度克制的游戏对象。

9. 导入并导入了鸳话李科从hash算法中导出的createEqual，用于在场景中创建相同的场景实例，以避免在更新游戏场景时产生意外的结果。

10. 导入并导入了scenePrefetcher，用于在游戏执行过程中保存和读取游戏中的偏好设置，包括字体、颜色、背景等等。

11. 导入并导入了WebGAL，用于在游戏执行过程中与WebGAL游戏服务器通信，包括登录、游戏更新和断开连接等操作。

12. 最后，定义了一些全局变量，包括当前场景、编辑过的场景、锁屏状态等，用于在游戏执行过程中进行管理和同步。


```js
import { logger } from '../../util/etc/logger';
import { sceneFetcher } from '../scene/sceneFetcher';
import { sceneParser } from '../../parser/sceneParser';
import { IStageState } from '@/store/stageInterface';
import { webgalStore } from '@/store/store';
import { resetStageState } from '@/store/stageReducer';
import { setVisibility } from '@/store/GUIReducer';
import { runScript } from '@/Core/controller/gamePlay/runScript';
import { stopAllPerform } from '@/Core/controller/gamePlay/stopAllPerform';
import cloneDeep from 'lodash/cloneDeep';
import uniqWith from 'lodash/uniqWith';
import { scenePrefetcher } from '@/Core/util/prefetcher/scenePrefetcher';

import { WebGAL } from '@/Core/WebGAL';

```

This is a JavaScript function that performs SceneFetcher and SceneManager operations for a L� graduate show.

It takes a backlogFile object containing the LLS国民经济orneum scene information, and it returns a Promise that resolves a Scene object.

The function fetches a scene from the backlogFile's sceneUrl, and then it pushes it to the WebGAL sceneManager's sceneData.currentScene.

It also performs a few other operations:

* It extracts the current sentence Id and the current stage state from the backlogFile's scene information.
* It stops all ongoing performances and removes them from the sceneManager's sceneData.currentScene.subSceneList.
* It populates the WebGAL sceneManager's sceneData.currentScene.scenePrefetcher array with the extracted subSceneList.
* It marks the extracted subSceneList as unique and adds it to the WebGAL sceneManager's settledScenes array.
* It returns a Promise that resolves the scene object.


```js
/**
 * 恢复演出
 */
export const restorePerform = () => {
  const stageState = webgalStore.getState().stage;
  stageState.PerformList.forEach((e) => {
    runScript(e.script);
  });
};

/**
 * 从 backlog 跳转至一个先前的状态
 * @param index
 */
export const jumpFromBacklog = (index: number) => {
  const dispatch = webgalStore.dispatch;
  // 获得存档文件
  const backlogFile = WebGAL.backlogManager.getBacklog()[index];
  logger.debug('读取的backlog数据', backlogFile);
  // 重新获取并同步场景状态
  sceneFetcher(backlogFile.saveScene.sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(
      rawScene,
      backlogFile.saveScene.sceneName,
      backlogFile.saveScene.sceneUrl,
    );
    // 开始场景的预加载
    const subSceneList = WebGAL.sceneManager.sceneData.currentScene.subSceneList;
    WebGAL.sceneManager.settledScenes.push(WebGAL.sceneManager.sceneData.currentScene.sceneUrl); // 放入已加载场景列表，避免递归加载相同场景
    const subSceneListUniq = uniqWith(subSceneList); // 去重
    scenePrefetcher(subSceneListUniq);
  });
  WebGAL.sceneManager.sceneData.currentSentenceId = backlogFile.saveScene.currentSentenceId;
  WebGAL.sceneManager.sceneData.sceneStack = cloneDeep(backlogFile.saveScene.sceneStack);

  // 强制停止所有演出
  stopAllPerform();

  // 弹出backlog项目到指定状态
  for (let i = WebGAL.backlogManager.getBacklog().length - 1; i > index; i--) {
    WebGAL.backlogManager.getBacklog().pop();
  }

  // 要记录本句 Backlog
  WebGAL.backlogManager.isSaveBacklogNext = true;

  // 恢复舞台状态
  const newStageState: IStageState = cloneDeep(backlogFile.currentStageState);

  dispatch(resetStageState(newStageState));

  // 恢复演出
  setTimeout(restorePerform, 0);

  // 关闭backlog界面
  dispatch(setVisibility({ component: 'showBacklog', visibility: false }));

  // 重新显示 TextBox
  dispatch(setVisibility({ component: 'showTextBox', visibility: true }));
};

```

# `/packages/webgal/src/Core/controller/storage/loadGame.ts`

这段代码主要是一个数学应用程序，会计算在给定一组数字中的校正因子。具体来说，它实现了以下功能：

1. 导入必要的模块和接口，包括 `ISaveData` 接口、`logger`、`sceneFetcher`、`sceneParser`、`webgalStore`、`resetStageState`、`setVisibility`、`restorePerform`、`cloneDeep`、`uniqWith`、`scenePrefetcher` 和 `setEbg`。

2. 加载自定义的 `logger` 模块。

3. 导入 `WebGAL` 模块，以便在程序中使用 WebGL。

4. 定义了一个名为 `calculateCorrectionFactors` 的函数，它接受一个包含数字的数组作为参数。函数的作用是在给定一组数字中计算出校正因子。

5. 引入了两个自定义的函数 `resetStageState` 和 `setVisibility`，它们用于重置舞台状态和设置可见性。

6. 引入了一个名为 `restorePerform` 的函数，它的作用是恢复指定的表演。

7. 引入了一个名为 `stopAllPerform` 的函数，它的作用是停止所有正在进行的表演。

8. 导入 `cloneDeep` 函数，以便使用 Lodash 库中的 `cloneDeep` 方法。

9. 导入 `uniqWith` 函数，以便使用 Lodash 库中的 `uniqWith` 方法。

10. 导入 `sceneFetcher`、`sceneParser` 和 `webgalStore` 模块，以便从商店中获取场景、脚本和权重。

11. 导入 `setEbg` 函数，以便设置场景的背景颜色。

12. 导入 `prefetcher` 模块，以便预加载场景。

13. 定义了一个名为 `setPerform` 的函数，它的作用是设置指定的表演。

14. 定义了一个名为 `setScene` 的函数，它的作用是设置场景的显示设置。

15. 定义了一个名为 `setVisibility` 的函数，它的作用是设置是否可见或隐藏舞台。

16. 定义了一个名为 `resolveActivePerform` 的函数，它的作用是获取当前正在进行的表演。

17. 定义了一个名为 `getActivePerform` 的函数，它的作用是获取当前正在进行的表演。

18. 定义了一个名为 `addWeight` 的函数，它的作用是在场景中添加权重。

19. 定义了一个名为 `removeWeight` 的函数，它的作用是从场景中删除权重。

20. 定义了一个名为 `getWeight` 的函数，它的作用是获取场景中的权重。

21. 定义了一个名为 `setWebGL` 的函数，它的作用是设置 WebGL 的状态。

22. 定义了一个名为 `setWebGALog` 的函数，它的作用是设置 WebGL 的日志记录。

23. 定义了一个名为 `setSceneC癌细胞` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `sceneC致癌因子` 函数。

24. 定义了一个名为 `setScene旁观者` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `scene旁观者` 函数。

25. 定义了一个名为 `setScene简约` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `scene简约` 函数。

26. 定义了一个名为 `setScene 摄像机移动` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `scene 摄像机移动` 函数。

27. 定义了一个名为 `setScene 加载音效` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `scene 加载音效` 函数。

28. 定义了一个名为 `setScene 加载动画` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `scene 加载动画` 函数。

29. 定义了一个名为 `setScene 音量` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `scene 音量` 函数。

30. 定义了一个名为 `setScene 平静` 的函数，它的作用是设置场景中当前工作的场景，以便执行 `scene 平静` 函数。


```js
import { ISaveData } from '@/store/userDataInterface';
import { logger } from '../../util/etc/logger';
import { sceneFetcher } from '../scene/sceneFetcher';
import { sceneParser } from '../../parser/sceneParser';
import { webgalStore } from '@/store/store';
import { resetStageState } from '@/store/stageReducer';
import { setVisibility } from '@/store/GUIReducer';
import { restorePerform } from './jumpFromBacklog';
import { stopAllPerform } from '@/Core/controller/gamePlay/stopAllPerform';
import cloneDeep from 'lodash/cloneDeep';
import uniqWith from 'lodash/uniqWith';
import { scenePrefetcher } from '@/Core/util/prefetcher/scenePrefetcher';
import { setEbg } from '@/Core/util/setEbg';

import { WebGAL } from '@/Core/WebGAL';

```

This is a JavaScript function that performs a preload and restore operation for a WebGL scene.

It takes a `rawScene` object and an options object as input.

The function first parses the raw scene data and sets the current scene to be the parsed scene. It then starts the scene's preload and loading of subscenes.

It also creates a unique list of all the subscenes and pushes it to the `scenesToLoad` array.

It then returns the current scene's URL, the subscene list, and the subscene list unique.

It also performs the restore operation for the stage state, the setVisibility function, and sets the background image.

It is called by the `preload` method in the `WebGLRenderer` class and it is useful for preloading the scene before the user interacts with the UI.


```js
/**
 * 读取游戏存档
 * @param index 要读取的存档的档位
 */
export const loadGame = (index: number) => {
  const userDataState = webgalStore.getState().userData;
  // 获得存档文件
  const loadFile: ISaveData = userDataState.saveData[index];
  logger.debug('读取的存档数据', loadFile);
  // 加载存档
  loadGameFromStageData(loadFile);
};

export function loadGameFromStageData(stageData: ISaveData) {
  if (!stageData) {
    logger.info('暂无存档');
    return;
  }
  const loadFile = stageData;
  // 重新获取并同步场景状态
  sceneFetcher(loadFile.sceneData.sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(
      rawScene,
      loadFile.sceneData.sceneName,
      loadFile.sceneData.sceneUrl,
    );
    // 开始场景的预加载
    const subSceneList = WebGAL.sceneManager.sceneData.currentScene.subSceneList;
    WebGAL.sceneManager.settledScenes.push(WebGAL.sceneManager.sceneData.currentScene.sceneUrl); // 放入已加载场景列表，避免递归加载相同场景
    const subSceneListUniq = uniqWith(subSceneList); // 去重
    scenePrefetcher(subSceneListUniq);
  });
  WebGAL.sceneManager.sceneData.currentSentenceId = loadFile.sceneData.currentSentenceId;
  WebGAL.sceneManager.sceneData.sceneStack = cloneDeep(loadFile.sceneData.sceneStack);

  // 强制停止所有演出
  stopAllPerform();

  // 恢复backlog
  const newBacklog = loadFile.backlog;
  WebGAL.backlogManager.getBacklog().splice(0, WebGAL.backlogManager.getBacklog().length); // 清空原backlog
  for (const e of newBacklog) {
    WebGAL.backlogManager.getBacklog().push(e);
  }

  // 恢复舞台状态
  const newStageState = cloneDeep(loadFile.nowStageState);
  const dispatch = webgalStore.dispatch;
  dispatch(resetStageState(newStageState));

  // 恢复演出
  setTimeout(restorePerform, 0);
  // restorePerform();

  dispatch(setVisibility({ component: 'showTitle', visibility: false }));
  dispatch(setVisibility({ component: 'showMenuPanel', visibility: false }));
  /**
   * 恢复模糊背景
   */
  setEbg(webgalStore.getState().stage.bgName);
}

```

# `/packages/webgal/src/Core/controller/storage/saveGame.ts`

这段代码是一个JavaScript函数，它导入了多个模块和接口：

1. 引入了'../../util/etc/logger'模块的logger变量；
2. 引入了'@/store/userDataInterface'中的ISaveData接口；
3. 引入了'./storageController'中的syncStorageFast函数；
4. 引入了'@/store/store'中的webgalStore变量；
5. 引入了'@/store/userDataReducer'中的setUserData函数；
6. 引入了'lodash/cloneDeep'函数。

这个函数的作用是保存游戏数据。它接收一个参数'index'，表示要保存的游戏的档位。函数内部首先获取保存游戏的存储数据，然后使用生成当前阶段数据的函数，将当前的游戏数据与存储数据进行同步。接着将存储数据中的'currentStageData'键替换为当前的游戏数据，再将index作为键，存储到newSaveData中。最后调用WebGAL的'setUserData'函数，将新的saveData保存到服务器上的对应游戏存档中。

这个函数的实现需要依赖多个模块和接口，包括逻辑存储器、游戏数据生成器、游戏存储器等。


```js
import { logger } from '../../util/etc/logger';
import { ISaveData } from '@/store/userDataInterface';
import { syncStorageFast } from './storageController';
import { webgalStore } from '@/store/store';
import { setUserData } from '@/store/userDataReducer';
import cloneDeep from 'lodash/cloneDeep';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 保存游戏
 * @param index 游戏的档位
 */
export const saveGame = (index: number) => {
  const userDataState = webgalStore.getState().userData;
  const saveData: ISaveData = generateCurrentStageData(index);
  logger.debug('存档数据：', saveData);
  const newSaveData = cloneDeep(userDataState.saveData);
  logger.debug('newSaveData:', newSaveData);
  newSaveData[index] = saveData;
  webgalStore.dispatch(setUserData({ key: 'saveData', value: [...newSaveData] }));
  logger.debug('存档完成，存档结果：', newSaveData);
  syncStorageFast();
};

```

这段代码定义了一个名为generateCurrentStageData的函数，用于生成游戏当前档位的数据快照。

该函数需要两个参数，一个是游戏的档位（index），另一个是描述游戏当前状态的变量 stageState。函数内部首先获取保存的后台数据，然后使用 WebGLStorm 库中的一个名为 backlog 的函数将其从本地存储中克隆并缓存。接着，使用 canvas 元素创建一个画布，并将其绘制了游戏当前场景的缩略图，然后将这个缩略图保存为一个 WebP 格式的图片。

函数返回一个 ISaveData 类型的对象，包含游戏当前档位的状态、保存的后台数据以及保存的时间戳等信息。


```js
/**
 * 生成现在游戏的数据快照
 * @param index 游戏的档位
 */
export function generateCurrentStageData(index: number) {
  const stageState = webgalStore.getState().stage;
  const saveBacklog = cloneDeep(WebGAL.backlogManager.getBacklog());

  /**
   * 生成缩略图
   */

  const canvas: HTMLCanvasElement = document.getElementById('pixiCanvas')! as HTMLCanvasElement;
  const canvas2 = document.createElement('canvas');
  const context = canvas2.getContext('2d');
  canvas2.width = 480;
  canvas2.height = 270;
  context!.drawImage(canvas, 0, 0, 480, 270);
  const urlToSave = canvas2.toDataURL('image/webp', 0.5);
  canvas2.remove();
  const saveData: ISaveData = {
    nowStageState: cloneDeep(stageState),
    backlog: saveBacklog, // 舞台数据
    index: index, // 存档的序号
    saveTime: new Date().toLocaleDateString() + ' ' + new Date().toLocaleTimeString('chinese', { hour12: false }), // 保存时间
    // 场景数据
    sceneData: {
      currentSentenceId: WebGAL.sceneManager.sceneData.currentSentenceId, // 当前语句ID
      sceneStack: cloneDeep(WebGAL.sceneManager.sceneData.sceneStack), // 场景栈
      sceneName: WebGAL.sceneManager.sceneData.currentScene.sceneName, // 场景名称
      sceneUrl: WebGAL.sceneManager.sceneData.currentScene.sceneUrl, // 场景url
    },
    previewImage: urlToSave,
  };
  return saveData;
}

```

# `/packages/webgal/src/Core/controller/storage/storageController.ts`

这段代码的作用是实现了一个将用户数据存储到本地存储的功能。

具体来说，代码中引入了三个库：localforage、@/store/userDataInterface、@/store/store以及@/Core/WebGAL。其中，localforage库用于在本地存储中写入和读取数据，@/store/userDataInterface和@/store/store用于与redux store进行交互，@/Core/WebGAL库用于在网页上展示游戏数据。

代码中定义了一个名为setStorage的函数，该函数内部调用了debounce函数来确保写入本地存储时的性能。函数内部定义了一个定时器，每100毫秒将用户数据存储到本地存储中，并在写入成功后输出日志。

最后，import语句中定义了两个导入的函数，分别用于从@/store/userDataInterface和@/Core/WebGAL中获取用户数据，以及初始化redux store。


```js
import * as localforage from 'localforage';
import { IUserData } from '@/store/userDataInterface';
import { logger } from '../../util/etc/logger';
import { webgalStore } from '@/store/store';
import { initState, resetUserData } from '@/store/userDataReducer';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 写入本地存储
 */
export const setStorage = debounce(() => {
  const userDataState = webgalStore.getState().userData;
  localforage.setItem(WebGAL.gameKey, userDataState).then(() => {
    logger.info('写入本地存储');
  });
}, 100);

```

这段代码的作用是从本地存储获取用户数据。它使用了 localforage.js 库来获取游戏数据，并使用 debounce() 函数来平滑滚动调用。

具体来说，代码首先通过调用 `localforage.getItem(WebGAL.gameKey)` 获取游戏数据，然后使用 Promise 链式调用其回调函数，该回调函数接收到的参数 `newUserData` 是我们需要处理的用户数据。

接下来，代码会判断 `newUserData` 是否为空或 `checkUserDataProperty` 函数返回的结果是否为 `true`。如果是，那么代码会将本地存储的数据置为零，并调用 `setStorage` 函数将数据存储到本地。这样，如果本地存储的数据为空或者 `checkUserDataProperty` 的返回结果为 `false`，我们就需要重新获取数据并重新初始化。

代码还使用了 `webgalStore.dispatch(resetUserData(newUserData))` 来更新游戏数据的本地存储。这个 `resetUserData` 函数会将传入的用户数据重置为游戏中的初始数据，并将其存储到游戏客户端的本地存储中。

最后，代码还定义了一个延迟为 100ms 的延迟，用于平滑滚动调用 `localforage.getItem(WebGAL.gameKey)`。这个延迟在代码中是 `, 100`。


```js
/**
 * 从本地存储获取数据
 */
export const getStorage = debounce(() => {
  localforage.getItem(WebGAL.gameKey).then((newUserData) => {
    // 如果没有数据或者属性不完全，重新初始化
    if (!newUserData || !checkUserDataProperty(newUserData)) {
      logger.warn('现在重置数据');
      setStorage();
      return;
    }
    webgalStore.dispatch(resetUserData(newUserData as IUserData));
  });
}, 100);

```

这段代码定义了一个名为 `debounce` 的函数，用于阻止一个函数在一段时间内连续触发。该函数接受两个参数：要执行的函数 `func` 和等待时间 `wait`。

`debounce` 的实现主要分为两个步骤。首先，定义了一个名为 `timeout` 的变量，用于存储触发函数的时间戳。然后，定义了一个名为 `context` 的函数，该函数接受任意数量的参数，并返回它们的 `K` 值。

`context` 函数的核心是 `setTimeout` 函数的克隆，用于在 `wait` 毫秒后触发 `func`。`timeout` 变量被重置，并在 `wait` 毫秒后再次被重置。这样，即使 `wait` 毫秒后的函数触发被阻止了，`timeout` 中的缓存仍然会保留，以便下一个触发时可以更快地重新触发。

最后，`debounce` 函数返回了一个 `context` 函数，可以在需要时返回，例如在 `debounce` 的回调函数中使用。


```js
/**
 * 防抖函数
 * @param func 要执行的函数
 * @param wait 防抖等待时间
 */
function debounce<T, K>(func: (...args: T[]) => K, wait: number) {
  let timeout: ReturnType<typeof setTimeout>;

  function context(...args: T[]): K {
    clearTimeout(timeout);
    let ret!: K;
    timeout = setTimeout(() => {
      ret = func.apply(context, args);
    }, wait);
    return ret;
  }

  return context;
}

```

这段代码定义了一个名为 `syncStorageFast` 的函数，它使用了 webglMemStore 模块的 `setItem` 和 `getItem` 方法，以及一些其他的逻辑调用。

首先，它从名为 `userData` 的状态中获取了存储用户数据的存储库（也就是一个 JavaScript 对象），并将其存储到了名为 `WebGAL.gameKey` 的键中。

然后，它使用 `localforage.setItem` 方法将用户数据存储到了 `WebGAL.gameKey` 键中的某个地方，然后回调该方法。

如果用户数据存储成功，它会执行一系列操作，包括将新的用户数据同步到本地存储中，使用 `webgalStore.dispatch` 方法调用一个名为 `resetUserData` 的函数，将新用户数据存储到本地存储中，然后输出一条日志信息。

最后，它定义了一个 `syncStorageFast` 函数，它会使用 `logger.info` 输出一条日志信息，然后调用 `setItem` 和 `getItem` 方法来同步用户数据和本地存储。


```js
export const syncStorageFast = () => {
  const userDataState = webgalStore.getState().userData;
  localforage.setItem(WebGAL.gameKey, userDataState).then(() => {
    localforage.getItem(WebGAL.gameKey).then((newUserData) => {
      // 如果没有数据，初始化
      if (!newUserData) {
        setStorage();
        return;
      }
      webgalStore.dispatch(resetUserData(newUserData as IUserData));
    });
    logger.info('同步本地存储');
  });
};

```

该代码定义了一个名为 `checkUserDataProperty` 的函数，它会接收一个名为 `userData` 的任意对象参数。该函数的作用是检查 `userData` 对象中是否定义了所有的 `initState` 对象中的属性。如果 `userData` 对象中缺少任何一个属性，那么函数的返回值将为 `false`。

该函数还有一个名为 `setStorageAsync` 的 exported 函数，它会使用 `localforage` 库将一个名为 `userData` 的游戏状态对象存储到本地。


```js
/**
 * 检查用户数据属性是否齐全
 * @param userData 需要检查的数据
 */
function checkUserDataProperty(userData: any) {
  let result = true;
  for (const key in initState) {
    if (!userData.hasOwnProperty(key)) {
      result = false;
    }
  }
  return result;
}

export async function setStorageAsync() {
  const userDataState = webgalStore.getState().userData;
  return await localforage.setItem(WebGAL.gameKey, userDataState);
}

```

这段代码定义了一个名为 `getStorageAsync` 的异步函数，它使用 `localforage` 库来获取游戏数据存储中的用户数据。如果用户数据不存在或数据无效，函数会将用户数据存储到本地，并使用 `webgalStore` 存储游戏的当前状态。如果用户数据有效，函数会将用户数据应用到游戏逻辑中。

具体来说，代码可以分为以下几个部分：

1. 定义了一个名为 `getStorageAsync` 的异步函数，它返回一个 Promise 对象，代表异步操作的结果。
2. 使用 `localforage.getItem` 方法获取游戏数据存储中的用户数据，并将其存储在 `newUserData` 变量中。
3. 如果 `newUserData` 存在，但 `checkUserDataProperty` 无法验证 `newUserData` 是否有效，那么函数会将用户数据存储到本地（使用 `webgalStore.getState().userData` 获取游戏状态），然后使用 `localforage.setItem` 方法将用户数据存储到游戏数据存储中。
4. 如果 `newUserData` 有效，那么函数会将用户数据应用到游戏逻辑中，并使用 `webgalStore.dispatch` 方法将 `resetUserData` 函数作为异步操作发送到游戏客户端。
5. 函数使用了 `logger.warn` 方法进行警告，当 `newUserData` 或 `checkUserDataProperty` 的任意一个失败时，会输出一条警告信息。


```js
export async function getStorageAsync() {
  const newUserData = await localforage.getItem(WebGAL.gameKey);
  if (!newUserData || !checkUserDataProperty(newUserData)) {
    const userDataState = webgalStore.getState().userData;
    logger.warn('现在重置数据');
    return await localforage.setItem(WebGAL.gameKey, userDataState);
  } else webgalStore.dispatch(resetUserData(newUserData as IUserData));
  return;
}

```

# `/packages/webgal/src/Core/gameScripts/bgm.ts`

This is a JavaScript module that exports a function called `bgm`, which is used to play a背景音乐 (BGM) in a Sentence scene. The function takes in a `sentence` argument, which is an object that contains the following properties:

* `content`: The URL of the BGM file.
* `name`: The name of the BGM file.
* `series`: The series of the BGM.
* `enter`: The Enter淡入时间，即背景音乐从静止状态到播放的比例。
* `volume`: The Volume音量比，即背景音乐的音量大小调整。

函数内部首先通过 `getSentenceArgByKey` 方法获取 `sentence` 中与 `unlockname` 和 `series` 相关的 arguments，然后将这些 arguments 发送给 `webgalStore.dispatch` 方法，将其存储到 `userData` 状态中。

接下来，调用 `playBgm` 函数播放BGM，并设置一些参数，包括：

* `url`: BGM文件的URL。
* `enter`: 淡入时间，以秒为单位。
* `volume`: BGM的音量比，范围为0到100。

函数的 `performName` 为空，因为它会在调用 `webgalStore.dispatch` 之后停止执行。

函数的 `duration` 为0，因为它不会让BGM停止。

函数的 `isHoldOn` 为真，因为函数已经获取到了控制权。

函数的 `stopFunction` 为空，因为函数没有停止函数。

函数的 `stopTimeout` 为 undefined，因为函数也没有设置超时停止。

函数返回一个执行概要，其中包括：

* `performName`: 表演名称，这个名称不会在控制台中显示。
* `duration`: 表演持续时间，这个时间不会根据BGM的播放而变化。
* `isHoldOn`: 将占用CPU的百分比。
* `stopFunction`: 停止函数，如果需要停止，这个函数会被调用。
* `blockingNext`: 阻止阻塞下一个循环的执行。
* `blockingAuto`: 阻止阻塞自动执行。
* `stopTimeout`: 如果设置了超时停止，这个参数就是`stopTimeout`。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { playBgm } from '@/Core/controller/stage/playBgm';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { webgalStore } from '@/store/store';
import { unlockBgmInUserData } from '@/store/userDataReducer';

/**
 * 播放一段bgm
 * @param sentence
 */
export const bgm = (sentence: ISentence): IPerform => {
  let url: string = sentence.content; // 获取bgm的url
  let name = '';
  let series = 'default';
  sentence.args.forEach((e) => {
    if (e.key === 'unlockname') {
      name = e.value.toString();
    }
    if (e.key === 'series') {
      series = e.value.toString();
    }
  });
  const enter = getSentenceArgByKey(sentence, 'enter'); // 获取bgm的淡入时间
  const volume = getSentenceArgByKey(sentence, 'volume'); // 获取bgm的音量比
  if (name !== '') webgalStore.dispatch(unlockBgmInUserData({ name, url, series }));
  playBgm(
    url,
    typeof enter === 'number' && enter >= 0 ? enter : 0, // 已正确设置淡入时间时，进行淡入
    typeof volume === 'number' && volume >= 0 && volume <= 100 ? volume : 100, // 已正确设置音量比时，进行音量调整
  );
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

# `/packages/webgal/src/Core/gameScripts/callSceneScript.ts`

该代码是一个函数，名为 `callSceneScript`，它接受一个参数 `sentence`，该参数是一个句子对象，包含了场景的名称、标签、描述等信息。函数的作用是调用一个场景，并在场景结束后回到调用这个场景的父场景。

具体来说，函数首先通过 `split` 方法将句子对象的内容按照 `/` 分割成多个场景名称，取最后一个场景名称作为场景名。然后调用 `callScene` 函数，将场景名和句子对象一起传递给该函数，同时传递给 `callScene` 的第二个参数 `sceneName`，以便在场景执行结束后回到调用这个场景的父场景。

函数的返回值是一个 `IPerform` 对象，该对象包含以下属性：

* `performName`：表演名称，即场景的名称。
* `duration`：表演持续时间，不包括在 `stop` 函数中。
* `isHoldOn`：锁定状态，用于判断是否允许用户锁定表演。
* `stopFunction`：当锁定表演时，用户需要点击的按钮或按键。
* `blockingNext`：锁定下一个允许用户锁定表演的按钮或按键。
* `blockingAuto`：自动锁定表演，即当锁定按钮或按键处于自动锁定状态时，自动锁定表演。
* `stopTimeout`：锁定的超时时间，如果 `stop` 函数被调用，将根据这个超时时间停止锁定。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { callScene } from '../controller/scene/callScene';

/**
 * 调用一个场景，在场景结束后回到调用这个场景的父场景。
 * @param sentence
 */
export const callSceneScript = (sentence: ISentence): IPerform => {
  const sceneNameArray: Array<string> = sentence.content.split('/');
  const sceneName = sceneNameArray[sceneNameArray.length - 1];
  callScene(sentence.content, sceneName);
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

# `/packages/webgal/src/Core/gameScripts/changeBg.ts`

这段代码的作用是创建一个名为 "scene1" 的场景对象，该场景对象属于一个名为 "scene" 的场景对象。这个场景对象包含了舞台上所有的参与者（表演者），以及用于控制表演者动作和状态的逻辑。

具体来说，这段代码做了以下几件事情：

1. 引入了 @/Core/controller/scene/sceneInterface 和 @/Core/Modules/perform/performInterface 两个模块，以便在场景中执行表演者的动作逻辑。
2. 引入了 @/Components/Stage/stage.module.scss 引入了一个名为 "stage" 的组件样式。
3. 导入了 webgalStore 去访问游戏中的库存数据，以及 stageReducer 和 userDataReducer 去处理库存数据的结果。
4. 导入了 getRandomPerformName 函数，用于从给定的选项中随机选择一个表演者的名字。
5. 导入了 styles，用于在场景中应用样式的功能。
6. 导入了 {...} 表示法，用于将多个组件组合成一个复杂对象。
7. 导入了 {ISentence, IPerform, ITransform, logger} 变量，分别表示场景中的句子、表演者和转换函数，以及用于输出日志信息的日志工具。
8. 导入了 generateTransformAnimationObj 和 IUserAnimation 两个函数，用于生成动画对象和用户动画。
9. 导入了 updateCurrentEffects 和 cloneDeep 两个函数，用于更新当前舞台效果和克隆对象。
10. 导入了 getAnimateDuration 函数，用于获取动画的持续时间。

最终，这段代码会创建一个名为 "scene1" 的场景对象，并在场景中执行表演者的动作逻辑。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
// import {getRandomPerformName} from '../../../util/getRandomPerformName';
import styles from '../../Components/Stage/stage.module.scss';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { unlockCgInUserData } from '@/store/userDataReducer';
import { logger } from '@/Core/util/etc/logger';
import { ITransform } from '@/store/stageInterface';
import { generateTransformAnimationObj } from '@/Core/gameScripts/function/generateTransformAnimationObj';
import { IUserAnimation } from '@/Core/Modules/animations';
import { updateCurrentEffects } from '../controller/stage/pixi/PixiController';
import cloneDeep from 'lodash/cloneDeep';
import { getAnimateDuration } from '@/Core/Modules/animationFunctions';
```



This is a JavaScript function that creates a simple animation effect for a background image. It takes a single argument, `animationObj`, which is an object containing the animation to apply.

The function first applies any default transformations to the image before applying the animation. If the animation has already been applied, it is skipped and the image is considered ready for display.

If the animation does not have a `name` parameter, it will be assigned a default name based on the hash value. The animation will perform a fade-in effect, lasting for the specified duration.

The animation will only perform the enter and exit animation for the `bg-main` key, according to the order in which the enter and exit animations are set.

Overall, this function is a simple utility for creating a background animation for a given image.


```js
import { WebGAL } from '@/Core/WebGAL';

/**
 * 进行背景图片的切换
 * @param sentence 语句
 * @return {IPerform}
 */
export const changeBg = (sentence: ISentence): IPerform => {
  const url = sentence.content;
  let name = '';
  let series = 'default';
  sentence.args.forEach((e) => {
    if (e.key === 'unlockname') {
      name = e.value.toString();
    }
    if (e.key === 'series') {
      series = e.value.toString();
    }
  });

  const dispatch = webgalStore.dispatch;
  if (name !== '') dispatch(unlockCgInUserData({ name, url, series }));

  /**
   * 删掉相关 Effects，因为已经移除了
   */
  const prevEffects = webgalStore.getState().stage.effects;
  const newEffects = cloneDeep(prevEffects);
  const index = newEffects.findIndex((e) => e.target === `bg-main`);
  if (index >= 0) {
    newEffects.splice(index, 1);
  }
  updateCurrentEffects(newEffects);

  // 处理 transform 和 默认 transform
  const transformString = getSentenceArgByKey(sentence, 'transform');
  let duration = getSentenceArgByKey(sentence, 'duration');
  if (!duration || typeof duration !== 'number') {
    duration = 1000;
  }
  let animationObj: (ITransform & {
    duration: number;
  })[];
  if (transformString) {
    try {
      const frame = JSON.parse(transformString.toString()) as ITransform & { duration: number };
      animationObj = generateTransformAnimationObj('bg-main', frame, duration);
      // 因为是切换，必须把一开始的 alpha 改为 0
      animationObj[0].alpha = 0;
      const animationName = (Math.random() * 10).toString(16);
      const newAnimation: IUserAnimation = { name: animationName, effects: animationObj };
      WebGAL.animationManager.addAnimation(newAnimation);
      duration = getAnimateDuration(animationName);
      WebGAL.animationManager.nextEnterAnimationName.set('bg-main', animationName);
    } catch (e) {
      // 解析都错误了，歇逼吧
      applyDefaultTransform();
    }
  } else {
    applyDefaultTransform();
  }

  function applyDefaultTransform() {
    // 应用默认的
    const frame = {};
    animationObj = generateTransformAnimationObj('bg-main', frame as ITransform & { duration: number }, duration);
    // 因为是切换，必须把一开始的 alpha 改为 0
    animationObj[0].alpha = 0;
    const animationName = (Math.random() * 10).toString(16);
    const newAnimation: IUserAnimation = { name: animationName, effects: animationObj };
    WebGAL.animationManager.addAnimation(newAnimation);
    duration = getAnimateDuration(animationName);
    WebGAL.animationManager.nextEnterAnimationName.set('bg-main', animationName);
  }

  // 应用动画的优先级更高一点
  if (getSentenceArgByKey(sentence, 'enter')) {
    WebGAL.animationManager.nextEnterAnimationName.set('bg-main', getSentenceArgByKey(sentence, 'enter')!.toString());
    duration = getAnimateDuration(getSentenceArgByKey(sentence, 'enter')!.toString());
  }
  if (getSentenceArgByKey(sentence, 'exit')) {
    WebGAL.animationManager.nextExitAnimationName.set('bg-main-off', getSentenceArgByKey(sentence, 'exit')!.toString());
    duration = getAnimateDuration(getSentenceArgByKey(sentence, 'exit')!.toString());
  }
  dispatch(setStage({ key: 'bgName', value: sentence.content }));

  return {
    performName: 'none',
    duration,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => true,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```

# `/packages/webgal/src/Core/gameScripts/changeFigure.ts`

这段代码的作用是更改场景中的立绘。具体来说，它会根据场景句子（对象）的变化，更新表现层的时间和渲染顺序，从而实现场景的快进、慢放、暂停和倒退。

具体实现步骤如下：

1. 从 `@/Core/controller/scene/sceneInterface` 包中导入 `ISentence` 和 `IPerform` 类，以及 `@/Core/store/stageReducer` 和 `@/Core/util/getSentenceArg` 方法。
2. 从 `@/Core/Modules/perform/performInterface` 包中导入 `webgalStore` 方法。
3. 从 `@/store/stageReducer` 包中导入 `updateCurrentEffects` 和 `generateTransformAnimationObj` 方法。
4. 从 `@/Core/util/cloneDeep` 包中导入 `cloneDeep` 方法。
5. 从 `@/Core/util/gameAssetsAccess/assetSetter` 包中导入 `assetSetter` 和 `fileType` 方法。
6. 从 `@/Core/util/etc/logger` 包中导入 `logger` 函数。
7. 从 `@/Core/Modules/animationFunctions` 包中导入 `generateTransformAnimationObj` 方法。
8. 从 `@/Core/WebGAL` 包中导入 `WebGAL` 类。
9. 创建一个 `ISentence` 对象，用来表示场景中的对象。
10. 创建一个 `IPerform` 对象，用来表示场景中的表现层。
11. 调用 `updateCurrentEffects` 方法，根据场景句子中的条件，更新表现层的时间和渲染顺序。
12. 调用 `generateTransformAnimationObj` 方法，生成一个新的动画对象。
13. 调用 `assetSetter` 方法，设置场景中的动画资源。
14. 调用 `logger` 函数，记录当前场景的渲染日志。
15. 调用 `generateTransformAnimationObj` 方法，生成一个新的动画对象。
16. 将生成的动画对象添加到表现层的时间栈中。
17. 返回生成的动画对象。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';
import { IPerform } from '@/Core/Modules/perform/performInterface';
import { webgalStore } from '@/store/store';
import { setStage } from '@/store/stageReducer';
import { updateCurrentEffects } from '../controller/stage/pixi/PixiController';
import cloneDeep from 'lodash/cloneDeep';
import { getSentenceArgByKey } from '@/Core/util/getSentenceArg';
import { IStageState, ITransform } from '@/store/stageInterface';
import { IUserAnimation } from '@/Core/Modules/animations';
import { generateTransformAnimationObj } from '@/Core/gameScripts/function/generateTransformAnimationObj';
import { assetSetter, fileType } from '@/Core/util/gameAssetsAccess/assetSetter';
import { logger } from '@/Core/util/etc/logger';
import { getAnimateDuration } from '@/Core/Modules/animationFunctions';
import { WebGAL } from '@/Core/WebGAL';
/**
 * 更改立绘
 * @param sentence 语句
 */
```

The code appears to be a JavaScript function called `performAction`. It takes a single argument, `action`, which is a string indicating the action to perform.

The function first clones a group of figures using the `cloneDeep` function. Then, it checks the index of the current free figure and, if it exists, sets its base position and name to the values provided in the `action` argument.

If the index is not found, the function adds a new free figure with the specified properties and sets its base position and name to the default values.

Finally, the function sets or dispatches the animation and motion names and stages, depending on whether the `motion` field is set.


```js
export const changeFigure = (sentence: ISentence): IPerform => {
  // 根据参数设置指定位置
  let pos: 'center' | 'left' | 'right' = 'center';
  let content = sentence.content;
  let isFreeFigure = false;
  let motion = '';
  let key = '';
  let duration = 500;
  let mouthOpen = '';
  let mouthClose = '';
  let mouthHalfOpen = '';
  let eyesOpen = '';
  let eyesClose = '';
  let animationFlag: any = '';
  let mouthAnimationKey: any = 'mouthAnimation';
  let eyesAnimationKey: any = 'blinkAnimation';
  const dispatch = webgalStore.dispatch;

  for (const e of sentence.args) {
    switch (e.key) {
      case 'left':
        if (e.value === true) {
          pos = 'left';
          mouthAnimationKey = 'mouthAnimationLeft';
          eyesAnimationKey = 'blinkAnimationLeft';
        }
        break;
      case 'right':
        if (e.value === true) {
          pos = 'right';
          mouthAnimationKey = 'mouthAnimationRight';
          eyesAnimationKey = 'blinkAnimationRight';
        }
        break;
      case 'clear':
        if (e.value === true) {
          content = '';
        }
        break;
      case 'id':
        isFreeFigure = true;
        key = e.value.toString();
        break;
      case 'motion':
        motion = e.value.toString();
        break;
      case 'mouthOpen':
        mouthOpen = e.value.toString();
        mouthOpen = assetSetter(mouthOpen, fileType.figure);
        break;
      case 'mouthClose':
        mouthClose = e.value.toString();
        mouthClose = assetSetter(mouthClose, fileType.figure);
        break;
      case 'mouthHalfOpen':
        mouthHalfOpen = e.value.toString();
        mouthHalfOpen = assetSetter(mouthHalfOpen, fileType.figure);
        break;
      case 'eyesOpen':
        eyesOpen = e.value.toString();
        eyesOpen = assetSetter(eyesOpen, fileType.figure);
        break;
      case 'eyesClose':
        eyesClose = e.value.toString();
        eyesClose = assetSetter(eyesClose, fileType.figure);
        break;
      case 'animationFlag':
        animationFlag = e.value.toString();
        break;
      case 'none':
        content = '';
        break;
      default:
        break;
    }
  }

  const id = key ? key : `fig-${pos}`;

  const currentFigureAssociatedAnimation = webgalStore.getState().stage.figureAssociatedAnimation;
  const filteredFigureAssociatedAnimation = currentFigureAssociatedAnimation.filter((item) => item.targetId !== id);
  const newFigureAssociatedAnimationItem = {
    targetId: id,
    animationFlag: animationFlag,
    mouthAnimation: {
      open: mouthOpen,
      close: mouthClose,
      halfOpen: mouthHalfOpen,
    },
    blinkAnimation: {
      open: eyesOpen,
      close: eyesClose,
    },
  };
  filteredFigureAssociatedAnimation.push(newFigureAssociatedAnimationItem);
  dispatch(setStage({ key: 'figureAssociatedAnimation', value: filteredFigureAssociatedAnimation }));

  /**
   * 删掉相关 Effects，因为已经移除了
   */
  const prevEffects = webgalStore.getState().stage.effects;
  const newEffects = cloneDeep(prevEffects);
  const index = newEffects.findIndex((e) => e.target === `fig-${pos}` || e.target === `${key}`);
  if (index >= 0) {
    newEffects.splice(index, 1);
  }
  updateCurrentEffects(newEffects);
  const setAnimationNames = (key: string, sentence: ISentence) => {
    // 处理 transform 和 默认 transform
    const transformString = getSentenceArgByKey(sentence, 'transform');
    const durationFromArg = getSentenceArgByKey(sentence, 'duration');
    if (durationFromArg && typeof durationFromArg === 'number') {
      duration = durationFromArg;
    }
    let animationObj: (ITransform & {
      duration: number;
    })[];
    if (transformString) {
      console.log(transformString);
      try {
        const frame = JSON.parse(transformString.toString()) as ITransform & { duration: number };
        animationObj = generateTransformAnimationObj(key, frame, duration);
        // 因为是切换，必须把一开始的 alpha 改为 0
        animationObj[0].alpha = 0;
        const animationName = (Math.random() * 10).toString(16);
        const newAnimation: IUserAnimation = { name: animationName, effects: animationObj };
        WebGAL.animationManager.addAnimation(newAnimation);
        duration = getAnimateDuration(animationName);
        WebGAL.animationManager.nextEnterAnimationName.set(key, animationName);
      } catch (e) {
        // 解析都错误了，歇逼吧
        applyDefaultTransform();
      }
    } else {
      applyDefaultTransform();
    }

    function applyDefaultTransform() {
      // 应用默认的
      const frame = {};
      animationObj = generateTransformAnimationObj(key, frame as ITransform & { duration: number }, duration);
      // 因为是切换，必须把一开始的 alpha 改为 0
      animationObj[0].alpha = 0;
      const animationName = (Math.random() * 10).toString(16);
      const newAnimation: IUserAnimation = { name: animationName, effects: animationObj };
      WebGAL.animationManager.addAnimation(newAnimation);
      duration = getAnimateDuration(animationName);
      WebGAL.animationManager.nextEnterAnimationName.set(key, animationName);
    }
    const enterAnim = getSentenceArgByKey(sentence, 'enter');
    const exitAnim = getSentenceArgByKey(sentence, 'exit');
    if (enterAnim) {
      WebGAL.animationManager.nextEnterAnimationName.set(key, enterAnim.toString());
      duration = getAnimateDuration(enterAnim.toString());
    }
    if (exitAnim) {
      WebGAL.animationManager.nextExitAnimationName.set(key + '-off', exitAnim.toString());
      duration = getAnimateDuration(exitAnim.toString());
    }
  };
  if (isFreeFigure) {
    const currentFreeFigures = webgalStore.getState().stage.freeFigure;

    /**
     * 重设
     */
    const index = currentFreeFigures.findIndex((figure) => figure.key === key);
    const newFreeFigure = cloneDeep(currentFreeFigures);
    if (index >= 0) {
      newFreeFigure[index].basePosition = pos;
      newFreeFigure[index].name = content;
    } else {
      // 新加
      if (content !== '') newFreeFigure.push({ key, name: content, basePosition: pos });
    }
    setAnimationNames(key, sentence);
    dispatch(setStage({ key: 'freeFigure', value: newFreeFigure }));
  } else {
    const positionMap = {
      center: 'fig-center',
      left: 'fig-left',
      right: 'fig-right',
    };
    const dispatchMap = {
      center: 'figName',
      left: 'figNameLeft',
      right: 'figNameRight',
    };

    key = positionMap[pos];
    setAnimationNames(key, sentence);
    dispatch(setStage({ key: dispatchMap[pos] as keyof IStageState, value: content }));
  }
  if (motion) {
    const index = webgalStore.getState().stage.live2dMotion.findIndex((e) => e.target === key);
    let motionArr = webgalStore.getState().stage.live2dMotion;
    if (index <= 0) {
      // 应用一个新的 motion
      motionArr = [...webgalStore.getState().stage.live2dMotion, { target: key, motion }];
    } else {
      motionArr[index].motion = motion;
      // deep clone
      motionArr = [...motionArr];
    }
    dispatch(setStage({ key: 'live2dMotion', value: motionArr }));
  }
  return {
    performName: 'none',
    duration,
    isHoldOn: false,
    stopFunction: () => {},
    blockingNext: () => false,
    blockingAuto: () => false,
    stopTimeout: undefined, // 暂时不用，后面会交给自动清除
  };
};

```