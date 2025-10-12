![image-20250609173633691](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609173633691.png)

这是 SQL Server 监控工具（大概率是 SQL Server Management Studio 或类似数据库性能监控平台）展示的 **MESDEVSRV** 服务器在 **2025 年 6 月 9 日 16:29–17:29** 期间的性能指标概览，核心从 CPU、内存、吞吐量、等待（Waits）4 个维度呈现数据库及主机（Machine）的资源使用情况，可辅助排查性能瓶颈，以下分模块解读：

### 1. 顶部概览

- **实例与时间**：聚焦 `MESDEVSRV` 实例，统计最近 1 小时（Last 1 hour）、截至 17:29 的指标，可通过时间控件调整范围。
- **指标开关**：勾选 `CPU、Memory、Throughput、Waits` 控制是否展示对应维度曲线，方便按需分析。

### 2. 核心指标曲线（CPU 维度示例）

- **绿色（SQL Server CPU）**：SQL Server 进程的 CPU 使用率波动曲线，反映数据库工作负载（如查询、事务）对 CPU 的消耗。若持续高位，可能是复杂查询、索引缺失等问题。
- **橙色（Machine CPU）**：服务器物理机 / 虚拟机的整体 CPU 使用率，体现系统级资源压力。若橙色远高于绿色，需排查操作系统层其他进程占用。
- **蓝色（基准线 / Baseline，可选）**：可对比历史正常负载，快速识别当前是否异常（如突增、持续超限）。

### 3. 细分指标面板

#### （1）CPU 面板

- **Machine（浅蓝）**：主机 CPU 平均使用率，反映服务器整体算力消耗。
- **SQL Server（深蓝）**：数据库进程专属 CPU 占比，定位数据库是否为 CPU 瓶颈源。若 SQL Server 占比长期接近主机 CPU，说明数据库负载主导系统压力。

#### （2）Memory 面板

- **Machine（浅紫）**：主机内存总占用，监控物理内存是否充足（若接近上限，可能触发分页，拖慢性能）。
- **SQL Server（深紫）**：数据库服务占用的内存，需结合 `max server memory` 配置，判断是否因内存分配不合理（如 SQL 内存过大挤压系统进程，或过小导致缓存不足）影响性能。

#### （3）Throughput 面板

- **Machine（浅绿）**：主机层面的 IO 吞吐量（MB/s），含磁盘读写、网络传输等，体现系统数据传输压力。
- **SQL Server（深绿）**：数据库进程的 IO 吞吐量，聚焦数据库文件（如 MDF、LDF）的读写。若持续高位，可能是磁盘性能不足、查询扫描大量数据（如缺失索引导致全表扫描）。

#### （4）Waits 面板

- **Resource（橙色）**：资源等待（如磁盘 IO 等待、锁等待），反映数据库因争抢资源（磁盘慢、并发冲突）被迫等待的频率。持续高值需排查：是否磁盘 IO 瓶颈（结合 Throughput）、是否存在锁竞争（如事务未及时提交）。
- **Signal（棕色）**：信号等待，指线程因 CPU 繁忙无法及时调度的等待。若长期高，说明 CPU 资源紧张，需优化 CPU 密集型查询或扩容硬件。

### 4. 典型分析场景

- **CPU 居高不下**：对比绿色（SQL CPU）和橙色（主机 CPU），若 SQL CPU 主导，检查是否有高消耗查询（如 `sys.dm_exec_query_stats` 找 CPU Top 语句）、是否缺失索引；若主机 CPU 高，排查操作系统其他进程。
- **IO 瓶颈嫌疑**：Throughput 持续峰值 + Waits 中 Resource 等待高，优先检查磁盘性能（如存储延迟）、优化查询减少不必要的 IO（如索引覆盖查询、避免全表扫描）。
- **内存预警**：若主机内存接近上限，SQL Server 内存却未合理利用（如 `max server memory` 设得过低），需调整内存配置；若 SQL 内存过高导致系统不稳定，适当降低上限。



简言之，这张图是数据库性能诊断的 “仪表盘”，通过多维度资源监控，快速定位 SQL Server 与主机的资源瓶颈，指导优化方向（如查询调优、硬件扩容、配置调整）。结合具体业务负载（如高峰时段、批量作业时间）分析曲线波动，能更精准找到问题根源。



![image-20250609100257859](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609100257859.png)

