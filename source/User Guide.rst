4.  快速入门
************************************

本章将介绍BDF的各种功能的基本使用，针对具体的计算功能给出基本的算例和数据读取分析说明。

4.1  Hartree-Fock
================================================

分子体系的哈密顿算符包含多电子的相互作用，很难求解。HF方法将多电子问题转化为单电子问题求解。HF方法定义了Fock算符，它体现出每个电子在其他电子产生的平均势场中运动，这称为单电子近似。

Hartree-Fock是量子化学最基本算法，如同计算机编程第一次成功输出“Hello world！”，完成一个完整的Hartree-Fock单点能量计算，
搞清计算输出内容，对学习和理解量子化学，掌握一个量子化学软件的基本使用方法大有裨益。本小节，我们将通过一系列的Hartree-Fock
计算的例子，引导用户使用BDFpro，帮助用户了解量子化学的一些基本知识。本节将采用BDFpro的简洁输入模式做为例子，为了使用户理解
BDFpro的简洁输入与高级输入模式的区别，我们也会给出每个简洁输入对应的高级输入文件，并给出分析。

4.1.1  水分子单点能量的Hartree-Fock计算输入文件
-------------------------------------------------------
首先准备水分子单点能量Hartree-Fock计算的输入文件，命名为 ``h2o.inp``, 输入内容如下：

.. code-block:: python

  #!bdf.sh
  HF/3-21G    

  geometry
  O
  H  1  R1 
  H  1  R1  2 109.

  R1=1.0 
  end geometry

输入的第一行必须以 ``#!`` 开始，跟着一个以 ``.sh`` 结尾的字符串，这一行是系统保留行，必须以这个格式输入，字符串仅允许字母。用户可以保持第一行为这个形式不变。
第二行 ``HF/3-21G`` 是BDFpro的计算参数控制行， ``HF`` 是Hartree-Fock的缩写， ``3-21G`` 指定计算使用 ``3-21G`` 基组。
第三行为空行。
第四行与第十行分别为 ``geometry`` 和 ``end geometry`` ，标记分子几何结构输入的起始与中止。
第五行到第九行用内坐标的模式输入了水分子的结构。

这个简单的输入对应的BDFpro高级输入为：

.. code-block:: python

  $compass
  geometry
  o
  h 1 1.0
  h 1 1.0 2 109.
  end geometry
  skeleton
  basis
  3-21g
  $end

  $xuanyuan
  direct
  maxmem
  512mw
  $end

  $scf
  charge
  0
  spin
  1
  rhf
  $end

从高级输入可以看出，BDFpro将按顺序执行模块 ``compass`` ， ``xuanyuan`` 和 ``scf`` 完成水分子的单点能量计算。
``compass`` 用于读入分子结构，基函数等基本信息，判断分子的对称性，将分子转动到标准取向(Standard orientation，详见BDFpro对群论的使用小节)，产生对称匹配轨道等，
并将这些信息存入BDFpro的执行目录下的文件 ``h2o.chkfil`` 。 ``compass`` 中的关键词
 * ``geommetry`` 到 ``end geometry`` 之间定义的分子结构;
 * ``basis`` 定义基组为 ``3-21G``;
 * ``Skeleton`` 指定只计算对称独立的单、双电子积分，构造骨架Fock矩阵并对称化(详见BDFpro对群论的使用小节)。 

执行完 ``compass`` 模块后，BDFpro利用 ``xuanyuan`` 模块计算单、双电子积分。
 * ``direct`` 关键词指定后续的自洽场计算采用积分直接的计算方法(详见BDFpro的积分计算方法小节);
 * ``maxmem`` 指定积分计算是可用的缓冲区内存为512 Mega Words。

最后，BDFpro执行 ``scf`` 模块，完成基于Hartree-Fock的自洽场计算。
 * ``rhf`` 指定使用限制性Hartree-Fock方法;
 * ``charge`` 指定体系的电荷为0;
 * ``spin`` 指定体系的自旋多重度为1。
这里 ``rhf`` 是必须输入的关键词， ``charge`` 和 ``spin`` 可以忽略。

4.1.2  执行计算
-------------------------------------------------------
执行计算，需要准备一个Shell脚本，命名为 ``run.sh`` ,放入 输入文件 ``h2o.inp`` 所在的目录。内容如下：

