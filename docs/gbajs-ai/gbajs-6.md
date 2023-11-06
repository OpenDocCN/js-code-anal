# GBAJS源码解析 6

# `js/video/worker.js`

This appears to be a JavaScript function that modifies a video background image in real-time, using the WebGL API. It appears to be iterating through the elements of an object called `dirty` and writing over the current pixel value if it is a number, using techniques like `video.writeBackgroundParamC()` and `video.writeBackgroundParamD()` The function is also using the `WIN0H`, `WIN1H`, `WIN0V`, and `WIN1V` Windows control codes to write to the background image, as well as the `BLDCNT`, `BLDALPHA`, `BLDY`, `MOSAIC`, and `memory` parameters. It also appears to be using the `receiveMemory()` function to read the image data from a `dirty` object.


```js
importScripts('software.js');

var video = new GameBoyAdvanceSoftwareRenderer();
var proxyBacking = null;
var currentFrame = 0;

self.finishDraw = function(pixelData) {
	self.postMessage({ type: 'finish', backing: pixelData, frame: currentFrame });
}

function receiveDirty(dirty) {
	for (var type in dirty) {
		switch (type) {
		case 'DISPCNT':
			video.writeDisplayControl(dirty[type]);
			break;
		case 'BGCNT':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundControl(i, dirty[type][i]);
				}
			}
			break;
		case 'BGHOFS':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundHOffset(i, dirty[type][i]);
				}
			}
			break;
		case 'BGVOFS':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundVOffset(i, dirty[type][i]);
				}
			}
			break;
		case 'BGX':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundRefX(i, dirty[type][i]);
				}
			}
			break;
		case 'BGY':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundRefY(i, dirty[type][i]);
				}
			}
			break;
		case 'BGPA':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundParamA(i, dirty[type][i]);
				}
			}
			break;
		case 'BGPB':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundParamB(i, dirty[type][i]);
				}
			}
			break;
		case 'BGPC':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundParamC(i, dirty[type][i]);
				}
			}
			break;
		case 'BGPD':
			for (var i in dirty[type]) {
				if (typeof(dirty[type][i]) === 'number') {
					video.writeBackgroundParamD(i, dirty[type][i]);
				}
			}
			break;
		case 'WIN0H':
			video.writeWin0H(dirty[type]);
			break;
		case 'WIN1H':
			video.writeWin1H(dirty[type]);
			break;
		case 'WIN0V':
			video.writeWin0V(dirty[type]);
			break;
		case 'WIN1V':
			video.writeWin1V(dirty[type]);
			break;
		case 'WININ':
			video.writeWinIn(dirty[type]);
			break;
		case 'WINOUT':
			video.writeWinOut(dirty[type]);
			break;
		case 'BLDCNT':
			video.writeBlendControl(dirty[type]);
			break;
		case 'BLDALPHA':
			video.writeBlendAlpha(dirty[type]);
			break;
		case 'BLDY':
			video.writeBlendY(dirty[type]);
			break;
		case 'MOSAIC':
			video.writeMosaic(dirty[type]);
			break;
		case 'memory':
			receiveMemory(dirty.memory);
			break;
		}
	}
}

```



这段代码定义了一个名为 `receiveMemory` 的函数，它接受一个名为 `memory` 的参数。

函数内部判断 `memory` 是否包含 `palette`、`oam` 和 `vram` 三个缓冲区。如果是，则分别将 `memory` 中的缓冲区内容复制到 `video.palette`、`video.oam` 和 `video.vram` 缓冲区中。

如果是，则从 `i` 循环开始，遍历 `memory.vram` 数组，如果当前元素不为 `null`，则将其作为起始点，将 `i` 左侧 12 位二进制数对应的 ASCII 字符值插入到 `video.vram` 缓冲区中。

注意，`receiveMemory` 函数中的参数 `memory` 是一个引用，即 `receiveMemory` 函数可以修改 `memory` 本身的内容。


```js
function receiveMemory(memory) {
	if (memory.palette) {
		video.palette.overwrite(new Uint16Array(memory.palette));
	}
	if (memory.oam) {
		video.oam.overwrite(new Uint16Array(memory.oam));
	}
	if (memory.vram) {
		for (var i = 0; i < 12; ++i) {
			if (memory.vram[i]) {
				video.vram.insert(i << 12, new Uint16Array(memory.vram[i]));
			}
		}
	}
}

```

这段代码定义了一个名为 `handlers` 的对象，它包含了一些处理 `video` 对象中 `data` 属性的函数。

`clear` 函数会在接收到 `data` 属性的基础上清除 `video` 对象中的所有内容。

`scanline` 函数会在接收到 `data` 属性的基础上，尝试使用 `proxyBacking` 属性（它是 `video` 对象的一个属性，代表视频的代理服务器）获取代理服务器返回的 `proxy` 对象，然后使用该 `proxy` 对象来绘制扫描线。扫描线会在 `video` 对象中绘制，以指定的 `proxyBacking` 属性为透明度，以指定的 `y` 属性的值作为起始位置，以指定的 `proxy` 对象中的 `drawScanline` 函数为绘图函数。

`start` 函数会在接收到 `data` 属性的基础上设置一个名为 `proxyBacking` 的代理服务器，并将它设置为 `video` 对象的一个属性。然后将 `video` 对象的一个属性 `backing` 设置为 `data.backing` 的值，以设置 `proxyBacking` 属性的值。

