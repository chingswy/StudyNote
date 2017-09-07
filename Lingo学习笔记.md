# 9.6
## 基础语法
* Lingo大小写不敏感
* 注释要以；结束
* 对约束进行命名，使用方括号括起来放在约束的前面。
* 约束命名的好处是解答报告方便阅读了。
* 对于约束，Slack or Surplus表示约束距离相等还差多少，如果约束是矛盾的那么这个值是负数，可以以此发现不可行模型中的矛盾约束
* Dual Price：对偶变量。
* 可以使用`TITLE XXX;`增加标题
* @WRAP(INDEX,LIMIT) 函数是的下标落在1和LIMIT之间
* 建立过滤器`PAIRS(ANALYSTS,ANALYSTS)|&2#gt#&1:RATING,MATCH`,&1，&2表示占位符，可以生成稀疏集
* 变量限定函数：
* * @GIN,整数
* * @BIN 0，1
* * @FREE 任何实数
* * @BND 在一定范围

### 集合处理函数
* @IN
* @INDEX
* @SIZE
* @WRAP

## 运输问题
某公司有6个货栈向8个销售商供应小装饰品。每个货栈的供应量是有限的，每一个销售商的需求量必须得到满足。公司要决定如何调运货栈的装饰品满足总运输成本最少。

第i个货栈的供应量为$WH_I$，第j个销售商的需求量为$V_j$，第i个货栈向第j个销售商的运输费用为$COST_{ij}$。

决策变量：定义$VOLUME_{ij}$表示从第i个货栈运输到第j个销售商的装饰品数量。

目标函数：

$$\min \sum_{i,j}cost\cdot volume$$

约束条件：
* 需求约束
* 供应能力约束

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

## 装配线平衡模型
有11项作业要指派到4个工作站上，作业之间有优先关系。需要找出分配方案是的装配线的总完成时间最小。

代码如下


    MODEL:
    ! Assembly line balancing model装配线平衡模型;   
    ! This model involves assigning tasks to stations
        in an assembly line so bottlenecks are avoided.
        Ideally, each station would be assigned an 
        equal amount of work.;
    SETS:
    ! The set of tasks to be assigned are A through K,
        and each task has a time to complete, T;
    TASK/ A B C D E F G H I J K/: T;
    ! 作业集合，有一个完成时间属性;
    ! Some predecessor,successor pairings must be
        observed(e.g. A must be done before B, B 
        before C, etc.);
    ! 作业之间的优先关系集合（A必须完成才能开始B这样的）;
    ! 这种表示方法需要学习一个;
    PRED( TASK, TASK)/ A,B  B,C  C,F  C,G  F,J  G,J
        J,K  D,E  E,H  E,I  H,J  I,J /;

    ! There are 4 workstations;
    STATION/1..4/;

    TXS( TASK, STATION): X;
    ! X 是决策变量;
    ! X(i,k) = 1 表示第I个作业指派给第K个工作站完成;
    ! X is the attribute from the derived set TXS 
        that represents the assignment. X(I,K) = 1
        if task I is assigned to station K;
    ENDSETS

    DATA:
    ! Data taken from Chase and Aquilano, POM;
    ! There is an estimated time required for each 
        task:
            A  B  C  D  E  F  G  H  I  J  K;
    ! 作业的完成时间;
    T = 45 11  9 50 15 12 12 12 12  8  9;
    ENDDATA

    ! The model;  
    ! *Warning* may be slow for more than 15 tasks;

    ! For each task, there must be one assigned 
        station;
    ! 每个工作必须指派到一个工作站;
    @FOR( TASK( I): @SUM( STATION( K): X( I, K)) = 1);

    ! Precedence constraints;
    ! For each precedence pair, the predecessor task
        I cannot be assigned to a later station than its
        successor task J;
    ! 对于每一个存在优先关系的作业对来说，
        前者对应的工作站 I 必须小于后者对应的工作站 J ;
    ! **这句操作非常骚强调一下**;
    ! 这个意思好像是说，两个相邻的任务，后面一个任务必须指派到前面一个任务的后面的工作站？？？

疑点 凭什么啊;
 这个求的是循环时间，不是平行机排序问题的时间


    @FOR( PRED( I, J):
    @SUM( STATION( K): 
        K * X( J, K) - K * X( I, K)) >= 0);

    ! For each station, the total time for the 
        assigned tasks must be less than the maximum
        cycle time, CYCTIME;
    @FOR( STATION( K):
    @SUM( TXS( I, K): T( I) * X( I, K)) <= CYCTIME);

    ! Minimize the maximum cycle time;
    MIN = CYCTIME;

    ! The X(I,J) assignment variables are 
        binary integers;
    @FOR( TXS: @BIN( X));

    END

