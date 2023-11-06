# GBAJS源码解析 4

# `js/savedata.js`



该代码定义了一个名为SRAMSavedata的函数，其作用是将一个指定大小的二进制数据存储到内存中，并支持存储8位和16位数据。

该函数使用了JavaScript中的MemoryView类，以便更轻松地管理内存。MemoryView类提供了对二进制数据的读取和写入能力，并允许将数据直接缝接到程序的堆上。

具体来说，该函数通过调用MemoryView的构造函数，创建一个新的MemoryView对象，并将其大小设置为指定的二进制数据的大小。然后，该对象上的store8和store16方法被用来存储8位和16位数据。其中，store8方法将数据写入到内存中指定的偏移量，并将writePending属性设置为true，表示写入操作正在进行中。而store16方法则将数据写入到内存中指定的偏移量，并同样将writePending属性设置为true。

这样，当调用该函数时，可以只需传入要存储的数据，函数会自动将其写入到内存中，并返回一个布尔值表示写入操作是否成功。


```js
function SRAMSavedata(size) {
	MemoryView.call(this, new ArrayBuffer(size), 0);

	this.writePending = false;
};

SRAMSavedata.prototype = Object.create(MemoryView.prototype);

SRAMSavedata.prototype.store8 = function(offset, value) {
	this.view.setInt8(offset, value);
	this.writePending = true;
};

SRAMSavedata.prototype.store16 = function(offset, value) {
	this.view.setInt16(offset, value, true);
	this.writePending = true;
};

```

这段代码定义了一个名为SRAMSavedata的类，该类继承自MemoryView类，用于在SRAM中进行数据存储和读取。该类包含了一个名为store32的函数，用于在SRAM中存储数据，该函数接受两个参数：一个offset和一个value，分别表示要存储的起始地址和值。函数首先将value设置为二进制形式，并将其写入到offset位置，然后设置一个pendingCommand标志，表明有命令正在等待执行。接着，调用writePending属性设置的命令，如果writePending为true，执行store32函数，否则不执行。

FlashSavedata类用于在SRAM中保存数据，该类包含了一个 bank0变量和一个bank1变量，分别用于存储SRAM的起始地址和结束地址。该类还包含了一些与SRAM操作有关的命令，如COMMAND_WIPE、COMMAND_ERASE_SECTOR、COMMAND_ERASE、COMMAND_ID、COMMAND_SWITCH_BANK和COMMAND_TERMINATE_ID。

此外，FlashSavedata类还包含一个idMode属性，用于控制是否在每次访问SRAM时都分配一个新的ID号。当idMode为true时，每次访问SRAM时，都将生成一个新的ID号；当idMode为false时，将使用之前分配的ID号。


```js
SRAMSavedata.prototype.store32 = function(offset, value) {
	this.view.setInt32(offset, value, true);
	this.writePending = true;
};

function FlashSavedata(size) {
	MemoryView.call(this, new ArrayBuffer(size), 0);

	this.COMMAND_WIPE = 0x10;
	this.COMMAND_ERASE_SECTOR = 0x30;
	this.COMMAND_ERASE = 0x80;
	this.COMMAND_ID = 0x90;
	this.COMMAND_WRITE = 0xA0;
	this.COMMAND_SWITCH_BANK = 0xB0;
	this.COMMAND_TERMINATE_ID = 0xF0;

	this.ID_PANASONIC = 0x1B32;
	this.ID_SANYO = 0x1362;

	this.bank0 = new DataView(this.buffer, 0, 0x00010000);
	if (size > 0x00010000) {
		this.id = this.ID_SANYO;
		this.bank1 = new DataView(this.buffer, 0x00010000);
	} else {
		this.id = this.ID_PANASONIC;
		this.bank1 = null;
	}
	this.bank = this.bank0;

	this.idMode = false;
	this.writePending = false;

	this.first = 0;
	this.second = 0;
	this.command = 0;
	this.pendingCommand = 0;
};

```

这段代码定义了 FlashSavedata.prototype 中的一部分方法，用于从闪存文件中读取数据。

load8函数作用于一个整数offset，首先检查该offset是否在idMode参数中设置为true。如果是，则执行以下操作：

1. 将id的3位二进制数向左移动3位，并取反，再将该3位二进制数与0xFF进行按位与操作，得到一个0到7之间的整数。
2. 如果offset小于2，则返回该整数。
3. 如果offset不满足条件1，则执行以下操作：

   a. 从flash文件中读取数据，存储到this.bank变量中。
   b. 将读取到的数据进行右移8位，即除以256，得到一个0到7之间的整数。
   c. 将上述两个操作结果进行按位与操作，得到一个0到15138之间的整数。
   d. 将上述结果与0xFF进行按位与操作，得到一个0到15138之间的整数。
   e. 返回上述结果。

load16函数作用于一个整数offset，与load8函数作用于一个整数offset相同。该函数首先调用load8函数，获取id为0的8位无符号整数，然后执行以下操作：

1. 将上述获取到的id的8位无符号整数与0xFF进行按位与操作，得到一个0到7之间的整数。
2. 将上述结果进行右移8位，即除以256，得到一个0到7之间的整数。
3. 将上述结果与上述获取到的id的8位无符号整数进行按位与操作，得到一个0到15138之间的整数。


```js
FlashSavedata.prototype = Object.create(MemoryView.prototype);

FlashSavedata.prototype.load8 = function(offset) {
	if (this.idMode && offset < 2) {
		return (this.id >> (offset << 3)) & 0xFF;
	} else if (offset < 0x10000) {
		return this.bank.getInt8(offset);
	} else {
		return 0;
	}
};

FlashSavedata.prototype.load16 = function(offset) {
	return (this.load8(offset) & 0xFF) | (this.load8(offset + 1) << 8);
};

```

This is a code snippet for a memory management unit (MMU) that implements the offsetting functionality for a RISC-based processor.

The MMU is responsible for managing the virtual address space and the physical memory. It maintains a mapping between the virtual address and the physical memory locations.

The code implements several commands for the MMU:

* Writing to a physical location: This command is responsible for writing data to a specific offset in the physical memory. It takes a virtual address and a value as input parameters.
* Erasing a physical location: This command is responsible for erasing a physical location in the physical memory. It takes a virtual address as an input parameter.
* Writing to all logical devices: This command is responsible for writing data to all logical devices (e.g. a RGB buffer). It takes a virtual address as an input parameter.
* Reading from all logical devices: This command is responsible for reading data from all logical devices. It takes a virtual address as an input parameter.
* Banks: This command is responsible for setting or resetting the bank table.
*段命令： This command is responsible for setting or resetting the段命令.
*页命令： This command is responsible for setting or resetting the页命令.
*配置命令： This command is responsible for setting or resetting the配置命令。


```js
FlashSavedata.prototype.load32 = function(offset) {
	return (this.load8(offset) & 0xFF) | (this.load8(offset + 1) << 8) | (this.load8(offset + 2) << 16) | (this.load8(offset + 3) << 24);
};

FlashSavedata.prototype.loadU8 = function(offset) {
	return this.load8(offset) & 0xFF;
};

FlashSavedata.prototype.loadU16 = function(offset) {
	return (this.loadU8(offset) & 0xFF) | (this.loadU8(offset + 1) << 8);
};

FlashSavedata.prototype.store8 = function(offset, value) {
	switch (this.command) {
	case 0:
		if (offset == 0x5555) {
			if (this.second == 0x55) {
				switch (value) {
				case this.COMMAND_ERASE:
					this.pendingCommand = value;
					break;
				case this.COMMAND_ID:
					this.idMode = true;
					break;
				case this.COMMAND_TERMINATE_ID:
					this.idMode = false;
					break;
				default:
					this.command = value;
					break;
				}
				this.second = 0;
				this.first = 0;
			} else {
				this.command = 0;
				this.first = value;
				this.idMode = false;
			}
		} else if (offset == 0x2AAA && this.first == 0xAA) {
			this.first = 0;
			if (this.pendingCommand) {
				this.command = this.pendingCommand;
			} else {
				this.second = value;
			}
		}
		break;
	case this.COMMAND_ERASE:
		switch (value) {
		case this.COMMAND_WIPE:
			if (offset == 0x5555) {
				for (var i = 0; i < this.view.byteLength; i += 4) {
					this.view.setInt32(i, -1);
				}
			}
			break;
		case this.COMMAND_ERASE_SECTOR:
			if ((offset & 0x0FFF) == 0) {
				for (var i = offset; i < offset + 0x1000; i += 4) {
					this.bank.setInt32(i, -1);
				}
			}
			break;
		}
		this.pendingCommand = 0;
		this.command = 0;
		break;
	case this.COMMAND_WRITE:
		this.bank.setInt8(offset, value);
		this.command = 0;

		this.writePending = true;
		break;
	case this.COMMAND_SWITCH_BANK:
		if (this.bank1 && offset == 0) {
			if (value == 1) {
				this.bank = this.bank1;
			} else {
				this.bank = this.bank0;
			}
		}
		this.command = 0;
		break;
	}
};

```

这段代码定义了一个名为 FlashSavedata.prototype 的对象，其中包括三个方法：store16、store32 和 replaceData。这些方法的主要作用是保存和替换数据到便携式存储器（如闪存）中。

具体来说，当调用 store16 或 store32 时，会抛出一个名为 "Unaligned save to flash!" 的错误，这意味着代码可能从非原始位置或非原始长度数据开始写入。

而 replaceData 方法的主要作用是在内存中插入或替换数据。它首先检查要存储或替换的数据的内存位置和长度。然后，它调用 MemoryView.prototype.replaceData 方法来在原始数据位置或指定位置插入新数据。

如果要在闪存中存储数据，该方法首先检查要存储数据的内存位置和长度。如果是，它创建一个新的 DataView 对象，并将其设置为要存储的数据的内存位置的偏移量。如果要在同一块闪存中存储不同长度的数据，该方法将根据要存储的数据长度创建一个或两个 DataView 对象。

此外，replaceData 方法还会在原始数据位置或指定位置插入新数据，并在完成插入后通知数据已成功替换。


```js
FlashSavedata.prototype.store16 = function(offset, value) {
	throw new Error("Unaligned save to flash!");
};

FlashSavedata.prototype.store32 = function(offset, value) {
	throw new Error("Unaligned save to flash!");
};

FlashSavedata.prototype.replaceData = function(memory) {
	var bank = this.view === this.bank1;
	MemoryView.prototype.replaceData.call(this, memory, 0);

	this.bank0 = new DataView(this.buffer, 0, 0x00010000);
	if (memory.byteLength > 0x00010000) {
		this.bank1 = new DataView(this.buffer, 0x00010000);
	} else {
		this.bank1 = null;
	}
	this.bank = bank ? this.bank1 : this.bank0;
};

```

该代码定义了一个名为EEPROMSavedata的函数，用于在EEPROM中读取/写入数据。以下是该函数的功能解释：

- 函数参数为两个整数，size 和 mmu。size表示EEPROM的存储器大小，mmu是一个用于读取/写入EEPROM的内存单元数(必须是4的倍数)。函数默认构造函数是使用new ArrayBuffer(size)创建一个大小为size的内存缓冲区，并将其赋值为0。

- 函数内部创建了一个名为this的变量，用于跟踪函数自身的状态，包括写地址、读比特计数器、读地址、命令、读/写 pending等。

- 函数的dma通道属性被设置为mmu.core.irq.dma[3]，用于从dma通道中读取/写入数据。

- 函数的COMMAND_NULL被设置为0,COMMAND_PENDING被设置为1,COMMAND_READ_PENDING被设置为2,COMMAND_READ被设置为4，用于在函数中执行相应的命令。

- 函数的实大小被设置为0，用于跟踪函数中实际读/写入的数据大小。

- 函数的addressBits被设置为0，用于指示函数正在读取/写入的地址位。

- 函数的writePending被设置为false，用于指示函数是否正在等待写入操作完成。

- 函数可以调用函数自身的方法来实现读/写入操作，包括writeAddress、readBitsRemaining、readAddress、command等。


```js
function EEPROMSavedata(size, mmu) {
	MemoryView.call(this, new ArrayBuffer(size), 0);

	this.writeAddress = 0;
	this.readBitsRemaining = 0;
	this.readAddress = 0;

	this.command = 0;
	this.commandBitsRemaining = 0;

	this.realSize = 0;
	this.addressBits = 0;
	this.writePending = false;

	this.dma = mmu.core.irq.dma[3];

	this.COMMAND_NULL = 0;
	this.COMMAND_PENDING = 1;
	this.COMMAND_WRITE = 2;
	this.COMMAND_READ_PENDING = 3;
	this.COMMAND_READ = 4;
};

```

这段代码将EEPROMSavedata.prototype对象从内存中的MemoryView.prototype对象中创建，并添加了两个函数：load8和load16，用于读取8位和16位数据。

load8函数接受一个offset参数，表示从EEPROM的某个地址开始，向后偏移多少个字节来读取数据。这个函数会抛出一个Error，因为它不支持8位数据的读取。

load16函数与load8类似，但是会读取16位数据。它也接受一个offset参数，并从EEPROM的某个地址开始，向后偏移多少个字节来读取数据。这个函数会抛出一个Error，因为它不支持16位数据的读取。

loadU8函数也接受一个offset参数，表示从EEPROM的某个地址开始，向后偏移多少个字节来读取数据。这个函数会抛出一个Error，因为它不支持8位数据的读取。

