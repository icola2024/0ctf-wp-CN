## EzLUTs
### 总览
打开 `EzLUTs_top_synth.v`，文件非常长，但是很快就能注意到里面除了 OBUF 和 IBUF 就只有各种大小的 LUT。而 `EzLUTs_tb.v` 文件很简略，仅仅是把 `flag_test` 作为 `EzLUTs_top` 模块的输入。前面提到 LUT 的连接组合一定是组合逻辑，所以立刻就能获得输出值。`EzLUTs_top` 模块只有一个一位的输出：`success`。很显然我们的目标就是给 `EzLUTs_top` 这个组合逻辑输入一串 flag，使其输出 `success` = 1。

### LUT 实例分析
再次仔细察看 `EzLUTs_top_synth.v`，不难发现 LUT 的 `INIT` 值只有四种模式：
- `[69]+`（1320个实例）
- `(?=[69]*0)[069]+`（107个实例）
- `[0]*[8421][0]*`（45个实例）
- `[0]+4114`（1个实例）

接下来我们逐类分析：

对于 `[69]+` 模式，所有 `INIT` 值去重之后只有 5 种不同的值：
```verilog
4'h6
8'h96
16'h6996
32'h96696996
64'h6996966996696996
```
还记得 `4'h6` 吗？我们在 EzLogic 里已经遇到过它了，这代表了用 LUT2 实现的一个异或门。那么我们有理由猜测，`8'h96` 是用 LUT3 实现的一个三输入异或门。把 `8'h96` 展开成二进制 `10010110`，然后同样写出真值表：

|输入I2|输入I1|输入I0|对应的查找表位置|输出O|
|---|---|---|---|---|
|0|0|0|1001011**0**|0|
|0|0|1|100101**1**0|1|
|0|1|0|10010**1**10|1|
|0|1|1|1001**0**110|0|
|1|0|0|100**1**0110|1|
|1|0|1|10**0**10110|0|
|1|1|0|1**0**010110|0|
|1|1|1|**1**0010110|1|

容易验证，确实是三输入异或门！

对后面的 LUT4、LUT5、LUT6 同理，这里不多赘述。

总结一下，这种模式下的 LUT2~LUT6 对应的约束条件是：
```
O=I0^I1
O=I0^I1^I2
O=I0^I1^I2^I3
O=I0^I1^I2^I3^I4
O=I0^I1^I2^I3^I4^I5
```

接下来分析最简单的 `[0]*[8421][0]*` 模式，相信聪明的做题人已经发现了，这类查找表展开成二进制的话，其中有且只有一位是“1”，其他全是“0”。

同样对所有这类 `INIT` 值进行去重，结果只有 14 种：
```verilog
32'h20000000
32'h08000000
64'h0000000000000002
64'h0000000000000010
64'h0000000000000020
64'h0000000000000800
64'h0000000100000000
64'h0000000800000000
64'h0000001000000000
64'h0000008000000000
64'h0008000000000000
64'h0800000000000000
64'h1000000000000000
64'h8000000000000000
```
以 `64'h0000000800000000` 为例，展开成二进制，可以发现其中的“1”在从右往左数的第 35 个（从 0 开始），35 转换成二进制是“100011”，故要使这个 LUT6 输出 1，输入必须是“100011”，也就是：I5=1，I4=0，I3=0，I2=0，I1=1，I0=1。很明显，这实现了一类“与”和“非”构成的逻辑。使用“与”和“非”运算来表示输入输出的约束条件：
```
O=I0&I1&~I2&~I3&~I4&I5
```

然后是 `(?=[69]*0)[069]+` 模式，不同于 `[69]+`，这里面混进了 0 元素，但是有 69，就一定还是和异或运算有关。

