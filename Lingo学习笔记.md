# 9.6
## 基础语法
* Lingo大小写不敏感
* 注释要以；结束
* 对约束进行命名，使用方括号括起来放在约束的前面。
* 约束命名的好处是解答报告方便阅读了。
* 对于约束，Slack or Surplus表示约束距离相等还差多少，如果约束是矛盾的那么这个值是负数，可以以此发现不可行模型中的矛盾约束
* Dual Price：对偶变量。
* 可以使用`TITLE XXX;`增加标题
* @WARP(INDEX,LIMIT) 函数是的下标落在1和LIMIT之间
* 建立过滤器`PAIRS(ANALYSTS,ANALYSTS)|&2#gt#&1:RATING,MATCH`,&1，&2表示占位符，可以生成稀疏集
* 变量限定函数：
* * @GIN,整数
* * @BIN 0，1
* * @FREE 任何实数
* * @BND 在一定范围
* 

## 运输问题
某公司有6个货栈向8个销售商供应小装饰品。每个货栈的供应量是有限的，每一个销售商的需求量必须得到满足。公司要决定如何调运货栈的装饰品满足总运输成本最少。

第i个货栈的供应量为$WH_I$，第j个销售商的需求量为$V_j$，第i个货栈向第j个销售商的运输费用为$COST_{ij}$。

决策变量：定义$VOLUME_{ij}$表示从第i个货栈运输到第j个销售商的装饰品数量。

目标函数：

$$\min \sum_{i,j}cost\cdot volume$$

约束条件：
* 需求约束
* 供应能力约束
* 
使用lingo实现

    MODEL:
    ! A 6 Warehouse 8 Vendor Transportation Problem;
    SETS:
    WAREHOUSES: CAPACITY;
    VENDORS: DEMAND;
    LINKS( WAREHOUSES, VENDORS): COST, VOLUME;
    ENDSETS
    ! Here is the data;
    DATA:
    !set members;
    WAREHOUSES = WH1 WH2 WH3 WH4 WH5 WH6;
    VENDORS = V1 V2 V3 V4 V5 V6 V7 V8;

    !attribute values;
    CAPACITY = 60 55 51 43 41 52;
    DEMAND = 35 37 22 32 41 32 43 38;
    COST = 6 2 6 7 4 2 5 9
            4 9 5 3 8 5 8 2
            5 2 1 9 7 4 3 3
            7 6 7 3 9 2 7 1
            2 3 9 5 7 2 6 5
            5 5 2 2 8 1 4 3;
    ENDDATA
    ! The objective;
    MIN = @SUM( LINKS( I, J): 
        COST( I, J) * VOLUME( I, J));
    ! The demand constraints;
    @FOR( VENDORS( J): 
        @SUM( WAREHOUSES( I): VOLUME( I, J)) = 
        DEMAND( J));
    ! The capacity constraints;
    @FOR( WAREHOUSES( I): 
        @SUM( VENDORS( J): VOLUME( I, J)) <= 
        CAPACITY( I));
    END

这种难度级别的线性规划lingo就是秒解。