```
INSERT INTO [rt_PrdSNMtlTrace_Detail]([mtlTraceInputID],[mtlRegisterID],[inDTime],[Location],[MasterPrdID],[oprID],[planID],[prcoessTraceID],[processSiteNo],[Qty],[RegisterType],[replaceLocation],[replaceMtlTraceInputID],[replaceQty],[RwkOpTraceDetailID]) values(@1,@2,@3,@4,@5,@6,@7,NULL,@8,@9,@10,@11,NULL,@12,NULL),(@13,@14,@15,@16,@17,@18,@19,NULL,@20,@21,@22,@23,NULL,@24,NULL),(@25,@26,@27,@28,@29,@30,@31,NULL,@32,@33,@34,@35,NULL,@36,NULL),(@37,@38,@39,@40,@41,@42,@43,NULL,@44,@45,@46,@47,NULL,@48,NULL),(@49,@50,@51,@52,@53,@54,@55,NULL,@56,@57,@58,@59,NULL,@60,NULL),(@61,@62,@63,@64,@65,@66,@67,NULL,@68,@69,@70,@71,NULL,@72,NULL),(@73,@74,@75,@76,@77,@78,@79,NULL,@80,@81,@82,@83,NULL,@84,NULL),(@85,@86,@87,@88,@89,@90,@91,NULL,@92,@93,@94,@95,NULL,@96,NULL),(@97,@98,@99,@100,@101,@102,@103,NULL,@104,@105,@106,@107,NULL,@108,NULL),(@109,@110,@111,@112,@113,@114,@115,NULL,@116,@117,@118,@119,NULL,@120,NULL),(@121,@122,@123,@124,@125,@126,@127,NULL,@128,@129,@130,@131,NULL,@132,NULL),(@133,@134,@135,@136,@137,@138,@139,NULL,@140,@141,@142,@143,NULL,@144,NULL),(@145,@146,@147,@148,@149,@150,@151,NULL,@152,@153,@154,@155,NULL,@156,NULL),(@157,@158,@159,@160,@161,@162,@163,NULL,@164,@165,@166,@167,NULL,@168,NULL),(@169,@170,@171,@172,@173,@174,@175,NULL,@176,@177,@178,@179,NULL,@180,NULL),(@181,@182,@183,@184,@185,@186,@187,NULL,@188,@189,@190,@191,NULL,@192,NULL),(@193,@194,@195,@196,@197,@198,@199,NULL,@200,@201,@202,@203,NULL,@204,NULL),(@205,@206,@207,@208,@209,@210,@211,NULL,@212,@213,@214,@215,NULL,@216,NULL),(@217,@218,@219,@220,@221,@222,@223,NULL,@224,@225,@226,@227,NULL,@228,NULL),(@229,@230,@231,@232,@233,@234,@235,NULL,@236,@237,@238,@239,NULL,@240,NULL),(@241,@242,@243,@244,@245,@246,@247,NULL,@248,@249,@250,@251,NULL,@252,NULL),(@253,@254,@255,@256,@257,@258,@259,NULL,@260,@261,@262,@263,NULL,@264,NULL),(@265,@266,@267,@268,@269,@270,@271,NULL,@272,@273,@274,@275,NULL,@276,NULL),(@277,@278,@279,@280,@281,@282,@283,NULL,@284,@285,@286,@287,NULL,@288,NULL),(@289,@290,@291,@292,@293,@294,@295,NULL,@296,@297,@298,@299,NULL,@300,NULL),(@301,@302,@303,@304,@305,@306,@307,NULL,@308,@309,@310,@311,NULL,@312,NULL),(@313,@314,@315,@316,@317,@318,@319,NULL,@320,@321,@322,@323,NULL,@324,NULL),(@325,@326,@327,@328,@329,@330,@331,NULL,@332,@333,@334,@335,NULL,@336,NULL),(@337,@338,@339,@340,@341,@342,@343,NULL,@344,@345,@346,@347,NULL,@348,NULL),(@349,@350,@351,@352,@353,@354,@355,NULL,@356,@357,@358,@359,NULL,@360,NULL),(@361,@362,@363,@364,@365,@366,@367,NULL,@368,@369,@370,@371,NULL,@372,NULL),(@373,@374,@375,@376,@377,@378,@379,NULL,@380,@381,@382,@383,NULL,@384,NULL),(@385,@386,@387,@388,@389,@390,@391,NULL,@392,@393,@394,@395,NULL,@396,NULL),(@397,@398,@399,@400,@401,@402,@403,NULL,@404,@405,@406,@407,NULL,@408,NULL),(@409,@410,@411,@412,@413,@414,@415,NULL,@416,@417,@418,@419,NULL,@420,NULL),(@421,@422,@423,@424,@425,@426,@427,NULL,@428,@429,@430,@431,NULL,@432,NULL),(@433,@434,@435,@436,@437,@438,@439,NULL,@440,@441,@442,@443,NULL,@444,NULL),(@445,@446,@447,@448,@449,@450,@451,NULL,@452,@453,@454,@455,NULL,@456,NULL),(@457,@458,@459,@460,@461,@462,@463,NULL,@464,@465,@466,@467,NULL,@468,NULL),(@469,@470,@471,@472,@473,@474,@475,NULL,@476,@477,@478,@479,NULL,@480,NULL),(@481,@482,@483,@484,@485,@486,@487,NULL,@488,@489,@490,@491,NULL,@492,NULL),(@493,@494,@495,@496,@497,@498,@499,NULL,@500,@501,@502,@503,NULL,@504,NULL),(@505,@506,@507,@508,@509,@510,@511,NULL,@512,@513,@514,@515,NULL,@516,NULL),(@517,@518,@519,@520,@521,@522,@523,NULL,@524,@525,@526,@527,NULL,@528,NULL),(@529,@530,@531,@532,@533,@534,@535,NULL,@536,@537,@538,@539,NULL,@540,NULL),(@541,@542,@543,@544,@545,@546,@547,NULL,@548,@549,@550,@551,NULL,@552,NULL),(@553,@554,@555,@556,@557,@558,@559,NULL,@560,@561,@562,@563,NULL,@564,NULL),(@565,@566,@567,@568,@569,@570,@571,NULL,@572,@573,@574,@575,NULL,@576,NULL),(@577,@578,@579,@580,@581,@582,@583,NULL,@584,@585,@586,@587,NULL,@588,NULL),(@589,@590,@591,@592,@593,@594,@595,NULL,@596,@597,@598,@599,NULL,@600,NULL),(@601,@602,@603,@604,@605,@606,@607,NULL,@608,@609,@610,@611,NULL,@612,NULL),(@613,@614,@615,@616,@617,@618,@619,NULL,@620,@621,@622,@623,NULL,@624,NULL),(@625,@626,@627,@628,@629,@630,@631,NULL,@632,@633,@634,@635,NULL,@636,NULL),(@637,@638,@639,@640,@641,@642,@643,NULL,@644,@645,@646,@647,NULL,@648,NULL),(@649,@650,@651,@652,@653,@654,@655,NULL,@656,@657,@658,@659,NULL,@660,NULL),(@661,@662,@663,@664,@665,@666,@667,NULL,@668,@669,@670,@671,NULL,@672,NULL),(@673,@674,@675,@676,@677,@678,@679,NULL,@680,@681,@682,@683,NULL,@684,NULL),(@685,@686,@687,@688,@689,@690,@691,NULL,@692,@693,@694,@695,NULL,@696,NULL),(@697,@698,@699,@700,@701,@702,@703,NULL,@704,@705,@706,@707,NULL,@708,NULL),(@709,@710,@711,@712,@713,@714,@715,NULL,@716,@717,@718,@719,NULL,@720,NULL),(@721,@722,@723,@724,@725,@726,@727,NULL,@728,@729,@730,@731,NULL,@732,NULL),(@733,@734,@735,@736,@737,@738,@739,NULL,@740,@741,@742,@743,NULL,@744,NULL),(@745,@746,@747,@748,@749,@750,@751,NULL,@752,@753,@754,@755,NULL,@756,NULL),(@757,@758,@759,@760,@761,@762,@763,NULL,@764,@765,@766,@767,NULL,@768,NULL),(@769,@770,@771,@772,@773,@774,@775,NULL,@776,@777,@778,@779,NULL,@780,NULL),(@781,@782,@783,@784,@785,@786,@787,NULL,@788,@789,@790,@791,NULL,@792,NULL),(@793,@794,@795,@796,@797,@798,@799,NULL,@800,@801,@802,@803,NULL,@804,NULL),(@805,@806,@807,@808,@809,@810,@811,NULL,@812,@813,@814,@815,NULL,@816,NULL),(@817,@818,@819,@820,@821,@822,@823,NULL,@824,@825,@826,@827,NULL,@828,NULL),(@829,@830,@831,@832,@833,@834,@835,NULL,@836,@837,@838,@839,NULL,@840,NULL),(@841,@842,@843,@844,@845,@846,@847,NULL,@848,@849,@850,@851,NULL,@852,NULL),(@853,@854,@855,@856,@857,@858,@859,NULL,@860,@861,@862,@863,NULL,@864,NULL),(@865,@866,@867,@868,@869,@870,@871,NULL,@872,@873,@874,@875,NULL,@876,NULL),(@877,@878,@879,@880,@881,@882,@883,NULL,@884,@885,@886,@887,NULL,@888,NULL),(@889,@890,@891,@892,@893,@894,@895,NULL,@896,@897,@898,@899,NULL,@900,NULL),(@901,@902,@903,@904,@905,@906,@907,NULL,@908,@909,@910,@911,NULL,@912,NULL),(@913,@914,@915,@916,@917,@918,@919,NULL,@920,@921,@922,@923,NULL,@924,NULL),(@925,@926,@927,@928,@929,@930,@931,NULL,@932,@933,@934,@935,NULL,@936,NULL),(@937,@938,@939,@940,@941,@942,@943,NULL,@944,@945,@946,@947,NULL,@948,NULL),(@949,@950,@951,@952,@953,@954,@955,NULL,@956,@957,@958,@959,NULL,@960,NULL),(@961,@962,@963,@964,@965,@966,@967,NULL,@968,@969,@970,@971,NULL,@972,NULL)
```

