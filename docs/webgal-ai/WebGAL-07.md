# WebGal源码解析 7

# `/packages/webgal/src/Core/util/getSentenceArg.ts`

这段代码定义了一个名为 `getSentenceArgByKey` 的函数，它接受两个参数：一个 `ISentence` 类型的句子对象（sentnece）和一个字符串参数 `argk`。函数的作用是返回与给定 `argk` 参数相关的 `sentnece` 对象中的参数。

函数首先从 `sentnece.args` 数组中查找与给定 `argk` 参数的参数。如果找到了这样的参数，函数将返回该参数的值。否则，函数返回 `null`。

简洁来说，这段代码定义了一个函数，用于根据给定的参数在 `sentnece` 对象中查找相应的参数并返回。


```js
import { ISentence } from '@/Core/controller/scene/sceneInterface';

export function getSentenceArgByKey(sentnece: ISentence, argk: string): null | string | boolean | number {
  const args = sentnece.args;
  const result = args.find((arg) => arg.key === argk);
  if (result) {
    return result.value;
  } else return null;
}

```

# `/packages/webgal/src/Core/util/match.ts`

这段代码定义了一个名为`Matcher`的类，用于在给定一个模式`T`和一个模式解析器`R`的情况下，创建一组验证函数。

具体来说，这段代码执行以下操作：

1. 定义一个名为`Case`的类型，其中`T`为类型参数，`R`为类型变量，类型定义为`[T, () => R]`，表示一个有两个元素的元组类型，第一个元素为`T`，第二个元素为`R`的类型，使得`Case`可以用来定义一个具有`T`和`R`两个元素的`Case`对象。

2. 定义一个名为`Matcher`的类，其中包含一个`private cases`成员变量，它是一个包含多个`Case`对象的数组，每个`Case`对象包含一个`pattern`和一个`fn`，表示一个验证函数，其中`pattern`是给定的模式，`fn`是一个返回类型为`R`的函数，它被绑定到给定的模式上。

3. 在`Matcher`构造函数中，将`subject`属性设置为要验证的值，`defaultCase`属性是一个一个`Case`对象，其中包含一个`default`方法，它具有与`defaultCase`相同的签名，但没有明确的返回类型。

4. `Matcher`还定义了一个`evaluate`方法，该方法接收两个参数：一个模式和一个解析函数，它返回一个解析函数的返回值，或者在遇到未匹配到的模式时返回未定义的类型。

5. 在`Matcher`的`Case`对象中，使用一个循环遍历所有的模式和解析函数组合，对于每个组合，首先检查给定的模式是否与`subject`相匹配，如果是，就尝试解析函数的返回值，否则继续执行下一个组合。

6. 如果`defaultCase`被定义，`Matcher`会使用它来代替未匹配到模式时的解析函数，即`evaluate`方法的第二组参数，它会递归地调用`evaluate`方法来获取解析函数的返回值。


```js
type Case<T, R> = [T, () => R];

class Matcher<T, R = any> {
  private cases: Array<Case<T, R>> = [];
  private subject: T;
  private defaultCase?: () => R;

  public constructor(subject: T) {
    this.subject = subject;
  }

  public _(pattern: T, fn: () => R): this {
    this.cases.push([pattern, fn]);
    return this;
  }

  public __(pattern: T, fn: () => R) {
    this.cases.push([pattern, fn]);
    return this.evaluate();
  }

  public default(fn: () => R) {
    this.defaultCase = fn;
    return this.evaluate();
  }

  private evaluate(): R | undefined {
    for (const [pattern, action] of this.cases) {
      if (pattern === this.subject) {
        return action();
      }
    }
    if (this.defaultCase) {
      return this.defaultCase();
    }
    return undefined;
  }
}

```

这段代码定义了一个名为 `match` 的函数，它接收一个名为 `subject` 的参数。这个函数返回一个名为 `Matcher` 的类，这个类的实例接收一个名为 `subject` 的参数。

`Matcher` 是一个类，它使用了 `any` 类型作为其属性的 `type` 属性。这个类的实例可以使用 `match` 函数来查找一个与传入的 `subject` 参数相匹配的 `any` 类型实例。

函数的作用是创建一个 `Matcher` 实例，并将它返回。这个实例可以在需要的时候用来查找一个与传入的 `subject` 参数相匹配的 `any` 类型实例。


```js
export function match<T, R = any>(subject: T): Matcher<T, R> {
  return new Matcher(subject);
}

```

# `/packages/webgal/src/Core/util/setEbg.ts`



这段代码定义了一个名为 `setEbg` 的函数，该函数接受一个字符串参数 `url`，用于设置背景图片的 URL。函数的作用是将选定元素 `#ebg` 的背景图片设置为给定 URL。

具体来说，代码首先获取了当前页面中 `#ebg` 元素的引用。如果该元素存在，则执行以下操作：

1. 如果给定的 URL 存在，将元素的 `style.backgroundImage` 属性设置为该 URL，用 `url()` 函数将字符串和 `"url("` 组合成一个新的 URL。
2. 如果给定的 URL 不存在，则不做任何操作，不会设置元素的背景图片。

这段代码的作用是设置一个元素的背景图片为给定 URL，仅在给定的 URL 存在时执行。这个功能可以用于任何 HTML 元素，包括但不限于 `div`, `p`, `img` 等。


```js
export function setEbg(url: string) {
  const ebg = document.getElementById('ebg');
  if (ebg) {
    ebg.style.backgroundImage = `url("${url}")`;
  }
}

```

# `/packages/webgal/src/Core/util/setLogo.ts`

这段代码定义了一个名为 `setLogo` 的函数，接受两个参数：`logoName` 和 `url`。函数的作用是在网页中加载一个名为 `logoImage` 的图片，并将其背景图片设置为给定 `url` 的图像。

具体来说，代码首先获取了名为 `logoImage` 的元素，这个元素通常是一个 `img` 标签。如果这个元素存在，那么代码会将 `url` 赋值给它的 `src` 属性，这样当这个图片加载完成时，它就会显示在页面上。如果 `logoImage` 元素不存在，那么代码会报错，因为没有一个 `img` 标签。

这段代码的作用是加载一个图片，并将其设置为指定 `url` 的图片作为背景。这个功能在网页开发中可能有多种应用，例如用于广告、标语、标志等。


```js
export function setLogo(logoName: string, url: string) {
  const logoImage = document.getElementById('logoImage');
  if (logoImage) {
    logoImage.style.backgroundImage = `url("${url}")`;
  }
}

```

# `/packages/webgal/src/Core/util/coreInitialFunction/bindExtraFunc.ts`

该代码是一个ESLint修饰的函数，用于在编场景模式下使用JMP（JavaScript Modeless Method of但是一般的）来同步组件的props和状态。

更具体地说，这段代码：

1. 使用@/Core/util/syncWithEditor/syncWithOrigine导入了一个名为syncFast的同步函数。
2. 定义了一个名为bindExtraFunc的函数，该函数使用了同步函数的语法，即返回一个函数，并在该函数内部使用了(window as any).JMP来声明一个别处m。
3. 通过将syncFast函数赋值给(window as any).JMP，将同步机制与原始对象的代理（window）相关联，从而实现在开发过程中通过同步机制来轻松地调试和修复代码。
4. 最后，导入了该函数以使其可以在应用程序中使用。


```js
import { syncFast } from '@/Core/util/syncWithEditor/syncWithOrigine';

export const bindExtraFunc = () => {
  (window as any).JMP = syncFast;
};

```

# `/packages/webgal/src/Core/util/coreInitialFunction/infoFetcher.ts`

该代码是一个JavaScript应用程序，它使用axios库从axios服务器发送请求，并使用一个自定义的logger库记录请求和响应中的信息。

axios用于从axios服务器发送请求，并使用response的data属性获取请求的数据。

自定义的logger库用于记录请求和响应中的信息。

assetSetter库用于设置游戏中的资产，例如图片、音频和动画等。

getStorage库用于从本地存储中获取游戏中的资产，例如图片、音频和动画等。

webgalStore用于在游戏中使用WebGL(Web图形库)来渲染场景。

setGuiAsset和setLogoImage用于设置游戏中的GUI(图形用户界面)资产，例如背景图、图标和广告等。

setEbg和setLogo用于设置游戏中的特效，例如背景图、图标和广告等。

initKey用于在游戏中的不同场景之间快速保存和加载进度。

WebgalParser用于解析WebGL场景，并将其渲染成渲染树。

最后，WebGAL用于在游戏中使用WebGL(Web图形库)来渲染场景。


```js
import axios from 'axios';
import { logger } from '../etc/logger';
import { assetSetter, fileType } from '../gameAssetsAccess/assetSetter';
import { getStorage } from '../../controller/storage/storageController';
import { webgalStore } from '@/store/store';
import { setGuiAsset, setLogoImage } from '@/store/GUIReducer';
import { setEbg } from '@/Core/util/setEbg';
import { setLogo } from '@/Core/util/setLogo';
import { initKey } from '@/Core/controller/storage/fastSaveLoad';
import { WebgalParser } from '@/Core/parser/sceneParser';
import { WebGAL } from '@/Core/WebGAL';

declare global {
  interface Window {
    renderPromise?: Function;
  }
}
```

