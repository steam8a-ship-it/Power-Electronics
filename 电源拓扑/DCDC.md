# ⚡ DC-DC 变换器学习笔记

> 📘 本文件用于系统记录 DC-DC 变换器相关知识，包括基本原理、拓扑结构、重点公式、器件选型、驱动方式及实际设计经验。
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

DC-DC 变换器的作用是将一种直流电压转换为另一种直流电压。

常见拓扑：

| 拓扑 | 主要功能 | 是否隔离 | 典型应用 |
|---|---|---|---|
| BUCK | 降压 | ❌ | 高压转低压 |
| BOOST | 升压 | ❌ | 低压转高压 |
| BUCK-BOOST | 升降压 | ❌ | 输入变化范围较大 |
| Flyback 反激 | 升降压 | ✅ | 中小功率隔离电源 |
| Forward 正激 | 降压为主 | ✅ | 中等功率隔离电源 |

DC-DC 中经常需要关注的参数：

- 输入电压 $V_{in}$
- 输出电压 $V_o$
- 输出电流 $I_o$
- 占空比 $D$
- 开关频率 $f_s$
- 电感 $L$
- 输出电容 $C_o$
- 电流纹波 $\Delta I_L$
- 电压纹波 $\Delta V_o$
- MOSFET 电压、电流应力
- 效率与器件损耗
- CCM / DCM 工作模式

---

<a id="buck"></a>

# 2️⃣ BUCK 降压变换器

> BUCK 是最基本的降压型 DC-DC 变换器。
>
> 理想情况下：
>
> $$
> \boxed{V_o=D V_{in}}
> $$

---

## 2.1 🔌 BUCK 基本拓扑

### 异步 BUCK

主要器件：

- 上管 MOSFET
- 续流二极管
- 电感 $L$
- 输出电容 $C_o$

工作过程：

**MOSFET 导通：**

输入电源向负载供电，同时电感储能。

**MOSFET 关断：**

电感电流不能突变，通过续流二极管继续向负载供电。

### ✅ 优点

- 结构简单
- 成本低
- 驱动容易

### ❌ 缺点

- 二极管存在正向压降
- 大电流时损耗明显
- 效率低于同步 BUCK

---

### 同步 BUCK

同步 BUCK 使用一个 MOSFET 替代续流二极管。

主要器件：

- 上管 MOSFET
- 下管 MOSFET
- 电感 $L$
- 输出电容 $C_o$

### ✅ 优点

- 导通损耗小
- 效率高
- 适合低压大电流

### ❌ 缺点

- 驱动更加复杂
- 上下管需要死区时间
- 控制错误可能造成桥臂直通

> [!IMPORTANT]
> 同步 BUCK 上下 MOSFET 不能同时导通，否则会形成：
>
> `VIN → 上管 → 下管 → GND`
>
> 的直通回路。

---

## 2.2 🧮 BUCK 重点公式

以下公式主要针对理想 **CCM 连续导通模式**。

### ① 占空比

$$
\boxed{
D=\frac{V_o}{V_{in}}
}
$$

因此：

$$
\boxed{
V_o=D V_{in}
}
$$

---

### ② 开关周期

$$
T_s=\frac{1}{f_s}
$$

其中：

- $T_s$：开关周期
- $f_s$：开关频率

---

### ③ 电感电流纹波

MOSFET 导通时：

$$
V_L=V_{in}-V_o
$$

因此：

$$
\Delta I_{L,on}
=
\frac{(V_{in}-V_o)D}{Lf_s}
$$

稳态情况下：

$$
\boxed{
\Delta I_L=
\frac{(V_{in}-V_o)D}{Lf_s}
}
$$

也可以写成：

$$
\boxed{
\Delta I_L=
\frac{V_o(1-D)}{Lf_s}
}
$$

---

### ④ 电感平均电流

理想 BUCK 中：

$$
\boxed{
I_{L,avg}=I_o
}
$$

---

### ⑤ 电感峰值电流

$$
\boxed{
I_{L,peak}
=
I_o+\frac{\Delta I_L}{2}
}
$$

电感谷值：

$$
I_{L,min}
=
I_o-\frac{\Delta I_L}{2}
$$

---

### ⑥ 输出电压纹波

忽略电容 ESR：

$$
\boxed{
\Delta V_o
\approx
\frac{\Delta I_L}{8f_sC_o}
}
$$

考虑 ESR：

$$
\boxed{
\Delta V_o
\approx
\frac{\Delta I_L}{8f_sC_o}
+
\Delta I_L\cdot ESR
}
$$

---

## 2.3 🧲 BUCK 电感选型与计算

通常先确定允许的电感纹波：

$$
\Delta I_L=kI_o
$$

一般可取：

$$
k=20\%\sim40\%
$$

由电感纹波公式得到：

$$
\boxed{
L=
\frac{(V_{in}-V_o)D}
{f_s\Delta I_L}
}
$$

或者：