这是 **SQL Server 查询执行计划（Query Execution Plan）** ，用于分析查询性能，核心信息拆解如下：

### 1. 整体结构

- **操作链路**：从右到左依次是 `Constant Scan`（常量扫描）→ `Compute Scalar`（标量计算）→ `Clustered Index Insert`（聚集索引插入）→ 最终通过 `INSERT` 语句写入数据。
- **成本占比**：`Clustered Index Insert` 占 **99.98%**，是性能瓶颈点，说明插入聚集索引的操作开销极大。

### 2. 关键操作解析

- `Clustered Index Insert`：
  - 对表 `[rt_PrdSNMltTrace_Detail]` 的聚集索引 `[PK_t_PrdSNMltTrace_Detail]` 执行插入。
  - 占了几乎全部 IO（100%）和 CPU（73.14%），若数据量大或索引设计不合理，会严重拖慢速度。
- **`Compute Scalar`**：对数据做标量计算（如表达式、函数运算），成本低（0.02%），影响小。
- **`Constant Scan`**：扫描常量数据（如字面量、固定值），为后续操作提供基础输入，成本也极低（0.01%）。

### 3. 优化建议

- **聚焦聚集索引插入**：检查表的聚集索引是否合理（如键值过大、碎片多），或考虑批量插入（减少多次索引维护开销）。
- **数据量验证**：确认插入 81 条记录的场景是否合理，若实际是高频插入，需优化索引或存储策略。
- **执行计划背景**：创建时间是 `1970-01-01`，可能是工具显示问题（实际执行时间需结合业务日志确认），不影响分析核心逻辑。



简单说，这个查询的性能 “卡” 在聚集索引插入，优先优化这部分就能大幅提升效率～

![image-20250609100640757](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609100640757.png)