这段代码是一个JavaScript函数，名为`infoFetcher`，用于获取游戏信息。它接受一个游戏信息路径的参数`url`，并返回一个 Promise。

函数内部使用`axios`库发起HTTP请求，获取游戏信息。请求成功后，将获取到的信息存储在`WebgalStore`的`GUI`状态中，并按照游戏的配置开始设置对应的状态。

具体来说，代码中实现了以下功能：

1. 获取游戏信息并打印日志：使用`axios`库获取游戏信息，并使用`logger.info`打印日志。
2. 设置游戏界面资产：使用`setGuiAsset`设置游戏界面的资产，包括背景图、标志图等。
3. 设置游戏名称：通过`WebGAL.gameName`设置游戏名称，并使用`document.title`设置页面标题。
4. 设置游戏键盘：通过`WebGAL.gameKey`设置游戏键盘，并在一个未知的函数中进行初始化。


```js
/**
 * 获取游戏信息
 * @param url 游戏信息路径
 */
export const infoFetcher = (url: string) => {
  const GUIState = webgalStore.getState().GUI;
  const dispatch = webgalStore.dispatch;
  axios.get(url).then((r) => {
    let gameConfigRaw: string = r.data;
    const gameConfig = WebgalParser.parseConfig(gameConfigRaw);
    logger.info('获取到游戏信息', gameConfig);
    // 按照游戏的配置开始设置对应的状态
    if (GUIState) {
      gameConfig.forEach((e) => {
        const { command, args } = e;

        switch (command) {
          case 'Title_img': {
            const titleUrl = assetSetter(args.join(''), fileType.background);
            dispatch(setGuiAsset({ asset: 'titleBg', value: titleUrl }));
            setEbg(titleUrl);
            break;
          }

          case 'Game_Logo': {
            const logoUrlList = args.map((url) => assetSetter(url, fileType.background));
            dispatch(setLogoImage(logoUrlList));
            break;
          }

          case 'Title_bgm': {
            const bgmUrl = assetSetter(args[0], fileType.bgm);
            dispatch(setGuiAsset({ asset: 'titleBgm', value: bgmUrl }));
            break;
          }

          case 'Game_name': {
            WebGAL.gameName = args[0];
            document.title = args[0];
            break;
          }

          case 'Game_key': {
            WebGAL.gameKey = args[0];
            getStorage();
            break;
          }
        }
      });
    }
    window?.renderPromise?.();
    delete window.renderPromise;
    initKey();
  });
};

```

# `/packages/webgal/src/Core/util/etc/logger.ts`

这段代码使用了 'Cloudlogjs' 模块中的 Cloudlog 类来创建一个日志打印工具。具体来说，它创建了一个名为 'logger' 的对象，然后使用 'new Cloudlog' 方法来初始化这个对象。通过调用 'logger' 对象上的 'log' 方法，可以输出日志信息。由于没有配置任何日志输出，因此不会输出具体的内容，但是可以根据上下文来确定如何输出日志。


```js
import Cloudlog from 'cloudlogjs';

/**
 * 日志打印工具
 */
export const logger = new Cloudlog();

```

# `/packages/webgal/src/Core/util/gameAssetsAccess/assetSetter.ts`

这段代码定义了一个枚举类型 `fileType`，用于指定在游戏中需要使用的不同类型的资源。

具体来说，这个枚举类型定义了 7 个枚举值，分别是：

- `background`：背景音乐或效果
- `bgm`：背景音乐，但不受控制
- `figure`：游戏中的模型或场景中的静态物体
- `scene`：游戏场景中的场景元素，如建筑、自然场景等
- `tex`：纹理，用于显示像素级别纹理
- `vocal`：游戏中的声音效果，如角色的对话
- `video`：视频，可能是游戏中的一个镜头或录制保存的游戏片段

在游戏开发中，不同的游戏引擎或框架可能需要使用不同的资源类型，这个枚举类型允许开发人员将它们分组为同一组类型，以更方便地管理资源。


```js
/**
 * @file 资源的引入可能是绝对链接，也可能是文件名，必须做必要的处理。
 */

/**
 * 内置资源类型的枚举
 */
export enum fileType {
  background,
  bgm,
  figure,
  scene,
  tex,
  vocal,
  video,
}

```

这段代码是一个 JavaScript 函数，名为 `assetSetter`，它接受两个参数 `fileName` 和 `assetType`，并返回处理后的资源路径（绝对或相对）。

具体来说，这段代码的作用是：

1. 如果传入的 `fileName` 是绝对链接，那么直接返回该链接。
2. 如果传入的 `assetType` 是 `fileType.background`、`fileType.scene`、`fileType.vocal` 或 `fileType.figure`，那么根据不同的 `fileType` 类型，将 `fileName` 拼接成一个相对路径，最终返回该路径。
3. 如果传入的 `assetType` 是 `fileType.video`，那么将 `fileName` 拼接成一个相对路径，最终返回该路径。
4. 如果无法处理 `fileName`，或者处理出的路径不合法（如不存在），那么返回一个空字符串。

例如，如果传入 `fileName` 为 `"example.txt"`，且 `assetType` 为 `fileType.video`，那么处理后的资源路径为 `"/game/video/example.txt"`。


```js
/**
 * 获取资源路径
 * @param fileName 资源的名称或地址
 * @param assetType 资源类型
 * @return {string} 处理后的资源路径（绝对或相对）
 */
export const assetSetter = (fileName: string, assetType: fileType): string => {
  // 是绝对链接，直接返回
  if (fileName.match('http://') || fileName.match('https://')) {
    return fileName;
  } else {
    // 根据类型拼接资源的相对路径
    let returnFilePath: string;
    switch (assetType) {
      case fileType.background:
        returnFilePath = `./game/background/${fileName}`;
        break;
      case fileType.scene:
        returnFilePath = `./game/scene/${fileName}`;
        break;
      case fileType.vocal:
        returnFilePath = `./game/vocal/${fileName}`;
        break;
      case fileType.figure:
        returnFilePath = `./game/figure/${fileName}`;
        break;
      case fileType.bgm:
        returnFilePath = `./game/bgm/${fileName}`;
        break;
      case fileType.video:
        returnFilePath = `./game/video/${fileName}`;
        break;
      default:
        returnFilePath = ``;
        break;
    }
    return returnFilePath;
  }
};

```

# `/packages/webgal/src/Core/util/pixiPerformManager/initRegister.ts`

这段代码的作用是引入了三个插件(cherryBlossoms、rain和snow)，然后将它们注册到游戏场景(game scene)中。这个代码片段来源于游戏开发引擎，是一个将插件引用到游戏中的常见做法。

具体来说，这段代码将会：

1. 引入cherryBlossoms、rain和snow三个插件，这些插件都是用pixi.js库编写的，用于在游戏中添加特效效果，例如花瓣、雨滴和雪景等。

2. 将这些插件注册到游戏场景中，使得它们在游戏运行时可以被使用。注册的方式可能因游戏引擎而异，但通常会涉及到将插件的代码与游戏代码集成，然后将注册信息保存到游戏状态中，以便在游戏运行时正确地使用插件。

3. 在使用这些插件的过程中，游戏引擎会根据插件的类型和参数自动应用它们的效果，并在游戏渲染时呈现出来。这样，游戏的画面会更加生动有趣，让玩家可以享受到更加丰富的游戏体验。


```js
import '../../gameScripts/pixiPerformScripts/cherryBlossoms';
import '../../gameScripts/pixiPerformScripts/rain';
import '../../gameScripts/pixiPerformScripts/snow';
```

# `/packages/webgal/src/Core/util/pixiPerformManager/pixiPerformManager.ts`

这段代码使用了typing库，用于定义了Container和IResult类型。

Container是一个定义了在PIXI.js中创建容器的接口。

logger是一个定义了用于在应用程序中记录和输出日志的接口，使用了从logger.js文件中引入的logger库。

IKey是定义了用于存储typing.align types中元组的类型，通过组合可以使用string、symbol或number类型的键和ES6的getProperty的语法来声明。

IReg作用于描述生成的Result类型，当生成的结果有container字段时，该字段的值为生成的Container对象，当生成的结果有tick关键字时，该关键字用于在应用中设置计时器。

IName是一个定义了用于存储生成的结果名称的类型，可以使用string或者将一个函数返回的文本。

IPerformCallback是一个用于回调应用程序中perform方法的函数，其返回值为IResult类型的子类型。

usePerformNSM是一个内部函数，用于创建一个Map，用于存储不同实体的perform方法，该Map的键是生成的IName，值为实现了IPerformCallback的函数。

useEffect是一个内部函数，用于执行生成的perform方法，该函数接收一个包含typing.align类型的元素的Map，该Map的key是生成的IName，值为实现了IPerformCallback的函数。

useDeferred是一个内部函数，用于执行代理人方法，该函数接收一个实现了IPerformCallback的函数作为代理人，并返回一个Deferred类型，其值为实现了IPerformCallback的函数返回的IResult类型的子类型。

