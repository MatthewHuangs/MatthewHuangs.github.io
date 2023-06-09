---
layout:       post
title:        "非科班 VASP 材料计算入门"
author:       "matt"
header-style: text
catalog:      true
tags:
    - physics
    - VASP
    - quantus
---

> VASP 是 Vienna Ab-inito Simulation Package 的缩写，它是基于密度泛函理论下并利用平面波赝势方法进行从头分子动力学和第一原理电子结构计算的软件包，是目前材料模拟和计算物质科学研究中非常流行的商用软件之一。

## 一、基本概念

**密度泛函理论**：研究多电子体系电子结构的量子力学方法。

**赝势**：或有效势（effective potential），是指在对能带结构进行数值计算时所引入的一个虚拟的势。

**分子动力学**：是一套分子模拟方法，该方法主要是依靠计算机来模拟分子、原子体系的运动，是一种多体模拟方法。

**第一原理**：或称从头算，指从基本的物理学定律出发，不外加假设与经验拟合的推导与计算。

**波函数**：量子力学中用来描述粒子的德布罗意（物质波，概率波，空间中某点某时刻可能出现的几率，大小受波动规律支配）波的函数，是空间和时间的函数，用ψ=ψ(x，y，z，t)表示。

**晶胞**：晶胞是能完整反映晶体内部原子或离子在三维空间分布的化学结构特征的基本重复平行六面体单元。大部分时候等同于原胞（晶格中最小重复单元）。

```txt
晶格是理论模型，有限种，是理论原子结构排列最小单元。晶胞是实际模型，无限种。

晶体结构 = 点阵（在空间任何方向上均为周期排列的无限个全同点的集合） + 结构基元（构成晶体的基本结构单元）
```

**基矢**：确定原胞大小的边矢量。原胞以基矢为周期排列，因此，基矢的大小又成为晶格常数。

**DOS**：density of state，态密度。状态密度或态密度为某一能量附近每单位能量区间里微观状态的数目，又叫做能态密度。

**势能面**：原子核左边和坐标的关系。单点能；结构优化是从一个初始结构出发，找到一个局部的能量最小点（盆地，多维的情况）；分子动力学模拟MD，在这个是势能面上原子核以牛顿运动方程不断的演化。

**结构优化**：也叫结构迟豫。是指对整个输入体系的坐标进行调整，得到一个相对稳定的基态结构。结构优化分**原子迟豫**和**电子迭代**两个嵌套的过程（电子迭代嵌套在原子迟豫中），达到原子迟豫收敛标准时进行下一步计算，直到达到自动中断或者最大原子迟豫步数。结果可以得到稳定结构的体系的总能量、晶体结构参数。得到CONTCAR。

**静态自洽计算**：是在结构优化的基础上，在体系能量达到较低、体系较稳定的情况下固定原子的位置坐标（即结构优化完的CONTCAR），再对体系中的电子进行调整（通过自洽迭代求解薛定谔方程，围观描述电子状态），以达到体系的最低能量。
完整地计算出体系基态下费米能级、 电子的波函数、电荷密度等信息，可以直接分析原子间的键合作用，也可以在非自洽之后进一步分析晶体的电子结构和材料的相关性质。得到自洽的电荷密度CHG、CHGCAR和E-fermi，EIGENCAR文件与syml文件通过脚本生成band和dos。

**非自洽计算**：是在自洽基础上改变k点（重新生成k点）等参数，根据不同需要选取能量或势函数或电子密度作为初始值，进行非自洽迭代计算，可用于求解DOS，能带（电子结构分析）或者光学等其他性质。

_<mark>说明：其他重要的基本概念会在文中出现的位置下发进行描述，在一开始了解这么多即可往下看。</mark>_  

## 二、编译安装

自行Google，利用压缩包含带的makefile即可。
tools/目录下有小程序可以辅助计算。

