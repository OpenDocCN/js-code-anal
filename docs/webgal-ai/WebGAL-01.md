# WebGal源码解析 1

# `/packages/parser/src/scriptParser/argsParser.ts`

function processArgs(args: string, fileType: fileType): Array<arg> {
 const returnArrayList: Array<arg> = [];
 // 处理参数
 // 不要去空格
 let newArgsRaw = argsRaw.replace(/ \w/g, " ");
 // 分割参数列表
 let rawArgsList: Array<string> = newArgsRaw.split(" -");
 // 去除空格
 rawArgsList = rawArgsList.filter((e) => {
   return e !== "";
 });
 rawArgsList.forEach((e) => {
   const equalSignIndex = e.indexOf('=');
   let argName = e.slice(0, equalSignIndex);
   let argValue: string | undefined = e.slice(equalSignIndex + 1);
   if (equalSignIndex < 0) {
     argName = e;
     argValue = undefined;
   }
   // 判断是不是语音参数
   if (argName.match(/.ogg|.mp3|.wav/)) {
     returnArrayList.push({
       key: "vocal",
       value: assetSetter(e, fileType.vocal)
     });
   } else {
     // 判断是不是省略参数
     if (argValue === undefined) {
       returnArrayList.push({
         key: argName,
         value: true
       });
     } else {
       // 是字符串描述的布尔值
       if (argValue === "true" || argValue === "false") {
         returnArrayList.push({
           key: argName,
           value: argValue === "true"
         });
       } else {
         // 是数字
         if (!isNaN(Number(argValue))) {
           returnArrayList.push({
             key: argName,
             value: Number(argValue)
           });
         } else {
           // 是普通参数
           returnArrayList.push({
             key: argName,
             value: argValue
           });
         }
       }
     }
   }
 });
 return returnArrayList;
};


```js
import {arg} from "../interface/sceneInterface";
import {fileType} from "../interface/assets";

/**
 * 参数解析器
 * @param argsRaw 原始参数字符串
 * @param assetSetter
 * @return {Array<arg>} 解析后的参数列表
 */
export function argsParser(argsRaw: string, assetSetter: (fileName: string, assetType: fileType) => string): Array<arg> {
  const returnArrayList: Array<arg> = [];
  // 处理参数
  // 不要去空格
  let newArgsRaw = argsRaw.replace(/ /g, " ");
  // 分割参数列表
  let rawArgsList: Array<string> = newArgsRaw.split(" -");
  // 去除空字符串
  rawArgsList = rawArgsList.filter((e) => {
    return e !== "";
  });
  rawArgsList.forEach((e) => {
    const equalSignIndex = e.indexOf('=');
    let argName = e.slice(0, equalSignIndex);
    let argValue: string | undefined = e.slice(equalSignIndex + 1);
    if (equalSignIndex < 0) {
      argName = e;
      argValue = undefined;
    }
    // 判断是不是语音参数
    if (argName.match(/.ogg|.mp3|.wav/)) {
      returnArrayList.push({
        key: "vocal",
        value: assetSetter(e, fileType.vocal)
      });
    } else {
      // 判断是不是省略参数
      if (argValue === undefined) {
        returnArrayList.push({
          key: argName,
          value: true
        });
      } else {
        // 是字符串描述的布尔值
        if (argValue === "true" || argValue === "false") {
          returnArrayList.push({
            key: argName,
            value: argValue === "true"
          });
        } else {
          // 是数字
          if (!isNaN(Number(argValue))) {
            returnArrayList.push({
              key: argName,
              value: Number(argValue)
            });
          } else {
            // 是普通参数
            returnArrayList.push({
              key: argName,
              value: argValue
            });
          }
        }
      }
    }
  });
  return returnArrayList;
};

```

# `/packages/parser/src/scriptParser/assetsScanner.ts`



该函数接受三个参数：`command`表示要执行的操作类型，`content`是要扫描的内容，`args`是参数列表。函数内部使用了变量`hasVocalArg`来记录参数`vocal`的个数，如果没有传递`vocal`参数，则返回的`asset`列表中，每个`asset`的`vocal`属性就是`false`。

函数处理操作类型为`say`、`changeBg`、`changeFigure`、`miniAvatar`、`video`和`bgm`时的情况。对于这些操作类型，函数会根据传递的参数内容，添加对应的`asset`到返回的`asset`列表中。

对于操作类型为`None`或`'none'`的情况，函数直接返回包含这些内容的`asset`列表。


```js
import { arg, commandType, IAsset } from '../interface/sceneInterface';
import { fileType } from '../interface/assets';

/**
 * 根据语句类型、语句内容、参数列表，扫描该语句可能携带的资源
 * @param command 语句类型
 * @param content 语句内容
 * @param args 参数列表
 * @return {Array<IAsset>} 语句携带的参数列表
 */
export const assetsScanner = (command: commandType, content: string, args: Array<arg>): Array<IAsset> => {
  let hasVocalArg = false;
  const returnAssetsList: Array<IAsset> = [];
  if (command === commandType.say) {
    args.forEach((e) => {
      if (e.key === 'vocal') {
        hasVocalArg = true;
        returnAssetsList.push({
          name: e.value as string,
          url: e.value as string,
          lineNumber: 0,
          type: fileType.vocal,
        });
      }
    });
  }
  if (content === 'none' || content === '') {
    return returnAssetsList;
  }
  // 处理语句携带的资源
  if (command === commandType.changeBg) {
    returnAssetsList.push({
      name: content,
      url: content,
      lineNumber: 0,
      type: fileType.background,
    });
  }
  if (command === commandType.changeFigure) {
    returnAssetsList.push({
      name: content,
      url: content,
      lineNumber: 0,
      type: fileType.figure,
    });
  }
  if (command === commandType.miniAvatar) {
    returnAssetsList.push({
      name: content,
      url: content,
      lineNumber: 0,
      type: fileType.figure,
    });
  }
  if (command === commandType.video) {
    returnAssetsList.push({
      name: content,
      url: content,
      lineNumber: 0,
      type: fileType.video,
    });
  }
  if (command === commandType.bgm) {
    returnAssetsList.push({
      name: content,
      url: content,
      lineNumber: 0,
      type: fileType.bgm,
    });
  }
  return returnAssetsList;
};

```

# `/packages/parser/src/scriptParser/commandParser.ts`

这段代码定义了一个名为`commandParser`的函数，它接受三个参数：`commandRaw`、`ADD_NEXT_ARG_LIST`和`SCRIPT_CONFIG`。通过解析`commandRaw`并应用`ADD_NEXT_ARG_LIST`，然后根据`SCRIPT_CONFIG`中的配置来确定返回的命令类型。

函数内部首先定义了一个`parsedCommand`的对象，用于存储处理后的命令信息。然后执行以下操作：

1. 检查`commandRaw`是否以`say`开头，如果是，则执行以下操作：

  1. 从`ADD_NEXT_ARG_LIST`中提取`speaker`参数，并将其添加到`additionalArgs`数组中。

2. 对于不是`say`开头的命令，执行以下操作：

  1. 如果`SCRIPT_CONFIG`中的配置是`greeting`，则将`additionalArgs`数组长度设置为`length`，否则将`additionalArgs`数组长度设置为0。

  2. 将`additionalArgs`数组长度设置为`length`，这意味着当`parsedCommand.additionalArgs`的长度达到`length`时，将不再添加新的参数。

3. 返回`parsedCommand`对象。

最后，函数内部会根据`SCRIPT_CONFIG`中的配置对返回的命令进行进一步处理，以确保其正确性和实用性。


```js
import { commandType, parsedCommand } from "../interface/sceneInterface";

/**
 * 处理命令
 * @param commandRaw
 * @param ADD_NEXT_ARG_LIST
 * @param SCRIPT_CONFIG
 * @return {parsedCommand} 处理后的命令
 */
export const commandParser = (commandRaw: string, ADD_NEXT_ARG_LIST:any, SCRIPT_CONFIG:any): parsedCommand => {
  let returnCommand: parsedCommand = {
    type: commandType.say, // 默认是say
    additionalArgs: []
  };
  // 开始处理命令内容
  const type: commandType = getCommandType(commandRaw, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);
  returnCommand.type = type;
  // 如果是对话，加上额外的参数
  if (type === commandType.say && commandRaw!== 'say') {
    returnCommand.additionalArgs.push({
      key: "speaker",
      value: commandRaw
    });
  }
  returnCommand = addNextArg(returnCommand, type, ADD_NEXT_ARG_LIST);
  return returnCommand;
};

```