exportdefault {
 performs,
 logger,
 IName,
 IReg,
 IPerformCallback,
 usePerformNSM,
 useEffect,
 useDeferred,
};


```js
import type { Container } from 'pixi.js';
import { logger } from '../etc/logger';

/**
 * 特效执行返回的结果
 */
export type IResult<R extends Omit<Record<string | symbol | number, unknown>, 'container' | 'tickerKey'> = {}> = {
  container: Container;
  tickerKey: string;
} & R;

type IName = string | (() => string);
type IPerformCallback = () => IResult;

const performs = new Map<string, IPerformCallback>();

```

这两段代码是一对箭头函数，它们都定义了一个名为`getName`的函数，参数`name`可以是字符串或者一个返回字符串的函数。

第一段代码的作用是获取一个字符串类型的参数`name`的名称。如果`name`参数没有被提供，函数将返回`null`。

第二段代码的作用是获取一个字符串类型的参数`name`的名称，即使它不会抛出空函数异常。这个函数使用了`typeof`操作符来检查`name`参数的类型，如果`name`是字符串类型，函数将直接返回`name`的名称；否则，函数将返回`null`。


```js
/**
 * 获取名称, 可能报空
 * @param name 名称或者返回名称的函数
 * @returns {string}
 */
function getName(name: IName): string | null {
  if (!name) return null;
  if (typeof name === 'string') return name;
  return name();
}

/**
 * 获取名称, 不会报空
 * @param name 名称或者返回名称的函数
 * @returns {string}
 */
```

以上代码定义了一个名为 `getKey` 的函数，接受一个名为 `name` 的参数。该函数的作用是获取给定名称的表演，如果历史上没有记录，则返回空字符串，否则返回该表演的名称。

接下来代码定义了一个名为 `registerPerform` 的函数，该函数接受两个参数，一个是特效名称 `name`，另一个是调用特效的函数 `callback`。该函数首先检查 `callback` 是否为函数类型，如果不是，则抛出错误。然后将 `callback` 存储到 `performs` 对象中，该对象的键是 `getKey(name)`。

这些函数一起工作，使得可以通过调用 `registerPerform` 函数并传入一个表演名称和相应函数作为参数，来实现给定名称的表演。如果历史上已经存在同名的表演，函数将不会创建新的表演，而是返回已经存在的名称。如果调用 `registerPerform` 函数时，传递的参数不符合规范，则函数将抛出错误并返回。


```js
function getKey(name: IName): string {
  const key = getName(name);
  if (!key) {
    logger.error('Get name of perform failed. There no name of the perform.');
    return '';
  }
  return key;
}

/**
 * 注册特效, 注意, 同名会注销旧特效
 * @param name 特效名
 * @param callback 调用特效的函数
 */
export function registerPerform(name: IName, callback: IPerformCallback): void {
  if (!callback || typeof callback !== 'function') throw new Error(`"${name}" is not a callback.`);
  performs.set(getKey(name), callback);
}

```

这段代码定义了一个名为 `call` 的函数，用于调用特效。函数有两个参数：`name` 和 `args`，其中 `name` 是指要调用哪个特效，而 `args` 是一个自定义的参数数组。函数返回一个 `IResult` 类型的结果。

函数的核心部分是使用 `perform.get` 方法获取特效的名字，然后再调用 `permits.get` 方法获取对应的 `Function` 类型。如果这两种方法返回的对象不是 `Function` 类型，函数会抛出一个错误。

如果已经获取到了特效的名字和对应的 `Function`，函数就会调用该 `Function` 的第一个参数，并将 `...` 传递给该 `Function` 的第二个参数。这样，就可以将 `args` 参数传递给函数内部需要调用的事物。

总之，这段代码定义了一个可以调用特定特效的函数，用于在需要时动态地加载和调用它们。


```js
/**
 * 调用特效
 * @param name 特效名
 * @param args 自定义的参数
 * @returns {IResult}
 */
export function call(name: IName, args: unknown[] = []): IResult {
  const callback = performs.get(getKey(name));

  if (!callback || !(callback instanceof Function)) {
    logger.error(`Can\'t call the perform named "${name}"`);
    throw new Error(`"${name}" don't have the pixiPerform callback.`);
  }
  return (callback as IPerformCallback)(...(args as []));
}

```

这段代码定义了两个函数，其中一个用于将特效从特效集中删除，另一个用于获取所有可调用的特效名称。

`unregisterPerform`函数接受一个特效名称作为参数，从特效集中删除该特效，并返回是否成功。如果成功，则返回`true`；否则返回`false`。

`getPerforms`函数返回一个包含所有可调用的特效名称的数组。它首先使用`keys`方法从`performs`对象中获取所有键，然后返回这些键的值列表。

这两个函数一起使用，以便用户可以查看可以调用的特效列表，并选择要注销或启用哪些特效。


```js
/**
 * 注销特效
 * @param name 特效名
 */
export function unregisterPerform(name: IName) {
  performs.delete(getKey(name));
}

/**
 * 获取全部可调用的特效特效名
 */
export function getPerforms(): string[] {
  return [...performs.keys()];
}

```

这是一个 Python 代码片段，从其 imports 导入了一个名为 "initRegister" 的函数。但具体这个函数的实现并没有在代码中给出，因此无法对其进行详细的解释。在实际应用中，我们应该关注函数的部分，而不是导入的模块名称。


```js
import('./initRegister');

```

# `/packages/webgal/src/Core/util/prefetcher/assetsPrefetcher.ts`

这段代码是一个预加载函数，用于在场景中加载资源。它接受一个包含场景资源的数组（assetList），并在循环中处理每个资产。

首先，代码检查资产列表中是否有已经存在的资产。如果是，则表示这个资源已经预加载过了，不需要再次加载，从而取消插入预加载标签（isInsert = false）。

否则，代码会输出一个警告消息（logger.warn），表示这个资源还在预加载列表中，需要进行预加载。

如果isInsert为false，那么代码将创建一个新的预加载标签（<link>）并设置其rel属性为'prefetch'，设置其href属性为该资产的url。如果之前已经存在一个或多个head元素，则将新的预加载标签插入到该位置（<head>）。

最后，代码将资产列表中的每个资产的url添加到已设置好的setledAssets数组中。


```js
import { IAsset } from '@/Core/controller/scene/sceneInterface';
import { logger } from '../etc/logger';

import { WebGAL } from '@/Core/WebGAL';

/**
 * 预加载函数
 * @param assetList 场景资源列表
 */
export const assetsPrefetcher = (assetList: Array<IAsset>) => {
  for (const asset of assetList) {
    // 是否要插入这个标签
    let isInsert = true;
    // 判断是否已经存在
    WebGAL.sceneManager.settledAssets.forEach((settledAssetUrl) => {
      if (settledAssetUrl === asset.url) {
        isInsert = false;
      }
    });
    if (!isInsert) {
      logger.warn('该资源已在预加载列表中，无需重复加载');
    } else {
      const newLink = document.createElement('link');
      newLink.setAttribute('rel', 'prefetch');
      newLink.setAttribute('href', asset.url);
      const head = document.getElementsByTagName('head');
      if (head.length) {
        head[0].appendChild(newLink);
      }
      WebGAL.sceneManager.settledAssets.push(asset.url);
    }
  }
};

```

# `/packages/webgal/src/Core/util/prefetcher/scenePrefetcher.ts`

这段代码定义了一个名为 `scenePrefetcher` 的函数，它的作用是预加载场景文件列表。它接受一个参数 `sceneList`，其中包含需要预加载的场景文件名称。

函数内部首先遍历 `sceneList` 中的每个场景文件，然后使用 `sceneFetcher` 函数下载这些场景文件。如果场景文件已经被加载过，函数会使用 `logger` 函数输出一条警告信息，否则会输出一条信息。最后，函数会定期遍历 `sceneList` 中的每个场景文件，并在预加载完成后清除 `sceneManager.settledScenes` 数组中的场景名称，以便下一次预加载时可以再次加载。


```js
/**
 * 场景预加载
 * @param sceneList 需要预加载的场景文件列表
 */
import { sceneFetcher } from '../../controller/scene/sceneFetcher';
import { sceneParser } from '../../parser/sceneParser';
import { logger } from '@/Core/util/etc/logger';

import { WebGAL } from '@/Core/WebGAL';

export const scenePrefetcher = (sceneList: Array<string>): void => {
  for (const e of sceneList) {
    if (!WebGAL.sceneManager.settledScenes.includes(e)) {
      logger.info(`现在预加载场景${e}`);
      sceneFetcher(e).then((r) => {
        sceneParser(r, e, e);
      });
    } else {
      logger.warn(`场景${e}已经加载过，无需再次加载`);
    }
  }
};

