# 初步入门 #
> ### 用Cplex的简要步骤 ###
> * 1 定义一个环境目标
> * 2 在环境中定义一个模型目标
> * 3 向环境中添加变量
> * 4 向模型中添加约束和目标函数
> * 5 定义一个Cplex目标
> * 6 将模型提取到CPLEX中
> * 7 求解模型
> * 8 输出模型求解过程中信息
> * 9 关闭环境
******
## 定义环境和模型 ##
使用以下语句来定义环境和建立一个模型对象，<font color=RoyalBlue>IloEnv</font>和<font color=RoyalBlue>IloModel</font>是环境和模型中的一个类
```C++
IloEnv myenv;
IloModel mymodel(myenv);
```

## 定义模型中变量 ##
共用两种方法来定义变量：
> 对于变量数较少的情况，使用类IloNumVar
> ```C++
> IloNumVar myvar(env, lower, upper, type);
> ```
> * env是定义的环境变量
> * lower是下界，upper是上界；正无穷用IloInfinity，负无穷用-IloInfinity
> * type是数据类型，包括ILOINT(整型)，ILOFLOAT(浮点数)，ILOBOOL(0/1)，如果不设置参数默认类型为浮点数
******
> 对于变量数较多的情况时，使用类IloNumVarArray
> ```C++
> IloNumVarArray myvar(env, n, lower, upper, type);
> ```
>  * 此处类似于C++中的数组，定义了一个包含n个变量的数组。对于每个变量可以通过下标索引来设置myvar[i]\(0<=i<=n-1)，每个变量的上下界用函数setBounds
> ```C++
> myvar[i].setBounds(lower, upper);
> ```

## 加入约束 ##
添加约束的代码如下：
 ```C++
 model.add(constraint);
 ```
### *对于单一约束可以用：*
```C++
model.add(IloRange(env, lower, expr, upper));
```
其中lower是表达式下界；upper是表达式上界；expr是表达式例如：2*myvar[0]+myvar[1]-4*myvar[1]，
约束为$2x_1+x_2-4x_3<=6$  
可以使用两种形式的定义：
```C++
model.add(IloRange(env, 6, 2*x[0]+x[1]-4*x[1], IloInfinity));
model.add( 6 <= 2*x[0]+x[1]-4*x[1]);
```
### *对于多个约束而言：*
利用循环语句来添加约束，在线性规划中$Ax=b$，
```C++
for(int i = 0; i < m; ++i)
{
    IloExpr myexpr(myenv);//定义空的约束表达式
    for(int j = 0; j < n; ++j)
    {
        myexpr += A[i][j] * x[j];//将每个约束加入到表达式中
    }
    mymodel.add(myenv, b[i], myexpr, b[i]); //将约束加入模型中
    myexpr.end(); //清空当前的约束表达式
}
```
或直接定义空的约束数组：
```C++
IloRangeArray mycon(env);
```
mycon包含的类型是IloRange，当约束都有相同的上下界时：
```C++
mycon.add(n, IloRange(env, lower, upper));
```
此时mycon中的变量没有对应的系数，需要添加系数：
```C++
mycon[i].setLinearCoef(var, coef);
```

## 加入目标 ##
使用model.add(objective)语句，IloMinimize和IloMaximize是最小化和最大化
```C++
model.add(IloMinimize(env, expr));
```

## 求解模型 ##
IloCplex是建立Cplex目标，extract将模型提取到Cplex中，solve是求解模型
```C++
IloCplex mycplex(env);
mycplex.extract(model);
mycplex.solve();
```
求解模型会调用Cplex求解器，求解的结果形式会保存为IloBool类型
$$返回结果=
\begin{cases}
IloTrue& \text{原问题可以找到可行解}\\
IloFlase& \text{其他情况}
\end{cases}$$

## 导出模型解 ##
*导出求解状态*
> ```C++
> cplex.getStatus();
> ```
> * 最优解被找到/ Optimal solution found
> * 可行解但不一定是最优解/ Feasible but not necessarily optimal solution found
> * 模型没有不可行解/ Model known to be infeasible
> * 不能明确是否有可行解存在/ Unknown whether any feasible solutions exist

*导出决策变量*
> ```C++
> cplex.getValue(var);
> ```

*导出目标函数值*
> ```C++
> cplex.getObjValue();
> ```

## 关闭模型 ##
当我们在写程序时，如果涉及多个模型，那么将当前模型从cplex对象中清除很有必要，同时要关闭环境变量
> ```C++
> cplex.clear();
> env.end();
> ```