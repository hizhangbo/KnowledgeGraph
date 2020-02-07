## JAVA 8
- lambda
  - 接口
    - 函数式接口
      - 自定义接口
        - 单一功能原则
        - @FunctionalInterface
      - 内置
      
        |接口|输入参数|返回类型|说明|
        |---|---|---|---|
        |`Predicate<T>`|T|boolean|断言|
        |`Function<T,R>`|T|R|输入T 输出R|
        |`Supplier<T>`|/|T|提供一个数据|
        |`Consumer<T>`|T|/|消费一个数据|
        |`UnaryOperator<T>`|T|T|一元函数（输入输出类型相同）|
        |`BinaryOperator<T>`|(T,T))|T|二元函数（输入输出类型相同）|
        |`BiFunction<T,U>`|(T,U))|R|两个输入的函数|
      - 方法引用简化Lambda
        - 函数执行体只有一条函数调用，且函数参数和lambda左边一样时，可使用两个冒号连接（类名/实例名）和方法名简化调用
    - 接口默认方法实现
      - default
  - 参数传递必须不可修改
    - final 修饰，传递值而非引用
  - 级联表达式
    - Function<Integer, Function<Integer,Integer>> func = x -> y -> x+y;
    - func.apply(1).apply(2);
  - 函数柯里化
    - 把多个参数的函数转化为只有一个参数的函数
    - 函数标准化
  - 高阶函数
    - 返回值为函数
- Stream
  - 优势
    - 惰性求值
      - 中间操作：返回Stream
      - 终止操作：返回具体结果
    - 屏蔽细节
    - 并行优化
  - 创建方式
    - |类型|方法|
      |---|---|
      |集合|Collection.stream/parallelStream|
      |数组|Arrays.stream|
      |数字Stream|IntStream/LongStream.range/rangeClosed|
      |--|Random.ints/longs/doubles|
      |自己创建|Stream.generate/iterate|
  - 中间操作
    - |状态依赖|方法|说明|
      |---|---|---|
      |无状态|map/mapToXxx|单层转换|
      |无状态|flatMap/flatMapToXxx|多层转换|
      |无状态|filter|过滤|
      |无状态|peek|遍历|
      |无状态|unordered|并行时使用|
      |有状态|distinct|去重|
      |有状态|sorted|排序|
      |有状态|limit/skip|分页|
  - 终止操作
    - |类型|方法|说明|
      |---|---|---|
      |非短路|forEach/forEachOrdered|遍历|
      |非短路|collect/toArray|转集合/转数组|
      |非短路|reduce|合并|
      |非短路|min/max/count|聚合|
      |短路|findFirst/findAny|只要符合条件则返回|
      |短路|allMatch/anyMatch/noneMatch|只要符合条件则返回|
  - 并行流 ParallelStream
    - stream.parallel()
  - **链式调用**
    - 无状态操作是连续的，内部迭代时每个元素会依次执行函数链
    - 有状态操作会等待无状态操作执行完成后，再执行有状态函数，然后再调用后面的函数链
    - 使用并行parallel时，有状态操作不一定并行，例如排序依然用主线程
  - collect
    - IntSummaryStatistics result = stream.collect(Collectors.summarizingInt(Student::getAge)); # 获取聚合计算结果
    - Map<Boolean, List<Student>> genderPartitions = stream.collect(Collectors.partitioningBy(s->s.getGender() == Gender.MALE)); # 依据某个条件分块
    - Map<Grade, List<Student>> grades = stream.collect(Collectors.groupingBy(Student::getGrade)); # 依据某个条件分组
    - Map<Grade, Long> grades = stream.collect(Collectors.groupingBy(Student::getGrade, Collectors.counting())); # 分组后计算