.. code-block:: python

    #!/bin/bash

    export BDFHOME=/home/bsuo/bdf-pkg-pro
    export BDF_TMPDIR=/tmp/$RANDOM

    ulimit -s unlimitted
    ulimit -t unlimitted

    export OMP_NUM_THREADS=4
    export OMP_STACKSIZE=1024M

    $BDFHOME/sbin/bdfdrv.py -r h2o.inp 

这里，我们准备了一个 ``Bash Shell`` 脚本，定义了一些基本的环境变量，并利用 ``$BDFHOME/sbin/bdfdrv.py`` 执行计算。这里

 * ``BDFHOME`` 变量指定BDFpro的安装目录；
 * ``BDF_TMPDIR`` 变量指定BDFpro运行时临时文件存放目录；
 * ``ulimit -s unlimitted`` 设定程序可用的Stack区内存不受限；
 * ``ulimit -t unlimitted`` 设定程序执行时间不受限；
 * ``export OMP_NUM_THREADS=4`` 设定可用4个OpenMP线程执行并行计算；
 * ``export OMP_STACKSIZE=1024`` 设定OpenMP可用的Stack区内存为1024兆字节。

执行计算的命令为

.. code-block:: python

    $ ./run.sh h2o.inp &>h2o.out&

由于BDFpro将默认输出打印到标准输出，这里我们用了Linux的重定向命令，将标准输出定向到文件 ``h2o.out`` 。

4.1.3  计算结果分析
-------------------------------------------------------
计算结束后，将得到 ``h2o.out`` , ``h2o.chkfil`` , ``h2o.scforb`` 等文件。
 
 * ``h2o.out`` 是文本文件，用户可读，存储BDFpro输出打印信息；
 * ``h2o.chkfil`` 是二进制文件，用户不可读，用户在BDFpro不同模块传递信息；
 * ``h2o.scforb`` 是文本文件，用户可读，存储了 ``SCF`` 自洽迭代的分子轨道，轨道能等信息。

如果输入文件采用的是BDFpro简洁输入模式， ``h2o.out`` 中会给出一些基本的用户设置信息,

.. code-block:: python

    |=========================================== BDF Control parameters ================================================|
    
    
     1: Input BDF Keywords
       xcfun=None    skeleton=True    scf=rhf    direct=True    
       charge=0    spin=1    
    
     3: Basis sets
        ['3-21g']
    
     4: Wavefunction, Charges and spin
       charge=0    nuclearcharge=10    spin=1    
    
     5: Energy method
        scf
    
     6: Acceleration method
        ERI
    
     7: Potential energy sufface method
        energy
    
    |====================================================================================================================|

这里，

 * ``Input BDF Keywords`` 给出了一些基本控制参数； 
 * ``Basis set`` 给出计算所用基组；
 * ``Wavefunction, Charges and spin`` 给出了体系电荷、总的核电荷数和自旋多重度(2S+1)；
 * ``Energy method`` 给出能量计算方法；
 * ``Accleration method`` 给出双电子积分计算加速方法；
 * ``Potential energy sufface method`` 给出势能面计算方法，这里是单点能量计算。

随后，系统执行 ``compass`` 模块，会给出如下提示：

.. code-block:: python

    |******************************************************************************|
    
        Start running module compass
        Current time   2021-11-18  11:26:28

    |******************************************************************************|


然后以笛卡尔坐标的形式打印输入的分子结构及每种类型原子的基函数

.. code-block:: python

    |-------------------------------------------------------------------------------------------|
    
     Atom           Cartcoord(Bohr)                 Charge Basis Auxbas Uatom Nstab Alink  Mass
      O        0.000000     0.000000     0.000000     8.00    1     0     0     0   E     15.9949
      H        1.889726     0.000000     0.000000     1.00    2     0     0     0   E      1.0073
      H       -0.615235     1.786771     0.000000     1.00    2     0     0     0   E      1.0073
    
    |--------------------------------------------------------------------------------------------|
    
      End of reading atomic basis sets ..
     Printing basis sets for checking ....
    
     Atomic label:  O   8
     Maximum L  1 6s3p ----> 3s2p NBF =   9
     #--->s function
          Exp Coef          Norm Coef       Con Coef
               322.037000   0.192063E+03    0.059239    0.000000    0.000000
                48.430800   0.463827E+02    0.351500    0.000000    0.000000
                10.420600   0.146533E+02    0.707658    0.000000    0.000000
                 7.402940   0.113388E+02    0.000000   -0.404454    0.000000
                 1.576200   0.355405E+01    0.000000    1.221562    0.000000
                 0.373684   0.120752E+01    0.000000    0.000000    1.000000
     #--->p function
          Exp Coef          Norm Coef       Con Coef
                 7.402940   0.356238E+02    0.244586    0.000000
                 1.576200   0.515227E+01    0.853955    0.000000
                 0.373684   0.852344E+00    0.000000    1.000000
    
    
     Atomic label:  H   1
     Maximum L  0 3s ----> 2s NBF =   2
     #--->s function
          Exp Coef          Norm Coef       Con Coef
                 5.447178   0.900832E+01    0.156285    0.000000
                 0.824547   0.218613E+01    0.904691    0.000000
                 0.183192   0.707447E+00    0.000000    1.000000

