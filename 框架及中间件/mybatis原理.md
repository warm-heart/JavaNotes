## 重要对象

### SqlSessionFactory

构建SqlSessionFactory

- 通过XMLConfigBuilder解析配置的XML文件，读出配置参数，包括基础配置XML文件和映射器XML文件；
- 使用Configuration对象创建SqlSessionFactory，SqlSessionFactory是一个接口，提供了一个默认的实现类DefaultSqlSessionFactory。

### sqlSession

sqlSession从代理工厂MapperProxyFactory获取代理类，代理类执行方法会进入MapperProxy的invoke方法，sqlSession调用excutor执行数据库操作。

### **MappedStatement**

它保存映射器的一个节点（select|insert|delete|update），包括配置的SQL，SQL的id、缓存信息、resultMap、parameterType、resultType等重要配置内容。它涉及的对象比较多，一般不去修改它。

### SqlSource

它是MappedStatement的一个属性，**主要作用是根据参数和其他规则组装SQL**，也是很复杂的，一般也不用修改它。

select * from user where id=?

###  BoundSql

对于参数和SQL，主要反映在BoundSql类对象上，在插件中，通过它获取到当前运行的SQL和参数以及参数规则，作出适当的修改，满足特殊的要求。**根据不同的sql入参，做出不同的调整，也就是最终数据库执行的带参数的sql语句**

### RowBounds

RowBounds为查询分页，在接口方法中参数中指定就行

使用方式

```
//相当于limit 0，10
如（String name，new RowBounds(0,10）
```

mybatis 查询时默认传入RowBounds，偏移量默认为0到Integer.MAX_VALUE;

```
//查询
 @Override
  public <E> List<E> selectList(String statement, Object parameter) {
  //传入默认的RowBounds
    return this.selectList(statement, parameter, RowBounds.DEFAULT);
  }

public class RowBounds {
  public static final int NO_ROW_OFFSET = 0;
  public static final int NO_ROW_LIMIT = Integer.MAX_VALUE;
  public static final RowBounds DEFAULT = new RowBounds();
  private final int offset;
  private final int limit;
  //RowBounds的默认构造方法
  public RowBounds() {
    this.offset = NO_ROW_OFFSET;
    this.limit = NO_ROW_LIMIT;
  }
```

### Executor

Mybatis的执行器，负责statement对象的组装以及sql的组装，并把statement对象交给StatementHandle执行。

执行器的种类有

- SimpleExecutor：
- BatchExecutor：批处理执行器
- 

### StatementHandle

接收Executor传来的statement对象并执行SQL

### ResultHandle

负责处理结果集

## 执行过程

### mapper接口原理

从sqlSession获取代理类MapperProxy 并执行，通过动态代理执行方法。

```
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
    if (Object.class.equals(method.getDeclaringClass())) {
      return method.invoke(this, args);
    } else if (isDefaultMethod(method)) {
      return invokeDefaultMethod(proxy, method, args);
    }
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
  final MapperMethod mapperMethod = cachedMapperMethod(method);
  //执行方法
  return mapperMethod.execute(sqlSession, args);
}
//获取MapperMethod
private MapperMethod cachedMapperMethod(Method method) {
  MapperMethod mapperMethod = methodCache.get(method);
  if (mapperMethod == null) {
    mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
    methodCache.put(method, mapperMethod);
  }
  return mapperMethod;
}
```

### MapperMethod

- **对参数进行封装**

```
Object param = method.convertArgsToSqlCommandParam(args);
```

- **调用sqlsession中对应方法，**

```
public Object execute(SqlSession sqlSession, Object[] args) {
  Object result;
  //判断类型
  switch (command.getType()) {
    case INSERT: {
   Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
      break;
    }
    case UPDATE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
      break;
    }
    case DELETE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
      break;
    }
    case SELECT:
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
      } else if (method.returnsMany()) {
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
      } else if (method.returnsCursor()) {
        result = executeForCursor(sqlSession, args);
      } else {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
      }
      break;
    case FLUSH:
      result = sqlSession.flushStatements();
      break;
    default:
      throw new BindingException("Unknown execution method for: " + command.getName());
  }
  if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
    throw new BindingException("Mapper method '" + command.getName() 
        + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
  }
  return result;
}
```

### sqlsession调用Executor



##### 获取MappedStatement并调用Executor

根据参数中statement（接口名+方法名）得到MappedStatement对象（包含了xml中的节点中的信息）并调用Executor

```
  @Override
  public int update(String statement, Object parameter) {
    try {
      dirty = true;
      MappedStatement ms = configuration.getMappedStatement(statement);
      //调用Executor
      return executor.update(ms, wrapCollection(parameter));
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

##### StatementHandler的获取

最终Executor中的方法会进入SimpleExecutor的doUpdate方法，根据MappedStatement中的statementType构建对应的StatementHandler，默认为PreparedStatementHandler

调用newStatementHandler生成StatementHandler对象

```
 @Override
  public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      
      //构建StatementHandler
      StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
      
      //获取Statement，主要表现为从数据源connection获取对应的prepareStatement或者Statement
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.update(stmt);
    } finally {
      closeStatement(stmt);
    }
  }
