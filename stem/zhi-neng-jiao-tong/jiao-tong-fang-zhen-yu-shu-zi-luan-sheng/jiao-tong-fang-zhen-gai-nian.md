# 交通仿真概念

交通仿真指应用系统仿真技术来研究交通行为，是对交通运动随时间和空间的变化进行跟踪描述的技术。

交通仿真主要分为宏观仿真和微观仿真。

宏观仿真下，交通流被看作连续流，不考虑个别车辆的运动，而是从统计意义上成批的考虑车辆的运动。宏观仿真过程通过速度--流量曲线控制交通流的运行，主要参数有**路段速度、密度**和**交通流量**等。宏观仿真对计算机资源要求较低，主要用于研究分析**交通基础设施的新建与扩建、宏观管理措施及交通发展政策**等。

微观仿真采用基于单个车辆行为的微观交通流模型，主要从车辆的驾驶行为、车道组的设置及交通设施的配置等各微观细节来分析交通系统的特征或者优化其性能。微观仿真模型的参数主要有车辆的当前速度、加速度和位置等，因此能够细致的反映出车辆的跟驰、超车及车道变换等微观行为。微观仿真对计算机资源要求较高，且受仿真车辆数量和路网规模的限制，难以在大规模网络上运行，主要用于研究**交通流与局部的道路设施的相互影响**，也用于**交通控制仿真**。

其优点在于：

* 经济性
* 安全性
* 可重复性
* 易用性
* 可控制性
* 可扩展性