```

# `/packages/webgal/src/Core/util/syncWithEditor/syncWithOrigine.ts`

这段代码定义了一个名为 `syncWithOrigine` 的函数，它接受两个参数 `sceneName` 和 `sentenceId`。函数的作用是切换到指定的场景，并将一个名为 `showTitle` 的组件设置为隐藏，另一个名为 `showMenuPanel` 的组件设置为隐藏。此外，它还调用了一个名为 `resetStage` 的函数，这个函数可能是用于在场景中执行一些清理操作。

具体来说，这段代码的主要作用是使用 WebGAL 库中的 `setVisibility` 和 `sceneFetcher` 函数将场景的可见性设置为 false，然后使用 `WebGAL.sceneManager` 对象中的 `resetStage` 函数将场景文件从磁盘读取到内存中。接下来，它将使用 `syncFast` 函数从指定的语句开始同步游戏进度，同时将 `WebGAL.gameplay` 对象中的 `isFast` 设置为 `true`，以使游戏进度更快。


```js
import { resetStage } from '@/Core/controller/stage/resetStage';
import { assetSetter, fileType } from '@/Core/util/gameAssetsAccess/assetSetter';
import { sceneFetcher } from '@/Core/controller/scene/sceneFetcher';
import { sceneParser } from '@/Core/parser/sceneParser';
import { logger } from '../etc/logger';
import { webgalStore } from '@/store/store';
import { setVisibility } from '@/store/GUIReducer';
import { nextSentence } from '@/Core/controller/gamePlay/nextSentence';

import { WebGAL } from '@/Core/WebGAL';

export const syncWithOrigine = (sceneName: string, sentenceId: number) => {
  logger.warn('正在跳转到' + sceneName + ':' + sentenceId);
  const dispatch = webgalStore.dispatch;
  dispatch(setVisibility({ component: 'showTitle', visibility: false }));
  dispatch(setVisibility({ component: 'showMenuPanel', visibility: false }));
  resetStage(true);
  // 重新获取初始场景
  const sceneUrl: string = assetSetter(sceneName, fileType.scene);
  // 场景写入到运行时
  sceneFetcher(sceneUrl).then((rawScene) => {
    WebGAL.sceneManager.sceneData.currentScene = sceneParser(rawScene, 'start.txt', sceneUrl);
    // 开始快进到指定语句
    const currentSceneName = WebGAL.sceneManager.sceneData.currentScene.sceneName;
    WebGAL.gameplay.isFast = true;
    syncFast(sentenceId, currentSceneName);
  });
};

```

此代码是一个 JavaScript 函数，名为 `syncFast`，其作用是确保在一个场景中，对给定场景的给定句子ID，在执行动画操作时，其Fast模式能够正确同步。

具体来说，代码首先检查给定的句子ID和当前场景名称是否满足条件。如果是，代码将调用 `nextSentence` 函数，这将尝试在给定场景中执行动画操作。然后，为了确保同步，代码将调用 `syncFast` 函数本身，传入两次，间隔为2秒。如果第一次调用 `syncFast` 时条件不满足，即场景中不存在给定的句子ID或者场景名称与当前场景名称不匹配，代码将设置 `WebGAL.gameplay.isFast` 状态为 `false`，这将阻止游戏中的动画效果。


```js
export function syncFast(sentenceId: number, currentSceneName: string) {
  if (
    WebGAL.sceneManager.sceneData.currentSentenceId < sentenceId &&
    WebGAL.sceneManager.sceneData.currentScene.sceneName === currentSceneName
  ) {
    nextSentence();
    setTimeout(() => syncFast(sentenceId, currentSceneName), 2);
  } else {
    WebGAL.gameplay.isFast = false;
  }
}

```

# `/packages/webgal/src/Core/util/syncWithEditor/webSocketFunc.ts`

This is a JavaScript function that starts a WebSocket connection to a server and sends messages for debugging purposes. The function is part of the `@/types/debugProtocol` and `@/Core/WebGAL` packages.

First, it checks if the current URL is http or https, and if it's not https, it returns immediately. Then, it constructs a WebSocket URL by appending a server name and a port to the base URL, which is different if https is used.

Next, it logs in the console and creates a WebSocket connection to that URL. It then listens for messages from the server and sends back the appropriate information using the `IDebugMessage` interface from the `@/types/debugMessage` package.

If it receives a `DebugCommand.SYNCFC` message, it sends a `sync` stage sync message to the server. The sync message includes the current scene name and scene ID, as well as the stage of the game.

If it receives any other messages, it logs the message and sends a `syncWithOrigine` function to the server. This function syncs the stage of the game with the original stage from the server.


```js
import { logger } from '../etc/logger';
import { syncWithOrigine } from '@/Core/util/syncWithEditor/syncWithOrigine';
import { DebugCommand, IDebugMessage } from '@/types/debugProtocol';
import { WebGAL } from '@/Core/WebGAL';
import { webgalStore } from '@/store/store';

export const webSocketFunc = () => {
  const loc: string = window.location.hostname;
  const protocol: string = window.location.protocol;
  if (protocol !== 'http:' && protocol !== 'https:') {
    return;
  }
  let wsUrl = `ws://${loc}:9999`;
  if (protocol === 'https:') {
    wsUrl = `wss://${loc}/webgalsync`;
  }
  logger.info('正在启动socket连接位于：' + wsUrl);
  const socket = new WebSocket(wsUrl);
  socket.onopen = () => {
    logger.info('socket已连接');
    function sendStageSyncMessage() {
      const message: IDebugMessage = {
        command: DebugCommand.SYNCFC,
        sceneMsg: {
          scene: WebGAL.sceneManager.sceneData.currentScene.sceneName,
          sentence: WebGAL.sceneManager.sceneData.currentSentenceId,
        },
        stageSyncMsg: webgalStore.getState().stage,
      };
      socket.send(JSON.stringify(message));
      logger.debug('传送信息', message);
      setTimeout(sendStageSyncMessage, 1000);
    }
    sendStageSyncMessage();
  };
  socket.onmessage = (e) => {
    logger.info('收到信息', e.data);
    const str: string = e.data;
    const message: IDebugMessage = JSON.parse(str);
    if (message.command === DebugCommand.JUMP) {
      syncWithOrigine(message.sceneMsg.scene, message.sceneMsg.sentence);
    }
  };
  socket.onerror = (e) => {
    logger.info('当前没有连接到 Terre 编辑器');
  };
};

```

# `/packages/webgal/src/hooks/useFontFamily.ts`

此代码使用了 React-Redux 库中的 `useSelector` hook 从 Redux 状态中取样 `userData` 选择器，并获取到 `optionData` 状态中的 `textboxFont` 属性的值。

然后，定义了一个 `useFontFamily` 函数，该函数使用 `getFont` 函数从 `textboxFont` 状态中取样，并返回字体族组合。

`getFont` 函数从 `textboxFont` 状态中取样，如果取到了 `textboxFont.song`，则返回字体为 "思源宋体"，否则返回字体为 "WebgalUI"，并添加 `serif"` 类型注释。

最后，`useFontFamily` 函数返回 `getFont` 函数的返回值，这样就可以将字体族组合返回给父组件使用了。


```js
import { useSelector } from 'react-redux';
import { RootState } from '@/store/store';
import { textFont } from '@/store/userDataInterface';

export function useFontFamily() {
  const fontFamily = useSelector((state: RootState) => state.userData.optionData.textboxFont);

  function getFont() {
    return fontFamily === textFont.song ? '"思源宋体", serif' : '"WebgalUI", serif';
  }

  return getFont();
}

```

# `/packages/webgal/src/hooks/useGenSyncRef.ts`

这段代码使用了 React 和 Redux 的一些特性。现在我来解释一下这段代码的作用。

首先，这是一段导入了 `useSelector` 和 `useRef` 的函数 `useGenSyncRef`。这个函数接收两个参数：一个是要订阅的 Redux  store 的 selector 函数，另一个是选中（selected）的对象。

`useGenSyncRef` 的作用是生成一个和 Redux 自动同步的 Ref 对象。这个 Ref 对象可以在选中（selected）的键上起到作用，当键变化时，会自动更新 Redux store 的 state。

代码中首先导入了 `useSelector`，并在 `useEffect` 钩子中创建了一个新的 `Ref` 对象，将 Redux store 赋值给这个 `Ref` 对象。这样做是为了确保当 `useSelector` 组件渲染时，`Ref` 对象已经与 Redux store 同步了。

最后，函数返回了一个由 `current` 和 `Store` 组成的对象，其中 `current` 属性是一个 `Selected` 类型的变量，它的值就是 Redux store 的 selected 键的值。


```js
import { useSelector } from 'react-redux';
import { useEffect, useRef } from 'react';

/**
 * 生成一个和redux自动同步的Ref对象
 */
export function useGenSyncRef<TState = unknown, Selected = unknown>(
  selector: (state: TState) => Selected,
): { readonly current: Selected } {
  const Store = useSelector(selector);
  const Ref = useRef(Store);
  useEffect(() => {
    Ref.current = Store;
  }, [Store]);
  return Ref;
}

```

# `/packages/webgal/src/hooks/useLanguage.ts`

这段代码是一个 React 组件，它使用了 Redux 状态管理库和 i18n 本地化组件。它的主要目的是获取用户选择的数据中当前语言的名称，并在需要时将其应用到页面上。

具体来说，这段代码实现了以下功能：