这段代码定义了一个名为 `getCommandType` 的函数，用于根据传入的 `command` 原始值判断是什么命令，并返回相应的命令类型。

函数接收三个参数：`command`，`ADD_NEXT_ARG_LIST` 和 `SCRIPT_CONFIG`。函数内部首先检查 `command` 是否匹配 `/if/` 表达式，如果是，则返回命令类型 `commandType.if`。否则，函数遍历 `SCRIPT_CONFIG` 数组，并将找到的命令类型存储在一个名为 `commandMap` 的 Map 中。最后，函数根据存储在 `commandMap` 中的命令类型返回相应的命令类型。

如果 `command` 不存在，或者 `SCRIPT_CONFIG` 数组为空，则函数返回命令类型 `commandType.say`。


```js
/**
 * 根据command原始值判断是什么命令
 * @param command command原始值
 * @param ADD_NEXT_ARG_LIST
 * @param SCRIPT_CONFIG
 * @return {commandType} 得到的command类型
 */
function getCommandType(command: string, ADD_NEXT_ARG_LIST:any, SCRIPT_CONFIG:any): commandType {
  // if (command.match(/if/)) {
  //   return commandType.if;
  // }
  const commandMap = new Map();
  SCRIPT_CONFIG.forEach((e:any) => {
    commandMap.set(e.scriptString, e.scriptType);
  });
  if (commandMap.has(command)) {
    return commandMap.get(command);
  } else return commandType.say;
}

```

该函数的作用是检查给定的 `thisCommandType` 是否包含在 `ADD_NEXT_ARG_LIST` 中。如果是，则将 `key` 属性设置为 `true`，并将 `value` 属性设置为 `true`。这将添加名为 `next` 的附加参数，其值为 `true`。返回修改后的 `commandToParse` 对象。


```js
function addNextArg(commandToParse: parsedCommand, thisCommandType: commandType, ADD_NEXT_ARG_LIST:any) {
  if (ADD_NEXT_ARG_LIST.includes(thisCommandType)) {
    commandToParse.additionalArgs.push({
      key: "next",
      value: true
    });
  }
  return commandToParse;
}

```

# `/packages/parser/src/scriptParser/contentParser.ts`

这段代码定义了一个名为 `contentParser` 的函数，它接受一个原始语句内容和一种语句类型以及一个 `assetSetter` 函数作为参数。函数的主要作用是将文件名更改为绝对地址或相对地址，根据使用情况而定。

具体来说，当语句类型为 `commandType.playEffect`、`commandType.changeBg`、`commandType.changeFigure` 或 `commandType.bgm` 时，函数会将 `assetSetter` 函数的第一个参数（原始语句内容）更改为相对于原文件名的地址，并将第二个参数（文件类型）更改为 `fileType.vocal`、`fileType.background` 或 `fileType.figure`。当语句类型为 `commandType.callScene`、`commandType.changeScene` 或 `commandType.miniAvatar` 时，函数会将 `assetSetter` 函数的第一个参数（原始语句内容）更改为相对于原文件名的地址，并将第二个参数（文件类型）更改为 `fileType.scene` 或 `fileType.figure`。

当语句类型为 `commandType.choose` 时，函数会将 `getChooseContent` 函数作为第一个参数，并将 `assetSetter` 函数的第一个参数（原始语句内容）作为第二个参数。

当语句类型为 `commandType.unlockBgm` 或 `commandType.unlockCg` 时，函数会将 `assetSetter` 函数的第一个参数（原始语句内容）更改为相对于原文件名的地址，并将第二个参数（文件类型）更改为 `fileType.bgm` 或 `fileType.background`。

如果语句类型不符合上述任何一种情况，函数会将原始语句内容返回。


```js
import { commandType } from "../interface/sceneInterface";
import { fileType } from "../interface/assets";

/**
 * 解析语句内容的函数，主要作用是把文件名改为绝对地址或相对地址（根据使用情况而定）
 * @param contentRaw 原始语句内容
 * @param type 语句类型
 * @param assetSetter
 * @return {string} 解析后的语句内容
 */
export const contentParser = (contentRaw: string, type: commandType, assetSetter: any) => {
  if (contentRaw === "none" || contentRaw === "") {
    return "";
  }
  switch (type) {
    case commandType.playEffect:
      return assetSetter(contentRaw, fileType.vocal);
    case commandType.changeBg:
      return assetSetter(contentRaw, fileType.background);
    case commandType.changeFigure:
      return assetSetter(contentRaw, fileType.figure);
    case commandType.bgm:
      return assetSetter(contentRaw, fileType.bgm);
    case commandType.callScene:
      return assetSetter(contentRaw, fileType.scene);
    case commandType.changeScene:
      return assetSetter(contentRaw, fileType.scene);
    case commandType.miniAvatar:
      return assetSetter(contentRaw, fileType.figure);
    case commandType.video:
      return assetSetter(contentRaw, fileType.video);
    case commandType.choose:
      return getChooseContent(contentRaw, assetSetter);
    case commandType.unlockBgm:
      return assetSetter(contentRaw, fileType.bgm);
    case commandType.unlockCg:
      return assetSetter(contentRaw, fileType.background);
    default:
      return contentRaw;
  }
};

```

此函数的作用是获取一个字符串，其中包含一个或多个由冒号分隔的新闻组，并将其中的内容根据给出的设置器(assetSetter)进行相应的处理。

具体来说，函数将原始内容字符串内容通过分割符"|"拆分成一个或多个新闻组。对于每个新闻组，函数 splitting出一对"键值对"(即冒号中的键和值)，并将它们存储在chooseKeyList和chooseValueList中。

接下来，函数使用map方法遍历chooseValueList中的每个值，并将它们与传入的设置器进行比较。如果设置器返回了一个对象，函数将返回该对象，否则继续使用原始值。

最后，函数将这些处理后的新闻组和键/值对组合成一个新的字符串，并将它返回。


```js
function getChooseContent(contentRaw: string, assetSetter: any): string {
  const chooseList = contentRaw.split("|");
  const chooseKeyList: Array<string> = [];
  const chooseValueList: Array<string> = [];
  for (const e of chooseList) {
    chooseKeyList.push(e.split(":")[0] ?? "");
    chooseValueList.push(e.split(":")[1] ?? "");
  }
  const parsedChooseList = chooseValueList.map((e) => {
    if (e.match(/\./)) {
      return assetSetter(e, fileType.scene);
    } else {
      return e;
    }
  });
  let ret = "";
  for (let i = 0; i < chooseKeyList.length; i++) {
    if (i !== 0) {
      ret = ret + "|";
    }
    ret = ret + `${chooseKeyList[i]}:${parsedChooseList[i]}`;
  }
  return ret;
}

```

# `/packages/parser/src/scriptParser/scriptParser.ts`

type: string,
command: { type: string, index: number }
object prototype,
当遇到一个给定的命令时，我首先检查给定的参数中是否包含一个名为 speaker 的参数。如果是，则我跳过该参数。否则，我将命令和参数列表作为对象实例化，将新的sentenceRaw对象和命令作为新对象添加到args数组中。如果给定的命令不是 say，则将新的sentenceRaw对象和命令作为新对象添加到args数组中。如果给定的命令是say，则我解析命令和给定的附加参数，然后根据解析出的命令类型生成新的sentenceRaw对象。在解析附加参数时，我首先使用正则表达式将附加参数区域分割为命令区域和内容区域。在生成新的sentenceRaw对象时，我继续使用给定的附加参数。最后，我根据解析出的命令类型生成新的命令对象，并将其添加到args数组中。如果给定的命令是say，则我继续解析附加参数，并将解析出的命令和附加参数添加到args数组中。然后，我根据解析出的命令类型生成新的命令对象，并将其添加到args数组中。如果给定的命令不是say，则我继续解析附加参数，并将解析出的命令和附加参数添加到args数组中。


