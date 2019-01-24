---
layout: post
title: rocketMQ消息堆积监控的java实现
category: arch
tags: [arch]
keywords: rocketMQ
---
## 前言：
最近搭框架用到了rocketMQ队列，需要实现java代码中实现队列中rocketMQ消息堆积的监控，即在先队列中放入消息时，获取当前队列中未消费消息的堆积量，用来判断是否将当前消息立马放入还是等待一段时间在放入，建立和队列的心跳连接，以避免生产者生产大量消息，而消费者未能及时消费，而引起的消息的大面积堆积。

## 1、rocketMQ部署，创建和使用
网上有大量资料，就不在赘述，请自行百度。

## 2、rocketMQ消息堆积监控的java实现
本人做此监控前在网上找了许多资料关于rocketMQ消息堆积监控，但网上都是基于服务器命令的消息堆积情况的查看，并没有找到我想要的，本着求人不如求己，我找到了rocketMQ提供的web页面管理的war包，部署后发现页面控制台中可以看到堆积信息，而后阅读其中的源码，，终于找到我想要的了，在rocketmq-tools-3.2.6.jar这个jar包中提供了查看消费信息的方法，前情说完了，上代码：

**初始化生产者客户端和监控端代码：**

``` java
	public void init() {
		//初始化生产者客户端
		try {
			producer = new DefaultMQProducer(groupName);
			producer.setNamesrvAddr(nameServ);
			producer.start();
		} catch (MQClientException e) {
			logger.error(e.getMessage());
		}
		//初始化监控客户端
		try {
			defaultMQAdminExt = new DefaultMQAdminExt();
		defaultMQAdminExt.setInstanceName(Long.toString(System.currentTimeMillis()));
		    defaultMQAdminExt.setNamesrvAddr(nameServ);
			defaultMQAdminExt.start();
		}catch (Exception e){
			logger.error("监控客户端启动失败...", e);
		}
		//当jvm关闭的时,关闭两个客户端
		Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
			public void run() {
				producer.shutdown();
				defaultMQAdminExt.shutdown();
			}
		}));
	}

```
**获取消息堆积量代码：**

``` java
/**
	 * 获取消息堆积量
	 * @version 1.0 
	 * @author zhangpeng
	 * @return long 
	 * @exception Exception
	 *
	 */
	public long getDiffNum(){
	    long diffTotal = 0L;
		try{
			//当消费端未消费时，此方法会报错
			ConsumeStats consumeStats = defaultMQAdminExt.examineConsumeStats(this.groupName);
		    List<MessageQueue> mqList = new LinkedList(); 
   mqList.addAll(consumeStats.getOffsetTable().keySet());
		    Collections.sort(mqList);
			//遍历所有的队列，计算堆积量
	        for (MessageQueue mq : mqList) {
	        	//只计算group下此生产端发送对应的Topic
	        	if(this.topicName.equals(mq.getTopic())){
		          OffsetWrapper offsetWrapper = (OffsetWrapper)consumeStats.getOffsetTable().get(mq);
		          long diff = offsetWrapper.getBrokerOffset() - offsetWrapper.getConsumerOffset();
		          diffTotal += diff;
	        	}
	        }
		}catch(Throwable e){
			logger.error("监控客户端获取消息堆积量异常，未能正常获取消息堆积量，消息堆积量默认设置为0", e);
			//此中出现任何错误，均返回堆积量为0；
			diffTotal = 0L;
			/**
			 * 此处屏蔽不要了，鉴于这种情况只是刚开始时消费端未消费时会出现，发生频率低，
			 * 且捕获异常后获取队列偏移量，无法确定异常类型，
			 * 会对后面的逻辑处理造成影响，故不兼容此情况了
			 */
			//当只有生产，还未有消费时，上述方法会报错，
			//这是只需获取topic中所有的Queue最大位移和即为消息堆积量
//			try{
//				TopicStatsTable topicStatsTable = defaultMQAdminExt.examineTopicStats(this.topicName);
//				List<MessageQueue> mqList = new LinkedList();
//				mqList.addAll(topicStatsTable.getOffsetTable().keySet());
//			    Collections.sort(mqList);
//			    diffTotal = 0L;
//			    for (MessageQueue mq : mqList) {
//		    		TopicOffset topicOffset = (TopicOffset)topicStatsTable.getOffsetTable().get(mq);
//			        long diff = topicOffset.getMaxOffset() - topicOffset.getMinOffset();
//			        diffTotal += diff;
//			    }
//			}catch(Throwable e1){
//				logger.error("监控获取消息堆积量出错，返回消息堆积量为0");
//				logger.error(e.getMessage(), e);
//		        logger.error(e1.getMessage(), e1);
//		        diffTotal = 0L;
//			}
		}
		return diffTotal;
	}
```

 - **通过调用getDiffNum()方法可获取到队列中的消息堆积量，，消息放入前可调用此方法，得到堆积量，而后判断是否立即放入**
 - **上述方法中是通过获取队列生产消息量减去消息消费量，从而得到了消息堆积量；**
 - **上述方法中有个问题是当topic为刚创建时，队列中只有生产者的生产的消息，而从未被消费过，此时上述方法会因获取不到消费信息而报错，此时需注意**，本人最初是采用当报错时，即认为是队列未消费，就直接获取生产消息量来作为堆积量（代码中catch屏蔽的那段代码），但这这种方式太过武断，若是其他原因的抛错也会进来，，不好，所以就给屏蔽了，不过本人认为刚创建而未有消费的情况下堆积量不会特别大，不如就返回堆积量为0，并且若保证生产者和消费者同步进行就可避免上述报错。
 
 **以上仅为本人在使用rocketMQ时，通过实验和阅读源码，得到的实践观点和经验，如若有不对之处，还望不吝指出**