$$
\boxed{
L=
\frac{V_o(1-D)}
{f_s\Delta I_L}
}
$$

### 电感选型主要检查

- 电感值 $L$
- 饱和电流 $I_{sat}$
- RMS 电流
- DCR
- 磁芯材料
- 工作频率
- 温升
- 高温降额

饱和电流至少满足：

$$
I_{sat}>I_{L,peak}
$$

实际设计应留一定余量。

---

## 2.4 🌊 纹波系数

电感电流纹波系数：

$$
\boxed{
r=
\frac{\Delta I_L}{I_o}
}
$$

例如：

$$
r=0.3
$$

表示：

> 电感峰峰值纹波约为输出平均电流的 **30%**。

常见经验：

| 纹波系数 | 特点 |
|---:|---|
| 20% | 纹波较小，电感较大 |
| 30% | ⭐ 常用折中 |
| 40% | 电感较小，峰值电流较大 |

### 纹波太大

- 峰值电流增加
- 输出纹波增加
- MOSFET RMS 电流增加
- EMI 增强

### 纹波太小

- 所需电感增大
- 体积增加
- 成本增加

---

## 2.5 🚦 BUCK 如何驱动

### 异步 BUCK

只需要主动控制上管：

```text
PWM HIGH
   ↓
上管 ON
   ↓
电感储能

PWM LOW
   ↓
上管 OFF
   ↓
二极管续流
```

如果采用 **N 沟道 MOSFET 作为高边开关**，通常需要：

- 高边 Gate Driver
- Bootstrap 自举电路

---

### 同步 BUCK

需要同时控制上下 MOSFET：

```text
上管 ON
下管 OFF

    ↓

Dead Time

    ↓

上管 OFF
下管 ON
```

死区期间：

```text
上管 OFF
下管 OFF
```

主要作用：

> 防止上下 MOSFET 同时导通。

常见实现：

```text
MCU / PWM Controller
        ↓
   Gate Driver
        ↓
   HO        LO
    ↓        ↓
  上管      下管
```

常见半桥驱动端口：

- `HO`：高边输出
- `LO`：低边输出
- `VB`：Bootstrap 电源
- `VS`：开关节点
- `VCC`：驱动供电
- `COM`：驱动地

---

## 2.6 🔧 BUCK 简单设计流程

### Step 1：确定参数

$$
V_{in},\quad V_o,\quad I_o,\quad f_s
$$

### Step 2：计算占空比

$$
D=\frac{V_o}{V_{in}}
$$

### Step 3：选择纹波系数

$$
r=20\%\sim40\%
$$

所以：

$$
\Delta I_L=rI_o
$$

### Step 4：计算电感

$$
L=
\frac{(V_{in}-V_o)D}
{f_s\Delta I_L}
$$

### Step 5：计算峰值电流

$$
I_{L,peak}
=
I_o+\frac{\Delta I_L}{2}
$$

### Step 6：选择器件

检查：

- MOSFET耐压
- MOSFET电流
- $R_{DS(on)}$
- 电感饱和电流
- 电感DCR
- 输出电容
- ESR
- 驱动电压
- 开关频率
- 温升

---

## 📌 BUCK 一句话总结

> BUCK 本质上是通过 MOSFET 高频开关控制电感周期性储能和放能，再利用 LC 滤波获得较低且稳定的直流输出。

最核心的几个关系：

$$
\boxed{
V_o=D V_{in}
}
$$

$$
\boxed{
\Delta I_L=
\frac{(V_{in}-V_o)D}{Lf_s}
}
$$

$$
\boxed{
I_{L,peak}
=
I_o+\frac{\Delta I_L}{2}
}
$$

---

<a id="boost"></a>

# 3️⃣ BOOST 升压变换器

> 🚧 待学习后补充

计划记录：

- 基本拓扑
- CCM / DCM
- 占空比关系
- 电感设计
- MOSFET和二极管应力
- 驱动方式

---

<a id="buckboost"></a>

# 4️⃣ BUCK-BOOST 变换器

> 🚧 待学习后补充

计划记录：

- 反相 BUCK-BOOST
- 四开关 BUCK-BOOST
- 升降压工作模式
- 控制策略

---

<a id="flyback"></a>

# 5️⃣ Flyback 反激变换器

> 🚧 待学习后补充

计划记录：

- 反激基本原理
- 变压器实际上作为耦合电感
- DCM / CCM
- 匝比设计
- MOSFET电压应力
- RCD钳位
- 光耦反馈

---

<a id="forward"></a>

# 6️⃣ Forward 正激变换器

> 🚧 待学习后补充

计划记录：

- 正激基本原理
- 与反激的区别
- 变压器复位
- 输出电感
- 单管/双管正激
- 开关管电压应力

---

<a id="other"></a>

# 7️⃣ 其它拓扑

后续根据学习情况增加：

- 半桥
- 全桥
- 推挽
- LLC
- CLLC
- SEPIC
- Zeta
