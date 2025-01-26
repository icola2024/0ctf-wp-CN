## EzLogic
### 前置知识
首先让我们回顾一下数字电路的知识。

**组合逻辑电路**（下面简称组合电路），是指在任何时刻，电路的输出状态只取决于同一时刻的输入状态，与电路原来的状态无关。

![](https://s2.loli.net/2025/01/23/yqWf5D3dzGmKkhn.png)

一些大家耳熟能详的“逻辑门”，都属于组合电路。这些门的输入一旦发生改变，通常可以认为输出立刻就会改变以匹配输入。当然，这些门的连接组合形成的网络也属于组合电路。通过设计不同的逻辑门组合，可以实现各种各样的逻辑功能，如加法器、比较器、编码器等。

![](https://s2.loli.net/2025/01/23/sVf9IU1peZ7auEb.png)

**时序逻辑电路**（下面简称时序电路），是指电路在任一时刻的输出不仅与当前时刻的输入有关，还与当前时刻的电路状态有关。时序电路可以看成组合电路加上存储电路（一般是寄存器）。寄存器存储状态信息，状态不同，电路执行的逻辑就可以不同，这使得时序电路能够实现更复杂的逻辑。

![](https://s2.loli.net/2025/01/23/SDEWdMgBlkzF1H5.png)

时序电路的工作离不开时钟信号。时钟信号是一个周期性变化的信号，它就像一个节拍器，为电路提供统一的时间基准。在每个时钟周期内，时序电路根据输入信号和当前存储的状态，按照预设的逻辑关系，更新存储元件的状态，并产生相应的输出信号。

以一个简单的D触发器为例，它有一个数据输入端D、一个时钟输入端CLK和一个输出端Q。**在时钟信号的上升沿**，D触发器会将数据输入端D的信号锁存到输出端Q，**并保持这个状态**，直到下一个时钟周期。这样，D触发器就实现了一个基本的存储功能，能够将输入信号在时钟信号的控制下，稳定地存储并输出。

![](https://s2.loli.net/2025/01/23/eCzBbgrVfLtQwc6.png)

如果再给D触发器加上时钟使能（Clock Enable, CE）和清除（Clear, CLR）信号，这就是 FPGA 逻辑里最常用的触发器之一——FDCE（D Flip-Flop with Clock Enable and Asynchronous Clear）

![](https://s2.loli.net/2025/01/23/Y2Ruplk3WAGfxdj.png)

### 电路分析
把 `EzLogic_top_synth.v` 加进 Vivado 项目里，设置为顶层模块，然后点击左侧的“Run Synthesis”，完成后打开 Schematic，即可看到电路的原理图。

![](https://s2.loli.net/2025/01/23/ROHedfUuTGqmkc8.png)

![](https://s2.loli.net/2025/01/23/AkRdgpaQz8NLvEq.png)

除了上面介绍过的 FDCE，LUT 大家也应该很熟悉了，就是一个查找表，用枚举的方式，可以替代任意复杂的组合电路。剩下的 IBUF、OBUF 都可以看成导线，只有 CARRY4 需要注意。

根据[文档](https://docs.amd.com/r/en-US/ug953-vivado-7series-libraries/CARRY4)，CARRY4 是一个加法器，还带进位，因此可以级联起来。查阅其他资料，可以通俗的解释6个端口：
- O：输出，加法结果
- S：输入，**两个加数的异或**
- DI：输入，其中一个加数
- CYINIT：输入，进位初始化
- CI：输入，级联进位
- CO：输出，级联进位

不难注意到这个 CARRY4 并不是通俗理解的输入两个数输出它们的和，其中一个输入是以两加数异或的形式给出的。这就能解释电路图里 CARRY4 旁边的 8 个 LUT2 了。

![](https://s2.loli.net/2025/01/23/huedjswLIBiCE3l.png)

查看代码，发现 8 个 LUT2 的 INIT 值均为 `4'h6`，也就是二进制的 `0110`。

```verilog
  LUT2 #(
    .INIT(4'h6)) 
    \data_reg[3]_i_2 
       (.I0(data_out_OBUF[5]),
        .I1(data_in_IBUF[3]),
        .O(\data_reg[3]_i_2_n_0 ));
```

如果还是没感觉，可以写成真值表的形式：

|输入A|输入B|对应的查找表位置|输出|
|---|---|---|---|
|0|0|011**0**|0|
|0|1|01**1**0|1|
|1|0|0**1**10|1|
|1|1|**0**110|0|

很明显是个异或运算。

再次回到原理图，根据 CI 和 CO 的连接关系，不难发现左边的 CARRY4 实现高四位加法，右边的 CARRY4 实现低四位加法。左边 CARRY4 的 CO 断路，说明整个 8 位加法器是自然溢出的（或者叫卷绕），这也是 C/C++ 加法的默认行为。

![](https://s2.loli.net/2025/01/23/fiM7596lJrCFpwG.png)

现在让我们来具体看看两个 8 位的加数分别是什么。根据连接关系，两个 CARRY4 的 DI 和 S 端口的值取决于外部输入和寄存器堆 data_reg_reg 的值。

下面逐比特进行寻找：
- 第0位（LSB）：data_reg_reg[4] + data_in[0]
![](https://s2.loli.net/2025/01/23/wQ9q7gLIc4y3pnX.png)
- 第1位：data_reg_reg[2] + data_in[1]
![](https://s2.loli.net/2025/01/23/dbgsyOIG3U9VZxC.png)
- 第2位：data_reg_reg[6] + data_in[2]
![](https://s2.loli.net/2025/01/23/PDZb6FsIY5ocCEm.png)
- 第3位：data_reg_reg[5] + data_in[3]
![](https://s2.loli.net/2025/01/23/JdgqpN1eP7ujTc8.png)
- 第4位：data_reg_reg[1] + data_in[4]
![](https://s2.loli.net/2025/01/23/VPM3B19LtGiUrsh.png)
- 第5位：data_reg_reg[3] + data_in[5]
- 第6位：data_reg_reg[0] + data_in[6]
- 第7位：data_reg_reg[7] + data_in[7]

模块输出端口直接连在 data_reg_reg 上，我们忽略 valid 信号，可以直接写出模块源文件的 Verilog 代码了：

```verilog
module EzLogic(
    input wire clk,
    input wire rst_n,
    input wire [7:0] data_in,
    output wire [7:0] data_out
);

    reg [7:0] data_reg_reg;

    wire [7:0] data_shuffle = {
        data_reg_reg[7], 
        data_reg_reg[0], 
        data_reg_reg[3], 
        data_reg_reg[1], 
        data_reg_reg[5], 
        data_reg_reg[6], 
        data_reg_reg[2],
        data_reg_reg[4]
    };

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            data_reg_reg <= 0;
        end
        else begin
            data_reg_reg <= data_shuffle + data_in;
        end
    end

    assign data_out = data_reg_reg;

endmodule
```

把目光转到 `EzLogic_tb.v` 文件，不难发现其逻辑是对于 FLAG_TO_TEST 的每一字符（从左侧开始），把 8 位二进制输出 EzLogic 模块，得到的输出放到 data_out_all 的指定位置。最后与 data_std 比较判断 flag。

由于 data_reg_reg 最开始是 0，那么第一个输入的字符相当于直接赋给了 data_reg_reg，输出一定和输入一样。data_std 的最高 8 位是 `8'h30`，也就是字符 `0`，正好符合 flag 的格式“0ops{”。

exp 的编写就很显然了，后一个输出是前一个输出按照一定顺序进行二进制重排，再加上输入得到的，那么我们只需要反过来做一个减法，就可以反推输入。（自然溢出的加法是可逆的）

```python
data_std = '30789d5692f2fe23bb2c5d9e16406653b6cb217c952998ce17b7143788d949952680b4bce4c30a96c753'
last_byte = '00000000'
res = ''
for i in range(0, len(data_std), 2):
    byte_hex = data_std[i:i+2]
    byte_int = int(byte_hex, 16)
    byte_bin = bin(byte_int)[2:].zfill(8)
    last_byte_reorder = ''.join([
        last_byte[7 - 7],
        last_byte[7 - 0],
        last_byte[7 - 3],
        last_byte[7 - 1],
        last_byte[7 - 5],
        last_byte[7 - 6],
        last_byte[7 - 2],
        last_byte[7 - 4],
    ]) # Python 的字符串下标是从左边开始，Verilog 里是右边 LSB
    res += chr((int(byte_bin, 2) - int(last_byte_reorder, 2)) % 256)
    last_byte = byte_bin

print(res)
```