```
SELECT [m].[PPID], [m].[AdjustQty], [m].[areaNo], [m].[BatchID], [m].[binLoc], [m].[BoxPPID], [m].[CustomerPO], [m].[DateCode], [m].[departmentNo], [m].[endDTime], [m].[ExpiredDTime], [m].[FailureDTime], [m].[feederNo], [m].[FinishedDTime], [m].[fromLoc], [m].[frommtlInventoryStatus], [m].[GRNo], [m].[inDTime], [m].[LastParentPPID], [m].[LendQty], [m].[Loc], [m].[LotCode], [m].[LotNo], [m].[ModifiedDTime], [m].[moduleNo], [m].[mtlControlStatus], [m].[mtlInventoryStatus], [m].[mtlStatus], [m].[noticeMessage], [m].[ParentPPID], [m].[partNo], [m].[planID], [m].[PO], [m].[POItem], [m].[PPIDType], [m].[prdLineNo], [m].[preProcessDTime], [m].[PrintDateTime], [m].[processSiteNo], [m].[Qty], [m].[rePrintFlag], [m].[refCode], [m].[remQty], [m].[ReturnQty], [m].[Rmks], [m].[ScrapedQty], [m].[siteCode], [m].[sourceType], [m].[startDTime], [m].[stationID], [m].[toLocSpec], [m].[TraceCode], [m].[trackNo], [m].[transInPPID], [m].[transInQty], [m].[TransOutQty], [m].[unAllocatedQty], [m].[UsedQty], [m].[UserID], [m].[VendorCode], [m].[VendorNo], [m].[WONo]
FROM [mtl_PPID] AS [m] WITH (NOLOCK)
WHERE [m].[PPID] IN ('ST2408011314400-005', 'ST2408024956300-005', 'ST2408023096800-005', 'ST2408013758400-005', 'ST2408024747700-005', 'ST2408019817700-005', 'ST2408027552000-005', 'ST2408022875600-005', 'ST2408004409300-005', 'ST2408013943000-005', 'ST2408027570000-005', 'ST2408021765500-005', 'ST2408021743800-005', 'ST2408008823700-005', 'ST2408023377900-005', 'ST2408022520900-005', 'ST2408013657800-005', 'ST2408002773100-005', 'ST2408022878600-005', 'ST2403076600100-005', 'ST2408013082400-005', 'ST2408010861500-005', 'ST2408021118400-005', 'ST2408004395200-005', 'ST2408025131600-005', 'ST2408029058100-005', 'ST2408005973200-005', 'ST2408027881900-005', 'ST2408002798100-005', 'ST2408000065600-005', 'ST2408025621300-005', 'ST2408000884600-005', 'ST2408028588800-005', 'ST2408002999300-005', 'ST2408025151700-005', 'ST2408026867900-005', 'ST2408002943900-005', 'ST2408023299700-005', 'ST2408017625300-005', 'ST2408003483200-005', 'ST2408025842600-005', 'ST2403093617200-005', 'ST2408004303700-005', 'ST2408017675700-005', 'ST2408019923900-005', 'ST2408022800000-005', 'ST2408020327600-005', 'ST2408018639400-005', 'ST2408013865600-005', 'ST2408014186600-005', 'ST2408021985000-005', 'ST2408023112200-005', 'ST2408028013900-005', 'ST2408028069100-005', 'ST2408013892400-005', 'ST2408010569200-005', 'ST2408028093100-005', 'ST2408008921900-005', 'ST2408003596000-005', 'ST2408026256200-005', 'ST2403057023200-005', 'ST2408028426600-005', 'ST2408021809000-005', 'ST2403059057400-005', 'ST2408006029500-005', 'ST2408025809500-005', 'ST2408005405000-005', 'ST2408025688400-005', 'ST2408009358300-005', 'ST2408010629000-005', 'ST2408009357900-005', 'ST2408005455500-005', 'ST2408021390400-005', 'ST2408013965100-005', 'ST2408021603500-005', 'ST2408025037300-005', 'ST2403050246500-005', 'ST2408009588200-005', 'ST2408005994900-005', 'ST2408025688500-005', 'ST2408009358400-005')
```

这是 SQL Server 的**预估查询执行计划（Estimated Query Plan）** ，用于分析 `SELECT` 查询的性能与执行逻辑，以下是详细拆解：

### 一、整体概览

查询逻辑：通过 `SELECT` 语句从表中读取数据，执行链路为 `Constant Scan`（常量扫描）→ `Nested Loops`（嵌套循环连接）→ `Compute Scalar`（标量计算）→ `Clustered Index Seek`（聚集索引查找） ，最终通过 `SELECT` 输出结果。

### 二、关键操作解析

#### 1. 性能占比核心：`Clustered Index Seek`（聚集索引查找）

- **成本占比**：`99.84%`（几乎是性能瓶颈点）
- **操作逻辑**：对表 `[mt_PPID]` 的聚集索引 `[PK_mt_PPID]` 执行**索引查找**，从索引结构中快速定位并读取数据。
- 资源消耗：
  - IO（输入输出）占比 `100%`：说明大部分磁盘读写开销集中在此（索引查找需访问磁盘页）。
  - CPU 占比 `26.98%`：索引遍历、数据筛选等逻辑消耗 CPU。
- **优化关联**：若查询慢，优先检查此索引是否合理（如索引键设计、碎片、是否覆盖查询所需字段 ）。

#### 2. `Nested Loops (Inner Join)`（嵌套循环连接）

- **成本占比**：`0.13%`（影响小，辅助连接数据）
- **操作逻辑**：以 “嵌套循环” 方式做**内连接**，将上游 `Compute Scalar` 结果与其他数据源（如常量扫描结果）关联，筛选出匹配行。
- **特点**：适合小数据量连接，若后续数据量增大，需关注是否因循环次数多导致性能下降。

#### 3. `Compute Scalar`（标量计算）

- **成本占比**：`0.00%`（几乎无开销）
- **操作逻辑**：对数据执行**标量运算**（如表达式计算、函数调用：`AdjustQty`、`DateCode` 等字段可能涉及转换 / 计算 ），为后续操作准备数据。
- **常见场景**：处理 `SELECT` 语句中的计算列（如 `SELECT Col1 + Col2 AS Total...` ）。

#### 4. `Constant Scan`（常量扫描）

- **成本占比**：`0.03%`（极低开销）
- **操作逻辑**：扫描**常量数据**（如查询中硬编码的值、固定参数 ），作为连接 / 计算的基础输入（比如 `WHERE` 条件里的常量过滤）。

### 三、执行计划关键结论

1. **性能瓶颈**：`Clustered Index Seek` 是绝对核心，需重点优化（如确认索引是否覆盖查询、索引碎片是否过高、查询条件是否精准命中索引 ）。
2. **数据流转**：从常量扫描 → 标量计算 → 嵌套循环连接 → 索引查找，链路清晰但高度依赖索引效率。
3. **资源分布**：IO 压力集中在索引查找，CPU 也主要消耗在此，需结合表数据量、索引设计做调优。

### 四、优化建议（可落地步骤）

1. 索引体检：

   - 检查 `[PK_mt_PPID]` 聚集索引是否为查询 “量身定制”（是否覆盖 `SELECT` 所需字段，避免回表 ）。
   - 用 `DBCC SHOWCONTIG` 或 `sys.dm_db_index_physical_stats` 查看索引碎片，高碎片时执行 `ALTER INDEX ... REBUILD` 整理。

2. 查询改写

   ：

   - 确认 `SELECT` 字段是否必要，减少不必要的列（降低 IO 传输量 ）。
   - 若 `WHERE` 条件复杂，尝试简化过滤逻辑，让索引查找更高效。