然后，自动判断分子对称性，并根据用户设置决定是否转动为标准取向模式，

.. code-block:: python

    Auto decide molecular point group! Rotate coordinates into standard orientation!
    Threshold= 0.10000E-08 0.10000E-11 0.10000E-03
    geomsort being called!
    gsym: C02V, noper=    4
    Exiting zgeomsort....
    epresentation generated
    Binary group is observed ...
    Point group name C(2V)                       4
    User set point group as C(2V)   
     Largest Abelian Subgroup C(2V)                       4
     Representation generated
     C|2|V|                    2

    Symmetry check OK
    Molecule has been symmetrized
    Number of symmery unique centers:                     2
    
    |-------------------------------------------------------------------------------------------|
    
     Atom           Cartcoord(Bohr)                 Charge Basis Auxbas Uatom Nstab Alink  Mass
      O        0.000000    -0.000000     0.219474     8.00    1     0     0     0   E     15.9949
      H       -1.538455     0.000000    -0.877896     1.00    2     0     0     0   E      1.0073
      H        1.538455    -0.000000    -0.877896     1.00    2     0     0     0   E      1.0073
    
    |--------------------------------------------------------------------------------------------|

细心的用户可能已经注意到，这里的水分子的坐标与输入的不一样。最后， ``compass`` 会产生对称匹配轨道（Symmetry adapted orbital），并给出偶极矩和四极矩所属
的不可约表示，打印 ``C2v`` 点群的乘法表，给出总的基函数数目和每个不可约表示对称匹配轨道数目。由于BDFpro深度使用了群论，感兴趣的用户可以通过BDFpro的输出对照学习群论知识。

.. code-block:: python

    Number of irreps:    4
    IRREP:   3   4   1
    DIMEN:   1   1   1
    
     Irreps of multipole moment operators ...
     Operator  Component    Irrep       Row
      Dipole       x           B1          1
      Dipole       y           B2          1
      Dipole       z           A1          1
      Quadpole     xx          A1          1
      Quadpole     xy          A2          1
      Quadpole     yy          A1          1
      Quadpole     xz          B1          1
      Quadpole     yz          B2          1
      Quadpole     zz          A1          1
    
     Generate symmetry adapted orbital ...
     Print Multab
      1  2  3  4
      2  1  4  3
      3  4  1  2
      4  3  2  1
    
    |--------------------------------------------------|
              Symmetry adapted orbital                   
    
      Total number of basis functions:      13      13
    
      Number of irreps:   4
      Irrep :   A1        A2        B1        B2      
      Norb  :      7         0         4         2
    |--------------------------------------------------|

这里， ``C2v`` 点群有4个一维不可约表示，标记为 ``A1, A2, B2, B2`` , 分别有 ``7, 0, 4, 2`` 个对称匹配的轨道。

.. note::

    Tips：不同的量子化学软件，可能会采用不同的分子标准取向，导致不可约表示出现的顺序不同。

最后， ``compass`` 计算正常结束，会给出如下输出：

.. code-block:: python

    |******************************************************************************|

        Total cpu     time:          0.00  S
        Total system  time:          0.00  S
        Total wall    time:          0.02  S
    
        Current time   2021-11-18  11:26:28
        End running module compass
    |******************************************************************************|


.. note::

    Tips：BDFpro的每个模块执行，都会有开始执行和之行结束的时间统计，也方便了用户具体定位哪个计算模块出错。


一般的，单点能量计算执行的第二个模块是 ``xuanyuan`` ，计算单、双电子积分。BDFpro简洁输入模式默认采用积分直接算法，
只计算和保存单电子积分及需要做Schwartz积分与筛选的特殊双电子积分。如果用户指定了 ``nodirect`` 关键词，双电子积分
将被计算并保存到硬盘。 ``xuanyuan`` 模块的输出比较简单，一般不需要特别关注。这里，我们给出最关键的输出：

