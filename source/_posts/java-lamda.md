---
title: java lamda
date: 2021-12-21 18:16:25
tags: java
categories: 语言
---
#### java lamda
```
@Data
class User {
   /** 用户id */
   private String id;
   /** 用户姓名 */
   private String name;
   /** 性别:0-男/1-女 */
   private String sex;
   /** 年龄 */
   private Integer age;
}
 
@Data
class UserSum {
   /** 性别:0-男/1-女 */
   private String sex;
   /** 年龄 */
   private Integer age;
 
    public UserSum (String sex, Integer age) {
            this.sex= sex;
            this.age= age;
        }
 
        @Override
        public String toString() {
            return this.sex+ ":" + this.age;
        }
 
        @Override
        public boolean equals(Object obj) {
            if(obj == null) {
                return false;
            }
            if(!(obj instanceof UserSum)) {
                return false;
            }
            if(obj == this) {
                return  true;
            }
            UserSum o = (UserSum)obj;
            return this.sex.equals(o.sex) && this.age.equals(o.age);
        }
 
        @Override
        public int hashCode() {
            return this.sex.hashCode() + this.age.hashCode();
        }
}
 
List<User> userList;
 
// 循环
userList.foreach(m->{
   // do something
});
 
// 过滤（筛选姓名为女的用户）
List<User> list = userList.stream().filter(m->"1".equals(m.getSex())).collect(Collectors.toList());
// 两层过滤
List<UserRole> list = userRoleList.stream().filter(m->userList.stream().filter(a->a.getId().equals(m.userId())).findFirst().isPresent()).collect(Collectors.toList());
 
// list值转换(获取对象某个字段)
List<String> list = userList.stream().map(User::getId).collect(Collectors.toList());
// list值转换（某个对象转为另一个对象）
List<UserVO> list = messageList.stream().map(m->{
   UserVO userVO = new UserVO();
   userVO.setId(m.getId());
   return userVO;
}).collect(Collectors.toList());
 
// list转map
Map<String, String> users = userList.stream().collect(Collectors.toMap(User::getId, User::getName));
Map<String, User> users = userList.stream().collect(Collectors.toMap(User::getName, Function.identity()));
Map<String, String> users = userList.stream().collect(Collectors.toMap(User::getId, u->u.getAge().toString())); // 类型转换
 // 解决key值重复
Map<String, User> users = userList.stream().collect(Collectors.toMap(User::getName, Function.identity(), (key1, key2) -> key2));
 
// map转list
List<User> userList = map.entrySet().stream().map(e -> new User(e.getKey(),e.getValue())).collect(Collectors.toList());
 
// list去重
List<User> userList = users.stream().collect(Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(user ->user.getName()))), ArrayList::new));
 
// 分组
Map<String, List<User>> map = userList.parallelStream().collect(Collectors.groupingBy(User::getSex));
Map<String, List<User>> map = userList.parallelStream().collect(Collectors.groupingBy(m->Integer.value(m.getSex())));
Map<String, List<String>> map = userList.parallelStream().collect(Collectors.groupingBy(User::getSex, Collectors.mapping(User::getId, Collectors.toList())));
// 分组并对key排序
TreeMap<String, List<User>>  map = list.stream().collect(Collectors.groupingBy(User::getSex, TreeMap::new, Collectors.toList()));
// 多条件分组
// 方式一
Map<String, Map<String, List<User>>> collect = list.stream().collect(Collectors.groupingBy(User::getSex, Collectors.groupingBy(User::getName)));
// 方式二
Map<UserSum, List<User>> collect1 = list.stream().collect(Collectors.groupingBy(um -> new UserSum(um.getSex(), um.getName())));
// 分组求和, BigDecimal可以用Collectors.reducing(BigDecimal.ZERO, User::getAge, BigDecimal::add)
Map<String, Map<String, Integer>> collect2 = list.stream().collect(Collectors.groupingBy(User::getSex, Collectors.groupingBy(User::getName, Collectors.summarizingInt(User::getAge))));
// 求每组最大值（返回对象）
Map<String, User> collect4 = list.stream().collect(Collectors.groupingBy(User::getSex, Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparing(User::getAge)), Optional::get)));
// 求每组最大值（返回值）
Map<String, Integer> collect5 = list.stream().collect(Collectors.groupingBy(User::getSex, Collectors.reducing(0, User::getAge, Integer::max)));
// 求平均数
Map<String, Double> collect8 = list.stream().collect(Collectors.groupingBy(User::getSex, Collectors.averagingInt(User::getAge)));
// 获取年龄小于30岁的（having group by ）
Map<Long, Long> collect = result.stream().collect(Collectors.collectingAndThen(Collectors.groupingBy(User::getAge, Collectors.counting()), m -> {
                m.values().removeIf(v -> v >= 30);
                return m;
            }));
 
// 排序
// 反序排序
List<User> list = userList.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());
// 按照某个字段排序（默认正序）
List<User> collect = users.stream().sorted(Comparator.comparing(User::getName)).collect(Collectors.toList());
// 按照某个字段排序（倒叙）
List<User> collect = users.stream().sorted(Comparator.comparing(User::getName).reversed()).collect(Collectors.toList());
// 某个字段处理后再排序
Collections.sort(userList, (o1, o2)->Integer.valueOf(o1.getId()).compareTo(Integer.valueOf(o2.getId())));
// 多字段排序先根据性别正序排序，再根据姓名倒叙排序
List<User> collection = users.stream().sorted(Comparator.comparing(User::getSex).thenComparing(User::getName).reversed()).collect(Collectors.toList());
 
// 求最大值、最小值
Integer max = list.stream().map(User::getAge).max(Integer::compareTo).get();
Integer min= list.stream().map(User::getAge).min(Integer::compareTo).get();
 
 
// 分页，从0开始取前5条
List<User> collection = list.stream().skip(0).limit(5).collect(Collectors.toList());
 
// 初始化map
Map<String, String> events = new ImmutableMap.Builder<String, String>().put("abc", "123").build();    
 
 
//String 字符串转换list<Integer>类型
List<Integer> channels = Arrays.stream(ids.split(",")).mapToInt(Integer::parseInt).boxed().collect(Collectors.toList());
```