3. **执行计划验证**：优化后重新生成执行计划，对比 `Clustered Index Seek` 成本占比、IO/CPU 消耗，确认是否改善。



简单说，这个查询的性能 “命脉” 在聚集索引查找，围绕索引优化就能大幅提升效率，可按上述步骤逐步排查

![image-20250609100925667](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609100925667.png)

```
insert into dtm_barrier(trans_type, gid, branch_id, op, barrier_id, reason) values(@trans_type,@gid,@branch_id,@op,@barrier_id,@reason)
```

这是 SQL Server 的**预估查询执行计划（Estimated Query Plan）** ，对应一条带参数的 `INSERT` 语句（因涉及 `Clustered Index Insert` 等操作），以下是分层拆解：

### 一、整体流程与核心操作

执行链路：`Constant Scan`（常量扫描）→ 多次 `Compute Scalar`（标量计算）→ `Nested Loops (Left Semi Join)`（左半连接）→ `Index Seek`（索引查找）→ `Assert`/`Top` → 最终 `Clustered Index Insert`（聚集索引插入） ，核心是**通过复杂逻辑准备数据后，插入聚集索引**。

### 二、关键操作深度解析

#### 1. 性能 “大头”：`Clustered Index Insert`（聚集索引插入）

- **成本占比**：`85.88%`（绝对性能瓶颈）

- **操作逻辑**：向表 `[dtm_barrier]` 的**聚集索引 `[PK_dtm_barrier_...]`** 插入数据，需维护索引结构（如页分裂、日志写入 ）。

- 资源消耗

  ：

  - IO（输入输出）占 `86.89%`：磁盘写入开销大（聚集索引数据直接存表，插入需写磁盘页 ）。
  - CPU 占 `1.2%`：索引维护的计算逻辑（如排序、页分配 ）。

- **优化关联**：若插入慢，需检查聚集索引设计（如键值是否过大、是否引发频繁页分裂 ）、是否可批量插入（减少多次索引维护 ）。

#### 2. 数据准备关键：`Index Seek`（索引查找）

- **成本占比**：`14.1%`（次要但必要环节）

- **操作逻辑**：在表 `[dtm_barrier]` 的**非聚集索引 `[idx_dtm_barrier]`** 中查找数据，为插入做前置校验 / 关联（可能是唯一性检查、关联其他数据 ）。

- 资源消耗

  ：

  - IO 占 `13.13%`，CPU 占 `25.17%`：索引遍历、数据筛选的开销。

- **优化点**：确认索引是否必要，或是否可通过索引覆盖（Covering Index）减少回表 / 提升查找效率。

#### 3. 辅助逻辑：`Nested Loops (Left Semi Join)`（左半连接）

- **成本占比**：`14.12%` 链路中的一环
- **操作逻辑**：以 “左半连接” 方式关联数据（只返回左表匹配行，用于存在性检查，比如避免重复插入 ）。
- **特点**：适合小数据量关联，若数据量大，需关注是否因循环次数多导致性能下降。

#### 4. 其他辅助操作

- **`Compute Scalar`**：多次对数据做标量计算（如参数转换、表达式运算 ），为插入 / 关联准备格式化数据，单步成本低但多步累积可能有影响。
- **`Assert`**：验证数据约束（如唯一性、检查约束 ），确保插入符合规则。
- **`Top`**：限制数据行数（可能是业务逻辑需要，或避免全表扫描 ）。

### 三、执行计划结论与优化方向

1. **核心瓶颈**：`Clustered Index Insert` 是性能 “卡点”，需优先优化聚集索引的插入效率。
2. **数据准备依赖**：`Index Seek` 和嵌套循环连接为插入做前置逻辑，需确认索引、连接是否合理。
3. **资源分布**：IO 压力集中在聚集索引插入（磁盘写入），CPU 分散在索引查找和计算，需结合表结构、数据量调优。

### 四、可落地优化建议

1. 聚集索引优化

   ：

   - 检查 `[PK_dtm_barrier_...]` 聚集索引键值长度，过长会增加页分裂概率，可考虑更紧凑的键设计。
   - 若为高频插入，可尝试 `ALTER INDEX ... REBUILD` 整理索引碎片，或调整填充因子（Fill Factor）减少页分裂。

2. 插入逻辑优化

   ：

   - 确认是否可**批量插入**（减少多次索引维护开销），比如将单条插入改为 `INSERT ... SELECT` 批量提交。
   - 检查前置 `Index Seek` 的必要性，若只是重复校验，可通过应用层逻辑提前过滤。

3. **执行计划验证**：优化后重新生成执行计划，对比 `Clustered Index Insert` 的成本占比、IO/CPU 消耗，确认是否改善。



简单总结：这个插入操作的性能被聚集索引插入 “拖后腿”，围绕索引设计、插入方式优化就能显著提升效率，按上述步骤逐一排查即可～

![image-20250609103214090](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609103214090.png)

```
(@__prdLineNo_0 varchar(16),@__processSiteNo_1 varchar(100),@__prdNo_2 varchar(48),@__prdRev_3 varchar(4),@__planId_4 varchar(36))SELECT [m].[mtlTraceInputID], [m].[areaNo], [m].[ASL], [m].[BatchNo], [m].[CheckDTime], [m].[Checked], [m].[Checker], [m].[DateCode], [m].[departMent], [m].[dueFinishedDTime], [m].[FeederNo], [m].[FinishedDTime], [m].[GRNo], [m].[IC], [m].[IsAllCheck], [m].[IsSwitch], [m].[LastModifyDTime], [m].[LoadedQty], [m].[LoadingListID], [m].[Location], [m].[LotCode], [m].[LotNo], [m].[Marking], [m].[ModuleNo], [m].[mtlInDTime], [m].[mtlPartNo], [m].[parentTrackNo], [m].[perQty], [m].[planID], [m].[Posted], [m].[prdLineNo], [m].[prdNo], [m].[prdRev], [m].[ProcessSiteNo], [m].[ReelID], [m].[RemQty], [m].[ReturnQty], [m].[RwkOpTraceDetailID], [m].[ScrapQty], [m].[siteCode], [m].[Station], [m].[StationID], [m].[ToSpecLoc], [m].[TraceCode], [m].[TrackNo], [m].[UsedQty], [m].[UserDefined1], [m].[UserDefined2], [m].[UserDefined3], [m].[UserDefined4], [m].[UserDefined5], [m].[userID], [m].[Vendor], [m].[VendorCode], [m].[VendorNo]
FROM [mtl_TraceInput] AS [m] WITH (NOLOCK)
WHERE [m].[prdLineNo] = @__prdLineNo_0 AND [m].[ProcessSiteNo] = @__processSiteNo_1 AND [m].[prdNo] = @__prdNo_2 AND [m].[prdRev] = @__prdRev_3 AND [m].[planID] = @__planId_4 AND ([m].[FinishedDTime] IS NULL)
```

