
  # ⚡ DC-DC 变换器学习笔记

> 📘 本文件用于系统记录 DC-DC 变换器相关知识，包括基本原理、拓扑结构、重点公式、器件选型、纹波设计、驱动方式及实际设计经验。
>
> 后续随着学习逐步补充 BUCK、BOOST、BUCK-BOOST、反激、正激等常见拓扑。

---

## 📚 目录

- [1. DC-DC 基础知识](#dcdc-basic)
- [2. BUCK 降压变换器](#buck)
- [3. BOOST 升压变换器](#boost)
- [4. BUCK-BOOST 变换器](#buckboost)
- [5. Flyback 反激变换器](#flyback)
- [6. Forward 正激变换器](#forward)
- [7. 其它拓扑](#other)

---

<a id="dcdc-basic"></a>

# 1️⃣ DC-DC 基础知识

DC-DC 变换器的作用是：

> 将一种直流电压转换为另一种直流电压。

常见拓扑：

| 拓扑 | 主要功能 | 是否隔离 | 常见应用 |
|---|---|---|---|
| BUCK | 降压 | ❌ | 高压转低压 |
| BOOST | 升压 | ❌ | 低压转高压 |
| BUCK-BOOST | 升降压 | ❌ | 宽输入范围 |
| Flyback 反激 | 升降压 | ✅ | 中小功率隔离电源 |
| Forward 正激 | 降压为主 | ✅ | 中等功率隔离电源 |

DC-DC 设计中经常关注：

- 输入电压 Vin
- 输出电压 Vo
- 输出电流 Io
- 占空比 D
- 开关频率 fs
- 电感 L
- 输出电容 Co
- 电感电流纹波
- 输出电压纹波
- MOSFET 电压应力
- MOSFET 电流应力
- 二极管应力
- 器件损耗
- 转换效率
- CCM / DCM 工作模式
- 环路稳定性
- 驱动与死区

---

<a id="buck"></a>

# 2️⃣ BUCK 降压变换器

> BUCK 是最基本的降压型 DC-DC 变换器。

理想情况下：

```math
V_o=D V_{in}
```

也就是说：

> 输出电压主要由输入电压和 PWM 占空比决定。

---

## 2.1 🔌 同步 BUCK 与异步 BUCK

### 🔹 异步 BUCK

异步 BUCK 主要由：

- 上管 MOSFET
- 续流二极管
- 电感 L
- 输出电容 Co
- 负载

组成。

### 工作过程

#### MOSFET 导通

输入电源经过 MOSFET 和电感向负载供电，同时给电感储能。

```text
VIN
 │
MOSFET
 │
电感
 │
输出
```

#### MOSFET 关断

电感电流不能突变，因此通过续流二极管继续向负载提供电流。

```text
电感
 │
负载
 │
续流二极管
```

### ✅ 优点

- 电路简单
- 成本低
- 控制简单
- 驱动容易

### ❌ 缺点

- 二极管存在正向压降
- 大电流时二极管损耗明显
- 效率低于同步 BUCK

---

### 🔹 同步 BUCK

同步 BUCK 使用一个 MOSFET 替代续流二极管。

<img width="941" height="314" alt="image" src="https://github.com/user-attachments/assets/c5ae9db5-879d-417b-a60e-c0824c63de2d" />

详解见 https://blog.csdn.net/weixin_42107954/article/details/131000253

主要由：

- 上管 MOSFET
- 下管 MOSFET
- 电感 L
- 输出电容 Co

组成。

### ✅ 优点

- 下管导通损耗较低
- 效率较高
- 特别适合低压、大电流场合

### ❌ 缺点

- 驱动更加复杂
- 上下管必须加入死区时间
- 控制错误可能导致上下管直通

> [!IMPORTANT]
> 同步 BUCK 上下 MOSFET 不能同时导通，否则会形成：

```text
VIN
 │
上管
 │
下管
 │
GND
```

也就是电源直接短路，产生很大的直通电流。

---

## 2.2 🧮 BUCK 重点公式

下面公式主要针对理想 **CCM 连续导通模式**。

---

### ① 占空比

理想 BUCK：

```math
V_o=D V_{in}
```

因此：

```math
D=\frac{V_o}{V_{in}}
```

其中：

- Vin：输入电压
- Vo：输出电压
- D：占空比

例如：

```text
Vin = 100V
Vo  = 50V
```

则理想占空比约为：

```math
D=\frac{50}{100}=0.5
```

即：

```text
D = 50%
```

---

### ② 开关周期

```math
T_s=\frac{1}{f_s}
```

其中：

- Ts：开关周期
- fs：开关频率

例如：

```text
fs = 100kHz
```

则：

```math
T_s=\frac{1}{100000}=10\mu s
```

---

### ③ 电感电流纹波

MOSFET 导通时，电感两端电压：

```math
V_L=V_{in}-V_o
```

根据电感基本关系：

```math
V_L=L\frac{di_L}{dt}
```

MOSFET 导通时间：

```math
t_{on}=D T_s
```

所以导通阶段电感电流上升量：

```math
\Delta I_{L,on}
=
\frac{(V_{in}-V_o)D}{L f_s}
```

MOSFET 关断后，电感进入续流阶段。

理想稳态下：

```math
\Delta I_{L,on}
=
\Delta I_{L,off}
```

所以电感峰峰值纹波电流：

```math
\Delta I_L
=
\frac{(V_{in}-V_o)D}{L f_s}
```

利用：

```math
D=\frac{V_o}{V_{in}}
```

还可以写成：

```math
\Delta I_L
=
\frac{V_o(1-D)}{L f_s}
```

---

### ④ 电感平均电流

理想 BUCK 中：

```math
I_{L,avg}=I_o
```

即：

> 电感平均电流基本等于输出电流。

---

### ⑤ 电感峰值与谷值电流

电感峰值电流：

```math
I_{L,peak}
=
I_o+\frac{\Delta I_L}{2}
```

电感谷值电流：

```math
I_{L,min}
=
I_o-\frac{\Delta I_L}{2}
```

因此电感电流大致呈三角波变化：

```text
          /\
         /  \
        /    \
-------/------\-------  Io
      /        \
```

> [!WARNING]
> 电感饱和电流必须高于电感峰值电流，否则电感可能进入饱和状态，导致电流迅速增大。

---

### ⑥ 输出电压纹波

忽略输出电容 ESR 时：

```math
\Delta V_o
\approx
\frac{\Delta I_L}{8 f_s C_o}
```

考虑 ESR 后：

```math
\Delta V_o
\approx
\frac{\Delta I_L}{8 f_s C_o}
+
\Delta I_L\cdot ESR
```

因此输出电压纹波主要与以下参数有关：

- 电感纹波电流
- 开关频率
- 输出电容容量
- 电容 ESR

一般来说：

```text
L ↑ → 电流纹波 ↓

C ↑ → 电压纹波 ↓

fs ↑ → 电流纹波和电压纹波 ↓
```

但提高开关频率会增加开关损耗。

---

## 2.3 🧲 BUCK 电感选型与计算

BUCK 电感通常根据允许的电感电流纹波进行设计。

先确定允许的纹波：

```math
\Delta I_L=k I_o
```

其中：

- k：电感电流纹波系数
- Io：输出电流

通常可以取：

```text
20% ～ 40%
```

较常见的折中值：

```text
30%
```

---

### 🔢 电感计算

由：

```math
\Delta I_L
=
\frac{(V_{in}-V_o)D}{L f_s}
```

得到：

```math
L
=
\frac{(V_{in}-V_o)D}
{f_s\Delta I_L}
```

也可以写成：

```math
L
=
\frac{V_o(1-D)}
{f_s\Delta I_L}
```

---

### 📌 电感选型重点

实际选电感不能只看电感值，还要看：

- 电感值 L
- 饱和电流 Isat
- RMS 电流
- DCR
- 磁芯材料
- 工作频率
- 温升
- 体积
- 高温降额

其中最重要的一项：

```math
I_{sat}>I_{L,peak}
```

工程上一般还需要留出一定裕量。

---

### 🧠 简单例子

假设：

```text
Vin = 100V
Vo  = 50V
Io  = 2A
fs  = 100kHz
纹波系数 = 30%
```

首先：

```math
D=\frac{50}{100}=0.5
```

纹波电流：

```math
\Delta I_L
=
0.3\times2
=
0.6A
```

计算电感：

```math
L
=
\frac{(100-50)\times0.5}
{100000\times0.6}
```

得到：

```math
L\approx417\mu H
```

所以实际可以选择附近的标准电感值，并结合纹波重新计算。

---

## 2.4 🌊 纹波系数

电感电流纹波系数定义为：

```math
r=
\frac{\Delta I_L}{I_o}
```

例如：

```math
r=0.3
```

表示：

> 电感峰峰值纹波电流约为平均输出电流的 30%。

---

### 📊 常见纹波系数

| 纹波系数 | 特点 |
|---:|---|
| 20% | 电流纹波较小，但所需电感较大 |
| 30% | ⭐ 常见折中设计 |
| 40% | 电感较小，但峰值电流和输出纹波增加 |

---

### 纹波过大

可能导致：

- 电感峰值电流增大
- MOSFET 电流应力增加
- 输出电压纹波增加
- RMS 电流增加
- EMI 增加
- 电感更容易饱和

---

### 纹波过小

可能导致：

- 电感值较大
- 电感体积增大
- 成本增加
- 动态响应可能变慢

因此设计时并不是纹波越小越好，而是需要折中。

---

## 2.5 🚦 BUCK 如何驱动

### 🔹 异步 BUCK

异步 BUCK 只需要主动控制上管 MOSFET。

工作逻辑：

```text
PWM HIGH
   ↓
上管 MOSFET ON
   ↓
输入向电感和负载供能
   ↓
电感储能
```

然后：

```text
PWM LOW
   ↓
MOSFET OFF
   ↓
续流二极管导通
   ↓
电感继续向负载供能
```

---

### 高边 MOSFET 驱动

如果上管使用 N 沟道 MOSFET：

```text
VIN
 │
MOSFET
 │
SW
```

MOSFET 导通以后，源极电压会上升。

因此：

> 栅极驱动电压必须继续高于源极一定电压，才能保证 MOSFET 完全导通。

所以常使用：

- High-Side Gate Driver
- Bootstrap 自举电路

---

### 🔹 同步 BUCK

同步 BUCK 需要控制上下两个 MOSFET。

典型驱动时序：

```text
上管 ON
下管 OFF

    ↓

Dead Time

    ↓

上管 OFF
下管 ON
```

然后再次经过死区切换回上管。

---

### ⏱ Dead Time 死区时间

死区期间：

```text
上管 OFF
下管 OFF
```

主要作用：

> 防止上下 MOSFET 因为开关延迟而短时间同时导通。

死区太小：

- 容易发生直通

死区太大：

- 体二极管导通时间增加
- 损耗增加
- 效率下降

所以死区也需要折中设计。

---

### 🔧 常见驱动结构

```text
MCU / PWM Controller
        │
        ▼
   Gate Driver
      │    │
     HO    LO
      │    │
      ▼    ▼
    上管  下管
```

常见半桥驱动芯片端口：

| 引脚 | 含义 |
|---|---|
| HO | 高边 MOSFET 栅极驱动 |
| LO | 低边 MOSFET 栅极驱动 |
| VB | 高边浮动电源 |
| VS | 高边浮动参考 / SW节点 |
| VCC | 驱动电源 |
| COM | 驱动地 |

---

### 驱动还需要关注

- 栅极驱动电压
- MOSFET 栅极电荷 Qg
- 驱动芯片峰值电流
- 栅极电阻
- 上升时间
- 下降时间
- dv/dt
- Miller 平台
- 死区时间
- Bootstrap 电容

---

## 2.6 🔧 BUCK 简单设计流程

### Step 1：确定输入输出参数

```text
Vin
Vo
Io
fs
```

---

### Step 2：计算占空比

```math
D=\frac{V_o}{V_{in}}
```

---

### Step 3：选择纹波系数

通常：

```text
20% ～ 40%
```

例如选择：

```text
30%
```

则：

```math
\Delta I_L=0.3I_o
```

---

### Step 4：计算电感

```math
L
=
\frac{(V_{in}-V_o)D}
{f_s\Delta I_L}
```

---

### Step 5：计算电感峰值电流

```math
I_{L,peak}
=
I_o+\frac{\Delta I_L}{2}
```

---

### Step 6：选择 MOSFET

重点检查：

- 耐压 VDS
- 最大漏极电流
- RDS(on)
- 栅极电荷 Qg
- 开关速度
- 结温
- 高温 RDS(on) 增大
- 封装散热能力

---

### Step 7：选择电感

重点检查：

- 电感值
- 饱和电流
- RMS 电流
- DCR
- 磁芯损耗
- 温升

---

### Step 8：选择输出电容

重点检查：

- 容量
- ESR
- 纹波电流能力
- 耐压
- 温度等级
- 寿命

---

### Step 9：设计驱动

检查：

- Gate Driver
- Bootstrap
- 栅极电阻
- 死区时间
- 驱动电压
- 栅极波形

---

### Step 10：实测验证

重点观察：

- SW节点波形
- MOSFET VGS
- MOSFET VDS
- 电感电流
- 输出纹波
- MOSFET温升
- 电感温升
- 效率
- EMI

---

## 📌 BUCK 一句话总结

> BUCK 本质上是通过 MOSFET 高频开关控制电感周期性储能和放能，再利用 LC 滤波获得较低且稳定的直流输出。

最核心的几个关系：

```math
V_o=D V_{in}
```

```math
\Delta I_L
=
\frac{(V_{in}-V_o)D}
{L f_s}
```

```math
I_{L,peak}
=
I_o+\frac{\Delta I_L}{2}
```

实际设计时主要围绕：

```text
占空比
+
开关频率
+
电感
+
电容
+
MOSFET
+
驱动
+
损耗
+
温升
```

展开。

---

<a id="boost"></a>

# 3️⃣ BOOST 升压变换器

> 🚧 待学习后补充

后续计划记录：

- BOOST 基本拓扑
- MOSFET导通过程
- MOSFET关断过程
- CCM / DCM
- 输入输出关系
- 占空比
- 电感设计
- 输出电容设计
- MOSFET应力
- 二极管应力
- 驱动方式
- 输出纹波
- 实际设计流程

---

<a id="buckboost"></a>

# 4️⃣ BUCK-BOOST 变换器

> 🚧 待学习后补充

后续计划记录：

- 反相 BUCK-BOOST
- 非反相 BUCK-BOOST
- 四开关 BUCK-BOOST
- BUCK模式
- BOOST模式
- 升降压切换
- 占空比关系
- 电感设计
- 控制策略
- 器件应力

---

<a id="flyback"></a>

# 5️⃣ Flyback 反激变换器

> 🚧 待学习后补充

后续计划记录：

- 反激基本原理
- MOSFET导通储能
- MOSFET关断传能
- 变压器与耦合电感
- CCM
- DCM
- 匝比
- 励磁电感
- MOSFET电压应力
- 二极管电压应力
- 漏感尖峰
- RCD钳位
- TVS钳位
- 光耦反馈
- 辅助绕组

---

<a id="forward"></a>

# 6️⃣ Forward 正激变换器

> 🚧 待学习后补充

后续计划记录：

- 正激基本原理
- 与反激的区别
- 变压器实时传能
- 输出电感
- 磁芯复位
- 单管正激
- 双管正激
- MOSFET应力
- 输出整流
- 续流
- 驱动方式

---

<a id="other"></a>

# 7️⃣ 其它拓扑

后续根据学习情况增加：

- Push-Pull 推挽
- Half-Bridge 半桥
- Full-Bridge 全桥
- SEPIC
- Zeta
- LLC
- CLLC