还是先对 `INIT` 值去重，结果只有 10 种：
```verilog
64'h0000000069969669
64'h0000000096696996
64'h0000699669960000
64'h0069690069000069
64'h0096960096000096
64'h6000006000000000
64'h6996000000006996
64'h9600009600969600
64'h9669000000009669
64'h9669699600000000
```
以 `64'h0000000069969669` 为例，列出真值表：
|I5|I4|I3|I2|I1|I0|O|
|---|---|---|---|---|---|---|
|0|0|0|0|0|0|1|
|0|0|0|0|0|1|0|
|0|0|0|0|1|0|0|
|0|0|0|0|1|1|1|
|0|0|0|1|0|0|0|
|0|0|0|1|0|1|1|
|0|0|0|1|1|0|1|
|0|0|0|1|1|1|0|
|0|0|1|0|0|0|0|
|0|0|1|0|0|1|1|
|0|0|1|0|1|0|1|
|0|0|1|0|1|1|0|
|0|0|1|1|0|0|1|
|0|0|1|1|0|1|0|
|0|0|1|1|1|0|0|
|0|0|1|1|1|1|1|
|0|1|0|0|0|0|0|
|0|1|0|0|0|1|1|
|0|1|0|0|1|0|1|
|0|1|0|0|1|1|0|
|0|1|0|1|0|0|1|
|0|1|0|1|0|1|0|
|0|1|0|1|1|0|0|
|0|1|0|1|1|1|1|
|0|1|1|0|0|0|1|
|0|1|1|0|0|1|0|
|0|1|1|0|1|0|0|
|0|1|1|0|1|1|1|
|0|1|1|1|0|0|0|
|0|1|1|1|0|1|1|
|0|1|1|1|1|0|1|
|0|1|1|1|1|1|0|
|1|X|X|X|X|X|0|

规律不难发现，当 I0~I4 里面有奇数个 1 时，输出就是 0；当 I0~I4 里面有偶数个 1 时，输出就是 1。若 I5 是 1，那么输出一定是 0。这样的逻辑可以用“与”“非”“异或”三者来描述：
```
O=~I5&~(I4^I3^I2^I1^I0)
```
其他查找表也同理，比如 `64'h0096960096000096` 对应的约束关系是：
```
O=(I0^I1^I2)&~(I3^I4^I5)
```

最后是 `[0]+4114` 模式，因为只有一个实例 `64'h0000000000004114`，直接进行分析，列出真值表：

|I5|I4|I3|I2|I1|I0|O|
|---|---|---|---|---|---|---|
|0|0|0|0|0|0|0|
|0|0|0|0|0|1|0|
|0|0|0|0|1|0|1|
|0|0|0|0|1|1|0|
|0|0|0|1|0|0|1|
|0|0|0|1|0|1|0|
|0|0|0|1|1|0|0|
|0|0|0|1|1|1|0|
|0|0|1|0|0|0|1|
|0|0|1|0|0|1|0|
|0|0|1|0|1|0|0|
|0|0|1|0|1|1|0|
|0|0|1|1|0|0|0|
|0|0|1|1|0|1|0|
|0|0|1|1|1|0|1|
|0|0|1|1|1|1|0|
|X|1|X|X|X|X|0|
|1|X|X|X|X|X|0|

关注 O=1 的行：

