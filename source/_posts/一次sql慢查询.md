---
title: 一次sql慢查询
date: 2019-11-06 14:26:34
tags: [others]
---


今天发现一个页面，打开后卡死，发现是sql执行时间太长导致。

## 1.原sql

<details>
<summary> 原sql </summary>
```
<e:case value="getParentNodes">
    <e:description>获取满足条件的所有父节点</e:description>
    <e:q4l var="parentNodeList">
        select
            t.CUSTOMER_NO
        from (
            select
                t1.customer_no,t1.parent_no
            from
                ${gis_user}.T_CUSTOMER_BASE_INFO t1
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t2 on T1.customer_no=T2.CUSTOM_NO AND T2.TYPE=1
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t3 on T1.customer_no=T3.CUSTOM_NO AND T3.TYPE=2
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t4 on T1.customer_no=T4.CUSTOM_NO AND T3.TYPE=3
            LEFT JOIN V_CMCODE_BRANCH v1 on t1.substation_code = v1.branch_no
            LEFT JOIN V_CMCODE_AREA v2 on v1.area_no = v2.area_no
            LEFT JOIN V_CMCODE_CITY v3 on v1.city_no = v3.city_no
            where 1 = 1
                and customer_no in (
                    select
                        t1.customer_no
                    from
                    ${gis_user}.T_CUSTOMER_BASE_INFO t1
                    LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t2 on T1.customer_no=T2.CUSTOM_NO AND T2.TYPE=1
                    LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t3 on T1.customer_no=T3.CUSTOM_NO AND T3.TYPE=2
                    LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t4 on T1.customer_no=T4.CUSTOM_NO AND T3.TYPE=3
                    LEFT JOIN V_CMCODE_BRANCH v1 on t1.substation_code = v1.branch_no
                    LEFT JOIN V_CMCODE_AREA v2 on v1.area_no = v2.area_no
                    LEFT JOIN V_CMCODE_CITY v3 on v1.city_no = v3.city_no
                    where 1 = 1
                    <e:if condition="${sessionScope.UserInfo.AREA_NO !='-1' && sessionScope.UserInfo.AREA_NO !='' && sessionScope.UserInfo.AREA_NO !=null}">
                        AND v2.AREA_NO = '${sessionScope.UserInfo.AREA_NO}'
                    </e:if>
                    //其它条件
                )
            <e:if condition="${sessionScope.UserInfo.AREA_NO !='-1' && sessionScope.UserInfo.AREA_NO !='' && sessionScope.UserInfo.AREA_NO !=null}">
                AND v2.AREA_NO = '${sessionScope.UserInfo.AREA_NO}'
            </e:if>
            //其它条件
            ) t
        where t.PARENT_NO not in (
            select
                t1.customer_no
            from
            ${gis_user}.T_CUSTOMER_BASE_INFO t1
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t2 on T1.customer_no=T2.CUSTOM_NO AND T2.TYPE=1
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t3 on T1.customer_no=T3.CUSTOM_NO AND T3.TYPE=2
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t4 on T1.customer_no=T4.CUSTOM_NO AND T3.TYPE=3
            LEFT JOIN V_CMCODE_BRANCH v1 on t1.substation_code = v1.branch_no
            LEFT JOIN V_CMCODE_AREA v2 on v1.area_no = v2.area_no
            LEFT JOIN V_CMCODE_CITY v3 on v1.city_no = v3.city_no
            where 1 = 1
            <e:if condition="${!empty(param.customerType) && param.customerType ne ''}">
                and C_TYPE = '${param.customerType}'
            </e:if>
            //其它条件
         )

    </e:q4l>${e:java2json(parentNodeList.list)}
</e:case>
```
</details>

简化下

```

select
  t.a
from ( select xxx from xx t1 where t1.a in (xxx) ) t
where t.b not in (select xxx from xxx)

```

## 2.原因

in 和 not in 操作效率低， 非常耗时，在数据量大点时，页面直接一直等待它.

## 3.解决

参考:https://www.cnblogs.com/phoenixfling/archive/2012/05/09/2492006.html

这里使用了第二种

```
<e:case value="getParentNodes">
        <e:description>获取满足条件的所有父节点</e:description>
        <e:q4l var="parentNodeList">

            select
            t1.customer_no
            from
            ${gis_user}.T_CUSTOMER_BASE_INFO t1
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t2 on T1.customer_no=T2.CUSTOM_NO AND T2.TYPE=1
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t3 on T1.customer_no=T3.CUSTOM_NO AND T3.TYPE=2
            LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t4 on T1.customer_no=T4.CUSTOM_NO AND T3.TYPE=3
            LEFT JOIN V_CMCODE_BRANCH v1 on t1.substation_code = v1.branch_no
            LEFT JOIN V_CMCODE_AREA v2 on v1.area_no = v2.area_no
            LEFT JOIN V_CMCODE_CITY v3 on v1.city_no = v3.city_no
            LEFT JOIN (select
                        t1.customer_no
                        from
                        ${gis_user}.T_CUSTOMER_BASE_INFO t1
                        LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t2 on T1.customer_no=T2.CUSTOM_NO AND T2.TYPE=1
                        LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t3 on T1.customer_no=T3.CUSTOM_NO AND T3.TYPE=2
                        LEFT JOIN ${gis_user}.T_CUSTOMER_PEPOLE t4 on T1.customer_no=T4.CUSTOM_NO AND T3.TYPE=3
                        LEFT JOIN V_CMCODE_BRANCH v1 on t1.substation_code = v1.branch_no
                        LEFT JOIN V_CMCODE_AREA v2 on v1.area_no = v2.area_no
                        LEFT JOIN V_CMCODE_CITY v3 on v1.city_no = v3.city_no
                        where 1 = 1
                        <e:if condition="${!empty(param.customerType) && param.customerType ne ''}">
                            and C_TYPE = '${param.customerType}'
                        </e:if>
                        //其它条件

            where 1 = 1  and tmp.customer_no is NULL


            <e:if condition="${sessionScope.UserInfo.AREA_NO !='-1' && sessionScope.UserInfo.AREA_NO !='' && sessionScope.UserInfo.AREA_NO !=null}">
                AND v2.AREA_NO = '${sessionScope.UserInfo.AREA_NO}'
            </e:if>
            //其它条件

        </e:q4l>${e:java2json(parentNodeList.list)}
    </e:case>
```

## 4.总结

in 和 not in操作,能不用就不用，尽量少用.