`finish` 函数会在接收到 `data` 属性的基础上，设置 `currentFrame` 为 `data.frame` 的值，然后遍历 `data.scanlines` 数组。对于每个 `scanline`，先使用 `proxyBacking` 属性获取代理服务器返回的 `proxy` 对象，然后使用该 `proxy` 对象来绘制扫描线。扫描线会在 `video` 对象中绘制，以指定的 `proxyBacking` 属性为透明度，以指定的 `y` 属性的值作为起始位置，以指定的 `drawScanline` 函数为绘图函数。在绘制扫描线的过程中，如果 `data.scanlines[i].dirty` 属性为 `true`，则使用 `receiveDirty` 函数通知 `proxy` 对象重新获取最新的 `data` 对象，并重新绘制扫描线。

最后，在 `proxyBacking` 属性被设置为 `null`、`video.setBacking` 函数被调用之后，以及 `finishDraw` 函数调用 `video` 对象的所有绘图函数结束时，会将 `currentFrame` 保存，以便在需要时使用。


```js
var handlers = {
	clear: function(data) {
		video.clear(data);
	},

	scanline: function(data) {
		receiveDirty(data.dirty);
		video.drawScanline(data.y, proxyBacking);
	},

	start: function(data) {
		proxyBacking = data.backing;
		video.setBacking(data.backing);
	},

	finish: function(data) {
		currentFrame = data.frame;
		var scanline = 0;
		for (var i = 0; i < data.scanlines.length; ++i) {
			for (var y = scanline; y < data.scanlines[i].y; ++y) {
				video.drawScanline(y, proxyBacking);
			}
			scanline = data.scanlines[i].y + 1;
			receiveDirty(data.scanlines[i].dirty);
			video.drawScanline(data.scanlines[i].y, proxyBacking);
		}
		for (var y = scanline; y < 160; ++y) {
			video.drawScanline(y, proxyBacking);
		}
		video.finishDraw(self);
	},
};

```

这段代码定义了一个 `self.onmessage` 函数，用于处理游戏中接收到的消息。

该函数接收一个 `message` 参数，包含了消息的数据和发送者的角色等信息。函数内部创建了一个 `handlers` 对象，它具有键 `message.data['type']` 的值，用于存储相应类型消息的处理函数。

函数的作用是，当接收到一个消息时，根据消息类型调用对应的处理函数，并将消息数据作为参数传递给处理函数。例如，如果接收到一个 `kick` 消息，会将消息数据 `{type: 'kick'}` 传递给处理函数 `function(message) { return `Kicked by ${message.data['Kender']}`; }`。

简而言之，该函数是一个消息处理器，根据消息类型执行相应的处理逻辑，并将消息数据作为参数传递给处理函数。


```js
self.onmessage = function(message) {
	handlers[message.data['type']](message.data);
};

```

# `resources/console.js`

该代码定义了一个名为“Console”的JavaScript类，用于在浏览器中捕获和处理GPU硬件断点。

构造函数接收一个包含GPU颜色缓冲器的对象（即gba参数），并创建与GPU硬件交互的内存区域、一个调试输出元素以及一个存储所有测试断点的数组。

该类还定义了一些与GPU硬件交互的函数，如向调试输出添加新断点、设置默认断点、检查和清除所有测试断点等。

该类还创建了一个名为“PaletteViewer”的元素，用于显示由GPU硬件捕获的可用颜色，以及一个名为“TileViewer”的元素，用于显示GPU内存中的纹理和颜色映射。

该类使用“setLogger”函数来记录调试输出，使用“doStep”函数来轮询GPU硬件的当前状态，并使用“testBreakpoints”函数来查找和设置GPU硬件的断点。


```js
function Console(gba) {
	this.cpu = gba.cpu;
	this.gba = gba;
	this.ul = document.getElementById('console');
	this.gprs = document.getElementById('gprs');
	this.memory = new Memory(gba.mmu);
	this.breakpoints = [];
	this.logQueue = [];

	this.activeView = null;
	this.paletteView = new PaletteViewer(gba.video.renderPath.palette);
	this.tileView = new TileViewer(gba.video.renderPath.vram, gba.video.renderPath.palette);
	this.update();

	var self = this;
	gba.setLogger(function (level, message) { self.log(level, message) });
	this.gba.doStep = function () { return self.testBreakpoints() };
}

```

It appears that the `updateGPRs` and `updateCPSR` methods are both intended to update the text content of certain elements in the page's HTML structure.

The `updateGPRs` method updates the text content of the elements with the `gprs` children of the current CPU instance. The method takes an empty string as its argument, which is likely intended to be the new text content for each element. The method loops through each element in the `gprs` array and sets the `textContent` property of the corresponding element to the `hex` of the corresponding value. The `hex` function converts a hexadecimal value to a hex string.

The `updateCPSR` method updates the text content of the elements with the `mode` and `execMode` children of the current CPU instance. The method takes an empty string as its argument, which is likely intended to be the new text content for each element. The method uses a helper function `bit` to update the text content of each element. The `bit` function takes two arguments: the `psr` property of the element and a `member` string. The function sets the text content of the element to the new text if the `psr` property is not defined, or sets the text content of the element to a CSS class if the `member` is `execMode` and the `psr` property is defined.

It appears that the `updateGPRs` method specifically updates the text content of the elements with the `gprs` children of the current CPU instance, while the `updateCPSR` method updates the text content of the elements with the `mode` and `execMode` children of the current CPU instance.