```txt
功能:
1.能量计算
相稳定性，相图；缺陷形成能；电化学电压曲线/反应自由能；表面/界面稳定性分析；力学性质
2.动力学计算
过渡态计算；反应路径预测；分子动力学计算；晶格动力学计算
3.电子态计算
成键分析；态密度，能带结构；氧化还原反应电子转移；极化与磁性分析；电子激发态分析

缺陷:
1.不适合大空间，大量原子的计算模拟；
2.不适合高分子，有机物，中大型团簇等非周期性结构计算；
3.不适合长时间的分子动力学模拟；
4.不适合大尺度缺陷（位错，断裂，无定型晶界）
```  

## 三、主要的输入/输出文件

一个软件的使用搞清楚输入、输出是第一步。

### 1.输入文件

VASP程序的主要输入文件有INCAR、POTCAR、POSCAR、KPOINTS，其他复杂的输入文件暂不考虑。

文件名 | 作用
-- | --
INCAR | 控制了vasp进行何种性质的计算，以及设置了计算方式中的一些重要参数
POTCAR | 包含了体系中各类元素的赝势
POSCAR | 描述了所计算的体系的晶胞参数，包括基矢或平移矢量、晶格常数、原子位置、分子动力学计算时原子的初始速度（不常用）等信息
KPOINTS | 描述了不可约布里渊区中k点取样，即k点设置

#### （1）INCAR 文件

It determines "what to do and how to do it". 尽量简洁。

```shell
SYSTEM = fcc si # 计算任务名
ISTART = 0 	    #是否读取WAVECAR里波函数信息
ICHARG = 2 	    #控制如何做电荷密度猜想
ENCUT  = 240 	 #平面波的截断能[eV]
ISMEAR = 0 	    #如何加弥散，处理分数占据的轨道
SIGMA  = 0.1  	 #弥散的展宽[eV]
```

- SYSTEM
计算任务名，随意
- ISTART
波函数初猜，WAVECAR保存计算波函数信息，0表示由程序初猜产生；==1==从WAVECAR里读取；还有2~3。结构优化和静态自洽计算都设置ISTART=0从头算，非自洽计算设置ISTART=1接着计算。
- ICHARG
电荷密度初猜，CHGCAR保存电荷密度信息，0由WAVECAR计算电荷信息；==1==从CHGCAR里读取外推；没有响应文件则自动2通过原子电荷密度叠加产生初猜；还有3、10、11、12。
- ENCUT
控制平面波的截断能（单位eV），即使用的平面波基组的大小。一般使用POTCAR给出的建议范围的最大值，或ENMAX的1.0到1.3倍。
- PREC
总体计算精度，有Low、Med、High、Normal、Single、Accurate。
- ISPIN
1：非自旋极化（适合非磁性体系），2：自旋极化（适合铁磁、反铁磁体系）。
- ISMEAR
轨道分数占据，采用某种smearing方法，-5：四面体方法（适合半导体和绝缘体）；0：高斯方法（适合导体、半导体和绝缘体）；N：Methfessel-Paxton方法（适合导体）。
- SIGMA
控制弥散的展宽。
- ALGO
电子波函数优化算法，N：DAV算法，收敛好速度慢；V：RMM算法，收敛差速度快；F：N和V的结合，类似V。
- EDIFF
自洽循环收敛标准，一般设为1E-5~1E-6。
- NELM
自洽循环最大计算步数。
- AMIX
自洽循环中电荷密度混合时新电荷密度的含量，默认0.4。
- ISIF
结构优化内容，2：固定晶胞优化离子；6：离子固定优化晶胞；3：离子和晶胞一起优化。
- NEDOS
态密度数据点个数，一般取1000~3000。
- LORBIT
总态密度轨道投影
- RWIGS
原子Wigner-Seitz半径，控制分态密度强度。
- LWAVE、LCHARG
输出波函数和电荷密度
- NCORE
并行计算单个轨道所有的核心数量

- 定义离子实或原子的优化
IBRION 原子位置优化的方法
POTIM 移动步长和步数：0：分子动力学模拟；1：准牛顿法；2：共轭梯度法；5：振动频率计算；6：弹性常数计算。
EDIFFG：离子位置优化收敛标准
NSW：离子位置优化最大步数（IBRION=1、2），分子动力学模拟步数（IBRION=0）