1. 引入了 `RootState`、`setOptionData` 和 `useTranslation` 函数，分别用于获取应用程序的全局状态、设置用户数据选项数据和用于本地化处理。
2. 引入了 `useGenSyncRef` 和 `logger` 函数，用于创建一个用于存储用户数据选项的引用实例，以及记录日志信息。
3. 导入了 `getLanguageName` 和 `setLanguageName` 函数，用于从 `language` 对象中获取当前语言名称并将其设置为全局变量 `lang`。
4. 导入了 `useDispatch` 和 `useGenSyncRef` 函数，用于创建一个用于通知用户选择数据变更的 Redux  dispatcher 和用于存储当前语言的同步请求实例。
5. 通过 `useEffect` 钩子，监听 `userData` 选项数据的变化，并在变化时调用 `setOptionData` 函数将其存储到 Redux 的 `optionData` 中。
6. 通过 `useEffect` 钩子，监听本地存储中的 `lang` 键的变化，并在变化时调用 `logger.info` 函数将其记录到日志中。
7. 创建了一个 `useMemo` hook，用于在第一次加载页面时从 `optionData` 中提取 `currentLanguage` 选项数据，并在后续根据需要动态更新。


```js
import { RootState } from '@/store/store';
import { setOptionData } from '@/store/userDataReducer';
import { useTranslation } from 'react-i18next';
import { useDispatch } from 'react-redux';
import { useGenSyncRef } from './useGenSyncRef';
import { logger } from '@/Core/util/etc/logger';
import { setStorage } from '@/Core/controller/storage/storageController';
import { language } from '@/config/language';

export function getLanguageName(lang: language): string {
  return language[lang];
}

export default function useLanguage() {
  const { i18n } = useTranslation();
  const userDataRef = useGenSyncRef((state: RootState) => state.userData);
  const dispatch = useDispatch();

  return (_lang?: language, isSyncStorage = true) => {
    const lang = _lang ?? userDataRef.current?.optionData.language ?? language.zhCn;

    const languageName = getLanguageName(lang);
    i18n.changeLanguage(languageName);

    dispatch(setOptionData({ key: 'language', value: lang }));
    logger.info('设置语言: ' + languageName);
    window?.localStorage.setItem('lang', lang.toString());
    if (isSyncStorage) {
      setStorage();
    }
  };
}

```

# `/packages/webgal/src/hooks/useLifeCycle.ts`

这两个函数都是用来在函数式组件中注册/卸除副作用函数(或者说在组件挂载/卸载时执行代码)。

* `useMounted` 函数接收一个副作用函数 `callback`。它的作用是在组件挂载时执行这个函数。函数式组件在挂载时，其上下文中可能会有一些依赖关系(比如状态、消息等等)需要被打扰，而 `useMounted` 函数可以让我们在挂载时执行一些副作用，避免了在这些依赖关系被打扰。
* `useUnMounted` 函数同样接收一个副作用函数 `callback`，但是它的作用是在组件卸载时执行这个函数。函数式组件在卸载时，其上下文中可能会有一些依赖关系(比如状态、消息等等)需要被打扰，而 `useUnMounted` 函数可以让我们在卸载时执行一些副作用，避免了在这些依赖关系被打扰。

`useEffect` 是 React 提供的一个库，可以让我们在函数式组件中使用副作用函数。它的第一个参数是一个回调函数，第二个参数是一个数组，里面都是依赖关系(可以理解为 props)，描述了在挂载/卸载时需要执行的副作用。


```js
import { useEffect } from 'react';

// 挂载
export function useMounted(callback: Function) {
  useEffect(() => {
    callback();
  }, []);
}

// 卸载
export function useUnMounted(callback: Function) {
  useEffect(() => {
    return function () {
      callback();
    };
  }, []);
}

```



该代码定义了一个名为 `useUpdated` 的函数组件，用于在数据发生变化时调用传递给它的回调函数。

具体来说，该组件创建了一个内部依赖变量 `callback`，它会在组件挂载后执行一次，如果内部依赖变量 `callback` 的值发生了改变，那么就会重新调用传递给它的回调函数。

该组件使用了 React 的 `useEffect` hook 来处理数据变化时的副作用，其中 `useEffect` 接收两个参数：一个是回调函数本身，另一个是回调函数的选项对象。

`useEffect` 钩子内部还有一个 `()` 函数，这个函数在每次渲染组件时都会被调用一次，它的作用是在渲染时更新组件的引用变量，以确保组件正确地获取最新的数据。

当组件渲染时，如果 `callback` 变量发生了改变，那么就会重新调用 `useEffect` 钩子中的 `callback` 函数，并且由于 `useEffect` 钩子是 `React.useEffect` 引入的，因此组件将会在组件挂载后自动获取最新的数据，从而确保组件始终获取到最新的数据状态。


```js
// 更新
export function useUpdated(callback: Function) {
  useEffect(() => {
    callback();
  });
}

```

# `/packages/webgal/src/hooks/useNotFirstEffect.ts`

这段代码是一个 React Hook，名为 `useNotFirstEffect`，其作用是监听一个名为 `callback` 的函数，并在 `firstly` 状态变量为 `true` 时执行该函数。

具体来说，代码执行以下步骤：

1. 从 `react` 包中导入 `DependencyList`、`useEffect` 和 `useState`。
2. 定义一个名为 `useNotFirstEffect` 的函数，该函数接收一个名为 `callback` 的函数参数，以及一个名为 `deps` 的依赖列表参数。
3. 在函数内部，使用 `useState` 创建一个名为 `firstly` 的状态变量，初始值为 `false`。
4. 使用 `useEffect` 创建一个被称为 `setFirstly` 的状态更新函数，该函数接收一个名为 `deps` 的依赖列表参数。
5. 在 `useEffect` 函数中，将 `firstly` 的值更新为 `true`，并将其添加到 `deps` 中的依赖列表中。
6. 调用 `callback` 函数，该函数将在 `firstly` 状态变量为 `true` 时被执行。

由于 `useEffect` 函数使用了 DependencyList，因此 `useNotFirstEffect` 函数将监听 `callback` 函数的调用，并在 `firstly` 状态变量为 `true` 时执行该函数。


```js
import { DependencyList, useEffect, useState } from 'react';

export default function useNotFirstEffect(callback: () => void, deps: DependencyList) {
  const [firstly, setFirstly] = useState<boolean>(false);

  useEffect(() => {
    setFirstly(true);
    if (firstly) callback();
  }, deps);
}

```

# `/packages/webgal/src/hooks/useSoundEffect.ts`

这段代码的作用是引入了一些外部资源，包括一个名为`maou_se_system35.mp3`的音频文件，一个名为`maou_se_system39.mp3`的音频文件，一个名为`maou_se_system40.mp3`的音频文件，一个名为`maou_se_system41.mp3`的音频文件，一个名为`maou_se_system48.mp3`的音频文件，一个名为`Motion-Pop03-4.mp3`的音频文件，一个名为`button-16.mp3`的音频文件，一个名为`button-20.mp3`的音频文件，一个名为`button-50.mp3`的音频文件，一个名为`click_soft_02.mp3`的音频文件，一个名为`Book_Page_Flip.mp3`的音频文件和一个名为`page-flip-1.mp3`的音频文件，以及一个名为`pause.mp3`的音频文件。

具体来说，这段代码主要实现了以下目的：

1. 引入了多个音频文件，包括`maou_se_system35.mp3`、`maou_se_system39.mp3`、`maou_se_system40.mp3`、`maou_se_system41.mp3`、`maou_se_system48.mp3`、`Motion-Pop03-4.mp3`、`button-16.mp3`、`button-20.mp3`、`button-50.mp3`、`click_soft_02.mp3`、`Book_Page_Flip.mp3`和`page-flip-1.mp3`。

2. 这些音频文件主要用于实现音频播放功能，例如在用户点击时播放相应的音频，或者在特定情境下播放指定的音频。


```js
import { setStage } from '@/store/stageReducer';
import mouseEnterSE from '@/assets/se/mouseEnter.mp3';
import maou_se_system35 from '@/assets/se/maou_se_system35.mp3';
import maou_se_system39 from '@/assets/se/maou_se_system39.mp3';
import maou_se_system40 from '@/assets/se/maou_se_system40.mp3';
import maou_se_system41 from '@/assets/se/maou_se_system41.mp3';
import maou_se_system48 from '@/assets/se/maou_se_system48.mp3';
import Motion_Pop03_4 from '@/assets/se/Motion-Pop03-4.mp3';
import button_16 from '@/assets/se/button-16.mp3';
import button_20 from '@/assets/se/button-20.mp3';
import button_50 from '@/assets/se/button-50.mp3';
import click_soft_02 from '@/assets/se/click_soft_02.mp3';
import Book_Page_Flip from '@/assets/se/Book_Page_Flip.mp3';
import page_flip_1 from '@/assets/se/page-flip-1.mp3';
import pause from '@/assets/se/pause.mp3';
```

该代码片段是一个 React 组件，它使用了一个音频标签（a标签）和一个自定义音频库（switch_1 和 water_drop_sound）来加载两个音频文件，并在播放时使用了一个暂停按钮（pausestart）和一个点击时切换音效的按钮（switch_1）。