loadU16函数与loadU8类似，但是会读取16位数据。它也接受一个offset参数，并从EEPROM的某个地址开始，向后偏移多少个字节来读取数据。这个函数不会抛出Error，因为它支持16位数据的读取。


```js
EEPROMSavedata.prototype = Object.create(MemoryView.prototype);

EEPROMSavedata.prototype.load8 = function(offset) {
	throw new Error("Unsupported 8-bit access!");
};

EEPROMSavedata.prototype.load16 = function(offset) {
	return this.loadU16(offset);
};

EEPROMSavedata.prototype.loadU8 = function(offset) {
	throw new Error("Unsupported 8-bit access!");
};

EEPROMSavedata.prototype.loadU16 = function(offset) {
	if (this.command != this.COMMAND_READ || !this.dma.enable) {
		return 1;
	}
	--this.readBitsRemaining;
	if (this.readBitsRemaining < 64) {
		var step = 63 - this.readBitsRemaining;
		var data = this.view.getUint8((this.readAddress + step) >> 3, false) >> (0x7 - (step & 0x7));
		if (!this.readBitsRemaining) {
			this.command = this.COMMAND_NULL;
		}
		return data & 0x1;
	}
	return 0;
};

```

This is a description of a command buffer object (CBO) for a microcontroller or microprocessor. It has a real size, which is the number of bytes in a single read/write cycle. It also has an address bits and a command bits, which determine the operation that is being performed. The command bits are set based on the read/write operation and the address bits are set based on the address being read.

The CBO has a number of methods that are available on the command buffer object, including the ability to write to a buffer, read from a buffer, and set the buffer to read/write mode. The write method takes an address and a value, and writes the value to the specified buffer at the specified address. The read method takes the address of the buffer, and reads the value at that address. The read/write mode method can be used to put the buffer in read/write mode, and the CBO will perform the appropriate write/read operation on the buffer.

The CBO also has a method called the reset command, which will reset all of its settings to their default values.


```js
EEPROMSavedata.prototype.load32 = function(offset) {
	throw new Error("Unsupported 32-bit access!");
};

EEPROMSavedata.prototype.store8 = function(offset, value) {
	throw new Error("Unsupported 8-bit access!");
};

EEPROMSavedata.prototype.store16 = function(offset, value) {
	switch (this.command) {
	// Read header
	case this.COMMAND_NULL:
	default:
		this.command = value & 0x1;
		break;
	case this.COMMAND_PENDING:
		this.command <<= 1;
		this.command |= value & 0x1;
		if (this.command == this.COMMAND_WRITE) {
			if (!this.realSize) {
				var bits = this.dma.count - 67;
				this.realSize = 8 << bits;
				this.addressBits = bits;
			}
			this.commandBitsRemaining = this.addressBits + 64 + 1;
			this.writeAddress = 0;
		} else {
			if (!this.realSize) {
				var bits = this.dma.count - 3;
				this.realSize = 8 << bits;
				this.addressBits = bits;
			}
			this.commandBitsRemaining = this.addressBits + 1;
			this.readAddress = 0;
		}
		break;
	// Do commands
	case this.COMMAND_WRITE:
		// Write
		if (--this.commandBitsRemaining > 64) {
			this.writeAddress <<= 1;
			this.writeAddress |= (value & 0x1) << 6;
		} else if (this.commandBitsRemaining <= 0) {
			this.command = this.COMMAND_NULL;
			this.writePending = true;
		} else {
			var current = this.view.getUint8(this.writeAddress >> 3);
			current &= ~(1 << (0x7 - (this.writeAddress & 0x7)));
			current |= (value & 0x1) << (0x7 - (this.writeAddress & 0x7));
			this.view.setUint8(this.writeAddress >> 3, current);
			++this.writeAddress;
		}
		break;
	case this.COMMAND_READ_PENDING:
		// Read
		if (--this.commandBitsRemaining > 0) {
			this.readAddress <<= 1;
			if (value & 0x1) {
				this.readAddress |= 0x40;
			}
		} else {
			this.readBitsRemaining = 68;
			this.command = this.COMMAND_READ;
		}
		break;
	}
};

```

这段代码是定义了一个名为EEPROMSavedata.prototype.store32的函数和一个名为EEPROMSavedata.prototype.replaceData的函数。

store32函数：
```jsjavascript
EEPROMSavedata.prototype.store32 = function(offset, value) {
	throw new Error("Unsupported 32-bit access!");
};
```
这个函数用来存储一个32位无符号整数（即32位二进制数）到EEPROM芯片中的指定偏移位置，但是会抛出一个新的错误，因为EEPROM芯片不支持32位访问。

replaceData函数：
```jsjavascript
EEPROMSavedata.prototype.replaceData = function(memory) {
	MemoryView.prototype.replaceData.call(this, memory, 0);
};
```
这个函数用来在EEPROM芯片的指定内存位置替换一个新的数据。它调用了MemoryView.prototype.replaceData的函数，并把新的内存地址传递给该函数，以便覆盖原来的数据。


```js
EEPROMSavedata.prototype.store32 = function(offset, value) {
	throw new Error("Unsupported 32-bit access!");
};

EEPROMSavedata.prototype.replaceData = function(memory) {
	MemoryView.prototype.replaceData.call(this, memory, 0);
};

```

# `js/sio.js`



该代码是一个 SIO 控制器类，用于控制 GameBoy 的串口通信。该类包含以下成员变量：

- SIO_NORMAL_8:8 位 SIO 口正常工作模式，高电平为1，低电平为0。
- SIO_NORMAL_32:32 位 SIO 口正常工作模式，高电平为1，低电平为0。
- SIO_MULTI: 多功能设置，可以设置为0、1或2。
- SIO_UART: 串口输出单元，用于与 external device 通信。
- SIO_GPIO: 通用输入输出单元，用于与 GameBoy 的其他部分通信，包括与 link layer 通信。
- SIO_JOYBUS: JoyBass 控制器上的 joy bus。

该类的构造函数默认值为以下值：

- SIO_NORMAL_8:0
- SIO_NORMAL_32:0
- SIO_MULTI:0
- SIO_UART:0
- SIO_GPIO:0
- SIO_JOYBUS:0

该类的 clear 函数用于将 SIO 口设置为输出模式，并将 GameBoy 的 link layer 设置为输出模式。该函数还设置多功能设置为0，并清除 JoyBass 控制器上的 joy bus。


```js
function GameBoyAdvanceSIO() {
	this.SIO_NORMAL_8 = 0;
	this.SIO_NORMAL_32 = 1;
	this.SIO_MULTI = 2;
	this.SIO_UART = 3;
	this.SIO_GPIO = 8;
	this.SIO_JOYBUS = 12;

	this.BAUD = [ 9600, 38400, 57600, 115200 ];
}

GameBoyAdvanceSIO.prototype.clear = function() {
	this.mode = this.SIO_GPIO;
	this.sd = false;

	this.irq = false;
	this.multiplayer = {
		baud: 0,
		si: 0,
		id: 0,
		error: 0,
		busy: 0,

		states: [ 0xFFFF, 0xFFFF, 0xFFFF, 0xFFFF ]
	};

	this.linkLayer = null;
};

```

这段代码是针对GameBoyAdvanceSIO类的一个setMode和writeRCNT方法。

setMode方法的目的是设置SIO模式。SIO模式有3种，分别为0x0001、0x0002和0x0004。在setMode方法中，首先检查输入的mode是否与0x8相等，如果不相等，则将mode按位与0xC，然后将结果存储到this.mode变量中。这样可以确保只有一种可能的SIO模式被设置。接着，输出一条信息，指出设置的SIO模式。

writeRCNT方法的目的是在SIO模式设置为GPIO时，向设备写入RCNT值。这个方法的前提条件是当前的SIO模式不是GPIO模式。在writeRCNT方法中，首先检查当前的SIO模式是否为GPIO，如果不是，则不做任何操作，直接返回。否则，调用一个内部函数，然后将rcnt值输出。这个内部函数可能是用于向设备写入数据的函数。


```js
GameBoyAdvanceSIO.prototype.setMode = function(mode) {
	if (mode & 0x8) {
		mode &= 0xC;
	} else {
		mode &= 0x3;
	}
	this.mode = mode;

	this.core.INFO('Setting SIO mode to ' + hex(mode, 1));
};

GameBoyAdvanceSIO.prototype.writeRCNT = function(value) {
	if (this.mode != this.SIO_GPIO) {
		return;
	}

	this.core.STUB('General purpose serial not supported');
};

```

该代码是一个SIO（System I/O）函数，其作用是读取或写入某个特定SIOCNT（系统I/O控制器）寄存器的值，并在GameBoyAdvanceSIO.prototype对象中定义了一个writeSIOCNT函数。

以下是该函数的实现过程：

1. 首先，根据所处的SIO模式（正常8位、32位或多态），确定是否支持读写该寄存器。
2. 如果支持读写，根据所处的SIO模式确定相应的行为：
	* 正常8位模式：执行无操作。
	* 32位模式：执行无操作。
	* 多态模式：
		+ 如果游戏主机支持Multiplayer（多人在线），则执行以下操作：
			- 设置Multiplayer的Baud值。
			- 如果LinkLayer（链路层）存在，则设置其Baud值。
			- 设置Multiplayer的Busy位。
			- 如果LinkLayer和Multiplayer的Busy位都设置为1，则开始Multiplayer传输。
			- 设置Irq（中断）位。
			- 执行写入操作。
			- 如果Multiplayer和Busy位都设置为0，则跳过写入操作。
		+ 否则：
			- 如果所处的SIO模式是UART（串行通信），则执行无操作。
			- 如果所处的SIO模式是GPIO（通用编程），则执行无操作。
			- 如果所处的SIO模式是JoyBUS（游戏机总线），则执行无操作。
3. 根据所处的SIO模式，执行相应的操作。

这个函数的作用是读写特定SIOCNT寄存器的值，根据所处的SIO模式执行相应的操作，并将其结果返回。


```js
GameBoyAdvanceSIO.prototype.writeSIOCNT = function(value) {
	switch (this.mode) {
	case this.SIO_NORMAL_8:
		this.core.STUB('8-bit transfer unsupported');
		break;
	case this.SIO_NORMAL_32:
		this.core.STUB('32-bit transfer unsupported');
		break;
	case this.SIO_MULTI:
		this.multiplayer.baud = value & 0x0003;
		if (this.linkLayer) {
			this.linkLayer.setBaud(this.BAUD[this.multiplayer.baud]);
		}

		if (!this.multiplayer.si) {
			this.multiplayer.busy = value & 0x0080;
			if (this.linkLayer && this.multiplayer.busy) {
				this.linkLayer.startMultiplayerTransfer();
			}
		}
		this.irq = value & 0x4000;
		break;
	case this.SIO_UART:
		this.core.STUB('UART unsupported');
		break;
	case this.SIO_GPIO:
		// This register isn't used in general-purpose mode
		break;
	case this.SIO_JOYBUS:
		this.core.STUB('JOY BUS unsupported');
		break;
	}
};

```

该代码是一个名为 `GameBoyAdvanceSIO.prototype.readSIOCNT` 的函数，属于 GameBoy Advance SIO 对象的继承。该函数通过读取状态寄存器(SIOCNT)来设置 GameBoy 的状态。

SIOCNT 寄存器是一个8位或32位的二进制数，用于控制 GameBoy 在不同状态下的行为。根据函数的参数，可以确定要设置的状态，并相应地执行 stub 函数(如果状态有相应的设置)。

以下是具体的说明：

- `this.mode` 获取当前 GameBoy 的模式，并将其作为参数传递给 `switch` 语句。
- `switch` 语句用于检查当前模式，根据不同的模式执行相应的 stub 函数。
- `case this.SIO_NORMAL_8:` 表示当前模式为正常模式下的 8 位传输，执行 `this.core.STUB('8-bit transfer unsupported')`。
- `case this.SIO_NORMAL_32:` 表示当前模式为正常模式下的 32 位传输，执行 `this.core.STUB('32-bit transfer unsupported')`。
- `case this.SIO_MULTI:` 表示当前模式为多玩家模式，执行以下操作：
 - 将 `this.multiplayer.baud` 值设置为二进制数中 4 位宽的第一位，然后将其与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
 - 将 `this.multiplayer.si` 值设置为二进制数中 4 位宽的第二位，然后将其与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
 - 将 `this.multiplayer.sd` 值与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
 - 如果 `this.multiplayer.id` 标志为1，则执行以下操作：
   - 将 `this.multiplayer.error` 值设置为二进制数中 4 位宽的第三位，然后将其与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
   - 将 `this.multiplayer.busy` 值设置为二进制数中 4位宽的第四位，然后将其与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
   - 如果 `this.multiplayer.irq` 标志为1，则执行以下操作：
     - 将 `this.multiplayer.ax` 值设置为二进制数中 4位宽的第五位，然后将其与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
     - 将 `this.multiplayer.ay` 值设置为二进制数中 4位宽的第六位，然后将其与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
     - 将 `this.multiplayer.az` 值设置为二进制数中 4位宽的第七位，然后将其与SIOCNT中的值进行按位与操作，得到一个新的值作为SIOCNT的值。
     - 没有上述操作，执行 `this.core.STUB('UART unsupported')`。
- `case this.SIO_UART:` 表示当前模式为 UART 模式，执行以下操作：
 - 如果当前已经设置为 UART 模式，则不做任何处理。
 - 否则执行 `this.core.STUB('UART unsupported')`。
