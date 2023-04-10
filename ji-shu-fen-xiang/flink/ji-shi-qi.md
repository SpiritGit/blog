# 计时器

### 场景假设：为什么需要计时器

**现有以下场景：**

车辆行驶过程中每隔5分钟发送一次`GPS`定位数据，但受环境或网络影响，数据时有丢失。现需提取出其在高速公路以内部分，并将其进行合并，合并原则为：一次通行所产生的`GPS`数据合并为一条轨迹。

**关键点**：

* 1 提取出高速公路内的`GPS`数据。（该关键点不是本篇博客的主角，参见通用算法篇：黑白位图算法）
* 2 将不同出行的`GPS`数据切段。

对于上述关键点2，可采取的解决方式为：当车辆连续发送`n`（可取3）次不在高速的记录，或距离最后一次发送`GPS`数据已过去`t`（可取30分钟）时间，则进行截断，并将前续的高速内`GPS`数据进行合并。

上述提到的`t`时间，则可借助`flink`提供的计时器进行实现。

### 计时器简介

`Timer`（计时器）是`flink streaming api`提供的用于感知并利用处理时间/事件时间变化的机制。`Ververica blog`上给出的描述如下：

> Timers are what make Flink streaming applications reactive and adaptable to processing and event time changes.

对于普通用户来说，最常见的显式利用`Timer`的地方就是`KeyedProcessFunction`和`CoProcessFunction`（参见ProcessFunctionAPI）。我们在其`processElement()`方法中注册`Timer`，然后覆写其`onTimer()`方法作为`Timer`触发时的回调逻辑。

#### 注册计时器

首先根据业务定义出要注册的时间戳`timer`（`timer`为`long`类型，即`unix`时间戳，当系统时间或水位线到达该时间戳是会出发`onTimer()`方法） 根据时间特征的不同，注册计时器方法如下：

* 处理时间——调用`Context.timerService().registerProcessingTimeTimer(timer)`注册
* 事件时间——调用`Context.timerService().registerEventTimeTimer(timer)`注册

#### 删除计时器

首先获取当前计时器`timer` 其次删除计时器，根据时间特征不同，

* 处理时间——调用 `ctx.timerService().deleteProcessingTimeTimer(timer)`
* 事件时间——调用`ctx.timerService().deleteEventTimeTimer(timer)`

#### 触发计时器

根据时间特征的不同，注册计时器方法如下：

* 处理时间——`onTimer()`在系统时间戳达到`Timer`设定的时间戳时自动触发
* 事件时间——`onTimer()`在`Flink`内部水印达到或超过`Timer`设定的时间戳时自动触发

### 利用`CoProcessFunction` + `Timer`解决上述问题

前续判别`GPS`数据是否在高速的步骤此处略去，得到结果为位于高速的`DataStream`和非高速的`DataStream`的`GPS`数据，由于两条数据流都会被轨迹切分所用到，因此将两条数据流合并得到`ConnectedStreams`，进一步进行`KeyBy`分组后传入`CoProcessFunction`，从而实现轨迹切分并输出完整的高速通行`GPS`轨迹，代码实现如下。

