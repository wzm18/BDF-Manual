5.  计算模块功能及输入
************************************
.. _bdf-route:

.. image:: images/modules.png
   :width: 600
   :align: center

5.1.  compass模块
================================================
Compass模块主要完成计算任务的初始化工作，包括读入用户定义的分子结构、基组等基本信息，判断分子的对称性及其分子点群，产生对称匹配的轨道等，并将其转换为BDF内部的数据存储起来。Compass模块的主要参数有：


5.2.  xuanyuan模块
================================================
Xuanyuan模块主要计算单、双电子积分和其他必要的积分并存储到文件中。


5.3.  scf模块
================================================
SCF模块是BDF的核心计算模块之一，进行Hartree-Fock和DFT计算。


5.4.  tddft模块
================================================
TDDFT模块基于线形响应理论，通过求解Casida方程计算分子激发态。TDDFT支持完全的TDHF、TDDFT，TDA、SA-TDDFT和X-TDDFT等方法。可以处理基态是闭壳层或者是开壳层的分子体系。其中SA-TDDFT与X-TDDFT是BDF的特色，对基态是开壳层大的分子体系，可以部分解决自旋污染问题。


5.5.  bdfopt模块
================================================
bdfopt模块是BDF程序的分子几何结构优化模块，可用来寻找极小点、过渡态、锥形交叉点等。与其他模块不同，包含bdfopt模块的输入文件，并不是按照模块的先后顺序线性执行的。


5.6.  traint模块
================================================
bdfopt模块是BDF程序的分子几何结构优化模块，可用来寻找极小点、过渡态、锥形交叉点等。与其他模块不同，包含bdfopt模块的输入文件，并不是按照模块的先后顺序线性执行的。


5.7.  MP2模块
================================================
MP2能量计算模块。


5.8.  mcscf模块
================================================
多组态自洽场计算模块，如果没有定义活性空间，则进行二阶收敛的RHF计算。如果不优化分子轨道，仅进行CASCI计算。


5.9.  drt模块
================================================
Drt模块与mrci模块联用，利用基于空穴-粒子对称的图形酉群(Hole-particle symmetry based graphical unitrary group approach - HP-GUGA)方法计算非收缩的多参考态单双激发组态相互作用(Multireference configuration interaction with single and double excitations - MRCISD)。drt模块用于产生基于HP-GUGA方法的活性空间不同行表(Distinct Row Tabular - DRT )。


5.10.  mrci模块
================================================
MRCI模块与DRT模块联用，执行非收缩的MRCI计算。


5.11.  xianci模块
================================================
xianci模块来自Xi‘anCI程序包，执行内收缩MRCI计算，MRPT2，NEVPT2，SDS-PT2等计算。xianci模块也可以执行非收缩MRCI计算，相当于将DRT和MRCI模块合并在一起。


5.12.  localmo模块
================================================
localmo模快用于产生定域化的分子轨道，包含了Boys，Pipek-Maye，改进的Boys定域化等方法。localmo还用于为FLMO和Local MCSCF方法产生初始的分子片定域轨道。


5.13.  grad模块
================================================
grad模块用于计算HF/MCSCF的解析梯度。


5.14.  resp模块
================================================
resp模块用于计算DFT/TDDFT的梯度，TDDFT的基态-激发态，激发态-激发态之间的非绝热耦合。


5.15.  expandmo模块
================================================
expandmo模块用于将小基组计算的MO扩展为大基组MO，扩展的MO可用于SCF的初始猜测，也可用于一些双基组(Dual Basis)的计算。此外，expandmo还可以根据基于利用原子价活性空间（atomic valance active space），自动构建MCSCF计算的活性空间和初始猜测轨道。

5.15.  elecoup模块
================================================
Elecoup主要功能有：
#. 基于HF计算同一分子的两个电子态之间的耦合积分； 
#. 计算两个分子片之间的电荷迁移积分； 
#. 计算两个分子片激发态间的能量转移积分。

