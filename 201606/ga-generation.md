# 遗传算法在自动组卷中的应用

## 遗传算法
遗传算法(Genetic Algorithm)是一种模拟自然界的进化规律-优胜劣汰演化来的随机搜索算法，其在解决多种约束条件下的最优解这类问题上具有优秀的表现.

### 1. 基本概念

在遗传算法中有几个基本的概念：基因、个体、种群和进化.基因是个体的表现，不同个体的基因序列不同；个体是指单个的生命，个体是组成种群的基础；而进化的基本单位是种群，一个种群里面有多个个体；进化是指一个种群进过优胜劣汰的自然选择后，产生一个新的种群的过程，理论上进化会产生更优秀的种群.

### 2. 算法流程
一个传统的遗传算法由以下几个组成部分：

* 初始化. 随机生成一个规模为N的种群，设置最大进化次数以及停止进化条件.
* 计算适应度. 适应度被用来评价个体的质量，且适应度是唯一评判因子.计算种群中每个个体的适应度，得到最优秀的个体.
* 选择. 选择是用来得到一些优秀的个体来产生下一代.选择算法的好坏至关重要，因为在一定程度上选择会影响种群的进化方向.常用的选择算法有：随机抽取、竞标赛选择以及轮盘赌模拟法等等.
* 交叉. 交叉是两个个体繁衍下一代的过程，实际上是子代获取父亲和母亲的部分基因，即基因重组.常用的交叉方法有：单点交叉、多点交叉等.
* 变异. 变异即模拟突变过程.通过变异，种群中个体变得多样化.但是变异是有一个概率的.

经典的遗传算法的流程图如下所示：

