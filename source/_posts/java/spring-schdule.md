---
title: spring-schdule
date: 2018-11-19 11:18:54
tags: [spring,springboot]
---

#### 1.@Schduled注解
配置完后，在方法上加@Scheduled("定时表达式")

#### 2.实现ScheduingConfigurer接口
```
@Lazy(false)
@EnableScheduling
@Component
public class DynamicScheduledTask implements SchedulingConfigurer {

	@Value("${abc.call-cron}")
	private String callCron;

  //重写configureTasks
	@Override
	public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		if(!"0".equals(callCron)){
			taskRegistrar.addTriggerTask(new Runnable() {
				@Override
				public void run() {
					//doSomething
				}
			}, new Trigger() {

				@Override
				public Date nextExecutionTime(TriggerContext triggerContext) {
          //定时器触发，callCron为定时表达式
					CronTrigger trigger = new CronTrigger(callCron);
					Date nextExecDate = trigger.nextExecutionTime(triggerContext);
					return nextExecDate;
				}
			});
		}

}

```
