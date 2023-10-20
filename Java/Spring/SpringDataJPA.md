# Spring Data JPA

## Spring Data Repository
Spring-Data-Repository的目标是显著减少实现各种数据持久化访问层的样板代码。

#### 1. 核心概念

Spring Data repository 抽象的核心接口是 **Repository\<T, ID>**，他接收实体类和实体类参数的ID来进行管理。这个接口主要作为标记接口，来捕获使用的类型以及发现这个接口的扩展接口。**CrudRepository**接口提供了复杂的功能来管理实体类。
```
public interface CrudRepository<T,ID> extends Repository<T, ID>{
    <S extends T> S save(S entity);
    
    Optional<T> findById(ID primaryKey);
    
    Iterable<T> findAll();
    
    long count();
    
    void delete(T entity);
    
    boolean existsById(ID primaryKey);
}
```