```js
import { arg, commandType, IAsset, ISentence, parsedCommand } from "../interface/sceneInterface";
import { commandParser } from "./commandParser";
import { argsParser } from "./argsParser";
import { contentParser } from "./contentParser";
import { assetsScanner } from "./assetsScanner";
import { subSceneScanner } from "./subSceneScanner";

/**
 * 语句解析器
 * @param sentenceRaw 原始语句
 * @param assetSetter
 * @param ADD_NEXT_ARG_LIST
 * @param SCRIPT_CONFIG
 */
export const scriptParser = (sentenceRaw: string, assetSetter: any, ADD_NEXT_ARG_LIST: any, SCRIPT_CONFIG: any): ISentence => {
  let command: commandType; // 默认为对话
  let content: string; // 语句内容
  let subScene: Array<string>; // 语句携带的子场景（可能没有）
  const args: Array<arg> = []; // 语句参数列表
  let sentenceAssets: Array<IAsset>; // 语句携带的资源列表
  let parsedCommand: parsedCommand; // 解析后的命令
  let commandRaw: string;

  // 正式开始解析

  // 去分号，前面已做，这里不再需要
  let newSentenceRaw = sentenceRaw.split(";")[0];
  if (newSentenceRaw === "") {
    // 注释提前返回
    return {
      command: commandType.comment, // 语句类型
      commandRaw: "comment", // 命令原始内容，方便调试
      content: sentenceRaw.split(";")[1] ?? "", // 语句内容
      args: [{ key: "next", value: true }], // 参数列表
      sentenceAssets: [], // 语句携带的资源列表
      subScene: [] // 语句携带的子场景
    };
  }
  // 截取命令
  const getCommandResult = /:/.exec(newSentenceRaw);
  /**
   * 拆分命令和语句，同时处理连续对话。
   */
  // 没有command，说明这是一条连续对话或单条语句
  if (getCommandResult === null) {
    commandRaw = newSentenceRaw;
    parsedCommand = commandParser(commandRaw, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);
    command = parsedCommand.type;
    for (const e of parsedCommand.additionalArgs) {
      // 由于是连续对话，所以我们去除 speaker 参数。
      if (command === commandType.say && e.key === "speaker") {
        continue;
      }
      args.push(e);
    }
  } else {
    commandRaw = newSentenceRaw.substring(0, getCommandResult.index);
    // 划分命令区域和content区域
    newSentenceRaw = newSentenceRaw.substring(getCommandResult.index + 1, newSentenceRaw.length);
    parsedCommand = commandParser(commandRaw, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);
    command = parsedCommand.type;
    for (const e of parsedCommand.additionalArgs) {
      args.push(e);
    }
  }
  // 截取参数区域
  const getArgsResult = / -/.exec(newSentenceRaw);
  // 获取到参数
  if (getArgsResult) {
    const argsRaw = newSentenceRaw.substring(getArgsResult.index, sentenceRaw.length);
    newSentenceRaw = newSentenceRaw.substring(0, getArgsResult.index);
    for (const e of argsParser(argsRaw, assetSetter)) {
      args.push(e);
    }
  }
  content = contentParser(newSentenceRaw, command, assetSetter); // 将语句内容里的文件名转为相对或绝对路径
  sentenceAssets = assetsScanner(command, content, args); // 扫描语句携带资源
  subScene = subSceneScanner(command, content); // 扫描语句携带子场景
  return {
    command: command, // 语句类型
    commandRaw: commandRaw, // 命令原始内容，方便调试
    content: content, // 语句内容
    args: args, // 参数列表
    sentenceAssets: sentenceAssets, // 语句携带的资源列表
    subScene: subScene // 语句携带的子场景
  };
};

```

# `/packages/parser/src/scriptParser/subSceneScanner.ts`

这段代码定义了一个名为 `subSceneScanner` 的函数，它会接受一个 `command` 参数和一个 `content` 参数。函数内部处理 `command` 参数，如果它为 `commandType.changeScene` 或 `commandType.callScene`，则执行 `content` 并将其添加到 `subSceneList` 中。如果 `command` 参数为 `commandType.choose`，则它将 `content`  splitting 为 `||` 并只取第二个值，如果匹配 `/./` 的话，将添加到 `subSceneList` 中。

函数的作用是扫描一个场景中的子场景，根据不同的 `command` 参数返回相应的子场景列表。它只能在命令类型为 `commandType.changeScene` 或 `commandType.callScene` 时使用，而且它并不接受一个命令和它的参数作为参数。


```js
/**
 * 扫描子场景
 * @param content 语句内容
 * @return {Array<string>} 子场景列表
 */
import { commandType } from "../interface/sceneInterface";

export const subSceneScanner = (command: commandType, content: string): Array<string> => {
  const subSceneList: Array<string> = [];
  if (command === commandType.changeScene || command === commandType.callScene) {
    subSceneList.push(content);
  }
  if (command === commandType.choose) {
    const chooseList = content.split("|");
    const chooseValue = chooseList.map((e) => e.split(":")[1] ?? "");
    chooseValue.forEach((e) => {
      if (e.match(/\./)) {
        subSceneList.push(e);
      }
    });
  }
  return subSceneList;
};

```

# `/packages/parser/test/debug.ts`

这段代码的作用是读取一个名为 "var.txt" 的文件，并将其内容转换为字符串，然后使用 SceneParser 解析该内容。具体来说，代码中定义了一个名为 "debug" 的函数，该函数使用 fs/promises 库的 readFile 方法读取到一个名为 "var.txt" 的文件。然后，将读取到的内容转换为字符串，并使用 SceneParser 中的 parse 方法将该内容解析为 SceneScript 配置对象。最后，将解析出的配置对象打印到控制台。


```js
import *  as fsp from "fs/promises";
import SceneParser, {ADD_NEXT_ARG_LIST, SCRIPT_CONFIG} from "../src";


async function debug() {
  const sceneRaw = await fsp.readFile('test/test-resources/var.txt');
  const sceneText = sceneRaw.toString();

  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const result = parser.parse(sceneText, "var", "/var.txt");
  console.log(result)
  const configFesult = parser.parseConfig(`
```

这段代码是一个JavaScript脚本，作用是显示一个WebGAL游戏的配置信息。

具体来说，它包括以下几部分：

1. `Game_name：欢迎使用WebGAL！`：这是游戏的主标题，显示欢迎使用WebGAL游戏的文本。

2. `Game_key:0f86dstRf`：这是一个64位游戏键，用于控制游戏难度和模式等设置。

3. `Title_img:WebGAL_New_Enter_Image.png`：这是游戏的标题图片，一个包含游戏名称的PNG文件。

4. `Title_bgm:s_Title.mp3`：这是游戏的背景音乐，一个名为`s_Title.mp3`的MP3文件。

5. `Title_logos: 1.png | 2.png | Image Logo.png| -show -active=false -add=op! -count=3;This is a fake config, do not reference anything.`：这是游戏中的三个logo，分别是1.png、2.png和3.png，它们以图片形式显示在游戏界面上。此外，还包含了一个配置文件，包含了上述内容的详细配置。

6. `console.log(configFesult)`：这个是console.log函数，输出游戏配置结果。

7. `console.log(parser.stringifyConfig(configFesult))`：这个是console.log函数，输出配置文件的字符串表示形式。

8. `debug()`：这个是javascript的一个函数，用于输出调试信息。

总之，这段代码的主要作用是输出一个WebGAL游戏的配置信息，以便开发人员更好地了解游戏的设置。


```js
Game_name:欢迎使用WebGAL！;
Game_key:0f86dstRf;
Title_img:WebGAL_New_Enter_Image.png;
Title_bgm:s_Title.mp3;
Title_logos: 1.png | 2.png | Image Logo.png| -show -active=false -add=op! -count=3;This is a fake config, do not reference anything.
  `)
  console.log(configFesult)
  console.log(parser.stringifyConfig(configFesult))
}

debug();

```

# `/packages/parser/test/parser.test.ts`



这段代码的作用是测试SceneParser组件的功能，以验证它是否能够解析包含标签和结束符的文本文件，并返回一个预期的结果。

具体来说，这段代码将读取一个名为"start.txt"的文本文件，并将其内容转换为字符串。然后，它创建了一个SceneParser对象，该对象使用给定的配置，将文本文件中的内容解析为FSP（文件系统Promises）对象，并返回一个预期的结果对象。

在测试中，它使用parser.parse函数将文本文件中的内容解析为命令、命令的原始内容、内容和参数列表、配置和场景列表。它将这些FSP对象传递给预期Sentence对象，然后使用expect函数验证结果是否符合预期。如果结果符合预期，expect函数将返回。