#### （2）POTCAR 文件

Potentials，赝势文件，用赝势文件的格式相应的命令把各元素的赝势**合并**到一个文件POTCAR中。不编译，每个元素都有对应的一个POTCAR文件，可以在VASP官网下载。

```shell
 PAW_PBE Zr 08Apr2002
 4.00000000000000000
 parameters from PSCTR are:
   VRHFIN =Zr: 5s4d5p
   LEXCH  = PE
   EATOM  =    77.8472 eV,    5.7216 Ry
...
   ENMAX  =  154.632; ENMIN  =  115.974 eV
 ...
 End of Dataset
 
 PAW_PBE Cu 05Jan2001
 ...
 End of Dataset
```

==注意==：根据不同的处理方式，可以将POTCAR文件分为不同类型。例如，根据处理了半芯态有A，A_sv和A_pv；根据ENMAX的大小有A，A_s和A_h。

#### （3）POSCAR 文件

Position，描述所计算体系的晶胞参数，原子个数及晶胞中原子的位置，以及分子动力学计算时原子的初始速度（不常用）。

```shell
Si-fcc                  !注释行，简短描述体系。FCC为面心立方晶格
5.43                    !基矢的缩放系数，可认为是晶格常数
0.00    0.50    0.50    !基矢除以缩放系数后的，与第二行的值一起描述基矢
0.50    0.00    0.50    !
0.50    0.50    0.00    !
2                       !原子个数
Direct                  !表示原子的坐标是相对于基矢给出的（分数坐标，晶格的几分之几）。为Carti时表示原子的坐标是以卡笛尔坐标系给出的绝对坐标
0.00    0.00    0.00    !原子的位置，有几个就有几行
0.25    0.25    0.25    !
```

#### （4）KPOINTS 文件

设置布里渊区k点网格取样大小或能带结构计算时沿高对称方向的k点。在这样一个K（频率）空间进行计算，定义K空间采样密度。KPOINTS中k点越多，计算越精确。

- 手动输入（自定义各个k点的坐标和权重）
仅适合能带计算时
- Line-mode
再计算能带时用，Reciprocal表示各k点相对于倒格子基矢来写，Cartesian表示各k点相对于卡笛尔直角坐标系。
- 程序自动产生k点
最常见，定义网格取样大小。自动产生k点，注释行下一行必须设为0。

```shell
Name           # 随便写
0              # 0表示各自自动生成
G              # Gamma centered Monkhorst-Pack grids
3 3 1          # 3*3*1表示在晶格的a、b方向取3个K点，c方向取1个k点，一共9个。由于对称性会减去很多k点。
0.0 0.0 0.0    # No shift
```

**说明**：
1）第一布里渊区（Brillouin zone）
是动量空间中晶体倒易点阵的原胞。周期性介质中的所有布洛赫波能在此空间中完全确定。

2）简约布里渊区
在对称性比较高的晶系的第一布里渊区内，我们需要计算的能带，很多部分都是重复的（可以通过对称性操作重复出来）。因此为了减少一些计算量，我们可以选出一系列高对称点围出一个简约布里渊区。

3）倒易空间（reciprocal space）
物理学中描述空间波函数的傅里叶变换后的周期性的一种方式。相对于正晶格所描述的实空间周期性，倒晶格描述的是动量空间，亦可认为是k空间的周期性。

4）能带
在形成分子时，原子轨道构成具有分立能级的分子轨道。由原子轨道所构成的分子轨道的数量非常之大，以至于可以将所形成的分子轨道的能级看成是准连续的，即形成了能带（准确来说是电子能带，描述了禁止或允许电子所带有的能量）。  

### 2.输出文件

VASP程序的输出文件主要有OUTCAR、CHG、CHGCAR、WAVECAR、DOSCAR、EIGENVAL、OSZICAR、CONTCAR、PCDAT、IBZKPT、XDATCAR。