```js
Console.prototype.updateGPRs = function() {
	for (var i = 0; i < 16; ++i) {
		this.gprs.children[i].textContent = hex(this.cpu.gprs[i]);
	}
}

Console.prototype.updateCPSR = function() {
	var cpu = this.cpu;
	var bit = function(psr, member) {
		var element = document.getElementById(psr);
		if (cpu[member]) {
			element.removeAttribute('class'); 
		} else {
			element.setAttribute('class', 'disabled');
		}
	}
	bit('cpsrN', 'cpsrN');
	bit('cpsrZ', 'cpsrZ');
	bit('cpsrC', 'cpsrC');
	bit('cpsrV', 'cpsrV');
	bit('cpsrI', 'cpsrI');
	bit('cpsrT', 'execMode');
	
	var mode = document.getElementById('mode');
	switch (cpu.mode) {
	case cpu.MODE_USER:
		mode.textContent = 'USER';
		break;
	case cpu.MODE_IRQ:
		mode.textContent = 'IRQ';
		break;
	case cpu.MODE_FIQ:
		mode.textContent = 'FIQ';
		break;
	case cpu.MODE_SUPERVISOR:
		mode.textContent = 'SVC';
		break;
	case cpu.MODE_ABORT:
		mode.textContent = 'ABORT';
		break;
	case cpu.MODE_UNDEFINED:
		mode.textContent = 'UNDEFINED';
		break;
	case cpu.MODE_SYSTEM:
		mode.textContent = 'SYSTEM';
		break;
	default:
		mode.textContent = '???';
		break;
	}
}

```

该代码是一个 JavaScript 类方法，它更改了 `Console.prototype.log` 函数的行为。具体来说，它根据传入的 `level` 值，修改了消息的前缀和后缀，然后将消息添加到 `logQueue` 数组中。如果 `level` 等同于 `this.gba.LOG_ERROR`，则会暂停脚本执行并输出 `[ERROR]` 消息。如果 `level` 等同于 `this.gba.LOG_WARN`，则会输出 `[WARN]` 消息。如果 `level` 等同于 `this.gba.LOG_STUB`，则会输出 `[STUB]` 消息。如果 `level` 等同于 `this.gba.LOG_INFO`，则会输出 `[INFO]` 消息。如果 `level` 等同于 `this.gba.LOG_DEBUG`，则会输出 `[DEBUG]` 消息。


```js
Console.prototype.log = function(level, message) {
	switch (level) {
	case this.gba.LOG_ERROR:
		message = '[ERROR] ' + message;
		break;
	case this.gba.LOG_WARN:
		message = '[WARN] ' + message;
		break;
	case this.gba.LOG_STUB:
		message = '[STUB] ' + message;
		break;
	case this.gba.LOG_INFO:
		message = '[INFO] ' + message;
		break;
	case this.gba.LOG_DEBUG:
		message = '[DEBUG] ' + message;
		break;
	}
	this.logQueue.push(message);
	if (level == this.gba.LOG_ERROR) {
		this.pause();
	}
	if (!this.stillRunning) {
		this.flushLog();
	}
}

```

这段代码定义了一个名为 `flushLog` 的函数，它是 `Console.prototype` 对象的一个方法。

该函数的作用是在控制台输出器(console)中输出一系列日志条目。函数首先检查是否需要滚动输出器，然后从队列中删除日志条目并将其添加到输出器中。最后，如果需要滚动输出器，函数会定期滚动输出器并添加新的日志条目。

具体来说，函数的实现可以分为以下几个步骤：

1. 定义一个 `doScroll` 变量，该变量在每次循环中使用 `ul.scrollTop` 减去 `ul.offsetHeight` 的值，表示是否需要滚动输出器。

2. 在每次循环中，定义一个 `entry` 变量，将 `this.logQueue.shift()` 中的日志条目添加到输出器中，并将 `ul.appendChild(entry)` 中的 `entry` 节点添加到输出器中。

3. 如果需要滚动输出器，函数会执行以下操作：

  a. 定义一个 `ul` 变量，用于存储输出器对象。

  b. 定义一个 `last` 变量，用于存储上一次滚动位置。

  c. 定义一个 `scrollUp` 函数，该函数定期滚动输出器并添加新的日志条目。该函数使用 `ul.scrollTop` 减去 `last` 并乘以一个比例系数(0.2 + 0.8 = 1)，来控制每次滚动的高度。函数使用 `setTimeout(scrollUp, 25)` 来延迟滚动操作，时间间隔为 25 毫秒。

  d. 在每次循环中，延迟 25 毫秒，然后调用 `scrollUp` 函数。

  e. 如果当前滚动位置等于输出器的高度减去输出器中的日志条目数，函数将更新滚动位置并重新计时。

  f. 在函数执行完毕后，将 `last` 变量中的值存储回 `ul.scrollHeight - ul.offsetHeight` 中，以便在需要时使用。


```js
Console.prototype.flushLog = function() {
	var doScroll = this.ul.scrollTop == this.ul.scrollHeight - this.ul.offsetHeight;
	while (this.logQueue.length) {
		var entry = document.createElement('li');
		entry.textContent = this.logQueue.shift();
		this.ul.appendChild(entry);
	}
	if (doScroll) {
		var ul = this.ul;
		var last = ul.scrollTop;
		var scrollUp = function() {
			if (ul.scrollTop == last) {
				ul.scrollTop = (ul.scrollHeight - ul.offsetHeight) * 0.2 + last * 0.8;
				last = ul.scrollTop;
				if (last != ul.scrollHeight - ul.offsetHeight) {
					setTimeout(scrollUp, 25);
				}
			}
		}
		setTimeout(scrollUp, 25);
	}

}

```

