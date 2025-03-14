# List和Map互转

### List转Map

> tips: 当集合对象key重复时可根据`(oldData, newData) -> newData`设置保留新值还是旧值，这里是保留新值

```
Map<Integer, User> map = list.stream().collect(Collectors.toMap(User::getId, t -> t, (oldData, newData) -> newData));

Map<Integer, String> map2 = list.stream().collect(Collectors.toMap(User::getId, User::getName, (oldData, newData) -> newData));
```

### List嵌套对象转List<String>

```
// 提取 List 对象中包含的 List<Integer> 转新集合
List<Integer> idList = list.stream().flatMap(e -> e.getIdList().stream()).collect(Collectors.toList());

// 提取 List 对象中包含的 List<DictBO> 中的code值 转 新集合
List<String> codeList = list.stream().filter(e -> Strings.isNotBlank(e.getContent())).flatMap(e -> {
    List<DictBO> list = JSONUtil.toList(e.getContent(), DictBO.class);
    return list.stream().map(DictBO::getCode).filter(Strings::isNotBlank);
}).collect(Collectors.toList());
```

#### list转Map<Integer, List<Integer>>

```
List<Integer> list = Lists.newArrayList(1, 2, 3, 1, 3);
// value可能会重复
Map<Integer, List<Integer>> idGroupMap = list.stream().collect(Collectors.groupingBy(e -> e)); // {"1":[1,1],"2":[2],"3":[3,3]}
```

#### list对象转Map<String, List<对象>>

```
Map<String, List<SysDictVO>> map = Maps.newHashMap();
List<SysDictVO> list = this.list();
if (CollectionUtils.isEmpty(list)) {
    return map;
}
for (SysDictVO item : list) {
    map.computeIfAbsent(item.getCode(), k -> new LinkedList<>()).add(item);
}
```

或

```
// 分组 value会重复
Map<String, List<SysDictVO>> map = list.stream().collect(Collectors.groupingBy(
                            SysDictVO::getCode, 
                            Collectors.mapping(t -> t, Collectors.toList())));

// 分组 value不重复
Map<Long, List<Long>> codeReTypesMap = list.stream().collect(Collectors.groupingBy(
                            SysDictVO::getCode,
                            Collectors.mapping(SysDictVO::getType, Collectors.collectingAndThen(Collectors.toSet(), ArrayList::new))));
```

#### list对象转Map<Long, Map<String, List<对象>>>

```
public class MetricsField {
    private Long userId;
    private String code;
    private String value;
}

Map<Long, Map<String, List<MetricsField>>> userIdReCodeResultMap = results.stream()
            .collect(Collectors.groupingBy(
                // 先根据userId分组
                MetricsField::getUserId,
                // 再根据同一组userId数据下的code分组
                Collectors.groupingBy(MetricsField::getCode)
            ));

// {"1":{"code2":[{"userId":1,"code":"code2","value":"value2"}],"code1":[{"userId":1,"code":"code1","value":"value1"}]}}
System.out.println(JSONUtil.toJsonStr(userIdReCodeResultMap));
```

### Map转List

```
List<User> list = map.entrySet().stream()
        .map(
                e -> User.builder().id(e.getKey()).name(e.getValue()).build()
        )
        .collect(Collectors.toList());
```

--- 

### demo

```java
package com.zhengqing.demo.daily.base.java8;

import cn.hutool.json.JSONUtil;
import com.google.common.collect.Lists;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.junit.Test;

import java.util.Date;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class Java8_list_to_map {
    @Test
    public void test() throws Exception {
        List<User> list = Lists.newArrayList(
                User.builder().id(1).age(16).name("小张").build(),
                User.builder().id(10).age(20).name("小孙").build(),
                User.builder().id(1).age(18).name("李四").build(),
                User.builder().id(3).age(6).name("王五").build()
        );

        // 当集合对象key重复时可根据`(oldData, newData) -> newData`设置保留新值还是旧值，这里是保留新值
        Map<Integer, User> map1 = list.stream().collect(Collectors.toMap(User::getId, t -> t, (oldData, newData) -> newData));
        System.out.println(JSONUtil.toJsonStr(map1));

        Map<Integer, String> map2 = list.stream().collect(Collectors.toMap(User::getId, User::getName, (oldData, newData) -> newData));
        System.out.println(JSONUtil.toJsonStr(map2));

        List<User> list2 = map2.entrySet().stream()
                .map(
                        e -> User.builder().id(e.getKey()).name(e.getValue()).build()
                )
                .collect(Collectors.toList());
        System.out.println(JSONUtil.toJsonStr(list2));
    }

    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    static class User {
        private Integer id;
        private String name;
        private Integer age;
        private Date time;
    }
}
```