# GBAJS源码解析 5

# `js/util.js`

这段代码定义了一个名为 Object.prototype.inherit 的函数，其作用是继承父对象(Object)的属性。

函数内部通过 for-in 循环遍历父对象的属性，并将其赋值给当前对象(this)。

接下来定义了一个名为 hex 的函数，该函数接受 3 个参数：一个数字(number)、一个字符串(leading)，以及一个布尔值(usePrefix)。函数的作用是在使用 leading 时将数字转换为十六进制字符串，并使用 leading 的前 8 位作为十六进制字符串的 leading。

函数内部首先检查 usePrefix 的值是否为 undefined，如果是，则使用 leading 的值作为十六进制字符串的 leading。然后将数字转换为 16 进制字符串，并使用 leading 作为十六进制字符串的 leading，将结果字符串转换为字符串，并获取其前 8 位作为标识符。最后，如果 leading 小于 0，则返回字符串，否则返回前 8 位标识符(作为十六进制字符串的标识符)。

总结起来，这段代码定义了一个 Object.prototype.inherit 函数和一个 hex 函数，用于在继承时继承父对象的属性和将数字转换为十六进制字符串。


```js
Object.prototype.inherit = function() {
	for (var v in this) {
		this[v] = this[v];
	}
};

function hex(number, leading, usePrefix) {
	if (typeof(usePrefix) === 'undefined') {
		usePrefix = true;
	}
	if (typeof(leading) === 'undefined') {
		leading = 8;
	}
	var string = (number >>> 0).toString(16).toUpperCase();
	leading -= string.length;
	if (leading < 0)
		return string;
	return (usePrefix ? '0x' : '') + new Array(leading + 1).join('0') + string;
}

```

This appears to be a JavaScript function that takes a URL image and serializes it into data that can be sent through the mail. The image is loaded from the URL, and is drawn to the canvas using the Graphics API. The image data is then converted to an array, and the data isqmail，包裹在一个Blob中，并设置序列izer为渠道，以便支持Chrome网络浏览器。最后，Serializer.deserialize将Blob对象的内容读取为数据，并调用该函数。


```js
Serializer = {
	TAG_INT: 1,
	TAG_STRING: 2,
	TAG_STRUCT: 3,
	TAG_BLOB: 4,
	TAG_BOOLEAN: 5,
	TYPE: 'application/octet-stream',

	pointer: function() {
		this.index = 0;
		this.top = 0;
		this.stack = [];
	},

	pack: function(value) {
		var object = new DataView(new ArrayBuffer(4));
		object.setUint32(0, value, true);
		return object.buffer;
	},

	pack8: function(value) {
		var object = new DataView(new ArrayBuffer(1));
		object.setUint8(0, value, true);
		return object.buffer;
	},

	prefix: function(value) {
		return new Blob([Serializer.pack(value.size || value.length || value.byteLength), value], { type: Serializer.TYPE });
	},

	serialize: function(stream) {
		var parts = [];
		var size = 4;
		for (i in stream) {
			if (stream.hasOwnProperty(i)) {
				var tag;
				var head = Serializer.prefix(i);
				var body;
				switch (typeof(stream[i])) {
				case 'number':
					tag = Serializer.TAG_INT;
					body = Serializer.pack(stream[i]);
					break;
				case 'string':
					tag = Serializer.TAG_STRING;
					body = Serializer.prefix(stream[i]);
					break;
				case 'object':
					if (stream[i].type == Serializer.TYPE) {
						tag = Serializer.TAG_BLOB;
						body = stream[i];
					} else {
						tag = Serializer.TAG_STRUCT;
						body = Serializer.serialize(stream[i]);
					}
					break;
				case 'boolean':
					tag = Serializer.TAG_BOOLEAN;
					body = Serializer.pack8(stream[i]);
					break;
				default:
					console.log(stream[i]);
					break;
				}
				size += 1 + head.size + (body.size || body.byteLength || body.length);
				parts.push(Serializer.pack8(tag));
				parts.push(head);
				parts.push(body);
			}
		}
		parts.unshift(Serializer.pack(size));
		return new Blob(parts);
	},

	deserialize: function(blob, callback) {
		var reader = new FileReader();
		reader.onload = function(data) {
			callback(Serializer.deserealizeStream(new DataView(data.target.result), new Serializer.pointer));
		}
		reader.readAsArrayBuffer(blob);
	},

	deserealizeStream: function(view, pointer) {
		pointer.push();
		var object = {};
		var remaining = view.getUint32(pointer.advance(4), true);
		while (pointer.mark() < remaining) {
			var tag = view.getUint8(pointer.advance(1));
			var head = pointer.readString(view);
			var body;
			switch (tag) {
			case Serializer.TAG_INT:
				body = view.getUint32(pointer.advance(4), true);
				break;
			case Serializer.TAG_STRING:
				body = pointer.readString(view);
				break;
			case Serializer.TAG_STRUCT:
				body = Serializer.deserealizeStream(view, pointer);
				break;
			case Serializer.TAG_BLOB:
				var size = view.getUint32(pointer.advance(4), true);
				body = view.buffer.slice(pointer.advance(size), pointer.advance(0));
				break;
			case Serializer.TAG_BOOLEAN:
				body = !!view.getUint8(pointer.advance(1));
				break;
			}
			object[head] = body;
		}
		if (pointer.mark() > remaining) {
			throw "Size of serialized data exceeded";
		}
		pointer.pop();
		return object;
	},

	serializePNG: function(blob, base, callback) {
		var canvas = document.createElement('canvas');
		var context = canvas.getContext('2d');
		var pixels = base.getContext('2d').getImageData(0, 0, base.width, base.height);
		var transparent = 0;
		for (var y = 0; y < base.height; ++y) {
			for (var x = 0; x < base.width; ++x) {
				if (!pixels.data[(x + y * base.width) * 4 + 3]) {
					++transparent;
				}
			}
		}
		var bytesInCanvas = transparent * 3 + (base.width * base.height - transparent);
		for (var multiplier = 1; (bytesInCanvas * multiplier * multiplier) < blob.size; ++multiplier);
		var edges = bytesInCanvas * multiplier * multiplier - blob.size;
		var padding = Math.ceil(edges / (base.width * multiplier));
		canvas.setAttribute('width', base.width * multiplier);
		canvas.setAttribute('height', base.height * multiplier + padding);

		var reader = new FileReader();
		reader.onload = function(data) {
			var view = new Uint8Array(data.target.result);
			var pointer = 0;
			var pixelPointer = 0;
			var newPixels = context.createImageData(canvas.width, canvas.height + padding);
			for (var y = 0; y < canvas.height; ++y) {
				for (var x = 0; x < canvas.width; ++x) {
					var oldY = (y / multiplier) | 0;
					var oldX = (x / multiplier) | 0;
					if (oldY > base.height || !pixels.data[(oldX + oldY * base.width) * 4 + 3]) {
						newPixels.data[pixelPointer++] = view[pointer++];
						newPixels.data[pixelPointer++] = view[pointer++];
						newPixels.data[pixelPointer++] = view[pointer++];
						newPixels.data[pixelPointer++] = 0;
					} else {
						var byte = view[pointer++];
						newPixels.data[pixelPointer++] = pixels.data[(oldX + oldY * base.width) * 4 + 0] | (byte & 7);
						newPixels.data[pixelPointer++] = pixels.data[(oldX + oldY * base.width) * 4 + 1] | ((byte >> 3) & 7);
						newPixels.data[pixelPointer++] = pixels.data[(oldX + oldY * base.width) * 4 + 2] | ((byte >> 6) & 7);
						newPixels.data[pixelPointer++] = pixels.data[(oldX + oldY * base.width) * 4 + 3];
					}
				}
			}
			context.putImageData(newPixels, 0, 0);
			callback(canvas.toDataURL('image/png'));
		}
		reader.readAsArrayBuffer(blob);
		return canvas;
	},

	deserializePNG: function(blob, callback) {
		var reader = new FileReader();
		reader.onload = function(data) {
			var image = document.createElement('img');
			image.setAttribute('src', data.target.result);
			var canvas = document.createElement('canvas');
			canvas.setAttribute('height', image.height);
			canvas.setAttribute('width', image.width);
			var context = canvas.getContext('2d');
			context.drawImage(image, 0, 0);
			var pixels = context.getImageData(0, 0, canvas.width, canvas.height);
			var data = [];
			for (var y = 0; y < canvas.height; ++y) {
				for (var x = 0; x < canvas.width; ++x) {
					if (!pixels.data[(x + y * canvas.width) * 4 + 3]) {
						data.push(pixels.data[(x + y * canvas.width) * 4 + 0]);
						data.push(pixels.data[(x + y * canvas.width) * 4 + 1]);
						data.push(pixels.data[(x + y * canvas.width) * 4 + 2]);
					} else {
						var byte = 0;
						byte |= pixels.data[(x + y * canvas.width) * 4 + 0] & 7;
						byte |= (pixels.data[(x + y * canvas.width) * 4 + 1] & 7) << 3;
						byte |= (pixels.data[(x + y * canvas.width) * 4 + 2] & 7) << 6;
						data.push(byte);
					}
				}
			}
			newBlob = new Blob(data.map(function (byte) {
				var array = new Uint8Array(1);
				array[0] = byte;
				return array;
			}), { type: Serializer.TYPE});
			Serializer.deserialize(newBlob, callback);
		}
		reader.readAsDataURL(blob);
	}
};

```

这段代码定义了三个方法，分别是Serializer.pointer.prototype.advance、Serializer.pointer.prototype.mark 和 Serializer.pointer.prototype.push。下面将分别解释这三个方法的作用。

1. Serializer.pointer.prototype.advance(amount) 是一个 advanced 函数，它接收一个 amount 参数。通过调用这个函数，可以指定一个指针（pointer）对象在数组中的移动步长（amount）。每次调用这个函数时，index 变量会增加 amount 指定的步长，然后返回新的索引（index）值。

2. Serializer.pointer.prototype.mark() 是一个 mark 函数，它返回一个指针对象在数组中的位置（position）索引。通过调用这个函数，可以指定一个指针（pointer）对象在数组中的位置，然后返回这个位置索引（position）值。

3. Serializer.pointer.prototype.push() 是一个 push 函数，它接收一个 top 参数。通过调用这个函数，可以将自己的 top 值添加到数组的末尾（end），并将 top 指针移动到当前索引（index）。


```js
Serializer.pointer.prototype.advance = function(amount) {
	var index = this.index;
	this.index += amount;
	return index;
};

Serializer.pointer.prototype.mark = function() {
	return this.index - this.top;
};

Serializer.pointer.prototype.push = function() {
	this.stack.push(this.top);
	this.top = this.index;
};

```

该代码定义了两个方法，一个是 `Serializer.pointer.prototype.pop`，另一个是 `Serializer.pointer.prototype.readString`。下面分别解释这两个方法的作用。

1. `Serializer.pointer.prototype.pop` 方法：

该方法实现了一个栈的弹出操作，将当前栈顶的元素赋值给 `this.top` 属性，并返回栈顶元素。这个方法在 Pointer 类中比较重要，因为它实现了 Pointer 类将栈中的元素弹出到 `this` 对象的实践。

2. `Serializer.pointer.prototype.readString` 方法：

该方法接受一个 `view` 对象作为参数，并返回一个由字符组成的字符串。它通过 `this.advance` 方法从 `view` 对象中读取字符，并计算出字符串的长度。接着，它通过循环，将每个字符的编码值存储在一个字符数组中，并使用字符串的 `join` 方法将编码后的字符连接成一个新的字符串。最后，它返回这个新的字符串。

需要注意的是，`Serializer.pointer.prototype.readString` 方法中的编码值范围是 `0` 到 `255`，而 `Serializer.pointer.prototype.advance` 方法的编码值范围是 `0` 到 `4294967295`。


```js
Serializer.pointer.prototype.pop = function() {
	this.top = this.stack.pop();
};

Serializer.pointer.prototype.readString = function(view) {
	var length = view.getUint32(this.advance(4), true);
	var bytes = [];
	for (var i = 0; i < length; ++i) {
		bytes.push(String.fromCharCode(view.getUint8(this.advance(1))));
	}
	return bytes.join('');
};

```

# `js/video.js`



这段代码定义了一个名为 "GameBoyAdvanceVideo" 的函数，它继承自一个名为 "GameBoyAdvanceSoftwareRenderer" 的类，用于在 21 位游戏机上播放视频。

以下是该函数的主要作用和组成部分的解释：

- `this.renderPath` 属性指定了要使用的图形渲染器。
- `this.CYCLES_PER_PIXEL` 属性定义了每行中的画笔数量，也就是在屏幕上每行显示的像素数量。
- `this.HORIZONTAL_PIXELS`、`this.HBLANK_PIXELS`、`this.HDRAW_LENGTH`、`this.HORIZONTAL_LENGTH` 和 `this.VERTICAL_PIXELS` 属性定义了屏幕上每列的像素数量以及空白和可绘制图形的总像素数量。
- `this.TOTAL_LENGTH` 属性定义了总像素数。
- `this.drawCallback` 和 `this.vblankCallback` 属性分别定义了在绘制图形时的回调函数，这些回调函数在游戏机启动时被调用，并在绘制图形时被再次调用。
- `this.renderPath.initialize = function()` 函数在初始化渲染器时被调用，用于设置初始状态，例如设置屏幕大小、清除屏幕等。
- `this.renderPath.paint = function()` 函数在绘制图形时被调用，用于将游戏机屏幕上的图形绘制到屏幕上。
- `this.renderPath.update = function()` 函数在更新屏幕时被调用，用于更新游戏机屏幕上的图形。


```js
function GameBoyAdvanceVideo() {
	this.renderPath = new GameBoyAdvanceSoftwareRenderer();

	this.CYCLES_PER_PIXEL = 4;

	this.HORIZONTAL_PIXELS = 240;
	this.HBLANK_PIXELS = 68;
	this.HDRAW_LENGTH = 1006;
	this.HBLANK_LENGTH = 226;
	this.HORIZONTAL_LENGTH = 1232;

	this.VERTICAL_PIXELS = 160;
	this.VBLANK_PIXELS = 68;
	this.VERTICAL_TOTAL_PIXELS = 228;

	this.TOTAL_LENGTH = 280896;

	this.drawCallback = function() {};
	this.vblankCallback = function() {};
};

```

这段代码是关于GameBoyAdvanceVideo.prototype.clear()函数的实现。该函数的主要作用是清除游戏机的屏幕，并将所有与屏幕相关的状态清零。以下是具体的实现步骤：

1. 使用this.renderPath.clear(this.cpu.mmu)将屏幕的渲染路径中的所有内容清空。

2. 设置DISPSTAT_MASK为0xFF38，以便在屏幕初始化时设置一些初始状态值。

3. 设置inHblank为false和inVblank为false，以便在屏幕初始化时初始化H和V blank。

4. 设置vcounter为0，以便在屏幕初始化时记录当前计数器中的计数值。

5. 设置vblankIRQ和hblankIRQ为0，以便在屏幕初始化时记录当前计数器中的计数值。

6. 设置vcountSetting为0，以便在屏幕初始化时设置计数器的计数模式。

7. 设置lastHblank为0，设置nextHblank为this.HDRAW_LENGTH，设置nextEvent为this.nextHblank。

8. 设置nextHblankIRQ为0，设置nextVblankIRQ为0，设置nextVcounterIRQ为0。

9. 在屏幕初始化完成后，该函数就不再被调用。


```js
GameBoyAdvanceVideo.prototype.clear = function() {
	this.renderPath.clear(this.cpu.mmu);

	// DISPSTAT
	this.DISPSTAT_MASK = 0xFF38;
	this.inHblank = false;
	this.inVblank = false;
	this.vcounter = 0;
	this.vblankIRQ = 0;
	this.hblankIRQ = 0;
	this.vcounterIRQ = 0;
	this.vcountSetting = 0;

	// VCOUNT
	this.vcount = -1;

	this.lastHblank = 0;
	this.nextHblank = this.HDRAW_LENGTH;
	this.nextEvent = this.nextHblank;

	this.nextHblankIRQ = 0;
	this.nextVblankIRQ = 0;
	this.nextVcounterIRQ = 0;
};

```

这段代码是 GameBoyAdvanceVideo.prototype.freeze() 方法，该方法的作用是缓存当前游戏板子的状态，以便在播放下一个视频帧时可以更快地重新加载游戏板子的状态。

具体来说，该方法返回一个包含以下属性的对象：

- inHblank: 当前游戏板子的初始化状态。
- inVblank: 当前游戏板子的初始化状态(与 inHblank 类似，但是仅在垂直方向上)。
- vcounter: 当前游戏板子的垂直计数器。
- vblankIRQ: 当前游戏板子的垂直 blank IRQ(即垂直方向上的空白槽)。
- hblankIRQ: 当前游戏板子的水平 blank IRQ(即水平方向上的空白槽)。
- vcountSetting: 当前游戏板子的垂直计数器设置。
- vcount: 当前游戏板子的垂直计数器值。
- lastHblank: 当前游戏板子的最后一个水平 blank。
- nextHblank: 下一个游戏板子的第一个水平 blank。
- nextEvent: 当前游戏板子的下一个垂直 blank IRQ。
- nextVblankIRQ: 下一个游戏板子的下一个垂直 blank IRQ。
- nextVcounterIRQ: 下一个游戏板子的下一个垂直计数器 IRQ。
- renderPath: 用于在 GameBoy Advance 画面中渲染背景的路径。

当该方法被调用时，它会在 GameBoyAdvanceVideo.prototype 的框架中添加上述属性，并确保这些属性在下一个视频帧加载游戏板子时被初始化。这样，游戏板子的初始状态就可以更快地在下一个视频帧中加载。


```js
GameBoyAdvanceVideo.prototype.freeze = function() {
	return {
		'inHblank': this.inHblank,
		'inVblank': this.inVblank,
		'vcounter': this.vcounter,
		'vblankIRQ': this.vblankIRQ,
		'hblankIRQ': this.hblankIRQ,
		'vcounterIRQ': this.vcounterIRQ,
		'vcountSetting': this.vcountSetting,
		'vcount': this.vcount,
		'lastHblank': this.lastHblank,
		'nextHblank': this.nextHblank,
		'nextEvent': this.nextEvent,
		'nextHblankIRQ': this.nextHblankIRQ,
		'nextVblankIRQ': this.nextVblankIRQ,
		'nextVcounterIRQ': this.nextVcounterIRQ,
		'renderPath': this.renderPath.freeze(this.core.encodeBase64)
	};
};

```

这段代码是 GameBoyAdvanceVideo.prototype.defrost 的定义，它是用于将一个视频帧从 RAM 中刷新到屏幕上的函数。

具体来说，它做了以下几件事情：

1. 它读取了一个叫做 frost 的参数，并将其存储在 GameBoyAdvanceVideo.prototype 的 inHblank、inVblank、vcounter、vblankIRQ、hblankIRQ、vcountSetting、vcount 和 lastHblank 成员中。

2. 它读取了 frost 对象中定义的 renderPath 属性，并将其存储在 GameBoyAdvanceVideo.prototype 的 renderPath 成员中。

3. 它设置了一系列与视频帧有关的计数器，包括 vcounter、vblankIRQ、hblankIRQ、nextVblankIRQ、nextVcounterIRQ 和 renderPath 的 defrost 函数。

4. 它调用 GameBoyAdvanceVideo.prototype 的 render 函数，并传递 inHblank、inVblank、vcounter 和 renderPath 参数。