这两段代码定义了 `Console.prototype.update` 和 `Console.prototype.setView` 函数。

1. `Console.prototype.update` 函数的作用是在每次渲染视图之前，更新控制台输出内容的当前时间戳和 CPU 计数器的值，并刷新所有与内存相关的指针，然后如果当前激活的视图存在，则将其重新绘制。

2. `Console.prototype.setView` 函数的作用是在激活的视图存在时，将其插入到 `debugViewer` 容器中，并将其重新绘制以显示激活的视图。它还清除 `debugViewer` 容器中的所有子节点，以便在将新的视图添加到容器中时不会出现意外。


```js
Console.prototype.update = function() {
	this.updateGPRs();
	this.updateCPSR();
	this.memory.refreshAll();
	if (this.activeView) {
		this.activeView.redraw();
	}
}

Console.prototype.setView = function(view) {
	var container = document.getElementById('debugViewer');
	while (container.hasChildNodes()) {
		container.removeChild(container.lastChild);
	}
	if (view) {
		view.insertChildren(container);
		view.redraw();
	}
	this.activeView = view;
}

```

这段代码定义了两个方法，一个是 `step()`，一个是 `runVisible()`。这两个方法都是属于 `Console.prototype` 对象。

`step()` 方法的目的是在每次循环中调用 `this.cpu.step()` 方法，并调用 `this.update()` 方法。如果 `step()` 方法内部出现异常，将捕获到异常并调用 `this.log()` 方法来记录调试信息，然后抛出异常。

`runVisible()` 方法的目的是在每次循环中调用 `run()` 函数。在 `run()` 函数中，首先检查 `this.stillRunning` 属性是否为 `true`，如果是，则不执行任何操作，直接返回。如果 `this.stillRunning` 属性不是 `true`，则说明有代码需要运行，将 `this.step()` 方法内部的异常捕获并记录调试信息，然后设置 `this.stillRunning` 属性为 `true`，并将 `run()` 函数设置为 `this.step()` 方法内部的回调函数。最后，调用 `setTimeout()` 函数来延迟 `run()` 函数的执行，其参数为 `0`。


```js
Console.prototype.step = function() {
	try {
		this.cpu.step();
		this.update();
	} catch (exception) {
		this.log(this.gba.LOG_DEBUG, exception);
		throw exception;
	}
}

Console.prototype.runVisible = function() {
	if (this.stillRunning) {
		return;
	}

	this.stillRunning = true;
	var self = this;
	run = function() {
		if (self.stillRunning) {
			try {
				self.step();
				if (self.breakpoints.length && self.breakpoints[self.cpu.gprs[self.cpu.PC]]) {
					self.breakpointHit();
					return;
				}
				self.flushLog();
				setTimeout(run, 0);
			} catch (exception) {
				self.log(this.gba.LOG_DEBUG, exception);
				self.pause();
				throw exception;
			}
		}
	}
	setTimeout(run, 0);
}

```

这段代码是用来在Console.prototype上添加一个名为“run”的方法，其功能是暂时停止Console的运行，并在需要时重新开始。

具体来说，代码首先检查对象实例是否仍在运行中，如果是，则直接返回。如果不是，则将对象实例的“stillRunning”属性设置为真，并设置regs和mem元素的样式为“disabled”(禁止访问)。

然后代码将设置一个计时器，在计时器到期时，将“run”方法调用本身，其中this指代当前对象实例，并传递一个匿名函数作为参数。

在这个匿名函数中，代码使用Date.now()获取当前的JavaScript运行时间，并将其作为参数传递给“gba.runStable()”方法。这个方法可能是从gba库中获取图形绘制信息的一个方法。

最后，代码将“disabled”类设置为regs和mem元素的样式，以阻止页面中显示这些元素，并继续使用“disabled”样式防止用户在页面上手动重新启动Console。


```js
Console.prototype.run = function() {
	if (this.stillRunning) {
		return;
	}

	this.stillRunning = true;
	var regs = document.getElementById('registers');
	var mem = document.getElementById('memory');
	var start = Date.now();
	regs.setAttribute('class', 'disabled');
	mem.setAttribute('class', 'disabled');
	var self = this;
	this.gba.runStable();
}

```

这段代码定义了一个新的 `Console.prototype.runFrame` 函数，它会每帧循环性地运行一次。

以下是这个函数的执行流程：

1. 如果当前对象 `this` 还在运行中，那么函数直接返回，因为 `this.stillRunning` 属性已经在代码中设置为 `true` 了。

2. 如果 `this.stillRunning` 属性为 `false` 的话，那么创建一个新的 `run` 函数，并将 `this` 对象作为参数传递给 `run` 函数。

3. `run` 函数内部，首先将 `regs` 元素的 `class` 属性设置为 `disabled`，将 `mem` 元素的 `class` 属性设置为 `disabled`。这样可以禁止对 `regs` 和 `mem` 元素进行操作，防止函数在运行过程中修改它们的样式。

4. 然后获取当前时间的微小区值，即 `Date.now()` 函数的值，并将其作为 `start` 变量传递给 `run` 函数。