```js
import SceneParser from "../src/index";
import {ADD_NEXT_ARG_LIST, SCRIPT_CONFIG} from "../src/config/scriptConfig";
import {expect, test} from "vitest";
import {commandType, ISentence} from "../src/interface/sceneInterface";
import * as fsp from 'fs/promises'
import {fileType} from "../src/interface/assets";

test("label", async () => {

  const sceneRaw = await fsp.readFile('test/test-resources/start.txt');
  const sceneText = sceneRaw.toString();

  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const result = parser.parse(sceneText, "start", "/start.txt");
  const expectSentenceItem: ISentence = {
    command: commandType.label,
    commandRaw: "label",
    content: "end",
    args: [
      {key: "next", value: true}
    ],
    sentenceAssets: [],
    subScene: []
  };
  expect(result.sentenceList).toContainEqual(expectSentenceItem);
});

```

该代码的作用是测试一个名为"args"的函数。该函数使用了两个前端的资源文件："start.txt"和"test-resources/start.txt"，并将它们的内容存储到变量中。"start.txt"文件可能包含一个描述如何改变场景的命令序列和一个用于指定新命令的文本。

函数中定义了一个名为"parser"的实例，它使用给定的"assetList"参数来解析文本。解析器的语法看起来像这样：
```jsphp
const parser = new SceneParser<string, string>((assetList) => {}, (fileName, assetType) => {
 return fileName;
}, ADD_NEXT_ARGS, SCRIPT_CONFIG);
```
这个实例接受两个参数：一个是一个表示资产列表的函数，另一个是一个表示文件名的函数。这些函数可能会接受其他参数，但我们在该代码中没有使用它们。

函数中使用了一个匿名函数"async () => { ... }"，这个函数内部使用了一组初始化的函数式编程技巧，这些技巧可能会在将来的代码中定义新的命令、解析文本或其他操作。

函数内部使用了一个变量"sceneText"来存储读取到的"start.txt"文件的输出。变量还被赋值为使用"parser"解析"sceneText"的结果。

函数中定义了一个变量"expectSentenceItem"，它是一个包含有关给定输入的期望输出的对象。在这个对象中，"命令"属性表示要执行的操作类型，它是'commandType.changeFigure'。"命令Raw"属性包含一个字符串，表示执行操作的原始命令。"content"属性包含要插入到场景中的新命令的原始文件名。"args"属性包含一个由键值对组成的数组，表示将插入到场景中的新命令的参数。"sentenceAssets"属性包含一个包含与给定命令相关的场景文件的引用。"subScene"属性包含一个或多个与给定命令相关的子场景。

最后，函数中使用"const result = parser.parse(sceneText, 'start', '/start.txt');"来解析"sceneText"并获取"result"变量。然后，它使用"expect(result.sentenceList).toContainEqual(expectSentenceItem);"来检查解析结果中是否包含期望的输出对象。如果解析成功，则函数内部可能会执行其他操作，例如将新命令添加到场景中。


```js
test("args", async () => {

  const sceneRaw = await fsp.readFile('test/test-resources/start.txt');
  const sceneText = sceneRaw.toString();

  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const result = parser.parse(sceneText, "start", "/start.txt");
  const expectSentenceItem: ISentence = {
    command: commandType.changeFigure,
    commandRaw: "changeFigure",
    content: "m2.png",
    args: [
      {key: "left", value: true},
      {key: "next", value: true}
    ],
    sentenceAssets: [{name: "m2.png", url: 'm2.png', type: fileType.figure, lineNumber: 0}],
    subScene: []
  };
  expect(result.sentenceList).toContainEqual(expectSentenceItem);
});

```

这段代码是一个网络截图选择器，用于在网页中查找指定的截图位置并将其显示出来。它主要做了以下几件事情：

1. 读取一个名为 "choose.txt" 的文本文件，并将其内容存储在变量 "sceneText" 中。
2. 定义了一个名为 "SceneParser" 的类，用于解析场景文件。这个类接受一个名为 "assetList" 的参数，它会将文件名和文件类型从参数中读取出来，并返回文件名。
3. 将解析场景文件的结果存储在一个名为 "parser" 的变量中。
4. 将 "sceneText" 和解析场景文件的结果作为参数，传递给 "parser.parse" 方法，并将结果存储在一个名为 "result" 的变量中。
5. 检查 "result.sentenceList" 是否包含一个 ISentence 对象，如果是，就说明解析成功。ISentence 对象包含以下属性：命令类型为 choose，命令原始值为 "choose"，内容为空，args为空，sentenceAssets为空，subScene为空。
6. 输出解析结果。


```js
test("choose", async () => {

  const sceneRaw = await fsp.readFile('test/test-resources/choose.txt');
  const sceneText = sceneRaw.toString();

  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const result = parser.parse(sceneText, "choose", "/choose.txt");
  const expectSentenceItem: ISentence = {
    command: commandType.choose,
    commandRaw: "choose",
    content: "",
    args: [],
    sentenceAssets: [],
    subScene: []
  };
  expect(result.sentenceList).toContainEqual(expectSentenceItem);
});

```

这段代码是一个用于测试 SceneScript 语法是否正确性的脚本。SceneScript 是一种用于制作交互式场景的编程语言，这段代码的作用是读取一个名为 "long-script.txt" 的文本文件，并将其内容转换成 SceneScript 语法，然后使用 SceneParser 将 SceneScript 语法解析成一个或多个 Sentence 对象，最后检查生成的 Sentence 对象是否符合预期的类型。

具体来说，代码首先读取一个名为 "long-script.txt" 的文本文件，并将其内容保存到变量 "sceneRaw" 中。然后，代码使用 fsp.readFile() 函数将 "sceneRaw" 中的内容转换成字符串类型，并将其保存到变量 "sceneText" 中。

接下来，代码定义了一个名为 parser 的 SceneParser 对象，用于将 SceneScript 语法解析成 Sentence 对象。parser 对象包含两个构造函数，第一个函数用于指定从哪个文件中读取数据，第二个函数用于指定解析函数中的配置选项。parser 对象还包含一个 parse() 方法，用于将 SceneScript 语法解析成一个或多个 Sentence 对象。

然后，代码使用 console.log() 函数打印出 "line count" 的值，即文本文件中行数的总和。接着，代码使用 console.time() 函数记录下 Parse() 方法执行的时间，以便在之后进行比较。然后，代码调用 parser.parse() 方法将 sceneText 解析成一个或多个 Sentence 对象，并将结果存储到变量 result 中。

最后，代码使用 expect() 函数检查生成的 Sentence 对象是否符合预期的类型。预期的 Sentence 对象是一个 ISentence 对象，其中包含命令类型、命令原始代码、内容、参数列表和子场景列表。如果生成的 Sentence 对象符合这个预期，expect() 函数将返回一个 Promise 对象，否则会抛出一个错误。


```js
test("long-script", async () => {

  const sceneRaw = await fsp.readFile('test/test-resources/long-script.txt');
  const sceneText = sceneRaw.toString();

  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  console.log('line count:', sceneText.split('\n').length)
  console.time('parse-time-consumed')
  const result = parser.parse(sceneText, "start", "/start.txt");
  console.timeEnd('parse-time-consumed')
  const expectSentenceItem: ISentence = {
    command: commandType.label,
    commandRaw: "label",
    content: "end",
    args: [
      {key: "next", value: true}
    ],
    sentenceAssets: [],
    subScene: []
  };
  expect(result.sentenceList).toContainEqual(expectSentenceItem);
});

```