具体来说，该组件从 @/assets/se/ 目录下进口了两个音频文件，分别是 pausestart.mp3 和 water-drop-sound.mp3。在组件内部，使用 useDispatch 和 useState 钩子来订阅一个名为 action 的状态变量。当用户点击暂停按钮时，该组件会将 action 发送到 store 中，然后调用 webgalStore 来进行音频播放。当用户点击切换按钮时，该组件会将 action 发送到 store 中，然后根据当前音效状态调用不同的音频文件。


```js
import pausestart from '@/assets/se/pausestart.mp3';
import s_cheremisinov_Click_03 from '@/assets/se/s-cheremisinov-Click-03.mp3';
import switch_1 from '@/assets/se/switch-1.mp3';
import water_drop_sound from '@/assets/se/water-drop-sound.mp3';
import aigei_se_01 from '@/assets/se/aigei-se-01.mp3';
import aigei_se_02 from '@/assets/se/aigei-se-02.mp3';
import aigei_se_03 from '@/assets/se/aigei-se-03.mp3';
import zhanZhang_y1970 from '@/assets/se/zhanZhang-y1970.mp3';
import taoshen_01 from '@/assets/se/taoshen-01.mp3';
import { useDispatch } from 'react-redux';
import { webgalStore } from '@/store/store';

/**
 * 调用音效
 */
```

很抱歉，我无法理解您提供的内容。您似乎在描述一个游戏中的鼠标事件，但我不清楚您需要什么样的帮助。如果您需要帮助，请提供更多详细信息，我将尽力回答您的问题。


```js
const useSoundEffect = () => {
  const dispatch = useDispatch();

  const playSeEnter = () => {
    dispatch(setStage({ key: 'uiSe', value: mouseEnterSE }));
  };
  const playSeEnterTitleButton = () => {
    dispatch(setStage({ key: 'uiSe', value: aigei_se_01 }));
  };
  const playSeEnterOptionSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: zhanZhang_y1970 }));
  };
  const playSeEnterMenuPanelSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: zhanZhang_y1970 }));
  };
  const playSeEnterSavePanelSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: zhanZhang_y1970 }));
  };
  const playSeEnterLoadPanelSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: zhanZhang_y1970 }));
  };
  const playSeEnterCGPanelSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: zhanZhang_y1970 }));
  };
  const playSeEnterExtraElement = () => {
    dispatch(setStage({ key: 'uiSe', value: mouseEnterSE }));
  };
  const playSeEnterExtraBGMButton = () => {
    dispatch(setStage({ key: 'uiSe', value: taoshen_01 }));
  };
  const playSeEnterCloseButton = () => {
    dispatch(setStage({ key: 'uiSe', value: aigei_se_03 }));
  };
  const playSeEnterOptionSlider = () => {
    dispatch(setStage({ key: 'uiSe', value: water_drop_sound }));
  };
  const playSeClick = () => {
    dispatch(setStage({ key: 'uiSe', value: maou_se_system39 }));
  };
  const playSeClickTitleButton = () => {
    dispatch(setStage({ key: 'uiSe', value: maou_se_system40 }));
  };
  const playSeClickCloseButton = () => {
    dispatch(setStage({ key: 'uiSe', value: button_16 }));
  };
  const playSeClickBottomControlPanelButton = () => {
    dispatch(setStage({ key: 'uiSe', value: button_20 }));
  };
  const playSeClickMenuNormalButton = () => {
    dispatch(setStage({ key: 'uiSe', value: button_50 }));
  };
  const playSeClickOptionSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: switch_1 }));
  };
  const playSeClickLoadPanelSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: page_flip_1 }));
  };
  const playSeClickSavePanelSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: page_flip_1 }));
  };
  const playSeClickSaveElement = () => {
    dispatch(setStage({ key: 'uiSe', value: maou_se_system48 }));
  };
  const playSeClickLoadElement = () => {
    dispatch(setStage({ key: 'uiSe', value: maou_se_system48 }));
  };
  const playSeClickToNextBgmButton = () => {
    dispatch(setStage({ key: 'uiSe', value: switch_1 }));
  };
  const playSeClickToLastBgmButton = () => {
    dispatch(setStage({ key: 'uiSe', value: switch_1 }));
  };
  const playSeClickBeginBgmButton = () => {
    dispatch(setStage({ key: 'uiSe', value: pausestart }));
  };
  const playSeClickStopBgmButton = () => {
    dispatch(setStage({ key: 'uiSe', value: pause }));
  };
  const playSeClickCGPanelSelect = () => {
    dispatch(setStage({ key: 'uiSe', value: Book_Page_Flip }));
  };
  const playSeClickCGElement = () => {
    dispatch(setStage({ key: 'uiSe', value: s_cheremisinov_Click_03 }));
  };
  const playSeClickBacklogJumpButton = () => {
    dispatch(setStage({ key: 'uiSe', value: click_soft_02 }));
  };

  return {
    playSeEnter, // 鼠标进入
    playSeEnterTitleButton, // 鼠标进入标题按钮
    playSeEnterOptionSelect, // 鼠标进入游戏选项切换按钮
    playSeEnterMenuPanelSelect, // 鼠标进入菜单页切换按钮
    playSeEnterSavePanelSelect, // 鼠标进入存档页切换按钮
    playSeEnterLoadPanelSelect, // 鼠标进入读档页切换按钮
    playSeEnterCGPanelSelect, // 鼠标进入CG页切换按钮
    playSeEnterExtraElement, // 鼠标进入鉴赏元素
    playSeEnterExtraBGMButton, // 鼠标进入鉴赏页BGM按钮
    playSeEnterCloseButton, // 鼠标进入关闭按钮
    playSeEnterOptionSlider, // 鼠标进入滑块选项
    playSeClick, // 鼠标点击
    playSeClickTitleButton, // 鼠标点击标题按钮
    playSeClickCloseButton, // 鼠标点击关闭按钮
    playSeClickBottomControlPanelButton, // 鼠标点击底部控制按钮
    playSeClickMenuNormalButton, // 鼠标点击菜单页普通按钮
    playSeClickOptionSelect, // 鼠标点击游戏选项切换按钮
    playSeClickLoadPanelSelect, // 鼠标点击读档页切换按钮
    playSeClickSavePanelSelect, // 鼠标点击存档页切换按钮
    playSeClickSaveElement, // 鼠标点击存档元素
    playSeClickLoadElement, // 鼠标点击读档元素
    playSeClickToNextBgmButton, // 鼠标点击下一首BGM按钮
    playSeClickToLastBgmButton, // 鼠标点击上一首BGM按钮
    playSeClickBeginBgmButton, // 鼠标点击开始播放BGM按钮
    playSeClickStopBgmButton, // 鼠标点击停止播放BGM按钮
    playSeClickCGPanelSelect, // 鼠标点击CG页切换按钮
    playSeClickCGElement, // 鼠标点击CG元素
    playSeClickBacklogJumpButton, // 鼠标点击日志页回溯按钮
  };
};

```

这段代码是一个自定义的 JavaScript 函数 `useSEByWebgalStore`，它用于在 WebALab 中的音效中进行选择。这个函数使用了 `webgalStore` 和 `useStage` 函数来触发音效。

具体来说，这个函数包含以下几个方法：

* `playSeEnterChoose`：触发音效的过程。当鼠标进入分支选择时，调用此方法来设置音效。
* `playSeClickChoose`：触发音效的过程。当鼠标点击分支选择时，调用此方法来设置音效。
* `playSeEnterDialogButton`：触发音效的过程。当鼠标进入提示框按钮时，调用此方法来设置音效。
* `playSeClickDialogButton`：触发音效的过程。当鼠标点击提示框按钮时，调用此方法来设置音效。

这个函数返回了一个包含音效选择方法的元组，可以被传给 `useSe sound` 函数使用。


```js
/**
 * 调用音效（只供 choose.tsx 使用）
 */
export const useSEByWebgalStore = () => {
  const playSeEnterChoose = () => {
    webgalStore.dispatch(setStage({ key: 'uiSe', value: aigei_se_02 }));
  };
  const playSeClickChoose = () => {
    webgalStore.dispatch(setStage({ key: 'uiSe', value: maou_se_system41 }));
  };
  const playSeEnterDialogButton = () => {
    webgalStore.dispatch(setStage({ key: 'uiSe', value: Motion_Pop03_4 }));
  };
  const playSeClickDialogButton = () => {
    webgalStore.dispatch(setStage({ key: 'uiSe', value: maou_se_system35 }));
  };
  return {
    playSeEnterChoose, // 鼠标进入分支选择
    playSeClickChoose, // 鼠标点击分支选择
    playSeClickDialogButton, // 鼠标点击提示框按钮
    playSeEnterDialogButton, // 鼠标进入提示框按钮
  };
};

```

该代码是一个 React 组件的导出函数，表示导入了名为 `useSoundEffect` 的自定义副作用函数。

这个导出函数的作用是允许组件使用自定义的副作用函数来处理组件的生命周期方法(比如 `useEffect` 和 `useAfterEffect`)，在副作用函数中执行一些异步任务或者在组件挂载、更新、卸载等情况下执行一些代码。