```

##### Statement的获取

根据StatementHandler构建相应的Statement

上面代码中调用prepareStatement方法对StatementHandler进行修饰，主要表现为从数据源connection获取与StatementHandler对应的prepareStatement或者Statement（包含sql语句与参数的最终组装）

**如PreparedStatementHandler的instantiateStatement方法就是获取对应数据库的Statement返回**

```
 @Override
  protected Statement instantiateStatement(Connection connection) throws SQLException {
  
    //获取sql语句
    String sql = boundSql.getSql();
    if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
      String[] keyColumnNames = mappedStatement.getKeyColumns();
      if (keyColumnNames == null) {
        return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
      } else {
        return connection.prepareStatement(sql, keyColumnNames);
      }
    } else if (mappedStatement.getResultSetType() != null) {
      return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
    } else {
      return connection.prepareStatement(sql);
    }
  }
```



### Executor调用StatementHandler

 调用StatementHandler方法，可以看到方法的参数为上一步获取的数据库Statement对象

```
@Override
 public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
      stmt = prepareStatement(handler, ms.getStatementLog());
      
      //调用StatementHandler方法
      return handler.update(stmt);
    } finally {
      closeStatement(stmt);
    }
  }
```

### StatementHandler执行SQL 并且调用ResultSetHandler 

ResultSetHandler的具体实现类是DefaultResultSetHandler，其实现的步骤就是将Statement执行后的结果集，按照Mapper文件中配置的ResultType或ResultMap来封装成对应的对象，最后将封装的对象返回.

```
@Override
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
  String sql = boundSql.getSql();
  statement.execute(sql);
  return resultSetHandler.<E>handleResultSets(statement);
}
```

## mybatis一级缓存

### 一级缓存原理

使用Hashmap存储。

禁用一级与二级缓存，使用第三方缓存（redis）

每一次非select操作都会清除缓存，delete、update、insert都会进入Executor的update重载方法

```
 @Override
  public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    //清除缓存
    clearLocalCache();
    //执行数据库操作
    return doUpdate(ms, parameter);
  }
```



一级缓存Hashmap的key设计    进行hashcode运算

```
 @Override
  public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    CacheKey cacheKey = new CacheKey();
    cacheKey.update(ms.getId());
    cacheKey.update(rowBounds.getOffset());
    cacheKey.update(rowBounds.getLimit());
    cacheKey.update(boundSql.getSql());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();
    // mimic DefaultParameterHandler logic
    
    //参数名称 如userName
    for (ParameterMapping parameterMapping : parameterMappings) {
      if (parameterMapping.getMode() != ParameterMode.OUT) {
        Object value;
        String propertyName = parameterMapping.getProperty();
        if (boundSql.hasAdditionalParameter(propertyName)) {
          value = boundSql.getAdditionalParameter(propertyName);
        } else if (parameterObject == null) {
          value = null;
        } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
          value = parameterObject;
        } else {
        //获取参数具体值
          MetaObject metaObject = configuration.newMetaObject(parameterObject);
          value = metaObject.getValue(propertyName);
        }
        cacheKey.update(value);
      }
    }
    if (configuration.getEnvironment() != null) {
      // issue #176
      cacheKey.update(configuration.getEnvironment().getId());
    }
    return cacheKey;
  }
```



### 使用一级缓存注意的问题



#### mybatis单独使用时一级缓存

1、两个sqlSession1，sqlSession2  ,在sqlSession1中查询两次，在高并发场景下会发生sqlSession2中修改数据，但sqlSession1中第二次结果与数据库不一致产生脏数据，如下所示。

```
SqlSession sqlSession = sqlSessionFactory.openSession(true);
            //SQL Mapper（映射器）：MyBatis新设计存在的组件，它由一个Java接口和XML文件（或注解）构成，需要给出对应的SQL和映射规则。它负责发送SQL去执行，并返回结果。
            RoleDao roleMapper = sqlSession.getMapper(RoleDao.class);
            List<Role> role = roleMapper.findRolesByUserId("1566036188458575655");
            System.out.println(role.get(0));


            Thread.sleep(2000);
            SqlSession sqlSession1 = sqlSessionFactory.openSession(true);
            RoleDao roleMapper1 = sqlSession1.getMapper(RoleDao.class);
            roleMapper1.update("ADMIN", "1566036188458575655");

            Thread.sleep(2000);
            List<Role> roles = roleMapper.findRolesByUserId("1566036188458575655");
            System.out.println(roles.get(0));
```

2、**同一个sqlSession下**，查询两次，如果更新第一次查询的数据（并不是修改数据库），第二次查询的数据也被修改，产生脏数据，如下。

```
 RoleDao roleMapper = sqlSession.getMapper(RoleDao.class);
            List<Role> role = roleMapper.findRolesByUserId("1566036188458575655");
            System.out.println(role.get(0));
            //更改数据，更改的也是一级缓存中地址中数据
            role.get(0).setRoleName("mybatis一级缓存脏数据");
     
            List<Role> roles = roleMapper.findRolesByUserId("1566036188458575655");
            System.out.println(roles.get(0)); //里面数据被更改
            //产生脏数据
            System.out.println(role == roles); //输出为true 说明上面更改的是一级缓存中地址中数据
            
            sqlSession.close();
            sqlSession1.close();