- `case this.SIO_GPIO:` 表示当前模式为普通模式，执行以下操作：
 - 如果不执行任何操作，则不做任何处理。
 - 否则执行 `this.core.STUB('JOY BUS unsupported')`。


```js
GameBoyAdvanceSIO.prototype.readSIOCNT = function() {
	var value = (this.mode << 12) & 0xFFFF;
	switch (this.mode) {
	case this.SIO_NORMAL_8:
		this.core.STUB('8-bit transfer unsupported');
		break;
	case this.SIO_NORMAL_32:
		this.core.STUB('32-bit transfer unsupported');
		break;
	case this.SIO_MULTI:
		value |= this.multiplayer.baud;
		value |= this.multiplayer.si;
		value |= (!!this.sd) << 3;
		value |= this.multiplayer.id << 4;
		value |= this.multiplayer.error;
		value |= this.multiplayer.busy;
		value |= (!!this.multiplayer.irq) << 14;
		break;
	case this.SIO_UART:
		this.core.STUB('UART unsupported');
		break;
	case this.SIO_GPIO:
		// This register isn't used in general-purpose mode
		break;
	case this.SIO_JOYBUS:
		this.core.STUB('JOY BUS unsupported');
		break;
	}
	return value;
};

```

这段代码是针对GameBoyAdvanceSIO.prototype中read函数的功能进行解释。

当调用read函数时，会根据传入的slot参数来尝试从转移寄存器中读取数据。根据所传入的模式，代码会执行相应的代码。

- 如果传入的是this.SIO_NORMAL_32模式，会执行this.core.STUB('32-bit transfer unsupported')代码，因为该模式不支持32位数据传输。
- 如果传入的是this.SIO_MULTI模式，会尝试从多玩家的状态中读取第slot个数据，并返回该数据。
- 如果传入的是this.SIO_UART模式，会尝试从UART缓冲区中读取数据，并返回一个值为0的布尔值，因为该模式不支持从UART缓冲区中读取数据。
- 如果传入的是任何其他模式，会执行this.core.WARN('Reading from transfer register in unsupported mode')代码，因为代码无法从该模式中正常读取数据。

return 0；代码会返回一个值为0的布尔值，表示本次尝试是否成功读取数据。


```js
GameBoyAdvanceSIO.prototype.read = function(slot) {
	switch (this.mode) {
	case this.SIO_NORMAL_32:
		this.core.STUB('32-bit transfer unsupported');
		break;
	case this.SIO_MULTI:
		return this.multiplayer.states[slot];
	case this.SIO_UART:
		this.core.STUB('UART unsupported');
		break;
	default:
		this.core.WARN('Reading from transfer register in unsupported mode');
		break;
	}
	return 0;
};

```

# `js/thumb.js`

这段代码定义了一个名为ARMCoreThumb的函数，该函数接受一个CPU对象作为参数。

ARMCoreThumb.prototype.constructADC = function(rd, rm) {
 var cpu = this.cpu;
 var gprs = cpu.gprs;
 return function() {
   cpu.mmu.waitPrefetch(gprs[cpu.PC]);
   var m = (gprs[rm] >>> 0) + !!cpu.cpsrC;
   var oldD = gprs[rd];
   var d = (oldD >>> 0) + m;
   var oldDn = oldD >> 31;
   var dn = d >> 31;
   var mn = m >> 31;
   cpu.cpsrN = dn;
   cpu.cpsrZ = !(d & 0xFFFFFFFF);
   cpu.cpsrC = d > 0xFFFFFFFF;
   cpu.cpsrV = oldDn == mn && oldDn != dn && mn != dn;
   gprs[rd] = d;
 };
};


```js
ARMCoreThumb = function (cpu) {
	this.cpu = cpu;
};

ARMCoreThumb.prototype.constructADC = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var m = (gprs[rm] >>> 0) + !!cpu.cpsrC;
		var oldD = gprs[rd];
		var d = (oldD >>> 0) + m;
		var oldDn = oldD >> 31;
		var dn = d >> 31;
		var mn = m >> 31;
		cpu.cpsrN = dn;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = d > 0xFFFFFFFF;
		cpu.cpsrV = oldDn == mn && oldDn != dn && mn != dn;
		gprs[rd] = d;
	};
};

```

这两段代码是JavaScript中的函数继承，涉及到ARMCoreThumb.prototype中的两个方法：constructADD1和constructADD2。

作用是创建两个Add1和Add2函数，通过组合CPU的内存对象和CPU的立即需要来执行加法操作。

构造Add1函数，在rd和rn参数中指定需要加法的寄存器，在构造函数内部，首先获取CPU实例并设置gprs寄存器，然后执行一个高精度加法操作，将结果存储在cpsrN、cpsrZ、cpsrC和cpsrV中，最后将结果存储回gprs中。

构造Add2函数，在rn参数中指定需要加法的寄存器，在构造函数内部，首先获取CPU实例并设置gprs寄存器，然后执行一个高精度加法操作，将结果存储在cpsrN中，最后将立即需要的immediate加入其中，并将结果存储回gprs中。


```js
ARMCoreThumb.prototype.constructADD1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = (gprs[rn] >>> 0) + immediate;
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = d > 0xFFFFFFFF;
		cpu.cpsrV = !(gprs[rn] >> 31) && ((gprs[rn] >> 31 ^ d) >> 31) && (d >> 31);
		gprs[rd] = d;
	};
};

ARMCoreThumb.prototype.constructADD2 = function(rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = (gprs[rn] >>> 0) + immediate;
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = d > 0xFFFFFFFF;
		cpu.cpsrV = !(gprs[rn] >> 31) && ((gprs[rn] ^ d) >> 31) && ((immediate ^ d) >> 31);
		gprs[rn] = d;
	};
};

```

这段代码定义了两个名为 `constructADD3` 和 `constructADD4` 的函数，它们属于 ARMCoreThumb 类。

`constructADD3` 函数的实现比较复杂，但可以大致理解为：

1. 初始化 cpu 和 gprs，并将这两个引用保存在 `this.cpu` 和 `this.gprs` 变量中。
2. 创建一个回调函数，该函数会在内部执行一些计算操作。
3. 通过 `cpu.mmu.waitPrefetch` 方法将 gprs 中的指针加到 cpu 的 mmu 缓存中。
4. 通过位运算 `(gprs[rn] << 0) + (gprs[rm] << 0)` 将两个 gprs 值相加并左移 31 位，得到一个新的 gprs 数组。
5. 通过一系列位运算 `(d & 0xFFFFFFFF) & ((gprs[rn] ^ d) << 31) & ((gprs[rm] ^ d) << 31) & ((gprs[rd] ^ d) << 31)` 将 d 和两个指针的值相加，并将结果与 0xFFFFFFFF 进行按位与和左移 31 位，得到一个新的 cpsr 数组。
6. 将新的 cpsr 数组保存回原来的 gprs 数组中。

`constructADD4` 函数的实现比较简单，但含义如下：

1. 初始化 cpu 和 gprs，并将这两个引用保存在 `this.cpu` 和 `this.gprs` 变量中。
2. 创建一个回调函数，该函数会在内部执行一些计算操作。
3. 通过 `cpu.mmu.waitPrefetch` 方法将 gprs 中的指针加到 cpu 的 mmu 缓存中。
4. 通过位运算 `gprs[rd] += gprs[rm]` 将两个 gprs 值相加并左移 31 位，得到一个新的 gprs 数组。

总的来说，这两个函数的主要作用是协助在 ARM 架构下执行 Add3 和 Add4 运算，并将计算结果保存回原来的 gprs 数组中。


```js
ARMCoreThumb.prototype.constructADD3 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = (gprs[rn] >>> 0) + (gprs[rm] >>> 0);
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = d > 0xFFFFFFFF;
		cpu.cpsrV = !((gprs[rn] ^ gprs[rm]) >> 31) && ((gprs[rn] ^ d) >> 31) && ((gprs[rm] ^ d) >> 31);
		gprs[rd] = d;
	};
};

ARMCoreThumb.prototype.constructADD4 = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] += gprs[rm];
	};
};

```

这两段代码是定义了两个名为 "constructADD5" 和 "constructADD6" 的函数，它们的目的是创建一个新的 "ARMCoreThumb" 对象中的 "additional memory regions" 属性。

具体来说，这两段代码实现了将一个 "gprs" 数组中的内存区域，与一个 "immediate" 参数相加，并将结果存储在 "gprs" 数组中的 "rd" 位置。这里使用了 "CPU 内存单元"(CPU内存单元是指 ArduPIC 中的内存区域，与实际物理内存不同)的 "waitPrefetch" 函数来挂起对该内存区域的应用程序上下文，以避免在函数返回前更改该内存区域的内容。


```js
ARMCoreThumb.prototype.constructADD5 = function(rd, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = (gprs[cpu.PC] & 0xFFFFFFFC) + immediate;
	};
};

ARMCoreThumb.prototype.constructADD6 = function(rd, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = gprs[cpu.SP] + immediate;
	};
};

```

这两段代码定义了两个名为 `constructADD7` 和 `constructAND` 的函数，它们都属于 ARM Core Thumb 类的一个原型。

`constructADD7` 函数接收一个 `immediate` 参数，并在创建一个新函数时调用该函数。在新函数中，使用 `cpu.mmu.waitPrefetch` 函数来挂载到指定的分页器（GPRS），然后通过 `gprs[cpu.PC]` 获取到需要加到分页器上的数据，并将 `immediate` 参数与分页器上的数据进行异或运算，最后将结果添加到指定的分页器上。

`constructAND` 函数接收两个 `rd` 和 `rm` 参数，并在创建一个新函数时调用该函数。在新函数中，使用 `cpu.mmu.waitPrefetch` 函数来挂载到指定的分页器，然后对两个分页器上的数据进行按位与运算，将 `rd` 对应的二进制位与 `rm` 对应的二进制位与分页器上的数据进行按位与运算，再将结果移除 31 位高字节，最后将 `rd` 和 `rm` 的高字节设置为 1，以确保 `rm` 对应的二进制位没有被设置为 0。