5. 最后，它会根据传入的 frost 对象，更新游戏中的状态，包括切换到下一帧、更新计数器、清除屏幕等等。


```js
GameBoyAdvanceVideo.prototype.defrost = function(frost) {
	this.inHblank = frost.inHblank;
	this.inVblank = frost.inVblank;
	this.vcounter = frost.vcounter;
	this.vblankIRQ = frost.vblankIRQ;
	this.hblankIRQ = frost.hblankIRQ;
	this.vcounterIRQ = frost.vcounterIRQ;
	this.vcountSetting = frost.vcountSetting;
	this.vcount = frost.vcount;
	this.lastHblank = frost.lastHblank;
	this.nextHblank = frost.nextHblank;
	this.nextEvent = frost.nextEvent;
	this.nextHblankIRQ = frost.nextHblankIRQ;
	this.nextVblankIRQ = frost.nextVblankIRQ;
	this.nextVcounterIRQ = frost.nextVcounterIRQ;
	this.renderPath.defrost(frost.renderPath, this.core.decodeBase64);
};

```

这段代码是 GameBoyAdvanceVideo.prototype.setBacking 的定义，用于在游戏板(GameBoy)上设置背景图像。

首先，它将传入一个背景图像，将其缓存到变量 backing 中。然后，它将清除原有的背景图像，以便在设置新的背景图像时不会出现重叠或混淆。

接下来，它通过 this.HORIZONTAL_PIXELS 和 this.VERTICAL_PIXELS 计算出背景图像在游戏板上的像素数量。然后，它循环遍历每个像素，设置它的数据为前景颜色(0xFF)来占据空间，从而清除背景并创建一个新的背景图像。

最后，它将背景图像设置为传入的图像数据，以便在游戏过程中正确显示。


```js
GameBoyAdvanceVideo.prototype.setBacking = function(backing) {
	var pixelData = backing.createImageData(this.HORIZONTAL_PIXELS, this.VERTICAL_PIXELS);
	this.context = backing;

	// Clear backing first
	for (var offset = 0; offset < this.HORIZONTAL_PIXELS * this.VERTICAL_PIXELS * 4;) {
		pixelData.data[offset++] = 0xFF;
		pixelData.data[offset++] = 0xFF;
		pixelData.data[offset++] = 0xFF;
		pixelData.data[offset++] = 0xFF;
	}

	this.renderPath.setBacking(pixelData);
}

```

This appears to be a description of a firmware or software component, specifically for a video game console or PC, that manages input and output for the game. It appears to use a hardware-based video blanking functionality, where the input device (such as a controller or mouse) is inserted into a specific section of the console or PC that is currently set to a blank state.

The firmware or software also appears to manage the input and output of the video display, which is likely to be a separate hardware device from the console or PC.

The documentation mentions a number of different settings and features, such as the ability to enable or disable the video blanking, the setting for the video blanking, and the starting position of the video blanking. It also mentions different types of input and output devices, such as the ability to connect external devices to the console or PC, and the ability to configure the settings of the devices.

There is also a section of the documentation that describes the different types of input and output devices that are supported, such as joysticks, mice, and gamepads. It provides details on how to set up the devices for use, and how to configure the settings of the devices.

Overall, it appears that this is a comprehensive firmware or software solution that is designed to manage the input and output of a wide range of video game devices, including the hardware video blanking functionality.


```js
GameBoyAdvanceVideo.prototype.updateTimers = function(cpu) {
	var cycles = cpu.cycles;

	if (this.nextEvent <= cycles) {
		if (this.inHblank) {
			// End Hblank
			this.inHblank = false;
			this.nextEvent = this.nextHblank;

			++this.vcount;

			switch (this.vcount) {
			case this.VERTICAL_PIXELS:
				this.inVblank = true;
				this.renderPath.finishDraw(this);
				this.nextVblankIRQ = this.nextEvent + this.TOTAL_LENGTH;
				this.cpu.mmu.runVblankDmas();
				if (this.vblankIRQ) {
					this.cpu.irq.raiseIRQ(this.cpu.irq.IRQ_VBLANK);
				}
				this.vblankCallback();
				break;
			case this.VERTICAL_TOTAL_PIXELS - 1:
				this.inVblank = false;
				break;
			case this.VERTICAL_TOTAL_PIXELS:
				this.vcount = 0;
				this.renderPath.startDraw();
				break;
			}

			this.vcounter = this.vcount == this.vcountSetting;
			if (this.vcounter && this.vcounterIRQ) {
				this.cpu.irq.raiseIRQ(this.cpu.irq.IRQ_VCOUNTER);
				this.nextVcounterIRQ += this.TOTAL_LENGTH;
			}

			if (this.vcount < this.VERTICAL_PIXELS) {
				this.renderPath.drawScanline(this.vcount);
			}
		} else {
			// Begin Hblank
			this.inHblank = true;
			this.lastHblank = this.nextHblank;
			this.nextEvent = this.lastHblank + this.HBLANK_LENGTH;
			this.nextHblank = this.nextEvent + this.HDRAW_LENGTH;
			this.nextHblankIRQ = this.nextHblank;

			if (this.vcount < this.VERTICAL_PIXELS) {
				this.cpu.mmu.runHblankDmas();
			}
			if (this.hblankIRQ) {
				this.cpu.irq.raiseIRQ(this.cpu.irq.IRQ_HBLANK);
			}
		}
	}
};

```

这段代码是用来在GameBoyAdvanceVideo.prototype中实现一个writeDisplayStat方法的方法。

该方法接收一个value参数，它代表要显示的图案的值。

然后通过判断该value的最低位是否为1（即0x0001），来判断是否需要执行HBlank和VBlank的刷新。

接着，根据该value的值，计算出下一个VcounterIRQ和下一个HBlank的值，并更新显示设置中的计数值。

如果当前正在显示的图案是HBlank，则会检查下一个VcounterIRQ是否在HBlank的下一个时间间隔内，如果不是，则将当前的VcounterIRQ设置为一个更大的值，以确保可以正确显示下一个HBlank。

最后，如果该方法正在被调用，但是当前的显示计数值还没有更新，则会将当前的计数值更新到下一个VcounterIRQ和下一个HBlank的值，以确保可以正确显示图案。


```js
GameBoyAdvanceVideo.prototype.writeDisplayStat = function(value) {
	this.vblankIRQ = value & 0x0008;
	this.hblankIRQ = value & 0x0010;
	this.vcounterIRQ = value & 0x0020;
	this.vcountSetting = (value & 0xFF00) >> 8;

	if (this.vcounterIRQ) {
		// FIXME: this can be too late if we're in the middle of an Hblank
		this.nextVcounterIRQ = this.nextHblank + this.HBLANK_LENGTH + (this.vcountSetting - this.vcount) * this.HORIZONTAL_LENGTH;
		if (this.nextVcounterIRQ < this.nextEvent) {
			this.nextVcounterIRQ += this.TOTAL_LENGTH;
		}
	}
};

```

这两段代码是JavaScript对GameBoyAdvanceVideo.prototype对象的扩展，主要作用是读取和绘制游戏画面。

