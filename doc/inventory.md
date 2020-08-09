```
折戟沉沙铁未销，自将摩洗认前朝。
东风不与周郎便，铜雀春深锁二乔。
《赤壁》—— 杜牧
```

#### 方案描述
创建订单时预占库存，预占成功，则进行订单创建。
用户支付成功后，发送扣减库存消息到MQ，库存服务进行库存真正扣减；
订单取消，发送释放库存消息到MQ，库存服务释放库存。

#### 序列图
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbde729bf8e5447dbf4c6c78533009d3~tplv-k3u1fbpfcp-zoom-1.image)

流程说明：

1~3. 将库存同步到redis中

4~5. 下单，订单服务发起预占库存请求（根据订单号）

6. 库存服务进行库存判断和预占

	```
    /**
     * 预占库存 伪代码
     * @param orderNo 订单号
     * @param skuId sku标识
     * @param quantity 预占数量
     */
    boolean preOccupy(String orderNo, String skuId, int quantity) {
    
    	boolean isPreOccupySuccess = false;
        
        int value = redis.decrby(skuId, quantity);
        if (value >= 0) {
        	// 库存充足
            // 生成库存预占流水记录（关键字段：orderNo,skuId,quantity,state(0-预占；1-已扣减；2-已释放)
            
            isPreOccupySuccess = true;
        
        } else {
        	// 库存不足，返还刚才预占的库存
            redis.incrby(skuId, qunatity);
        }
        	
        
    }
    ```

7. 返回库存预占结果

	1）预占成功，继续下单流程
    
    2）预占失败，库存不足，无法创建订单
    
8. 支付成功，订单服务发送扣减库存消息到MQ

9. 订单取消，订单服务发送释放库存消息到MQ

10. 库存服务接收到消息：

	1）扣减库存消息，库存预占流水对应记录状态更新为已扣减
    
    2）释放库存消息，库存预占流水对应记录状态更新为已释放，库存表和redis中返还相应的库存
    
    
    
#### 备注
可以增加库存调度服务，对超过一定时间的预占库存进行释放：
假如订单超时是30分钟，则调度服务可以查询出处于预占状态的超时的流水（大于30分钟），向订单服务确认是否下单成功，如果成功，则修改状态为已扣减，如果失败，则返还库存