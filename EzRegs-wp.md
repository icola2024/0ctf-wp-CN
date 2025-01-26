## EzRegs
### 电路分析
把 `EzRegs_top_synth.v` 加进 Vivado 项目里，设置为顶层模块。右键点击“SYNTHESIS”，在综合选项里关闭所有优化。

![](https://s2.loli.net/2025/01/25/KVsBYSciFJ6EHIq.png)

完成设置后点击左侧的“Run Synthesis”，待综合完成打开 Schematic，即可看到电路的原理图。

观察发现 `EzRegs_top_synth` 主要由一个 pe_array，再加外围控制电路构成。

![](https://s2.loli.net/2025/01/25/VeDEcgPpHlGBQit.png)

pe_array 里有很多小的 pe 模块。出题人非常良心地没有把整个电路展平，因此分析的时候可以选择性展开查看，非常方便。

![](https://s2.loli.net/2025/01/25/VMPZBL6kt8cjSbW.png)

### 数据输入
先从简单的开始分析，pe_array 的数据输入

![](https://s2.loli.net/2025/01/25/KZn2i5sYvpCeuaR.png)

LUT 的 `INIT` 为 `4'h8`，因此这里就是控制模块输入的值，若 valid_i 为高则输入 data_i，否则输入 0。

### 状态机
在 Schematic 全图左上角观察到一堆相对独立的 LUT 和 FF，名字甚至都是 counter 开头，那就可以确定是有限状态机（FSM）了。

![](https://s2.loli.net/2025/01/25/E7mpMlLSnWsTBbv.png)

把这部分逻辑提取出来，可以发现除时钟（clk）和重置（rst_n）信号外，状态机只由 valid_i 信号控制。

![](https://s2.loli.net/2025/01/25/D9z2G7TBaAWefZx.png)

状态机的输出，连接到 pe array 的只有 reset 和 enable 信号。

![](https://s2.loli.net/2025/01/25/HZ4gEAJewLWt5CN.png)

根据 `EzRegs_tb.v` 内的逻辑，valid_i 和 data_i 共同作用把 FLAG_TO_TEST 逐字节输入模块，因此 valid_i 拉高周期数和 FLAG_TO_TEST 长度相同，42 个周期。

我们花一点时间，把状态机部分的代码从 `EzRegs_top_synth.v` 里提出来，单独保存为 `EzRegs_FSM.v` 文件：
```verilog
module EzRegs_FSM (
    input wire clk,
    input wire rst_n,
    input wire valid_i,
    output wire enable,
    output wire reset
);

  wire clk_IBUF;
  wire clk_IBUF_BUFG;
  wire \counter[0]_i_1_n_0 ;
  wire \counter[0]_i_2_n_0 ;
  wire \counter[1]_i_1_n_0 ;
  wire \counter[2]_i_1_n_0 ;
  wire \counter[3]_i_1_n_0 ;
  wire \counter[4]_i_1_n_0 ;
  wire \counter[5]_i_1_n_0 ;
  wire \counter[6]_i_1_n_0 ;
  wire \counter[6]_i_2_n_0 ;
  wire \counter[6]_i_3_n_0 ;
  wire \counter[6]_i_4_n_0 ;
  wire \counter[6]_i_5_n_0 ;
  wire \counter_reg_n_0_[0] ;
  wire \counter_reg_n_0_[1] ;
  wire \counter_reg_n_0_[2] ;
  wire \counter_reg_n_0_[3] ;
  wire \counter_reg_n_0_[4] ;
  wire \counter_reg_n_0_[5] ;
  wire \counter_reg_n_0_[6] ;
  wire pe_arr_enable;
  wire pe_array_INST_i_11_n_0;
  wire pe_array_INST_i_12_n_0;
  wire pe_array_INST_i_1_n_0;
  wire rst_n_IBUF;
  wire valid_i_IBUF;

  BUFG clk_IBUF_BUFG_inst
       (.I(clk_IBUF),
        .O(clk_IBUF_BUFG));
  IBUF clk_IBUF_inst
       (.I(clk),
        .O(clk_IBUF));
  LUT3 #(
    .INIT(8'h40)) 
    \counter[0]_i_1 
       (.I0(\counter_reg_n_0_[0] ),
        .I1(valid_i_IBUF),
        .I2(\counter[0]_i_2_n_0 ),
        .O(\counter[0]_i_1_n_0 ));
  LUT6 #(
    .INIT(64'hFFFFFFFFFFFFFF7F)) 
    \counter[0]_i_2 
       (.I0(\counter_reg_n_0_[5] ),
        .I1(\counter_reg_n_0_[1] ),
        .I2(\counter_reg_n_0_[3] ),
        .I3(\counter_reg_n_0_[2] ),
        .I4(\counter_reg_n_0_[4] ),
        .I5(\counter_reg_n_0_[6] ),
        .O(\counter[0]_i_2_n_0 ));
  LUT4 #(
    .INIT(16'h0880)) 
    \counter[1]_i_1 
       (.I0(valid_i_IBUF),
        .I1(pe_array_INST_i_11_n_0),
        .I2(\counter_reg_n_0_[1] ),
        .I3(\counter_reg_n_0_[0] ),
        .O(\counter[1]_i_1_n_0 ));
  LUT5 #(
    .INIT(32'h08808080)) 
    \counter[2]_i_1 
       (.I0(valid_i_IBUF),
        .I1(pe_array_INST_i_11_n_0),
        .I2(\counter_reg_n_0_[2] ),
        .I3(\counter_reg_n_0_[0] ),
        .I4(\counter_reg_n_0_[1] ),
        .O(\counter[2]_i_1_n_0 ));
  LUT6 #(
    .INIT(64'h0880808080808080)) 
    \counter[3]_i_1 
       (.I0(valid_i_IBUF),
        .I1(pe_array_INST_i_11_n_0),
        .I2(\counter_reg_n_0_[3] ),
        .I3(\counter_reg_n_0_[2] ),
        .I4(\counter_reg_n_0_[1] ),
        .I5(\counter_reg_n_0_[0] ),
        .O(\counter[3]_i_1_n_0 ));
  LUT6 #(
    .INIT(64'h8080808008808080)) 
    \counter[4]_i_1 
       (.I0(valid_i_IBUF),
        .I1(pe_array_INST_i_11_n_0),
        .I2(\counter_reg_n_0_[4] ),
        .I3(\counter_reg_n_0_[2] ),
        .I4(\counter_reg_n_0_[0] ),
        .I5(\counter[6]_i_5_n_0 ),
        .O(\counter[4]_i_1_n_0 ));
  LUT6 #(
    .INIT(64'h0880808080808080)) 
    \counter[5]_i_1 
       (.I0(valid_i_IBUF),
        .I1(pe_array_INST_i_11_n_0),
        .I2(\counter_reg_n_0_[5] ),
        .I3(\counter[6]_i_4_n_0 ),
        .I4(\counter_reg_n_0_[3] ),
        .I5(\counter_reg_n_0_[1] ),
        .O(\counter[5]_i_1_n_0 ));
  LUT2 #(
    .INIT(4'hB)) 
    \counter[6]_i_1 
       (.I0(valid_i_IBUF),
        .I1(pe_array_INST_i_11_n_0),
        .O(\counter[6]_i_1_n_0 ));
  LUT6 #(
    .INIT(64'h8080808008808080)) 
    \counter[6]_i_2 
       (.I0(valid_i_IBUF),
        .I1(pe_array_INST_i_11_n_0),
        .I2(\counter_reg_n_0_[6] ),
        .I3(\counter[6]_i_4_n_0 ),
        .I4(\counter_reg_n_0_[5] ),
        .I5(\counter[6]_i_5_n_0 ),
        .O(\counter[6]_i_2_n_0 ));
  LUT1 #(
    .INIT(2'h1)) 
    \counter[6]_i_3 
       (.I0(rst_n_IBUF),
        .O(\counter[6]_i_3_n_0 ));
  LUT3 #(
    .INIT(8'h80)) 
    \counter[6]_i_4 
       (.I0(\counter_reg_n_0_[4] ),
        .I1(\counter_reg_n_0_[2] ),
        .I2(\counter_reg_n_0_[0] ),
        .O(\counter[6]_i_4_n_0 ));
  LUT2 #(
    .INIT(4'h7)) 
    \counter[6]_i_5 
       (.I0(\counter_reg_n_0_[1] ),
        .I1(\counter_reg_n_0_[3] ),
        .O(\counter[6]_i_5_n_0 ));
  FDCE #(
    .INIT(1'b0)) 
    \counter_reg[0] 
       (.C(clk_IBUF_BUFG),
        .CE(\counter[6]_i_1_n_0 ),
        .CLR(\counter[6]_i_3_n_0 ),
        .D(\counter[0]_i_1_n_0 ),
        .Q(\counter_reg_n_0_[0] ));
  FDCE #(
    .INIT(1'b0)) 
    \counter_reg[1] 
       (.C(clk_IBUF_BUFG),
        .CE(\counter[6]_i_1_n_0 ),
        .CLR(\counter[6]_i_3_n_0 ),
        .D(\counter[1]_i_1_n_0 ),
        .Q(\counter_reg_n_0_[1] ));
  FDCE #(
    .INIT(1'b0)) 
    \counter_reg[2] 
       (.C(clk_IBUF_BUFG),
        .CE(\counter[6]_i_1_n_0 ),
        .CLR(\counter[6]_i_3_n_0 ),
        .D(\counter[2]_i_1_n_0 ),
        .Q(\counter_reg_n_0_[2] ));
  FDCE #(
    .INIT(1'b0)) 
    \counter_reg[3] 
       (.C(clk_IBUF_BUFG),
        .CE(\counter[6]_i_1_n_0 ),
        .CLR(\counter[6]_i_3_n_0 ),
        .D(\counter[3]_i_1_n_0 ),
        .Q(\counter_reg_n_0_[3] ));
  FDCE #(
    .INIT(1'b0)) 
    \counter_reg[4] 
       (.C(clk_IBUF_BUFG),
        .CE(\counter[6]_i_1_n_0 ),
        .CLR(\counter[6]_i_3_n_0 ),
        .D(\counter[4]_i_1_n_0 ),
        .Q(\counter_reg_n_0_[4] ));
  FDCE #(
    .INIT(1'b0)) 
    \counter_reg[5] 
       (.C(clk_IBUF_BUFG),
        .CE(\counter[6]_i_1_n_0 ),
        .CLR(\counter[6]_i_3_n_0 ),
        .D(\counter[5]_i_1_n_0 ),
        .Q(\counter_reg_n_0_[5] ));
  FDCE #(
    .INIT(1'b0)) 
    \counter_reg[6] 
       (.C(clk_IBUF_BUFG),
        .CE(\counter[6]_i_1_n_0 ),
        .CLR(\counter[6]_i_3_n_0 ),
        .D(\counter[6]_i_2_n_0 ),
        .Q(\counter_reg_n_0_[6] ));

  LUT1 #(
    .INIT(2'h1)) 
    pe_array_INST_i_1
       (.I0(pe_array_INST_i_11_n_0),
        .O(pe_array_INST_i_1_n_0));

  LUT6 #(
    .INIT(64'hFFFFFFFFFFFEFFFF)) 
    pe_array_INST_i_11
       (.I0(\counter_reg_n_0_[6] ),
        .I1(\counter_reg_n_0_[4] ),
        .I2(\counter_reg_n_0_[2] ),
        .I3(\counter[6]_i_5_n_0 ),
        .I4(\counter_reg_n_0_[5] ),
        .I5(\counter_reg_n_0_[0] ),
        .O(pe_array_INST_i_11_n_0));
  LUT6 #(
    .INIT(64'h000000000000002A)) 
    pe_array_INST_i_12
       (.I0(valid_i_IBUF),
        .I1(\counter_reg_n_0_[0] ),
        .I2(\counter_reg_n_0_[1] ),
        .I3(\counter_reg_n_0_[2] ),
        .I4(\counter_reg_n_0_[4] ),
        .I5(\counter_reg_n_0_[6] ),
        .O(pe_array_INST_i_12_n_0));
  LUT6 #(
    .INIT(64'hABAAABAAABAABBAA)) 
    pe_array_INST_i_2
       (.I0(pe_array_INST_i_12_n_0),
        .I1(\counter_reg_n_0_[6] ),
        .I2(\counter_reg_n_0_[5] ),
        .I3(valid_i_IBUF),
        .I4(\counter_reg_n_0_[3] ),
        .I5(\counter_reg_n_0_[4] ),
        .O(pe_arr_enable));

  IBUF rst_n_IBUF_inst
       (.I(rst_n),
        .O(rst_n_IBUF));
  IBUF valid_i_IBUF_inst
       (.I(valid_i),
        .O(valid_i_IBUF));

  assign enable = pe_arr_enable;
  assign reset = pe_array_INST_i_1_n_0;
endmodule
```
以及编写一个简单的 Testbench：
```verilog
`timescale 1us / 100ns
module EzRegs_FSM_tb #(
    parameter N = 42
)();
    reg clk, rst_n;
    reg start;
    reg [7:0] counter;
    reg valid_i;

    wire pe_array_reset;
    wire pe_array_enable;

    EzRegs_FSM EzRegs_FSM_INST(
        .clk            (clk    ),
        .rst_n          (rst_n  ),
        .valid_i        (valid_i),
        .reset          (pe_array_reset),
        .enable         (pe_array_enable)
    );

    initial begin
        $dumpfile("EzRegs_FSM.vcd");
        $dumpvars(0, EzRegs_FSM_tb);
        clk = 0;
        rst_n = 0;
        valid_i = 0;
        start = 0;
        #6
        rst_n = 1;
        start = 1;
    end

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            counter <= 0;
        end
        else if (start) begin
            if (counter < N) begin
                counter <= counter + 1;
                valid_i <= 1;
            end
            else begin
                start <= 0;
                valid_i <= 0;
            end
        end
    end
    
    always #1 clk = ~clk;
endmodule
```

运行仿真，得到波形图

![](https://s2.loli.net/2025/01/25/sOqe3vjnozdNJkT.png)

很明显这个状态机就是个简单的计数，输入前 42 个字节的时候拉高 pe_array 的 enable，下一个周期拉高 pe_array 的 reset。

![](https://s2.loli.net/2025/01/25/zZrYLFtp6j9PiIg.png)

### 数据输出

![](https://s2.loli.net/2025/01/25/mQ9feWoza5OC7hv.png)

再看数据输出部分，pe array 输出了 40 Bytes 的数据，首先经过一个 LUT2，所有 LUT2 的 I1 输入端口连接着状态机部分的 `pe_array_INST_i_11_n_0`，这个信号刚才似乎忘了引出，让我们重新跑一遍仿真，把这个信号加入波形图看一眼：

![](https://s2.loli.net/2025/01/25/Ymzs4EoQBc6HkIV.png)

哦，原来就是 reset 的反向。

再看下这些 LUT2 的 `INIT` 值，全都是 `4'h2`，那么这实际上还是个清零控制，表达式可以写为：
```
O = (~I1) ? I0 : 0
```
也就是当 I1 为 0 时（对应地，reset 为 1）**传递** I0，否则输出 0。reset 为 1 刚好就是所有数据处理完，pe array 准备重置的时刻。在下一个时钟上升沿到来时，pe array 重置，**同时** pe array 的输出数据被后面的 FF 存起来。

LUT2 的数据连到了 FDCE 的 D 端口，FDCE 的 CLR 连在了 `counter[6]_i_3_n_0`，往前找是个 LUT1，其实就是 rst_n 的非。

![](https://s2.loli.net/2025/01/25/Wvd4nwSAm3bsx9Q.png)

FDCE 的 CE 则是直接连在了 VCC。

![](https://s2.loli.net/2025/01/25/MbU2wOxQH9CBeg5.png)

至此，pe array 外围电路分析完毕。总结一下就是逐周期给 pe array 送入 42 个 8 位的数据，完成后重置 pe array 同时保存输出数据。

### PE ARRAY 初步分析
查看 pe array，里面有 40 个 pe，对应了输出的字节数量。

![](https://s2.loli.net/2025/01/25/pUAwJFCLqHhc7QP.png)

除了时钟和重置，pe 还有 reset、enable、data_i、data_o、top_line 端口。

简单分析连接关系，`genblk2[0].pe_INST` 的 data_o 连到了 `genblk2[1].pe_INST` 的 data_i，`genblk2[1].pe_INST` 的 data_o 连到了 `genblk2[2].pe_INST` 的 data_i，以此类推，`genblk2[39].pe_INST` 的 data_o 和输入数据进行了一些操作连到了 top_line，而 top_line 直接连到每个 pe 模块。

![](https://s2.loli.net/2025/01/25/Wvq8UHldh9bMcEa.png)

在 pe 内部，主要由一个 op4、一个 op3，一些 LUT 和 8 个 FDCE 组成。

![](https://s2.loli.net/2025/01/26/3a7tuMvXBecLmhz.png)

不难验证 FDCE 的 CE 和 CLR 的连接都符合常理：
- 当 rst_n=0，全部清零
- 当 rst_n=1
  - 当 reset=1，全部清零
  - 当 reset=0
    - 当 enable=0，CE=0，数据保持不变
    - 当 enable=1，CE=1，数据锁存到 Q 端口

### 算子分析
现在解决问题的关键就是 pe 里涉及到的 op3 和 op4 算子是什么运算。展开 op4，发现里面还套了 op1 和 op2。四个算子的输入输出端口如下：

|   |输入|输出|
|---|---|---|
|op1|a[7:0]|c[7:0]|
|op2|a[7:0]|c[7:0]|
|op3|a[7:0], b[7:0]|c[7:0]|
|op4|a[7:0], b[7:0]|c[7:0]|

下面我们就来分析这四个算子。

#### op3
仍然先挑软柿子捏，我们发现 op3 的逻辑异常简单：

![](https://s2.loli.net/2025/01/25/2aGJMsCrYIRHqX5.png)

看一下这堆 LUT2 的 `INIT` 值，是 `4'h6`！又见到了我们的老朋友，异或运算。

#### op1
op1 里面东西比较多，有 LUT 和 MUX，不过好在没有 FF，因此还是个组合逻辑。

![](https://s2.loli.net/2025/01/25/TmEg8pPYSs5not6.png)

因为输入只有 2^8=256 种可能，不妨把 op1 单独拿出来跑一下函数表。
```verilog
module op1_tb();
    reg[7:0] a;
    wire [7:0] c;

    op1__3 op1_INST(
        .a(a),
        .c(c)
    );

    initial begin
        a = 0;
    end
    always #1 begin
        if (a < 8'd255) 
            a <= a + 1;
    end
endmodule
```

![](https://s2.loli.net/2025/01/25/NJ7EWIVYHhUGb2C.png)

结果非常的有规律，前面几个都是 2 的幂次，而当 a=8，输出 c=256 超出了 8 位二进制的范围的时候，输出了 45。再后面是 90、180，似乎仍然是不停地乘以 2。180 乘以 2 是 360 又超过 255 了，这次变成了 69。统计一下输出的数据，0~255 刚好各出现了一遍，这样的运算实际上是有限域 $\text{GF}(2^8)$ 在某个生成多项式下的指数运算。而生成多项式就是溢出的时候用作范围控制的因子，使用异或运算“加”到溢出的数上。

根据异或的可逆性，生成多项式就是 `256^45=360^69=301`，也就是二进制的“100101101”，写成质多项式形式：$x^8+x^5+x^3+x^2+1$。

这一步可能确实有点难，毕竟不是所有人都学过有限域。但是其实 ChatGPT 是能给出提示的。

![](https://s2.loli.net/2025/01/25/DLoVHOseq7I4Gx2.png)

GPT 甚至还给了份代码，虽然它给出了错误的生成多项式，但是这个时候聪明的做题人应该已经发现超出范围的值和实际值的异或的结果是一个定值。

![](https://s2.loli.net/2025/01/25/WPQ5qR1GcOaV34U.png)

#### op2 
op2 也是单输入单输出的，同样也来跑一下仿真获得函数表。

![](https://s2.loli.net/2025/01/25/k6lHvzjhe7uBisS.png)

从画红框的几组数据中就不难猜出，这是 $\text{GF}(2^8)$ 下的对数运算，也就是 op1 的反函数。

Google 一下，可以发现这样一个 PDF：https://codyplanteen.com/assets/rs/gf256_log_antilog.pdf

在 301 (0x12D) 的两页，直接就能找到两张函数表，与 op1 和 op2 一模一样 

![](https://s2.loli.net/2025/01/25/iMjJ3SRcod2OEwl.png)

#### op4
最后一个算子是 op4，在分析 op4 之前我们回过头看一下 op3。在 EzLUTs 里我们就知道 $\text{GF}(2)$ 上的加法就是异或，事实上对于 $\text{GF}(256)$ 也是一样，op3 是有限域的加法，那么根据有限域的定义，op4 大概率就是有限域的乘法了。

开始分析前，有个迷惑的点，就是 op4 的代码里虽然是两个 8 比特输入，但是第二个输入 b 似乎完全没有用到，因此原理图上也被隐藏了。

![](https://s2.loli.net/2025/01/25/mlUPaMy537dQEcA.png)

![](https://s2.loli.net/2025/01/25/jGqf64hMgZODTi9.png)

这种情况大概率是因为输入是一个定值，被 Vivado 综合器给直接优化掉了。

先挑 `pe__1`（`genblk2[0].pe_INST`） 里的 op4 进行分析，输入进来之后先过了个 op2（对数运算），然后连到了两个 CARRY4

![](https://s2.loli.net/2025/01/25/vb2lXYBtrnQModA.png)

这两个 CARRY4 的连接方法确实是比较抽象，我们把代码拿出来仔细研究：
```verilog
  LUT1 #(
    .INIT(2'h1)) 
    inst_op1_1_i_12
       (.I0(l_a[3]),
        .O(inst_op1_1_i_12_n_0));
  LUT1 #(
    .INIT(2'h1)) 
    inst_op1_1_i_13
       (.I0(l_a[2]),
        .O(inst_op1_1_i_13_n_0));
  LUT1 #(
    .INIT(2'h1)) 
    inst_op1_1_i_14
       (.I0(l_a[1]),
        .O(inst_op1_1_i_14_n_0));
  CARRY4 inst_op1_1_i_11
       (.CI(\<const0> ),
        .CO({inst_op1_1_i_11_n_0,inst_op1_1_i_11_n_1,inst_op1_1_i_11_n_2,inst_op1_1_i_11_n_3}),
        .CYINIT(l_a[0]),
        .DI(l_a[4:1]),
        .O({inst_op1_1_i_11_n_4,inst_op1_1_i_11_n_5,inst_op1_1_i_11_n_6,inst_op1_1_i_11_n_7}),
        .S({l_a[4],inst_op1_1_i_12_n_0,inst_op1_1_i_13_n_0,inst_op1_1_i_14_n_0}));
  CARRY4 inst_op1_1_i_9
       (.CI(inst_op1_1_i_11_n_0),
        .CO({p_0_in,NLW_inst_op1_1_i_9_CO_UNCONNECTED[2],inst_op1_1_i_9_n_2,inst_op1_1_i_9_n_3}),
        .CYINIT(\<const0> ),
        .DI({\<const0> ,l_a[7:5]}),
        .O({inst_op1_1_i_9_n_5,inst_op1_1_i_9_n_6,inst_op1_1_i_9_n_7}),
        .S({\<const1> ,l_a[7:5]}));
```
第一级 `inst_op1_1_i_11`，我们还是先把加数提取出来，一个是 `DI`，一个是 `DI^S`。这里 `S` 的后三位是 LUT1 的结果，三个 LUT1 分别对 `l_a[3:1]` 进行了取反操作。根据异或的性质，`(x)^(~x)=1`，故第一个加数是 `l_a[4:1]`，第二个加数是 `4'b0111`。

但是这里 `CYINIT` 不是定值，而是 `l_a[0]`，根据 `CYINIT` 的含义“进位初始化”，可以认为前面还有一个一位的全加器，可能会产生进位。很明显，这个一位全加器的两个加数分别是 `l_a[0]` 和 `1`，这样子产生的进位就是 `l_a[0]`（全加器进位公式：`a&b`）。不过这个“全加器”实际上是不存在的，所以 `l_a[0]` 和 `1` 加法的和在这里并不能体现出来。

总结一下，`inst_op1_1_i_11` 做的加法是：`l_a[4:0] + 5'b01111`，但是最低位的值这里没能得到。

![](https://s2.loli.net/2025/01/26/8eZvstuMUqVQ2JG.png)

然后是 `inst_op1_1_i_9`，第一个加数是 `{0, l_a[7:5]}`，第二个加数是 `4'b1000`，`CI` 取了前一个 CARRY4 的进位，`CYINIT` 为 0，故两个 CARRY4 合起来实现了 9 位二进制数的加法：

![](https://s2.loli.net/2025/01/26/Ma7kl12h4f8NTmW.png)

第二级 CARRY4 的输出，其实只连了三根线，也就是说整个 9 位二进制加法里，最高位的结果也没有用到，用到的只有级联进位的输出 `CO` 里的 `p_0_in`。由于第二个加数的第九位是 `1`，相当于实现了传递效果，只要第八位有进位，`p_0_in` 就是 1。因此 `p_0_in` 实际指示了 8 位二进制加法的结果是否超出了最大值的范围。为了方便，后续我们定义 8 位二进制加法的和为 `sum[7:0]`。

加法器分析完毕，后面的 9 个 LUT 用于计算 op1 的 8 位二进制输入 `l_c[7:0]`。这 9 个 LUT 的连接也是非常复杂，既连接了加法器的输出，还把原始的 `l_a` 也引过来了。

![](https://s2.loli.net/2025/01/26/iNITlELSuoCsh3H.png)

我们从最低位开始分析，op1 的输入 `l_c[0]` 是从一个 LUT2 出来的，这个 LUT2（`inst_op1_1_i_8`）的 INIT 值为 `4'h9`，逻辑表达式为 `O=(I0&I1)|(~I0&~I1)`。`I0` 连接着 `l_a[0]`，`I1` 连接着 `p_0_in`，考虑把 `p_0_in` 作为选择信号，则逻辑表达式为：
```
l_c[0] = p_0_in ? (l_a[0]) : (~l_a[0])
```
`~l_a[0]` 还可以写为 `l_a[0] ^ 1`，那么事实上这就是刚才我们没有得到的 9 位二进制数加法和的最低位，也就是竖式里的“X”，也就是 `sum[0]`。

再看 `l_c[1]`，这是个 LUT3（`inst_op1_1_i_7`），`INIT` 值为 `8'hB4`，`I0` 连接着 `l_a[0]`，`I1` 连接着 `p_0_in`，`I2` 连接着 `inst_op1_1_i_11_n_7`（加法和的第二位）。同样用 `p_0_in` 做选择，逻辑表达式为
```
l_c[1] = p_0_in ? ((l_a[0] & sum[1]) | (~l_a[0] & ~sum[1])) : (sum[1])
```
这里把 `(l_a[0] & sum[1]) | (~l_a[0] & ~sum[1])` 写成异或的形式 `~(l_a[0] ^ sum[1])`，再把“非”给 `l_a[0]`，变成 `sum[0]`。
```
l_c[1] = p_0_in ? (sum[1] ^ sum[0]) : (sum[1])
```

下面是 `l_c[2]`，这是个 LUT4（`inst_op1_1_i_6`），`INIT` 值为 `16'hDF20`，`I0` 连接着 `p_0_in`，`I1` 连接着 `l_a[0]`，`I2` 连接着 `sum[1]`，`I3` 连接着 `sum[2]`。同样用 `p_0_in` 做选择，逻辑表达式为
```
l_c[2] = p_0_in ? ((~I3&I2&~I1) | (I3&~I2) | (I3&I1)) : (sum[2])
```
其中
```
(~I3&I2&~I1) | (I3&~I2) | (I3&I1)
= (~sum[2]&sum[1]&sum[0]) | (sum[2]&~sum[1]) | (sum[2]&~sum[0])
= (~sum[2]&sum[1]&sum[0]) | (sum[2]&(~sum[1]|~sum[0]))
= (~sum[2] & (sum[1]&sum[0])) | (sum[2] & ~(sum[1]&sum[0]))
= sum[2] ^ (sum[1] & sum[0])
```
感兴趣的话可以继续推导其余的 `l_c[3]~l_c[7]`，这里直接列出全部的式子（相信聪明的做题人也已经猜到规律了）：
```
l_c[0] = p_0_in ? (sum[0] ^ 1) : (sum[0])
l_c[1] = p_0_in ? (sum[1] ^ sum[0]) : (sum[1])
l_c[2] = p_0_in ? (sum[2] ^ (sum[1] & sum[0])) : (sum[2])
l_c[3] = p_0_in ? (sum[3] ^ (sum[2] & sum[1] & sum[0])) : (sum[3])
l_c[4] = p_0_in ? (sum[4] ^ (sum[3] & sum[2] & sum[1] & sum[0])) : (sum[4])
l_c[5] = p_0_in ? (sum[5] ^ (sum[4] & sum[3] & sum[2] & sum[1] & sum[0])) : (sum[5])
l_c[6] = p_0_in ? (sum[6] ^ (sum[5] & sum[4] & sum[3] & sum[2] & sum[1] & sum[0])) : (sum[6])
l_c[7] = p_0_in ? (sum[7] ^ (sum[6] & sum[5] & sum[4] & sum[3] & sum[2] & sum[1] & sum[0])) : (sum[7])
```
这么有规律的式子，是在进行什么操作呢？不难发现，当 `p_0_in` 为 1 时，实际上实现了一个“+1”的加法器，从 `sum[0]` 开始：
- `sum[0] + 1` 的和是 `sum[0] ^ 1`，进位是 `sum[0] & 1 = sum[0]`
- `sum[1] + sum[0]` 的和是 `sum[1] ^ sum[0]`，进位是 `sum[1] & sum[0]`
- `sum[2] + sum[1] & sum[0]` 的和是 `sum[2] ^ (sum[1] & sum[0])`，进位是 `sum[2] & sum[1] & sum[0]`
- ……

根据 `p_0_in` 的含义，我们可以用一句话描述从 `l_a` 到 `l_c` 的算法：计算 `l_a + 8'b00001111` 的和，若超过了 `8'd255`，则输出和的后 8 位再加一的值，否则直接输出和的后 8 位。

接下来，op1（指数运算）旁边还有一些 LUT，把模块的输入 `a` 引过来了，很容易知道它们的功能是判断 `a` 是否是 0，若是 0 则输出 0，否则输出 op1 的结果。

至此，op4 的电路分析完毕，那么 op4 究竟是不是我们猜测的有限域乘法呢？别急，其实有些内容被我们忽略了，在 Verilog 代码里，`pe__1` 里的 op4 的 `b` 端口其实是写了输入值的：
```
  op4__2 inst_op4
       (.a(top_line),
        .b({\<const1> ,\<const1> ,\<const1> ,\<const0> ,\<const0> ,\<const1> ,\<const0> ,\<const0> }),
        .c(tmp_res_1));
```
确实是定值 `8'b11100100`（`8'd228`），考虑到乘法应该是有交换律的，我们把这个数拿到 $\text{GF}(256)$ 的对数表查一下，刚好结果就是 15（`8'b00001111`）！所以前面我们的猜测是完全正确的，正因为输入是定值，连接到 `b` 的一整个 op2（对数运算）都被优化掉了，只剩下一个非常抽象的加法器。

对 `a` 和 `b` 分别取对数，相加并处理越界（实际上是模 255，但是自然溢出是模 256，所以溢出了要补加 1），再取指数，确实是有限域乘法的算法。

### PE ARRAY 流程分析
回到 pe，op3 是有限域加加法，op4 是有限域乘法，那么很明显 pe 里就是一个简单的乘加运算，表达式为：
```
data_o = top_line * coeff + data_i
```
![](https://s2.loli.net/2025/01/26/XvhIu6qkONUSrAH.png)

如前所述，每个时钟周期 pe 把乘加运算的结果向下传递，而最后一个 data_o 经过一个 op3 和输入数据相加，再经过一个 op4 乘以 1（相当于没乘），再通过 LUT2（另一个输入连接到了 reset，用于重置），最终成为 top_line 的值。

![](https://s2.loli.net/2025/01/26/DCw8FTKScIpiXRQ.png)

这是一种什么运算？

不如缩小规模，尝试模拟一下看看。我们假设只有两个 pe，对应的 coeff 分别是 2 和 1，然后依次输入数据 `1,0,0,0`，假设所有运算在实数域下进行。

![](https://s2.loli.net/2025/01/26/HtfEvcxSep8Ii5J.png)

似乎看不出什么规律。刚才的假设其实有点问题，因为在 $\text{GF}(256)$ 下，op3 既可以是加法，也可以是减法。我们尝试把 pe 中的加法器换成减法器：

![](https://s2.loli.net/2025/01/26/lDEqmJ2WwhgrFk3.png)

好像没啥变化，还是很抽象，但是如果我们把斜的数据流变成正的，把减法写成竖式呢？

![](https://s2.loli.net/2025/01/26/kIx2GHXJ9DEUfSK.png)

似乎有点意思，相信聪明的做题人应该能把剩下的东西补上了：

![](https://s2.loli.net/2025/01/26/udg4yQ8fn2MHU1L.png)

没错，这个 pe array 实现了一个多项式除法器。这个硬件结构的学名是“反馈移位寄存器”，正如前面分析的，从第一个 pe 一直流到最后一个 pe，这是移位；最后一个 pe 的数据和输入相加得到 top_line 再连接回每个 pe，这是反馈。

注意到除法竖式里面除数最高位的系数“1”没染色，事实上这个 1 对应的是计算 top_line 时候的 op4 对应的一个常数系数。但是聪明的做题人可能要问了，这里是不是少了一个除法模块？因为做多项式除法的时候，当前位的商应该是当前余数最高次项系数**除以**除式的最高次项系数。除以一个数，相当于乘以这个数的逆，而无论是在有限域还是实数域，1 的逆都是 1，因此这里看似是一个无效的乘法，实际上前面应该还有个求逆的电路被优化掉了。

![](https://s2.loli.net/2025/01/26/zcIWY3fHmOjweoR.png)

注意到除法竖式里面被除多项式最低两位的系数“0”也没染色，这是因为我们电路里输出数据是从 pe 的最末端输入的，相当于已经给被除多项式乘以了 $x^k$，其中 $k$ 是 pe 的个数。当然输入也可以在左边，这样的话实际效率是比较低的。

![](https://s2.loli.net/2025/01/26/nDPjSBpGEyQMNl1.png)

### 整体算法分析
分析了这么久，又是有限域又是多项式除法的，这到底是个啥算法？

不妨来看看 `EzRegs_tb.v` 里我们还没关注到的部分——输出的校验。
```verilog
    wire [0:8*(N+K)-1] data_final;
    generate
        for (i=0;i<N;i=i+1) begin
            assign data_final[i*8 +: 8] = flag_test_arr[i];
        end
        for (i=0;i<K;i=i+1) begin
            assign data_final[(8*N + i*8) +: 8] = data_o[K-1-i];
        end
    endgenerate

    wire [0:8*(N+K)-1] data_mask = 'h0000000000ff00ff00000000000000000000000000000000000000ffff000000000000ffff0000000000_000000ffffffffff00ffffffffffffffffffffffffffff00ffffffffffffff00ffffffffffff0000;
    wire [0:8*(N+K)-1] data_std  = 'h000000000034003300000000000000000000000000000000000000632d00000000000036370000000000_0000009fc64e8e240060ede11ed46ebb29bf829ed418da001425281233decd007be4de63872a0000;
    assign success = ((data_final & data_mask) == data_std);
```

我们已经知道 data_o 其实就是多项式除法的余式的系数，把这 40 个系数拼到一起，和 flag_test_arr 连接起来，一起放在 data_final 里。然后使用一个掩码“与”上 data_mask，检查是否等于 data_std。这个掩码只有 0x00 和 0xff 两种数值，0x00 “与”任何数一定还是 0x00，data_std 的对应位置确实是这样。实际上这个掩码是告诉你只有 0xff 的地方需要检查，其他位置可以忽略。再换句话说就是给出了所有 0xff 处的数值，要用这些信息恢复出 flag。那么这显然是一个信息编码与纠错的问题，又因为用到了有限域和多项式除法，可以猜测编码的算法是知名的 Reed-Solomon 算法（如果不了解，问 GPT 即可）。

### EXP 编写
简单了解一下 Reed-Solomon 算法，就不难~~拷打 GPT~~写出一份 EXP：
```python
import reedsolo as rs
N = 42
K = 40

# 别忘了指定生成多项式，默认值不是 0x12d
rs.init_tables(0x12d)

mesecc = bytes.fromhex("000000000034003300000000000000000000000000000000000000632d000000000000363700000000000000009fc64e8e240060ede11ed46ebb29bf829ed418da001425281233decd007be4de63872a0000")

# 已知信息越多，越容易解码，把已知的开头加上
mesecc = b'0ops{' + mesecc[5:]

# 由于已知哪些字节是丢失的，使用 RS 算法的 Erase 模式可以提升一倍的纠错能力
erase_pos =[]
for index, byte in enumerate(mesecc):
    if(byte == 0):
        erase_pos.append(index)

rmes, recc, errata_pos = rs.rs_correct_msg(
    mesecc,
    K,
    erase_pos=erase_pos
)
print(rmes)
```