1. GameBoyAdvanceVideo.prototype.readDisplayStat = function() {
这一段代码返回GameBoyAdvanceVideo对象在空白行（inVblank）和扫描线（inHblank）以及计数器（vcounter）的组合，它是一个3位的二进制数，对应游戏画面上的空白行、扫描线和计数器的状态。空白行表示屏幕上是否有显示游戏画面，扫描线表示游戏画面是否在显示计数器中的某个位置，计数器表示游戏画面在显示计数器中的位置。

2. GameBoyAdvanceVideo.prototype.finishDraw = function(pixelData) {
这一段代码用来结束游戏画面的绘制，将游戏画面中的像素数据（pixelData）绘制到屏幕上。首先，将像素数据传递给JavaScript的绘图上下文（context）。然后，调用一个名为“drawCallback”的函数，该函数将被绘制的像素数据传递给上下文。通过调用“drawCallback”，可以确保游戏画面中的所有像素数据在绘制完成后被正确地显示出来。


```js
GameBoyAdvanceVideo.prototype.readDisplayStat = function() {
	return (this.inVblank) | (this.inHblank << 1) | (this.vcounter << 2);
};

GameBoyAdvanceVideo.prototype.finishDraw = function(pixelData) {
	this.context.putImageData(pixelData, 0, 0);
	this.drawCallback();
};

```

# `js/video/proxy.js`



该代码定义了一个名为MemoryProxy的JavaScript对象，用于实现对内存块的代理。

该对象的属性包括：

- owner：指定内存块的所有者，可以是任何JavaScript对象。
- blocks：一个块状内存区，用于存储内存块。
- blockSize：指定每个内存块的大小，以字节为单位。
- mask：指定内存块的掩码，用于指定哪些块属于该内存块。
- size：指定内存块的总大小，以字节为单位。

该对象的方法包括：

- initialize：初始化内存块，创建一个大小为size的内存块，并将其块状分配。如果块状分配成功，将初始化成功返回。
- fromArray：从给定的JavaScript数组中复制一个大小为size的内存块，并将其块状分配。如果从数组中复制成功，将返回分配的内存块。
- toArray：将内存块中的所有块复制到一个JavaScript数组中，并返回该数组。
- size：返回内存块的总大小。

该内存块的块状分配策略如下：

- 如果块状大小为0，则直接使用该内存块的全部空间，并返回该内存块。
- 如果块状大小为1，则创建一个大小为1<<blockSize的内存块，并将该块中的所有块复制到新创建的内存块中，并将该块添加到blocks数组中。
- 如果块状大小大于1，则创建一个大小为31的内存块，其中包含所有块的掩码，并将该块添加到blocks数组中。然后，遍历blocks数组，将所有块复制到新创建的内存块中，并更新blocks数组中块的掩码。最后，将新创建的内存块添加到blocks数组中。


```js
function MemoryProxy(owner, size, blockSize) {
	this.owner = owner;
	this.blocks = [];
	this.blockSize = blockSize;
	this.mask = (1 << blockSize) - 1;
	this.size = size;
	if (blockSize) {
		for (var i = 0; i < (size >> blockSize); ++i) {
			this.blocks.push(new MemoryView(new ArrayBuffer(1 << blockSize)));
		}
	} else {
		this.blockSize = 31;
		this.mask = -1;
		this.blocks[0] = new MemoryView(new ArrayBuffer(size));
	}
};

```

这是一个JavaScript类MemoryProxy的methods属性的结合。

combine()方法：
该方法的作用是将给定的内存对象中的所有块合并成一个新对象，并返回合并后的内存对象。

if条件判断：
- 如果内存对象中的块的数量大于1，那么创建一个新对象，并将所有块合并成一个新对象。
- 如果内存对象中只有一个块，那么直接返回该块的内存对象。

replace()方法：
该方法的作用是将一个内存对象(buffer)替换为另一个内存对象(new memoryView)。

该方法接收一个内存对象(buffer)，并将其中的所有块复制到一个新的内存对象中。然后，它通过块索引(注意，不是slice()方法)复制块的起始位置和长度，因此，它将用新的内存对象替换原始内存对象。


```js
MemoryProxy.prototype.combine = function() {
	if (this.blocks.length > 1) {
		var combined = new Uint8Array(this.size);
		for (var i = 0; i < this.blocks.length; ++i) {
			combined.set(new Uint8Array(this.blocks[i].buffer), i << this.blockSize);
		}
		return combined.buffer;
	} else {
		return this.blocks[0].buffer;
	}
};

MemoryProxy.prototype.replace = function(buffer) {
	for (var i = 0; i < this.blocks.length; ++i) {
		this.blocks[i] = new MemoryView(buffer.slice(i << this.blockSize, (i << this.blockSize) + this.blocks[i].buffer.byteLength));
	}
};

```

这段代码定义了一个MemoryProxy对象，它的load8, load16, loadU8, and loadU16方法均属于一个名为0的函数，接收一个8或16位整数偏移量作为参数，返回对应的块中的字节数组。

load8方法的实现比较简单，直接从块中对应偏移量的位置取出字节数组，并解码为8位。

load16方法的实现也比较简单，同样从块中对应偏移量的位置取出字节数组，并解码为16位。

loadU8方法的实现稍微复杂一些，从块中对应偏移量的位置取出字节数组，并解码为8位无符号整数。注意，由于是字节数组，所以需要对结果进行一次双截断，将最高位清零。

loadU16方法的实现则更加复杂，需要从块中对应偏移量的位置取出字节数组，并解码为16位无符号整数。同样需要注意，由于是字节数组，所以需要对结果进行一次双截断，将最高位清零。


```js
MemoryProxy.prototype.load8 = function(offset) {
	return this.blocks[offset >> this.blockSize].load8(offset & this.mask);
};

MemoryProxy.prototype.load16 = function(offset) {
	return this.blocks[offset >> this.blockSize].load16(offset & this.mask);
};

MemoryProxy.prototype.loadU8 = function(offset) {
	return this.blocks[offset >> this.blockSize].loadU8(offset & this.mask);
};

MemoryProxy.prototype.loadU16 = function(offset) {
	return this.blocks[offset >> this.blockSize].loadU16(offset & this.mask);
};

```

这段代码定义了一个名为MemoryProxy的JavaScript类，它实现了对内存的读写操作。在读方面，它使用了JavaScript中的块（block）和偏移量（offset）的概念，通过从内存中的一个块中读取数据，并将其块上的数据解码成32位无符号整数，然后将其存储到目标块中。写方面，它使用了类似于 above 的机制，但只将8位无符号整数的值存储到目标块中，同时需要确保目标块的偏移量不越界。

在实际应用中，MemoryProxy的作用是充当内存的客户端和目标，它将客户端的内存映射到目标内存的块上，并负责处理目标块的读写操作。通过这种方式，可以实现对目标内存的并发访问，而不需要担心目标内存空间的不连续性。


```js
MemoryProxy.prototype.load32 = function(offset) {
	return this.blocks[offset >> this.blockSize].load32(offset & this.mask);
};

MemoryProxy.prototype.store8 = function(offset, value) {
	if (offset >= this.size) {
		return;
	}
	this.owner.memoryDirtied(this, offset >> this.blockSize);
	this.blocks[offset >> this.blockSize].store8(offset & this.mask, value);
	this.blocks[offset >> this.blockSize].store8((offset & this.mask) ^ 1, value);
};

MemoryProxy.prototype.store16 = function(offset, value) {
	if (offset >= this.size) {
		return;
	}
	this.owner.memoryDirtied(this, offset >> this.blockSize);
	return this.blocks[offset >> this.blockSize].store16(offset & this.mask, value);
};

```

这段代码定义了一个 `MemoryProxy` 类，用于在页面上存储和读取 32 位无符号整数。

该 `MemoryProxy` 类包含一个 `store32` 方法和一个 `invalidatePage` 方法。

- `store32` 方法接受两个参数：一个整数偏移量和一个 32 位无符号整数。如果偏移量超出了对象的大小，函数返回，不会做任何操作。否则，该方法会将 `this.owner` 对象中的内存标记为失效，然后尝试通过 `this.blocks` 数组中当前偏移量对应的块来写入值。

- `invalidatePage` 方法接受一个单例的 `address` 参数，用于通知所有需要重新绘制的像素。

该代码创建了一个 `GameBoyAdvanceRenderProxy` 类，用于在页面上渲染游戏画面。该类在 `FinishDraw` 函数中使用 `finish` 函数来通知浏览器重绘已经完成的部分，并在必要时设置 `delay` 并暂停帧率。


```js
MemoryProxy.prototype.store32 = function(offset, value) {
	if (offset >= this.size) {
		return;
	}
	this.owner.memoryDirtied(this, offset >> this.blockSize);
	return this.blocks[offset >> this.blockSize].store32(offset & this.mask, value);
};

MemoryProxy.prototype.invalidatePage = function(address) {};

function GameBoyAdvanceRenderProxy() {
	this.worker = new Worker('js/video/worker.js');

	this.currentFrame = 0;
	this.delay = 0;
	this.skipFrame = false;

	this.dirty = null;
	var self = this;
	var handlers = {
		finish: function(data) {
			self.backing = data.backing;
			self.caller.finishDraw(self.backing);
			--self.delay;
		}
	};
	this.worker.onmessage = function(message) {
		handlers[message.data['type']](message.data);
	}
};

```

这段代码定义了一个名为 `GameBoyAdvanceRenderProxy.prototype.memoryDirtied` 的函数，它接受两个参数：一个内存变量 `mem` 和一个二进制块 `block`。

该函数的主要作用是检查给定的内存区域是否已经被激活，如果该内存区域是活动状态，那么函数会将相应区域的脏数据（即初始值）复制到 `dirty` 对象中。如果给定的内存区域不是活动状态，那么函数不会做任何事情，但是需要在 `dirty` 对象中记录下来，以便在后续清理时处理。

具体来说，该函数主要实现了以下操作：

1. 将给定的 `mem` 内存区域与 `dirty` 对象中的所有其他区域进行比较，如果两个区域都相同，那么将对应区域的脏数据复制到 `dirty.memory` 对象中。
2. 如果给定的 `mem` 内存区域与 `dirty` 对象中的任何区域都不同，那么创建一个新的 `dirty.memory.vram` 数组，并将新区域的脏数据复制到该数组中，同时将新区域的数据复制到 `dirty.memory.vram[block]` 属性中。

最后，该函数被归类到 `GameBoyAdvanceRenderProxy.prototype` 对象中，以便在整个游戏中均匀使用，从而确保所有游戏板子的渲染过程中都能够正确处理脏数据。


```js
GameBoyAdvanceRenderProxy.prototype.memoryDirtied = function(mem, block) {
	this.dirty = this.dirty || {};
	this.dirty.memory = this.dirty.memory || {};
	if (mem === this.palette) {
		this.dirty.memory.palette = mem.blocks[0].buffer;
	}
	if (mem === this.oam) {
		this.dirty.memory.oam = mem.blocks[0].buffer;
	}
	if (mem === this.vram) {
		this.dirty.memory.vram = this.dirty.memory.vram || [];
		this.dirty.memory.vram[block] = mem.blocks[block].buffer;
	}
};

```

这段代码是关于GameBoyAdvanceRenderProxy的清除和冻结方法。

这段代码定义了两个函数，一个是clear，另一个是freeze。

1. clear函数：

这个函数的作用是清除GameBoyAdvanceRenderProxy的内存，并创建一个新的内存区域。然后将GameBoyAdvanceRenderProxy中的内存置为脏(dirty)，并将scanline队列清空。最后，将一个包含初始化数据的Message对象发送给worker，这个对象会被用于同步化。

2. freeze函数：

这个函数的作用是接收一个编码为base64的值，并将其转换为一个对象，包含三个属性，分别是在屏幕上(palette)、虚拟内存中(vram)和OAM中的数据。然后将这个对象存储到变量中，并将GameBoyAdvanceRenderProxy中的内存置为冻结状态。这个函数会在需要时重新初始化屏幕上、虚拟内存中和OAM中的数据，但不会在每次启动游戏时都重新初始化。

这两个函数可以保证GameBoyAdvanceRenderProxy在清除或冻结时，可以正确地初始化游戏机的内存，从而确保游戏能够在正确的时机开始运行。


```js
GameBoyAdvanceRenderProxy.prototype.clear = function(mmu) {
	this.palette = new MemoryProxy(this, mmu.SIZE_PALETTE_RAM, 0);
	this.vram = new MemoryProxy(this, mmu.SIZE_VRAM, 13);
	this.oam = new MemoryProxy(this, mmu.SIZE_OAM, 0);

	this.dirty = null;
	this.scanlineQueue = [];

	this.worker.postMessage({ type: 'clear', SIZE_VRAM: mmu.SIZE_VRAM, SIZE_OAM: mmu.SIZE_OAM });
};

GameBoyAdvanceRenderProxy.prototype.freeze = function(encodeBase64) {
	return {
		'palette': Serializer.prefix(this.palette.combine()),
		'vram': Serializer.prefix(this.vram.combine()),
		'oam': Serializer.prefix(this.oam.combine())
	};
};

```

这两段代码是JavaScript中的函数，它们属于GameBoyAdvanceRenderProxy类。

第一段代码定义了一个名为defrost的方法，该方法接受两个参数：frost和decodeBase64。defrost的作用是将frost对象和decodeBase64对象中的像素值替换到GameBoyAdvanceRenderProxy实例的palette和vram成员中。然后，它遍历vram中的每个块，并将frost对象和decodeBase64对象中的对应块的像素值替换到该块中。

第二段代码定义了一个名为writeDisplayControl的方法，该方法接受一个参数：一个布尔值，表示是否重置显示控制器。writeDisplayControl的作用是在屏幕上设置一个显示控制器的状态，将其值存储在GameBoyAdvanceRenderProxy实例的一个dirty对象中。


```js
GameBoyAdvanceRenderProxy.prototype.defrost = function(frost, decodeBase64) {
	this.palette.replace(frost.palette);
	this.memoryDirtied(this.palette, 0);
	this.vram.replace(frost.vram);
	for (var i = 0; i < this.vram.blocks.length; ++i) {
		this.memoryDirtied(this.vram, i);
	}
	this.oam.replace(frost.oam);
	this.memoryDirtied(this.oam, 0);
};

GameBoyAdvanceRenderProxy.prototype.writeDisplayControl = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.DISPCNT = value;
};

```

这段代码是针对GameBoyAdvanceRenderProxy类的公有方法，用于在背景图片（Background Image）上设置上下文（Context）。通过写入控制（Write Background Controls）。

首先，在每一个实现此方法的实例中，都会创建一个对象，用于存储背景图片和设置的值，例如：

```jsjavascript
this.dirty = this.dirty || {};
```

这个对象是每个实例都独立的，如果有多个实例，它们都会存在于一个对象中。

接下来，通过添加新的设置值，将该值添加到脏写设置（dirty）中：

```jsjavascript
this.dirty.BGCNT = this.dirty.BGCNT || [];
```

如果dirty对象中不存在BGCNT，那么将其添加到BGCNT数组中。

接着，将新的值添加到指定的背景图片上：

```jsjavascript
this.dirty.BGHOFS = this.dirty.BGHOFS || [];
```

如果dirty对象中不存在BGHOFS，那么将其添加到BGHOFS数组中。

```jsjavascript
this.dirty.BGVOFS = this.dirty.BGVOFS || [];
```

最后，使用writeBackgroundControl、writeBackgroundHOffset、writeBackgroundVOffset方法，分别设置背景图片上下文（Background Image Context）、背景图片水平偏移（Background Image Horizontal Offset）和背景图片垂直偏移（Background Image Vertical Offset）。

这段代码的主要作用是让游戏背景图片具有更好的控制，通过调整上下文、水平偏移和垂直偏移，可以更方便地管理游戏背景图片。


```js
GameBoyAdvanceRenderProxy.prototype.writeBackgroundControl = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGCNT = this.dirty.BGCNT || [];
	this.dirty.BGCNT[bg] = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBackgroundHOffset = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGHOFS = this.dirty.BGHOFS || [];
	this.dirty.BGHOFS[bg] = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBackgroundVOffset = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGVOFS = this.dirty.BGVOFS || [];
	this.dirty.BGVOFS[bg] = value;
};

```

该代码是定义了一个名为GameBoyAdvanceRenderProxy的继承自GameBoyAdvanceRenderProxy的类，并在该类的内部定义了三个方法，分别名为writeBackgroundRefX、writeBackgroundRefY和writeBackgroundParamA。

writeBackgroundRefX方法接受两个参数，第一个参数是一个背景(bg)和一个值(value)，将其写入该类的dirty对象中。如果该类已经存在dirty对象，则直接覆盖原有属性。否则，创建一个新的dirty对象，并添加BGX和BGY数组，将bg和value分别添加到数组中。

writeBackgroundRefY方法与writeBackgroundRefX类似，只是第二个参数是垂直方向上的值。

writeBackgroundParamA方法与writeBackgroundParamB类似，只是接收的参数是垂直方向上而不是水平方向上。


```js
GameBoyAdvanceRenderProxy.prototype.writeBackgroundRefX = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGX = this.dirty.BGX || [];
	this.dirty.BGX[bg] = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBackgroundRefY = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGY = this.dirty.BGY || [];
	this.dirty.BGY[bg] = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBackgroundParamA = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGPA = this.dirty.BGPA || [];
	this.dirty.BGPA[bg] = value;
};

```

这是一段 JavaScript 代码，定义了 GameBoyAdvanceRenderProxy.prototype.writeBackgroundParamB、writeBackgroundParamC 和 writeBackgroundParamD 三个方法，用于在渲染器中设置背景参数（BGPB、BGPC 和 BGPD）。

具体来说，每个方法接收一个背景参数（bg）和一个参数值（value），将它们加入到自身 dirty 对象中。dirty 对象是一个JavaScript对象，用于记录渲染器状态。当渲染器需要重新渲染时，会遍历 dirty 对象，将背景参数和参数值设置为它们原本的值，从而更新渲染结果。


```js
GameBoyAdvanceRenderProxy.prototype.writeBackgroundParamB = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGPB = this.dirty.BGPB || [];
	this.dirty.BGPB[bg] = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBackgroundParamC = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGPC = this.dirty.BGPC || [];
	this.dirty.BGPC[bg] = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBackgroundParamD = function(bg, value) {
	this.dirty = this.dirty || {};
	this.dirty.BGPD = this.dirty.BGPD || [];
	this.dirty.BGPD[bg] = value;
};

```

这段代码是定义了三个函数，分别称为 "writeWin0H"、"writeWin1H" 和 "writeWin0V"，它们都接受一个参数 "value"。

这些函数的作用是为 GameBoyAdvanceRenderProxy 对象的一个核心成员变量（可能是游戏画面或渲染状态）写入到了 Windows 0 版本的 ROM 中。这些函数的具体实现主要依赖于 GameBoyAdvanceRenderProxy.prototype 对象的一个内部的 `writeWin0H` 方法。通过调用这些函数，可以实现对 GameBoyAdvanceRenderProxy 对象中相应成员变量的修改，从而达到修改游戏画面或渲染状态的目的。


```js
GameBoyAdvanceRenderProxy.prototype.writeWin0H = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.WIN0H = value;
};

GameBoyAdvanceRenderProxy.prototype.writeWin1H = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.WIN1H = value;
};

GameBoyAdvanceRenderProxy.prototype.writeWin0V = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.WIN0V = value;
};

```



该代码是定义了三个函数 `writeWin1V`, `writeWinIn`, 和 `writeWinOut`，它们都接受一个参数 `value`。

这些函数的作用是为 GameBoyAdvanceRenderProxy 类的实例添加或设置 `WIN1V`, `WININ`, 和 `WINOUT` 成员变量，以便在渲染过程中正确地设置或清除屏幕。

具体来说，当 `writeWin1V` 函数被调用时，它会在 `this.dirty` 对象中添加一个新的成员变量 `WIN1V`，并将 `value` 参数作为它的值。

类似地，当 `writeWinIn` 函数被调用时，它会在 `this.dirty` 对象中添加一个新的成员变量 `WININ`，并将 `value` 参数作为它的值。

最后，当 `writeWinOut` 函数被调用时，它会在 `this.dirty` 对象中添加一个新的成员变量 `WINOUT`，并将 `value` 参数作为它的值。

这些函数的作用是帮助管理 GameBoyAdvanceRenderProxy 实例的屏幕状态，通过它们可以更方便地在渲染过程中设置或清除屏幕上的精灵，精灵盒，等游戏元素。


```js
GameBoyAdvanceRenderProxy.prototype.writeWin1V = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.WIN1V = value;
};

GameBoyAdvanceRenderProxy.prototype.writeWinIn = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.WININ = value;
};

GameBoyAdvanceRenderProxy.prototype.writeWinOut = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.WINOUT = value;
};

```

该代码定义了 GameBoyAdvanceRenderProxy.prototype.writeBlendControl,writeBlendAlpha 和 writeBlendY 三个方法，用于控制混合模式中红、绿、蓝通道的亮度值和透明度值的写入。

具体来说，writeBlendControl 方法用于设置混合模式中红、绿、蓝通道的亮度值，如果该控制已经存在，则将其添加到 dirty 对象中。如果该控制不存在，则创建一个新的 dirty 对象，其中包含红、绿、蓝通道的亮度值，并将其添加到 dirty 对象中。

writeBlendAlpha 方法与writeBlendControl 类似，但只设置混合模式中绿、蓝通道的亮度值和透明度值，如果该控制已经存在，则将其添加到 dirty 对象中。如果该控制不存在，则创建一个新的 dirty 对象，其中包含绿、蓝通道的亮度值和透明度值，并将其添加到 dirty 对象中。

writeBlendY 方法与writeBlendAlpha 和writeBlendControl 类似，但只设置混合模式中红、蓝通道的亮度值和透明度值，如果该控制已经存在，则将其添加到 dirty 对象中。如果该控制不存在，则创建一个新的 dirty 对象，其中包含红、蓝通道的亮度值和透明度值，并将其添加到 dirty 对象中。


```js
GameBoyAdvanceRenderProxy.prototype.writeBlendControl = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.BLDCNT = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBlendAlpha = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.BLDALPHA = value;
};

GameBoyAdvanceRenderProxy.prototype.writeBlendY = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.BLDY = value;
};

```

该代码定义了GameBoyAdvanceRenderProxy的writeMosaic和clearSubsets方法，用于在渲染过程中同步子部件的属性。

writeMosaic的作用是将给定的值存储在GameBoyAdvanceRenderProxy的dirty对象中，该对象用于记录渲染过程中的状态更改。如果子部件的属性被更改，则需要在屏幕上绘制它们，因此writeMosaic方法确保在每次需要同步子部件时调用。

clearSubsets的作用是清除指定内存区域中所有与游戏板(GameBoy)有关的缓冲区。它与writeMosaic一起使用，因为每次在屏幕上同步子部件时，都需要清除屏幕上的缓冲区，以便下一次的渲染可以重新开始。这个方法的实现分为三个步骤：

1. 如果屏幕上包含了一个可寻址的缓冲区，并且这个缓冲区没有被分配给任何子部件，那么将它清除。

2. 遍历与游戏板有关的内存区域，找到所有与游戏板有关的缓冲区，并将它们清除。

3. 遍历与游戏板有关的内存区域，找到所有与游戏板有关的缓冲区，并将它们清除。

这个实现确保所有与游戏板有关的缓冲区在每次同步游戏板时都被清除，使得屏幕可以正确地重新开始绘制游戏板上的内容。


```js
GameBoyAdvanceRenderProxy.prototype.writeMosaic = function(value) {
	this.dirty = this.dirty || {};
	this.dirty.MOSAIC = value;
};

GameBoyAdvanceRenderProxy.prototype.clearSubsets = function(mmu, regions) {
	this.dirty = this.dirty || {};
	if (regions & 0x04) {
		this.palette = new MemoryProxy(this, mmu.SIZE_PALETTE_RAM, 0);
		mmu.mmap(mmu.REGION_PALETTE_RAM, this.palette);
		this.memoryDirtied(this.palette, 0);
	}
	if (regions & 0x08) {
		this.vram = new MemoryProxy(this, mmu.SIZE_VRAM, 13);
		mmu.mmap(mmu.REGION_VRAM, this.vram);
		for (var i = 0; i < this.vram.blocks.length; ++i) {
			this.memoryDirtied(this.vram, i);
		}
	}
	if (regions & 0x10) {
		this.oam = new MemoryProxy(this, mmu.SIZE_OAM, 0);
		mmu.mmap(mmu.REGION_OAM, this.oam);
		this.memoryDirtied(this.oam, 0);
	}
};

```

这段代码是针对GameBoyAdvanceRenderProxy类的两个静态方法，分别是setBacking和drawScanline。

setBacking方法接受一个backing参数，将其设置为游戏板（底）的背景颜色。然后，它向 worker 发送一个消息，其中包含一个包含 backing 的类型字段，以及一个包含绘制数据的字段。

drawScanline方法用于在屏幕上绘制扫描线。它首先检查是否已经计算过扫描线，如果没有，就继续绘制。然后，它检查是否正在渲染（通过使用skipFrame 是否为true来决定是否渲染）。如果是，那么就遍历内存，提取出OAM数据，对其进行处理并将其存储到内存中。接下来，它将扫描线添加到dirty队列中，然后清空dirty变量。最后，它使用drawScanlineQueue方法来处理扫描线队列，清空queue，并将dirty变量设置为null。


```js
GameBoyAdvanceRenderProxy.prototype.setBacking = function(backing) {
	this.backing = backing;
	this.worker.postMessage({ type: 'start', backing: this.backing });
};

GameBoyAdvanceRenderProxy.prototype.drawScanline = function(y) {
	if (!this.skipFrame) {
		if (this.dirty) {
			if (this.dirty.memory) {
				if (this.dirty.memory.palette) {
					this.dirty.memory.palette = this.dirty.memory.palette.slice(0);
				}
				if (this.dirty.memory.oam) {
					this.dirty.memory.oam = this.dirty.memory.oam.slice(0);
				}
				if (this.dirty.memory.vram) {
					for (var i = 0; i < 12; ++i) {
						if (this.dirty.memory.vram[i]) {
							this.dirty.memory.vram[i] = this.dirty.memory.vram[i].slice(0);
						}
					}
				}
			}
			this.scanlineQueue.push({ y: y, dirty: this.dirty });
			this.dirty = null;
		}
	}
};

```

该代码是针对GameBoyAdvanceRenderProxy的渲染循环进行封装的一个方法。

GameBoyAdvanceRenderProxy.prototype.startDraw方法的作用是在每一次循环开始时增加当前帧数，并根据delay值来决定是否立即开始绘制，如果delay值小于等于0，则不绘制，否则会增加delay值。

GameBoyAdvanceRenderProxy.prototype.finishDraw方法用于结束当前循环的绘制，并传入一个caller参数，它接收到的消息是一个包含状态的对象，包括finish状态、scanlines队列和当前帧数等。

该代码还包含一个skipFrame变量，用于记录是否已经发生了异步绘制，如果已经发生了，则不会执行以下代码：

```js
if (!this.skipFrame) {
		++this.delay;
	}
```

```js
if (this.delay > 2) {
		this.skipFrame = true;
	}
```

这段代码的作用是在delay大于2时，增加delay值来控制异步绘制的频率，从而提高渲染效率。


```js
GameBoyAdvanceRenderProxy.prototype.startDraw = function() {
	++this.currentFrame;
	if (this.delay <= 0) {
		this.skipFrame = false;
	}
	if (!this.skipFrame) {
		++this.delay;
	}
};

GameBoyAdvanceRenderProxy.prototype.finishDraw = function(caller) {
	this.caller = caller;
	if (!this.skipFrame) {
		this.worker.postMessage({ type: 'finish', scanlines: this.scanlineQueue, frame: this.currentFrame });
		this.scanlineQueue = [];
		if (this.delay > 2) {
			this.skipFrame = true;
		}
	}
};

```

# `js/video/software.js`



该代码定义了一个名为MemoryAligned16的函数，它接受一个参数size，表示要分配的内存大小。

在函数内部，使用JavaScript的Array类创建了一个大小为size-1的二维数组buffer，并将其赋值为0。

该函数还定义了两个方法，load8和load16，用于从buffer中读取8位或16位数据。

load8方法的实现是通过在buffer数组的对应位置与offset进行按位与操作，然后将结果左移两位并取反，再将低8位转换为补码并高8位取反，最后将结果返回。

load16方法的实现与load8方法类似，但需要将offset多了一位，即先将offset左移4位，再进行按位与操作，然后将结果左移16位并取反，最后将结果返回。

整个函数的实现方式比较复杂，旨在提供一种高效率的内存读取方式，可以在大内存读取时使用。


```js
function MemoryAligned16(size) {
	this.buffer = new Uint16Array(size >> 1);
};

MemoryAligned16.prototype.load8 = function(offset) {
	return (this.loadU8(offset) << 24) >> 24;
};

MemoryAligned16.prototype.load16 = function(offset) {
	return (this.loadU16(offset) << 16) >> 16;
};

MemoryAligned16.prototype.loadU8 = function(offset) {
	var index = offset >> 1;
	if (offset & 1) {
		return (this.buffer[index] & 0xFF00) >>> 8;
	} else {
		return this.buffer[index] & 0x00FF;
	}
};

```

这段代码定义了一个名为MemoryAligned16的类，其三个实例方法分别名为loadU16、load32和store8，以及一个名为console的构造函数console。这些方法的实现如下：

```js
MemoryAligned16.prototype.loadU16 = function(offset) {
   return this.buffer[offset >> 1];
};

MemoryAligned16.prototype.load32 = function(offset) {
   return this.buffer[(offset >> 1) & ~1] | (this.buffer[(offset >> 1) | 1] << 16);
};

MemoryAligned16.prototype.store8 = function(offset, value) {
   var index = offset >> 1;
   this.store16(offset, (value << 8) | value);
};

MemoryAligned16.prototype.store16 = function(offset, value) {
   this.buffer[offset >> 1] = value;
};
```

首先，定义了一个MemoryAligned16类，包含三个实例方法和一个名为console的构造函数。

loadU16和load32方法都使用this.buffer[offset >> 1]来获取指定偏移量的字节数组元素，并将其返回。

store8方法使用this.store16(offset, (value << 8) | value)来将指定偏移量的字节数组元素存储为value的高位8位和低位8位组成的二进制数。这里的value必须大于0，但小于16。

store16方法使用this.buffer[offset >> 1] = value来将指定偏移量的字节数组元素存储为value。这里的value必须大于0。

console.log方法会在控制台上打印出this.buffer.length。


```js
MemoryAligned16.prototype.loadU16 = function(offset) {
	return this.buffer[offset >> 1];
};

MemoryAligned16.prototype.load32 = function(offset) {
	return this.buffer[(offset >> 1) & ~1] | (this.buffer[(offset >> 1) | 1] << 16);
};

MemoryAligned16.prototype.store8 = function(offset, value) {
	var index = offset >> 1;
	this.store16(offset, (value << 8) | value);
};

MemoryAligned16.prototype.store16 = function(offset, value) {
	this.buffer[offset >> 1] = value;
};

```

这是一个JavaScript类，名为"MemoryAligned16"，实现了游戏BoyAdvanceVRAM的功能。

类中的三个方法具体作用如下：

1. store32：用于向VRAM中写入数据。第一个参数offset表示数据在VRAM中的位置，第二个参数value表示要存储的数据。

2. insert：用于从VRAM中读取数据并将其存储到指定的位置。第一个参数start表示要读取的起始位置，第二个参数data表示要读取的数据。

3. invalidatePage：用于刷新VRAM中的页面，即清除VRAM中所有数据，并将其重置为默认值。

此类的实例可以被赋值给一个变量，例如：

```js
var gameBoyAdvanceVRAM = new GameBoyAdvanceVRAM(8192);
```

这将创建一个大小为8192字节的VRAM，并将其赋值给名为gameBoyAdvanceVRAM的变量。可以使用以上三个方法来对VRAM进行写入和读取，以及通过调用invalidatePage方法来刷新VRAM中的页面。


```js
MemoryAligned16.prototype.store32 = function(offset, value) {
	var index = offset >> 1;
	this.store16(offset, this.buffer[index] = value & 0xFFFF);
	this.store16(offset + 2, this.buffer[index + 1] = value >>> 16);
};

MemoryAligned16.prototype.insert = function(start, data) {
	this.buffer.set(data, start);
};

MemoryAligned16.prototype.invalidatePage = function(address) {};

function GameBoyAdvanceVRAM(size) {
	MemoryAligned16.call(this, size);
	this.vram = this.buffer;
};

```

这段代码定义了一个名为GameBoyAdvanceOAM的类，它继承自MemoryAligned16.prototype。

GameBoyAdvanceOAM类实现了对一个大小为size的二维数组的操作，该数组被称为OAM(Objective and Accessible Model)，并将其作为类属性保存。

此外，该类还创建了一个缓冲区objs和一个scalerot数组，数组长度为32，每个元素都是一个包含四个值的对象，分别代表OAM的四个方面(Objective、Accessible、Model和Texture)。

scalerot数组中的每个值都包含a、b、c、d四个值，分别代表OAM的四个方面中的权重值，从0到255之间。


```js
GameBoyAdvanceVRAM.prototype = Object.create(MemoryAligned16.prototype);

function GameBoyAdvanceOAM(size) {
	MemoryAligned16.call(this, size);
	this.oam = this.buffer;
	this.objs = new Array(128);
	for (var i = 0; i < 128; ++i) {
		this.objs[i] = new GameBoyAdvanceOBJ(this, i);
	}
	this.scalerot = new Array(32);
	for (var i = 0; i < 32; ++i) {
		this.scalerot[i] = {
			a: 1,
			b: 0,
			c: 0,
			d: 1
		};
	}
};

```

This appears to be a JavaScript class for handling 16-bit color images. It appears to have several methods, including store16, which stores a 16-bit color image at a given offset, and several other methods for manipulating the color and orientation of the image.

The store16 method takes an offset and a value as arguments, and stores the image at that offset in the same format. The other methods may have similar behavior, but the exact implementation is not provided.

It's worth noting that this class is defined using the "Texture.derived" module, which suggests that it may be part of a larger JavaScript library for handling texture data.


```js
GameBoyAdvanceOAM.prototype = Object.create(MemoryAligned16.prototype);

GameBoyAdvanceOAM.prototype.overwrite = function(memory) {
	for (var i = 0; i < (this.buffer.byteLength >> 1); ++i) {
		this.store16(i << 1, memory[i]);
	}
};

GameBoyAdvanceOAM.prototype.store16 = function(offset, value) {
	var index = (offset & 0x3F8) >> 3;
	var obj = this.objs[index];
	var scalerot = this.scalerot[index >> 2];
	var layer = obj.priority;
	var disable = obj.disable;
	var y = obj.y;
	switch (offset & 0x00000006) {
	case 0:
		// Attribute 0
		obj.y = value & 0x00FF;
		var wasScalerot = obj.scalerot;
		obj.scalerot = value & 0x0100;
		if (obj.scalerot) {
			obj.scalerotOam = this.scalerot[obj.scalerotParam];
			obj.doublesize = !!(value & 0x0200);
			obj.disable = 0;
			obj.hflip = 0;
			obj.vflip = 0;
		} else {
			obj.doublesize = false;
			obj.disable = value & 0x0200;
			if (wasScalerot) {
				obj.hflip = obj.scalerotParam & 0x0008;
				obj.vflip = obj.scalerotParam & 0x0010;
			}
		}
		obj.mode = (value & 0x0C00) >> 6; // This lines up with the stencil format
		obj.mosaic = value & 0x1000;
		obj.multipalette = value & 0x2000;
		obj.shape = (value & 0xC000) >> 14;

		obj.recalcSize();
		break;
	case 2:
		// Attribute 1
		obj.x = value & 0x01FF;
		if (obj.scalerot) {
			obj.scalerotParam = (value & 0x3E00) >> 9;
			obj.scalerotOam = this.scalerot[obj.scalerotParam];
			obj.hflip = 0;
			obj.vflip = 0;
			obj.drawScanline = obj.drawScanlineAffine;
		} else {
			obj.hflip = value & 0x1000;
			obj.vflip = value & 0x2000;
			obj.drawScanline = obj.drawScanlineNormal;
		}
		obj.size = (value & 0xC000) >> 14;

		obj.recalcSize();
		break;
	case 4:
		// Attribute 2
		obj.tileBase = value & 0x03FF;
		obj.priority = (value & 0x0C00) >> 10;
		obj.palette = (value & 0xF000) >> 8; // This is shifted up 4 to make pushPixel faster
		break;
	case 6:
		// Scaling/rotation parameter
		switch (index & 0x3) {
		case 0:
			scalerot.a = (value << 16) / 0x1000000;
			break;
		case 1:
			scalerot.b = (value << 16) / 0x1000000;
			break;
		case 2:
			scalerot.c = (value << 16) / 0x1000000;
			break;
		case 3:
			scalerot.d = (value << 16) / 0x1000000;
			break;
		}
		break;
	}

	MemoryAligned16.prototype.store16.call(this, offset, value);
};

```

该代码定义了一个名为 "GameBoyAdvancePalette" 的函数，它使用两个数组 "colors" 和 "adjustedColors"，以及一个数组 "passthroughColors"，用于在游戏板(GameBoy)的屏幕背景(backdrop)和前景(obj)。它还定义了一个名为 "blendY" 的变量，表示融合亮度(Blend Mode)，以及一个名为 "overwrite" 的函数，用于向内存写入数据。

具体来说，该函数的作用是，通过将屏幕背景和前景的图像混合在一起，并在游戏板的屏幕背景上增加亮度，从而创建出逼真的游戏画面。通过将背景和前景的图像存储在数组中，并在混合时使用 blendY 变量来控制亮度的强度，从而实现最佳的游戏画面效果。


```js
function GameBoyAdvancePalette() {
	this.colors = [ new Array(0x100), new Array(0x100) ];
	this.adjustedColors = [ new Array(0x100), new Array(0x100) ];
	this.passthroughColors = [
		this.colors[0], // BG0
		this.colors[0], // BG1
		this.colors[0], // BG2
		this.colors[0], // BG3
		this.colors[1], // OBJ
		this.colors[0] // Backdrop
	];
	this.blendY = 1;
};

GameBoyAdvancePalette.prototype.overwrite = function(memory) {
	for (var i = 0; i < 512; ++i) {
		this.store16(i << 1, memory[i]);
	}
};

```

这段代码是关于GameBoyAdvancePalette类的三个load函数，用于从文件中读取2个字节(即4个像素)的颜色数据。

第一个函数是loadU8，第二个函数是loadU16，第三个函数是load16，它们都接收一个整数offset，返回一个0到255之间的字节数。

函数内部首先调用一个内部函数loadU16，传入offset，返回以offset & 1为二进制位宽的4字节颜色值的最高位，然后进行右移8位，再与0xFF进行按位与操作，得到一个0到255之间的字节数，最后将其赋值给loadU8函数返回。

函数内部第二个函数是load16，它和loadU16类似，只不过返回的是一个16字节的颜色值，即4个像素的颜色值。

第三个函数是load32，它和loadU16和load16都类似，只不过返回的是一个32字节的颜色值，即8个像素的颜色值。


```js
GameBoyAdvancePalette.prototype.loadU8 = function(offset) {
	return (this.loadU16(offset) >> (8 * (offset & 1))) & 0xFF;
};

GameBoyAdvancePalette.prototype.loadU16 = function(offset) {
	return this.colors[(offset & 0x200) >> 9][(offset & 0x1FF) >> 1];
};

GameBoyAdvancePalette.prototype.load16 = function(offset) {
	return (this.loadU16(offset) << 16) >> 16;
};

GameBoyAdvancePalette.prototype.load32 = function(offset) {
	return this.loadU16(offset) | (this.loadU16(offset + 2) << 16);
};

```

该代码定义了 GameBoyAdvancePalette.prototype.store16 和 store32 方法，以及 invalidatePage 和 convert16To32 方法。这些方法用于在 GameBoy Advance 游戏开发过程中对颜色进行存储和转换。

store16 方法接受两个参数：偏移量和值。偏移量可以是 0x0001 到 0x00FFF 之间的任意整数，值可以是 0 到 0xFFFF 之间的任意整数。该方法将颜色存储在彩色内存的指定行和列中，通过传输出设备的显示模式，即 0x0001 表示行 0，列 0。

store16 方法的第二部分是调整 Colors 数组，该数组存储了每个像素的 RGB 值。通过 store16 方法的第一个参数 offset，可以指定从哪里开始读取偏移量，以及从哪里开始写入值。

store32 方法与 store16 方法类似，但是每个像素的值是 0xFFFF 中的一个字节，而不是两个字节。

validatePage 方法用于在从内存中读取数据时进行校验，确保所读取的页面的物理地址是正确的。

convert16To32 方法用于将 16 位的颜色值转换成 32 位的颜色值。这个方法将两个输入参数（颜色值和输入图像的 RGB 通道）传递给输出函数，然后将输出图像的每个像素的 RGB 值替换为输入图像的对应像素的 RGB 值。


```js
GameBoyAdvancePalette.prototype.store16 = function(offset, value) {
	var type = (offset & 0x200) >> 9;
	var index = (offset & 0x1FF) >> 1;
	this.colors[type][index] = value;
	this.adjustedColors[type][index] = this.adjustColor(value);
};

GameBoyAdvancePalette.prototype.store32 = function(offset, value) {
	this.store16(offset, value & 0xFFFF);
	this.store16(offset + 2, value >> 16);
};

GameBoyAdvancePalette.prototype.invalidatePage = function(address) {};

GameBoyAdvancePalette.prototype.convert16To32 = function(value, input) {
	var r = (value & 0x001F) << 3;
	var g = (value & 0x03E0) >> 2;
	var b = (value & 0x7C00) >> 7;

	input[0] = r;
	input[1] = g;
	input[2] = b;
};

```

这段代码定义了一个名为 "mix" 的函数，它接受四个参数：aWeight、aColor、bWeight 和 bColor，它们都是用于 GameBoyAdvancePalette 中的颜色数组。

函数的作用是将传入的四个参数（aColor、bColor、aWeight 和 bWeight）混合在一起，生成一个新的颜色，并返回这个新的颜色的值。

函数的实现采用了位运算和取模运算，具体解释如下：

1. 将 aColor & 0x001F 和 bColor & 0x001F 分别求反得到 aColor' 和 bColor'，这样就可以对两个参数进行按位与操作，得到一个只有透明度信息的掩码。
2. 将 aColor & 0x03E0 和 bColor & 0x7C00 分别求反得到 aColor' 和 bColor'，这样就可以对两个参数进行按位与操作，得到一个只有亮度信息的掩码。
3. 将 aWeight * ar、bWeight * br、aWeight * ag 和 bWeight * bb 分别求和，得到四个权重之和，再取最低值得 0。
4. 对步骤 3 中的四个和取反得到四个权重之和的补码，再按位或操作得到最终结果。

函数的输出结果是一个 8 位（即 00000000 - 10000000）无符号整数，表示新的颜色。这个新的颜色用于 GameBoyAdvancePalette 中 mix 属性的计算。


```js
GameBoyAdvancePalette.prototype.mix = function(aWeight, aColor, bWeight, bColor) {
	var ar = (aColor & 0x001F);
	var ag = (aColor & 0x03E0) >> 5;
	var ab = (aColor & 0x7C00) >> 10;

	var br = (bColor & 0x001F);
	var bg = (bColor & 0x03E0) >> 5;
	var bb = (bColor & 0x7C00) >> 10;

	var r = Math.min(aWeight * ar + bWeight * br, 0x1F);
	var g = Math.min(aWeight * ag + bWeight * bg, 0x1F);
	var b = Math.min(aWeight * ab + bWeight * bb, 0x1F);

	return r | (g << 5) | (b << 10);
};

```

该代码定义了GameBoyAdvancePalette的makeDarkPalettes和makeBrightPalettes方法。

makeDarkPalettes和makeBrightPalettes方法都可以接受一个layers参数，表示要绘制的图层数。在方法内部，首先检查对象实例的adjustColor属性是否与dark属性相同，如果是，则执行内部代码，否则重新设置adjustColor属性并调用resetPalettes方法。

接下来，该方法会调用resetPaletteLayers方法来设置调色板和图层。在调用makeDarkPalettes或makeBrightPalettes方法之后，会创建一个暗色调的调色板，并在所有图层上应用它。

另外，该代码还定义了一个未命名的函数adjustColorDark，该函数与makeDarkPalettes方法中的adjustColor参数重载，但不会影响该函数的名称。


```js
GameBoyAdvancePalette.prototype.makeDarkPalettes = function(layers) {
	if (this.adjustColor != this.adjustColorDark) {
		this.adjustColor = this.adjustColorDark;
		this.resetPalettes();
	}
	this.resetPaletteLayers(layers);
};

GameBoyAdvancePalette.prototype.makeBrightPalettes = function(layers) {
	if (this.adjustColor != this.adjustColorBright) {
		this.adjustColor = this.adjustColorBright;
		this.resetPalettes();
	}
	this.resetPaletteLayers(layers);
};

```

这段代码定义了 GameBoyAdvancePalette 类的两个实例方法 `makeNormalPalettes` 和 `makeSpecialPalette`。

`makeNormalPalettes` 方法使用 `this.passthroughColors` 数组，将颜色通道 0 设置为第一个颜色，然后将颜色通道 1 和 2 设置为第二个颜色，接下来将颜色通道 3 和 4 设置为第一个颜色，最后将颜色通道 5 和 6 设置为第二个颜色。这样，通过调用该方法，可以创建一个正常的颜色 palette。

`makeSpecialPalette` 方法使用 `this.passthroughColors` 数组，并将颜色通道 4 设置为要调整的颜色通道。然后，调用 `adjustedColors` 数组中相应索引处的值，将颜色通道 4 设置为预设颜色值。通过调用该方法，可以在一个给定的层创建一个特殊的颜色 palette。

注意，这两个方法后面都继承自 GameBoyAdvancePalette 类，因此都可以使用 `this.colors` 和 `this.passthroughColors` 变量。


```js
GameBoyAdvancePalette.prototype.makeNormalPalettes = function() {
	this.passthroughColors[0] = this.colors[0];
	this.passthroughColors[1] = this.colors[0];
	this.passthroughColors[2] = this.colors[0];
	this.passthroughColors[3] = this.colors[0];
	this.passthroughColors[4] = this.colors[1];
	this.passthroughColors[5] = this.colors[0];
};

GameBoyAdvancePalette.prototype.makeSpecialPalette = function(layer) {
	this.passthroughColors[layer] = this.adjustedColors[layer == 4 ? 1 : 0];
};

GameBoyAdvancePalette.prototype.makeNormalPalette = function(layer) {
	this.passthroughColors[layer] = this.colors[layer == 4 ? 1 : 0];
};

```

这段代码是用来重置GameBoyAdvancePalette中的分层颜色值的。

在GameBoyAdvancePalette.prototype.resetPaletteLayers(layers)函数中，首先检查layers中包含的层数，如果有0x01，则将这个层的颜色值赋给this.passthroughColors[0]，否则将这个层的颜色值赋给this.passthroughColors[layers & 0x01]。

然后，继续检查layers中包含的层数，如果有0x02，则将这个层的颜色值赋给this.passthroughColors[1]，否则将这个层的颜色值赋给this.passthroughColors[layers & 0x02]。

接下来，继续检查layers中包含的层数，如果有0x04，则将这个层的颜色值赋给this.passthroughColors[2]，否则将这个层的颜色值赋给this.passthroughColors[layers & 0x04]。

然后，继续检查layers中包含的层数，如果有0x08，则将这个层的颜色值赋给this.passthroughColors[3]，否则将这个层的颜色值赋给this.passthroughColors[layers & 0x08]。

接着，继续检查layers中包含的层数，如果有0x10，则将这个层的颜色值赋给this.passthroughColors[4]，否则将这个层的颜色值赋给this.passthroughColors[layers & 0x10]。

最后，继续检查layers中包含的层数，如果有0x20，则将这个层的颜色值赋给this.passthroughColors[5]，否则将这个层的颜色值赋给this.passthroughColors[layers & 0x20]。

这样，就可以将GameBoyAdvancePalette中的所有分层颜色值重置为第一层颜色值。


```js
GameBoyAdvancePalette.prototype.resetPaletteLayers = function(layers) {
	if (layers & 0x01) {
		this.passthroughColors[0] = this.adjustedColors[0];
	} else {
		this.passthroughColors[0] = this.colors[0];
	}
	if (layers & 0x02) {
		this.passthroughColors[1] = this.adjustedColors[0];
	} else {
		this.passthroughColors[1] = this.colors[0];
	}
	if (layers & 0x04) {
		this.passthroughColors[2] = this.adjustedColors[0];
	} else {
		this.passthroughColors[2] = this.colors[0];
	}
	if (layers & 0x08) {
		this.passthroughColors[3] = this.adjustedColors[0];
	} else {
		this.passthroughColors[3] = this.colors[0];
	}
	if (layers & 0x10) {
		this.passthroughColors[4] = this.adjustedColors[1];
	} else {
		this.passthroughColors[4] = this.colors[1];
	}
	if (layers & 0x20) {
		this.passthroughColors[5] = this.adjustedColors[0];
	} else {
		this.passthroughColors[5] = this.colors[0];
	}
};

```

这段代码是针对GameBoyAdvancePalette.prototype对象的一个方法，名为resetPalettes，它重置了游戏板(即GameBoy)的所有颜色分布，使得每一行、每一列、每一页(或者每一帧)的顏色都回到了初始值。

具体来说，这个方法接收一个包含两个数组的参数outPalette和inPalette。outPalette参数表示输出的颜色分布，inPalette参数表示输入的颜色分布。在方法中，首先定义了一个变量i，然后使用for循环从0到255遍历这两个数组。在循环内部，使用this.adjustedColors[i]方法调整inPalette数组中的每个颜色，然后将该颜色分配给outPalette数组中的对应索引。这样，输出的颜色分布就是根据input颜色分布进行调整后得到的颜色。

接下来，这个方法又使用另外一个for循环，从0到255再次遍历inPalette数组。不过，这次循环使用的是this.colors[i]方法获取输入颜色分布中的颜色，而不是adjustedColors[i]方法。这是因为adjustedColors方法输出的颜色分布与input颜色分布可能存在颜色空间不同的情况，而colors方法返回的是一组RGB颜色值，不会存在颜色空间混淆的情况。因此，在这个循环中，方法使用colors方法获取输入颜色分布中的每个颜色，然后将其分配给outPalette数组中的对应索引。这样，再次调整后的颜色分布就是根据input颜色分布得到的颜色。

最后，这个方法将调整后的outPalette和inPalette数组返回给调用者，使得它们可以被用来绘制游戏板上的图形。


```js
GameBoyAdvancePalette.prototype.resetPalettes = function() {
	var i;
	var outPalette = this.adjustedColors[0];
	var inPalette = this.colors[0];
	for (i = 0; i < 256; ++i) {
		outPalette[i] = this.adjustColor(inPalette[i]);
	}

	outPalette = this.adjustedColors[1];
	inPalette = this.colors[1];
	for (i = 0; i < 256; ++i) {
		outPalette[i] = this.adjustColor(inPalette[i]);
	}
}

```

这两段代码是用来在GameBoyAdvancePalette.prototype中访问颜色和调整颜色的。

第一段代码定义了一个名为accessColor的函数，它接收两个参数：一个表示层（通常为0或15，具体层定义在GameBoyAdvancePalette.layers中），和一个表示索引（0或15，具体索引定义在GameBoyAdvancePalette.layers中）的参数。函数返回该层和该索引处的颜色值。这个函数可以看做是颜色的一个副本，在没有进行任何变换的情况下，返回原始的颜色值。

第二段代码定义了一个名为adjustColorDark的函数，它接收一个颜色值（0或4096，具体颜色值定义在GameBoyAdvancePalette.color中），并返回调整后的颜色值。函数的实现主要是对输入的颜色值进行亮度调整和对比度调整。具体来说，函数会将输入颜色值的一些RGB分量减去一个灰度值（通常是50），然后将得到的结果按位或运算得到一个新的颜色值。这个函数可以看做是用来控制颜色的亮度和对比度的。


```js
GameBoyAdvancePalette.prototype.accessColor = function(layer, index) {
	return this.passthroughColors[layer][index];
};

GameBoyAdvancePalette.prototype.adjustColorDark = function(color) {
	var r = (color & 0x001F);
	var g = (color & 0x03E0) >> 5;
	var b = (color & 0x7C00) >> 10;

	r = r - (r * this.blendY);
	g = g - (g * this.blendY);
	b = b - (b * this.blendY);

	return r | (g << 5) | (b << 10);
};

```

这段代码定义了一个名为adjustColorBright的函数，它是GameBoyAdvancePalette.prototype.adjustColorBright的Prototype。

函数的作用是调整传入的彩色数组的亮度，通过计算红色、绿色和蓝色的亮度值，然后根据传入的blendY值对它们进行加权求和，并将结果二进制位化得到新的彩色数组。

此外，还定义了一个名为adjustColor的函数，与adjustColorBright非常相似，只是内部实现略有不同。

最后，定义了一个名为setBlendY的函数，用于设置混合亮度的值，当调用此函数时，会更新混合亮度的值并重新设置游戏板的背景颜色。


```js
GameBoyAdvancePalette.prototype.adjustColorBright = function(color) {
	var r = (color & 0x001F);
	var g = (color & 0x03E0) >> 5;
	var b = (color & 0x7C00) >> 10;

	r = r + ((31 - r) * this.blendY);
	g = g + ((31 - g) * this.blendY);
	b = b + ((31 - b) * this.blendY);

	return r | (g << 5) | (b << 10);
};

GameBoyAdvancePalette.prototype.adjustColor = GameBoyAdvancePalette.prototype.adjustColorBright;

GameBoyAdvancePalette.prototype.setBlendY = function(y) {
	if (this.blendY != y) {
		this.blendY = y;
		this.resetPalettes();
	}
};

```



该函数是用来创建一个GameBoy Advance OAM对象，以便在游戏内进行绘制和显示。其中，参数oam和index用于设置该对象的属性和状态。

具体来说，该函数定义了以下参数：

- tileOffset：该对象每秒旋转的 tile 数量，值为0时表示不旋转。
- x：对象在水平方向上的位置，初始值为0。
- y：对象在垂直方向上的位置，初始值为0。
- scalerot：控制对象的缩放比例，初始值为0。
- doubleSize：是否使用双重大小，初始值为false。
- disable：是否禁用对象，初始值为false。
- mode：控制对象的模式，可以是0~7，其中0表示正常模式，1表示镜像模式，2表示合成模式，3表示不变形模式，4表示反锯齿模式，5表示模糊模式，6表示边缘模式，7表示画中画模式。
- mosaic：是否使用马赛克纹理，初始值为false。
- multialpha：是否使用多透明模式，初始值为false。
- shape：控制对象的形状，可以是0~9，其中0表示默认形状，1表示矩形，2表示圆，3表示正方形，4表示三角形，5表示四面体，6表示八边形，7表示十六边形，8表示十进制多边形，9表示声音形状。
- scalerotParam：控制对象缩放的比例因子，用于控制对象的缩放。
- hflip：是否水平翻转，初始值为false。
- vflip：是否垂直翻转，初始值为false。
- tileBase：纹理基地址，初始值为0。
- priority：对象的优先级，用于控制对象在场景中的绘制顺序。
- palette：控制对象的调色板，可以是0~255。
- drawScanline：用于绘制扫描线的函数，可以通过该函数向对象绘制扫描线。
- pushPixel：用于将像素压入到纹理中的函数，可以通过该函数向对象绘制像素。
- cacheWidth和cacheHeight：分别用于控制对象纹理的宽度和高度，初始值为8。

通过这些参数的设置，可以创建一个具有不同属性的GameBoy Advance OAM对象，用于游戏内对象的绘制和显示。


```js
function GameBoyAdvanceOBJ(oam, index) {
	this.TILE_OFFSET = 0x10000;
	this.oam = oam;

	this.index = index;
	this.x = 0;
	this.y = 0;
	this.scalerot = 0;
	this.doublesize = false;
	this.disable = 1;
	this.mode = 0;
	this.mosaic = false;
	this.multipalette = false;
	this.shape = 0;
	this.scalerotParam = 0;
	this.hflip = 0;
	this.vflip = 0;
	this.tileBase = 0;
	this.priority = 0;
	this.palette = 0;
	this.drawScanline = this.drawScanlineNormal;
	this.pushPixel = GameBoyAdvanceSoftwareRenderer.pushPixel;
	this.cachedWidth = 8;
	this.cachedHeight = 8;
};

```

This is a JavaScript method for rendering a video element that uses the video2canvas library to render the video. The method takes in several parameters, such as the video element, the palette for the color scaling, and the tile size for the video. It also defines several local variables, such as the current offset for the video, the total number of tiles, and the tile offset for the tile system.

The method first sets the palette to a basic color scale and sets the tile size to 1. It then loops through each tile in the video and compares the tile offset to the current offset. If the tile offset is greater than the current offset, the method shifts the tile to the left and wraps around to the beginning of the tile offset.

The method then calculates the tile row and column based on the tile offset and local x and y. It pushes the pixel color and the offset to the video2canvas library and increments the current offset.

It also has a variable hflip which is a boolean value if the video is flipped horizontally or vertically.

It is important to note that the video2canvas library is not included in the video. You need to import it first before using the method.


```js
GameBoyAdvanceOBJ.prototype.drawScanlineNormal = function(backing, y, yOff, start, end) {
	var video = this.oam.video;
	var x;
	var underflow;
	var offset;
	var mask = this.mode | video.target2[video.LAYER_OBJ] | (this.priority << 1);
	if (this.mode == 0x10) {
		mask |= video.TARGET1_MASK;
	}
	if (video.blendMode == 1 && video.alphaEnabled) {
		mask |= video.target1[video.LAYER_OBJ];
	}

	var totalWidth = this.cachedWidth;
	if (this.x < video.HORIZONTAL_PIXELS) {
		if (this.x < start) {
			underflow = start - this.x;
			offset = start;
		} else {
			underflow = 0;
			offset = this.x;
		}
		if (end < this.cachedWidth + this.x) {
			totalWidth = end - this.x;
		}
	} else {
		underflow = start + 512 - this.x;
		offset = start;
		if (end < this.cachedWidth - underflow) {
			totalWidth = end;
		}
	}
	
	var localX;
	var localY;
	if (!this.vflip) {
		localY = y - yOff;
	} else {
		localY = this.cachedHeight - y + yOff - 1;
	}
	var localYLo = localY & 0x7;
	var mosaicX;
	var tileOffset;

	var paletteShift = this.multipalette ? 1 : 0;

	if (video.objCharacterMapping) {
		tileOffset = ((localY & 0x01F8) * this.cachedWidth) >> 6;
	} else {
		tileOffset = (localY & 0x01F8) << (2 - paletteShift);
	}

	if (this.mosaic) {
		mosaicX = video.objMosaicX - 1 - (video.objMosaicX + offset - 1) % video.objMosaicX;
		offset += mosaicX;
		underflow += mosaicX;
	}
	if (!this.hflip) {
		localX = underflow;
	} else {
		localX = this.cachedWidth - underflow - 1;
	}

	var tileRow = video.accessTile(this.TILE_OFFSET + (x & 0x4) * paletteShift, this.tileBase + (tileOffset << paletteShift) + ((localX & 0x01F8) >> (3 - paletteShift)), localYLo << paletteShift);
	for (x = underflow; x < totalWidth; ++x) {
		mosaicX = this.mosaic ? offset % video.objMosaicX : 0;
		if (!this.hflip) {
			localX = x - mosaicX;
		} else {
			localX = this.cachedWidth - (x - mosaicX) - 1;
		}
		if (!paletteShift) {
			if (!(x & 0x7) || (this.mosaic && !mosaicX)) {
				tileRow = video.accessTile(this.TILE_OFFSET, this.tileBase + tileOffset + (localX >> 3), localYLo);
			}
		} else {
			if (!(x & 0x3) || (this.mosaic && !mosaicX)) {
				tileRow = video.accessTile(this.TILE_OFFSET + (localX & 0x4), this.tileBase + (tileOffset << 1) + ((localX & 0x01F8) >> 2), localYLo << 1);
			}
		}
		this.pushPixel(video.LAYER_OBJ, this, video, tileRow, localX & 0x7, offset, backing, mask, false);
		offset++;
	}
};

```

This appears to be a JavaScript function that modifies the video sprite based on the `localX`, `localY`, `totalWidth`, `totalHeight`, `cachedWidth`, `cachedHeight`, `objMosaicX`, `objMosaicY`, `objCharacterMapping`, `paletteShift`, `paletteBase`, `totalColumns`, `totalRows`, `video.accessTile`, `video.LAYER_OBJ`, `this`, `offset`, `backing`, `mask`, `false` parameters.

It first scales the `localX` by multiplying it by the `scalerotOam.a` and adding it to the `localY` by multiplying it by the `scalerotOam.b`, and then adding the difference between the `totalWidth` and `totalHeight` to the `cachedWidth` and `cachedHeight`.

Then, it checks if the `localX` is within the range of `this.scalerotOam.a` and `this.scalerotOam.b` and if the `localY` is within the range of `this.scalerotOam.c` and `this.scalerotOam.d`. If either condition is `true`, it adjusts the position of the tile by subtracting the `x %` and `y %` from the `localX` and adding it to the `localY`, respectively.

Finally, it checks the value of `objMosaicX` and `objMosaicY` and, if `objMosaicX` is `true`, it adjusts the position of the tile based on the subtraction of `localX %` and `x %` and the multiplication of `this.scalerotOam.a` and `this.scalerotOam.b`. If `objMosaicY` is `true`, it adjusts the position of the tile based on the subtraction of `localY %` and `y %` and the multiplication of `this.scalerotOam.a` and `this.scalerotOam.b`.

It also checks if the tile is part of the `objMosaicX` and `objMosaicY` and, if it is, it assigns the appropriate offset to the `tileOffset` for the corresponding palette shift.

It is used to modify the video sprite by adjusting its position and size based on the parameters passed to it.


```js
GameBoyAdvanceOBJ.prototype.drawScanlineAffine = function(backing, y, yOff, start, end) {
	var video = this.oam.video;
	var x;
	var underflow;
	var offset;
	var mask = this.mode | video.target2[video.LAYER_OBJ] | (this.priority << 1);
	if (this.mode == 0x10) {
		mask |= video.TARGET1_MASK;
	}
	if (video.blendMode == 1 && video.alphaEnabled) {
		mask |= video.target1[video.LAYER_OBJ];
	}

	var localX;
	var localY;
	var yDiff = y - yOff;
	var tileOffset;

	var paletteShift = this.multipalette ? 1 : 0;
	var totalWidth = this.cachedWidth << this.doublesize;
	var totalHeight = this.cachedHeight << this.doublesize;
	var drawWidth = totalWidth;
	if (drawWidth > video.HORIZONTAL_PIXELS) {
		totalWidth = video.HORIZONTAL_PIXELS;
	}

	if (this.x < video.HORIZONTAL_PIXELS) {
		if (this.x < start) {
			underflow = start - this.x;
			offset = start;
		} else {
			underflow = 0;
			offset = this.x;
		}
		if (end < drawWidth + this.x) {
			drawWidth = end - this.x;
		}
	} else {
		underflow = start + 512 - this.x;
		offset = start;
		if (end < drawWidth - underflow) {
			drawWidth = end;
		}
	}

	for (x = underflow; x < drawWidth; ++x) {
		localX = this.scalerotOam.a * (x - (totalWidth >> 1)) + this.scalerotOam.b * (yDiff - (totalHeight >> 1)) + (this.cachedWidth >> 1);
		localY = this.scalerotOam.c * (x - (totalWidth >> 1)) + this.scalerotOam.d * (yDiff - (totalHeight >> 1)) + (this.cachedHeight >> 1);
		if (this.mosaic) {
			localX -= (x % video.objMosaicX) * this.scalerotOam.a + (y % video.objMosaicY) * this.scalerotOam.b;
			localY -= (x % video.objMosaicX) * this.scalerotOam.c + (y % video.objMosaicY) * this.scalerotOam.d;
		}

		if (localX < 0 || localX >= this.cachedWidth || localY < 0 || localY >= this.cachedHeight) {
			offset++;
			continue;
		}

		if (video.objCharacterMapping) {
			tileOffset = ((localY & 0x01F8) * this.cachedWidth) >> 6;
		} else {
			tileOffset = (localY & 0x01F8) << (2 - paletteShift);
		}
		tileRow = video.accessTile(this.TILE_OFFSET + (localX & 0x4) * paletteShift, this.tileBase + (tileOffset << paletteShift) + ((localX & 0x01F8) >> (3 - paletteShift)), (localY & 0x7) << paletteShift);
		this.pushPixel(video.LAYER_OBJ, this, video, tileRow, localX & 0x7, offset, backing, mask, false);
		offset++;
	}
};

```

这段代码是用来在GameBoyAdvanceOBJ.prototype中重算对象大小的函数。

首先，该函数的实现基于一个物体的形状(shape)，然后根据物体的形状设置其计算大小时使用的参数。

具体来说，当物体的形状为0时，函数将根据物体的尺寸计算出其高度和宽度，并使用一个8位二进制数存储该物体的尺寸信息，这意味着同一种形状的物体，其尺寸信息将存储在同一个位置，并且每个物体只存储一次。

当物体的形状为1时，函数将根据物体的尺寸计算出其高度和宽度，并使用一个8位二进制数存储该物体的尺寸信息。对于每种尺寸，函数都会计算出物体的高度、宽度或高度和宽度，并将它们存储在同一个位置。这意味着同一种形状的物体，其尺寸信息将存储在同一个位置，但每个物体可能会存储不同的尺寸，因此需要根据实际情况进行计算。

当物体的形状为2或3时，函数将根据物体的尺寸计算出其高度和宽度，并使用一个16位二进制数存储该物体的尺寸信息。对于每种尺寸，函数都会计算出物体的高度、宽度或高度和宽度，并将它们存储在同一个位置。这意味着同一种形状的物体，其尺寸信息将存储在同一个位置，但每个物体可能会存储不同的尺寸，因此需要根据实际情况进行计算。


```js
GameBoyAdvanceOBJ.prototype.recalcSize = function() {
	switch (this.shape) {
	case 0:
		// Square
		this.cachedHeight = this.cachedWidth = 8 << this.size;
		break;
	case 1:
		// Horizontal
		switch (this.size) {
		case 0:
			this.cachedHeight = 8;
			this.cachedWidth = 16;
			break;
		case 1:
			this.cachedHeight = 8;
			this.cachedWidth = 32;
			break;
		case 2:
			this.cachedHeight = 16;
			this.cachedWidth = 32;
			break;
		case 3:
			this.cachedHeight = 32;
			this.cachedWidth = 64;
			break;
		}
		break;
	case 2:
		// Vertical
		switch (this.size) {
		case 0:
			this.cachedHeight = 16;
			this.cachedWidth = 8;
			break;
		case 1:
			this.cachedHeight = 32;
			this.cachedWidth = 8;
			break;
		case 2:
			this.cachedHeight = 32;
			this.cachedWidth = 16;
			break;
		case 3:
			this.cachedHeight = 64;
			this.cachedWidth = 32;
			break;
		}
		break;
	default:
		// Bad!
	}
};

```

This appears to be a class in the GameBoy Advance game engine that defines an OBJ layer. The OBJ layer is used to visualize the扫描lines, scanlines, and mosaics of a GameBoy Advance game.

The OBJ layer has a priority field, which is an index into the list of OBJ layers, and an enabled field, which is a boolean value that is set to false by default and is used to determine whether to enable or disable the layer.

The OBJ layer also has a layer index field, which is an index into the list of layers in the GameBoy Advance game, and a vertical pixel offset field, which is a value that is added to the OBJ layer's vertical pixel offset if the layer is a scanline or scanline-like layer.

The OBJ layer has a drawScanline method, which is used to draw scanlines, scanlines, or mosaics onto the layer's pixel array. This method takes four parameters: the backing layer, the layer index, the start and end scanlines, and the layer's scanline or scanline-like objects.

The drawScanline method works by setting the pixel values at the specified start and end scanlines of the layer to the values of the layer's scanline or scanline-like objects, and then drawing the line from the start to the end scanlines of the layer.

The OBJ layer has a constructor method, which is used to create a new OBJ layer object, and a destructor method, which is used to destroy an OBJ layer object.


```js
function GameBoyAdvanceOBJLayer(video, index) {
	this.video = video;
	this.bg = false;
	this.index = video.LAYER_OBJ;
	this.priority = index;
	this.enabled = false;
	this.objwin = 0;
};

GameBoyAdvanceOBJLayer.prototype.drawScanline = function(backing, layer, start, end) {
	var y = this.video.vcount;
	var wrappedY;
	var mosaicY;
	var obj;
	if (start >= end) {
		return;
	}
	var objs = this.video.oam.objs;
	for (var i = 0; i < objs.length; ++i) {
		obj = objs[i];
		if (obj.disable) {
			continue;
		}
		if ((obj.mode & this.video.OBJWIN_MASK) != this.objwin) {
			continue;
		}
		if (!(obj.mode & this.video.OBJWIN_MASK) && this.priority != obj.priority) {
			continue;
		}
		if (obj.y < this.video.VERTICAL_PIXELS) {
			wrappedY = obj.y;
		} else {
			wrappedY = obj.y - 256;
		}
		var totalHeight;
		if (!obj.scalerot) {
			totalHeight = obj.cachedHeight;
		} else {
			totalHeight = obj.cachedHeight << obj.doublesize;
		}
		if (!obj.mosaic) {
			mosaicY = y;
		} else {
			mosaicY = y - y % this.video.objMosaicY;
		}
		if (wrappedY <= y && (wrappedY + totalHeight) > y) {
			obj.drawScanline(backing, mosaicY, wrappedY, start, end);
		}
	}
};

```

This appears to be a JavaScript class for drawing video backgrounds. It defines a number of properties and methods related to its functionality, such as the background mask, the layer index, and the priority of the background layer. It also defines a drawBackdrop method that appears to handle the blend and masking of the background color, as well as a number of interactivity options related to the background layer (e.g. the ability to target specific pixel color changes with the TARGET1 layer).


```js
GameBoyAdvanceOBJLayer.prototype.objComparator = function(a, b) {
	return a.index - b.index;
};

function GameBoyAdvanceSoftwareRenderer() {
	this.LAYER_BG0 = 0;
	this.LAYER_BG1 = 1;
	this.LAYER_BG2 = 2;
	this.LAYER_BG3 = 3;
	this.LAYER_OBJ = 4;
	this.LAYER_BACKDROP = 5;

	this.HORIZONTAL_PIXELS = 240;
	this.VERTICAL_PIXELS = 160;

	this.LAYER_MASK = 0x06;
	this.BACKGROUND_MASK = 0x01;
	this.TARGET2_MASK = 0x08;
	this.TARGET1_MASK = 0x10;
	this.OBJWIN_MASK = 0x20;
	this.WRITTEN_MASK = 0x80;

	this.PRIORITY_MASK = this.LAYER_MASK | this.BACKGROUND_MASK;

	this.drawBackdrop = new (function(video) {
		this.bg = true;
		this.priority = -1;
		this.index = video.LAYER_BACKDROP;
		this.enabled = true;

		this.drawScanline = function(backing, layer, start, end) {
			// TODO: interactions with blend modes and OBJWIN
			for (var x = start; x < end; ++x) {
				if (!(backing.stencil[x] & video.WRITTEN_MASK)) {
					backing.color[x] = video.palette.accessColor(this.index, 0);
					backing.stencil[x] = video.WRITTEN_MASK;
				} else if (backing.stencil[x] & video.TARGET1_MASK) {
					backing.color[x] = video.palette.mix(video.blendB, video.palette.accessColor(this.index, 0), video.blendA, backing.color[x]);
					backing.stencil[x] = video.WRITTEN_MASK;
				}
			}
		}
	})(this);
};

```

This is a JavaScript class that defines a GameBoy Advance Software renderer.

It has a `pushPixel` method that allows you to add pixels to the screen, and a `drawScanline` method that allows you to draw scanlines on the screen in different modes.

The `bgModes` array is a collection of different backgrounds that can be used to differentiate between different layers.

The `drawLayers` array is a collection of different layers that can be rendered on the screen.

The `objwinLayer` property is an instance of an object that allows you to access and draw objects that have been added to the screen.

The `drawBackdrop` property is an instance of an object that allows you to draw a backdrop that covers the entire screen.


```js
GameBoyAdvanceSoftwareRenderer.prototype.clear = function(mmu) {
	this.palette = new GameBoyAdvancePalette();
	this.vram = new GameBoyAdvanceVRAM(mmu.SIZE_VRAM);
	this.oam = new GameBoyAdvanceOAM(mmu.SIZE_OAM);
	this.oam.video = this;
	this.objLayers = [
		new GameBoyAdvanceOBJLayer(this, 0),
		new GameBoyAdvanceOBJLayer(this, 1),
		new GameBoyAdvanceOBJLayer(this, 2),
		new GameBoyAdvanceOBJLayer(this, 3)
	];
	this.objwinLayer = new GameBoyAdvanceOBJLayer(this, 4);
	this.objwinLayer.objwin = this.OBJWIN_MASK;

	// DISPCNT
	this.backgroundMode = 0;
	this.displayFrameSelect = 0;
	this.hblankIntervalFree = 0;
	this.objCharacterMapping = 0;
	this.forcedBlank = 1;
	this.win0 = 0;
	this.win1 = 0;
	this.objwin = 0;

	// VCOUNT
	this.vcount = -1;

	// WIN0H
	this.win0Left = 0;
	this.win0Right = 240;

	// WIN1H
	this.win1Left = 0;
	this.win1Right = 240;

	// WIN0V
	this.win0Top = 0;
	this.win0Bottom = 160;

	// WIN1V
	this.win1Top = 0;
	this.win1Bottom = 160;

	// WININ/WINOUT
	this.windows = new Array();
	for (var i = 0; i < 4; ++i) {
		this.windows.push({
			enabled: [ false, false, false, false, false, true ],
			special: 0
		});
	};

	// BLDCNT
	this.target1 = new Array(5);
	this.target2 = new Array(5);
	this.blendMode = 0;

	// BLDALPHA
	this.blendA = 0;
	this.blendB = 0;

	// BLDY
	this.blendY = 0;

	// MOSAIC
	this.bgMosaicX = 1;
	this.bgMosaicY = 1;
	this.objMosaicX = 1;
	this.objMosaicY = 1;

	this.lastHblank = 0;
	this.nextHblank = this.HDRAW_LENGTH;
	this.nextEvent = this.nextHblank;

	this.nextHblankIRQ = 0;
	this.nextVblankIRQ = 0;
	this.nextVcounterIRQ = 0;

	this.bg = new Array();
	for (var i = 0; i < 4; ++i) {
		this.bg.push({
			bg: true,
			index: i,
			enabled: false,
			video: this,
			vram: this.vram,
			priority: 0,
			charBase: 0,
			mosaic: false,
			multipalette: false,
			screenBase: 0,
			overflow: 0,
			size: 0,
			x: 0,
			y: 0,
			refx: 0,
			refy: 0,
			dx: 1,
			dmx: 0,
			dy: 0,
			dmy: 1,
			sx: 0,
			sy: 0,
			pushPixel: GameBoyAdvanceSoftwareRenderer.pushPixel,
			drawScanline: this.drawScanlineBGMode0
		});
	}

	this.bgModes = [
		this.drawScanlineBGMode0,
		this.drawScanlineBGMode2, // Modes 1 and 2 are identical for layers 2 and 3
		this.drawScanlineBGMode2,
		this.drawScanlineBGMode3,
		this.drawScanlineBGMode4,
		this.drawScanlineBGMode5
	];

	this.drawLayers = [
		this.bg[0],
		this.bg[1],
		this.bg[2],
		this.bg[3],
		this.objLayers[0],
		this.objLayers[1],
		this.objLayers[2],
		this.objLayers[3],
		this.objwinLayer,
		this.drawBackdrop
	];

	this,objwinActive = false;
	this.alphaEnabled = false;

	this.scanline = {
		color: new Uint16Array(this.HORIZONTAL_PIXELS),
		// Stencil format:
		// Bits 0-1: Layer
		// Bit 2: Is background
		// Bit 3: Is Target 2
		// Bit 4: Is Target 1
		// Bit 5: Is OBJ Window
		// Bit 6: Reserved
		// Bit 7: Has been written
		stencil: new Uint8Array(this.HORIZONTAL_PIXELS)
	};
	this.sharedColor = [ 0, 0, 0 ];
	this.sharedMap = {
		tile: 0,
		hflip: false,
		vflip: false,
		palette: 0
	};
};

```

这是一段用于在GameBoyAdvanceSoftwareRenderer中清除子集的JavaScript代码。

GameBoyAdvanceSoftwareRenderer是一个游戏引擎的子系统，可以用来渲染游戏画面。这个代码段是一个静态变量，存储了一个名为clearSubsets的函数。

这个函数有两个参数，一个是mmu，一个是regions。mmu是一个GameBoyAdvanceSoftwareRenderer的内存单元结构，包含了游戏画面中所有的像素数据。regions是一个32位的二进制数，包含了所有子集中的区域。

如果regions的最低位是1，那么会执行第一个if语句。在if语句中，会使用this.palette.overwrite()方法将一个Uint16Array(mmu.SIZE_PALETTE >> 1)的内存单元中的所有像素颜色值替换成一个新的颜色值，这个新的颜色值是由this.palette.getColor(0)组成的。

如果regions的最低位是2，那么会执行第二个if语句。在if语句中，会使用this.vram.insert()方法将一个Uint16Array(mmu.SIZE_VRAM >> 1)的内存单元中的所有像素颜色值替换成一个新的颜色值，这个新的颜色值是由this.vram.getColor(0)组成的。

如果regions的最低位是4，那么会执行第三个if语句。在if语句中，会使用this.oam.overwrite()方法将一个Uint16Array(mmu.SIZE_OAM >> 1)的内存单元中的所有像素颜色值替换成一个新的颜色值，这个新的颜色值是由this.oam.getColor(0)组成的。同时，还会将这个内存单元中的视频属性设置为this.oam.video，这个视频属性是一个包含视频背景颜色的Uint16Array，这个新的视频背景颜色由this.oam.getColor(10)组成。


```js
GameBoyAdvanceSoftwareRenderer.prototype.clearSubsets = function(mmu, regions) {
	if (regions & 0x04) {
		this.palette.overwrite(new Uint16Array(mmu.SIZE_PALETTE >> 1));
	}

	if (regions & 0x08) {
		this.vram.insert(0, new Uint16Array(mmu.SIZE_VRAM >> 1));
	}

	if (regions & 0x10) {
		this.oam.overwrite(new Uint16Array(mmu.SIZE_OAM >> 1));
		this.oam.video = this;
	}
};

```

这段代码是针对GameBoyAdvanceSoftwareRenderer的几个静态方法进行定义的。

freeze()方法用于冻结GameBoyAdvanceSoftwareRenderer对象，此时对象将不再响应任何事件，并且无法被操作。

defrost(frost)方法用于将frost对象中的GameBoyAdvanceSoftwareRenderer对象从冻结状态中解冻，但不会恢复原始的GameBoyAdvanceSoftwareRenderer对象。

setBacking(backing)方法用于设置或取消对象的背景（backing），背景的成分将保存到对象的一个pixelData成员中。此方法可以在游戏初始化时设置，也可以在游戏进行时进行更改。

在freeze()方法中，没有对GameBoyAdvanceSoftwareRenderer对象进行任何修改，因此不会影响其行为。在defrost()方法中，如果frost对象中包含有GameBoyAdvanceSoftwareRenderer对象，那么该对象将继续处于冻结状态，直到frost对象被清除或重新设置。在setBacking()方法中，对象将始终保存其背景，无论背景是否被设置。


```js
GameBoyAdvanceSoftwareRenderer.prototype.freeze = function() {
};

GameBoyAdvanceSoftwareRenderer.prototype.defrost = function(frost) {
};

GameBoyAdvanceSoftwareRenderer.prototype.setBacking = function(backing) {
	this.pixelData = backing;

	// Clear backing first
	for (var offset = 0; offset < this.HORIZONTAL_PIXELS * this.VERTICAL_PIXELS * 4;) {
		this.pixelData.data[offset++] = 0xFF;
		this.pixelData.data[offset++] = 0xFF;
		this.pixelData.data[offset++] = 0xFF;
		this.pixelData.data[offset++] = 0xFF;
	}
};

```

DisplayControl is a class that controls the display of a set of object layers in a attributed control. It takes a single value as an argument, which is the DisplayControl value.

The DisplayControl value is a bitwise combination of different settings that determine the color and transparency of each layer. The value is divided into four sections:

1. Background color and transparency:
	* Background mode: 0 - No background, 1 - With a transparent background, 2 - With a gradient background
	* Display frame select: 0 - Display all layers, 1 - Display only layer 1, 2 - Display only layer 2, 3 - Display all layers
	* HBlank interval: 0 - Disable, 1 - Enable, 2 - Enable for a limited time
	* Object character mapping: 0 - Do not use object character mapping, 1 - Use object character mapping
	* Forced blank: 0 - Do not have a blank, 1 - Yes, 2 - Yes, but reset the layer
	* BG color and transparency: 0 - 0x0001 for a transparent, 0x0002 for a gradient
	* Win0, Win1, and objwin: 0 - 0x2000, 0x4000, and 0x8000, respectively
	* Objwin layer enable: 0 - Only the left layer, 1 - Both left and right layers, or 2 - All layers
2. Object layers enable/disable:
	* layer 0: 0 - Disable, 1 - Enable, 2 - Enable for a limited time
	* layer 1: 0 - Disable, 1 - Enable, 2 - Enable for a limited time
	* layer 2: 0 - Disable, 1 - Enable, 2 - Enable for a limited time
	* layer 3: 0 - Disable, 1 - Enable, 2 - Enable for a limited time
	* Win0, Win1, and objwin: 0 - 0x2000, 0x4000, and 0x8000, respectively
3. Display attributes:
	* Win0: 0 - 0x2000, 0x4000, and 0x8000, respectively
	* Win1: 0 - 0x2000, 0x4000, and 0x8000, respectively
	* Objwin: 0 - 0x1000 and 0x8000, respectively

The DisplayControl value is combined from these settings, with the win0, win1, and objwin settings having the highest priority.


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeDisplayControl = function(value) {
	this.backgroundMode = value & 0x0007;
	this.displayFrameSelect = value & 0x0010;
	this.hblankIntervalFree = value & 0x0020;
	this.objCharacterMapping = value & 0x0040;
	this.forcedBlank = value & 0x0080;
	this.bg[0].enabled = value & 0x0100;
	this.bg[1].enabled = value & 0x0200;
	this.bg[2].enabled = value & 0x0400;
	this.bg[3].enabled = value & 0x0800;
	this.objLayers[0].enabled = value & 0x1000;
	this.objLayers[1].enabled = value & 0x1000;
	this.objLayers[2].enabled = value & 0x1000;
	this.objLayers[3].enabled = value & 0x1000;
	this.win0 = value & 0x2000;
	this.win1 = value & 0x4000;
	this.objwin = value & 0x8000;
	this.objwinLayer.enabled = value & 0x1000 && value & 0x8000;

	// Total hack so we can store both things that would set it to 256-color mode in the same variable
	this.bg[2].multipalette &= ~0x0001;
	this.bg[3].multipalette &= ~0x0001;
	if (this.backgroundMode > 0) {
		this.bg[2].multipalette |= 0x0001;
	}
	if (this.backgroundMode == 2) {
		this.bg[3].multipalette |= 0x0001;
	}
	
	this.resetLayers();
};

```

这段代码是JavaScript中的一个函数，属于GameBoyAdvanceSoftwareRenderer.prototype的对象。这个函数的主要作用是在游戏背景上设置一个文本框或者一个背景图片，通过传入要设置的背景颜色（或者灰度值）和文本框（或者背景图片）的索引（或者像素位置），来修改游戏背景的显示效果。

具体来说，这段代码需要从GameBoyAdvanceSoftwareRenderer.prototype对象中获取一个背景变量（bg变量），然后设置文本框或者背景图片的索引值，修改背景的显示效果，最后调用drawLayer函数来绘制设置好的背景。其中，bg变量可以是一个数组，每个元素代表一个文本框或者背景图片，每个元素都有一个priority值来表示优先级，控制背景颜色的透明度。另外，如果文本框（或者背景图片）的索引值（或者像素位置）超出了游戏背景的2层或者当前的背景模式为0，那么还需要对背景进行扩展，将其multipalette属性中的值进行相应的调整。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundControl = function(bg, value) {
	var bgData = this.bg[bg];
	bgData.priority = value & 0x0003;
	bgData.charBase = (value & 0x000C) << 12;
	bgData.mosaic = value & 0x0040;
	bgData.multipalette &= ~0x0080;
	if (bg < 2 || this.backgroundMode == 0) {
		bgData.multipalette |= value & 0x0080;
	}
	bgData.screenBase = (value & 0x1F00) << 3;
	bgData.overflow = value & 0x2000;
	bgData.size = (value & 0xC000) >> 14;

	this.drawLayers.sort(this.layerComparator);
};

```

这段代码是针对GameBoyAdvanceSoftwareRenderer.prototype中的三个writeBackground*函数进行解释。

writeBackgroundHOffset函数接受两个参数，一个是背景（bg）参数，另一个是偏移量（value）参数。它将偏移量（value）和0x1FF&（与value的最高位相同为1）进行与运算，然后将结果赋值给this.bg[bg].x属性。

writeBackgroundVOffset函数与writeBackgroundHOffset类似，但它处理的是垂直方向上的偏移量（value）。它将偏移量（value）和0x1FF&进行与运算，然后将结果赋值给this.bg[bg].y属性。

writeBackgroundRefX函数与writeBackgroundVOffset类似，但它处理的是水平方向上的偏移量（value）。它将偏移量（value）和0x1FF&进行与运算，然后将结果赋值给this.bg[bg].refx属性，同时将sx属性与refx属性进行与运算。

writeBackgroundRefY函数与writeBackgroundVOffset类似，但它处理的是垂直方向上的偏移量（value）。它将偏移量（value）和0x1FF&进行与运算，然后将结果赋值给this.bg[bg].refy属性，同时将sy属性与refy属性进行与运算。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundHOffset = function(bg, value) {
	this.bg[bg].x = value & 0x1FF;
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundVOffset = function(bg, value) {
	this.bg[bg].y = value & 0x1FF;
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundRefX = function(bg, value) {
	this.bg[bg].refx = (value << 4) / 0x1000;
	this.bg[bg].sx = this.bg[bg].refx;
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundRefY = function(bg, value) {
	this.bg[bg].refy = (value << 4) / 0x1000;
	this.bg[bg].sy = this.bg[bg].refy;
};

```

这段代码是GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundParamX函数，用于在背景参数中设置四个不同的参数(四个亮度)，每个参数的二进制形式都被存储在一个16位的整数中，然后将其值转换为小数，以便在屏幕上正确显示亮度。

具体来说，writeBackgroundParamX函数接收两个参数，第一个参数bg是一个8位的整数，表示背景参数(0-F)，第二个参数value是一个16位的整数，表示每个参数的二进制形式。函数内部使用((value << 16) / 0x1000000)计算出每个参数的小数形式，然后将其存储在bg中对应的偏移位置上，最终在屏幕上正确显示亮度。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundParamA = function(bg, value) {
	this.bg[bg].dx = (value << 16) / 0x1000000;
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundParamB = function(bg, value) {
	this.bg[bg].dmx = (value << 16) / 0x1000000;
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundParamC = function(bg, value) {
	this.bg[bg].dy = (value << 16) / 0x1000000;
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBackgroundParamD = function(bg, value) {
	this.bg[bg].dmy = (value << 16) / 0x1000000;
};

```

这两函数是用来在GameBoyAdvanceSoftwareRenderer中根据不同的LCM值设置显示的左上角和右上角的像素值。LCM值（Link Cell Multi- color Terminal）是一种用于显示颜色的分层系统，每层像素都有8个灰度值。

在这两个函数中，函数名中的数字代表左上角（或右上角）的像素值，而函数值中的数字代表左上角（或右上角）的LCM值。通过计算得到该像素值在LCM值范围内的最小值，然后将其赋值给变量。

如果计算得到的像素值比该LCM值更大，那么将该LCM值设为其计算结果，否则将该LCM值设为其对应的像素值。

例如，如果LCM值为255，那么win0H和win1H分别将像素值0x255 >> 8和0x255 >> 8赋值给this.win0Left和this.win1Right变量。

这两个函数的作用是确保在游戏机的屏幕上，游戏按照规范的LCM值进行显示，即使游戏中的某些元素超出了游戏板能够显示的范围，例如透明度或背景颜色等。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeWin0H = function(value) {
	this.win0Left = (value & 0xFF00) >> 8;
	this.win0Right = Math.min(this.HORIZONTAL_PIXELS, value & 0x00FF);
	if (this.win0Left > this.win0Right) {
		this.win0Right = this.HORIZONTAL_PIXELS;
	}
};

GameBoyAdvanceSoftwareRenderer.prototype.writeWin1H = function(value) {
	this.win1Left = (value & 0xFF00) >> 8;
	this.win1Right = Math.min(this.HORIZONTAL_PIXELS, value & 0x00FF);
	if (this.win1Left > this.win1Right) {
		this.win1Right = this.HORIZONTAL_PIXELS;
	}
};

```

这段代码是定义了 GameBoyAdvanceSoftwareRenderer.prototype.writeWin0V 和 GameBoyAdvanceSoftwareRenderer.prototype.writeWin1V 两个函数，用于将值转换为垂直像素数并输出到屏幕上。

具体来说，writeWin0V 和 writeWin1V 函数分别接受一个整数参数 value，然后将其按位与 0xFF00 并右移 8 位，得到一个 8 位无符号整数。然后，函数会根据这个无符号整数的最高位来决定是否将值保留到垂直像素数中。如果最高位是 0，那么函数会将值直接输出，否则会将值转换为垂直像素数并输出。

这里，GameBoyAdvanceSoftwareRenderer.prototype.writeWin0V 和 GameBoyAdvanceSoftwareRenderer.prototype.writeWin1V 函数的实现主要依赖于他们在屏幕上输出像素的机制。在 GameBoyAdvanceSoftwareRenderer 中，像素被输出到屏幕上的方式是通过响应用户的 drawRequest 事件，发送像素数据到屏幕上的像素泵。因此，writeWin0V 和 writeWin1V 函数的作用就是在像素泵中输出值，而不是在屏幕上显示像素。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeWin0V = function(value) {
	this.win0Top = (value & 0xFF00) >> 8;
	this.win0Bottom = Math.min(this.VERTICAL_PIXELS, value & 0x00FF);
	if (this.win0Top > this.win0Bottom) {
		this.win0Bottom = this.VERTICAL_PIXELS;
	}
};

GameBoyAdvanceSoftwareRenderer.prototype.writeWin1V = function(value) {
	this.win1Top = (value & 0xFF00) >> 8;
	this.win1Bottom = Math.min(this.VERTICAL_PIXELS, value & 0x00FF);
	if (this.win1Top > this.win1Bottom) {
		this.win1Bottom = this.VERTICAL_PIXELS;
	}
};

```

该代码是一个JavaScript接口，描述了如何写入到一个GameBoyAdvanceSoftwareRenderer的游戏窗口中。

具体来说，writeWindow函数接收一个整数参数index和两个整数参数value，然后将其按位与后赋值给窗口对象的相应位，最后将结果返回。writeWinIn函数则是writeWindow函数的别称，只需要传递一个整数参数value即可。

通过编写游戏窗口的enabled位来实现控制游戏窗口的显示状态。当value的最低位为1时，窗口将启用第一行，值为2时，启用第二行，值为4时，启用第三行，值为8时，启用第四行，值为10时，启用特殊的图形显示，值为16时，将当前窗口设置为640x480的分辨率。value的更高位则用于设置窗口的特殊功能，如亮度、对比度和特殊的透明模式等。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeWindow = function(index, value) {
	var window = this.windows[index];
	window.enabled[0] = value & 0x01;
	window.enabled[1] = value & 0x02;
	window.enabled[2] = value & 0x04;
	window.enabled[3] = value & 0x08;
	window.enabled[4] = value & 0x10;
	window.special = value & 0x20;
};

GameBoyAdvanceSoftwareRenderer.prototype.writeWinIn = function(value) {
	this.writeWindow(0, value);
	this.writeWindow(1, value >> 8);
};

```

This appears to be a JavaScript function that takes a single parameter `value` of type `null` or `undefined`. It then performs a blending operation based on the value and the blend mode set by the user.

The function has several properties on which it appears to take input, such as `this.target1` and `this.target2`, which appear to be variables representing different parts of the image, and `this.blendMode`, which appears to be a combination of a blend mode and a set of blend rules.

The function performs a series of loops, each of which performs a different type of blending operation using the values specified by the user. The blend mode can take one of four values: 0 (normal), 1 (alpha), 2 (brighter), or 3 (darker).


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeWinOut = function(value) {
	this.writeWindow(2, value);
	this.writeWindow(3, value >> 8);
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBlendControl = function(value) {
	this.target1[0] = !!(value & 0x0001) * this.TARGET1_MASK;
	this.target1[1] = !!(value & 0x0002) * this.TARGET1_MASK;
	this.target1[2] = !!(value & 0x0004) * this.TARGET1_MASK;
	this.target1[3] = !!(value & 0x0008) * this.TARGET1_MASK;
	this.target1[4] = !!(value & 0x0010) * this.TARGET1_MASK;
	this.target1[5] = !!(value & 0x0020) * this.TARGET1_MASK;
	this.target2[0] = !!(value & 0x0100) * this.TARGET2_MASK;
	this.target2[1] = !!(value & 0x0200) * this.TARGET2_MASK;
	this.target2[2] = !!(value & 0x0400) * this.TARGET2_MASK;
	this.target2[3] = !!(value & 0x0800) * this.TARGET2_MASK;
	this.target2[4] = !!(value & 0x1000) * this.TARGET2_MASK;
	this.target2[5] = !!(value & 0x2000) * this.TARGET2_MASK;
	this.blendMode = (value & 0x00C0) >> 6;

	switch (this.blendMode) {
	case 1:
		// Alpha
		// Fall through
	case 0:
		// Normal
		this.palette.makeNormalPalettes();
		break;
	case 2:
		// Brighter
		this.palette.makeBrightPalettes(value & 0x3F);
		break;
	case 3:
		// Darker
		this.palette.makeDarkPalettes(value & 0x3F);
		break;
	}
};

```

该代码是自定义了 GameBoyAdvanceSoftwareRenderer.prototype.setBlendEnabled 方法，用于设置层在透明度和透明模式下的颜色。

首先，该方法接受三个参数：layer、enabled 和 override。其中，layer 代表要设置 blend 属性的层，enabled 表示是否启用混合模式，而 override 则是一个布尔值，表示是否覆盖默认设置。

在 if 语句中，如果enabled为真，则执行第一個 if 语句块。在 if 语句块中，首先检查 enabled 是否为真，如果是，则执行一系列的 switch 语句。在 switch 语句中，定义了四种不同的混合模式，分别是在不启用混合模式、启用alpha混合模式、启用正常混合模式和启用 brighter/darker 混合模式下的设置。在每种混合模式下，使用 this.palette.makeNormalPalette(layer) 或 this.palette.makeSpecialPalette(layer) 来设置底层像素的颜色。

如果enabled为假，则执行 if 语句块的 else 部分。在这里，使用 this.palette.makeNormalPalette(layer) 来设置底层像素的颜色，其中layer代表层的索引。


```js
GameBoyAdvanceSoftwareRenderer.prototype.setBlendEnabled = function(layer, enabled, override) {
	this.alphaEnabled = enabled && override == 1;
	if (enabled) {
		switch (override) {
		case 1:
			// Alpha
			// Fall through
		case 0:
			// Normal
			this.palette.makeNormalPalette(layer);
			break;
		case 2:
			// Brighter
		case 3:
			// Darker
			this.palette.makeSpecialPalette(layer);
			break;
		}
	} else {
		this.palette.makeNormalPalette(layer);
	}
};

```

这两段代码是针对GameBoyAdvanceSoftwareRendererprototype中的writeBlendAlpha和writeBlendY方法进行的。

writeBlendAlpha方法的目的是设置渲染器中精灵的透明度。它接受一个值，该值范围从0到255，表示透明度。这个值被转换成8位无符号整数，并将其与0x1F00进行按位与操作，然后将其除以16得到一个0到255之间的值，用于设置透明度。如果生成的值超过1，则将其设为1，否则保留原始值。

writeBlendY方法的目的是设置渲染器中精灵的亮度。它接受一个值，表示透明度。这个值的范围从0到15，从0开始表示完全透明，从16开始表示完全不透明。它被转换成8位无符号整数，并将其与16除以16得到一个0到15之间的值，用于设置透明度。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeBlendAlpha = function(value) {
	this.blendA = (value & 0x001F) / 16;
	if (this.blendA > 1) {
		this.blendA = 1;
	}
	this.blendB = ((value & 0x1F00) >> 8) / 16;
	if (this.blendB > 1) {
		this.blendB = 1;
	}
};

GameBoyAdvanceSoftwareRenderer.prototype.writeBlendY = function(value) {
	this.blendY = value;
	this.palette.setBlendY(value >= 16 ? 1 : (value / 16));
};

```

该代码是GameBoyAdvanceSoftwareRenderer.prototype.writeMosaic函数和resetLayers函数的定义。

writeMosaic函数的作用是创建一个分频网格，将游戏机屏幕的扫描线数据编码到编码器中。通过传入一个值，可以控制扫描线的密度、颜色和透明度。具体来说，这个值二进制形式存储在从0到F-1517(即00010111)，其中最高位表示扫描线的颜色和透明度，其次是扫描线的数量，即4位二进制数，然后是扫描线的扫描方向和宽度，即8位二进制数。

resetLayers函数的作用是重置场景中所有图层的绘制设置。具体来说，如果当前的游戏机模式大于1，则禁用背景层和第一个图层的绘制，如果当前的游戏机模式为0或2，则禁用第二个图层的绘制。如果当前的游戏机模式为0，则使用第一个图层的绘制模式，并将第二个图层的扫描线设置为当前的游戏机模式。如果当前的游戏机模式为2，则使用第二个图层的绘制模式，并将扫描线设置为当前的游戏机模式。最后，调用layerComparator函数对层进行排序，以便正确地渲染它们。


```js
GameBoyAdvanceSoftwareRenderer.prototype.writeMosaic = function(value) {
	this.bgMosaicX = (value & 0xF) + 1;
	this.bgMosaicY = ((value >> 4) & 0xF) + 1;
	this.objMosaicX = ((value >> 8) & 0xF) + 1;
	this.objMosaicY = ((value >> 12) & 0xF) + 1;
};

GameBoyAdvanceSoftwareRenderer.prototype.resetLayers = function() {
	if (this.backgroundMode > 1) {
		this.bg[0].enabled = false;
		this.bg[1].enabled = false;
	}
	if (this.bg[2].enabled) {
		this.bg[2].drawScanline = this.bgModes[this.backgroundMode];
	}
	if ((this.backgroundMode == 0 || this.backgroundMode == 2)) {
		if (this.bg[3].enabled) {
			this.bg[3].drawScanline = this.bgModes[this.backgroundMode];
		}
	} else {
		this.bg[3].enabled = false;
	}
	this.drawLayers.sort(this.layerComparator);
};

```

这段代码定义了 GameBoyAdvanceSoftwareRenderer.prototype.layerComparator 函数，用于比较两个不同层级的像素的优先级。

该函数有两个参数，分别是 a 和 b，它们都代表一个像素对象。函数首先计算两个像素对象的优先级差异，如果没有差异，则返回 -1，否则返回 b.index - a.index。

接下来定义了另一个函数 GameBoyAdvanceSoftwareRenderer.prototype.accessMapMode0，该函数接收一个四元组 base、size、x 和 yBase，以及一个输出对象 out。函数首先计算从 base 到 (x >> 2) & 0x3E) 的偏移量 offset，然后计算一个 16 字节的内存区域，并将 offset 和 (x & 0x100) << 3 和 yBase 相加。

最后，将计算得到的内存区域中的值输出给 out.tile、out.hflip 和 out.vflip 属性，同时将 palette 属性设置为 (mem & 0xF000) >> 8 的值，以便在 pushPixel 函数中使用。


```js
GameBoyAdvanceSoftwareRenderer.prototype.layerComparator = function(a, b) {
	var diff = b.priority - a.priority;
	if (!diff) {
		if (a.bg && !b.bg) {
			return -1;
		} else if (!a.bg && b.bg) {
			return 1;
		}

		return b.index - a.index;
	}
	return diff;
};

GameBoyAdvanceSoftwareRenderer.prototype.accessMapMode0 = function(base, size, x, yBase, out) {
	var offset = base + ((x >> 2) & 0x3E) + yBase;

	if (size & 1) {
		offset += (x & 0x100) << 3;
	}

	var mem = this.vram.loadU16(offset);
	out.tile = mem & 0x03FF;
	out.hflip = mem & 0x0400;
	out.vflip = mem & 0x0800;
	out.palette = (mem & 0xF000) >> 8 // This is shifted up 4 to make pushPixel faster
};

```

This appears to be a function written in the OpenGL Extension Wrangler Library (GLEW) that handles the stencil object in a multi-pass render pipeline.

It appears to be selecting the appropriate combination of data from the input data (oldStencil) to determine the blend that should be applied to the pixel, based on the input priority mask.

The function first checks if the pixel is already in the stencil, and if it is, it returns without drawing it again.

If the pixel is not in the stencil, it then checks the input priority mask to determine which blend to apply. If it is a blend target, it checks if it is the blend target for Target 1 or Target 2. If it is not a blend target, it then checks if it is the blend target for Target 1 or Target 2 or it is the Object Win layer.

If the pixel is a blend target, it then checks if it is blend A or blend B. If it is blend A, it gets the blend color from the palette and mixes it with the pixel color to get the final blend. If it is blend B, it sets the blend to the pixel color.

If the function is not the blend target for any of the layers, it does nothing and returns.


```js
GameBoyAdvanceSoftwareRenderer.prototype.accessMapMode1 = function(base, size, x, yBase, out) {
	var offset = base + (x >> 3) + yBase;

	out.tile = this.vram.loadU8(offset);
};

GameBoyAdvanceSoftwareRenderer.prototype.accessTile = function(base, tile, y) {
	var offset = base + (tile << 5);
	offset |= y << 2;

	return this.vram.load32(offset);
}

GameBoyAdvanceSoftwareRenderer.pushPixel = function(layer, map, video, row, x, offset, backing, mask, raw) {
	var index;
	if (!raw) {
		if (this.multipalette) {
			index = (row >> (x << 3)) & 0xFF;
		} else {
			index = (row >> (x << 2)) & 0xF;
		}
		// Index 0 is transparent
		if (!index) {
			return;
		} else if (!this.multipalette) {
			index |= map.palette;
		}
	}

	var stencil = video.WRITTEN_MASK;
	var oldStencil = backing.stencil[offset];
	var blend = video.blendMode;
	if (video.objwinActive) {
		if (oldStencil & video.OBJWIN_MASK) {
			if (video.windows[3].enabled[layer]) {
				video.setBlendEnabled(layer, video.windows[3].special && video.target1[layer], blend);
				if (video.windows[3].special && video.alphaEnabled) {
					mask |= video.target1[layer];
				}
				stencil |= video.OBJWIN_MASK;
			} else {
				return;
			}
		} else if (video.windows[2].enabled[layer]) {
			video.setBlendEnabled(layer, video.windows[2].special && video.target1[layer], blend);
			if (video.windows[2].special && video.alphaEnabled) {
				mask |= video.target1[layer];
			}
		} else {
			return;
		}
	}

	if ((mask & video.TARGET1_MASK) && (oldStencil & video.TARGET2_MASK)) {
		video.setBlendEnabled(layer, true, 1);
	}

	var pixel = raw ? row : video.palette.accessColor(layer, index);

	if (mask & video.TARGET1_MASK) {
		video.setBlendEnabled(layer, !!blend, blend);
	}
	var highPriority = (mask & video.PRIORITY_MASK) < (oldStencil & video.PRIORITY_MASK);
	// Backgrounds can draw over each other, too.
	if ((mask & video.PRIORITY_MASK) == (oldStencil & video.PRIORITY_MASK)) {
		highPriority = mask & video.BACKGROUND_MASK;
	}

	if (!(oldStencil & video.WRITTEN_MASK)) {
		// Nothing here yet, just continue
		stencil |= mask;
	} else if (highPriority) {
		// We are higher priority
		if (mask & video.TARGET1_MASK && oldStencil & video.TARGET2_MASK) {
			pixel = video.palette.mix(video.blendA, pixel, video.blendB, backing.color[offset]);
		}
		// We just drew over something, so it doesn't make sense for us to be a TARGET1 anymore...
		stencil |= mask & ~video.TARGET1_MASK;
	} else if ((mask & video.PRIORITY_MASK) > (oldStencil & video.PRIORITY_MASK)) {
		// We're below another layer, but might be the blend target for it
		stencil = oldStencil & ~(video.TARGET1_MASK | video.TARGET2_MASK);
		if (mask & video.TARGET2_MASK && oldStencil & video.TARGET1_MASK) {
			pixel = video.palette.mix(video.blendB, pixel, video.blendA, backing.color[offset]);
		} else {
			return;
		}
	} else {
		return;
	}

	if (mask & video.OBJWIN_MASK) {
		// We ARE the object window, don't draw pixels!
		backing.stencil[offset] |= video.OBJWIN_MASK;
		return;
	}
	backing.color[offset] = pixel;
	backing.stencil[offset] = stencil;
};

```

该代码是一个JavaScript类的实例，它是GameBoyAdvanceSoftwareRenderer.prototype.identity的定义。

identity的作用是返回输入的参数x，并将其作为原始值返回，不会进行任何修改。

drawScanlineBlank的作用是在GameBoyAdvanceSoftwareRenderer.prototype.drawScanline的回调函数中执行。它遍历了模型和屏幕上的所有像素，并将它们设置为透明的颜色（0xFFFF），以使扫描线区域在背景中显示。它还设置了一个stencil（淀粉）数组，其中包含绘图阶段所需的纹理。

prepareScanline的作用是在GameBoyAdvanceSoftwareRenderer.prototype.drawScanline的回调函数中执行。它遍历了模型和屏幕上的所有像素，并将它们设置为屏幕背景的纹理（此处的LAYER_BACKDROP），以使扫描线区域在背景中显示。它还设置了一个stencil（淀粉）数组，其中包含绘图阶段所需的纹理。


```js
GameBoyAdvanceSoftwareRenderer.prototype.identity = function(x) {
	return x;
};

GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBlank = function(backing) {
	for (var x = 0; x < this.HORIZONTAL_PIXELS; ++x) {
		backing.color[x] = 0xFFFF;
		backing.stencil[x] = 0;
	}
};

GameBoyAdvanceSoftwareRenderer.prototype.prepareScanline = function(backing) {
	for (var x = 0; x < this.HORIZONTAL_PIXELS; ++x) {
		backing.stencil[x] = this.target2[this.LAYER_BACKDROP];
	}
};

```

This appears to be a JavaScript function that renders a video background, which is made up of multiple tile maps that are repeated to fill the entire screen. The function takes in a video object and a tile map object, and uses the tile map object to determine which tile to display at any given time.

It starts by setting the local x-coordinate to zero and the local y-coordinate to the negative value of the tile map's y-coordinate, which is the baseline position of the tile.

Then it sets the tile offset based on the tile position of the tile map, and the tile rotation of the tile.

It then checks if the tile offset is within the boundaries of the tile map, and if it's not it retrieves the tile at that offset and displays it.

It also checks if the tile offset is within the boundaries of the tile map and displays it in the correct flipping mode.

Finally, it pushes the pixel data for the tile at the specified offset, and increments the offset for the next iteration of the loop.


```js
GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBGMode0 = function(backing, bg, start, end) {
	var video = this.video;
	var x;
	var y = video.vcount;
	var offset = start;
	var xOff = bg.x;
	var yOff = bg.y;
	var localX;
	var localXLo;
	var localY = y + yOff;
	if (this.mosaic) {
		localY -= y % video.bgMosaicY;
	}
	var localYLo = localY & 0x7;
	var mosaicX;
	var screenBase = bg.screenBase;
	var charBase = bg.charBase;
	var size = bg.size;
	var index = bg.index;
	var map = video.sharedMap;
	var paletteShift = bg.multipalette ? 1 : 0;
	var mask = video.target2[index] | (bg.priority << 1) | video.BACKGROUND_MASK;
	if (video.blendMode == 1 && video.alphaEnabled) {
		mask |= video.target1[index];
	}

	var yBase = (localY << 3) & 0x7C0;
	if (size == 2) {
		yBase += (localY << 3) & 0x800;
	} else if (size == 3) {
		yBase += (localY << 4) & 0x1000;
	}

	var xMask;
	if (size & 1) {
		xMask = 0x1FF;
	} else {
		xMask = 0xFF;
	}

	video.accessMapMode0(screenBase, size, (start + xOff) & xMask, yBase, map);
	var tileRow = video.accessTile(charBase, map.tile << paletteShift, (!map.vflip ? localYLo : 7 - localYLo) << paletteShift);
	for (x = start; x < end; ++x) {
		localX = (x + xOff) & xMask;
		mosaicX = this.mosaic ? offset % video.bgMosaicX : 0;
		localX -= mosaicX;
		localXLo = localX & 0x7;
		if (!paletteShift) {
			if (!localXLo || (this.mosaic && !mosaicX)) {
				video.accessMapMode0(screenBase, size, localX, yBase, map);
				tileRow = video.accessTile(charBase, map.tile, !map.vflip ? localYLo : 7 - localYLo);
				if (!tileRow && !localXLo) {
					x += 7;
					offset += 8;
					continue;
				}
			}
		} else {
			if (!localXLo || (this.mosaic && !mosaicX)) {
				video.accessMapMode0(screenBase, size, localX, yBase, map);
			}
			if (!(localXLo & 0x3) || (this.mosaic && !mosaicX)) {
				tileRow = video.accessTile(charBase + (!!(localX & 0x4) == !map.hflip ? 4 : 0), map.tile << 1, (!map.vflip ? localYLo : 7 - localYLo) << 1);
				if (!tileRow && !(localXLo & 0x3)) {
					x += 3;
					offset += 4;
					continue;
				}
			}
		}
		if (map.hflip) {
			localXLo = 7 - localXLo;
		}
		bg.pushPixel(index, map, video, tileRow, localXLo, offset, backing, mask, false);
		offset++;
	}
};

```

这段代码是一个在给定的视频中根据偏移量、背景颜色和混合模式等因素来对视频进行处理的过程。处理后的结果将会显示在屏幕上。

具体来说，代码首先定义了一个变量`mask`，它包含了视频的背景颜色、目标1和目标2的值，以及一个掩码，用于隐藏背景中某些像素。然后，代码判断了`blendMode`的值是否为1，如果是，就说明代码需要对背景进行颜色调整。接下来，代码遍历了给定的视频区域，对每个像素的RGBA值进行了处理，具体来说，它计算了每个像素与背景颜色以及目标1的差值，然后将这些差值与掩码和背景颜色进行与运算，最后将处理后的像素放入最终的缓冲区中。




```js
GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBGMode2 = function(backing, bg, start, end) {
	var video = this.video;
	var x;
	var y = video.vcount;
	var offset = start;
	var localX;
	var localY;
	var screenBase = bg.screenBase;
	var charBase = bg.charBase;
	var size = bg.size;
	var sizeAdjusted = 128 << size;
	var index = bg.index;
	var map = video.sharedMap;
	var color;
	var mask = video.target2[index] | (bg.priority << 1) | video.BACKGROUND_MASK;
	if (video.blendMode == 1 && video.alphaEnabled) {
		mask |= video.target1[index];
	}

	var yBase;

	for (x = start; x < end; ++x) {
		localX = bg.dx * x + bg.sx;
		localY = bg.dy * x + bg.sy;
		if (this.mosaic) {
			localX -= (x % video.bgMosaicX) * bg.dx + (y % video.bgMosaicY) * bg.dmx;
			localY -= (x % video.bgMosaicX) * bg.dy + (y % video.bgMosaicY) * bg.dmy;
		}
		if (bg.overflow) {
			localX &= sizeAdjusted - 1;
			if (localX < 0) {
				localX += sizeAdjusted;
			}
			localY &= sizeAdjusted - 1;
			if (localY < 0) {
				localY += sizeAdjusted;
			}
		} else if (localX < 0 || localY < 0 || localX >= sizeAdjusted || localY >= sizeAdjusted) {
			offset++;
			continue;
		}
		yBase = ((localY << 1) & 0x7F0) << size;
		video.accessMapMode1(screenBase, size, localX, yBase, map);
		color = this.vram.loadU8(charBase + (map.tile << 6) + ((localY & 0x7) << 3) + (localX & 0x7));
		bg.pushPixel(index, map, video, color, 0, offset, backing, mask, false);
		offset++;
	}
};

```

这段代码是GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBGmode3方法，用于在GameBoy游戏机上绘制扫描线背景模式。

该方法接收4个参数：

- backing(Backing)：背景颜色，以像素为单位。
- bg(Background)：要绘制的背景的GameBoy索引。
- start(Start)：开始绘制的位置，以像素为单位。
- end(End)：结束绘制的位置，以像素为单位。

方法内部首先定义了一些变量，包括x,y,offset,localX,localY,index,map,color,mask等，然后进行循环操作，从start位置开始，以end位置结束绘制 scanline，绘制过程中，使用到了bgMosaicX和bgMosaicY属性，根据bg的priority值来判断是否使用到了target1或target2，然后根据localX和localY计算出绘制位置，再从vram中读取对应颜色的16位无符号整数，最后将像素加入到了bg中，并且使用到mask变量，最终输出结果。


```js
GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBGMode3 = function(backing, bg, start, end) {
	var video = this.video;
	var x;
	var y = video.vcount;
	var offset = start;
	var localX;
	var localY;
	var index = bg.index;
	var map = video.sharedMap;
	var color;
	var mask = video.target2[index] | (bg.priority << 1) | video.BACKGROUND_MASK;
	if (video.blendMode == 1 && video.alphaEnabled) {
		mask |= video.target1[index];
	}

	var yBase;

	for (x = start; x < end; ++x) {
		localX = bg.dx * x + bg.sx;
		localY = bg.dy * x + bg.sy;
		if (this.mosaic) {
			localX -= (x % video.bgMosaicX) * bg.dx + (y % video.bgMosaicY) * bg.dmx;
			localY -= (x % video.bgMosaicX) * bg.dy + (y % video.bgMosaicY) * bg.dmy;
		}
		if (localX < 0 || localY < 0 || localX >= video.HORIZONTAL_PIXELS || localY >= video.VERTICAL_PIXELS) {
			offset++;
			continue;
		}
		color = this.vram.loadU16(((localY * video.HORIZONTAL_PIXELS) + localX) << 1);
		bg.pushPixel(index, map, video, color, 0, offset, backing, mask, true);
		offset++;
	}
};

```

This is a JavaScript definition for a class called `VideoObject`. The `VideoObject` class appears to represent a video that can be displayed on a website. It has properties for the video's position, dimensions, and pixel data, as well as methods for scrolling through the video and applying effects to individual pixels.

The `backing` property is a vector for the video's background color, and the `bg` property is a vector for the video's background image. The `start` and `end` properties define the range of pixels that will be included in the video.

The `drawPixel` method appears to be the main method for drawing individual pixels on the video, passing in the required arguments (such as the pixel position, the video's background, and the pixel color). The method handles scaling and offsetting the pixel based on the video's display settings.

The `scroll` method appears to be responsible for scrolling through the video, with the `start` and `end` properties defining the starting and ending points. The method loops through each pixel in the video, applying the appropriate effect and offsetting the pixel based on the video's display settings.

The `applyEffect` method appears to be a method for applying effects to individual pixels, passing in the required arguments (such as the pixel position, the desired effect, and the video's background). The method applies the effect to the pixel and re-renders the video to account for the change.


```js
GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBGMode4 = function(backing, bg, start, end) {
	var video = this.video;
	var x;
	var y = video.vcount;
	var offset = start;
	var localX;
	var localY;
	var charBase = 0;
	if (video.displayFrameSelect) {
		charBase += 0xA000;
	}
	var size = bg.size;
	var index = bg.index;
	var map = video.sharedMap;
	var color;
	var mask = video.target2[index] | (bg.priority << 1) | video.BACKGROUND_MASK;
	if (video.blendMode == 1 && video.alphaEnabled) {
		mask |= video.target1[index];
	}

	var yBase;

	for (x = start; x < end; ++x) {
		localX = bg.dx * x + bg.sx;
		localY = 0 | bg.dy * x + bg.sy;
		if (this.mosaic) {
			localX -= (x % video.bgMosaicX) * bg.dx + (y % video.bgMosaicY) * bg.dmx;
			localY -= (x % video.bgMosaicX) * bg.dy + (y % video.bgMosaicY) * bg.dmy;
		}
		yBase = (localY << 2) & 0x7E0;
		if (localX < 0 || localY < 0 || localX >= video.HORIZONTAL_PIXELS || localY >= video.VERTICAL_PIXELS) {
			offset++;
			continue;
		}
		color = this.vram.loadU8(charBase + (localY * video.HORIZONTAL_PIXELS) + localX);
		bg.pushPixel(index, map, video, color, 0, offset, backing, mask, false);
		offset++;
	}
};

```

这段代码是 GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBGmode5 函数，它执行以下操作：

1. 初始化值：将 backing、bg 和 start、end 设为零。
2. 获取 video 对象的视频和图形模式是否为 5，如果是，则将绘制背景颜色设置为 255 黑色，否则不绘制。
3. 计算偏移量，它将根据 video.displayFrameSelect 是否启用进行调整，启用时偏移量将增加。
4. 设置 localX 和 localY，它们将用于计算从起始位置到结束位置的像素位置。
5. 设置颜色和透明度，使用从背景映射器中读取的 Uint16 类型的值。
6. 遍历起始位置到结束位置的所有像素点，将它们绘制到屏幕上。
7. 如果遍历过程中遇到了负数或大于 160 或小于 0，则相应地调整偏移量，以便在循环结束时回到正确的位置。
8. 最后，将绘制到的像素点存回背景缓冲区。


```js
GameBoyAdvanceSoftwareRenderer.prototype.drawScanlineBGMode5 = function(backing, bg, start, end) {
	var video = this.video;
	var x;
	var y = video.vcount;
	var offset = start;
	var localX;
	var localY;
	var charBase = 0;
	if (video.displayFrameSelect) {
		charBase += 0xA000;
	}
	var index = bg.index;
	var map = video.sharedMap;
	var color;
	var mask = video.target2[index] | (bg.priority << 1) | video.BACKGROUND_MASK;
	if (video.blendMode == 1 && video.alphaEnabled) {
		mask |= video.target1[index];
	}

	var yBase;

	for (x = start; x < end; ++x) {
		localX = bg.dx * x + bg.sx;
		localY = bg.dy * x + bg.sy;
		if (this.mosaic) {
			localX -= (x % video.bgMosaicX) * bg.dx + (y % video.bgMosaicY) * bg.dmx;
			localY -= (x % video.bgMosaicX) * bg.dy + (y % video.bgMosaicY) * bg.dmy;
		}
		if (localX < 0 || localY < 0 || localX >= 160 || localY >= 128) {
			offset++;
			continue;
		}
		color = this.vram.loadU16(charBase + ((localY * 160) + localX) << 1);
		bg.pushPixel(index, map, video, color, 0, offset, backing, mask, true);
		offset++;
	}
};

```



This appears to be a code snippet for a 2D game development engine. It appears to be handling the rendering of windows, objects, and the background.

The code uses the Windowing System v8.10, which provides a layer-based windowing system with support for both windows and objects. The Windowing System v8.10 is included in the Na Inform 8.10 distribution.

The code defines a `Layer` class which is used to represent a layer in the window. A layer is a collection of scanlines, bitmap handles, and other related information that is used for rendering.

The `Layer` class has a number of methods for managing its scanlines, bitmap handles, and other resources. It also provides methods for setting and getting the layer's background color and its transparency.

The `Layer` class has a `drawScanline` method which is used to render a scanline from the specified starting point to the specified ending point. This method takes as input the scanline data, the starting point and ending point, and the layer index.

The `drawScanline` method uses the `getX` and `getY` methods to determine the starting and ending x-positions of the scanline to render. It then sets the `x1` and `x2` variables to the starting and ending x-positions, respectively. It then scales the `x` and `y` values based on the layer index and the blend mode, and finally draws the scanline using the `setBlendEnabled` method.

The `setBlendEnabled` method is used to enable or disable the blend functionality for the layer. It takes as input the blend mode, and as output it does not affect the layer's blend behavior.

The `finishScanline` method is used to finished rendering the scanlines of the layer.

Overall, this code appears to be a part of a larger game development engine, but it is not complete on its own and would need to be integrated with other code to function as part of a complete game.


```js
GameBoyAdvanceSoftwareRenderer.prototype.drawScanline = function(y) {
	var backing = this.scanline;
	if (this.forcedBlank) {
		this.drawScanlineBlank(backing);
		return;
	}
	this.prepareScanline(backing);
	var layer;
	var firstStart;
	var firstEnd;
	var lastStart;
	var lastEnd;
	this.vcount = y;
	// Draw lower priority first and then draw over them
	for (var i = 0; i < this.drawLayers.length; ++i) {
		layer = this.drawLayers[i];
		if (!layer.enabled) {
			continue;
		}
		this.objwinActive = false;
		if (!(this.win0 || this.win1 || this.objwin)) {
			this.setBlendEnabled(layer.index, this.target1[layer.index], this.blendMode);
			layer.drawScanline(backing, layer, 0, this.HORIZONTAL_PIXELS);
		} else {
			firstStart = 0;
			firstEnd = this.HORIZONTAL_PIXELS;
			lastStart = 0;
			lastEnd = this.HORIZONTAL_PIXELS;
			if (this.win0 && y >= this.win0Top && y < this.win0Bottom) {
				if (this.windows[0].enabled[layer.index]) {
					this.setBlendEnabled(layer.index, this.windows[0].special && this.target1[layer.index], this.blendMode);
					layer.drawScanline(backing, layer, this.win0Left, this.win0Right);
				}
				firstStart = Math.max(firstStart, this.win0Left);
				firstEnd = Math.min(firstEnd, this.win0Left);
				lastStart = Math.max(lastStart, this.win0Right);
				lastEnd = Math.min(lastEnd, this.win0Right);
			}
			if (this.win1 && y >= this.win1Top && y < this.win1Bottom) {
				if (this.windows[1].enabled[layer.index]) {
					this.setBlendEnabled(layer.index, this.windows[1].special && this.target1[layer.index], this.blendMode);
					if (!this.windows[0].enabled[layer.index] && (this.win1Left < firstStart || this.win1Right < lastStart)) {
						// We've been cut in two by window 0!
						layer.drawScanline(backing, layer, this.win1Left, firstStart);
						layer.drawScanline(backing, layer, lastEnd, this.win1Right);
					} else {
						layer.drawScanline(backing, layer, this.win1Left, this.win1Right);
					}
				}
				firstStart = Math.max(firstStart, this.win1Left);
				firstEnd = Math.min(firstEnd, this.win1Left);
				lastStart = Math.max(lastStart, this.win1Right);
				lastEnd = Math.min(lastEnd, this.win1Right);
			}
			// Do last two
			if (this.windows[2].enabled[layer.index] || (this.objwin && this.windows[3].enabled[layer.index])) {
				// WINOUT/OBJWIN
				this.objwinActive = this.objwin;
				this.setBlendEnabled(layer.index, this.windows[2].special && this.target1[layer.index], this.blendMode); // Window 3 handled in pushPixel
				if (firstEnd > lastStart) {
					layer.drawScanline(backing, layer, 0, this.HORIZONTAL_PIXELS);
				} else {
					if (firstEnd) {
						layer.drawScanline(backing, layer, 0, firstEnd);
					}
					if (lastStart < this.HORIZONTAL_PIXELS) {
						layer.drawScanline(backing, layer, lastStart, this.HORIZONTAL_PIXELS);
					}
					if (lastEnd < firstStart) {
						layer.drawScanline(backing, layer, lastEnd, firstStart);
					}
				}
			}

			this.setBlendEnabled(this.LAYER_BACKDROP, this.target1[this.LAYER_BACKDROP] && this.windows[2].special, this.blendMode);
		}
		if (layer.bg) {
			layer.sx += layer.dmx;
			layer.sy += layer.dmy;
		}
	}

	this.finishScanline(backing);
};

```

该代码是关于GameBoyAdvanceSoftwareRenderer.prototype.finishScanline函数的定义，用于在渲染结束时扫描场景中的所有像素。

具体来说，该函数接收一个backing参数，它是一个包含透明度的对象。函数内部使用了一系列循环和一个if语句来检查场景中的每个像素是否使用了纹理，以及是否为目标2。如果是目标2，则函数将使用目标1的纹理混合该像素的颜色。

对于每个像素，函数首先检查它是否在纹理中，如果是，则使用纹理的颜色。如果不是，则使用共享颜色(像素的数据结构中存储的顏色)，通常是缓存的颜色。然后，该颜色被转换为32位无符号整数，并存储到像素数据中。最后，在循环结束后，将像素数据中的所有字节数翻倍，以使它们在内存中正确排列。


```js
GameBoyAdvanceSoftwareRenderer.prototype.finishScanline = function(backing) {
	var color;
	var bd = this.palette.accessColor(this.LAYER_BACKDROP, 0);
	var xx = this.vcount * this.HORIZONTAL_PIXELS * 4;
	var isTarget2 = this.target2[this.LAYER_BACKDROP];
	for (var x = 0; x < this.HORIZONTAL_PIXELS; ++x) {
		if (backing.stencil[x] & this.WRITTEN_MASK) {
			color = backing.color[x];
			if (isTarget2 && backing.stencil[x] & this.TARGET1_MASK) {
				color = this.palette.mix(this.blendA, color, this.blendB, bd);
			}
			this.palette.convert16To32(color, this.sharedColor);
		} else {
			this.palette.convert16To32(bd, this.sharedColor);
		}
		this.pixelData.data[xx++] = this.sharedColor[0];
		this.pixelData.data[xx++] = this.sharedColor[1];
		this.pixelData.data[xx++] = this.sharedColor[2];
		xx++;
	}
};

```

这段代码是针对GameBoyAdvanceSoftwareRenderer.prototype中的startDraw和finishDraw方法进行的描述。

startDraw方法的作用是在开始绘制图形时执行，它内部没有任何具体的逻辑。

finishDraw方法的作用是在结束绘制图形时执行，它接收一个caller对象作为参数，然后将渲染器的背景透明度设置为0，并且将渲染器的精灵的x和y坐标设置为精灵的实际位置。最后，调用者也可以选择调用finishDraw方法来结束绘制，该方法将使用self.pixelData作为参数，而不是this.pixelData。

从代码中可以看出，该 renderer 的精灵是存储在256x256像素的纹理中，并且每次调用startDraw方法时，会将背景的透明度设置为50%。而调用finishDraw方法时，会将背景的透明度设置为0，并将精灵的x和y坐标设置为精灵的实际位置。


```js
GameBoyAdvanceSoftwareRenderer.prototype.startDraw = function() {
	// Nothing to do
};

GameBoyAdvanceSoftwareRenderer.prototype.finishDraw = function(caller) {
	this.bg[2].sx = this.bg[2].refx;
	this.bg[2].sy = this.bg[2].refy;
	this.bg[3].sx = this.bg[3].refx;
	this.bg[3].sy = this.bg[3].refy;
	caller.finishDraw(this.pixelData);
};

```