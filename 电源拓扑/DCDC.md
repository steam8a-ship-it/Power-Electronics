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

DC-DC 中经常需要关注：

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

理想情况下：

$$
\boxed{
V_o=D V_{in}
}
$$

---

## 2.1 🔌 BUCK 基本拓扑

### 🔹 异步 BUCK

主要器件：

- 上管 MOSFET
- 续流二极管
- 电感 $L$
- 输出电容 $C_o$
- 负载

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

### 🔹 同步 BUCK

<img width="941" height="314" alt="image" src="https://github.com/user-attachments/assets/c5ae9db5-879d-417b-a60e-c0824c63de2d" />

详解见 https://blog.csdn.net/weixin_42107954/article/details/131000253


同步 BUCK 使用一个 MOSFET 替代续流二极管。

主要器件：

- 上管 MOSFET
- 下管 MOSFET
- 电感 $L$
- 输出电容 $C_o$

### ✅ 优点

- 导通损耗小
- 效率高
- 适合低压、大电流输出

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

---

### ① 占空比

理想 BUCK：

$$
V_o=D V_{in}
$$

因此：

$$
\boxed{
D=\frac{V_o}{V_{in}}
}
$$

其中：

- $V_{in}$：输入电压
- $V_o$：输出电压
- $D$：PWM 占空比

---

### ② 开关周期

$$
\boxed{
T_s=\frac{1}{f_s}
}
$$

其中：

- $T_s$：开关周期
- $f_s$：开关频率

---

### ③ 电感电流纹波

MOSFET 导通时，电感两端电压为：

$$
V_L=V_{in}-V_o
$$

根据电感基本关系：

$$
V_L=L\frac{di_L}{dt}
$$

因此，导通阶段电感电流上升量为：

$$
\Delta I_{L,\mathrm{on}}
=
\frac{(V_{in}-V_o)D}{L f_s}
$$

MOSFET 关断后，电感进入续流阶段。

稳态情况下：

$$
\Delta I_{L,\mathrm{on}}
=
\Delta I_{L,\mathrm{off}}
$$

所以电感峰峰值纹波电流为：

$$
\boxed{
\Delta I_L
=
\frac{(V_{in}-V_o)D}{L f_s}
}
$$

也可以写成：

$$
\boxed{
\Delta I_L
=
\frac{V_o(1-D)}{L f_s}
}
$$

---

### ④ 电感平均电流

理想 BUCK 中：

$$
\boxed{
I_{L,\mathrm{avg}}=I_o
}
$$

即：

> 电感平均电流基本等于输出电流。

---

### ⑤ 电感峰值与谷值电流

电感峰值电流：

$$
\boxed{
I_{L,\mathrm{peak}}
=
I_o+\frac{\Delta I_L}{2}
}
$$

电感谷值电流：

$$
\boxed{
I_{L,\mathrm{min}}
=
I_o-\frac{\Delta I_L}{2}
}
$$

> [!WARNING]
> 电感饱和电流 $I_{sat}$ 必须高于 $I_{L,\mathrm{peak}}$，实际设计还应该留有一定裕量。

---

### ⑥ 输出电压纹波

忽略输出电容 ESR 时：

$$
\boxed{
\Delta V_o
\approx
\frac{\Delta I_L}{8f_sC_o}
}
$$

考虑电容 ESR 后：

$$
\boxed{
\Delta V_o
\approx
\frac{\Delta I_L}{8f_sC_o}
+
\Delta I_L\cdot ESR
}
$$

因此输出纹波主要与以下参数有关：

- 电感纹波电流 $\Delta I_L$
- 开关频率 $f_s$
- 输出电容 $C_o$
- 电容 ESR

---

## 2.3 🧲 BUCK 电感选型与计算

通常先确定允许的电感纹波：

$$
\Delta I_L=kI_o
$$

其中 $k$ 为电感电流纹波系数。

一般可取：

$$
k=20\%\sim40\%
$$

由电感纹波公式：

$$
\Delta I_L
=
\frac{(V_{in}-V_o)D}{L f_s}
$$

可以得到电感值：

$$
\boxed{
L=
\frac{(V_{in}-V_o)D}{f_s\Delta I_L}
}
$$

或者：

$$
\boxed{
L=
\frac{V_o(1-D)}{f_s\Delta I_L}
}
$$

---

### ✅ 电感选型主要检查

- 电感值 $L$
- 饱和电流 $I_{sat}$
- RMS 电流
- 直流电阻 DCR
- 磁芯材料
- 工作频率
- 温升
- 高温降额

基本要求：

$$
I_{sat}>I_{L,\mathrm{peak}}
$$