这段代码是一个用于测试中的一种场景，主要作用是读取一个名为 "var.txt" 的文件，并将其内容解析为 Scenep向东巴 Cand ID，然后输出结果。
```jsjavascript
async function test("var", async () => {
   const sceneRaw = await fsp.readFile('test/test-resources/var.txt');
   const sceneText = sceneRaw.toString();

   const parser = new SceneParser((assetList) => {
       assetList.push(assetRegistry.config.assetRegistry.resolveAssetPath("/path/to/expect.asset")));
   }, (fileName, assetType) => {
       return fileName;
   }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

   const result = parser.parse(sceneText, "var", "/var.txt");
   const expectSentenceItem: ISentence = {
       command: commandType.say,
       commandRaw: "WebGAL",
       content: "a=1?",
       args: [{key: 'speaker', value: 'WebGAL'}, {key: 'when', value: "a==1"}],
       sentenceAssets: [],
       subScene: []
   };
   expect(result.sentenceList).toContainEqual(expectSentenceItem);
});
```
首先，这段代码读取一个名为 "var.txt" 的文件，并将其内容存储在变量 `sceneRaw` 中。
```jsjavascript
const sceneRaw = await fsp.readFile('test/test-resources/var.txt');
```
接着，这段代码使用 `fsp.writeFile` 方法将 `sceneRaw` 保存到一个名为 "var.txt" 的文件中。
```jsjavascript
const sceneText = sceneRaw.toString();
await fsp.writeFile('test/test-resources/var.txt', sceneText);
```
然后，这段代码定义了一个 `SceneParser` 类，用于读取和解析 JSON 文件中的 Sentence 对象。
```jsarduino
const parser = new SceneParser((assetList) => {
   assetList.push(assetRegistry.config.assetRegistry.resolveAssetPath("/path/to/expect.asset")));
}, (fileName, assetType) => {
   return fileName;
}, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);
```
在这个 `SceneParser` 类中，首先将 `assetList` 属性中的内容添加到 `assetRegistry` 实例的配置属性中，然后定义了 `fileName` 和 `assetType` 的处理函数。
```jsarduino
const result = parser.parse(sceneText, "var", "/var.txt");
```
接下来，定义了一个 `expectSentenceItem` 对象，用于表示期望的 Sentence 对象。
```jsarduino
const expectSentenceItem: ISentence = {
   command: commandType.say,
   commandRaw: "WebGAL",
   content: "a=1?",
   args: [{key: 'speaker', value: 'WebGAL'}, {key: 'when', value: "a==1"}],
   sentenceAssets: [],
   subScene: []
};
```
最后，使用 `parser.parse` 方法将 `sceneText` 解析为 Sentence 对象，并输出结果。
```jsarduino
const result = parser.parse(sceneText, "var", "/var.txt");
const expectSentenceList = result.sentenceList;
```


```js
test("var", async () => {

  const sceneRaw = await fsp.readFile('test/test-resources/var.txt');
  const sceneText = sceneRaw.toString();

  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const result = parser.parse(sceneText, "var", "/var.txt");
  const expectSentenceItem: ISentence = {
    command: commandType.say,
    commandRaw: "WebGAL",
    content: "a=1?",
    args: [{key: 'speaker', value: 'WebGAL'}, {key: 'when', value: "a==1"}],
    sentenceAssets: [],
    subScene: []
  };
  expect(result.sentenceList).toContainEqual(expectSentenceItem);
});

```

这段代码是一个AJAX调用，它使用了一个自定义的JavaScript类（SceneParser）来解析WebGL（Web Graphics Library）配置文件。这个测试脚本的作用是验证SceneParser类是否可以正确解析一个基本的WebGL配置文件。

具体来说，这段代码会执行以下操作：

1. 创建一个SceneParser实例，这个实例有一个`parseConfig`方法，它接受一个字符串参数，表示一个WebGL配置文件的前缀部分。
2. 使用`parseConfig`方法解析字符串中的配置信息。
3. 通过调用`expect`方法，验证生成的结果是否符合预期。具体来说，期望的结果应该是一个object，它包含以下几个属性和选项：
	* `command`：期望的命令，这里是'Title_logos'。
	* `args`：期望的参数，是一个数组，包含三个图像文件的路径。
	* `options`：期望的选项，是一个包含多个选项的对象。选项包括：
		+ `show`：期望是否在运行时显示器中显示元素。
		+ `active`：期望是否在运行时启用元素的动画效果。
		+ `add`：期望是否将指定的选项添加到生成的配置中。
		+ `count`：期望将选项的数量。
	* `expect`：期望的结果应该是一个object，它应该包含上述属性和选项中的所有属性。


```js
test("config", async () => {
  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const configFesult = parser.parseConfig(`
Game_name:欢迎使用WebGAL！;
Game_key:0f86dstRf;
Title_img:WebGAL_New_Enter_Image.png;
Title_bgm:s_Title.mp3;
Title_logos: 1.png | 2.png | Image Logo.png| -show -active=false -add=op! -count=3;This is a fake config, do not reference anything.
  `)
  expect(configFesult).toContainEqual({
    command: 'Title_logos',
    args: ['1.png', '2.png', 'Image Logo.png'],
    options: [
      {key: 'show', value: true},
      {key: 'active', value: false},
      {key: 'add', value: 'op!'},
      {key: 'count', value: 3},
    ]
  })
})

```

这段代码是一个Axios测试用例，用于测试如何使用WebGAL（Web Graphics Library）将JSON配置文件解析为配置对象。

具体来说，这段代码的作用是：

1. 定义了一个名为`parser`的Axios配置器对象，用于将JSON配置文件解析为配置对象。

2. 使用`parser`将一个JSON配置文件（由`parser`定义的函数返回）解析为配置对象。

3. 使用`configFesult`变量存储解析后的配置对象。

4. 定义了一个`stringifyResult`变量，用于存储JSON配置文件的字符串表示。

5. 定义了一个`configResult2`变量，用于存储解析后的配置对象。

6. 使用`expect`函数，对`configResult2`的类型进行判断，确保它是一个预期的对象。

7. 输出结果：如果`configResult2`是一个预期的对象，那么输出预览图（通常是一个具有阴影的图片）的宽度和高度。

由于`stringifyResult`和`configResult2`都只是对配置文件的`JSON`格式的字符串表示，因此可以确定`configResult2`的类型与原始配置文件的`JSON`格式是一致的。


```js
test("config-stringify", async () => {
  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const configFesult = parser.parseConfig(`
Game_name:欢迎使用WebGAL！;
Game_key:0f86dstRf;
Title_img:WebGAL_New_Enter_Image.png;
Title_bgm:s_Title.mp3;
Title_logos: 1.png | 2.png | Image Logo.png| -show -active=false -add=op! -count=3;This is a fake config, do not reference anything.
  `)
  const stringifyResult = parser.stringifyConfig(configFesult);
  const configResult2 = parser.parseConfig(stringifyResult)
  expect(configResult2).toContainEqual({
    command: 'Title_logos',
    args: ['1.png', '2.png', 'Image Logo.png'],
    options: [
      {key: 'show', value: true},
      {key: 'active', value: false},
      {key: 'add', value: 'op!'},
      {key: 'count', value: 3},
    ]
  })
})


```

这段代码是一个使用JavaScript的实验，旨在测试SceneParser库的功能。SceneParser是一个用于解析JSON或XML文件中的Scene对象的工具类。在这个实验中，我们创建了一个SceneParser实例，该实例使用一个函数解析文件中的Scene对象。

该函数的第一个参数是一个包含Scene对象的数组，每个Scene对象都会经过以下处理：我们创建了一个Speaker对象，该对象的值设置为我们想要从文件中提取的说话者的ID。然后我们创建了一个函数，该函数将在文件中找到某个JSON或XML对象，并返回其内容。最后，我们创建了一个ADD\_NEXT\_ARG\_LIST和一个SCRIPT\_CONFIG函数，这些函数用于我们将解析过程与实际运行的语句一起发送到控制台。

在我们的代码中，我们使用const parser = new SceneParser(addNextArgumentList, scriptConfig)函数来创建一个SceneParser实例。然后我们使用parser.parse()函数将我们要解析的文件内容。我们还有一个expect()函数，用于断言从解析结果中返回的sentenceList包含我们期望的内容。在这个例子中，我们期望的结果是{command: {key: 'say', value: '123'}, args: {key: 'speaker', value: 'xx'}, sentenceAssets: [], subScene: [] }。


```js
test("say statement", async () => {
  const parser = new SceneParser((assetList) => {
  }, (fileName, assetType) => {
    return fileName;
  }, ADD_NEXT_ARG_LIST, SCRIPT_CONFIG);

  const result = parser.parse(`say:123 -speaker=xx;`,'test','test')
  expect(result.sentenceList).toContainEqual({
    command: commandType.say,
    commandRaw: "say",
    content: "123",
    args: [{key: 'speaker', value: 'xx'}],
    sentenceAssets: [],
    subScene: []
  })
})

```

# `/packages/server/index.js`

该代码使用了四个不同的模块：express、cloudlogjs、fs和rl。它们各自的作用如下：