文件名 | 作用
-- | --
OUTCAR| 包含了vasp计算后得到的绝大部分结果，每步迭代的详细情况
CHG | 体系的电荷密度文件，精度低
CHGCAR | 体系的电荷密度文件，精度高
WAVECAR | 体系的电子波函数，二进制文件
DOSCAR | 体系的电子态密度（density of state，DOS）
EIGENVAL | 体系的本征值，即轨道能量和电子占据，每个k点分别对应一套能级和占据
**OSZICAR** | 每次迭代或离子移动情况的简单汇总（能量信息）
**CONTCAR** | 给出的例子进行弛豫时，每次移动后体系的晶格参数（稳定结构坐标）
PCDAT | 有关分子动力学模拟中的一些结果
IBZKPT | 不可约布里渊区k点的坐标，可当作KPOINTS用
XDATCAR | 在结构优化或MD计算中保存每一步的结构信息
REPORT | 分子动力学MD的输出信息总结

```shell
# 指定标准输出和标准错误文件
mpirun -n 2 {vasp path} 1> vasp.out 2> vasp.err
```

#### （1）OUTCAR 文件

OUTCAR文件非常大，除非是必须的信息，不然可以从其他文件中获取信息。

包含了vasp计算后得到的绝大部分结果，每步迭代的详细情况。INCAR中的NWRITE关键字控制OUTCAR输出的详细程度，默认为2，可不写。

```txt
VASP编译、运行信息
---------------------------------
INCAR,POTCAR,POSCAR注释等相关信息
---------------------------------
ion 离子实信息
---------------------------------
.
.
.
---------------------------------
----- Iteration    1(   1)  -----
第一次迭代信息：
  计算用时（查看LOOP电子布/LOOP+离子步，才开始就可以查看用时是否合理）
  计算结果,能量
----- Iteration    1(   2)  -----
第二次迭代信息：
  计算用时
  计算结果,能量
---------------------------------
...
----- Iteration    2(   1)  -----
第二次迭代信息：
  计算用时
  计算结果,能量
---------------------------------
```

查看所计算体系的各个参数如下：

- 体积  
  
```shell
# 可以添加 | wc -l 统计行数
$ grep 'volume' OUTCAR

volume/ion in A,a.u. =
  给出以A^3/atom和a.u.^3/atom为单位的体积。
volume of cell :
  给出以A^3/unit cell为单位的体积。
```

- 总能（TOTEN，totally energy）  

```shell
# ISMEAR=-5时
$ grep 'TOTEN' OUTCAR
  取free energy TOTEN，与energy without entropy（通过估算电子熵，把T*S在总能中减掉）相等

# ISMEAR等于其他值时
$ grep 'entropy=' OUTCAR
  取energy whithout entropy=后面的值
```

- 费米能级  

```shell
# 最高占据能带的最高点的能量，每个irreducible k点都会有一套能量
$ grep 'Fermi' OUTCAR | tail -1
```

- 倒格子基矢  

```shell
$ vi OUTCAR
> g/reciprocal lattice vectors 或 g/recip
```

- 原子的坐标和受力信息  

```shell
$ vi OUTCAR
> g/POSITION
> g/TOTAL-FORCE
```

原子受力的单位是eV/angstrom

- 最终的能量  
取energy(sigma->0)

- 总结  

```shell
total amount of memory used by VASP

General timing and accounting information for this job
```

#### （2）CONTCAR 文件

VASP计算最后输出的一个结构信息。
给出的离子进行弛豫（Relaxation，在某一个渐变物理过程中，从某一个状态逐渐地恢复到平衡态的过程）时，每次移动后体系的晶格参数，与POSCAR的内容相同，可直接拷贝成POSCAR进行后续计算，中断后亦可。