这是 SQL Server 的**预估查询执行计划（Estimated Query Plan）** ，对应一条带参数的 `SELECT` 查询，核心是通过 **索引查找（Index Seek） + 键查找（Key Lookup）** 组合获取数据，以下是分层拆解：

### 一、整体流程与核心操作

执行链路：`Constant Scan`（常量扫描，图中未完全显示但隐含参数传递）→ `Compute Scalar`（标量计算）→ `Nested Loops (Inner Join)`（嵌套循环内连接）→ 分支 1（`Index Seek`） + 分支 2（`Index Seek` + `Key Lookup`） → 最终通过 `SELECT` 输出结果。
核心逻辑：利用索引快速定位数据，因索引未覆盖全部查询字段，需回表做 `Key Lookup` 补充字段。

### 二、关键操作深度解析

#### 1. 性能平分秋色：`Index Seek` + `Key Lookup`

- `Index Seek`（索引查找）
  - **成本占比**：`49.97%`（和 `Key Lookup` 几乎均分成本）
  - **操作逻辑**：在表 `[mtl_TraceInput]` 的**非聚集索引 `[Index_mtl_TraceInput_RevLineSiteNo]`** 中，根据查询条件（`SEEK PREDICATE`）快速定位符合条件的行，获取索引键值。
  - 资源消耗：
    - IO（输入输出）占 `50.00%`，CPU 占 `49.32%`：磁盘读取索引页 + 遍历索引树的开销。
- `Key Lookup`（键查找，回表）
  - **成本占比**：`49.97%`
  - **操作逻辑**：利用 `Index Seek` 得到的聚集索引键（如主键），到表的**聚集索引（`[PK_MTL_TRACEINPUT]`）** 中 “回表” 查找完整行数据（补充非索引覆盖的字段 ）。
  - 资源消耗：
    - IO 占 `50.00%`，CPU 占 `49.32%`：回表时需额外读取聚集索引的数据页，是性能损耗点。

#### 2. 辅助逻辑：`Nested Loops (Inner Join)`

- **成本占比**：`49.97%` 链路中的连接环节
- **操作逻辑**：以 “嵌套循环” 方式做**内连接**，将上游 `Compute Scalar` 处理后的数据，与 `Index Seek` + `Key Lookup` 得到的结果关联，筛选出最终需要的行。
- **特点**：适合小数据量连接，若后续数据量增大，需关注是否因循环次数多导致性能下降。

#### 3. 其他辅助操作

- **`Compute Scalar`**：对输入数据执行标量计算（如参数转换、表达式运算 ），为连接 / 查询准备格式化数据，单步成本低（`0.06%` 等）但为必要环节。

### 三、执行计划结论与优化方向

1. **核心瓶颈**：`Key Lookup` 是隐性性能损耗（回表操作），若查询频繁且数据量大，会显著拖慢速度。
2. **索引设计问题**：当前非聚集索引 `[Index_mtl_TraceInput_RevLineSiteNo]` **未覆盖全部查询字段**，导致必须回表。
3. **资源分布**：IO 和 CPU 均匀分布在 `Index Seek` 和 `Key Lookup`，说明回表操作的磁盘 / 计算开销与索引查找相当。

### 四、可落地优化建议

1. **索引覆盖（Covering Index）优化**：

   - 改写非聚集索引，包含查询所需的

     所有字段

     （

     ```
     SELECT
     ```

     ```
     WHERE
     ```

      

     ```sql
     CREATE INDEX [Index_mtl_TraceInput_Covering] 
     ON [mtl_TraceInput] (-- 原索引键列 --)
     INCLUDE (-- 补充查询需要的其他列 --);
     ```

      

     ```
     Key Lookup
     ```

     ，直接从非聚集索引获取完整数据。

2. **查询改写**：

   - 检查 `SELECT` 字段是否必要，减少不必要的列（降低索引覆盖的字段数量，让索引更紧凑 ）。
   - 若业务允许，尝试简化查询条件，让 `Index Seek` 更高效（如避免复杂函数、模糊匹配 ）。

3. **执行计划验证**：

   - 优化索引后，重新生成执行计划，确认 `Key Lookup` 是否消失，`Index Seek` 成本是否降低。



简单总结：这个查询的性能被 “回表操作（Key Lookup）” 拖累，通过**索引覆盖优化**能直接解决问题，让查询效率翻倍～

![image-20250609104002230](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609104002230.png)

```
(@1 varchar(8000),@2 numeric(6,2),@3 varchar(8000))UPDATE [mtl_TraceInput] set [LastModifyDTime] = @1,[UsedQty] = @2  WHERE [mtlTraceInputID]=@3
```

这是 SQL Server 的**预估查询执行计划（Estimated Query Plan）** ，对应一条 `UPDATE` 语句，用于更新表 `[mtl_TraceInput]` 的数据。以下是详细分析：

### 一、查询语句与执行链路

- **SQL 语句**：

  sql

  

  

  

  

  

  ```sql
  UPDATE [mtl_TraceInput] 
  SET [LastModifyTime] = @1, [UsedQty] = @2 
  WHERE [mtlTraceInputID] = @3
  ```

  逻辑：根据 `[mtlTraceInputID]` 主键条件，更新 `LastModifyTime` 和 `UsedQty` 字段。

