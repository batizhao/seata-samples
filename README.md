# 快速开始

这是一个 Seata +  Spring Boot 分布式事务示例。

## 环境

* Seata 1.4
* Spring Boot 2.3.5.RELEASE
* 其它版本如果遇到兼容性问题，查看[部署指南](https://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)

## 步骤

* 下载 [seata](http://seata.io/zh-cn/blog/download.html) 并启动 ```sh seata-server.sh```

* 执行 db.sql，因为 teleDB 测试库只能建表不能建库，所以这里 3 张业务表都放在一个库里，效果和分库是一样的。

  > 实际使用中，每个库都要有  undo_log 表。这里共用一张  undo_log 表。
  >
  > 生产环境还要考虑 Seata 的 HA

* 修改 application. yml 库配置

* 分别启动 4 个服务 ``` mvn spring-boot:run```

## 测试

#### 用例说明：

执行 commit 接口，1001 用户在 account 中 money -5，在 order 订单中增加一条消费记录；在 storage 库存中减一。

执行 rollback 接口，1002 用户在执行过程中触发一个异常，整个事务回滚。

#### 成功场景:

  ```sh
# curl -X POST http://127.0.0.1:8084/api/business/purchase/commit
  ```

此时返回结果为：true，看后台日志 commit， account.money 减少 5，order 增加 1 条记录。

#### 失败场景:

 UserId 为 1002  的用户下单，sbm-account-service 会抛出异常，事务会回滚

  ```sh
# curl -X POST http://127.0.0.1:8084/api/business/purchase/rollback
  ```

此时返回结果为：false，看后台日志 rollback。

``` java
public void debit(String userId , BigDecimal num) {
    Account account = accountMapper.selectByUserId(userId);
    account.setMoney(account.getMoney().subtract(num));
    accountMapper.updateById(account);

    //这里故意为 1002 抛了个异常
    if (ERROR_USER_ID.equals(userId)) {
        throw new RuntimeException("account branch exception");
    }
}
```