```js
ARMCoreThumb.prototype.constructADD7 = function(immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[cpu.SP] += immediate;
	};
};

ARMCoreThumb.prototype.constructAND = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = gprs[rd] & gprs[rm];
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

该代码是一个JavaScript方法，名为"ARMCoreThumb.prototype.constructASR1"，属于"ARMCoreThumb"类的原型。

该方法接受三个参数：rd、rm和immediate，其中rd和rm是整数，immediate是一个布尔值，表示是否立即执行操作。

以下是该方法的实现步骤：

1. 获取当前CPU实例中的gprs寄存器，并将其存储在变量"cpu"中。
2. 创建一个名为"function() "的函数，作为该方法的返回值。
3. 在函数内部，使用"cpu.mmu.waitPrefetch"方法将当前CPU的内存单元预先加载到gprs寄存器中。
4. 如果immediate是真，那么执行以下操作：
  1. 读取rm中存储的整数，并将其存储在cpu.cpsrC变量中。
  2. 如果rm中的整数大于等于31，则将整个gprs数组设置为2147483647，表示FFFFFFFFFF。
  3. 否则，将rm数组中存储的整数设置为0，并将cd一起设置为immediate-1。
5. 如果immediate是假，那么执行以下操作：
  1. 将rm数组中存储的整数按位异或gprs数组中存储的整数，并将其存储在rd变量中。
  2. 将immediate-1与rm数组中存储的整数按位与，并将其存储在cpu.cpsrC变量中。
  3. 将gprs数组中存储的整数中，从右向左第31位的二进制位设置为1，表示已经执行immediate。
6. 在函数内部，使用"cpu.cpsrN"和"cpu.cpsrZ"变量，将gprs数组中存储的整数按位展开，得到最新的cpu.cpsr值。


```js
ARMCoreThumb.prototype.constructASR1 = function(rd, rm, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		if (immediate == 0) {
			cpu.cpsrC = gprs[rm] >> 31;
			if (cpu.cpsrC) {
				gprs[rd] = 0xFFFFFFFF;
			} else {
				gprs[rd] = 0;
			}
		} else {
			cpu.cpsrC = gprs[rm] & (1 << (immediate - 1));
			gprs[rd] = gprs[rm] >> immediate;
		}
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这段代码定义了一个名为 "ARMCoreThumb" 的对象的一个名为 "constructASR2" 的方法，其参数为两个整数 "rd" 和 "rm"。

该方法的实现如下：

1. 创建一个名为 "cpu" 的变量，并将其设置为 "this" 的实例；
2. 创建一个名为 "gprs" 的数组，将其设置为 "cpu.gprs" 的值；
3. 返回一个函数，该函数会在内部执行以下操作：

   a. 获取 "cpu.mmu" 对象的一个子对象，即 "gprs[cpu.PC]" 的值；
   b. 如果 "rs" 小于 32，则执行以下操作：
       - 异或操作：用 "rs" 和 "rs-1" 两个参数替换 "rs" 和 "rs-1";
       - 左移操作：将 "rs" 的二进制位向左移动 "rs-1" 位；
   c. 如果 "rs" 大于或等于 32，则执行以下操作：
       - 异或操作：用 "rs" 和 "rs-1" 两个参数替换 "rs" 和 "rs-1";
       - 右移操作：将 "rs" 的二进制位向右移动 "rs-1" 位；
       - 如果 "rs" 的二进制位为 1，则执行以下操作：
           - 异或操作：用 "rs" 和 "rs-1" 两个参数替换 "rs" 和 "rs-1";
           - 右移操作：将 "rs" 的二进制位向右移动 "rs-1" 位；
           - 逻辑非操作：用 "cpu.cpsrC" 和 "rs" 两个参数替换 "rs" 和 "cpu.cpsrC";
           - 左移操作：将 "rs" 的二进制位向左移动 "cpu.cpsrC" 的位数；
           - 逻辑非操作：用 "cpu.cpsrN" 和 "rs" 两个参数替换 "rs" 和 "cpu.cpsrN";
           - 逻辑非操作：用 "cpu.cpsrZ" 和 "rs" 两个参数替换 "rs" 和 "cpu.cpsrZ";
           - 异或操作：用 "cpu.cpsrC" 和 "cpu.cpsrN" 两个参数替换 "cpu.cpsrC" 和 "cpu.cpsrN";
           - 异或操作：用 "rs" 和 "rs-1" 两个参数替换 "rs" 和 "rs-1";
       - 逻辑非操作：用 "cpu.cpsrC" 和 "cpu.cpsrN" 两个参数替换 "cpu.cpsrC" 和 "cpu.cpsrN";
       - 逻辑非操作：用 "cpu.cpsrC" 和 "rs" 两个参数替换 "cpu.cpsrC" 和 "rs";
       - 逻辑非操作：用 "cpu.cpsrN" 和 "rs" 两个参数替换 "cpu.cpsrN" 和 "rs";
       - 逻辑非操作：用 "cpu.cpsrC" 和 "cpu.cpsrN" 两个参数替换 "cpu.cpsrC" 和 "cpu.cpsrN";
       - 异或操作：用 "cpu.cpsrC" 和 "rs" 两个参数替换 "cpu.cpsrC" 和 "rs";
       - 异或操作：用 "rs" 和 "rs-1" 两个参数替换 "rs" 和 "rs-1";


```js
ARMCoreThumb.prototype.constructASR2 = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var rs = gprs[rm] & 0xFF;
		if (rs) {
			if (rs < 32) {
				cpu.cpsrC = gprs[rd] & (1 << (rs - 1));
				gprs[rd] >>= rs;
			} else {
				cpu.cpsrC = gprs[rd] >> 31;
				if (cpu.cpsrC) {
					gprs[rd] = 0xFFFFFFFF;
				} else {
					gprs[rd] = 0;
				}
			}
		}
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这两段代码是JavaScript中的函数，定义在类中名为"ARMCoreThumb"的继承者中。

这两段代码都接受两个参数：一个需要立即执行的操作（在本例中为+immediate），另一个是一个布尔条件操作，如果条件为真，则操作将被立即执行，否则可能不会立即执行。

对于"ARMCoreThumb.prototype.constructB1"函数，它会执行一个体外的操作，通过一个闭包函数来实现：

1. 获取当前的CPU实例，并保存到变量"cpu"中；
2. 获取CPU的MMU，并使用"waitPrefetch"方法将当前的GPRS（可能是CPU需要访问的内存区域）的[cpu.PC]位置的参数传递给该方法；
3. 通过结合当前的操作和条件，创建一个新的函数，该函数会执行给定的立即操作，并将结果返回给调用者。新函数中的操作会使用MMU，在获取到GPRS[cpu.PC]后，立即执行给定的立即操作，然后将GPRS[cpu.PC]的值更新为需要立即操作的值。

对于"ARMCoreThumb.prototype.constructB2"函数，它的作用类似于"constructB1"函数，但只接受需要立即执行的操作的参数，即不带条件的立即操作。它与"constructB1"函数的区别在于返回类型上，前者返回一个函数，后者返回一个空函数。


```js
ARMCoreThumb.prototype.constructB1 = function(immediate, condOp) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		if (condOp()) {
			gprs[cpu.PC] += immediate;
		}
	};
};

ARMCoreThumb.prototype.constructB2 = function(immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[cpu.PC] += immediate;
	};
};

```

这段代码定义了 `ARMCoreThumb` 类中的两个构造函数 `constructBIC` 和 `constructBL1`。

`constructBIC` 函数接收两个参数 `rd` 和 `rm`，并将它们作为参数传递给 `this.constructBIC` 函数内部。函数内部创建一个新的函数，这个新函数与 `this.cpu` 属性相关联，并且使用 `cpu.mmu.waitPrefetch` 方法来保证 `cpu` 对象的正确性，然后执行下列操作：

1. 通过 `gprs` 数组中的 `cpu.PC` 属性，将 `gprs[rm]` 的二进制表示读取出来。
2. 将 `rd` 参数设置为 `rm`。
3. 将 `cpu.cpsrN` 和 `cpu.cpsrZ` 属性设置为 `gprs[rd]` 的最高 31 位，以便在计算 `gprs[rm]` 的值时进行截断。
4. 将 `gprs[rm]` 的值左移 31 位，并将其与 `0xFFFFFFFF` 进行与运算，得到一个只有 0 或 1 的二进制字符串，作为 `gprs[cpu.PC]` 的值。
5. 将 `cpu.mmu.waitPrefetch` 方法与 `gprs[rd]` 挂起，以确保 `cpu.PC` 的正确性。

`constructBL1` 函数接收一个 `immediate` 参数，并将其作为参数传递给 `this.constructBL1` 函数内部。函数内部创建一个新的函数，这个新函数与 `this.cpu` 属性相关联，并且使用 `cpu.mmu.waitPrefetch` 方法来保证 `cpu` 对象的正确性，然后执行下列操作：

1. 通过 `gprs` 数组中的 `cpu.PC` 属性，将 `gprs[cpu.PC]` 读取出来。
2. 将 `immediate` 参数加上去，并将结果存储到 `gprs[cpu.PC]` 中。
3. `gprs[cpu.LR]` 的值被设置为 `cpu.PC` 加上 `immediate`。


```js
ARMCoreThumb.prototype.constructBIC = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = gprs[rd] & ~gprs[rm];
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

ARMCoreThumb.prototype.constructBL1 = function(immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[cpu.LR] = gprs[cpu.PC] + immediate;
	}
};

```

这两段代码是JavaScript中的函数，属于Object模型的继承。

ARMCoreThumb是Thumb这个动画游玩引擎中的一部分，暴露了一些与CPU相关的操作。

第一段代码定义了一个名为constructBL2的函数，接受一个参数immediate，表示是否立即执行操作。函数的作用是获取CPU实例，并创建一个名为function()的函数，用于在需要时执行CPU操作。

第二段代码定义了一个名为constructBX的函数，接受两个参数rd和rm，表示需要左移或右移的位数。函数的作用是获取CPU实例，并创建一个名为function()的函数，用于在需要时执行CPU操作。


```js
ARMCoreThumb.prototype.constructBL2 = function(immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var pc = gprs[cpu.PC];
		gprs[cpu.PC] = gprs[cpu.LR] + (immediate << 1);
		gprs[cpu.LR] = pc - 1;
	}
};

ARMCoreThumb.prototype.constructBX = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		cpu.switchExecMode(gprs[rm] & 0x00000001);
		var misalign = 0;
		if (rm == 15) {
			misalign = gprs[rm] & 0x00000002;
		}
		gprs[cpu.PC] = gprs[rm] & 0xFFFFFFFE - misalign;
	};
};

```

这段代码定义了一个名为 "ARMCoreThumb.prototype.constructCMN" 的函数，它接受两个参数 "rd" 和 "rm"，并将它们作为参数。函数内部创建了一个新的 "ARMCoreThumb" 对象，并将它作为 "constructCMN" 函数的第一个参数。

接下来，函数内部通过 "this.cpu" 访问到 "ARMCoreThumb" 对象的一个名为 "cpu" 的属性，并将它与第二个参数 "rm" 组合起来，创建一个新的 "ARMCoreThumb" 对象。

接下来，函数内部创建了一个新的 "function" 对象，并将其命名为 "constructCMN"。

函数内部调用了 "ARMCoreThumb" 对象的一个名为 "mmu" 的方法 "waitPrefetch"，将两个参数 "gprs[cpu.PC]" 和 "rd" 作为参数传入，并将 "gprs[rm]" 留空。

接下来，函数内部计算出了 "aluOut"，并将它作为参数传入 "cpu.cpsrN" 和 "cpu.cpsrZ" 方法的第一个参数。

然后，函数内部判断 "aluOut" 是否大于 0xFFFFFFFF，如果是，则将 "cpu.cpsrC" 设置为 1，并将 "cpu.cpsrN" 和 "cpu.cpsrZ" 设置为 0。如果 "aluOut" 不大于 0xFFFFFFFF，则执行以下操作：

1. 将 "gprs[rd]" 和 "gprs[rm]" 存储为 "aluOut" 的两个分频因子。
2. 将 "aluOut" 的二进制形式存储在 "gprs[cpu.PC]" 中。
3. 将 "aluOut" 的最高 31 位清零，并将 "gprs[rm]" 和 "gprs[rd]" 存储为 "aluOut" 的低 31 位。
4. 如果 "gprs[rm]" 和 "gprs[rd]" 存储的值相等，并且 "aluOut" 的二进制形式与它们存储的值相等，则设置 "cpu.cpsrV" 为 1，并将 "cpu.cpsrN" 和 "cpu.cpsrZ" 设置为 0。
5. 否则，设置 "cpu.cpsrV" 为 0，并将 "cpu.cpsrN" 和 "cpu.cpsrZ" 设置为 "aluOut" 的低 31 位。

最终，函数返回了一个新的 "ARMCoreThumb" 对象。


```js
ARMCoreThumb.prototype.constructCMN = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var aluOut = (gprs[rd] >>> 0) + (gprs[rm] >>> 0);
		cpu.cpsrN = aluOut >> 31;
		cpu.cpsrZ = !(aluOut & 0xFFFFFFFF);
		cpu.cpsrC = aluOut > 0xFFFFFFFF;
		cpu.cpsrV = (gprs[rd] >> 31) == (gprs[rm] >> 31) &&
					(gprs[rd] >> 31) != (aluOut >> 31) &&
					(gprs[rm] >> 31) != (aluOut >> 31);
	};
};

```

这两段代码是JavaScript中的函数，它们定义了两个名为"constructCMP1"和"constructCMP2"的函数。它们的作用是用来比较两个32位无符号整数的差值是否为0，并返回比较结果。

"constructCMP1"函数接受两个参数：一个32位无符号整数和一个即时（即时我们看到的是一个浮点数，实际执行的是整数计算）。这两个参数分别存储在"cpu.gprs"数组中。"constructCMP1"函数返回一个函数，这个函数会执行以下操作：

1. 获取CPU实例并复制到函数内部。
2. 调用CPU的"mmu.waitPrefetch"方法，该方法要求CPU的内存管理单元预先加载指定寄存器或内存区域的数据。在这里，我们使用"gprs[cpu.PC]"来获取要比较的第一个32位无符号整数的寄存器值。
3. 计算两个32位无符号整数的差值，我们将这个差值存储在"an"变量中。
4. 将"an"的二进制位向左移动31位，得到"dn"的值，存储在"cpu.cpsrN"变量中。
5. 判断"dn"是否为0或者"an"是否为1，如果是，那么这两个32位无符号整数的差值不为0，即这两个整数至少有一个不是0。
6. 将"cpu.cpsrC"的值设置为1，因为第一个32位无符号整数减去第二个32位无符号整数的结果为0，即这两个整数相等。"cpu.cpsrV"的值设置为1，因为两个32位无符号整数的差值不是0，即这两个整数至少有一个不是0。
7. 调用函数内部定义的"cpu.mmu.waitPrefetch"方法，要求CPU内存管理单元加载第一个32位无符号整数的下一页内存。

"constructCMP2"函数接受两个参数：一个32位无符号整数和一个整数。这两个参数分别存储在"cpu.gprs"数组中。"constructCMP2"函数返回一个函数，这个函数会执行以下操作：

1. 获取CPU实例并复制到函数内部。
2. 调用CPU的"mmu.waitPrefetch"方法，该方法要求CPU的内存管理单元预先加载指定寄存器或内存区域的数据。在这里，我们使用"gprs[cpu.PC]"来获取要比较的第一个32位无符号整数的寄存器值。
3. 调用"ARMCoreThumb.prototype.constructCMP1"函数，并将rd和rm作为参数传入，得到两个32位无符号整数的差值。
4. 将"d"的值存储在"cpu.cpsrN"变量中。
5. 将"rm"的值存储在"cpu.cpsrM"变量中。
6. 调用函数内部定义的"cpu.mmu.waitPrefetch"方法，要求CPU内存管理单元加载第二个32位无符号整数的下一页内存。


```js
ARMCoreThumb.prototype.constructCMP1 = function(rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var aluOut = gprs[rn] - immediate;
		cpu.cpsrN = aluOut >> 31;
		cpu.cpsrZ = !(aluOut & 0xFFFFFFFF);
		cpu.cpsrC = (gprs[rn] >>> 0) >= immediate;
		cpu.cpsrV = (gprs[rn] >> 31) && ((gprs[rn] ^ aluOut) >> 31);
	};
}