### 最短线路模型
在有向图中寻找两个节点直接的最短距离。
使用的是DJKStra算法。动态规划的思路。
代码：DYNAMB

    MODEL:
    SETS:
    ! Dynamic programming illustration (see Anderson,
    Sweeney & Williams, An Intro to Mgt Science, 
    6th Ed.). We have a network of 10 cities.  We
    want to find the length of the shortest route
    from city 1 to city 10.;

    ! Here is our primitive set of ten cities, where
    F( i) represents the shortest path distance
    from city i to the last city;
    CITIES /1..10/: F;

    ! The derived set ROADS lists the roads that 
    exist between the cities (note: not all city
    pairs are directly linked by a road, and roads
    are assumed to be one way.);
    ROADS( CITIES, CITIES)/
    1,2  1,3  1,4
    2,5  2,6  2,7
    3,5  3,6  3,7
    4,5  4,6
    5,8  5,9
    6,8  6,9
    7,8  7,9
    8,10
    9,10/: D;
    ! 生成的派生集合不是把所有的组合都写出来
        把直接通的道路表示出来就可以了;
    ! D( i, j) is the distance from city i to j;
    ENDSETS

    DATA:
    ! Here are the distances that correspond to the 
    above links;
    D =
        1    5    2
        13   12   11
        6   10    4
        12   14
        3    9
        6    5
        8   10
        5
        2;
    ENDDATA

    ! If you are already in City 10, then the cost to
    travel to City 10 is 0;
    F( @SIZE( CITIES)) = 0;

    ! The following is the classic dynamic programming
    recursion. In words, the shortest distance from
    City i to City 10 is the minimum over all 
    cities j reachable from i of the sum of the 
    distance from i to j plus the minimal distance
    from j to City 10;
    @FOR( CITIES( i)| i #LT# @SIZE( CITIES):
    F( i) = @MIN( ROADS( i, j): D( i, j) + F( j))
    );
    ! 对于每一个城市来说，城市i到城市10 的最短距离就是对于所有城市i的相邻城市中，城市i到城市j的距离加上城市j到城市10的距离的最小值
    这个看上去好像有递归求解的过程，不知道会不会影响效率;
    END

### 金融模型 没看
### 排队模型  没看
### 排课模型

## 市场营销模型
### 马尔可夫链模型


    MODEL:
    ! Markov chain model;
    SETS:
    ! There are four states in our model and over time
    the model will arrive at a steady state 
    equilibrium.
    SPROB( J) = steady state probability
    稳定情况下的概率值;
    STATE/ A B C D/: SPROB;

    ! For each state, there's a probability of moving
    to each other state. TPROB( I, J) = transition
    probability
    对于每一个状态，下一阶段以一定概率变为其他状态;
    SXS( STATE, STATE): TPROB;
    ENDSETS

    DATA:
    ! The transition probabilities. These are proba-
    bilities of moving from one state to the next in
    each time period. Our model has four states, for
    each time period there's a probability of moving
    to each of the four states. The sum of proba-
    bilities across each of the rows is 1, since the
    system either moves to a new state or remains in
    the current one.;
    TPROB = .75  .1  .05  .1
            .4   .2  .1   .3
            .1   .2  .4   .3
            .2   .2  .3   .3;
    ENDDATA

    ! The model;
    !  Steady state equations;
    !   Only need N equations, so drop last
     这里的意思是说，稳定的概率需要满足稳定性条件，如果不等就不满足
     那么就会寻找其他解;
    @FOR( STATE( J)| J #LT# @SIZE( STATE):
    SPROB( J) = @SUM( SXS( I, J): SPROB( I) *
        TPROB( I, J))
    );

    ! The steady state probabilities must sum to 1;
    @SUM( STATE: SPROB) = 1;

    ! Check the input data, warn the user if the sum of
    probabilities in a row does not equal 1.;
    @FOR( STATE( I):
    @WARN( 'Probabilities in a row must sum to 1.',
        @ABS( 1 - @SUM( SXS( I, K): TPROB( I, K)))
        #GT# .000001);
    );

    END

### 联合分析模型
这个是物品有两个属性联合控制物品的排行，如何根据

    MODEL:

    ! Conjoint analysis model to decide how much weight
    to give to the two product attributes of warranty
    length and price;

    SETS:
    ! The three possible warranty lengths;
        WARRANTY /LONG, MEDIUM, SHORT/ : WWT;
    ! where WWT( i) = utility assigned to warranty i;

    ! The three possible price levels (high, 
    medium, low);
        PRICE /HIGH, MEDIUM, LOW/ : PWT;
    ! where PWT( j) = utility assigned to price j;

    ! We have a customer preference ranking for each
    combination;
        WP( WARRANTY, PRICE) : RANK;
    ENDSETS

    DATA:
    ! Here is the customer preference rankings running
    from a least preferred score of 1 to the most
    preferred of 9.  Note that long warranty and low
    price are most preferred with a score of 9, 
    while short warranty and high price are least 
    preferred with a score of 1;

        RANK = 7  8  9
                3  4  6
                1  2  5;
    ENDDATA

    SETS:
    ! The next set generates all unique pairs of product
    configurations such that the second configuration
    is preferred to the first;
        WPWP( WP, WP) | RANK( &1, &2) #LT# 
        RANK( &3, &4): ERROR;
    ! The attribute ERROR computes the error of our 
    estimated preference from the preferences given us
    by the customer;
    ENDSETS

    ! For every pair of rankings, compute the amount by
    which our computed ranking violates the true 
    ranking. Our computed ranking for the (i,j) 
    combination is given by the sum WWT( i) + PWT( j).
    (NOTE: This makes the bold assumption that 
    utilities are additive!);
        @FOR( WPWP( i, j, k, l): ERROR( i, j, k, l) >=
        1 + ( WWT( i) + PWT( j)) - 
        ( WWT( k) + PWT( l))
        );
    ! The 1 is required on the righthand-side of the 
    above equation to force ERROR to be nonzero in the
    case where our weighting scheme incorrectly 
    predicts that the combination (i,j) is equally
    preferred to the (k,l) combination.

    Since variables in LINGO have a default lower 
    bound of 0, ERROR will be driven to zero when we
    correctly predict that (k,l) is preferred to 
    (i,j).

    Next, we minimize the sum of all errors in order 
    to make our computed utilities as accurate as
    possible;
        MIN = @SUM( WPWP: ERROR);
    END