```shell
Cu(50)Zr(50)  # Name 
   20.8000000000000  # 晶胞扩展系数
   1.0000000000000000    0.0000000000000000    0.0000000000000000  # 晶格矢量
   0.0000000000000000    1.0000000000000000    0.0000000000000000
   0.0000000000000000    0.0000000000000000    1.0000000000000000
   Zr   Cu  # 元素符号
   250   250  # 每种元素对应的原子个数
Direct  # 表示分数坐标
   0.9751273876113978  0.0696249851382856  0.0880676998578145  # 原子核坐标
   0.7885795399471279  0.9515124194039911  0.8085391066035814
   ...
   # 比POSCAR多出的信息，对应每个原子核的速度。静态计算时为0，MD计算时非0
   ...
```

#### （3）CHG 和 CHGCAR 文件

体系的电荷密度文件，还包含晶格矢量、原子核坐标。其中CHGCAR中的数据可以用来处理电荷密度图，电荷差分密度图ELF等信息。
CHG在结构优化和分子模拟计算中每10步模拟一次，精度低；CHGCAR保存计算最后一次的电荷密度，精度高。

#### （4）OSZICAR 文件

我们读取VASP计算结果的时候要取OSZICAR中最后一个E0的能量，也就是OUTCAR中最后一个energy(sigma->0)的能量。

==说明==：我们关心的应当是结果中的结构、能量、电学性质（导电性）。
多体的构型和能量的变化规律是非常复杂的，无法翻越势垒（势能比附近的势能都高的空间区域，基本上就是极值点附近的一小片区域），初始状态只会找到局部稳定的构型结构。  

## 三、使用说明

### 1.运行

yhrun -N 2 -n 24 -p debug /{path}/vasp
参数 | 含义
-- | --
yhrun | slurm作业管理系统中，并行执行mpi程序的命令，类似mpirun
-N | 任务所需的总节点数
-n | 任务所需的总核数
-p | 计算分区
/{path}/vasp_std | vasp_std可执行程序所在位置，vasp有std、gam、ncl三种模式

![VASP运行后的结果](https://km.woa.com/asset/e78675aa72734ea6a1a1bd3cb788c68f?height=550&width=576)

### 2.计算过程

![enter image description here](https://km.woa.com/asset/908b0140b8a44cfdbc684578974ee1e8?height=447&width=506)

#### 步骤一：结构优化

- 作用
结构优化是指对整个输入体系的坐标进行调整，得到一个相对稳定的基态结构,因此可以得到体系的总能量、晶体结构参数。

- 主要输出结果  

```txt
准备INCAR、POSCAR、POTCAR、KPOINTS四个输入文件
```

OSZICAR文件：优化的过程信息，关心最后一步结果。
CONTCAR文件：优化后的晶格参数和原子位置，并将文件作为POSCAR进行下一步的计算。

```shell
cp CONTCAR POSCAR
```

- 参数设定
ISTART=0从头算；
不能使用ISMEAR=-5，需要使用SIGMA；
IBRION=2：共轭梯度算法(CG)；  

#### 步骤二：静态自洽

- 作用
静态：原子位置保持不动，不再进行原子迟豫，体系能量达到较低,体系较稳定；
自洽：电子再进行自洽计算，以达到体系的最低能量。

- 主要输出结果  
  
```txt
修改POSCAR文件后，修改INCAR和其他输入文件
```

电子自洽计算完整地计算出体系基态下费米能级(E-fermi)、电子的波函数(WAVECAR)、电荷密度(CHG)等信息，可以直接分析原子间的键合作用，也可以在非自洽之后进一步分析晶体的电子结构和材料的相关性质。

- 参数设定
ISTART=0从头算；
NELM 电子优化(SCF)最大计算步数(默认60)  

#### 步骤三：非自洽计算

- 作用
在自洽基础上改变k点（重新生成k点）等参数，根据不同需要选取能量或势函数或电子密度作为初始值，进行非自洽迭代计算。

- 主要输出结果
可用于求解DOS，能带（电子结构分析）或者光学等其他性质。

- 参数设定
ISTART=1接着计算；
ICHARGE=11(1从CHGCAR读入电荷密度+10电荷密度计算过程保持不变)；