ARMCoreThumb.prototype.constructCMP2 = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = gprs[rd];
		var m = gprs[rm];
		var aluOut = d - m;
		var an = aluOut >> 31;
		var dn = d >> 31;
		cpu.cpsrN = an;
		cpu.cpsrZ = !(aluOut & 0xFFFFFFFF);
		cpu.cpsrC = (d >>> 0) >= (m >>> 0);
		cpu.cpsrV = dn != (m >> 31) && dn != an;
	};
};

```

这两段代码是 `ARMCoreThumb` 类中的两个静态方法，用于执行 CPU 的内存操作。

`constructCMP3` 函数的目的是比较两个整数的大小，返回一个比较结果。它接收两个参数 `rd` 和 `rm`，分别代表两个被比较的整数，然后执行一系列 CPU 指令来将它们相减，并将结果保存到 `cpu.cpsrN` 和 `cpu.cpsrZ` 变量中。最后，该函数返回一个布尔值，表示两个整数的大小关系。

`constructEOR` 函数的目的是执行一个字节数的异或操作，并返回结果。它接收两个参数 `rd` 和 `rm`，分别代表两个要异或操作的字节数，然后执行一系列 CPU 指令来执行异或操作，并将结果保存到 `cpu.cpsrN` 变量中。最后，该函数返回一个布尔值，表示两个字节数是否异或。


```js
ARMCoreThumb.prototype.constructCMP3 = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var aluOut = gprs[rd] - gprs[rm];
		cpu.cpsrN = aluOut >> 31;
		cpu.cpsrZ = !(aluOut & 0xFFFFFFFF);
		cpu.cpsrC = (gprs[rd] >>> 0) >= (gprs[rm] >>> 0);
		cpu.cpsrV = ((gprs[rd] ^ gprs[rm]) >> 31) && ((gprs[rd] ^ aluOut) >> 31);
	};
};

ARMCoreThumb.prototype.constructEOR = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = gprs[rd] ^ gprs[rm];
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这段代码定义了一个名为`ARMCoreThumb.prototype.constructLDMIA`的函数，它接收两个参数`rs`和`rn`。

这个函数的作用是在ARMCoreThumb继承的基础上，实现了LDMIA(低电量内存访问)功能。LDMIA允许在ARMCoreThumb中更安全地访问低电量内存，避免了可能导致程序崩溃或死机的问题。

函数内部，首先通过`this.cpu`访问当前ARMCoreThumb实例的`cpu`属性，然后通过`cpu.gprs`访问`cpu`的`gprs`属性。接着定义了一个新的函数，它接收一个空括号，表示没有参数。

在函数内部，使用嵌套循环遍历参数`rs`和`rn`。循环变量`m`从0x01开始，每次增加4，用来二进制位移，遍历`rs`的位。如果位`rs`与`m`相与，则执行以下操作：

1. 通过`gprs[cpu.PC]`读取低电量内存中字节的第`cpu.PC`位，并将其存储在`gprs`数组中。
2. 通过`cpu.mmu.load32(address)`将读取到的字节加载到`cpu.mmu`中。
3. 通过`address += 4`更新`address`的值，以便在下一轮循环中继续访问下一个位。
4. 通过`++total`增加累计的电量。

最后，在循环结束后，通过`cpu.mmu.waitMulti32(address, total)`函数，等待`cpu.mmu`中的内存完成写回。

如果`rs`的位`1`和`rs`的位`0`都为1，则表示`rs`中的字节已经被加载到内存中，不需要再进行写回。在这种情况下，可以通过下划线`_`来标记该代码块，以避免代码冗长。


```js
ARMCoreThumb.prototype.constructLDMIA = function(rn, rs) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var address = gprs[rn];
		var total = 0;
		var m, i;
		for (m = 0x01, i = 0; i < 8; m <<= 1, ++i) {
			if (rs & m) {
				gprs[i] = cpu.mmu.load32(address);
				address += 4;
				++total;
			}
		}
		cpu.mmu.waitMulti32(address, total);
		if (!((1 << rn) & rs)) {
			gprs[rn] = address;
		}
	};
};

```

这两段代码是定义了 `ARMCoreThumb` 类的两个构造函数 `constructLDR1` 和 `constructLDR2`。

`constructLDR1` 函数的实现如下：

```jsjavascript
ARMCoreThumb.prototype.constructLDR1 = function(rd, rn, immediate) {
 var cpu = this.cpu;
 var gprs = cpu.gprs;
 return function() {
   cpu.mmu.waitPrefetch(gprs[cpu.PC]);
   var n = gprs[rn] + immediate;
   gprs[rd] = cpu.mmu.load32(n);
   cpu.mmu.wait32(n);
   ++cpu.cycles;
 };
};
```

这个构造函数接收三个参数：

* `rd`：类的主机（LDR1）的内存区域（0）。
* `rn`：目标（LDR1）的内存区域（1）。
* `immediate`：是否立即加载，如果为 `true`，则需要在函数内部手动初始化内存区域。

这个函数的作用是加载类的主机 LDR1 并设置其第二个参数的值，然后激活内存区域。

`constructLDR2` 函数的实现如下：

```jsjavascript
ARMCoreThumb.prototype.constructLDR2 = function(rd, rn, rm) {
 var cpu = this.cpu;
 var gprs = cpu.gprs;
 return function() {
   cpu.mmu.waitPrefetch(gprs[cpu.PC]);
   gprs[rd] = cpu.mmu.load32(gprs[rn] + gprs[rm]);
   cpu.mmu.wait32(gprs[rn] + gprs[rm]);
   ++cpu.cycles;
 };
};
```

这个构造函数接收三个参数：

* `rd`：类的主机（LDR1）的内存区域（0）。
* `rn`：目标（LDR1）的内存区域（1）。
* `rm`：内存区域（2）。

这个函数的作用是加载类的主机 LDR1 并设置其第二个参数的值，然后激活内存区域。


```js
ARMCoreThumb.prototype.constructLDR1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var n = gprs[rn] + immediate;
		gprs[rd] = cpu.mmu.load32(n);
		cpu.mmu.wait32(n);
		++cpu.cycles;
	};
};

ARMCoreThumb.prototype.constructLDR2 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.load32(gprs[rn] + gprs[rm]);
		cpu.mmu.wait32(gprs[rn] + gprs[rm]);
		++cpu.cycles;
	}
};

```

这两段代码是定义了两个名为 `constructLDR3` 和 `constructLDR4` 的函数，它们都属于 `ARMCoreThumb` 类的原型中。

这两段代码的主要作用是实现低层级别的硬件抽象，即通过 `gprs` 数组来访问 CPU 中的内存，并实现快速的内存读取。通过这两个函数，可以在 `ARMCoreThumb` 类中更方便地使用低层级的硬件抽象。

具体来说，这两段代码实现了以下功能：

1. `constructLDR3(rd, immediate)`:

这个函数接收两个参数：一个 `rd` 表示需要读取的内存区域，一个 `immediate` 表示一个立即操作数，用于指定在 `rd` 内存区域中的偏移量。

函数的主要实现步骤如下：

- 从 `ARMCoreThumb` 类的实例中，获取当前的 CPU 实例和缓存器的上下文。
- 使用缓存器中的 CPU 实例的 `gprs` 属性的值，从内存中读取 `rd` 内存区域对应的位置，并加上 `immediate` 参数指定的立即操作数。
- 将读取到的数据长度转换为字节数，并将其除以 8，得到一个商 `gprsSize`，表示 `gprs` 数组的大小。
- 使用 CPU 实例的 `mmu` 属性的 `waitPrefetch` 方法，轮询并等待 `gprs` 数组中 `gprsSize` 数量个元素完成预读取，然后返回对应的预读取结果。
- 使用 CPU 实例的 `mmu` 属性的 `load32` 方法，从 `gprs` 数组中读取 `rd` 内存区域对应的位置，并加上 `immediate` 参数指定的立即操作数，得到一个加载到的 32 位整数。
- 将加载到的 32 位整数除以 8，得到一个商 ` cycles`。
- 增加当前循环计数器 `cpu.cycles`。

2. `constructLDR4(rd, immediate)`:

这个函数与 `constructLDR3` 类似，只是实现了一些变化：

- 与 `constructLDR3` 不同，这个函数需要先创建一个 `constructLDR4` 的实例，然后再调用它的函数。
- 这个函数接收两个参数：一个 `rd` 表示需要读取的内存区域，一个 `immediate` 表示一个立即操作数，用于指定在 `rd` 内存区域中的偏移量。
- 使用 `ARMCoreThumb` 类的 `constructLDR4` 函数创建一个新的 `ARMCoreThumb` 实例，并设置其 `immediate` 参数为 `rd` 和 `immediate` 组成的偏移量，使得 `constructLDR4` 函数可以正确地读取 `rd` 内存区域。
- 使用 `constructLDR4` 函数的 `mmu.waitPrefetch` 方法，轮询并等待 `rd` 内存区域对应的位置完成预读取，然后返回对应的预读取结果。
- 使用 `constructLDR4` 函数的 `mmu.load32` 方法，从 `rd` 内存区域对应的位置，加上 `immediate` 参数指定的立即操作数，得到一个加载到的 32 位整数。
- 将加载到的 32 位整数除以 8，得到一个商 ` cycles`。
- 增加当前循环计数器 `cpu.cycles`。


```js
ARMCoreThumb.prototype.constructLDR3 = function(rd, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.load32((gprs[cpu.PC] & 0xFFFFFFFC) + immediate);
		cpu.mmu.wait32(gprs[cpu.PC]);
		++cpu.cycles;
	};
};

ARMCoreThumb.prototype.constructLDR4 = function(rd, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.load32(gprs[cpu.SP] + immediate);
		cpu.mmu.wait32(gprs[cpu.SP] + immediate);
		++cpu.cycles;
	};
};

```

这段代码定义了两个名为 `constructLDRB1` 和 `constructLDRB2` 的函数，用于在 thumb 代码执行期间加载依赖的字节数组。

`constructLDRB1` 函数接收三个参数：一个字节数组 `rd`、一个字节数组 `rn` 和一个布尔值 `immediate`。函数返回一个函数，该函数会在执行期间执行以下操作：

1. 获取 CPU 和 GPRS 缓存中的数据。
2. 将 `rd` 字节数组的下一个元素和 `immediate` 参数中的值相加，并将结果存储到 `n` 中。
3. 从 CPU 中的内存中加载字节数组 `gprs[cpu.PC]` 中的值，并将其存储到指定的位置。
4. 从 CPU 中的内存中加载字节数组 `gprs[rn]` 和 `gprs[rm]` 中的值，并将它们存储到指定的位置。
5. 将加载的字节数组数量加一。

`constructLDRB2` 函数与 `constructLDRB1` 类似，只是使用不同的字节数组索引。函数接收三个参数：一个字节数组 `rd`、一个字节数组 `rn` 和一个字节数组 `rm`。函数返回一个函数，该函数会在执行期间执行以下操作：

1. 获取 CPU 和 GPRS 缓存中的数据。
2. 从 CPU 中的内存中加载字节数组 `gprs[cpu.PC]` 中的值，并将其存储到指定的位置。
3. 从 CPU 中的内存中加载字节数组 `gprs[rn]` 和 `gprs[rm]` 中的值，并将它们存储到指定的位置。
4. 将加载的字节数组数量加一。


```js
ARMCoreThumb.prototype.constructLDRB1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		var n = gprs[rn] + immediate;
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.loadU8(n);
		cpu.mmu.wait(n);
		++cpu.cycles;
	};
};

ARMCoreThumb.prototype.constructLDRB2 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.loadU8(gprs[rn] + gprs[rm]);
		cpu.mmu.wait(gprs[rn] + gprs[rm]);
		++cpu.cycles;
	};
};

```

这段代码定义了两个名为 `constructLDRH1` 和 `constructLDRH2` 的函数，它们都属于 `ARMCoreThumb` 类。

这两个函数都接收三个参数：`rd`、`rn` 和 `rm`，分别表示右侧寄存器的指数、基地址寄存器的指数和内存模块的数量。这些参数都是 8 位整数，分别表示右侧寄存器和基地址寄存器。

这两个函数都返回一个函数，这个函数会在 CPU 上执行一次。在函数执行时，它首先会获取右侧寄存器和基地址寄存器中的值，并将它们存储到本地。然后，它会使用 CPU 的内存管理单元(MMU)加载目的值，并将其存储到相应的寄存器中。此外，它还会在本地循环计数器 `cpu.cycles` 上加 1。

这两个函数的实现看起来是为了在执行某些操作时，自动初始化右侧寄存器和基地址寄存器，并确保在执行操作时 CPU 的 cycles 计数器总是为 1。


```js
ARMCoreThumb.prototype.constructLDRH1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		var n = gprs[rn] + immediate;
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.loadU16(n);
		cpu.mmu.wait(n);
		++cpu.cycles;
	};
};

ARMCoreThumb.prototype.constructLDRH2 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.loadU16(gprs[rn] + gprs[rm]);
		cpu.mmu.wait(gprs[rn] + gprs[rm]);
		++cpu.cycles;
	};
};

```

这两段代码定义了 `ARMCoreThumb.prototype.constructLDRSB` 和 `ARMCoreThumb.prototype.constructLDRSH` 函数，它们的作用是加载两个整数的低字节并将其存储在指定的 GPRS 变量中。

