---
layout:     post
title:      Capacitated Facility Location Problem
subtitle:   
date:       2018-12-20
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - 算法分析与设计
    - np问题
---
# 题目概述
![image](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/%E6%88%AA%E5%9B%BE/%E9%A2%98%E7%9B%AE%E8%A6%81%E6%B1%82.png?raw=true)

# 使用语言和运行环境
Python2.7.14 + win10

# 处理输入数据
输入数据的格式如下：
![image](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/%E6%88%AA%E5%9B%BE/%E8%BE%93%E5%85%A5%E6%95%B0%E6%8D%AE%E6%A0%BC%E5%BC%8F.png?raw=true)
一开始我以为demand和assignment cost是固定10个数据一行，所以我是一行一行读取，然后跑数据时发现后面几个测试数据的格式不是一行10个。所以我只能改成先把全部数据存在list中，然后再逐个分配。

代码如下：

```
# 保存各种数据的全局变量
FACILITY_NUM = 0  # 工厂数量
CUSTOMER_NUM = 0  # 顾客数量
OPENING_COST = []  # 工厂开放开销
CAPACITY = []  # 工厂容量
DEMAND = []  # 每个顾客的需求
ASSIGNMENT_COST = []  # 每个顾客分配给工厂时候的开销
OPEN_STATUS = []  # 每个工厂开闭状态，1表示开，0表示关


# 读取文件，加载各种数据
def load(file_path):
    with open(file_path, 'r') as f:
        # 声明全局变量
        global FACILITY_NUM
        global CUSTOMER_NUM
        global OPENING_COST
        global CAPACITY
        global DEMAND
        global ASSIGNMENT_COST
        global OPEN_STATUS

        # 获取数据
        FACILITY_NUM, CUSTOMER_NUM = f.readline().strip("\n").split()
        FACILITY_NUM = int(FACILITY_NUM)
        CUSTOMER_NUM = int(CUSTOMER_NUM)

        OPEN_STATUS = [0] * FACILITY_NUM

        for i in range(FACILITY_NUM):
            line = f.readline().strip("\n").split()
            CAPACITY.append(int(line[0]))
            OPENING_COST.append(int(float(line[1])))

        data_set = []
        source_in_line = f.readlines()
        for line in source_in_line:
            temp1 = line.strip("\n")
            temp2 = temp1.split()
            data_set.append(temp2)

        data = []
        for i in data_set:
            for j in i:
                data.append(j)

        for i in range(CUSTOMER_NUM):
            DEMAND.append(int(float(data[i])))

        begin = CUSTOMER_NUM
        for i in range(CUSTOMER_NUM):
            temp = []  # 保存每个顾客分配给每个工厂的开销
            for j in range(FACILITY_NUM):
                temp.append(int(float(data[begin])))
                begin += 1
            ASSIGNMENT_COST.append(temp)
```

# 方法一：贪心算法
## 思路
首先从前往后遍历每个用户，对每个用户，遍历所有工厂，根据工厂当前容量capacity和用户的需求demand比较，如果capacity >= demand则把该工厂加入到一个可用工厂集合中。然后取出该可用工厂集合中opencost + assginment cost最小的工厂作为当前用户分配到的工厂。重复此操作直到遍历完所有用户，并把每次遍历的总开销total_cost叠加得到最终总开销。

## 代码
代码如下：