![](http://7xryc7.com1.z0.glb.clouddn.com/lc.png)

### 3. java实现
为了防止进化方向出现偏差，在本算法中采用精英主义，即每次进化都保留上一代种群中最优秀的个体。

* 个体适应度：通过比较个体与期望值的相同位置上的基因，相同则适应度加1
* 选择策略：随机产生一个淘汰数组，选择淘汰数组中的最优秀个体作为选择结果，即模拟优胜劣汰的过程
* 交叉策略：对于个体的每个基因，产生一个随机数，如果随机数小于交叉概率，则继承父亲该位置的基因，否则继承母亲的该位置的基因
* 变异策略：个体的基因序列上的每个基因都有变异的机会，如果随机概率大于变异概率，则进行基因突变，本例中的突变策略是：随机产生一个0或者1

计算适应度
```
/**
 * 通过和solution比较 ，计算个体的适应值
 * @param individual 待比较的个体
 * @return  返回适应度
 */
public static int getFitness(Individual individual) {
    int fitness = 0;
    for (int i = 0; i < individual.size() && i < solution.length; i++) {
        if (individual.getGene(i) == solution[i]) {
            fitness++;
        }
    }
    return fitness;
}
```
选择算子
```
/**
 * 随机选择一个较优秀的个体。用于进行交叉
 * @param pop 种群
 * @return
 */
private static Individual tournamentSelection(Population pop) {
    Population tournamentPop = new Population(tournamentSize, false);
    // 随机选择 tournamentSize 个放入 tournamentPop 中
    for (int i = 0; i < tournamentSize; i++) {
        int randomId = (int) (Math.random() * pop.size());
        tournamentPop.saveIndividual(i, pop.getIndividual(randomId));
    }
    // 找到淘汰数组中最优秀的
    Individual fittest = tournamentPop.getFittest();
    return fittest;
}
```
交叉算子
```
/**
 * 两个个体交叉产生下一代
 * @param indiv1 父亲
 * @param indiv2 母亲
 * @return 后代
 */
private static Individual crossover(Individual indiv1, Individual indiv2) {
    Individual newSol = new Individual();
    // 随机的从两个个体中选择
    for (int i = 0; i < indiv1.size(); i++) {
        if (Math.random() <= uniformRate) {
            newSol.setGene(i, indiv1.getGene(i));
        } else {
            newSol.setGene(i, indiv2.getGene(i));
        }
    }
    return newSol;
}
```
变异算子
```
/**
 * 突变个体。突变的概率为 mutationRate
 * @param indiv 待突变的个体
 */
private static void mutate(Individual indiv) {
    for (int i = 0; i < indiv.size(); i++) {
        if (Math.random() <= mutationRate) {
            // 生成随机的 0 或 1
            byte gene = (byte) Math.round(Math.random());
            indiv.setGene(i, gene);
        }
    }
}
```

### 4. 测试结果
测试结果如下图

![](http://7xryc7.com1.z0.glb.clouddn.com/result1.png)

------
## 遗传算法与自动组卷

随着软件和硬件技术的发展，在线考试系统正在逐渐取代传统的线下笔试。对于一个在线考试系统而言，考试试卷的质量很大程度上代表着该系统的质量，试卷是否包含足够多的题型、是否包含指定的知识点以及试卷整体的难度系数是否合适等等，这些都能作为评价一个在线测评系统的指标.如果单纯的根据组卷规则直接从数据库中获取一定数量的试题组成一套试卷，由于只获取一次，并不能保证这样的组卷结果是一个合适的结果，而且可以肯定的是，这样得到的结果基本不会是一个优秀的解.显而易见，我们需要一个优秀的自动组卷算法，遗传算法就非常适合解决自动组卷的问题，其具有自进化、并行执行等特点

### 1. 对遗传算法的改进

使用传统的遗传算法进行组卷时会出现一些偏差，进化的结果不是非常理想.具体表现为：进化方向出现偏差、搜索后期效率低、容易陷入局部最优解等问题.针对这些问题，本系统对传统的遗传算法做了一些改进，具体表现为：使用精英主义模式(即每次进化都保留上一代种群的最优解)、实数编码以及选择算子的优化.

#### 1.1 染色体编码方式的改进

染色体编码是遗传算法首先要解决的问题，是将个体的特征抽象为一套编码方案.在传统的遗传算法解决方案中，二进制编码使用的最多，就本系统而言，二进制编码形成的基因序列为整个题库，这种方案不是很合适，因为二进制编码按照题库中试题的相对顺序将题库编码成一个01字符串，1代表试题出现，0代表没有显然这样的编码规模太大，对于一个优秀的题库而言，十万的试题总量是很常见的，对于一个长度为十万的字符串进行编码和解码显然太繁琐. 经过查阅资料，于是决定采用实数编码作为替代，将试题的id作为基因，试卷和染色体建立映射关系，同一类型的试题放在一起.比如，要组一套java考试试卷，题目总数为15：填空3道，单选10道，主观题2道.那么进行实数编码后，其基因序列分布表现为：

![](http://7xryc7.com1.z0.glb.clouddn.com/st.png)

#### 1.2 初始化种群设计

初始化试卷时不采取完全随机的方式.通过分析不难发现，组卷主要有题型、数量、总分、知识点和难度系数这五个约束条件，在初始化种群的时候，我们可以根据组卷规则随机产生指定数量的题型，这样在一开始种群中的个体就满足了题型、数量和总分的约束，使约束条件从5个减少为2个：知识点和难度系数.这样算法的迭代次数被减少，收敛也将加快.

#### 1.3 适应度函数设计

在遗传算法中，适应度是评价种群中个体的优劣的唯一指标，适应度可以影响种群的进化方向.由于在初始化时，种群中个体已经满足了题型、数量和总分这三个约束条件，所以个体的适应度只与知识点和难度系数有关.

试卷的难度系数计算公式为：

![](http://7xryc7.com1.z0.glb.clouddn.com/gs1.png)

n是组卷规则要求的题目总数，Ti，Ki分别是第i题的难度系数和分数.

本例中使用知识点覆盖率来评价知识点.即一套试卷要求包含N个知识点，而某个体中包含的知识点数目为M(去重后的结果，M<=N)，那么该个体的知识点覆盖率为：M/N. 因此，适应度函数为：

![](http://7xryc7.com1.z0.glb.clouddn.com/gs2.png)

其中，M/N为知识点覆盖率；EP为用户输入的整体期望难度，P为整体实际难度；知识点权重用t1表示，难度系数权重用t2表示.

#### 1.4 选择算子与交叉算子的改进

本例中的选择策略为：指定一个淘汰数组的大小(笔者使用的是5)，从原种群中随机挑选个体组成一个淘汰种群，将淘汰种群中的最优个体作为选择算子的结果.

交叉算子实际上是染色体的重组.本系统中采用的交叉策略为：在(0,N)之间随机产生两个整数n1，n2，父亲基因序列上n1到n2之间的基因全部遗传给子代，母亲基因序列上的n1到n2之外的基因遗传给子代，但是要确保基因不重复，如果出现重复(实验证明有较大的概率出现重复)，那么从题库中挑选一道与重复题的题型相同、分值相同且包含的知识点相同的试题遗传给子代.所有的遗传都要保证基因在染色体上的相对位置不变.

#### 1.5 变异算子的改进

基因变异的出现增加了种群的多样性.在本系统中，每个个体的每个基因都有变异的机会，如果随机概率小于变异概率，那么基因就可以突变.突变基因的原则为：与原题的同题型、同分数且同知识点的试题.有研究表明，对于变异概率的选择，在0.1-0.001之间最佳，本例中选取了0.085作为变异概率.

#### 1.6 组卷规则
组卷规则是初始化种群的依赖。组卷规则由用户指定，规定了用户期望的试卷的条件：试卷总分、包含的题型与数量、期望难度系数、期望覆盖的知识点。在本例中将组卷规则封装为一个JavaBean

### 2. java实现
#### 2.1 试卷个体

个体，即试卷.本例中将试卷个体抽象成一个JavaBean，其有id，适应度、知识点覆盖率、难度系数、总分、以及个体包含的试题集合这6个属性，以及计算知识点覆盖率和适应度这几个方法.在计算适应度的时候，知识点权重为0.20，难度系数权重为0.80.
```
/**
 * 计算试卷总分
 *
 * @return
 */
public double getTotalScore() {
    if (totalScore == 0) {
        double total = 0;
        for (QuestionBean question : questionList) {
            total += question.getScore();
        }
        totalScore = total;
    }
    return totalScore;
}

/**
 * 计算试卷个体难度系数 计算公式： 每题难度*分数求和除总分
 *
 * @return
 */
public double getDifficulty() {
    if (difficulty == 0) {
        double _difficulty = 0;
        for (QuestionBean question : questionList) {
            _difficulty += question.getScore() * question.getDifficulty();
        }
        difficulty = _difficulty / getTotalScore();
    }
    return difficulty;
}
 /**
     * 计算知识点覆盖率 公式为：个体包含的知识点/期望包含的知识点
     *
     * @param rule
     */
    public void setKpCoverage(RuleBean rule) {
        if (kPCoverage == 0) {
            Set<String> result = new HashSet<String>();
            result.addAll(rule.getPointIds());
            Set<String> another = questionList.stream().map(questionBean -> String.valueOf(questionBean.getPointId())).collect(Collectors.toSet());
            // 交集操作
            result.retainAll(another);
            kPCoverage = result.size() / rule.getPointIds().size();
        }
    }

/**
 * 计算个体适应度 公式为：f=1-(1-M/N)*f1-|EP-P|*f2
 * 其中M/N为知识点覆盖率，EP为期望难度系数，P为种群个体难度系数，f1为知识点分布的权重
 * ，f2为难度系数所占权重。当f1=0时退化为只限制试题难度系数，当f2=0时退化为只限制知识点分布
 *
 * @param rule 组卷规则
 * @param f1   知识点分布的权重
 * @param f2   难度系数的权重
 */
public void setAdaptationDegree(RuleBean rule, double f1, double f2) {
    if (adaptationDegree == 0) {
        adaptationDegree = 1 - (1 - getkPCoverage()) * f1 - Math.abs(rule.getDifficulty() - getDifficulty()) * f2;
    }
}

public boolean containsQuestion(QuestionBean question) {
    if (question == null) {
        for (int i = 0; i < questionList.size(); i++) {
            if (questionList.get(i) == null) {
                return true;
            }
        }
    } else {
        for (QuestionBean aQuestionList : questionList) {
            if (aQuestionList != null) {
                if (aQuestionList.equals(question)) {
                    return true;
                }
            }
        }
    }
    return false;
}
```
#### 2.2 种群初始化

种群初始化。将种群抽象为一个Java类Population，其有初始化种群、获取最优个体的方法，关键代码如下
```
/**
 * 初始种群
 *
 * @param populationSize 种群规模
 * @param initFlag       初始化标志 true-初始化
 * @param rule           规则bean
 */
public Population(int populationSize, boolean initFlag, RuleBean rule) {
    papers = new Paper[populationSize];
    if (initFlag) {
        Paper paper;
        Random random = new Random();
        for (int i = 0; i < populationSize; i++) {
            paper = new Paper();
            paper.setId(i + 1);
            while (paper.getTotalScore() != rule.getTotalMark()) {
                paper.getQuestionList().clear();
                String idString = rule.getPointIds().toString();
                // 单选题
                if (rule.getSingleNum() > 0) {
                    generateQuestion(1, random, rule.getSingleNum(), rule.getSingleScore(), idString,
                            "单选题数量不够，组卷失败", paper);
                }
                // 填空题
                if (rule.getCompleteNum() > 0) {
                    generateQuestion(2, random, rule.getCompleteNum(), rule.getCompleteScore(), idString,
                            "填空题数量不够，组卷失败", paper);
                }
                // 主观题
                if (rule.getSubjectiveNum() > 0) {
                    generateQuestion(3, random, rule.getSubjectiveNum(), rule.getSubjectiveScore(), idString,
                            "主观题数量不够，组卷失败", paper);
                }
            }
            // 计算试卷知识点覆盖率
            paper.setKpCoverage(rule);
            // 计算试卷适应度
            paper.setAdaptationDegree(rule, Global.KP_WEIGHT, Global.DIFFCULTY_WEIGHt);
            papers[i] = paper;
        }
    }
}

private void generateQuestion(int type, Random random, int qustionNum, double score, String idString,
                              String errorMsg, Paper paper) {
    QuestionBean[] singleArray = QuestionService.getQuestionArray(type, idString
            .substring(1, idString.indexOf("]")));
    if (singleArray.length < qustionNum) {
        log.error(errorMsg);
        return;
    }
    QuestionBean tmpQuestion;
    for (int j = 0; j < qustionNum; j++) {
        int index = random.nextInt(singleArray.length - j);
        // 初始化分数
        singleArray[index].setScore(score);
        paper.addQuestion(singleArray[index]);
        // 保证不会重复添加试题
        tmpQuestion = singleArray[singleArray.length - j - 1];
        singleArray[singleArray.length - j - 1] = singleArray[index];
        singleArray[index] = tmpQuestion;
    }
}
```
#### 2.3 选择算子与交叉算子的实现
选择算子的实现：
```
/**
 * 选择算子
 *
 * @param population
 */
private static Paper select(Population population) {
    Population pop = new Population(tournamentSize);
    for (int i = 0; i < tournamentSize; i++) {
        pop.setPaper(i, population.getPaper((int) (Math.random() * population.getLength())));
    }
    return pop.getFitness();
}
```

交叉算子的实现.本系统实现的算子为两点交叉，在算法的实现过程中需要保证子代中不出现相同的试题.关键代码如下：
```
/**
 * 交叉算子
 *
 * @param parent1
 * @param parent2
 * @return
 */
public static Paper crossover(Paper parent1, Paper parent2, RuleBean rule) {
    Paper child = new Paper(parent1.getQuestionSize());
    int s1 = (int) (Math.random() * parent1.getQuestionSize());
    int s2 = (int) (Math.random() * parent1.getQuestionSize());

    // parent1的startPos endPos之间的序列，会被遗传到下一代
    int startPos = s1 < s2 ? s1 : s2;
    int endPos = s1 > s2 ? s1 : s2;
    for (int i = startPos; i < endPos; i++) {
        child.saveQuestion(i, parent1.getQuestion(i));
    }

    // 继承parent2中未被child继承的question
    // 防止出现重复的元素
    String idString = rule.getPointIds().toString();
    for (int i = 0; i < startPos; i++) {
        if (!child.containsQuestion(parent2.getQuestion(i))) {
            child.saveQuestion(i, parent2.getQuestion(i));
        } else {
            int type = getTypeByIndex(i, rule);
            QuestionBean[] singleArray = QuestionService.getQuestionArray(type, idString.substring(1, idString
                    .indexOf("]")));
            child.saveQuestion(i, singleArray[(int) (Math.random() * singleArray.length)]);
        }
    }
    for (int i = endPos; i < parent2.getQuestionSize(); i++) {
        if (!child.containsQuestion(parent2.getQuestion(i))) {
            child.saveQuestion(i, parent2.getQuestion(i));
        } else {
            int type = getTypeByIndex(i, rule);
            QuestionBean[] singleArray = QuestionService.getQuestionArray(type, idString.substring(1, idString
                    .indexOf("]")));
            child.saveQuestion(i, singleArray[(int) (Math.random() * singleArray.length)]);
        }
    }

    return child;
}
```
#### 2.4 变异算子的实现
本系统中变异概率为0.085，对种群的每个个体的每个基因都有变异机会.变异策略为：在(0,1)之间产生一个随机数，如果小于变异概率，那么该基因突变.关键代码如下：
```
/**
 * 突变算子 每个个体的每个基因都有可能突变
 *
 * @param paper
 */
public static void mutate(Paper paper) {
    QuestionBean tmpQuestion;
    List<QuestionBean> list;
    int index;
    for (int i = 0; i < paper.getQuestionSize(); i++) {
        if (Math.random() < mutationRate) {
            // 进行突变，第i道
            tmpQuestion = paper.getQuestion(i);
            // 从题库中获取和变异的题目类型一样分数相同的题目（不包含变异题目）
            list = QuestionService.getQuestionListWithOutSId(tmpQuestion);
            if (list.size() > 0) {
                // 随机获取一道
                index = (int) (Math.random() * list.size());
                // 设置分数
                list.get(index).setScore(tmpQuestion.getScore());
                paper.saveQuestion(i, list.get(index));
            }
        }
    }
}
```

#### 2.5 进化的整体流程
本系统中采用精英策略，每次进化都保留上一代最优秀个体.这样就能避免种群进化方向发生变化，出现适应度倒退的情况.关键代码如下：
```
// 进化种群
public static Population evolvePopulation(Population pop, RuleBean rule) {
    Population newPopulation = new Population(pop.getLength());
    int elitismOffset;
    // 精英主义
    if (elitism) {
        elitismOffset = 1;
        // 保留上一代最优秀个体
        Paper fitness = pop.getFitness();
        fitness.setId(0);
        newPopulation.setPaper(0, fitness);
    }
    // 种群交叉操作，从当前的种群pop 来 创建下一代种群 newPopulation
    for (int i = elitismOffset; i < newPopulation.getLength(); i++) {
        // 较优选择parent
        Paper parent1 = select(pop);
        Paper parent2 = select(pop);
        while (parent2.getId() == parent1.getId()) {
            parent2 = select(pop);
        }
        // 交叉
        Paper child = crossover(parent1, parent2, rule);
        child.setId(i);
        newPopulation.setPaper(i, child);
    }
    // 种群变异操作
    Paper tmpPaper;
    for (int i = elitismOffset; i < newPopulation.getLength(); i++) {
        tmpPaper = newPopulation.getPaper(i);
        mutate(tmpPaper);
        // 计算知识点覆盖率与适应度
        tmpPaper.setKpCoverage(rule);
        tmpPaper.setAdaptationDegree(rule, Global.KP_WEIGHT, Global.DIFFCULTY_WEIGHt);
    }
    return newPopulation;
}
```

### 3. 测试结果

组卷规则为：期望试卷难度系数0.82，共100分，20道选择题，2分一道，10道填空题，2分一道，4道主观题，10分一道，要求囊括6个知识点.
外在的条件为：题库试题总量为10950，期望适应度值为0.98，种群最多迭代100次.
测试代码如下：
```
/**
 * 组卷过程
 *
 * @param rule
 * @return
 */
public static Paper generatePaper(RuleBean rule) {
    Paper resultPaper = null;
    // 迭代计数器
    int count = 0;
    int runCount = 100;
    // 适应度期望值z
    double expand = 0.98;
    if (rule != null) {
        // 初始化种群
        Population population = new Population(20, true, rule);
        System.out.println("初次适应度  " + population.getFitness().getAdaptationDegree());
        while (count < runCount && population.getFitness().getAdaptationDegree() < expand) {
            count++;
            population = GA.evolvePopulation(population, rule);
            System.out.println("第 " + count + " 次进化，适应度为： " + population.getFitness().getAdaptationDegree());
        }
        System.out.println("进化次数： " + count);
        System.out.println(population.getFitness().getAdaptationDegree());
        resultPaper = population.getFitness();
    }
    return resultPaper;
}
```

测试结果如下：

![](http://7xryc7.com1.z0.glb.clouddn.com/rs2.png)

可以看到改进后的遗传算法具有较好的表现

------------
## 参考资料

1. [实例讲解遗传算法——基于遗传算法的自动组卷系统【理论篇】](http://www.cnblogs.com/artwl/archive/2011/05/19/2051556.html)
2. [遗传算法-入门(demo java)](http://www.acmerblog.com/genetic-algorithm-for-beginners-java-demo-5755.html)


本文中的完整代码可在[github](https://github.com/jslixiaolin/GADemo)上下载.
你可以通过`jslinxiaoli@foxmail.com`联系我.
欢迎在[github](https://github.com/jslixiaolin)或者[知乎](http://www.zhihu.com/people/li-xiao-lin-4)上关注我 ^_^.
也可以访问个人博客: https://github.com/jslixiaolin/blog