5. 最后， `run` 函数内部执行以下操作：

  a. 调用 `this.gba.step()` 函数，更新游戏主轴的移动状态。

  b. 调用 `this.pause()` 函数，暂停游戏主轴的移动。

  c. 将 `self` 对象从 `run` 函数中分离出来，防止 `run` 函数继续执行 `this.gba.step()` 和 `this.pause()` 函数。

6. 最后，将 `run` 函数的时间设置为 `setTimeout(run, 0)` 函数的参数，即当 `run` 函数被执行时，它会在 `0` 毫秒后执行一次。


```js
Console.prototype.runFrame = function() {
	if (this.stillRunning) {
		return;
	}

	this.stillRunning = true;
	var regs = document.getElementById('registers');
	var mem = document.getElementById('memory');
	var start = Date.now();
	regs.setAttribute('class', 'disabled');
	mem.setAttribute('class', 'disabled');
	var self = this;
	run = function() {
		self.gba.step();
		self.pause();
	}
	setTimeout(run, 0);
}

```

这段代码定义了两个方法： pause() 和 breakpointHit()。

pause() 方法的作用是暂停当前的 Console 对象（也就是屏幕上显示的程序）的执行，并暂停 GBA（图形业务应用程序编程接口）对象的暂停。同时，将 stillRunning 属性设置为 false，即表示当前正处于暂停状态。接着，从 regs 和 memory 两个元素中移除它们的 class 属性，然后调用 update() 方法更新屏幕上显示的内容。最后，调用 flushLog() 方法将日志信息存储到内存中，以便在必要时进行输出。

breakpointHit() 方法的作用是在当前的 breakpoint 被触发时暂停执行，并输出一条调试信息。首先，调用 pause() 方法暂停当前的执行。接着，调用 stillRunning 的属性，以确保在暂停期间，如果 stillRunning 的值为 false，那么就表示当前正处于暂停状态。然后，调用 gba.pause() 方法暂停 GBA 对象的暂停。接着，获取 regs 和 memory 两个元素，并从它们中移除 class 属性。然后，调用 update() 方法更新屏幕上显示的内容，并调用 flushLog() 方法将日志信息存储到内存中。最后，输出一条调试信息，信息的内容是 "Hit breakpoint at " + hex(this.cpu.gprs[this.cpu.PC])，其中 "Hit breakpoint at " 将获取到的 breakpoint ID 转换为 hex 格式，然后将其输出。


```js
Console.prototype.pause = function() {
	this.stillRunning = false;
	this.gba.pause();
	var regs = document.getElementById('registers');
	var mem = document.getElementById('memory');
	mem.removeAttribute('class');
	regs.removeAttribute('class');
	this.update();
	this.flushLog();
}

Console.prototype.breakpointHit = function() {
	this.pause();
	this.log(this.gba.LOG_DEBUG, 'Hit breakpoint at ' + hex(this.cpu.gprs[this.cpu.PC]));
}

```

该代码定义了一个名为 addBreakpoint 的方法，它接受一个地址参数 addr。该方法将添加到程序中的断点（breakpoint）设置为 true，并将其 id 设置为 BP<addr>。

为了在断点列表中显示添加的断点，代码检查 bpLi 是否存在。如果不存在，则创建一个新的列表项并将其添加到 bpLi 上。然后，代码创建一个新的带有地址和 checked 属性的 span 元素，并将其添加到 bpLi 的 list 项中。

最后，代码将添加的断点添加到 id 为 breakpointView 的元素中，并将其添加到程序中的控制台输出窗口中。


```js
Console.prototype.addBreakpoint = function(addr) {
	this.breakpoints[addr] = true;
	var bpLi = document.getElementById('bp' + addr);
	if (!bpLi) {
		bpLi = document.createElement('li');
		bpLi.address = addr;
		var cb = document.createElement('input');
		cb.setAttribute('type', 'checkbox');
		cb.setAttribute('checked', 'checked');
		var self = this;
		cb.addEventListener('click', function() {
			self.breakpoints[addr] = cb.checked;
		}, false);
		bpLi.appendChild(cb);
		bpLi.appendChild(document.createTextNode(hex(addr)));
		document.getElementById('breakpointView').appendChild(bpLi);
	}
}

```

这段代码定义了一个名为 "Console.prototype.testBreakpoints" 的函数，以及一个名为 "Memory" 的类。

"Console.prototype.testBreakpoints" 函数用于在控制台输出 "breakpointHit()" 函数的实现。具体而言，该函数会检测该对象是否包含在对象本身定义的 breakpoints 数组中。如果是，函数将调用 "breakpointHit()" 函数，并返回 false；否则，函数返回 "breakpointHit()" 函数的调用，但不会输出任何信息。

"Memory" 类用于管理内存。该类包含一个 "ul" 元素，用于在控制台输出行数和当前页面的内存使用情况。该类还包含一个 "scrollTop" 变量，用于记录要滚动到的位置。该类还定义了一个 "resize" 事件处理程序，用于在窗口大小发生变化时调整该类的布局。