```
# 贪心算法
def greedy(output_file, input_file):
    begin_time = time.clock()  # 记录开始时间

    global CAPACITY

    capacity = CAPACITY[:]
    open_status = [0] * FACILITY_NUM  # 每个工厂的开闭状态
    assign_cost = 0  # 客户分配给工厂的总开销
    open_cost = 0  # 工厂开放总开销
    total_cost = 0  # 总开销
    assignment = [-1] * CUSTOMER_NUM  # 每个顾客分配到工厂的索引

    for cus in range(CUSTOMER_NUM):
        facility_index = []  # 可以被选中的工厂（即容量足够当前顾客使用）
        for i in range(FACILITY_NUM):
            if capacity[i] >= DEMAND[cus]:
                facility_index.append(i)

        # 获取当前顾客分配给每个工厂时的开销
        cur_assignment_cost = ASSIGNMENT_COST[cus]

        # 计算对于当前顾客每个工厂的分配开销和开启开销总和
        cur_total_cost = [sum(x) for x in zip(cur_assignment_cost, OPENING_COST)]

        # 选择可用的最小的cur_total_cost的工厂
        selected_index = facility_index[0]
        for i in facility_index:
            if cur_total_cost[i] < cur_total_cost[selected_index]:
                selected_index = i

        # 设置选中时的状态
        open_status[selected_index] = 1
        capacity[selected_index] = capacity[selected_index] - DEMAND[cus]
        assignment[cus] = selected_index

        # 累加分配开销
        assign_cost += cur_assignment_cost[selected_index]

    # 累加工厂开放开销
    for i in range(FACILITY_NUM):
        open_cost += open_status[i] * OPENING_COST[i]

    # 计算总开销
    total_cost = open_cost + assign_cost

    # 记录结束时间
    end_time = time.clock()
    # 计算运行时间
    used_time = end_time - begin_time

    # 输出结果
    # 输出结果到csv文件
    with open('sum.csv', 'a+') as csvfile:
        spamwriter = csv.writer(csvfile, dialect='excel')
        spamwriter.writerow([input_file[10:], total_cost, used_time])

    # 输出结果到txt文件
    with open(output_file, "a") as f:
        start = time.clock()  # 记录开始时间
        # 输出输入样例
        f.write("%s \n" % input_file)
        # 输出总开销
        f.write("Result: %d \n" % total_cost)
        # 输出工厂开闭状态
        f.write("Status of facilities: {} \n".format(open_status))
        # 输出每个顾客分配到哪个工厂
        f.write("The assignment of customers to facilities: {} \n".format(assignment))
        # 输出运行时间
        f.write("Time used: %f \n" % float(used_time))
        # 输出换行符
        f.write("\n")

    # 返回最终结果
    return total_cost
```

# 方法二：基于贪心的局部搜索算法
我们首先来看看局部搜索算法的定义：
![image](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/%E6%88%AA%E5%9B%BE/%E5%B1%80%E9%83%A8%E6%90%9C%E7%B4%A2%E7%AE%97%E6%B3%95%E5%AE%9A%E4%B9%89.png?raw=true)

因此，我们要实现一个局部搜索算法，我们需要做到以下三个步骤：
1. 确定初始解
2. 确定邻域怎么获取
3. 确定邻域怎么到达

## 获取局部搜索算法的初始解
我将上面用到的贪心算法作为局部搜索算法的初始解，定义了初始解的类的代码如下：

```
# 生成一个基于贪心算法的局部搜索算法初始解
class Solution:
    assignment = []  # 每个顾客分配到哪个工厂
    capacity = []  # 每个工厂剩余容量
    open_status = []  # 每个工厂开放状态
    assgin_cost = 0  # 分配总开销
    open_cost = 0  # 开放总开销
    total_cost = 0  # 总开销

    # 构造函数
    def __init__(self):
        # 初始化变量
        self.assignment = [-1] * CUSTOMER_NUM
        self.capacity = CAPACITY[:]
        self.open_status = [0] * FACILITY_NUM

        for cus in range(CUSTOMER_NUM):
            facility_index = []  # 可被选中的工厂索引集合
            for i in range(FACILITY_NUM):
                if self.capacity[i] >= DEMAND[cus]:
                    facility_index.append(i)

            # 获取当前顾客分配给每个工厂时的开销
            cur_assignment_cost = ASSIGNMENT_COST[cus]

            # 计算对于当前顾客每个工厂的分配开销和开启开销总和
            cur_total_cost = [sum(x) for x in zip(cur_assignment_cost, OPENING_COST)]

            # 选择可用的最小的cur_total_cost的工厂
            selected_index = facility_index[0]
            for i in facility_index:
                if cur_total_cost[i] < cur_total_cost[selected_index]:
                    selected_index = i

            self.capacity[selected_index] = self.capacity[selected_index] - DEMAND[cus]
            self.assignment[cus] = selected_index

            self.assgin_cost += ASSIGNMENT_COST[cus][selected_index]
            self.open_status[selected_index] = 1

        for i in range(FACILITY_NUM):
            self.open_cost += self.open_status[i] * OPENING_COST[i]

        self.total_cost = self.assgin_cost + self.open_cost

    def result(self):
        return self.total_cost
```
可以看到算法和我们上面讲到的贪心算法一样。