![](https://s2.loli.net/2025/01/24/yALrYNliUaw5bcf.png)

发现其实和“69”的模式一样，也是异或：
```
O=~(I0)&~(I4)&~(I5)&(I1^I2^I3)
```

回顾这四种模式，其实可以用统一的模型来表述：
```math
\text{O}=\neg \text{P}_1 \wedge \neg \text{P}_2\wedge\cdots\neg \text{P}_{n}\wedge\text{Q}_1\wedge \text{Q}_2\wedge\cdots \text{Q}_{m}
```
其中
```math
\begin{array}{c}
\text{P}_{i} = \text{I}_{p_{i,1}}\oplus\text{I}_{p_{i,2}}\oplus\cdots\\
\text{Q}_{i} = \text{I}_{q_{i,1}}\oplus\text{I}_{q_{i,2}}\oplus\cdots\\
\end{array}
```
为了把查找表解析成 POXOR（Product of Exclusive OR）的形式，我们可以写个脚本来处理。


```python
import itertools
import math

def get_constraints_from_LUT(LUT_INIT: str):
    N = round(math.log2(len(LUT_INIT)*4))
    table_bin = bin(int(LUT_INIT, 16))[2:].zfill(len(LUT_INIT)*4)

    arr: list[str] = []
    for (index, value) in enumerate(table_bin[::-1]):
        if (value == '1'):
            arr.append(bin(index)[2:].zfill(N))

    constraints: list[tuple[list[int], int]] = []
    confirmed: list[int] = []
    nums: list[int] = list(range(N))

    # 暴力查找 POXOR（Product of Exclusive OR）模式的所有因子
    # 从一个变量开始
    for k in range(1, N+1):
        combinations = list(itertools.combinations(nums, k))
        # 枚举 k 个变量的所有异或组合
        for comb in combinations:
            # 已有约束条件，就不再判断了
            if (set(confirmed) & set(comb)):
                continue

            for t in [0,1]:
                # 判断 arr 里所有数当前 comb 位上的值异或起来是否都是 t
                flag = 0
                for cond in arr:
                    xor_value = 0
                    for i in list(comb):
                        xor_value ^= int(cond[(N-1) - i])
                    flag += xor_value
                if (flag == t * len(arr)):
                    constraints.append((list(comb), t))
                    confirmed.extend(list(comb))
    return constraints

def print_constraints(constraints: list[tuple[list[int], int]]):
    parts = []
    for c in constraints:
        part = '^'.join([f'I{idx}' for idx in c[0]])
        if (c[1] == 0):
            part = f"~({part})"
        else:
            part = f"({part})"
        parts.append(part)
    print("O=" + '&'.join(parts))
```

尝试一下：


```python
print_constraints(get_constraints_from_LUT("0069690069000069"))
```

    O=~(I0^I1^I2)&~(I3^I4^I5)
    

### LUT 解析
这个就没啥可说的，直接上正则表达式。有些人似乎都用上语法解析器了，大可不必啊。


```python
class LUT:
    def __init__(self, name: str, table: str, type: int):
        self.name = name
        self.table = table
        self.table_bin = bin(int(table, 16))[2:].zfill((1<<type))
        self.type = type
        self.inputs: list[str] = []
        self.output: str = None

    def add_input(self, net: str):
        self.inputs.append(net)

    def set_output(self, net: str):
        self.output = net

    def __str__(self):
        return (
f"""
LUT{self.type} #(
    .INIT({(1<<self.type)}'h{self.table})
) {self.name} (
{''.join([f"    .I{i}({v}),{chr(10)}" for (i,v) in enumerate(self.inputs)])}{f"    .O({self.output})"}
);
"""
        )
```


```python
import re
pattern1 = re.compile(r"LUT(\d) #\(\s*\.INIT\(\d{1,2}'h(\d+)\)\)\s*(\w+)\s*\((.+?)\);", re.RegexFlag.DOTALL)
pattern2 = re.compile(r"\.(I\d|O)\(([\\\w\[\] ]+)\)")

def parse_LUT_io(inst: LUT, content: str):
    matches = pattern2.findall(content)
    for match in matches:
        port_name = match[0]
        net_name = match[1]
        if (port_name == 'O'):
            inst.set_output(net_name)
        else:
            inst.add_input(net_name)
            
def parse_LUTs(text: str) -> list[LUT]:
    matches = pattern1.findall(text)
    luts = []
    for match in matches:
        lut_type = match[0]
        lut_name = match[2]
        lut_table = match[1]
        lut_io = match[3]
        lut_inst = LUT(lut_name, lut_table, int(lut_type))
        parse_LUT_io(lut_inst, lut_io)
        luts.append(lut_inst)
    return luts
```

写个测试验证一下：


```python
test = parse_LUTs(
    """
  LUT6 #(
    .INIT(64'h6996966996696996)) 
    success_OBUF_inst_i_1369
       (.I0(data_IBUF[297]),
        .I1(data_IBUF[129]),
        .I2(data_IBUF[249]),
        .I3(data_IBUF[97]),
        .I4(data_IBUF[233]),
        .I5(data_IBUF[329]),
        .O(success_OBUF_inst_i_1369_n_0));
    """
)
print(str(test[0]))
```

    
    LUT6 #(
        .INIT(64'h6996966996696996)
    ) success_OBUF_inst_i_1369 (
        .I0(data_IBUF[297]),
        .I1(data_IBUF[129]),
        .I2(data_IBUF[249]),
        .I3(data_IBUF[97]),
        .I4(data_IBUF[233]),
        .I5(data_IBUF[329]),
        .O(success_OBUF_inst_i_1369_n_0)
    );
    
    

接下来把整个文件的 LUT 都解析出来，顺便构建一下字典，方便后面使用。
- `lut_output_dic`：由 LUT 的输出网络名获取 LUT 编号，可以会有多个
- `net_num_dic`：由网络名获取网络编号
- `num_net_dic`：由网络编号获取网络名


```python
top_file = open(r"EzLUTs_top_synth.v", 'r').read()
luts = parse_LUTs(top_file)
luts_cnt = len(luts)
lut_output_dic: dict[str, list[int]] = {}
for idx, lut in enumerate(luts):
    entry = lut_output_dic.get(lut.output)
    if (entry):
        entry.append(idx)
    else:
        lut_output_dic[lut.output] = [idx]
print(luts_cnt)
```

    1473
    


```python
nets: set[str] = set()
for lut in luts:
    nets = nets.union(set(lut.inputs))
    nets.add(lut.output)
net_num_dic: dict[str, int] = {v:i for (i,v) in enumerate(nets)}
num_net_dic: dict[int, str] = {i:v for (i,v) in enumerate(nets)}
nets_cnt = len(net_num_dic)
print(nets_cnt)
```

    1809
    

### LUT 组合分析
对于 POXOR 形式的组合逻辑，一旦我们确定了输出的值，就可以得知输入的约束，一定是如下的两种形式：
- 某个输入的值是 0/1
- 某些输入异或起来的值是 0/1

观察 `success` 信号连接的 LUT6，`INIT` 值为 `64'h8000000000000000`，也就是要使 `success` = 1，输入必须全为 1。

![](https://s2.loli.net/2025/01/24/1qs4ZvLJQhCWtM8.png)

我们有理由相信，从电路的输出逆推下去，最终可以得到一串关于模块输入（`data`，或者 `data_IBUF`）的约束关系。而且由异或运算的结合律，这个约束关系一定也是上述的两种形式。


```python
from collections import deque
# 第一阶段
# BFS 方法遍历所有输出网络约束为“值等于0/1”的 LUT
# 从 success_OBUF 连接的 LUT 开始
bfs_list: deque[tuple[int, int]] = deque([(lut_output_dic['success_OBUF'][0], 1)])
processed_luts: set[int] = set()

# 存储所有“某些网络的值异或起来等于 0/1”形式的约束，待下一轮处理
inferred_values: list[tuple[set[int], int]] = []

while(len(bfs_list) > 0):
    (lut_idx, output_net_value)= bfs_list.popleft()
    if (lut_idx in processed_luts):
        continue
    processed_luts.add(lut_idx)
    lut = luts[lut_idx]
    output_net_idx = net_num_dic[lut.output]
    cons = get_constraints_from_LUT(lut.table)

    # 如果 LUT 的约束不是纯异或，也就是严格 POXOR，但是输出为 0
    # a&b=0 可能有 a=0 或 b=0 两种情况
    if (len(cons) != 1 and output_net_value == 0):
        raise "more than one possibility"

    for con in cons:
        g_i_nets = [lut.inputs[i] for i in con[0]]
        if (len(g_i_nets) == 1): # “某个网络的值等于 0/1”
            source_luts = lut_output_dic.get(g_i_nets[0])
            if (source_luts):
                for s_lut in source_luts:
                    bfs_list.append((s_lut, con[1]))
        else: # “某些网络的值异或起来等于 0/1”
            g_o_value = con[1] if output_net_value == 1 else (1 - con[1])
            inferred_values.append((set([net_num_dic[net] for net in g_i_nets]), g_o_value))
```


```python
# 第二阶段
# 剩下的应该都是纯异或的 LUT
# 把输出和输入的连接关系提取出来
conn_set: dict[int, set[int]] = {}
for i in range(luts_cnt):
    lut_idx = i
    if (lut_idx in processed_luts):
        continue
    processed_luts.add(lut_idx)
    lut = luts[lut_idx]
    
    cons = get_constraints_from_LUT(lut.table)
    if (len(cons) != 1):
        raise "POXOR LUT should not appear at this stage"
    con = cons[0]
    conn_set[net_num_dic[lut.output]] = set(
        [net_num_dic[lut.inputs[i]] for i in con[0]]
    )
```


```python
print(len(processed_luts))
print(len(inferred_values))
print(len(conn_set))
```

    1473
    336
    1159
    

执行到这里，其实我们已经要到终点了。`inferred_values` 存储了 336 个约束关系，相当于 336 个方程。而未知数 `data_IBUF` 刚好也是 336 位。但是 `inferred_values` 里并不是关于 `data_IBUF` 的直接表达式，而是需要通过 `conn_set` 进行变量代换，才能得到关于 `data_IBUF` 的直接表达式。


```python
# DFS 把 conn_set[net] 里的网络逐层代换，直到网络名以 data_IBUF 开头
def dfs_expand_to_data_in(net: int):
    if (num_net_dic[net].startswith("data_IBUF")):
        return set([net])
    conn = conn_set.get(net)
    if (conn is None):
        raise "Error, no connection"
    return set().union(*[dfs_expand_to_data_in(n) for n in conn])

final_equations: list[tuple[set[int], int]] = []
for inferred in inferred_values:
    input_net_idx = set().union(*[dfs_expand_to_data_in(n) for n in inferred[0]])
    eq_v = inferred[1]
    eq_s = [int(re.match("data_IBUF\[(\d+)\]", num_net_dic[net])[1]) for net in input_net_idx]
    final_equations.append((eq_s, eq_v))
```

为了形象说明，不如把 `final_equations` 打印出来：


```python
for eq in final_equations:
    print(f"{' ^ '.join([f'data_IBUF[{i}]' for i in eq[0]])} = {eq[1]}")
```

    data_IBUF[100] ^ data_IBUF[60] ^ data_IBUF[164] ^ data_IBUF[68] ^ data_IBUF[148] ^ data_IBUF[236] ^ data_IBUF[156] ^ data_IBUF[140] ^ data_IBUF[276] ^ data_IBUF[332] ^ data_IBUF[4] ^ data_IBUF[324] ^ data_IBUF[316] ^ data_IBUF[260] ^ data_IBUF[284] ^ data_IBUF[116] ^ data_IBUF[92] ^ data_IBUF[292] ^ data_IBUF[84] ^ data_IBUF[28] ^ data_IBUF[180] ^ data_IBUF[268] ^ data_IBUF[300] = 1
    data_IBUF[91] ^ data_IBUF[267] ^ data_IBUF[331] ^ data_IBUF[299] ^ data_IBUF[179] ^ data_IBUF[291] ^ data_IBUF[315] ^ data_IBUF[163] ^ data_IBUF[59] ^ data_IBUF[275] ^ data_IBUF[67] ^ data_IBUF[259] ^ data_IBUF[283] ^ data_IBUF[139] ^ data_IBUF[27] ^ data_IBUF[155] ^ data_IBUF[235] ^ data_IBUF[147] ^ data_IBUF[115] ^ data_IBUF[3] ^ data_IBUF[99] ^ data_IBUF[323] ^ data_IBUF[83] = 0
    data_IBUF[273] ^ data_IBUF[329] ^ data_IBUF[313] ^ data_IBUF[265] ^ data_IBUF[153] ^ data_IBUF[289] ^ data_IBUF[1] ^ data_IBUF[81] ^ data_IBUF[137] ^ data_IBUF[297] ^ data_IBUF[145] ^ data_IBUF[57] ^ data_IBUF[177] ^ data_IBUF[281] ^ data_IBUF[257] ^ data_IBUF[65] ^ data_IBUF[113] ^ data_IBUF[25] ^ data_IBUF[321] ^ data_IBUF[233] ^ data_IBUF[97] ^ data_IBUF[161] ^ data_IBUF[89] = 0
    data_IBUF[127] ^ data_IBUF[263] ^ data_IBUF[295] ^ data_IBUF[63] ^ data_IBUF[79] ^ data_IBUF[207] ^ data_IBUF[31] ^ data_IBUF[175] ^ data_IBUF[335] ^ data_IBUF[231] ^ data_IBUF[311] ^ data_IBUF[255] ^ data_IBUF[95] ^ data_IBUF[247] ^ data_IBUF[303] ^ data_IBUF[287] ^ data_IBUF[327] ^ data_IBUF[15] ^ data_IBUF[279] ^ data_IBUF[23] ^ data_IBUF[183] ^ data_IBUF[239] = 1
    data_IBUF[310] ^ data_IBUF[174] ^ data_IBUF[22] ^ data_IBUF[246] ^ data_IBUF[326] ^ data_IBUF[262] ^ data_IBUF[62] ^ data_IBUF[14] ^ data_IBUF[254] ^ data_IBUF[182] ^ data_IBUF[294] ^ data_IBUF[30] ^ data_IBUF[206] ^ data_IBUF[78] ^ data_IBUF[230] ^ data_IBUF[126] ^ data_IBUF[334] ^ data_IBUF[94] ^ data_IBUF[278] ^ data_IBUF[286] ^ data_IBUF[302] ^ data_IBUF[238] = 0
    data_IBUF[91] ^ data_IBUF[171] ^ data_IBUF[331] ^ data_IBUF[195] ^ data_IBUF[187] ^ data_IBUF[179] ^ data_IBUF[291] ^ data_IBUF[315] ^ data_IBUF[227] ^ data_IBUF[75] ^ data_IBUF[59] ^ data_IBUF[123] ^ data_IBUF[19] ^ data_IBUF[275] ^ data_IBUF[67] ^ data_IBUF[139] ^ data_IBUF[27] ^ data_IBUF[283] ^ data_IBUF[107] ^ data_IBUF[99] ^ data_IBUF[323] ^ data_IBUF[131] ^ data_IBUF[83] = 0
    data_IBUF[226] ^ data_IBUF[330] ^ data_IBUF[170] ^ data_IBUF[90] ^ data_IBUF[98] ^ data_IBUF[74] ^ data_IBUF[122] ^ data_IBUF[106] ^ data_IBUF[274] ^ data_IBUF[194] ^ data_IBUF[138] ^ data_IBUF[282] ^ data_IBUF[186] ^ data_IBUF[58] ^ data_IBUF[18] ^ data_IBUF[82] ^ data_IBUF[322] ^ data_IBUF[290] ^ data_IBUF[130] ^ data_IBUF[66] ^ data_IBUF[178] ^ data_IBUF[314] ^ data_IBUF[26] = 1
    data_IBUF[273] ^ data_IBUF[329] ^ data_IBUF[313] ^ data_IBUF[73] ^ data_IBUF[169] ^ data_IBUF[289] ^ data_IBUF[121] ^ data_IBUF[185] ^ data_IBUF[137] ^ data_IBUF[81] ^ data_IBUF[17] ^ data_IBUF[129] ^ data_IBUF[177] ^ data_IBUF[57] ^ data_IBUF[281] ^ data_IBUF[105] ^ data_IBUF[65] ^ data_IBUF[225] ^ data_IBUF[193] ^ data_IBUF[25] ^ data_IBUF[321] ^ data_IBUF[97] ^ data_IBUF[89] = 1
    data_IBUF[217] ^ data_IBUF[329] ^ data_IBUF[169] ^ data_IBUF[1] ^ data_IBUF[153] ^ data_IBUF[81] ^ data_IBUF[185] ^ data_IBUF[137] ^ data_IBUF[121] ^ data_IBUF[17] ^ data_IBUF[209] ^ data_IBUF[105] ^ data_IBUF[161] ^ data_IBUF[41] ^ data_IBUF[9] ^ data_IBUF[241] ^ data_IBUF[25] ^ data_IBUF[113] ^ data_IBUF[97] ^ data_IBUF[233] = 0
    ......
    data_IBUF[100] ^ data_IBUF[20] ^ data_IBUF[212] ^ data_IBUF[44] ^ data_IBUF[164] ^ data_IBUF[236] ^ data_IBUF[156] ^ data_IBUF[140] ^ data_IBUF[188] ^ data_IBUF[332] ^ data_IBUF[4] ^ data_IBUF[12] ^ data_IBUF[116] ^ data_IBUF[172] ^ data_IBUF[220] ^ data_IBUF[244] ^ data_IBUF[108] ^ data_IBUF[124] ^ data_IBUF[84] ^ data_IBUF[28] = 0
    data_IBUF[127] ^ data_IBUF[239] ^ data_IBUF[215] ^ data_IBUF[143] ^ data_IBUF[167] ^ data_IBUF[119] ^ data_IBUF[31] ^ data_IBUF[175] ^ data_IBUF[159] ^ data_IBUF[103] ^ data_IBUF[191] ^ data_IBUF[47] ^ data_IBUF[247] ^ data_IBUF[87] ^ data_IBUF[223] ^ data_IBUF[15] ^ data_IBUF[7] ^ data_IBUF[23] ^ data_IBUF[111] ^ data_IBUF[335] = 0
    

### 有限域的魔法
好了，那么剩下的事情就交给数学了。现在的问题是：如何解一个关于异或运算的方程组？

事实上，异或运算可以看成有限域 $\text{GF}(2)$ 上的加法，而一个方程里是否含有 `data_IBUF[i]` 可以用 $\text{GF}(2)$ 上的乘法来控制。举个例子，假设未知数 `data` 只有 5 个，对于方程
```
data[4] ^ data[3] ^ data[0] = 1
```
可以写成：
```
1 * data[4] + 1 * data[3] + 0 * data[2] + 0 * data[1] + 1 * data[0] = 1
```
其中的加法和乘法都是有限域的运算符。

很显然这就是求解线性方程组的标准模式。根据有限域的性质，解线性方程组仍然可以用传统的高斯消元等方法。下面使用 numpy 和 galois 库进行求解。


```python
import numpy as np
import galois

N = len(final_equations)
GF2 = galois.GF(2)

M = GF2.Zeros((N, N))
b = GF2.Zeros(N)

for i, eq in enumerate(final_equations):
    for j in eq[0]:
        M[i, j] = GF2(1)
    b[i] = eq[1]

data_bits_arr = np.dot(np.linalg.inv(M), b)
```

最后，解码出 flag


```python
data_bits = ''.join([str(i) for i in data_bits_arr])
data_bytes = [int(data_bits[i:i+8], 2) for i in range(0, len(data_bits), 8)]
flag = bytes(data_bytes).decode()
print(flag)
```

    0ops{ce5cff29-4d78-4c72-ace3-e4f9405dda30}
    