```js
Console.prototype.testBreakpoints = function() {
	if (this.breakpoints.length && this.breakpoints[this.cpu.gprs[this.cpu.PC]]) {
		this.breakpointHit();
		return false;
	}
	return this.gba.waitFrame();
};

Memory = function(mmu) {
	this.mmu = mmu;
	this.ul = document.getElementById('memoryView');
	row = this.createRow(0);
	this.ul.appendChild(row);
	this.rowHeight = row.offsetHeight;
	this.numberRows = this.ul.parentNode.offsetHeight / this.rowHeight + 2;
	this.ul.removeChild(row);
	this.scrollTop = 50 - this.ul.parentElement.firstElementChild.offsetHeight;

	for (var i = 0; i < this.numberRows; ++i) {
		this.ul.appendChild(this.createRow(i << 4));
	}
	this.ul.parentElement.scrollTop = this.scrollTop;

	var self = this;
	this.ul.parentElement.addEventListener('scroll', function(e) { self.scroll(e) }, true);
	window.addEventListener('resize', function(e) { self.resize() }, true);
}

```

这段代码是用来影响一个元素的滚动位置，并防止元素溢出其父元素容器。具体来说，它执行以下操作：

1. 如果元素的父元素滚动位置低于元素滚动位置，则将元素的内容移动到父元素中，并使元素滚动到父元素中。
2. 如果元素的父元素的滚动位置高于元素滚动位置，则将元素的内容移动到父元素中，并使元素滚动到父元素中。
3. 如果元素的父元素滚动位置等于元素滚动位置，则防止元素溢出父元素容器。

该函数使用了一个 while 循环和一个 e 对象来阻止默认行为。该函数通过不断移动元素并更改其父元素的滚动位置来实现滚动。


```js
Memory.prototype.scroll = function(e) {
	while (this.ul.parentElement.scrollTop - this.scrollTop < this.rowHeight) {
		if (this.ul.firstChild.offset == 0) {
			break;
		}
		var victim = this.ul.lastChild;
		this.ul.removeChild(victim);
		victim.offset = this.ul.firstChild.offset - 16;
		this.refresh(victim);
		this.ul.insertBefore(victim, this.ul.firstChild);
		this.ul.parentElement.scrollTop += this.rowHeight;
	}
	while (this.ul.parentElement.scrollTop - this.scrollTop > this.rowHeight * 2) {
		var victim = this.ul.firstChild;
		this.ul.removeChild(victim);
		victim.offset = this.ul.lastChild.offset + 16;
		this.refresh(victim);
		this.ul.appendChild(victim);
		this.ul.parentElement.scrollTop -= this.rowHeight;
	}
	if (this.ul.parentElement.scrollTop < this.scrollTop) {
		this.ul.parentElement.scrollTop = this.scrollTop;
		e.preventDefault();
	}
}

```

这段代码是用来在Memory对象中重排其子元素的大小。其大致作用如下：

1. 首先，它计算出当前对象的行数（即ul元素的父元素的offsetHeight除以rowHeight），并加上2，得到一个新的行数。
2. 接着，它检查新的行数是否大于ul元素的子元素数量。如果是，那么它将遍历ul元素的子元素，并为每个子元素创建一个新的行，然后将offset坐标（即新行数所占的偏移量减去16，再将当前行的偏移量加上16，即offset+16），并将其设置为新的父元素的偏移量。
3. 如果新的行数小于ul元素的子元素数量，那么它将遍历ul元素的子元素，并删除最后一个子元素（即ul元素的最后一个子元素），并将偏移量设置为其父元素的偏移量。

总之，这段代码的主要作用是调整Memory对象中ul元素的子元素的大小，以适应新的行数。


```js
Memory.prototype.resize = function() {
	this.numberRows = this.ul.parentNode.offsetHeight / this.rowHeight + 2;
	if (this.numberRows > this.ul.children.length) {
		var offset = this.ul.lastChild.offset + 16;
		for (var i = 0; i < this.numberRows - this.ul.children.length; ++i) {
			var row = this.createRow(offset);
			this.refresh(row);
			this.ul.appendChild(row);
			offset += 16;
		}
	} else {
		for (var i = 0; i < this.ul.children.length - this.numberRows; ++i) {
			this.ul.removeChild(this.ul.lastChild);
		}
	}
}

```

这段代码是JavaScript中的一个对象的内存 Refresh 方法，其作用是修复内存单元格引用导致的显示错乱和改变。

具体来说，当内存单元格中的值被更改时，这个方法会通过以下步骤来修复：

1. 检查更改的值是否与旧值相同。如果是，那么显示不会发生变化，即直接返回。

2. 如果更改的值与旧值不同，那么将新旧值存储在变量中，并更新内存单元格引用。

3. 通过遍历内存单元格，并检查每个内存单元格中的值是否与新旧值中的任何一个相同，如果是，那么就设置这个内存单元格中的元素显示为绿色的 "memoryCell"。从而突出显示发生了更改的内存单元格。

4. 如果显示变化，那么将新旧值存储在变量中，并更新内存单元格引用。设置显示为绿色的 "memoryCell"。


```js
Memory.prototype.refresh = function(row) {
	var showChanged;
	var newValue;
	var child;
	row.firstChild.textContent = hex(row.offset);
	if (row.oldOffset == row.offset) {
		showChanged = true;
	} else {
		row.oldOffset = row.offset;
		showChanged = false;
	}
	for (var i = 0; i < 16; ++i) {
		child = row.children[i + 1];
		try {
			newValue = this.mmu.loadU8(row.offset + i);
			if (newValue >= 0) {
				newValue = hex(newValue, 2, false);
				if (child.textContent == newValue) {
					child.setAttribute('class', 'memoryCell');
				} else if (showChanged) {
					child.setAttribute('class', 'memoryCell changed');
					child.textContent = newValue;
				} else {
					child.setAttribute('class', 'memoryCell');
					child.textContent = newValue;
				}
			} else {
				child.setAttribute('class', 'memoryCell');
				child.textContent = '--';				
			}
		} catch (exception) {
			child.setAttribute('class', 'memoryCell');
			child.textContent = '--';
		}
	}
}

```