## 确定邻域和邻域到达方式
在这里我通过交换两个customer的工厂来得到一个邻域。这样做的好处是不用考虑交换的工厂是否开闭（因为已经有customer分配所以两个都是开的），唯一需要注意的便是在交换的时候要判断两个工厂当前的capacity是否满足交换条件。然后我选择随机选取作为邻域的到达方式，通过迭代1000次来得到最终的结果。

具体代码如下：

```
# 计算在随机交换两个customer对应的工厂后的total_cost
def random_swap(solution, iter_times):
    local_assignment = solution.assignment[:]
    local_capacity = solution.capacity[:]

    for i in range(iter_times):
        customer1 = random.randint(0, CUSTOMER_NUM - 1)
        customer2 = random.randint(0, CUSTOMER_NUM - 1)
        while customer1 == customer2:
            customer2 = random.randint(0, CUSTOMER_NUM - 1)

        factory1 = local_assignment[customer1]
        factory2 = local_assignment[customer2]
        assignment_cost1 = ASSIGNMENT_COST[customer1]
        assignment_cost2 = ASSIGNMENT_COST[customer2]
        capacity1 = local_capacity[factory1]
        capacity2 = local_capacity[factory2]

        if(capacity1 >= abs(DEMAND[customer1] - DEMAND[customer2])
                and capacity2 >= abs(DEMAND[customer1] - DEMAND[customer2])):
            result = (solution.total_cost + assignment_cost1[factory2] + assignment_cost2[factory1]
                    - assignment_cost1[factory1] - assignment_cost2[factory2])
            solution.assignment[customer1] = factory2
            solution.assignment[customer2] = factory1
            solution.total_cost = result
            break


# 局部搜索函数
def local_search(output_file, input_file):
    begin_time = time.clock()  # 记录开始时间
    init_solution = Solution()  # 生产一个随机解

    iter_times = 1000  # 迭代次数

    random_swap(init_solution, iter_times)

    # 记录结束时间
    end_time = time.clock()
    # 计算运行时间
    used_time = end_time - begin_time

    # 输出结果
    # 输出结果到csv文件
    with open('sum.csv', 'a+') as csvfile:
        spamwriter = csv.writer(csvfile, dialect='excel')
        spamwriter.writerow([input_file[10:], init_solution.total_cost, used_time])

    # 输出结果到txt文件
    with open(output_file, "a") as f:

        # 输出输入样例
        f.write("%s \n" % input_file)
        # 输出总开销
        f.write("Result: %d \n" % init_solution.total_cost)
        # 输出工厂开闭状态
        f.write("Status of facilities: {} \n".format(init_solution.open_status))
        # 输出每个顾客分配到哪个工厂
        f.write("The assignment of customers to facilities: {} \n".format(init_solution.assignment))
        # 输出运行时间
        f.write("Time used: %f \n" % float(used_time))
        # 输出换行符
        f.write("\n")

    return init_solution.total_cost
```

# 输出结果
## Result table
![image](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/%E6%88%AA%E5%9B%BE/p1.png?raw=true)
![image](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/%E6%88%AA%E5%9B%BE/p2.png?raw=true)
![image](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/%E6%88%AA%E5%9B%BE/p3.png?raw=true)

csv文件：[sum.csv](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/sum.csv)

## 详细输出结果
贪心算法详细输出结果：[greedy.txt](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/greedy.txt)

局部搜索算法详细输出结果：[localsearch.txt](https://github.com/Richbabe/CapacitatedFacilityLocationProblem/blob/master/localsearch.txt)

# 分析与总结
从result table中可以看到，贪心算法在result和运行时间上都要优于局部搜索算法。我认为可能是由于局部搜索算法的邻域到达方式过于随机导致的。因此如果要达到比贪心算法更好的效果，可以尝试以下改进：
* 使用局部搜索算法，每次将一组邻域放到一个集合中，每次迭代取集合中结果最好的领域作为本次迭代的解。
* 使用局部搜索算法，将邻域的获取方式换成将一个工厂的开闭状态flip（即本来开的工厂关闭，本来关闭的工厂开），不过这样会带来繁琐的计算。
* 使用模拟退火算法代替局部搜索算法，避免陷入局部最优解。

# 结语
本博客的代码和资源均可在我的[github](https://github.com/Richbabe/CapacitatedFacilityLocationProblem)上下载，别忘了点颗Star哟！