1. express： Express是一个流行的Node.jsWeb框架，它提供了创建RESTfulAPI所需的基本功能。
2. cloudlogjs： 这是一个用于记录云计算事件的JavaScript库，它允许将日志记录到云服务中。
3. fs：文件系统（fs）模块提供了一个Promise-based的API，用于从本地文件系统中读取和写入文件。
4. path：路径（path）模块提供了一个处理操作系统路径的工具，它可以将字符串转换为Node.js中的路径。
5. rl：readline（读线性）是一个用于从控制台接收输入的模块，它可以通过执行readline.question程序来获取用户输入。
6. open：open（打开）是一个用于打开操作系统的工具，它可以在命令行中运行一个应用程序。

总体而言，这段代码的主要目的是创建一个Web服务器，用于捕获用户在控制台输入的日志信息，并将其记录到云服务中。


```js
const express = require('express');
const Cloudlog = require("cloudlogjs");
const fs = require('fs');
const path = require('path');
const rl = require('readline');
const open = require('open');

// 读取控制台输入
const readline = rl.createInterface({
    input: process.stdin,
    output: process.stdout
})

const server = new express();
const Port = process.env.PORT || 3000;
```

这段代码是一个 Node.js 脚本，它创建了一个 Cloudlog 对象，用于输出 WebGAL 服务器启动参数、工作目录以及检测并获取 WebGAL 游戏资源所在目录。

具体来说，这段代码的作用如下：

1. 创建一个 Cloudlog 对象，并将其命名为 "Cloudlog"。
2. 使用 `process.argv` 读取控制台数据，并输出这些数据。
3. 使用 `process.cwd` 读取工作目录，并输出该目录的路径。
4. 检测并获取 WebGAL 游戏资源所在目录。如果指定工作目录为绝对路径模式，则尝试使用 `/` 作为路径前缀。否则，尝试使用相对路径模式。如果找到了游戏资源所在目录，则输出它。
5. 输出工作目录的路径。

这段代码主要用于设置 WebGAL 服务器启动参数，并根据指定参数模式选择游戏资源所在目录。


```js
const logger = new Cloudlog();

// 读取控制台数据
const args = process.argv;
logger.info(`WebGAL Server 启动参数：`, args);

// 读取工作目录
const cwd = process.cwd();
logger.info(`WebGAL 工作目录当前为 ${cwd}`);

// 检测并获取 WebGAL 游戏资源所在目录
let webgalWd = '';

// 参数模式
if (args.length >= 3) {
    // 参数就是工作目录
    const wdr = args[2];
    logger.info(`指定工作目录：${wdr}`);
    if (wdr[0] === '/' || wdr.match(/^\w:/)) {
        // 绝对路径模式
        logger.debug('绝对路径模式');
        webgalWd = wdr;
    } else {
        // 相对路径模式
        logger.debug('相对路径模式');
        const rwd = wdr.split(/[\\\/]/g);
        webgalWd = path.join(cwd, ...rwd);
    }
    // 输出
    logger.info(`工作目录被设置为 ${webgalWd}`);
}

```

这段代码是一个自动探测模式，用于在尝试启动 WebGAL-Server 时，如果自动启动失败或者没有在当前目录下找到 WEBGAL 项目目录，则给出相应的提示信息并返回当前目录。

具体来说，代码首先检查是否有一个名为 "index.html" 的文件，如果存在，则尝试启动 WEBGAL，并将其设置为当前目录。如果 "index.html" 文件不存在，代码会找到一个名为 "WEBGAL" 的文件，并将其复制一份到当前目录中。接下来，代码遍历当前目录下的所有子目录，并在每个子目录中查找名为 "WEBGAL" 的文件。如果找到了名为 "WEBGAL" 的文件，则将其路径设置为当前目录，并将 WEBGAL-Server 启动到服务器。如果未找到名为 "WEBGAL" 的文件，代码会输出一条提示信息，并返回当前目录。


```js
// 自动探测模式
if (webgalWd === '') {
    const dirInfo = fs.readdirSync(cwd);
    if (dirInfo.includes('index.html')) {
        logger.info(`在当前工作目录下启动 WebGAL`);
        webgalWd = cwd;
    } else {
        // 全转成大写，复制一份副本
        const dirInfoUpperCase = dirInfo.map(e => e.toUpperCase());
        if (dirInfoUpperCase.includes('WEBGAL')) {
            // 找 index
            const index = dirInfoUpperCase.findIndex(e => e === 'WEBGAL');
            const trueDirName = dirInfo[index];
            webgalWd = path.join(cwd, trueDirName);
        } else {
            // 没找到
            logger.info(`未找到 WebGAL 文件，请在 WebGAL 项目目录下启动 WebGAL-Server 或在本目录下的 WebGAL 文件夹下启动。`);
        }
    }
}

```

这段代码是一个 Node.js 的服务器端代码，它取决于一个名为 "webgalWd" 的变量。如果 "webgalWd" 变量为 true，那么这段代码将启动一个名为 "webgalWd" 的服务器，监听端口，并允许浏览器访问该服务器上的资源。如果 "webgalWd" 变量为 false，那么这段代码将会在服务器启动时输出一条错误消息，然后等待并等待用户输入任意内容，直到用户主动退出。

具体来说，这段代码包含以下几个步骤：

1. 如果 "webgalWd" 变量为 true，那么执行以下操作：
   a. 创建一个 Express.static 实例，将 "webgalWd" 目录下的所有文件静态挂载到服务器上，使得服务器能够访问该目录下的所有资源。
   b. 创建一个 HTTP 服务器，并将该服务器绑定到 "webgalWd" 的端口上，使得来自浏览器的请求可以到达该服务器。
   c. 调用 Open（ ）方法，打开一个 HTTP 链接，该链接指向 "webgalWd" 的 IP 地址和端口号。

2. 如果 "webgalWd" 变量为 false，那么执行以下操作：
   a. 输出一条错误消息，并使用 readline.on（line）方法监听来自浏览器的请求，以便在接收到写得不好或者用户突然关闭浏览器的情况。
   b. 等待用户输入任意内容，然后使用 process.exit（ ）方法退出进程。


```js
if (webgalWd) {
    // 监听端口
    server.use(express.static(webgalWd))//allow browser access resources
    server.listen(Port, () => {
        logger.info(`启动 WebGAL 服务器，运行于 http://localhost:${Port} .`)
    })
    open(`http://localhost:${Port}`);
} else {
    logger.error(`未找到启动文件，请退出`);
    readline.on('line', () => {
        process.exit();
    })
}

```

# WebGAL-Server
WebGAL Server and Command LIne Interface.

WebGAL 本地调试服务器

## 使用说明
### 自动探测模式
自动探测模式在以下两种情况下探测 WebGAL 工程文件：

1、当前目录就是 WebGAL 工程的根目录。

2、当前目录包含名为 'WebGAL' 的目录（不区分大小写），这个目录是 WebGAL 工程的根目录。

### 指定目录模式
通过命令行运行此程序，第一个（且唯一一个）参数就是要指定的绝对或相对路径，是 WebGAL 的工程目录。

**注意：Windows 下的路径不能以 \ 分隔，请以 \\ 或 / 分隔，否则会发生错误。**


# `/packages/webgal/.eslintrc.js`

这段代码是一个 TypeScript 模块导出，其中包含了一个自定义的全局变量 `myGlobal`，以及一个 `complexity` 属性的规则集，用于限制代码的复杂度。

`extends` 指定了这个模块从哪些模块中继承，包括 `alloy`,`alloy/react`,`alloy/typescript`，以及 `plugin:prettier/recommended`。

`env` 指定了允许哪些全局变量被使用，包括 `browser`、`node`、`mocha`、`jest` 和 `jQuery`。

`globals` 指定了允许哪些全局变量被重新赋值，包括 `myGlobal`。

`rules` 指定了自定义的规则集，用于限制代码的复杂度和格式。其中，`semi` 属性指定了允许 SEMI 风格，而 `indent` 属性指定了允许使用 Indent 风格。

`complexity` 属性指定了允许的最大复杂度，其值为 [`error`, 30] 意味着如果一段代码的复杂度超过了这个限制，那么它的行为将受到警告。这个设置是非常灵活的，可以根据具体项目的需求进行调整。


```js
module.exports = {
  extends: ['alloy', 'alloy/react', 'alloy/typescript', 'plugin:prettier/recommended'],
  env: {
    // 你的环境变量（包含多个预定义的全局变量）
    //
    // browser: true,
    // node: true,
    // mocha: true,
    // jest: true,
    // jquery: true
  },
  globals: {
    // 你的全局变量（设置为 false 表示它不允许被重新赋值）
    //
    // myGlobal: false
  },
  rules: {
    // 自定义你的规则
    // 最大圈复杂度
    complexity: ['error', 30],
    'linebreak-style': ['error', 'unix'],
    semi: 2,
    // indent: ['error', 2],
    'semi-style': ['error', 'last'],
    'react/jsx-no-useless-fragment': [
      'error',
      {
        allowExpressions: true,
      },
    ],
  },
};