```java
public class TraceCut extends CoProcessFunction<Map, Map, Map<String, TreeSet<GPSBean>>> {
    private ValueState<TreeSet<GPSBean>> routeState; // 保存在高速的GPS数据
    private ValueState<String> outOfHighwayTimes; // 保存连续出现非高速GPS数据次数
    private ValueState<Long> timerState; // 保存计时器的时间戳

    @Override
    public void open(Configuration parameters) throws Exception {
        ValueStateDescriptor stateDescriptor1 = new ValueStateDescriptor<>("route state", GPSBean.class);
        routeState = getRuntimeContext().getState(stateDescriptor1);

        ValueStateDescriptor stateDescriptor2 = new ValueStateDescriptor<>("contiously out of hw times", String.class);
        outOfHighwayTimes = getRuntimeContext().getState(stateDescriptor2);

        ValueStateDescriptor stateDescriptor3 = new ValueStateDescriptor<>("last gps time", Types.LONG);
        timerState = getRuntimeContext().getState(stateDescriptor3);
    }

    @Override
    public void processElement1(Map onHwData, CoProcessFunction<Map, Map, Map<String, TreeSet<GPSBean>>>.Context ctx,
            Collector<Map<String, TreeSet<GPSBean>>> out) throws Exception {

        // 初始化对象
        String veh_plate = onHwData.get("veh_plate").toString();
        String recordtime = onHwData.get("recordtime").toString();
        String longitude = onHwData.get("longitude").toString();
        String latitude = onHwData.get("latitude").toString();
        String altitude = onHwData.get("altitude").toString();
        String direction = onHwData.get("direction").toString();
        String speed = onHwData.get("speed").toString();

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        sdf.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
        Date dateTime = sdf.parse(recordtime);
        long timeLong = dateTime.getTime();


        GPSBean currentObj =  new GPSBean(veh_plate, recordtime, longitude, latitude, altitude,
        direction, speed, timeLong);

        // 获取状态
        TreeSet<GPSBean> curRoute = routeState.value();
        String curTimesStr = outOfHighwayTimes.value();

        // 清除前序不在高速下的次数记录
        if (curTimesStr != null) {
            outOfHighwayTimes.clear();
        }

        if (curRoute == null) {
            System.out.println("new route ===============");
            TreeSet<GPSBean> newTreeSet = new TreeSet<>(); //新建轨迹
            newTreeSet.add(currentObj);
            routeState.update(newTreeSet);
            long timer = ctx.timerService().currentProcessingTime() + StaticParameter.outOfHwMillSeconds; //当前时间 + 指定的延迟时间
            ctx.timerService().registerProcessingTimeTimer(timer); //使用上一行中 timer 注册计时器，timer时间内无新的在高速内的数据，则到期
            timerState.update(timer);
        } else {
            System.out.println("add route +++++++++++++");
            curRoute.add(currentObj); //追加轨迹
            routeState.update(curRoute);

            long lastTimer = timerState.value();
            ctx.timerService().deleteProcessingTimeTimer(lastTimer); //删除原定时器
            long timer = ctx.timerService().currentProcessingTime() + StaticParameter.outOfHwMillSeconds;
            ctx.timerService().registerProcessingTimeTimer(timer);// 新设置30分钟定时器
            timerState.update(timer);
        } 
    }

    @Override
    public void processElement2(Map notOnHwData, CoProcessFunction<Map, Map, Map<String, TreeSet<GPSBean>>>.Context ctx,
            Collector<Map<String, TreeSet<GPSBean>>> out) throws Exception {

        String curTimesStr = outOfHighwayTimes.value();
        TreeSet<GPSBean> curRoute = routeState.value();

        int curTimes; 

        // 若有前序在高速上的记录，则 连续高速下gps次数 + 1， 否则不保存状态
        if (curRoute != null) {
            //连续出现高速下gps次数 + 1
            if (curTimesStr  == null) {
                curTimes = 1;
            } else {
                curTimes = Integer.valueOf(curTimesStr) + 1;
            }

            // 达到轨迹切分阈值（连续出现高速下gps n 次）
            if ( curTimes == StaticParameter.timesForCut ) {

                System.out.println("n times out of hw");

                Map wholeRoute = new HashMap<>();
                wholeRoute.put("jshw_trace:",curRoute);

                System.out.println("size: " + curRoute.size());
                System.out.println(wholeRoute);

                out.collect(wholeRoute);

                String vpn = curRoute.first().getVeh_plate();
                JedisCluster jedisCluster = FlinkRedisConn.getJedisCluster(); //集群
                jedisCluster.del("taisl:lkyw:gps:" + vpn); //删除redis实时轨迹
                jedisCluster.close();

                routeState.clear(); //清除轨迹状态
                outOfHighwayTimes.clear();
                timerState.clear();
            } else {
                System.out.println("out of hw times: " + String.valueOf(curTimes));
                outOfHighwayTimes.update(String.valueOf(curTimes));

                long lastTimer = timerState.value();
                ctx.timerService().deleteProcessingTimeTimer(lastTimer); //删除原定时器
                long timer = ctx.timerService().currentProcessingTime() + StaticParameter.outOfHwMillSeconds;
                ctx.timerService().registerProcessingTimeTimer(timer);// 新设置30分钟定时器
                timerState.update(timer);
            }
        }
    }

    @Override
    public void onTimer(long timestamp, OnTimerContext ctx,
            Collector<Map<String, TreeSet<GPSBean>>> out) throws Exception {

        TreeSet<GPSBean> firedRoute = routeState.value();
        
        if (firedRoute != null) {

            System.out.println("timer fired!");

            Map wholeRoute = new HashMap<>();
            wholeRoute.put("jshw_trace:",firedRoute);
            out.collect(wholeRoute);
            routeState.clear(); //清除轨迹状态

            String vpn = firedRoute.first().getVeh_plate();
            JedisCluster jedisCluster = FlinkRedisConn.getJedisCluster(); //集群
            jedisCluster.del("taisl:lkyw:gps:" + vpn); //删除redis实时轨迹
            jedisCluster.close();
        }
        
        outOfHighwayTimes.clear(); //删除出高速计数器
        timerState.clear();
    }
}
```