.. code-block:: python

    [aoint_1e]
      Calculating one electron integrals ...
      S T and V integrals ....
      Dipole and Quadupole integrals ....
      Finish calculating one electron integrals ...
    
     ---------------------------------------------------------------
      Timing to calculate 1-electronic integrals                                      
    
      CPU TIME(S)      SYSTEM TIME(S)     WALL TIME(S)
              0.017            0.000               0.000
     ---------------------------------------------------------------
    
     Finish calculating 1e integral ...
     Direct SCF required. Skip 2e integral!
     Set significant shell pairs!
    
     Number of significant pairs:        7
     Timing caluclate K2 integrals.
     CPU:       0.00 SYS:       0.00 WALL:       0.00
    
从输出我们看到单电子重叠、动能与核吸引积分被计算，还计算了偶极矩和四极矩积分。由于输入要求积分直接的SCF计算(Direct SCF)，双电子积分计算被忽略。

最后，BDFpro调用 ``scf`` 模块执行 ``RHF`` 自洽场计算。需要关注的信息有：

.. code-block:: python

     Wave function information ...
     2*Na,2*Nb =                    10                   10
     Total Nuclear charge    :      10
     Total electrons         :      10
     ECP-core electrons      :       0
     Spin (2S+1)             :       1
     Num. of alpha electrons :       5
     Num. of beta  electrons :       5

这里给出了电荷、自旋多重度，核电荷数及电子数等信息，用户应当检查电子态是否正确。
然后，首先进行原子计算，并产生分子计算的初始猜测密度矩阵，

.. code-block:: python

     [ATOM SCF control]
      heff=                     0
     After initial atom grid ...
     Finish atom    1  O             -73.8654283850
     After initial atom grid ...
     Finish atom    2  H              -0.4961986360
    
     Superposition of atomic densities as initial guess.

并检查处理基函数可能的线性相关问题，

.. code-block:: python

     Check basis set linear dependence! Tolerance =   0.100000E-04

然后进入SCF迭代，8次迭代收敛后关闭DIIS和Level shift等加速收敛方法并重新计算能量，

.. code-block:: python

    Iter.   idiis  vshift       SCF Energy            DeltaE          RMSDeltaD          MaxDeltaD      Damping    Times(S) 
       1      0    0.000     -75.4652250437      -0.6073993867       0.0394104979       0.2382197472    0.0000      0.00
       2      1    0.000     -75.5358877159      -0.0706626722       0.0138968193       0.0808310470    0.0000      0.00
       3      2    0.000     -75.5741871530      -0.0382994371       0.0044235916       0.0290160747    0.0000      0.00
       4      3    0.000     -75.5835808854      -0.0093937324       0.0009616649       0.0037827401    0.0000      0.00
       5      4    0.000     -75.5838268981      -0.0002460127       0.0001465257       0.0008712033    0.0000      0.00
       6      5    0.000     -75.5838316668      -0.0000047687       0.0000123001       0.0000735848    0.0000      0.00
       7      6    0.000     -75.5838316945      -0.0000000277       0.0000012422       0.0000074870    0.0000      0.00
       8      7    0.000     -75.5838316948      -0.0000000003       0.0000004656       0.0000025498    0.0000      0.00
     diis/vshift is closed at iter =   8
       9      0    0.000     -75.5838316948      -0.0000000000       0.0000000463       0.0000002212    0.0000      0.00
    
      Label              CPU Time        SYS Time        Wall Time
     SCF iteration time:         0.017 S        0.017 S        0.000 S

最后打印不同项的能量贡献和维里比，

.. code-block:: python

     Final scf result
       E_tot =               -75.58383169
       E_ele =               -84.37566837
       E_nn  =                 8.79183668
       E_1e  =              -121.94337426
       E_ne  =              -197.24569473
       E_kin =                75.30232047
       E_ee  =                37.56770589
       E_xc  =                 0.00000000
      Virial Theorem      2.003738

这里，

 * ``E_tot`` 是系统总能量;
 * ``E_ele`` 是电子能量;
 * ``E_nn``  是原子核排斥能;
 * ``E_1e``  是单电子能量;
 * ``E_ne``  是原子核对电子的吸引能;
 * ``E_kin``  是电子动能;
 * ``E_ee`` 是双电子能，包括库伦排斥和交换能；
 * ``E_xc`` 是交换相关能，DFT计算时不为0.