```

解决：

设置mybatis一级缓存作用范围

<setting name="localCacheScope" value="STATEMENT"/>

每次查询后都清空缓存

```
sqlSession.clearCache();
```

#### spring中的mybatis一级缓存 

1、如果两次查询在一个事务中，则从一级缓存中获取（使用的时同一个sqlSession），有可能出现脏数据。

2、如果两次查询不在一个事务中，则两个查询都是从数据库中查询。

解决方案，禁用一级缓存

```
    //出现脏数据
    
    @Test
    @Transactional
    public void getUser() {
        log.info("输出的list {}" + userDao.findByUserId("1566036188458575655"));
        User user = userDao.findByUserId("1566036188458575655");
        System.out.println(user);
        
        //不修改数据库，仅仅修改第一次查询数据
        user.setUserName("测试一级缓存修改");
        User user1 = userDao.findByUserId("1566036188458575655");
        System.out.println(user==user1); //打印内存地址  输出true
        System.out.println(user1);   //输出的userName为测试一级缓存修改，可见出现了脏数据
    }
```

## 二级缓存原理

**二级缓存开关 cacheEnable（mybatis全部配置中）默认开启，但不是决定性条件，mybatis二级缓存的真正开启条件是mapper中的<cache>标签，当解析到cache标签时，会为mapperStatement对象创建Cache对象，如果没有标签，不会创建对象。**

```

  @Override
  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
      throws SQLException {
    Cache cache = ms.getCache();        获取二级缓存对象
   如果创建了二级缓存对象则会从缓存查数据
    if (cache != null) {
      flushCacheIfRequired(ms);
      if (ms.isUseCache() && resultHandler == null) {
        ensureNoOutParams(ms, parameterObject, boundSql);
        @SuppressWarnings("unchecked")
        List<E> list = (List<E>) tcm.getObject(cache, key);
        if (list == null) {
          list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
          tcm.putObject(cache, key, list); // issue #578 and #116
        }
        return list;
      }
    }
     如果没有创建对象，则会直接执行代理对象的query方法
    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  }
```

key 与一级缓存相同

多个sqlSession共享

查询操作会先把数据存储到TransactionalCacheManager中的TransactionalCache中，TransactionalCache二级缓存只有sqlSession提交过后才会生效，sqlSession提交过后会把TransactionalCache中的数据存储到PerpetualCache对象中，MapperStatement持有PerpetualCache对象。

```
public class TransactionalCacheManager {

  private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();

```

TransactionalCache类存储数据

```
public class TransactionalCache implements Cache {

  private static final Log log = LogFactory.getLog(TransactionalCache.class);

  private final Cache delegate;
  private boolean clearOnCommit;
  private final Map<Object, Object> entriesToAddOnCommit;
  private final Set<Object> entriesMissedInCache;
```

commit操作会把数据转存到真正的二级缓存区域

```
  @Override
  public void commit(boolean required) throws SQLException {
    delegate.commit(required);
    //转存到二级缓存PerpetualCache里
    tcm.commit();
```

对于修改操作会清空二级缓存

```

  @Override
  public void clear() {
  
  //标识符设置为true
    clearOnCommit = true;
    entriesToAddOnCommit.clear();
  }
 //清空真正二级缓存区域
  public void commit() {
    if (clearOnCommit) {
      delegate.clear();
    }
    flushPendingEntries();
    reset();
  }

```

下面第一次和第二次查询数据一样，但是中间有一次修改操作，对于同一个mapper数据是正确的，因为commit操作会清空MapperStatement下的真正二级缓存区域。

但是对于多个和不同的mapper对于关联数据或者同一张表的操作会产生脏数据

```

            SqlSession sqlSession = sqlSessionFactory.openSession(false);
            SqlSession sqlSession1 = sqlSessionFactory.openSession(false);
            SqlSession sqlSession2 = sqlSessionFactory.openSession(false);
            RoleDao roleMapper = sqlSession.getMapper(RoleDao.class);
            RoleDao roleMapper1 = sqlSession1.getMapper(RoleDao.class);
            RoleDao roleMapper2 = sqlSession2.getMapper(RoleDao.class);


            //sqlSession查询
            List<Role> role = roleMapper.findRolesByUserId("1566036188458575655");
            System.out.println("sqlSession第一次查询" + role);
            //sqlSession.commit();
            sqlSession.close();

            //sqlSession1 修改数据
            roleMapper1.update("二级缓存测试", "1566036188458575655");
            //System.out.println("sqlSession1修改之后未提交查询查询" + roleMapper1.findRolesByUserId("1566036188458575655"));
            //sqlSession1.commit();
            sqlSession1.close();
            

            //sqlSession2提交后查询
            List<Role> role2 = roleMapper2.findRolesByUserId("1566036188458575655");
            System.out.println("sqlSession2提交后查询" + role2);
            //sqlSession2.commit();
            sqlSession2.close();
```

## 拦截器