`constructLDRSB` 函数的实现是通过 `cpu.mmu.waitPrefetch` 函数来确保 CPU 内的内存已经被加载到 MMA 缓存中，然后通过 `gprs[cpu.PC]` 获取要加载的低字节，并使用 `cpu.mmu.load8` 函数将其加载到指定的 GPRS 变量中。最后，通过 `cpu.mmu.wait` 函数来等待缓存中的数据加载完成，并增加 CPU 循环计数器的循环次数，以便在循环结束后返回结果。

`constructLDRSH` 函数与 `constructLDRSB` 函数类似，只是它的功能是将两个整数的低字节存储在两个 GPRS 变量中。它的实现是通过 `cpu.mmu.waitPrefetch` 函数来确保 CPU 内的内存已经被加载到 MMA 缓存中，然后通过循环遍历每个 GPRS 变量，并将要加载的低字节存储到对应的变量中。最后，通过 `cpu.mmu.wait` 函数来等待缓存中的数据加载完成，并增加 CPU 循环计数器的循环次数，以便在循环结束后返回结果。


```js
ARMCoreThumb.prototype.constructLDRSB = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.load8(gprs[rn] + gprs[rm]);
		cpu.mmu.wait(gprs[rn] + gprs[rm]);
		++cpu.cycles;
	};
};

ARMCoreThumb.prototype.constructLDRSH = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = cpu.mmu.load16(gprs[rn] + gprs[rm]);
		cpu.mmu.wait(gprs[rn] + gprs[rm]);
		++cpu.cycles;
	};
};

```

这段代码定义了一个名为"ARMCoreThumb.prototype.constructLSL1"的函数，它接受三个参数：rd、rm和immediate。

这个函数的作用是在Thumb类中实现了一个对CPU的8位LSL(高精度整数减法)操作。它通过以下步骤来实现：

1. 获取目标CPU实例并将其存储在变量cpu中。
2. 获取CPU的8位通用寄存器并将其存储在变量gprs中。
3. 如果immediate参数为0，则执行以下操作：
  1. 等待CPU的通用寄存器gprs[cpu.PC]的完成预取操作。
  2. 如果immediate参数为1，则执行以下操作：
  3. 交换gprs[rd]和gprs[rm]的值。
  4. 将32位无符号整数immediate左移31位，并将其与gprs[rd]进行按位与操作，得到新的gprs[rd]。
  5. 将新的gprs[rd]左移31位，并将其与0进行按位与操作，得到gprs[rd]的低31位，存储在变量cpu.cpsrN中。
  6. 清除gprs[rd]中高31位的位，存储在变量cpu.cpsrZ中。
4. 在函数内部，通过gprs[rd]的值计算出cpu.cpsrC的值。
5. 在计算cpu.cpsrC的值时，将immediate左移31位，并将其与1进行按位与操作，得到一个新的32位无符号整数immediate'。
  然后使用按位与操作将immediate'与gprs[rm]的值进行按位与操作，得到一个新的32位无符号整数immediate''。
  最后，将immediate''与gprs[rm]的值进行按位与操作，得到cpu.cpsrC的值。
  
如果immediate参数为0，则不会执行第3步操作，也就是不会交换gprs[rd]和gprs[rm]的值。在这种情况下，函数的行为与预期的行为不一致，可能会导致代码不可用。


```js
ARMCoreThumb.prototype.constructLSL1 = function(rd, rm, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		if (immediate == 0) {
			gprs[rd] = gprs[rm];
		} else {
			cpu.cpsrC = gprs[rm] & (1 << (32 - immediate));
			gprs[rd] = gprs[rm] << immediate;
		}
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这段代码定义了一个名为`ARMCoreThumb.prototype.constructLSL2`的函数，它实现了LSL2（流水线存储器）的创建。

LSL2是流水线存储器的一个变种，它在流水线中执行指令。通过使用两个8位的RS信号来指示正在执行的指令，可以确保在流水线中按顺序执行指令。

该函数的具体实现包括以下步骤：

1. 检查输入的参数，确保它们是有效的。
2. 初始化通用寄存器，并将输入的指令行号存储到通用寄存器中。
3. 准备输入寄存器，根据输入的指令行号设置输入寄存器的相应位。
4. 根据输入的指令行号执行指令，并更新通用寄存器。
5. 如果输入的指令行号是32位，则执行完指令后，将通用寄存器的内容保存回到输入寄存器中。
6. 如果输入的指令行号是32位，则执行指令过程中，将输入寄存器的位清空。


```js
ARMCoreThumb.prototype.constructLSL2 = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var rs = gprs[rm] & 0xFF;
		if (rs) {
			if (rs < 32) {
				cpu.cpsrC = gprs[rd] & (1 << (32 - rs));
				gprs[rd] <<= rs;
			} else {
				if (rs > 32) {
					cpu.cpsrC = 0;
				} else {
					cpu.cpsrC = gprs[rd] & 0x00000001;
				}
				gprs[rd] = 0;
			}
		}
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这段代码定义了一个名为 "ARMCoreThumb.prototype.constructLSR1" 的函数，它的作用是在 Thumb 类中实现了一个 LSR1 指令的功能。

具体来说，这个函数接受三个参数：rd、rm 和 immediate，其中rd 和 rm 是两个整数，immediate 是一个布尔值，表示是否为立即执行。函数返回一个函数，这个函数会在执行 LSR1 指令时执行。

在函数内部，首先通过 this.cpu 获取当前的 CPU 实例，然后通过 cpu.gprs 获取到它所控制的 GPU 中的寄存器情况。接着定义了一个名为 "function()" 的函数体，这个函数体会在每次执行 LSR1 指令时被调用。

在 function() 函数体中，首先通过 cpu.mmu.waitPrefetch 方法等待内存中的指令准备就绪。接着根据参数 immediate 的值，判断是否为立即执行，如果是，则执行 LSR1 指令，否则，先将 immediate 指定的二进制位清零，然后将立即指定的二进制位与 GPU 寄存器中的字节的和与指定的字节的和进行按位与操作，再将得到的结果与指定的字节的和进行按位或操作，最后将 CPU 中的字节的和与指定的字节进行按位与操作，得到的结果再与 CPU 中的字节的和进行按位与操作，最后输出一个布尔值，表示是否执行了 LSR1 指令。


```js
ARMCoreThumb.prototype.constructLSR1 = function(rd, rm, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		if (immediate == 0) {
			cpu.cpsrC = gprs[rm] >> 31;
			gprs[rd] = 0;
		} else {
			cpu.cpsrC = gprs[rm] & (1 << (immediate - 1));
			gprs[rd] = gprs[rm] >>> immediate;
		}
		cpu.cpsrN = 0;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
}

```

这段代码定义了一个名为 `ARMCoreThumb.prototype.constructLSR2` 的函数，它接收两个参数 `rd` 和 `rm`，并将它们作为参数传递给构造函数。

该函数返回一个函数，该函数执行以下操作：

1. 等待 CPU 内存中的页面缓存，确保该页面的页号在 CPU 缓存中。
2. 计算 `rs`，即 `rm` 中高位 8 位或 32 位减 1。
3. 如果 `rs` 小于 32，那么执行以下操作：
	1. 如果 `rs` 小于 16，将 `rs` 存储在 `cpu.cpsrC` 寄存器中，并将 `gprs[cpu.PC + rs]` 和 `rs` 存储在 `cpu.cpsrN` 和 `cpu.cpsrZ` 寄存器中。
	2. 如果 `rs` 大于或等于 16，执行以下操作：
		1. 如果 `rs` 大于 16，将 `cpu.cpsrC` 设置为 0。
		2. 如果 `rs` 大于 32，执行以下操作：
			1. 将 `cpu.cpsrN` 和 `cpu.cpsrZ` 设置为 `rs` 左移 31 位并取反，存储在 `cpu.cpsrC` 和 `cpu.cpsrN` 寄存器中。
			2. 将 `rs` 存储在 `cpu.cpsrC` 寄存器中。

`ARMCoreThumb.prototype.constructLSR2` 函数用于初始化两个 8 位整数 `rd` 和 `rm`，并将它们传递给构造函数。函数返回的函数用于执行 CPU 缓存器的刷新和计算，以准备执行指令流水线中的指令。


```js
ARMCoreThumb.prototype.constructLSR2 = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var rs = gprs[rm] & 0xFF;
		if (rs) {
			if (rs < 32) {
				cpu.cpsrC = gprs[rd] & (1 << (rs - 1));
				gprs[rd] >>>= rs;
			} else {
				if (rs > 32) {
					cpu.cpsrC = 0;
				} else {
					cpu.cpsrC = gprs[rd] >> 31;
				}
				gprs[rd] = 0;
			}
		}
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这两段代码是JavaScript中的函数，属于"ARMCoreThumb"类的一个静态成员函数。"constructMOV1"函数接受两个参数，一个是寄存器变量"rn"，另一个是布尔值"immediate"，它们都会被转换成布尔值后作为函数参数传递给"ARMCoreThumb"类的一个构造函数函数。

这两段代码会向CPU的MMU发送两个消息，第一个消息是设置寄存器"gprs[cpu.PC]"的立即访问权限，然后设置CPU中的状态寄存器"cpsrN"为即将要发送给PC的立即值左移31位后的值，同时将"cpsrZ"设置为false。第二个消息是设置寄存器"gprs[rd]"的立即访问权限，并将d设置为gprs[rn]左移31位后的值。


```js
ARMCoreThumb.prototype.constructMOV1 = function(rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rn] = immediate;
		cpu.cpsrN = immediate >> 31;
		cpu.cpsrZ = !(immediate & 0xFFFFFFFF);
	};
};

ARMCoreThumb.prototype.constructMOV2 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = gprs[rn];
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = 0;
		cpu.cpsrV = 0;
		gprs[rd] = d;
	};
};

```

这段代码定义了两个名为`constructMOV3`和`constructMUL`的函数，属于`ARMCoreThumb`类的实例。

这两个函数用于实现MOV3和MUL指令。在ARMCoreThumb中，这些指令并不是通过`fetch`指令并使用CPU的MMU来实现的，而是通过软件模拟器实现，因此这些函数的实际实现与硬件无关。

具体来说，这两个函数分别实现了以下功能：

1. `constructMOV3(rd, rm)`函数：

这个函数接收两个整数参数`rd`和`rm`，并将它们设置为整数类型。然后，该函数返回一个函数，这个函数会在CPU的MMU中等待缓存器的准备，然后将两个参数的值复制到缓存器中，并返回一个空函数。

2. `constructMUL(rd, rm)`函数：

这个函数与`constructMOV3`类似，但只接收一个整数参数`rm`，然后返回一个函数。这个函数的作用与`constructMOV3`函数相反，会将两个整数的乘积结果复制到缓存器中，并检查是否发生了溢出。

由于这两个函数的实际实现与硬件无关，因此它们的实现可以非常简单，只需要在一些关键点上进行一些辅助代码，比如在函数声明时记录参数类型，提供一些额外的错误处理等等。


```js
ARMCoreThumb.prototype.constructMOV3 = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = gprs[rm];
	};
};

ARMCoreThumb.prototype.constructMUL = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		cpu.mmu.waitMul(gprs[rm]);
		if ((gprs[rm] & 0xFFFF0000) && (gprs[rd] & 0xFFFF0000)) {
			// Our data type is a double--we'll lose bits if we do it all at once!
			var hi = ((gprs[rd] & 0xFFFF0000) * gprs[rm]) & 0xFFFFFFFF;
			var lo = ((gprs[rd] & 0x0000FFFF) * gprs[rm]) & 0xFFFFFFFF;
			gprs[rd] = (hi + lo) & 0xFFFFFFFF;
		} else {
			gprs[rd] *= gprs[rm];
		}
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这段代码定义了两个函数：`constructMVN` 和 `constructNEG`。这两个函数都属于 `ARMCoreThumb` 类的原型。

`constructMVN` 函数的实现比较复杂，但可以理解为：

1. 检查 `rd` 和 `rm` 两个参数，如果 `rd` 是 `rm` 的负数，那么这两个函数根据 `ARMCoreThumb.prototype.constructNEG` 函数计算出的结果。
2. 如果 `rd` 和 `rm` 都不是 `ARMCoreThumb.prototype.constructNEG` 函数计算出的结果，那么生成一个新函数，这个新函数和 `constructNEG` 函数的实现比较类似，但会带有一个参数 `d`，这个参数表示 `rm`。
3. 生成新函数后，调用这个新函数。

`constructNEG` 函数的实现比较简单，但可以理解为：

1. 检查 `rd` 和 `rm` 两个参数，如果 `rd` 是 `ARMCoreThumb.prototype.constructNEG` 函数计算出的结果，那么生成一个新函数，这个新函数和 `constructNEG` 函数的实现比较类似，但会带有一个参数 `d`，这个参数表示 `rm`。
2. 生成新函数后，调用这个新函数。

这两个函数可能会在后续的处理中用到，尤其在涉及 `NEG` 类型的数据处理。


```js
ARMCoreThumb.prototype.constructMVN = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = ~gprs[rm];
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

ARMCoreThumb.prototype.constructNEG = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = -gprs[rm];
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = 0 >= (d >>> 0);
		cpu.cpsrV = (gprs[rm] >> 31) && (d >> 31);
		gprs[rd] = d;
	};
};

```

这段代码定义了两个函数：`ARMCoreThumb.prototype.constructORR` 和 `ARMCoreThumb.prototype.constructPOP`。这两个函数用于在 Thumb 类中实现 `Object.constructor` 函数。