实际设计一般需要继续留一定裕量。

---

## 2.4 🌊 纹波系数

电感电流纹波系数可以定义为：

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

### 📊 常见取值

| 纹波系数 | 特点 |
|---:|---|
| 20% | 纹波较小，所需电感较大 |
| 30% | ⭐ 常用折中 |
| 40% | 电感较小，但峰值电流和纹波增大 |

### 纹波太大

可能导致：

- 电感峰值电流增大
- 输出纹波增大
- MOSFET RMS 电流增大
- EMI 增强
- 器件损耗增加

### 纹波太小

可能导致：

- 所需电感增大
- 电感体积增大
- 成本提高

---

## 2.5 🚦 BUCK 如何驱动

### 🔹 异步 BUCK

异步 BUCK 主要主动控制上管 MOSFET。

```text
PWM HIGH
   ↓
上管 ON
   ↓
输入向电感和负载供能
   ↓
电感储能
```

```text
PWM LOW
   ↓
上管 OFF
   ↓
二极管续流
   ↓
电感继续向负载供能
```

如果采用 **N 沟道 MOSFET 作为高边开关**，通常需要：

- 高边 Gate Driver
- Bootstrap 自举电路

原因是高边 MOSFET 导通后源极电位会升高，因此：

> 栅极电压必须继续高于源极电压，才能保持 MOSFET 导通。

---

### 🔹 同步 BUCK

同步 BUCK 需要控制上下两个 MOSFET。

基本时序：

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

> 防止上下 MOSFET 同时导通形成桥臂直通。

---

### 🔧 常见驱动结构

```text
MCU / PWM Controller
        ↓
    Gate Driver
        ↓
   ┌────┴────┐
   ↓         ↓
  HO        LO
   ↓         ↓
 上管       下管
```

常见半桥驱动端口：

- `HO`：High-side Output，高边驱动输出
- `LO`：Low-side Output，低边驱动输出
- `VB`：Bootstrap 高边驱动电源
- `VS`：高边浮动参考 / 开关节点
- `VCC`：驱动芯片供电
- `COM`：驱动地

> [!TIP]
> 实际设计中还需要关注栅极驱动电阻、MOSFET 栅极电荷、驱动电流、开关速度和死区时间。

---

## 2.6 🔧 BUCK 简单设计流程

### Step 1：确定基本参数

$$
V_{in},\quad V_o,\quad I_o,\quad f_s
$$

---

### Step 2：计算占空比

$$
D=\frac{V_o}{V_{in}}
$$

---

### Step 3：选择纹波系数

一般：

$$
r=20\%\sim40\%
$$

因此：

$$
\Delta I_L=rI_o
$$

---

### Step 4：计算电感

$$
L=
\frac{(V_{in}-V_o)D}{f_s\Delta I_L}
$$

---

### Step 5：计算电感峰值电流

$$
I_{L,\mathrm{peak}}
=
I_o+\frac{\Delta I_L}{2}
$$

---

### Step 6：选择实际器件

重点检查：

- MOSFET 耐压
- MOSFET 电流
- $R_{DS(on)}$
- MOSFET 栅极电荷
- 电感饱和电流
- 电感 DCR
- 输出电容
- 电容 ESR
- 驱动能力
- 开关频率
- 功耗
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
\Delta I_L
=
\frac{(V_{in}-V_o)D}{L f_s}
}
$$

$$
\boxed{
I_{L,\mathrm{peak}}
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
- 开关导通与关断过程
- CCM / DCM
- 占空比关系
- 电感设计
- MOSFET电压和电流应力
- 二极管应力
- 输出纹波
- 驱动方式

---

<a id="buckboost"></a>

# 4️⃣ BUCK-BOOST 变换器

> 🚧 待学习后补充

计划记录：

- 反相 BUCK-BOOST
- 四开关 BUCK-BOOST
- BUCK工作区
- BOOST工作区
- 升降压工作模式
- 控制策略
- 器件应力

---

<a id="flyback"></a>

# 5️⃣ Flyback 反激变换器

> 🚧 待学习后补充

计划记录：

- 反激基本工作原理
- 导通与关断能量传递
- 变压器 / 耦合电感
- CCM / DCM
- 匝比设计
- MOSFET电压应力
- 二极管应力
- RCD钳位
- 光耦反馈
- 辅助绕组

---

<a id="forward"></a>

# 6️⃣ Forward 正激变换器

> 🚧 待学习后补充

计划记录：

- 正激基本原理
- 与反激的区别
- 变压器复位
- 输出电感
- 单管正激
- 双管正激
- MOSFET电压应力
- 输出整流与续流

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