- **执行链路**：`UPDATE` 操作直接调用 `Clustered Index Update`（聚集索引更新），无其他复杂环节（因是主键精准匹配，定位数据高效 ）。

### 二、核心操作：`Clustered Index Update`（聚集索引更新）

- **成本占比**：`100%`（唯一主要操作，所有开销集中在此 ）
- **操作逻辑**：
  1. 通过聚集索引 `[PK_MTL_TRACEINPUT]`（主键索引，因 `WHERE` 条件用了主键 `mtlTraceInputID` ）快速定位到要更新的行。
  2. 修改该行的 `LastModifyTime` 和 `UsedQty` 字段值。
  3. 维护聚集索引的结构（如日志写入、页分裂预防等，若数据页有变化 ）。
- **资源消耗**：
  - IO（输入输出）占 `100%`：更新聚集索引需读写磁盘数据页（找到行 → 修改 → 写回 ）。
  - CPU 占 `100%`：索引定位、数据修改、日志记录等逻辑消耗 CPU。

### 三、执行计划结论与优化点

1. **执行效率本质**：
   因是**主键精准匹配**（`WHERE [mtlTraceInputID] = @3` ），SQL Server 能直接通过聚集索引快速定位到 1 行数据（`records: 1` ），执行计划非常简洁高效。
2. **潜在优化思考**：
   - **索引健康**：检查聚集索引 `[PK_MTL_TRACEINPUT]` 是否有碎片（用 `sys.dm_db_index_physical_stats` ），高碎片时可 `REBUILD` 索引提升读写效率。
   - **参数嗅探**：若执行计划因参数（`@3` 等）导致效率变化，可检查是否存在 “参数嗅探” 问题（需结合实际执行情况判断 ）。
   - **日志与锁**：高频更新时，需关注事务日志写入压力和锁竞争（虽当前单次更新影响小，但批量场景需注意 ）。

### 四、总结

这条 `UPDATE` 语句的执行计划非常高效，因利用了**聚集索引（主键）的精准查找**，直接定位并更新数据。若遇到性能问题，优先检查：



- 聚集索引是否碎片化严重；
- 更新是否因并发导致锁等待；
- 参数是否稳定（避免极端值引发低效计划 ）。



简单说，这是 “最优解” 级别的执行计划，主键精准更新天生高效，日常维护好索引即可



![image-20250609133911440](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609133911440.png)

```
select top 1 planno,prdsn,line from rt_Test_PrdSN_ProcessSiteNo_ICT with(nolock) where nextflag = 0  and planno='12271022-0414N-L1-011'
```

### 一、执行计划分析

这是 **SQL Server 中带 `WITH(NOLOCK)` 的 `SELECT TOP 1` 查询**，目标是从 `rt_Test_PrdSN_ProcessSiteNo_ICT` 表取满足 `nextflag = 0` 且 `planno='12271022-0414N-L1-011'` 的首行数据。执行计划核心问题是 **`Key Lookup`（回表）导致的性能损耗**，拆解如下：

#### 1. 关键操作与成本占比

- **`Index Seek`（索引查找）**：
  - 成本占比 `15.42%`，利用非聚集索引 `[rt_Test_PrdSN_ProcessSiteNo_ICT].[rt_Test_PrdSN_ProcessSiteNo_ICT_NextFlag]` 筛选 `nextflag = 0` 和 `planno` 条件的数据。
  - 作用：快速缩小范围，找到符合条件的索引键。
- **`Key Lookup`（键查找 / 回表）**：
  - 成本占比 `84.42%`，是性能瓶颈！因非聚集索引未覆盖 `SELECT` 所需的 `planno`、`prdsn`、`line` 字段，需用索引键（如主键）到**聚集索引（`[PK_DS8U5_rt_Test_PrdSN_ProcessSiteNo_ICT]`）** 中 “回表” 取完整数据。
  - 问题：回表会额外读取聚集索引的数据页，增加 IO 和 CPU 开销。

#### 2. 辅助操作

- **`Nested Loops (Inner Join)`**：嵌套循环连接，关联 `Index Seek` 和 `Key Lookup` 的结果，筛选出最终行。
- **`Top`**：限制结果为 1 行，提前终止不必要的后续操作（因 `SELECT TOP 1` ）。

### 二、优化方案（消除回表，提升效率）

核心思路：**创建 “覆盖索引”**，让非聚集索引包含 `SELECT` 所需的所有字段，避免 `Key Lookup`。

#### 1. 步骤 1：分析查询需求

查询需要的字段：



- 筛选条件：`nextflag`（索引已用）、`planno`
- 返回字段：`planno`、`prdsn`、`line`



目标：让非聚集索引直接覆盖这些字段，无需回表。

#### 2. 步骤 2：创建覆盖索引

sql

```sql
CREATE INDEX [IX_rt_Test_PrdSN_ProcessSiteNo_ICT_Covering] 
ON [rt_Test_PrdSN_ProcessSiteNo_ICT] (nextflag, planno)  -- 筛选条件放索引键
INCLUDE (prdsn, line);  -- 返回字段放 INCLUDE，避免回表
```



**逻辑**：索引键（`nextflag, planno`）用于快速筛选数据。

- `INCLUDE` 包含 `prdsn, line`，使索引直接覆盖查询所需字段，消除 `Key Lookup`。

#### 3. 步骤 3：验证执行计划

创建索引后，重新执行查询并查看执行计划，理想结果：



- `Key Lookup` 消失，只剩 `Index Seek`（或更高效的索引扫描）。
- 成本占比集中在索引查找，IO 和 CPU 开销显著降低。

#### 4. 可选：清理旧索引（按需）

若原非聚集索引 `[rt_Test_PrdSN_ProcessSiteNo_ICT_NextFlag]` 无其他用途，可删除以减少维护开销：

```sql
DROP INDEX [rt_Test_PrdSN_ProcessSiteNo_ICT_NextFlag] 
ON [rt_Test_PrdSN_ProcessSiteNo_ICT];
```

