# 轨道积分（Orbit Integration）



## 基本数学原理（数值积分）

解决的问题：通过数值法求解出状态量（代求参数）在给定时间点的数值

基本思想：通过给定实时间步长，近似线性地进行求解，基本思路是确定每个步长下的增量（增量函数）为：
$$
X^{'}=F(X,t)\\
w_{i+1}=w_{i}+h\phi(w_{i},t,h)
$$
其中，增量函数决定了该近似方法的好坏，一般用全局误差的阶数来衡量方法的好坏。

**单步法**

核心思路只依赖过去单个时刻的值，其中R-K法是一族求解器的统称

**多步法**

分为显示和隐式两种方法，通常结合两种方法进行求解，即预报-校正过程：

预测：利用显示多步法进行求解

校正：利用显示多步法求得的预报值，再利用隐式多步法进行求解

## 导航卫星轨道积分

除了需要求解出状态量（位置、速度）在后续时间点的值，还需要求出偏导数矩阵的值

这里偏导数矩阵给出了改正量之间的关系

因此待估状态数为：6（位置速度）+6*（6+p）（位置速度对初始时刻状态参数的偏导数）

令状态量X为
$$
X=[X,V,P]
$$
其偏导数为：
$$
X^{'}=[V,a,0]
$$
在初始值上线性泰勒展开为
$$
A(t) =
\left( \begin{array} \\
0 & I & 0 \\
\frac{\partial{a}}{\partial{X}} & \frac{\partial{a}}{\partial{V}} & \frac{\partial{a}}{\partial{P}} \\
\end{array} \right)
$$
待估的状态转移矩阵为
$$
dX_{t}=\phi{(t,t_{0})}dX_{0}\\
\phi{(t,t_{0})}=
\left( \begin{array} \\
\frac{\partial{X}}{\partial{X_{0}}} & \frac{\partial{X}}{\partial{V_{0}}} & \frac{\partial{X}}{\partial{P_{0}}} \\
\frac{\partial{V}}{\partial{X_{0}}} & \frac{\partial{V}}{\partial{V_{0}}} & \frac{\partial{V}}{\partial{P_{0}}} \\
0 & 0 & I \\
\end{array} \right)
$$
状态转移矩阵的偏导函数值就可以通过A(t)和Φ(t,t0)得到：
$$
\phi{(t,t_{0})}^{'}=A(t)\phi{(t,t_{0})}\\
\phi{(t,t_{0})}^{'}=
\left( \begin{array} \\
\frac{\partial{V}}{\partial{X_{0}}} & \frac{\partial{V}}{\partial{V_{0}}} & \frac{\partial{V}}{\partial{P_{0}}} \\
\frac{\partial{a}}{\partial{X}}\frac{\partial{X}} {\partial{X_{0}}}+\frac{\partial{a}}{\partial{V}}\frac{\partial{V}} {\partial{X_{0}}} & \frac{\partial{a}}{\partial{X}}\frac{\partial{X}} {\partial{V_{0}}}+\frac{\partial{a}}{\partial{V}}\frac{\partial{V}} {\partial{V_{0}}} & \frac{\partial{a}}{\partial{X}}\frac{\partial{X}} {\partial{P_{0}}}+\frac{\partial{a}}{\partial{V}}\frac{\partial{V}} {\partial{P_{0}}}+\frac{\partial{a}}{\partial{P}} \\
0 & 0 & I \\
\end{array} \right)
$$
因此本质上来说，待求解的状态量包括：X,V,以及Φ。

而计算偏导函数重点在于求出，加速度，加速度**对当前状态量**的偏导数

## 轨道比较（ORBDIF）

这里本质上需要求解的是两个轨道在对应时间采样点上的三维偏差

ORBDIF文件分为三个部分：

* APRI记录了初始偏差**（Unit：m）**，即直接将sp3坐标(TRS坐标系下)旋转到CRS坐标系下，和ORB文件中坐标进行做差。同时因为这个旋转中涉及到一些参数值（地心点，地心运动参数值），所以会在后续过程中进行估计（STRD 表征估计模式，S表示Scale，T表示平移，RD表示旋转相关？？）
* 后续ACR部分**（Unit：mm）**。这里是计算了旋转参数（STRD）的值，将其改正后，再将偏差旋转到对应的ACR（星固坐标系）坐标系下的值

* ACR偏差值的统计结果**（Unit：mm)**：
  * max_ACR表示最大值
  
  * min_ACR表示最小值
  
  * fit_ACR记录了RMS值
  
  * RMS值记录了1D的RMS值
  
  * $$
    RMS_{1D}=\frac{RMS_{3D}}{\sqrt{3}}
    $$
  
  * RMSmean记录了所有卫星的平均1D RMS值
  
  * Unit weight sigm0表示验后单位权中误差（该值反应观测方程的中误差大小？因为观测方程都设置为了单位权）所以应该大概是反应了所有卫星的一种平均差异（接近RMS Mean，不过评判还是看RMS mean为主）

**ORB大致精度**

GPS浮点解 三次迭代后3-4cm

GPS固定解 1-2cm

单次SRIF（不转移状态矩阵）+质量控制能到4 cm

戴SRIF浮点解精度 46mm

现在单次SRIF精度 58mm （如何提升1cm。。。



# 卫星姿态模型

