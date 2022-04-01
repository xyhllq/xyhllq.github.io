
## 一.分组
### 1.多条件分组
```

Map<String, List<GroupByEntity>> resultMap = groupByList.stream().collect(Collectors.groupingBy(record -> record.getKeyFirst() + "-" + record.getKeySencond()));

```


## 二.排序

### 1.正序/倒序

```
//默认正序
//Comparator.comparing(SortEntity::getKeyFirst,Comparator.naturalOrder())
//Comparator.naturalOrder()可以省略
sortList = sortList.stream()
                .sorted(Comparator.comparing(SortEntity::getKeyFirst))
                .collect(Collectors.toList());

//倒叙
sortList = sortList.stream()
                .sorted(Comparator.comparing(SortEntity::getKeyFirst,Comparator.reverseOrder()))
                .collect(Collectors.toList());

```

### 2.多条件排序
```

sortList = sortList.stream()
                .sorted(Comparator.comparing(SortEntity::getKeyFirst)//默认正序
                                  .thenComparing(SortEntity::getKeySecond,Comparator.reverseOrder())//倒序
                )
                .collect(Collectors.toList());

```