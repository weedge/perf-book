# 主要 CPU 微架构列表 {.unnumbered}

\markright{主要 CPU 微架构列表}

[待办事项]: 在下面的表格中添加边框线

在下面的表格中，我们展示了来自 Intel、AMD 和基于 ARM 的供应商的最新 ISA 和微架构。当然，并不是所有的设计都列在这里。我们只包括了那些在本书中引用的或者代表了平台演变中的重大转变的设计。

---------------------------------------------------------------
    名称         三字母缩写       发布年份        支持的ISA客户端/服务器芯片
------------  ---------------  ---------------  ---------------
   Nehalem         NHM             2008             SSE4.2

Sandy Bridge       SNB             2011              AVX

   Haswell         HSW             2013              AVX2

   Skylake         SKL             2015         AVX2 / AVX512

 Sunny Cove        SNC             2019             AVX512

 Golden Cove       GLC             2021         AVX2 / AVX512 

Redwood Cove       RWC             2023         AVX2 / AVX512 

---------------------------------------------------------------

Table: 最近的英特尔 Core 微架构列表。 {#tbl:IntelUarchs}

----------------------------------------------
    名称         发布年份       支持的 ISA
------------  ---------------  ---------------
Streamroller      2014              AVX

  Excavator       2015              AVX2

   Zen            2017              AVX2

   Zen2           2019              AVX2

   Zen3           2020              AVX2

   Zen4           2022             AVX512

----------------------------------------------

Table: 最近的 AMD 微架构列表。 {#tbl:AMDUarchs}

------------------------------------------------------------------
    ISA        ISA 发布年份     ARM微架构(最新)     第三方微架构
                                           
------------  ---------------  --------------   ------------------
  ARMv8-A          2011          Cortex-A73        Apple A7-A10;
                                                  Quallcomm Kryo;
                                                 Samsung M1/M2/M3

 ARMv8.2-A         2016         Neoverse N1;         Apple A11;
                                 Cortex-X1           Samsung M4;
                                                    Ampere Altra

 ARMv8.4-A         2017         Neoverse V1        AWS Graviton3;
                                                   Apple A13, M1

 ARMv9.0-A         2018         Neoverse N2;    Microsoft Cobalt 100;
(64位)                           Neoverse V2;        NVIDIA Grace
                                 Cortex X3

 ARMv8.6-A         2019             ---          Apple A15, A16, M2, M3
(64位)

 ARMv9.2-A         2020          Cortex X4              ---

------------------------------------------------------------------

Table: 最近的 ARM ISA 列表，以及它们自己和第三方的实现。 {#tbl:ARMUarchs}

\bibliography{biblio}