`constructORR` 函数接收两个参数 `rd` 和 `rm`，它们分别表示两个整数，用于初始化 CPU 的寄存器状态。函数内部创建一个新的函数，该函数实现了 `Object.constructor` 函数，但是仅在 `rd` 和 `rm` 都为 0 时才会调用，否则会执行以下操作：

1. 读取 CPU 的寄存器状态并保存到 `cpu.gprs` 数组中。
2. 将 `rd` 和 `rm` 的二进制位与 `gprs[cpu.PC]` 进行按位与操作，并将结果保存到 `cpu.cpsrN` 和 `cpu.cpsrZ` 变量中。
3. 调用 `cpu.mmu.waitPrefetch` 函数来预读取 `gprs[cpu.PC]` 所代表的数据，然后将其赋值给 `cpu.cpsrN` 和 `cpu.cpsrZ` 变量。
4. 在调用 `Object.constructor` 函数时，如果 `rd` 或 `rm` 中有一个为 1，则执行以下操作：
  1. 读取 `gprs[cpu.PC]` 所代表的数据，并将其保存到 `gprs[rs & m]` 变量中。
  2. 将 `gprs[rm]` 和 `gprs[rs & m]` 合并为一个新的数组，并将其保存到 `gprs[cpu.PC]` 变量中。
  3. 增加 `cpu.cycles` 计数器的值。
  4. 调用 `cpu.mmu.waitPrefetch` 函数来预读取 `gprs[cpu.SP]` 所代表的数据，然后将其赋值给 `gprs[cpu.SP]` 变量。

`constructPOP` 函数与 `constructORR` 函数类似，但是它需要传递两个参数 `rs` 和 `rm`，分别表示两个整数。函数内部创建一个新的函数，该函数实现了 `Object.constructor` 函数，但是仅在 `rs` 和 `rm` 都为 0 时才会调用，否则会执行以下操作：

1. 读取 CPU 的寄存器状态并保存到 `cpu.gprs` 数组中。
2. 将 `rs` 和 `rm` 的二进制位与 `gprs[cpu.PC]` 进行按位与操作，并将结果保存到 `cpu.cpsrN` 和 `cpu.cpsrZ` 变量中。
3. 调用 `cpu.mmu.waitPrefetch` 函数来预读取 `gprs[cpu.SP]` 所代表的数据，然后将其赋值给 `gprs[cpu.SP]` 变量。
4. 在调用 `Object.constructor` 函数时，如果 `rs` 或 `rm` 中有一个为 1，则执行以下操作：
  1. 读取 `gprs[cpu.PC]` 所代表的数据，并将其保存到 `gprs[rs & m]` 变量中。
  2. 将 `gprs[rm]` 和 `gprs[rs & m]` 合并为一个新的数组，并将其保存到 `gprs[cpu.PC]` 变量中。
  3. 增加 `cpu.cycles` 计数器的值。
  4. 调用 `cpu.mmu.waitPrefetch` 函数来预读取 `gprs[cpu.SP]` 所代表的数据，然后将其赋值给 `gprs[cpu.SP]` 变量。


```js
ARMCoreThumb.prototype.constructORR = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		gprs[rd] = gprs[rd] | gprs[rm];
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

ARMCoreThumb.prototype.constructPOP = function(rs, r) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		++cpu.cycles;
		var address = gprs[cpu.SP];
		var total = 0;
		var m, i;
		for (m = 0x01, i = 0; i < 8; m <<= 1, ++i) {
			if (rs & m) {
				cpu.mmu.waitSeq32(address);
				gprs[i] = cpu.mmu.load32(address);
				address += 4;
				++total;
			}
		}
		if (r) {
			gprs[cpu.PC] = cpu.mmu.load32(address) & 0xFFFFFFFE;
			address += 4;
			++total;
		}
		cpu.mmu.waitMulti32(address, total);
		gprs[cpu.SP] = address;
	};
};

```

该代码定义了一个名为 "ARMCoreThumb.prototype.constructPUSH" 的函数，它是 "ARMCoreThumb" 类的一个实例方法。

该函数接收两个参数 "rs" 和 "r"，它们是整数类型的参数。

该函数的作用是执行一个 "PUSH" 操作，将两个整数的值异或后结果存储到指定的内存位置，并确保在存储之前将所存储的值清零。

该函数的实现包括以下几个步骤：

1. 根据 CPU 类型和特定的寄存器，获取要存储的内存位置的地址。
2. 计算要存储在内存中的值，并将其存储在指定的寄存器中。
3. 如果参数 "r" 是 true，则执行以下操作：
a. 从指定的内存位置中读取值并存储在指定的寄存器中。
b. 将读取的值异或要存储的值，并将其存储在指定的寄存器中。
c. 将要存储的值减去 4。
d. 递增变量 "total"，以便在存储完值后计算出总和。
4. 对于每个内存位置 "m"，根据参数 "rs" 是否为 1，执行以下操作：
a. 如果 "rs" 是 1，则执行以下操作：
i. 从指定的内存位置中读取值并存储在指定的寄存器中。
ii. 将读取的值异或要存储的值，并将其存储在指定的寄存器中。
iii. 将要存储的值减去 4。
iv. 递增变量 "m"，以便下一次只读取到下一个内存位置。
b. 如果 "rs" 不是 1，则不执行操作 a。
5. 在所有内存位置都存储完值后，通过调用 "cpu.mmu.waitMulti32" 函数，等待从指定的内存位置得到的总和缓存到指定的寄存器中。
6. 最后，通过调用 "gprs" 数组中 "gprs[cpu.SP]" 获取存储的内存位置的地址，将存储的值存储回指定的内存位置。


```js
ARMCoreThumb.prototype.constructPUSH = function(rs, r) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		var address = gprs[cpu.SP] - 4;
		var total = 0;
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		if (r) {
			cpu.mmu.store32(address, gprs[cpu.LR]);
			address -= 4;
			++total;
		}
		var m, i;
		for (m = 0x80, i = 7; m; m >>= 1, --i) {
			if (rs & m) {
				cpu.mmu.store32(address, gprs[i]);
				address -= 4;
				++total;
				break;
			}
		}
		for (m >>= 1, --i; m; m >>= 1, --i) {
			if (rs & m) {
				cpu.mmu.store32(address, gprs[i]);
				address -= 4;
				++total;
			}
		}
		cpu.mmu.waitMulti32(address, total);
		gprs[cpu.SP] = address + 4;
	};
};

```

这段代码定义了一个名为 `ARMCoreThumb.prototype.constructROR` 的函数，用于在模2 shadow追踪模式下，从指定的寄存器 `rd` 和寄存器 `rm` 中读取一个 32 位有符号整数。

该函数的实现分为两个部分：

1. 读取指定寄存器 `rd` 和 `rm` 的值，并将它们存储在一个 `cpu` 对象的一个全局变量 `cpu` 中。

2. 通过调用 `cpu.mmu.waitPrefetch` 函数，使 CPU 中的内存单元处于预读状态，以便在下一个时钟周期到来时能够被重新启用。

3. 通过 `gprs` 数组访问器从 CPU 中的内存中读取指定的寄存器 `rm`，并将其存储在 `rs` 变量中。

4. 如果读取到的寄存器 `rs` 的值为非零值，则执行以下操作：

  a. 通过 `cpu.cpsrC` 寄存器获取一个 32 位二进制整数，并将其左移 4 位，再与 `rm` 寄存器中的值进行按位与运算，得到一个 32 位有符号整数。

  b. 将得到的结果存储在 `cpu.cpsrN` 寄存器中。

  c. 将 `cpu.cpsrZ` 标志设置为 `1`，以指示 read 操作成功。

5. 通过调用 `cpu.cpsrC` 和 `cpu.cpsrN` 函数，将计算得到的符号位移量设置为 `rs` 变量的最高 3 位，即 `rs & 0xFF`。

6. 调用该函数后，它将自动更新 CPU 中的内存单元，并在下一个时钟周期到来时重新启用它们。


```js
ARMCoreThumb.prototype.constructROR = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var rs = gprs[rm] & 0xFF;
		if (rs) {
			var r4 = rs & 0x1F;
			if (r4 > 0) {
				cpu.cpsrC = gprs[rd] & (1 << (r4 - 1));
				gprs[rd] = (gprs[rd] >>> r4) | (gprs[rd] << (32 - r4));
			} else {
				cpu.cpsrC = gprs[rd] >> 31;
			}
		}
		cpu.cpsrN = gprs[rd] >> 31;
		cpu.cpsrZ = !(gprs[rd] & 0xFFFFFFFF);
	};
};

```

这段代码定义了一个名为 `ARMCoreThumb.prototype.constructSBC` 的函数，属于 "ARMCoreThumb" 类。

这个函数接收两个参数 `rd` 和 `rm`，分别表示 "remap" 和 "remap"，用于在 ARMCoreThumb 类中映射到对应的函数和变量。

函数内部，首先通过 `this.cpu` 访问当前的 CPU 对象，然后通过 `cpu.gprs` 获取它所关联的 GPRS 对象。

接下来，定义了一个内部函数，它接受一个空括号括号 `{}` 作为参数，表示函数调用时的参数。

在函数体内，首先执行 CPU 对象的 `mmu.waitPrefetch` 方法，将参数 `gprs[cpu.PC]` 传递给其中的一个 GPRS 对象，确保在参数列表中。

然后，根据 `gprs[rm]` 和 `gprs[rd]` 对象的值，计算出 `m` 和 `d` 变量，其中 `m` 是 `(gprs[rm] >>> 0) + !cpu.cpsrC`,`d` 是 `(gprs[rd] >>> 0) - m`，分别用于计算 `cpu.cpsrN` 和 `cpu.cpsrZ` 变量。

接着，执行计算结果，将 `cpu.cpsrN`、`cpu.cpsrZ` 和 `cpu.cpsrC` 设置为 `d >> 31`、`!(d & 0xFFFFFFFF)` 和 `(gprs[rd] ^ m) >> 31 && (gprs[rd] ^ d) >> 31` 分别用于设置 `cpu.cpsrC` 和 `cpu.cpsrV` 变量。

最后，通过 `gprs[rd]` 更新 `gprs` 对象，确保 `d` 的值被正确设置。

整个函数的作用是将 ARMCoreThumb 类的 `constructSBC` 函数映射到一个新的函数，用于创建新的 "ARMCoreThumb" 对象。


```js
ARMCoreThumb.prototype.constructSBC = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var m = (gprs[rm] >>> 0) + !cpu.cpsrC;
		var d = (gprs[rd] >>> 0) - m;
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = (gprs[rd] >>> 0) >= (d >>> 0);
		cpu.cpsrV = ((gprs[rd] ^ m) >> 31) && ((gprs[rd] ^ d) >> 31);
		gprs[rd] = d;
	};
};

```

该代码定义了一个名为 "ARMCoreThumb" 的对象，其 "constructSTMIA" 方法接收两个参数：一个整数（可能是寄存器或内存）和一个整数（可能是整数或字符串）。该方法返回一个函数，该函数执行以下操作：

1. 读取指定寄存器或内存的值并将其存储在整数变量 "address" 中。
2. 设置计数器 "m" 为 1，然后遍历从 0x01 到 0x07 的所有幂次。
3. 对于每个幂次，检查给定的寄存器或内存是否与 "m" 相与。如果是，则执行以下操作：
	1. 将 "address" 和寄存器或内存的值存储在内存中。
	2. 将计数器 "total" 加 1。
	3. 执行循环中的其他操作。
4. 对于每个幂次，执行以下操作：
	1. 如果给定的寄存器或内存与计数器 "m" 相与，则执行以下操作：
		1. 将 "address" 和寄存器或内存的值存储在内存中。
		2. 将计数器 "total" 加 1。
		3. 执行循环中的其他操作。
5. 调用 "cpu.mmu.waitMulti32" 函数，该函数接收两个参数：内存中的 "address" 和计数器 "total"。
	1. 将计算出的地址存储回给定的寄存器或内存。

通过执行上述操作，该代码实现了从指定寄存器或内存中读取数据，并执行一系列 CPU 指令，然后将结果写回到指定的寄存器或内存中的功能。


```js
ARMCoreThumb.prototype.constructSTMIA = function(rn, rs) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.wait(gprs[cpu.PC]);
		var address = gprs[rn];
		var total = 0;
		var m, i;
		for (m = 0x01, i = 0; i < 8; m <<= 1, ++i) {
			if (rs & m) {
				cpu.mmu.store32(address, gprs[i]);
				address += 4;
				++total;
				break;
			}
		}
		for (m <<= 1, ++i; i < 8; m <<= 1, ++i) {
			if (rs & m) {
				cpu.mmu.store32(address, gprs[i]);
				address += 4;
				++total;
			}
		}
		cpu.mmu.waitMulti32(address, total);
		gprs[rn] = address;
	};
};

```

这段代码定义了 `ARMCoreThumb` 类中的两个 `constructSTR1` 和 `constructSTR2` 方法，用于在 ARM 架构的硬件抽象层（硬件）中执行指令。

`constructSTR1` 函数接受三个参数：
1. `rd`：寄存器列表，包含要访问的寄存器。
2. `rn`：寄存器名称，指定要访问的寄存器。
3. `immediate`：紧急值，用于指定立即执行的指令。

