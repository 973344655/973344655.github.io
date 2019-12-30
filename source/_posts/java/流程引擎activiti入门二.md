---
title: 流程引擎activiti入门 -- 使用
date: 2019-11-28 15:26:40
tags: [java]
---

接上文的take-leave.bpmn20.xml流程。

# 一.流程信息实体类

流程过程中需要的一些信息。

```
@Data
public class BusinessEntity implements Serializable {

    @NotNull(message = "审核人员1不能为空")
    private String leader1;
    private boolean agree1;
    private String desc1;

    @NotNull(message = "审核人员2不能为空")
    private String leader2;
    private boolean agree2;
    private String desc2;

    @NotNull(message = "审核人员3不能为空")
    private String leader3;
    private boolean agree3;
    private String desc3;

    @NotNull(message = "审核人员4不能为空")
    private String leader4;
    private boolean agree4;
    private String desc4;

    private TaskInfoEntity taskInfo;
    private String processId;
}
```
```
@Data
public class TaskInfoEntity  {

    private String taskId;
    private String taskName;
    private String processInstanceId;
    private String assignee;

    public TaskInfoEntity(Task task){

        this.assignee = task.getAssignee();
        this.processInstanceId  = task.getProcessInstanceId();
        this.taskId = task.getId();
        this.taskName = task.getName();
    }


}
```

# 二.一个简单流程

```
@Controller
@RequestMapping(value = "/business")
public class BusinessController {
    @Autowired
    BusinessService businessService;



    @RequestMapping(value = "/start",method = RequestMethod.POST)
    @ResponseBody
    public Object start(@RequestBody BusinessEntity entity){
      return ResultMessage.success(businessService.start(entity));
    }

    @RequestMapping(value = "/approve1",method = RequestMethod.POST)
    @ResponseBody
    public Object approve1(@RequestBody BusinessEntity entity){
        return businessService.approve1(entity);
    }

    @RequestMapping(value = "/approve2",method = RequestMethod.POST)
    @ResponseBody
    public Object approve2(@RequestBody BusinessEntity entity){
        return businessService.approve2(entity);
    }

    @RequestMapping(value = "/approve3",method = RequestMethod.POST)
    @ResponseBody
    public Object approve3(@RequestBody BusinessEntity entity){
        return businessService.approve3(entity);
    }

    @RequestMapping(value = "/approve4",method = RequestMethod.POST)
    @ResponseBody
    public Object approve4(@RequestBody BusinessEntity entity){
        return businessService.approve4(entity);
    }

}

```

```
@Service
public class BusinessService {

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private TaskService taskService;

    @Autowired
    private ProcessEngine processEngine;



    public Object start(BusinessEntity entity){

        Map<String, Object> vars = new HashMap<>();
        vars.put("business",entity);
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("process_business_opportunities_examine",vars);

        entity.setProcessId(processInstance.getProcessInstanceId());
        System.out.println("processId: " + entity.getProcessId());
        List<Task> list = taskService.createTaskQuery().processInstanceId(processInstance.getProcessInstanceId()).list();
        if(list == null || list.size() <= 0){
            return "task生成错误";
        }
        entity.setTaskInfo(new TaskInfoEntity(list.get(0)));

        return entity;
    }

    public Object approve1(BusinessEntity entity){

        if(!entity.isAgree1()){
            return "审批不通过";
        }

        String taskId1 = entity.getTaskInfo().getTaskId();
        taskService.complete(taskId1);

        List<Task> list = taskService.createTaskQuery().processInstanceId(entity.getProcessId()).list();
        if(list == null || list.size() <= 0){
            return "task生成错误";
        }
        entity.setTaskInfo(new TaskInfoEntity(list.get(0)));

        return entity;
    }
    public Object approve2(BusinessEntity entity){

        if(!entity.isAgree2()){
            return "审批不通过";
        }

        String taskId1 = entity.getTaskInfo().getTaskId();
        taskService.complete(taskId1);

        List<Task> list = taskService.createTaskQuery().processInstanceId(entity.getProcessId()).list();
        if(list == null || list.size() <= 0){
            return "task生成错误";
        }
        entity.setTaskInfo(new TaskInfoEntity(list.get(0)));

        return entity;
    }
    public Object approve3(BusinessEntity entity){

        if(!entity.isAgree3()){
            return "审批不通过";
        }

        String taskId1 = entity.getTaskInfo().getTaskId();
        taskService.complete(taskId1);

        List<Task> list = taskService.createTaskQuery().processInstanceId(entity.getProcessId()).list();
        if(list == null || list.size() <= 0){
            return "task生成错误";
        }
        entity.setTaskInfo(new TaskInfoEntity(list.get(0)));

        return entity;
    }
    public Object approve4(BusinessEntity entity){

        if(!entity.isAgree4()){
            return "审批不通过";
        }

        String taskId1 = entity.getTaskInfo().getTaskId();
        taskService.complete(taskId1);


        return "流程: " + entity.getProcessId() + "  已完成";
    }
}

```
