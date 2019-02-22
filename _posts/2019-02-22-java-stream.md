---
layout:       post
title:        "Java Stream的骚操作 "
subtitle:     "记录使用stream轻松转换后去重的骚操作"
date:         2019-02-22
author:       "Rong"
header-mask:  0.3
catalog:      true
tags:
    - JAVA
    - Stream流
---

 今天工作中遇到需求去重的一个问题，先看下数据格式：

```json
{
	"id": 0,
	"name": "",
	"total": 0,
	"used": 0,
	"startDay": "",
	"endDay": "",
	"state": 0,
	"remain": 0,
	"tickets": [
		{
			"id": 0,
			"tmId": 0,
			"scId": 0,
			"scName": "",
			"quota": 0,
			"sumUsed": 0
		}
	]
}
```

这是前端传给我的JSON数据，再看下后端java数据格式：

```java
@ApiModelProperty(value = "补贴券管理Id")
    @Column(name = "TM_ID")
    private Long id;

    @ApiModelProperty(value = "补贴券项目名",required = true)
    @Column(name = "TM_NAME")
    private String name;

    @ApiModelProperty(value = "发放总额度（总补贴额度）",required = true)
    @Column(name = "TM_TOTAL_QUOTA")
    private Long total;
```

这里有部分数据需要脱敏，我用IDEA的 正则匹配先过滤下数据库字段的列

![idea正则过滤](/img/in-post/post-incloud/2019-02-22_idea_replacepng.png)  

脱敏后的字段如：

```java
/**
 * 〈补贴券VO〉
 *
 * @author mazhongrong@smeyun.com
 * @date 2019/2/15
 */
@Data
@Table(name = "TBL_TICKET_MGT")
public class TicketVO implements Serializable {
    private static final long serialVersionUID = 7360617243527084341L;

    @ApiModelProperty(value = "补贴券管理Id")
    private Long id;

    @ApiModelProperty(value = "补贴券项目名",required = true)
    private String name;

    @ApiModelProperty(value = "发放总额度（总补贴额度）",required = true)
    private Long total;

    @ApiModelProperty(value = "已用额度")
    private Long used;

    @ApiModelProperty(value = "发放时间起",required = true)
    private String startDay;

    @ApiModelProperty(value = "发放时间止",required = true)
    private String endDay;
    /**
     * 发放状态
     * 0：未开始
     * 1；执行中
     * 2：已结束
     * 3：暂停',
     */
    @ApiModelProperty(value = "发放状态0：未开始 1；执行中 2：已结束 3：暂停")
    private Integer state;

    @ApiModelProperty(value = "剩余")
    private Long remain;

    @ApiModelProperty(value = "企业补贴券详细")
    private List<Quota> tickets;

}
```

```java
/**
 * 〈企业补贴额度〉
 *
 * @author mazhongrong@smeyun.com
 * @date 2019/2/15
 */
@Data
@Table(name = "TBL_EN_QUOTA")
@ApiModel("企业补贴额度明细")
public class Quota implements Serializable {
    private static final long serialVersionUID = -3528114897530756709L;

    @ApiModelProperty(value = "企业补贴券id")
    private Long id;
    
    @ApiModelProperty(value = "补贴券管理id")
    private Long tmId;
    
    @ApiModelProperty(value = "类目id",required = true)
    private Long scId;
    
    @ApiModelProperty(value = "类目名",required = true)
    private String scName;
    
    @ApiModelProperty(value = "补贴上限额度",required = true)
    private Long quota;
    
    /**
     * 企业订单已使用计算汇总
     */
    @ApiModelProperty(value = "已补贴金额")
    private Long sumUsed;
```



## 说重点：判断 `List<Quota>`中`scId`和`scName`不为空，且所有Quota的`quota`之和不能大于`total，`且scId 不能重复

- 直接上码：

  ```java
  List<Quota> tickets = ticket.getTickets();
          int length = tickets.size();
          // 是否有类目id和名字为空
          int checkNull =
                  tickets.stream().filter(t ->t.getScId() != null && StringUtils.isNotBlank(t.getScName()))
                          .collect(Collectors.toList()).size();
          if (length != checkNull) {
              return ErrorDefined.ACTIVITY_PARAMETER_ERROR;
          }
          // 是否有重合类目
          // 根据scId去重
          ArrayList<Quota> collect = tickets.stream().collect(
                  Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparingLong(Quota::getScId))), ArrayList::new)
          );
          if (length != collect.size()) {
              return ErrorDefined.ADD_COUPONS_TYPE_ERROR;
          }
          // 子券的补贴上限金额不能超过总券的总补贴金额；
          long sum = tickets.stream().mapToLong(Quota::getQuota).sum();
          if (ticket.getTotal() < sum) {
              return ErrorDefined.ADD_COUPONS_COUNT_ERROR;
          }
  ```

- 分析：

  - 首先拿去出tickets，stream流后根据scId和scName不为空过滤，再对比长度是否缩减
  - 首先我们将Quota::getScId转换TreeSet（根据数据结构我们指定只有不同元素才会添加进去），再将TreeSet转回ArrayList(不回丢失数据)，最后我们再对比下数据大小，有缩减说明有相同id的类目数据被添加了。

- 结论：

  虽然前端做了判断不能将重复类目的数据传入，但后端不能信任任何传递数据，需要自己验证；也实现过其它方法，比如可以重写比对方法等，效果和编码简洁均不理想。

  熟悉Java的流操作和数据结构对我们的编码很重要！

撒花*★,°*:.☆(￣▽￣)/$:*.°★* 。

