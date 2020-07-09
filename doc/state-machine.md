```
    葡萄美酒夜光杯，欲饮琵琶马上催。
    醉卧沙场君莫笑，古来征战几人回。
    ——<<凉州词>> 王翰
```

#### 概念理解

状态机能够根据触发事件按照预订的状态进行转移并执行一些动作

有限状态机简写为FSM（Finite State Machine），有限个状态。

四要素：
1. 现态（State）: 当前所处的状态
2. 事件（Event）: 触发某个状态转换的条件
3. 动作（Action）: 条件满足后执行的操作。动作执行完毕后，可以迁移到新的状态，也可以仍旧保持原状态。动作不是必需的，当条件满足后，也可以不执行任何动作，直接迁移到新状态。
4. 转换（Transition）：就是次态，条件满足后要迁往的新状态。“次态”是相对于“现态”而言的。


#### 核心类图

![](https://user-gold-cdn.xitu.io/2020/7/9/1733229365457431?w=1140&h=699&f=png&s=61908)


* Handler 处理器，就是事件触发后执行的操作，对应四要素的Action。比如说提交订单的业务处理器（SubmitOrderHandler），就需要实现该接口。
* StateMachineDomain 获取/设置状态的抽象，每个状态实体领域实现该接口
* StateMachineConfiguration 状态机配置
* StateMachineConfigurationHolder 状态机配置持有者
* StateMachine 状态机


#### 订单状态机
以简化的订单为例

![](https://user-gold-cdn.xitu.io/2020/7/9/173322e26cd8add1?w=996&h=269&f=png&s=20023)

1. Order实体实现StateMachineDomain接口
2. 定义订单状态枚举(OrderState)和订单事件枚举（OrderEvent）
3. 实现事件触发处理器，实现Handler接口
4. 初始化状态机配置 StateMachineConfigurationHolder 和 StateMachineConfiguration 
5. 创建状态机 StateMachine
6. 触发事件，执行状态转换

初始化状态机配置和创建状态机案例
```
 StateMachineConfigurationHolder<OrderState, OrderEvent, Handler> holder = new StateMachineConfigurationHolder();


        holder.source(OrderState.SUBMIT_ORDER)
                .event(OrderEvent.SUBMIT_ORDER)
                .handler(new SubmitOrderHandler())
                .target(OrderState.WAIT_PAY)
                .build();
                
 StateMachine<OrderState, OrderEvent, Handler> stateMachine = new StateMachine<>(holder);
```

触发事件

```
 Order order = new Order();
 order.setOrderNo(10086L);
 order.setState(OrderState.SUBMIT_ORDER);

 stateMachine.transition(order, OrderEvent.SUBMIT_ORDER);
 Assert.assertEquals(OrderState.WAIT_PAY, order.getState());
```


代码参考：https://github.com/Mr-LuXiaoHua/dohko-state-machine