这段代码定义了两个方法，一个是 `Memory.prototype.refreshAll`，另一个是 `Memory.prototype.createRow`。

`refreshAll` 方法的作用是刷新 `Memory` 对象的所有子元素。具体来说，它遍历 `ul` 对象的所有子元素，然后递归地调用 `refresh` 方法，将子元素也进行了刷新。

`createRow` 方法的作用是创建一个行（`li`）元素，并将指定的 `startOffset` 作为行的高度插入到行元素中。行元素包含一个带有类名 `memoryOffset` 的 `span` 元素和一个包含十六进制数字的 `span` 元素，分别表示行的高度和行内的单元格数量。行元素的 `offset` 属性设置为 `startOffset`，并且 `oldOffset` 属性被设置为 `startOffset`，以便在渲染时可以正确地还原行元素的相对位置。

在 `createRow` 方法的实现中，使用了 `document.createElement` 方法创建了行元素的 `span` 元素，并使用 `document.createElement('span')` 方法创建了包含十六进制数字的 `span` 元素。为了使行元素看起来更好看，还使用了 `setAttribute` 方法设置了 `memoryCell` 类别的样式，并将 `class` 属性设置为 `memoryCell`。

然后，在 `for` 循环中，使用 `document.createElement('span')` 方法创建了用于表示单元格数量的 `span` 元素，并使用 `setAttribute` 方法设置了 `class` 属性和 `textContent` 属性。最后，将行元素的 `offset` 属性设置为 `startOffset`，并将 `oldOffset` 属性设置为 `startOffset`。


```js
Memory.prototype.refreshAll = function() {
	for (var i = 0; i < this.ul.children.length; ++i) {
		this.refresh(this.ul.children[i]);
	}
}

Memory.prototype.createRow = function(startOffset) {
	var li = document.createElement('li');
	var offset = document.createElement('span');
	offset.setAttribute('class', 'memoryOffset');
	offset.textContent = hex(startOffset);
	li.appendChild(offset);

	for (var i = 0; i < 16; ++i) {
		var b = document.createElement('span');
		b.textContent = '00';
		b.setAttribute('class', 'memoryCell');
		li.appendChild(b);
	}
	li.offset = startOffset;
	li.oldOffset = startOffset;
	return li;
}

```

这段代码定义了一个名为 "Memory.prototype.scrollTo" 的函数，用于在需要时将视口滚动到内存中的某个位置。它使用了 JavaScript 中的就地（值）级联操作，通过将偏移量 "offset" 增加适当的值，将其存储到 "offset" 变量中，然后使用 var 声明的 "i" 变量对该变量进行循环操作，每次循环操作将视口的某个子元素的 scrollTop 属性设置为上一次 scrollTop 加上当前子元素的偏移量，从而实现了将视口滚动到目标位置的目的。

这段代码的作用是用于实现一个简单的滚动条，该滚动条可以滚动到内存中的某个位置，并且该位置将实时更新到视口的滚动位置。这个滚动条主要用于在某些需要将视口滚动到特定位置的场景中使用，例如在某些浏览器中的表格单元格中。


```js
Memory.prototype.scrollTo = function(offset) {
	offset &= 0xFFFFFFF0;
	if (offset) {
		for (var i = 0; i < this.ul.children.length; ++i) {
			var child = this.ul.children[i];
			child.offset = offset + (i - 1) * 16;
			this.refresh(child);
		}
		this.ul.parentElement.scrollTop = this.scrollTop + this.rowHeight;
	} else {
		for (var i = 0; i < this.ul.children.length; ++i) {
			var child = this.ul.children[i];
			child.offset = offset + i * 16;
			this.refresh(child);
		}
		this.ul.parentElement.scrollTop = this.scrollTop;
	}
}

```

该代码定义了一个名为PaletteViewer的JavaScript对象，用于在画布上显示一个十六进制颜色调制的图集。这个对象包含一个颜色调制的图集、一个用于显示该图集的画布对象和一个绘制颜色调度的绘图上下文。通过调用redraw方法，该对象会在画布上重新绘制它所包含的所有颜色，以使颜色在屏幕上更新。此外，insertChildren方法可以在将新的元素添加到画布上时调用，而redraw方法会在调用时自动调用。


```js
function PaletteViewer(palette) {
	this.palette = palette;
	this.view = document.createElement('canvas');
	this.view.setAttribute('class', 'paletteView');
	this.view.setAttribute('width', '240');
	this.view.setAttribute('height', '500');
}

PaletteViewer.prototype.insertChildren = function(container) {
	container.appendChild(this.view);
}

PaletteViewer.prototype.redraw = function() {
	var context = this.view.getContext('2d');
	context.clearRect(0, 0, this.view.width, this.view.height);
	for (var p = 0; p < 2; ++p) {
		for (var y = 0; y < 16; ++y) {
			for (var x = 0; x < 16; ++x) {
				var color = this.palette.loadU16((p * 256 + y * 16 + x) * 2);
				var r = (color & 0x001F) << 3;
				var g = (color & 0x03E0) >> 2;
				var b = (color & 0x7C00) >> 7;
				context.fillStyle = '#' + hex(r, 2, false) + hex(g, 2, false) + hex(b, 2, false);
				context.fillRect(x * 15 + 1, y * 15 + p * 255 + 1, 13, 13);
			}
		}
	}
}

```