该函数执行以下操作：
1. 获取 CPU 和 GPRS 缓存，以便在操作期间使用它们。
2. 使用 `gprs[rn]` 和 `immediate` 计算目标寄存器的值，并将结果存储到 CPU 的 MMU 寄存器中。
3. 使用 `cpu.mmu.store32` 命令将目标值存储到指定的寄存器中，然后使用 `cpu.mmu.wait` 命令等待直到指定的寄存器值已经被写入。
4. 使用 `cpu.mmu.wait32` 命令等待目标值的变化，然后继续执行指令。

`constructSTR2` 函数接受三个参数：
1. `rd`：寄存器列表，包含要访问的寄存器。
2. `rn`：寄存器名称，指定要访问的寄存器。
3. `rm`：寄存器名称，指定要立即执行的指令。

该函数执行以下操作：
1. 获取 CPU 和 GPRS 缓存，以便在操作期间使用它们。
2. 使用 `gprs[rn]` 和 `rm` 计算目标寄存器的值，并将结果存储到 CPU 的 MMU 寄存器中。
3. 使用 `cpu.mmu.store32` 命令将目标值存储到指定的寄存器中，然后使用 `cpu.mmu.wait` 命令等待直到指定的寄存器值已经被写入。
4. 使用 `cpu.mmu.wait32` 命令等待目标值的变化，然后继续执行指令。


```js
ARMCoreThumb.prototype.constructSTR1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		var n = gprs[rn] + immediate;
		cpu.mmu.store32(n, gprs[rd]);
		cpu.mmu.wait(gprs[cpu.PC]);
		cpu.mmu.wait32(n);
	};
};

ARMCoreThumb.prototype.constructSTR2 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.store32(gprs[rn] + gprs[rm], gprs[rd]);
		cpu.mmu.wait(gprs[cpu.PC]);
		cpu.mmu.wait32(gprs[rn] + gprs[rm]);
	};
};

```

这两段代码是对象 Spread 的箭头函数，它们的作用是创建两个不同的函数，一个是 ARMCoreThumb.prototype.constructSTR3，另一个是 ARMCoreThumb.prototype.constructSTRB1。这两个函数都接收三个参数：rd（相对距离，即参数从0开始计算），immediate（立即操作，即立即执行的指令），和rd（被参数，即局部变量接收的值）。

具体来说，这两段代码实现了一个工厂函数，它们根据传入的rd和immediate参数，返回一个名为function()的函数。这个函数通过两次取余运算，将参数从低位字节开始高位递增，然后将立即操作立即执行，并将结果存储到目标变量的低位字节。


```js
ARMCoreThumb.prototype.constructSTR3 = function(rd, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.store32(gprs[cpu.SP] + immediate, gprs[rd]);
		cpu.mmu.wait(gprs[cpu.PC]);
		cpu.mmu.wait32(gprs[cpu.SP] + immediate);
	};
};

ARMCoreThumb.prototype.constructSTRB1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		var n = gprs[rn] + immediate;
		cpu.mmu.store8(n, gprs[rd]);
		cpu.mmu.wait(gprs[cpu.PC]);
		cpu.mmu.wait(n);
	};
};

```

这段代码定义了 `ARMCoreThumb` 类中的两个构造函数 `constructSTRB2` 和 `constructSTRH1`。

`constructSTRB2` 函数接收三个参数 `rd`、`rm` 和 `rn`，并将它们存储在 `this.cpu` 变量中 (可能是 `CPU` 对象的一个属性)，然后返回一个函数，这个函数执行以下操作：

1. 从 `gprs` 数组中获取 `rn` 和 `rm` 寄存器对应的值。
2. 将这两个值加上 `rd` 参数的值，存储在 `gprs` 数组中。
3. 从 `CPU` 对象中的 `mmu` 属性中获取 `gprs` 数组中 `rd` 位置的值，并将其与 `gprs` 数组中 `rn` 位置的值相加，存储在 `cpu.mmu` 属性中。
4. 从 `CPU` 对象中的 `PC` 属性中获取 `gprs` 数组中 `cpu.PC` 位置的值，并将其与 `gprs` 数组中 `rm` 位置的值相加，存储在 `cpu.mmu` 属性中。
5. 从 `CPU` 对象中的 `mmu` 属性中获取 `gprs` 数组中 `rn` 位置的值，并将其与 `gprs` 数组中 `rm` 位置的值相加，存储在 `cpu.mmu` 属性中。

`constructSTRH1` 函数与 `constructSTRB2` 类似，只是返回了一个 `function` 对象，而不是一个类。其接收三个参数 `rd`、`rn` 和 `immediate`，并将它们存储在 `this.cpu` 变量中 (可能是 `CPU` 对象的一个属性)，然后返回一个函数，这个函数执行以下操作：

1. 从 `gprs` 数组中获取 `rn` 和 `rm` 寄存器对应的值。
2. 将这两个值加上 `rd` 参数的值，存储在 `gprs` 数组中。
3. 从 `CPU` 对象中的 `mmu` 属性中获取 `gprs` 数组中 `rd` 位置的值，并将其与 `gprs` 数组中 `rm` 位置的值相加，存储在 `cpu.mmu` 属性中。
4. 从 `CPU` 对象中的 `PC` 属性中获取 `gprs` 数组中 `cpu.PC` 位置的值，并将其与 `gprs` 数组中 `rm` 位置的值相加，存储在 `cpu.mmu` 属性中。
5. 从 `CPU` 对象中的 `mmu` 属性中获取 `gprs` 数组中 `rm` 位置的值，并将其与 `gprs` 数组中 `rm` 位置的值相加，存储在 `cpu.mmu` 属性中。


```js
ARMCoreThumb.prototype.constructSTRB2 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.store8(gprs[rn] + gprs[rm], gprs[rd]);
		cpu.mmu.wait(gprs[cpu.PC]);
		cpu.mmu.wait(gprs[rn] + gprs[rm]);
	}
};

ARMCoreThumb.prototype.constructSTRH1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		var n = gprs[rn] + immediate;
		cpu.mmu.store16(n, gprs[rd]);
		cpu.mmu.wait(gprs[cpu.PC]);
		cpu.mmu.wait(n);
	};
};

```

这段代码定义了两个名为`constructSTRH2`和`constructSUB1`的构造函数，它们属于`ARMCoreThumb`类。

这两个构造函数都接收三个参数：一个整数（`rd`表示寄存器名称）和一个整数（`rn`表示寄存器名称），以及一个布尔值（`immediate`表示是否立即执行操作，true表示立即执行，false表示延迟执行）。

函数内部通过`cpu`对象调用`mmu`指令，并将三个参数（`gprs[rn] + gprs[rm]`表示两个寄存器`rn`和`rm`的值相加）存储到指定的CPU寄存器中。然后，函数使用`cpu.mmu.wait`方法来等待CPU的内存访问操作完成。

`ARMCoreThumb.constructSTRH2`函数的实现主要关注内存访问操作。它确保在执行具体指令之前，CPU的内存已经被加载到指定的寄存器中。然后，它通过`cpu.mmu.wait`方法等待内存访问操作完成，然后执行给定的操作，最后再将内存写回原来的寄存器中。

`ARMCoreThumb.constructSUB1`函数的实现主要关注立即执行情况。它确保在执行具体指令之前，CPU的内存已经被加载到指定的寄存器中。然后，它通过`cpu.mmu.waitPrefetch`方法来延迟获取内存的访问，确保所有依赖的内存布局都已经准备就绪。

接下来，它会根据延迟时间（即立即执行的延迟时间）中指定的寄存器值，对指定的寄存器（`rn`）的值进行相应的计算，并将计算结果存储到指定的寄存器中。


```js
ARMCoreThumb.prototype.constructSTRH2 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.store16(gprs[rn] + gprs[rm], gprs[rd]);
		cpu.mmu.wait(gprs[cpu.PC]);
		cpu.mmu.wait(gprs[rn] + gprs[rm]);
	}
};

ARMCoreThumb.prototype.constructSUB1 = function(rd, rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = gprs[rn] - immediate;
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = (gprs[rn] >>> 0) >= immediate;
		cpu.cpsrV = (gprs[rn] >> 31) && ((gprs[rn] ^ d) >> 31);
		gprs[rd] = d;
	};
}

```

这段代码定义了两个名为 `constructSUB2` 和 `constructSUB3` 的函数，它们都属于 ARMCoreThumb 类。

这些函数的主要目的是实现减法运算，具体解释如下：

### `constructSUB2(rn, immediate)`

这个函数接收两个参数：一个整数 `rn` 和一个布尔值 `immediate`。函数内部首先尝试使用 `cpu.cpu` 属性获取当前实例的 CPU，然后使用 `cpu.gprs` 属性获取 CPU 内的总线事务请求(G- Register Prefetch)，最后执行以下操作：

1. 从 `gprs` 数组中读取 `cpu.PC` 所对应的寄存器中的值，即 CPU 内存器的偏移量。
2. 计算 `d = gprs[rn] - immediate` 并将其与 `immediate` 进行按位与运算，得到一个新的二进制数 `d`。
3. 更新 CPU 内部的计数器，包括 `cpsrN`、`cpsrZ` 和 `cpsrC`。其中，`cpsrN`、`cpsrZ` 和 `cpsrC` 是通过 `cpu.cpsrN`、`cpu.cpsrZ` 和 `cpu.cpsrC` 属性获取的，分别记录了 CPU 内存器中偏移量为 `rn` 和 `rm` 的寄存器中的值。
4. 将 `d` 赋值给 `gprs[rn]`，并将 `immediate` 置为 `true`。

函数返回一个新的函数，这个新函数会在执行完上述操作后返回，其内部实现了从 `gprs` 数组中读取指定寄存器中的值，并执行减法运算。

### `constructSUB3(rd, rn, rm)`

这个函数与 `constructSUB2` 类似，但是会读取两个寄存器 `rd` 和 `rn`，而不是只读取一个寄存器 `rm`。具体来说，这个函数的实现如下：

1. 从 `gprs` 数组中读取指定寄存器 `rm` 中的值。
2. 从 `gprs` 数组中读取指定寄存器 `rd` 中的值。
3. 更新 CPU 内部的计数器，包括 `cpsrN`、`cpsrZ` 和 `cpsrC`。其中，`cpsrN`、`cpsrZ` 和 `cpsrC` 是通过 `cpu.cpsrN`、`cpu.cpsrZ` 和 `cpu.cpsrC` 属性获取的，分别记录了 CPU 内存器中偏移量为 `rm` 和 `rd` 的寄存器中的值。
4. 将新的计数器中的值赋值给 `gprs[rn]` 和 `gprs[rm]`。
5. 将 `immediate` 置为 `true`，即输入参数 `rd` 和 `rm` 都是有效的。

函数返回一个新的函数，这个新函数会在执行完上述操作后返回，其内部实现了从 `gprs` 数组中读取指定寄存器中的值，并执行减法运算。


```js
ARMCoreThumb.prototype.constructSUB2 = function(rn, immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = gprs[rn] - immediate;
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = (gprs[rn] >>> 0) >= immediate;
		cpu.cpsrV = (gprs[rn] >> 31) && ((gprs[rn] ^ d) >> 31);
		gprs[rn] = d;
	};
};

ARMCoreThumb.prototype.constructSUB3 = function(rd, rn, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var d = gprs[rn] - gprs[rm];
		cpu.cpsrN = d >> 31;
		cpu.cpsrZ = !(d & 0xFFFFFFFF);
		cpu.cpsrC = (gprs[rn] >>> 0) >= (gprs[rm] >>> 0);
		cpu.cpsrV = (gprs[rn] >> 31) != (gprs[rm] >> 31) &&
					(gprs[rn] >> 31) != (d >> 31);
		gprs[rd] = d;
	};
};

```

这两段代码涉及到Thumb文化与机器(Thumb for Culture)技术，允许在JavaScript中使用Thumb库。

`ARMCoreThumb.prototype.constructSWI`函数的作用是为Thumb程序中的ARM芯片设置硬件中断(irq)。该函数接受一个参数`immediate`，它会立即通知操作系统有硬件中断发生。然后，它返回一个函数，这个函数会在CPU的立即门(irq)上执行一条机器指令，并设置CPU的MMU偏好(waitPrefetch)。这条机器指令会暂停CPU的执行，直到MMU的偏好被设置完成。这个函数的实现确保了在Thumb程序中，当需要硬件中断时，操作系统能够及时通知JavaScript程序。

`ARMCoreThumb.prototype.constructTST`函数的作用与`constructSWI`函数类似，但是它设置的是Thumb for Culture目标(TST)的硬件中断。函数接受两个参数`rd`和`rm`，它们代表需要设置的目标硬件中断的寄存器。函数返回一个函数，这个函数会在CPU的立即门上执行一条机器指令，并设置CPU的CMU偏好(waitPrefetch)。这条机器指令会暂停CPU的执行，直到CMU的偏好被设置完成。这个函数的实现确保了在Thumb for Culture目标中，当需要硬件中断时，操作系统能够及时通知JavaScript程序。


```js
ARMCoreThumb.prototype.constructSWI = function(immediate) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.irq.swi(immediate);
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
	}
};

ARMCoreThumb.prototype.constructTST = function(rd, rm) {
	var cpu = this.cpu;
	var gprs = cpu.gprs;
	return function() {
		cpu.mmu.waitPrefetch(gprs[cpu.PC]);
		var aluOut = gprs[rd] & gprs[rm];
		cpu.cpsrN = aluOut >> 31;
		cpu.cpsrZ = !(aluOut & 0xFFFFFFFF);
	};
};

```