```

# `/packages/webgal/.prettierrc.js`

该代码是一个 JavaScript 的 `.prettierrc.js` 文件，它是 Prettier 插件的一个配置文件。在这里，我们定义了一些关于如何格式化 JavaScript 代码的规则。

首先，我们定义了 `printWidth`，它指定了在输出时每行代码的最大字符数，使用了 120 个字符。

接着，我们定义了 `tabWidth`，它指定了在 `printWidth` 中使用的制表符的数量，使用了 2 个字符。

我们定义了 `useTabs`，它指定了是否使用制表符来对齐输出，使用了 `false`。

我们定义了 `semi`，它指定了是否在代码中使用单引号来引用对象属性，使用了 `true`。

我们定义了 `doubleQuote`，它指定了是否在代码中使用双引号来引用字符串属性，使用了 `true`。

我们定义了 `quoteProps`，它指定了在引用字符串属性时如何解析类的属性，使用了 `'as-needed'`。

我们定义了 `jsx`，它指定了在 `doubleQuote` 和 `singleQuote` 中哪个应该被使用，使用了 `false`。

我们定义了 `jsxSingleQuote`，它指定了在 `doubleQuote` 和 `singleQuote` 中哪个应该被使用，使用了 `false`。

我们定义了 `trailingComma`，它指定了在对象属性的尾随分号（trailing）中是否使用制表符，使用了 `'all'`。

我们定义了 `bracketSpacing`，它指定了是否在括号中使用制表符来对齐括号，使用了 `true`。

我们定义了 `jsx`，它指定了在 `doubleQuote` 和 `singleQuote` 中哪个应该被使用，使用了 `false`。

我们定义了 `箭头函数`，它指定了当只有一个参数时，是否需要在函数括号中使用括号，使用了 `'always'`。

我们定义了 `rangeStart` 和 `rangeEnd`，它们指定了格式化时可用的范围，范围为文件的全部内容。

我们定义了 `requirePragma`，它指定了是否需要使用 `@prettier` 命令在文件开头插入 `require` 函数，使用了 `false`。

我们定义了 `insertPragma`，它指定了是否在文件开头插入 `@prettier` 命令，使用了 `false`。

我们定义了 `proseWrap`，它指定了是否在输出中使用折行，使用了 `'preserve'`。

我们定义了 `htmlWhitespaceSensitivity`，它指定了在插入 HTML 时，对齐策略，使用了 `'css'`。

我们定义了 `vueIndentScriptAndStyle`，它指定了是否在 Vue 组件的 `script` 和 `style` 中使用缩进，使用了 `false`。


```js
// .prettierrc.js
module.exports = {
    // 一行最多 120 字符
    printWidth: 120,
    // 使用 n 个空格缩进
    tabWidth: 2,
    // 不使用缩进符，而使用空格
    useTabs: false,
    // 行尾需要有分号
    semi: true,
    // 使用单引号
    singleQuote: true,
    // 对象的 key 仅在必要时用引号
    quoteProps: 'as-needed',
    // jsx 不使用单引号，而使用双引号
    jsxSingleQuote: false,
    // 末尾需要有逗号
    trailingComma: 'all',
    // 大括号内的首尾需要空格
    bracketSpacing: true,
    // jsx 标签的反尖括号需要换行
    bracketSameLine: false,
    // 箭头函数，只有一个参数的时候，也需要括号
    arrowParens: 'always',
    // 每个文件格式化的范围是文件的全部内容
    rangeStart: 0,
    rangeEnd: Infinity,
    // 不需要写文件开头的 @prettier
    requirePragma: false,
    // 不需要自动在文件开头插入 @prettier
    insertPragma: false,
    // 使用默认的折行标准
    proseWrap: 'preserve',
    // 根据显示样式决定 html 要不要折行
    htmlWhitespaceSensitivity: 'css',
    // vue 文件中的 script 和 style 内不用缩进
    vueIndentScriptAndStyle: false,
    // 换行符使用 lf
    endOfLine: 'lf',
    // 格式化内嵌代码
    embeddedLanguageFormatting: 'auto',
}

```

# `/packages/webgal/vite.config.ts`

This is a script written in JavaScript that watches for changes in a specific directory and triggers a build process if certain files or folders are missing or updated. It uses the Vite development server as the build tool, and the `@vitejs/plugin-visualizer` plugin to visualize the changes made to the codebase.

The script is set up to watch for changes in the `src/Core/gameScripts/pixiPerformScripts` directory, and the corresponding files are considered "触目惊弦" (ts) and "必须包含在模板中" (js), based on their file extension. When a change is detected, a new file is added to the `lastFiles` array, which is then used to trigger the build process.

The build process includes the following steps:

1. 安装ts-词法分析器和js-essential-utils库。
2. 分析.ts和.js文件，根据文件扩展名将它们归类到相应的文件类型中。
3. 根据lastFiles数组中的文件或文件夹是否发生变化来决定是否触发重建。
4. 如果发生变化，将当前文件内容写入新的重建文件中，并使用ts-词法分析器来格式化新内容。
5. 如果需要，将lastFiles数组中的文件或文件夹移动到新的位置。
6. 重建完成后，将新的文件或目录添加到.gitignore文件中，以避免在提交时包含无用内容。

该脚本使用ts-词法分析器来对TypeScript文件进行格式化，这可能会在某些情况下导致一些语法错误或类型错误。对于这种情况，生成的lint报告也会被提交到 congest 进行格式化和检查，以确保代码质量。


```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import loadVersion from 'vite-plugin-package-version';
import { resolve, relative } from 'path';
import { visualizer } from 'rollup-plugin-visualizer';
import { readdirSync, watch, writeFileSync } from 'fs';
import { isEqual } from 'lodash';
// https://vitejs.dev/config/

// @ts-ignore
const env = process.env.NODE_ENV;
console.log(env);
(() => {
  const pixiPerformScriptDirPath = './src/Core/gameScripts/pixiPerformScripts/';
  const pixiPerformManagerDirPath = './src/Core/util/pixiPerformManager/';
  const relativePath = relative(pixiPerformManagerDirPath, pixiPerformScriptDirPath).replaceAll('\\', '/');
  let lastFiles: string[] = [];

  function setInitFile() {
    console.log('正在自动编写pixi特效依赖注入');
    writeFileSync(
      resolve(pixiPerformManagerDirPath, 'initRegister.ts'),
      lastFiles
        .map((v) => {
          const filePath = relativePath + '/' + v.slice(0, v.lastIndexOf('.'));
          return `import '${filePath}';`;
        })
        .join('\n'),
      { encoding: 'utf-8' },
    );
  }

  function getPixiPerformScriptFiles() {
    const pixiPerformScriptFiles = readdirSync(pixiPerformScriptDirPath, { encoding: 'utf-8' }).filter((v) =>
      ['ts', 'js', 'tsx', 'jsx'].includes(v.slice(v.indexOf('.') + 1, v.length)),
    );
    if (!isEqual(pixiPerformScriptFiles, lastFiles)) {
      lastFiles = pixiPerformScriptFiles;
      setInitFile();
    }
  }

  getPixiPerformScriptFiles();

  if (env !== 'production') watch(pixiPerformScriptDirPath, { encoding: 'utf-8' }, getPixiPerformScriptFiles);
})();