这段代码定义了一个名为 TileViewer 的 JavaScript 类，用于在 VRAM（Video Random Access Memory）中显示一个包含有色块的图象。

该类包含以下方法：

1. `TileViewer(vram, palette)`：构造函数，接受两个参数：VRAM 和颜色调色板。
2. `this.BG_MAP_WIDTH`：一个与 VRAM 相关的常量，用于在图象中绘制背景。
3. `this.vram`：一个与 VRAM 相关的实例变量。
4. `this.palette`：一个与颜色调色板相关的实例变量。
5. `this.activePalette`：一个与当前活动颜色调色板相关的实例变量。
6. `this.insertChildren(container)`：一个私有方法，用于将图象和当前颜色调色板添加到指定的容器中。

该类使用 `document.createElement` 方法创建了一个 `canvas` 元素，并设置其 `class` 属性为 "tileView"，宽度为 256，高度为 512。然后，它将创建一个具有上述属性的 `canvas` 元素，并将其添加到图象容器中。

另外，`TileViewer` 类还有一个 `insertChildren` 方法，用于将图象和当前颜色调色板添加到指定的容器中。


```js
function TileViewer(vram, palette) {
	this.BG_MAP_WIDTH = 256;
	this.vram = vram;
	this.palette = palette;

	this.view = document.createElement('canvas');
	this.view.setAttribute('class', 'tileView');
	this.view.setAttribute('width', '256');
	this.view.setAttribute('height', '512');

	this.activePalette = 0;
}

TileViewer.prototype.insertChildren = function(container) {
	container.appendChild(this.view);
};

```

TileViewer.prototype.loadImage = function(data, palette, offset, stride) {
var context = this.ctx;
var imageData = context.createImageData(this.BG_MAP_WIDTH, 512);
var t = 0;
for (var y = 0; y < 512; y += 8) {
for (var x = 0; x < this.BG_MAP_WIDTH; x += 8) {
var offsetX = x + offset;
var offsetY = y + offset;
var tile = data.data;
		context.putImageData(imageData, offsetX, offsetY, stride, stride);
			++t;
		}
}
context.putImageData(imageData, 0, 0);
};

TileViewer.prototype.drawTile = function(data, tile, palette, offset, stride) {
for (var j = 0; j < 8; ++j) {
var memOffset = tile << 5;
memOffset |= j << 2;
var row = this.vram.load32(memOffset);
for (var i = 0; i < 8; ++i) {
var index = (row >> (i << 2)) & 0xF;
var color = this.palette.loadU16((index << 1) + (palette << 5));
var r = (color & 0x001F) << 3;
var g = (color & 0x03E0) >> 2;
var b = (color & 0x7C00) >> 7;
data[(offset + i + stride * j) * 4 + 0] = r;
data[(offset + i + stride * j) * 4 + 1] = g;
data[(offset + i + stride * j) * 4 + 2] = b;
data[(offset + i + stride * j) * 4 + 3] = 255;
}
};


```js
TileViewer.prototype.redraw = function() {
	var context = this.view.getContext('2d');
	var data = context.createImageData(this.BG_MAP_WIDTH, 512);
	var t = 0;
	for (var y = 0; y < 512; y += 8) {
		for (var x = 0; x < this.BG_MAP_WIDTH; x += 8) {
			this.drawTile(data.data, t, this.activePalette, x + y * this.BG_MAP_WIDTH, this.BG_MAP_WIDTH);
			++t;
		}
	}
	context.putImageData(data, 0, 0);
};

TileViewer.prototype.drawTile = function(data, tile, palette, offset, stride) {
	for (var j = 0; j < 8; ++j) {
		var memOffset = tile << 5;
		memOffset |= j << 2;

		var row = this.vram.load32(memOffset);
		for (var i = 0; i < 8; ++i) {
			var index = (row >> (i << 2)) & 0xF;
			var color = this.palette.loadU16((index << 1) + (palette << 5));
			var r = (color & 0x001F) << 3;
			var g = (color & 0x03E0) >> 2;
			var b = (color & 0x7C00) >> 7;
			data[(offset + i + stride * j) * 4 + 0] = r;
			data[(offset + i + stride * j) * 4 + 1] = g;
			data[(offset + i + stride * j) * 4 + 2] = b;
			data[(offset + i + stride * j) * 4 + 3] = 255;
		}
	}
};

```

# `resources/xhr.js`



该函数使用 XMLHttpRequest 对象从指定 URL 获取数据，并将获取的数据存储在内存中，同时使用回调函数来处理数据。

具体来说，它创建一个 XMLHttpRequest 对象，设置其打开方式为 GET，并指定响应类型为 arraybuffer。然后，它使用 `onload` 事件监听器来处理 XMLHttpRequest 的响应，当响应到来时，将响应数据存储在变量 `xhr.response` 中，并调用回调函数来处理数据。

最后，它使用 `send` 方法发送请求，并将回调函数作为参数传递给 `onload` 事件监听器。当请求发送后，函数会等待响应到来并执行相应的回调函数来处理数据。


```js
function loadRom(url, callback) {
	var xhr = new XMLHttpRequest();
	xhr.open('GET', url);
	xhr.responseType = 'arraybuffer';

	xhr.onload = function() { callback(xhr.response) };
	xhr.send();
}

```