能量打印后输出的是轨道的占据情况，轨道能，HUMO-LOMO能量和gap信息。

.. code-block:: python

     [Final occupation pattern: ]
    
     Irreps:        A1      A2      B1      B2  
    
     detailed occupation for iden/irep:      1   1
        1.00 1.00 1.00 0.00 0.00 0.00 0.00
     detailed occupation for iden/irep:      1   3
        1.00 0.00 0.00 0.00
     detailed occupation for iden/irep:      1   4
        1.00 0.00
     Alpha       3.00    0.00    1.00    1.00
    
    
     [Orbital energies:]
    
     Energy of occ-orbs:    A1            3
                 -20.43281195      -1.30394125      -0.52260024
     Energy of vir-orbs:    A1            4
                   0.24980046       1.23122290       1.86913815       3.08082943
    
     Energy of occ-orbs:    B1            1
                  -0.66958992
     Energy of vir-orbs:    B1            3
                   0.34934415       1.19716413       2.03295437
    
     Energy of occ-orbs:    B2            1
                  -0.47503768
     Energy of vir-orbs:    B2            1
                   1.78424252
    
     Alpha   HOMO energy:      -0.47503768 au     -12.92643838 eV  Irrep: B2      
     Alpha   LUMO energy:       0.24980046 au       6.79741929 eV  Irrep: A1      
     HOMO-LUMO gap:       0.72483814 au      19.72385767 eV

这里

 * ``[Final occupation pattern: ]`` 给出的是轨道占据情况。由于我们进行的是限制性Hartree-Fock计算，占据情况只给出了Alpha轨道的信息，按照不可约表示分别给出。从这个例子可以看出，A1轨道的前3个， B1和B2轨道的第1个分别有1个电子占据。由于本算例是RHF，alpha与beta轨道是一样的，所以A1表示有3个双占据轨道，B1和B2表示分别有1个占据轨道。
 * ``[Orbital energies:]`` 按照不可约表示分别给出轨道能；
 * ``Alpha   HOMO energy:`` 给出了HOMO轨道能量，单位为au及eV，属于B2表示；
 * ``Alpha   LUMO energy:`` 给出了LUMO轨道能量，单位为au及eV，属于B2表示；
 * ``HOMO-LUMO gap:`` 给出HOMO和LUMO轨道的能差。

为了减少输出行数，BDFpro默认不打印轨道成分及分子轨道系数，只按照不可约表示分类给出部分轨道占据数和轨道能信息，如下：

.. code-block:: python

      Symmetry   1 A1      
    
             Orbital                 1              2              3              4              5              6
             Energy            -20.43281       -1.30394       -0.52260        0.24980        1.23122        1.86914
             Occ No.             2.00000        2.00000        2.00000        0.00000        0.00000        0.00000
    
    
      Symmetry   2 A2      
    
    
      Symmetry   3 B1      
    
             Orbital                 8              9             10             11
             Energy             -0.66959        0.34934        1.19716        2.03295
             Occ No.             2.00000        0.00000        0.00000        0.00000
    
    
      Symmetry   4 B2      
    
             Orbital                12             13
             Energy             -0.47504        1.78424
             Occ No.             2.00000        0.00000
             
``scf`` 模块最后打印的是Mulliken和Lowdin布居分析的结果，分子的偶极矩信息。

.. code-block:: python

     [Mulliken Population Analysis]
      Atomic charges: 
         1O      -0.7232
         2H       0.3616
         3H       0.3616
         Sum:    -0.0000
    
     [Lowdin Population Analysis]
      Atomic charges: 
         1O      -0.4756
         2H       0.2378
         3H       0.2378
         Sum:    -0.0000
    
    
     [Dipole moment: Debye]
               X          Y          Z     
       Elec:-.1081E-64 0.4718E-32 -.2368E+01
       Nucl:0.0000E+00 0.0000E+00 0.5644E-15
       Totl:   -0.0000     0.0000    -2.3684
       


4.2  Kohn-Sham
================================================



4.3  溶剂化模型
================================================



4.4  高斯基组
================================================

为了求解HF、KS-DFT方程，需要把分子轨道展开为单电子基函数的线性组合：

.. math::
    \varphi_{i}(r) = C_{1,i}\chi_{1}(r) + C_{2,i}\chi_{2}(r) + C_{3,i}\chi_{3}(r) + \dots + C_{N,i}\chi_{N}(r)