由于副作用函数是异步执行的，因此可以避免阻塞UI线程，提高应用的性能和响应性。同时，通过 `useSoundEffect` 函数，可以方便地实现一些复杂的声音效果，比如在不同状态下的背景声音、点击声音等等。


```js
export default useSoundEffect;

```

# `/packages/webgal/src/hooks/useTextOptions.ts`

这两段代码定义了两个函数 `useTextDelay` 和 `useTextAnimationDuration`，它们都接受一个参数 `type`，并返回一个速度类型。根据传递的速度类型，函数会返回相应的延迟时间或动画持续时间。

`useTextDelay` 函数接受一个速度类型参数 `type`，然后根据不同的速度类型返回一个延迟时间。

* `case playSpeed.slow:`：如果传递的速度类型是 `playSpeed.slow`，则返回 80 延迟时间。
* `case playSpeed.normal:`：如果传递的速度类型是 `playSpeed.normal`，则返回 350 延迟时间。
* `case playSpeed.fast:`：如果传递的速度类型是 `playSpeed.fast`，则返回 200 延迟时间。

`useTextAnimationDuration` 函数与 `useTextDelay` 类似，但它接受一个速度类型参数 `type`，然后根据不同的速度类型返回一个动画持续时间。

* `case playSpeed.slow:`：如果传递的速度类型是 `playSpeed.slow`，则返回 800 动画持续时间。
* `case playSpeed.normal:`：如果传递的速度类型是 `playSpeed.normal`，则返回 350 动画持续时间。
* `case playSpeed.fast:`：如果传递的速度类型是 `playSpeed.fast`，则返回 200 动画持续时间。


```js
import { playSpeed } from '@/store/userDataInterface';

export function useTextDelay(type: playSpeed) {
  switch (type) {
    case playSpeed.slow:
      return 80;
    case playSpeed.normal:
      return 35;
    case playSpeed.fast:
      return 3;
  }
}

export function useTextAnimationDuration(type: playSpeed) {
  switch (type) {
    case playSpeed.slow:
      return 800;
    case playSpeed.normal:
      return 350;
    case playSpeed.fast:
      return 200;
  }
}

```

# `/packages/webgal/src/hooks/useTrans.ts`

这段代码是一个 React 组件内部使用的工具函数，主要作用是为翻译函数提供一些额外的功能。

首先，它导入了 `useTranslation` hook，这个 hook 是 React i18next 库中的一个用于 translations 的工具函数，它可以帮助我们创建一个能够处理多语言文本的环境。

接下来，定义了一个 `useTrans` 函数，它的输入参数 `prefix` 是一个字符串，用于指定在翻译时需要添加的前缀。当 `prefix` 存在时，函数将在输入字符串时自动添加前缀；如果 `prefix` 不存在，函数将直接返回。

接下来，定义了一个 `translation` 函数，这个函数接收一个字符串参数，根据输入字符串是否以"$"开头，来决定是否从输入字符串中提取出翻译关键词。如果输入字符串以"$"开头，则函数会将去掉"$"的字符串作为参数传递给 `trans` 函数，并将提取出的关键词作为参数返回。否则，函数会将 `translation` 函数与 `trans` 函数组合，传入输入字符串作为第一个参数，将返回结果作为第二个参数返回。

最后，定义了一个 `translation` 函数，这个函数接收一个字符串参数，并将它作为参数传递给 `trans` 函数，并将返回结果作为参数返回。这个函数类似于 `translation` 函数，但是它能够处理多个关键词，可以将关键词作为数组传递给 `trans` 函数，并将多个结果返回。


```js
import { useTranslation } from 'react-i18next';

/**
 * @param prefix 翻译时自动添加的前缀
 * @returns 翻译函数, 输入key时会自动添加前缀, "$" 开头则不填加. 输入多个 key 则会返回翻译数组.
 */
export default function useTrans(prefix?: string) {
  const { t } = useTranslation();
  const trans = (key: string) => t(key[0] === '$' ? key.slice(1) : prefix + key);

  function translation(key: string): string;
  function translation(key: string, ...keys: string[]): string[];
  function translation(key: string, ...keys: string[]) {
    if (keys.length) return [trans(key), ...keys.map((v) => trans(v))];
    return trans(key);
  }

  return translation;
}

```

# `/packages/webgal/src/hooks/useValue.ts`

该代码使用了 React 中的 `useState` hook，用于在函数组件中管理状态。`useValue` 函数接受一个初始状态值 `initialState`，并返回一个名为 `value` 的状态变量和一个名为 `set` 的方法，用于设置或获取 `value` 的值。

具体来说，`useValue` 函数内部首先使用 `useState` hook 创建一个名为 `value` 的状态变量，然后设置一个函数 `setValue` 来设置或获取 `value` 的值。这个函数接受一个新的值 `newValue`，将其设置为 `newValue`，并更新 `value` 的状态。

`useValue` 的作用域是函数组件，所以只需要在函数内部创建一次 `value` 变量，并通过调用 `setValue` 来更新它的值。当 `value` 的值发生改变时，`setValue` 方法会将其传递给依赖数组，从而通知所有依赖于 `value` 的组件进行更新。


```js
import { useState } from 'react';

export function useValue<T>(initialState: T) {
  const [value, setValue] = useState(initialState);
  return {
    value,
    set: function (newValue: any) {
      this.value = newValue;
      setValue(newValue);
    },
  };
}

```

# `/packages/webgal/src/store/guiInterface.ts`

以上代码定义了一个名为IGuiState的接口，该接口描述了GUI页面的状态，包括显示或隐藏初始界面、标题界面、菜单界面等。它还包含了一些与GUI界面相关的设置，如当前菜单界面的标签、是否显示背景音乐和图片、是否显示额外的GUI元素等。


```js
/**
 * 当前Menu页面显示的Tag
 */
export enum MenuPanelTag {
  Save, // “保存”选项卡
  Load, // “读取”选项卡
  Option, // “设置”选项卡
}

/**
 * @interface IGuiState GUI状态接口
 */
export interface IGuiState {
  showStarter: boolean; // 是否显示初始界面（用于使得bgm可以播放)
  showTitle: boolean; // 是否显示标题界面
  showMenuPanel: boolean; // 是否显示Menu界面
  showTextBox: boolean;
  currentMenuTag: MenuPanelTag; // 当前Menu界面的选项卡
  showBacklog: boolean;
  titleBgm: string; // 标题背景音乐
  titleBg: string; // 标题背景图片
  logoImage: string[];
  showExtra: boolean;
  showGlobalDialog: boolean;
  showPanicOverlay: boolean;
  isEnterGame: boolean;
}

```

这段代码定义了一个名为 "componentsVisibility" 的类型，它使用 "Pick" 函数从 "IGuiState" 对象中选择 "currentMenuTag"、"titleBg" 和 "logoImage" 的键，然后排除这些键。接着，它定义了一个名为 "GuiAsset" 的类型，它使用 "Pick" 函数从 "IGuiState" 对象中选择 "titleBgm" 或 "titleBg"，然后选择其中一个值。

然后，它定义了一个名为 "IGuiStore" 的接口，它包含 "GuiState" 和 "setGuiAsset"、"setVisibility" 和 "setMenuPanelTag" 方法。其中，"setGuiAsset" 方法接受一个键和另一个字符串参数，它用于设置商店中的 "GuiAsset" 对象。"setVisibility" 方法接受一个键和另一个布尔参数，它用于设置商店中 "componentsVisibility" 的值。"setMenuPanelTag" 方法接受一个 "MenuPanelTag" 对象，它用于设置菜单面板的标签。"IGuiState" 和 "Exclude" 函数用于从 "IGuiState" 对象中排除 "currentMenuTag" 和 "logoImage" 键。

最后，它定义了一个名为 "setVisibilityPayload" 的接口，它包含两个属性：一个 "component" 键和一个 "visibility" 属性。"setVisibilityPayload" 用于设置商店中的 "setVisibility" 方法的参数，它的键必须是 "componentsVisibility" 中的一个键，而它的值必须是布尔值。


```js
export type componentsVisibility = Pick<
  IGuiState,
  Exclude<keyof IGuiState, 'currentMenuTag' | 'titleBg' | 'titleBgm' | 'logoImage'>
>;
// 标题资源
export type GuiAsset = Pick<IGuiState, 'titleBgm' | 'titleBg'>;

export interface IGuiStore {
  GuiState: IGuiState;
  setGuiAsset: <K extends keyof GuiAsset>(key: K, value: string) => void;
  setVisibility: <K extends keyof componentsVisibility>(key: K, value: boolean) => void;
  setMenuPanelTag: (value: MenuPanelTag) => void;
}

export interface setVisibilityPayload {
  component: keyof componentsVisibility;
  visibility: boolean;
}

```

这段代码定义了一个名为 `setAssetPayload` 的接口，其有两个属性：`asset` 和 `value`。

然后，定义了一个名为 `GuiStore` 的类型，其是一个 IGuiStore 的实例。

这里，IGuiStore 可以被理解为一个抽象类，它告诉我们要创建一个可以设置资产名称和值的类。


