```
落魄江湖载酒行，楚腰纤细掌中轻。
十年一觉扬州梦，赢得青楼薄幸名。
《遣怀》—— 杜牧
```

#### 功能点
|  功能  |  接口 |
| ---------|------- |
| 充值     | /recharge|
| 冻结     | /frozen|
| 解冻     | /unfrozen|
| 扣减     | /dedubt|
| 查询余额  | /query-balance|
| 账户明细     | /detail|

#### 数据库设计

###### t_wallet(钱包表)

|  功能  |  类型 | 说明 |
| ---------|------- |-----|
| wallet_id     | varchar| 钱包标识 |
| user_id     | varchar| 用户标识 |
| balance     | long | 总余额，单位：分可用余额 = balance - frozen_amount|
| frozen_amount     | long| 冻结金额，单位：分|
| state  | int| 钱包状态：0-停用；1-正常|
| give_amount|long     |赠送金额，单位：分|


###### t_wallet_detail(钱包明细记录)

|  功能  |  类型 | 说明 |
| ---------|------- |-----|
| wallet_id     | varchar| 钱包标识 |
| trade_type     | int| 交易类型，1-转入（充值、返还等）；2-转出（扣减、服务费结算等） |
| change_amount     | 变动金额，充值为正数，扣减为负数，单位：分|
| after_change_amount     | long| 变动后金额，单位：分|
| give_amount  | long| 赠送金额，充值时使用，单位：分|
| give_amount|long     |赠送金额，单位：分|
| change_note|varchar     |备注，系统自动备注|


###### t_wallet_order(钱包订单)

|  功能  |  类型 | 说明 |
| ---------|------- |-----|
| order_id     | varchar| 订单标识 |
| order_state     | int| 订单状态，1-冻结；2-解冻（返还余额）；3-扣减；4-充值； |
| order_no    | varchar | 订单号，业务方传递|
| change_amount     | long| 变动金额，充值为正数，扣减为负数，单位：分|
| give_amount  | long| 赠送金额，充值时使用，单位：分|
| give_amount|long     |赠送金额，单位：分|

#### 钱包订单变化
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/867862993a264c2999f7b359cdbbd350~tplv-k3u1fbpfcp-zoom-1.image)

##### 冻结请求：

     1）t_wallet_order 增加新记录，状态为 冻结（1）

     2）t_wallet  增加冻结金额（frozen_amount）

     3）t_wallet_detail  增加新转出记录（待定是否需要）

##### 解冻请求：

    1）t_wallet_order 更新状态为 解冻（2）

    2）t_wallet 减去冻结金额

    3）t_wallet_detail  增加转入记录（待定是否需要）


##### 扣减请求：

    1）t_wallet_order 如果存在冻结记录，则更新状态为 扣减（4）

    2）t_wallet_order 如果不存在记录，则增加新记录，状态为 扣减（4）

    3）t_wallet 余额减去订单金额，冻结金额减去订单金额

    4）t_wallet_detail 增加一条转出记录

##### 充值请求：

    1）t_wallet_order 增加新记录，状态为 充值（3）

    2）t_wallet 余额加上订单金额

    3）t_wallet_detail 增加一条转入记录


#### 组合支付时的流程图
用户同时使用在线支付和余额支付结合的方式付款
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97b40dfe64ba4bbbaf589c3c6dd9635a~tplv-k3u1fbpfcp-zoom-1.image)