在量子化学的计算中，基函数只有数学意义，没有物理意义。基函数越多则结果越精确，但是也取决于怎么合理地设置基函数。当基函数无穷多，称为完备集，就达到了完备基组极限（Complete Basis set limit, CBS），能够完美展开分子轨道。实际用的基组尺寸是有限的而达不到CBS，由此导致的计算结果的误差称为基组不完备性误差。

用多少基函数，就会产生多少分子轨道，但是只有占据轨道，以及低阶的非占据轨道（价层空轨道）通常有化学意义。如果基函数取的就是原子轨道，称为原子轨道线性组合（linear combination of atomic orbitals，LCAO），但是这只是结构化学上纯概念的东西，实际计算中使用的基函数并不是真实的原子轨道。

量子化学中常用的基函数如下：

#. 高斯型函数（Gauss type function, GTF）：因其在数序形式上易于计算双电子积分，绝大多数量子化学程序使用的都是高斯型基函数。
#. Slater轨道（Slater type orbital, STO）：半经验以及少数量子化学程序（如ADF）所用的基函数。难以计算双电子积分，但相对于高斯型基组它的径向行为更接近于实际原子轨道，因此只需要较少数目的STO就可以达到较多数目GTF的计算结果。
#. 平面波（Plane wave）：专门适用于周期性计算的基函数，计算孤立体系时比高斯型基组性价比低得多。
#. 数值基组（Numerical basis set）：极少程序支持，典型的是Dmol3、Siesta。基函数并没有解析的数学形式，而是通过离散分布的点描述。

BDF软件采用的是高斯型基函数。



4.5  势能面和几何优化
================================================

几何优化的目的是找到体系势能面的极小点。能找到哪个极小点取决于输入文件中提供的初猜结构，离哪个极小点越近，一般越容易收敛到哪个极小点。

几何优化在数学上等价于寻找多元函数极值问题：

.. math::
    F_{i} = -\frac{\partial E(R_1,R_2,\dots,R_N)}{\partial R_i} = 0, i=1,2,\dots,N

几何优化常用算法如下：

#. 最速下降法（Steepest descent）：最速下降法就是沿着负梯度的方向进行线搜索，对于远离极小点的结构，最速下降法优化效率非常高，但临近极小点时收敛慢，容易震荡。
#. 共轭梯度法（Conjugate gradient）：共轭梯度法是最速下降法的改良，每步优化方向与前一步的优化方向相组合,能一定程度缓解震荡问题。
#. 牛顿法（Newton method）：牛顿法的思路是将函数相对于当前位置进行泰勒级数展开。牛顿法收敛很快，对于二次函数一步就可以走到极小点。但是牛顿法需要求解Hessian矩阵，计算非常太昂贵，一般几何优化中使用准牛顿法。
#. 准牛顿法（Quasi-Newton method）：准牛顿法通过近似方法构建 Hessian矩阵，当前步的Hessian矩阵基于当前步的受力和上一步的Hessian矩阵来得到。具体做法有多种，最常用的是BFGS法，还有DFP、MS、PSB等。由于准牛顿法的 Hessian是近似构建的，所以每一步优化的准确度低于牛顿法，达到收敛所需步数更多。但由于每一步耗时大为降低，所以优化总耗时还是显著减少了。


4.5.1.  基态结构优化
-------------------------------------------------------

4.5.2.  激发态结构优化
-------------------------------------------------------

4.5.3.  限制性结构优化
-------------------------------------------------------

4.5.4.  冗余坐标优化
-------------------------------------------------------

4.5.5.  几何优化常见问题
-------------------------------------------------------

4.5.5.1. 虚频问题
########################################################

4.5.5.2. 对称性问题
########################################################

4.5.5.3. 几何优化不收敛
########################################################

4.6  激发态计算
================================================





4.7  光谱
================================================

4.7.1 红外
-------------------------------------------------------

4.7.2 拉曼
-------------------------------------------------------

4.7.3 吸收
-------------------------------------------------------

4.7.4 发射
-------------------------------------------------------

4.7.5 旋轨耦合校正
-------------------------------------------------------

4.7.6 NMR
-------------------------------------------------------


4.8  数值Hessian
================================================



4.9  相对论效应
================================================



4.10  QM/MM
================================================



4.11  加速算法
================================================



4.12  波函数分析&单电子性质
================================================