```

这段代码是一个 TypeScript 导出的 CSS 配置文件 defineConfig 的定义。

export default defineConfig({
 plugins: [
   react(),
   loadVersion(),
   visualizer(),
 ],
 resolve: {
   alias: {
     '@': resolve('src'),
   },
 },
});

这段代码的作用是定义一个 CSS 配置文件。它通过使用 plugins 数组中的 React、Visualizer 和LoadVersion 等库定义了项目中使用的 CSS 构建工具。同时，通过使用 resolve 属性，将模块别名 '@' 映射到项目根目录的 'src' 目录。这样可以确保在开发过程中，模块别名 '@' 对应的文件在不同的项目中都是访问到的。


```js
export default defineConfig({
  plugins: [
    react(),
    loadVersion(),
    // @ts-ignore
    visualizer(),
  ],
  resolve: {
    alias: {
      '@': resolve('src'),
    },
  },
});

```

# `/packages/webgal/public/webgal-serviceworker.js`

This is a JavaScript function that uses the `match` method to retrieve a cached response from a specified cache, or to fetch a resource from the network and cache it for future use.

It takes an `event.request` object, which is passed to the `then` method. If the cache contains a cached response for the given `event.request`, the function returns the cached response. Otherwise, it makes a network request to fetch the resource and cache it for future use.

The function uses the `fetch` method to make the network request. It also uses the `json` method of the `fetch` response to extract the headers of the response, which are passed to the `set` method of the cache object.

If the cache does not contain a cached response for the given `event.request`, the function will fetch the resource from the network and cache it for future use. The cached resource will have a unique `id` that is added to the cache's `key` property, which is a string of the URL of the resource.

The function also uses the `eslint-disable-next-line` directive to disable the maximum nesting callbacks.


```js
self.addEventListener('install', (ev) => {
  // console.log('[service worker] installing');
  ev.waitUntil(self.skipWaiting());
});

// fetch事件是每次页面请求资源时触发的
self.addEventListener('fetch', function (event) {
  const url = event.request.url;
  const isReturnCache = !!(url.match('/assets/') && !url.match('game'));
  if (isReturnCache) {
    // console.log('%cCACHED: ' + url, 'color: #005CAF; padding: 2px;');
  }
  if (!isReturnCache) {
    event.respondWith(fetch(event.request));
  } else {
    event.respondWith(
      // 检查在缓存中是否有匹配的资源
      caches.match(event.request).then(function (response) {
        // 如果缓存中有匹配的资源，则返回缓存资源
        if (response) {
          return response;
        }
        // 如果没有匹配的资源，则尝试从网络请求
        // 同时将获取的资源存入缓存
        return fetch(event.request)
          .then(function (networkResponse) {
            console.log('%cCACHED: ' + url, 'color: #005CAF; padding: 2px;');
            if (networkResponse.status === 206 && event.request.headers.has('range')) {
              // 如果是部分响应且请求带有Range头，则创建新的请求，将完整响应返回给客户端
              // eslint-disable-next-line max-nested-callbacks
              return fetch(event.request.url).then(function (fullNetworkResponse) {
                const headers = {};
                for (let entry of fullNetworkResponse.headers.entries()) {
                  headers[entry[0]] = entry[1];
                }
                const fullResponse = new Response(fullNetworkResponse.body, {
                  status: fullNetworkResponse.status,
                  statusText: fullNetworkResponse.statusText,
                  headers: headers,
                });
                const clonedResponse = fullResponse.clone();
                // eslint-disable-next-line max-nested-callbacks
                caches.open('my-cache').then(function (cache) {
                  cache.put(event.request, clonedResponse);
                });
                return fullResponse;
              });
            }
            const clonedResponse = networkResponse.clone();
            // eslint-disable-next-line max-nested-callbacks
            caches.open('my-cache').then(function (cache) {
              cache.put(event.request, clonedResponse);
            });
            return networkResponse;
          })
          .catch(function (error) {
            console.error('Fetching failed:', error);
            throw error;
          });
      }),
    );
  }
});

```

# `/packages/webgal/src/vite-env.d.ts`

这段代码是一个 TypeScript 依賴檔案，它引用了名为 "vite/client" 的远荆类型。通常情况下，TypeScript 依賴檔案是用于定义外部模块中引用的不确定型值的。不过这里，模块 "vite/client" 已经被明确定义了，所以这个依賴檔案的作用就是定义了 "vite/client" 的类型。


```js
/// <reference types="vite/client" />

```

# `/packages/webgal/src/Components/Stage/MainStage/useSetBg.ts`

This is a React component that uses the `useSetBg` function from the `@/Core/controller/stage/pixi/PixiController` interface to dynamically update the background stage in a Pixi game. 

The `useSetBg` function takes an `IStageState` object and a `bgName` as inputs. It uses the `useEffect` hook to fetch and apply a background animation based on the specified `bgName`. 

The background animation is created using the `WebGAL` module from the `@/Core/controller/stage/pixi/PixiController` module. It is then registered with the `registerPresetAnimation` method and added to the animation effects of the `bg-main` key in the `IStageObject` object.

The `useEffect` hook also uses the `setEbg` function from the `@/Core/util/etc/logger` module to update the `bgName` in the `IStageState` object.


```js
import { IStageState } from '@/store/stageInterface';
import { useEffect } from 'react';
import { logger } from '@/Core/util/etc/logger';
import { IStageObject } from '@/Core/controller/stage/pixi/PixiController';
import { setEbg } from '@/Core/util/setEbg';

import { getEnterExitAnimation } from '@/Core/Modules/animationFunctions';
import { WebGAL } from '@/Core/WebGAL';

export function useSetBg(stageState: IStageState) {
  const bgName = stageState.bgName;

  /**
   * 设置背景
   */
  useEffect(() => {
    const thisBgKey = 'bg-main';
    if (bgName !== '') {
      const currentBg = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisBgKey);
      if (currentBg) {
        if (currentBg.sourceUrl !== bgName) {
          removeBg(currentBg);
        }
      }
      WebGAL.gameplay.pixiStage?.addBg(thisBgKey, bgName);
      setEbg(bgName);
      logger.debug('重设背景');
      const { duration, animation } = getEnterExitAnimation('bg-main', 'enter', true);
      WebGAL.gameplay.pixiStage!.registerPresetAnimation(animation, 'bg-main-softin', thisBgKey, stageState.effects);
      setTimeout(() => WebGAL.gameplay.pixiStage!.removeAnimationWithSetEffects('bg-main-softin'), duration);
    } else {
      const currentBg = WebGAL.gameplay.pixiStage?.getStageObjByKey(thisBgKey);
      if (currentBg) {
        removeBg(currentBg);
      }
    }
  }, [bgName]);
}

```

该函数名为 `removeBg`，其作用是移除场景中的一个透明背景(`IStageObject` 类型的对象，在游戏内称为 `bgObject`)。

首先，它使用 `WebGAL.gameplay.pixiStage` 中的 `removeAnimationWithSetEffects` 方法，移除场景中名为 `bg-main-softin` 的动画，并且设置一个 `@WebGLRenderPath` 效果，使背景渐变 softoff。

然后，它将场景对象 `bgObject` 的 `key` 属性更改为 `'bg-main-off'`，以表示背景已移除。

接着，它使用 `WebGAL.gameplay.pixiStage` 中的 `removeStageObjectByKey` 方法，移除场景中名为 `bg-main-off` 的舞台对象。

接着，它使用 `WebGAL.gameplay.pixiStage` 中的 `registerAnimation` 方法，注册一个名为 `bg-main-softoff` 的动画，该动画为场景中名为 `bg-main-off` 的舞台对象添加了一个透明背景。该动画的时间为 `duration`，即所设置的时间间隔。

最后，它使用 `setTimeout` 函数，在 `duration` 毫秒之后，移除场景中名为 `bg-main-softoff` 的动画，并移除场景中名为 `bg-main-off` 的舞台对象。


```js
function removeBg(bgObject: IStageObject) {
  WebGAL.gameplay.pixiStage?.removeAnimationWithSetEffects('bg-main-softin');
  const oldBgKey = bgObject.key;
  bgObject.key = 'bg-main-off';
  WebGAL.gameplay.pixiStage?.removeStageObjectByKey(oldBgKey);
  const { duration, animation } = getEnterExitAnimation('bg-main-off', 'exit', true);
  WebGAL.gameplay.pixiStage!.registerAnimation(animation, 'bg-main-softoff', 'bg-main-off');
  setTimeout(() => {
    WebGAL.gameplay.pixiStage?.removeAnimation('bg-main-softoff');
    WebGAL.gameplay.pixiStage?.removeStageObjectByKey('bg-main-off');
  }, duration);
}

```