### 三、优化前后对比

| 优化前                           | 优化后                           |
| -------------------------------- | -------------------------------- |
| 存在 `Key Lookup`（84.42% 成本） | 无 `Key Lookup`，仅 `Index Seek` |
| 需回表读取聚集索引数据           | 直接从覆盖索引取数               |
| IO 和 CPU 开销高                 | IO 和 CPU 开销显著降低           |

### 四、额外建议

- **`WITH(NOLOCK)` 风险**：虽能减少锁竞争，但可能读取未提交数据（脏读），需确认业务是否允许。
- **索引维护**：定期检查覆盖索引的碎片（用 `sys.dm_db_index_physical_stats` ），碎片高时重建（`ALTER INDEX ... REBUILD` ）。



总结：通过**覆盖索引**消除回表，可让查询性能大幅提升，执行计划更简洁高效！

![image-20250609141325876](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250609141325876.png)

```
(@__8__locals1_gid_0 nvarchar(50),@__8__locals1_methodName_2 nvarchar(4000))SELECT [e].[Id], [e].[Added], [e].[After], [e].[Before], [e].[DtmId], [e].[Entity], [e].[ExpiresAt], [e].[Retries], [e].[Rmks], [e].[Status]
FROM [EntityUpdateLog] AS [e] WITH (NOLOCK)
WHERE [e].[DtmId] = @__8__locals1_gid_0 AND [e].[Retries] = 0 AND ([e].[Entity] = N'EasEquipment' OR [e].[Rmks] = @__8__locals1_methodName_2)
ORDER BY [e].[Added] DESC
```

### 一、执行计划分析

这是 **SQL Server 中带参数的 `SELECT` 查询**，目标从 `[EntityUpdateLog]` 表取数据，执行计划存在 **2 类核心问题**：

#### 1. 关键操作与性能瓶颈

- **`Clustered Index Scan`（聚集索引扫描）**：
  - 成本占比 `99.89%`，是性能瓶颈！因查询需扫描整个聚集索引（`[PK_EntityUpdateLog]`）来筛选数据，而非精准查找。
  - 问题：若表数据量大，全表扫描会导致极高 IO 和 CPU 开销。
- **辅助操作**：
  - `Parallelism (Gather Streams)`：并行收集结果（占比 `99.89%` 链路的一部分），但因扫描成本高，并行收益有限。
  - `Sort`：对结果排序（占比 `0.02%` ），若排序字段无索引，会额外消耗资源。

#### 2. 警告（Warnings）分析

- `Type conversion: Cardinality Estimate`

  （类型转换导致基数估计错误）：

  - 原因：查询中参数（`@__8__locals1_gid_0`、`@__8__locals1_methodName_2`）与表字段可能存在**隐式类型转换**（如 `varchar` 与 `nvarchar` 不匹配 ）。
  - 影响：SQL Server 无法准确预估符合条件的行数，导致执行计划选择低效（如本案例的全表扫描，而非索引查找）。

### 二、优化方案（分步骤解决）

#### 1. 解决 “类型转换” 问题（核心前提）

隐式类型转换会破坏索引使用、导致基数估计错误，必须优先修复：



sql

```sql
-- 原查询可能存在类型不匹配（如参数是 nvarchar，表字段是 varchar）
-- 显式转换参数类型，与表字段一致
SELECT [e].[Id], [e].[Added], ... 
FROM [EntityUpdateLog] [e]
WHERE [e].[Gid] = CONVERT(varchar(50), @__8__locals1_gid_0)  -- 假设表字段是 varchar(50)
  AND [e].[MethodName] = CONVERT(varchar(4000), @__8__locals1_methodName_2);  -- 与表字段类型匹配
```



**逻辑**：让参数类型与表字段严格一致（如均为 `varchar` 或 `nvarchar` ），消除隐式转换，使 SQL Server 能准确预估行数并选择更优计划。

#### 2. 创建 “覆盖索引” 消除全表扫描

若查询需频繁执行，创建覆盖索引让查询无需扫描全表：



sql











```sql
CREATE INDEX [IX_EntityUpdateLog_Covering] 
ON [EntityUpdateLog] (Gid, MethodName)  -- 筛选条件字段放索引键
INCLUDE (Id, Added, After, Before, DtmId, ...);  -- 返回字段放 INCLUDE
```



**逻辑**：



- 索引键（`Gid, MethodName`）用于快速筛选数据，替代全表扫描。
- `INCLUDE` 包含所有 `SELECT` 字段，避免回表，进一步提升效率。

#### 3. 验证执行计划

优化后重新执行查询，理想结果：



- `Clustered Index Scan` 替换为 `Index Seek`（通过新索引精准查找）。
- 警告消失，基数估计准确，执行计划成本大幅降低。

#### 4. 清理与维护（可选）

若原查询无特殊需求，可删除不必要的并行提示（如 `MAXDOP` 控制），或定期维护索引碎片：



sql











```sql
-- 重建索引（碎片高时）
ALTER INDEX [IX_EntityUpdateLog_Covering] ON [EntityUpdateLog] REBUILD;
```

### 三、优化前后对比

| 优化前                             | 优化后                   |
| ---------------------------------- | ------------------------ |
| 全表扫描（`Clustered Index Scan`） | 索引查找（`Index Seek`） |
| 类型转换警告导致低效计划           | 无警告，执行计划精准     |
| 高 IO、CPU 开销                    | 低 IO、CPU 开销          |

### 四、额外建议

- **参数嗅探**：若优化后仍有性能波动，检查是否因 “参数嗅探” 导致计划缓存异常，可尝试 `OPTION (RECOMPILE)` 强制重新生成计划。
- **业务逻辑简化**：若查询只需首行或少量数据，可增加 `TOP` 限制（如 `SELECT TOP 1 ...` ），让执行计划更早终止扫描。



总结：核心问题是**类型转换破坏执行计划** + **全表扫描性能差**，通过**显式类型转换** + **覆盖索引**可彻底解决，让查询效率提升数倍甚至数十倍！