```js
export interface setAssetPayload {
  asset: keyof GuiAsset;
  value: string;
}

export type GuiStore = IGuiStore;

```

# `/packages/webgal/src/store/GUIReducer.ts`

该代码是一个JavaScript文件，用于记录当前GUI的状态信息。该文件在引擎初始化时会被重置。作者是Mahiru。

该文件中定义了一个名为`initState`的函数，它返回一个包含GUI初始状态的对象。这些状态信息用于在用户界面初始化时设置初始值，并在用户交互时进行相应的更改。

该文件中还引入了来自`@/Core/controller/storage/storageController`和`@/store/guiInterface`的几个函数和变量。这些函数和变量将在文件中详细说明它们的作用和用法。


```js
/**
 * @file 记录当前GUI的状态信息，引擎初始化时会重置。
 * @author Mahiru
 */
import { getStorage } from '@/Core/controller/storage/storageController';
import { GuiAsset, IGuiState, MenuPanelTag, setAssetPayload, setVisibilityPayload } from '@/store/guiInterface';
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

/**
 * 初始GUI状态表
 */
const initState: IGuiState = {
  showBacklog: false,
  showStarter: true,
  showTitle: true,
  showMenuPanel: false,
  showTextBox: true,
  currentMenuTag: MenuPanelTag.Option,
  titleBg: '',
  titleBgm: '',
  logoImage: [],
  showExtra: false,
  showGlobalDialog: false,
  showPanicOverlay: false,
  isEnterGame: false,
};

```

这段代码定义了一个名为 `GUISlice` 的 Slice，用于管理 GUI(图形用户界面)的状态。GUI 状态被存储在一个名为 `gui` 的 Redux 状态中，该状态包含了 `setVisibility`、`setMenuPanelTag`、`setGuiAsset` 和 `setLogoImage` 等方法，用于设置或获取 GUI 组件的显示状态、菜单面板的当前选中项、GUI 资源的值和图标。

具体来说，`setVisibility` 方法接受一个包含组件名称和显示状态的数组，例如 `["comp1", "comp2"]` 将组件 `comp1` 的显示状态设置为 `true`，组件 `comp2` 的显示状态设置为 `false`。

`setMenuPanelTag` 方法接受一个包含菜单面板标签的数组，例如 `["comp1", "comp2"]` 将组件 `comp1` 的菜单面板标签设置为 `"comp2"`。

`setGuiAsset` 方法接受一个包含 GUI 资源名称和资源的值的数组，例如 `["comp1", "comp2", "path/to/resource"]` 将 GUI 资源 `path/to/resource` 的值设置为 `"resource"`。

`setLogoImage` 方法接受一个包含图标图片的数组，例如 `["path/to/logo.png", "path/to/备用logo.png"]` 将 GUI 资源 `path/to/logo.png` 的值设置为 `"path/to/备用logo.png"`。


```js
/**
 * GUI状态的Reducer
 */
const GUISlice = createSlice({
  name: 'gui',
  initialState: initState,
  reducers: {
    /**
     * 设置GUI的各组件的显示状态
     * @param state 当前GUI状态
     * @param action 改变显示状态的Action
     */
    setVisibility: (state, action: PayloadAction<setVisibilityPayload>) => {
      getStorage();
      const { component, visibility } = action.payload;
      state[component] = visibility;
    },
    /**
     * 设置MenuPanel的当前选中项
     * @param state 当前GUI状态
     * @param action 改变当前选中项的Action
     */
    setMenuPanelTag: (state, action: PayloadAction<MenuPanelTag>) => {
      getStorage();
      state.currentMenuTag = action.payload;
    },
    /**
     * 设置GUI资源的值
     * @param state 当前GUI状态
     * @param action 改变资源的Action
     */
    setGuiAsset: (state, action: PayloadAction<setAssetPayload>) => {
      const { asset, value } = action.payload;
      state[asset] = value;
    },
    setLogoImage: (state, action: PayloadAction<string[]>) => {
      state.logoImage = [...action.payload];
    },
  },
});

```

这段代码定义了一个名为GUI Slice的抽象操作类，以及一个名为GUI Slice Reducer的类。它们都属于GUI Slice组件。

GUI Slice是一个用于管理图形用户界面（GUI）组件的状态的库。这个库定义了几个动作（action）以及一个reducer来处理这些动作。

动作（action）是GUI Slice提供的方法，它们用于更新组件的状态。这些动作可以用来设置组件的可见性、菜单面板标记、图标等。

reducer（reducer class）是一个可变的函数，它接收一个reducable state和一个action，并返回一个新的reducable state。这个新的reducable state包含了根据action做出的修改。

动作和reducer的组合，使得GUI Slice可以方便地管理组件状态，并使得组件状态的变化更加直观和易于理解。


```js
export const { setVisibility, setMenuPanelTag, setGuiAsset, setLogoImage } = GUISlice.actions;
export default GUISlice.reducer;

// export function GuiStateStore(): GuiStore {
//     const [GuiState, setGuiState] = useState(initState);
//     /**
//      * 设置各组件的可见性
//      * @param key 设置的组件
//      * @param value 可见性，true or false
//      */
//     const setVisibility = <K extends keyof componentsVisibility>(key: K, value: boolean) => {
//
//         setGuiState(state => {
//             getStorage();
//             state[key] = value;
```

这段代码是一个 JavaScript 函数，作用于在 Menu 组件中设置一个名为 "showMenuPanel" 的标签页的显示状态。

具体来说，代码首先检查给定的键是否为 "showMenuPanel" 或 "showBacklog"。如果是，就执行以下操作：

1. 将 state 对象中的 "showTextBox" 属性设置为 !value，其中 value 参数表示标签页的显示状态(true 为显示，false 为隐藏)，也就是根据给定的键的值来控制标签页的显示状态。

2. 返回一个空对象，表示经过操作后的 state 对象。

3. 在函数内部，使用一个三元表达式来检查给定的键是否为 "showMenuPanel"。如果是，就将 "showTextBox" 的值设置为 !value，这里 !value 的作用是取反，也就是如果 value 为 true，则 !value 为 false，然后将 false 赋值给 state 中的 "showTextBox"，使得标签页不再显示；如果 value 为 false，则 !value 为 true，然后将 true 赋值给 state 中的 "showTextBox"，使得标签页继续显示。

4. 在函数外部，使用 applyState 方法将上述操作后的 state 对象返回，并将其作为参数传递给 setMenuPanelTag 这个方法，从而实现设置 Menu 组件显示的标签页的目的。


```js
//             if (key === 'showMenuPanel' || key === 'showBacklog') {
//                 state['showTextBox'] = !value;
//             }
//             return {...state};
//         });
//
//     };
//
//     /**
//      * 设置Menu组件显示的标签页
//      * @param value 标签页
//      */
//     const setMenuPanelTag = (value: MenuPanelTag) => {
//
//         setGuiState(state => {
```

这段代码的作用是设置一个名为"guiAsset"的资源在状态中的当前菜单页。具体来说，它首先从全局存储中获取一个guiAsset object，然后将其状态中的currentMenuTag属性设置为传入的key，并将返回的新的guiAsset object赋值给state.currentMenuTag。

接下来，代码使用了一种三元表达式来创建一个新的guiAsset object，其中第一个参数是"key"，第二个参数是"value"，两者都是字符串类型的字符串。这个表达式创建了一个新的只读的（只读权限）guiAsset对象，并将它赋值给state.currentMenuTag。

最后，代码使用了一个箭头函数，来访问一个从state中返回的菜单对象。这个函数将guiAsset对象的value属性设置为传入的值，并将返回的新的guiAsset对象赋值给state.currentMenuTag。


```js
//             getStorage();
//             state.currentMenuTag = value;
//             return {...state};
//         });
//
//     };
//
//     /**
//      * 设置标题页的资源路径
//      * @param key 资源名
//      * @param value 资源路径
//      */
//     const setGuiAsset = <K extends keyof GuiAsset>(key: K, value: string) => {
//
//         setGuiState(state => {
```

这段代码是一个 JavaScript 函数，它的作用是创建一个新的 `GuiState` 对象，并设置其 `visibility` 属性为 `true`，同时设置其 `menuPanelTag` 属性为 `2`。

具体来说，代码首先创建一个名为 `state` 的二维数组，然后将 `value` 赋值给数组的第一个元素（也就是 `key` 的值）。

接着，代码使用对象 Spread 运算符，将 `state` 数组中的所有属性都复制一份，并将 `{...state}` 的值返回。这个值是一个新的 `state` 对象，包含了原来 `state` 对象的所有属性和值。

最后，代码创建了一个新的 `GuiState` 对象，设置了其 `visibility` 属性为 `true`，`menuPanelTag` 属性为 `2`。这里的 `GuiState` 是一个用于控制 GUI 状态的类，而 `setGuiAsset` 和 `setVisibility` 函数是用于设置 GUI 资产可见性和隐藏性的方法。


```js
//             state[key] = value;
//             return {...state};
//         });
//
//     };
//
//     return {
//         GuiState,
//         setGuiAsset,
//         setVisibility,
//         setMenuPanelTag,
//     };
// }

```