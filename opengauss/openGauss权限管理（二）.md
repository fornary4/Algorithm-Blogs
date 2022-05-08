# openGauss权限管理（二）

### 基础权限控制模型

**常见的权限控制模型有三种：**<font color = red>基于策略的访问控制模型，基于角色的访问控制模型以及基于会话和角色的访问控制模型。</font>openGauss数据库采用基于角色的权限访问控制模型，利用角色来组织和管理权限，能够大大简化对权限的授权管理。借助角色机制，当给一组权限相同的用户授权时，只需将权限授予角色，再将角色授予这组用户即可，不需要对用户逐一授权。而且利用角色权限分离可以很好地控制不同用户拥有不同的权限，相互制约达到平衡。

伴随着数据库的发展以及所面向业务场景的扩展，对数据库权限分离以及权限管理的细粒度划化提出了更高的要求，为了满足多样化用户的业务安全要求，openGauss数据库针对权限模型进行了更细粒度的权限划分，使得用户可以更灵活地依据实际业务进行用户权限分配和管理。

### **openGauss数据库权限层级**

在openGauss数据库系统的对象布局逻辑结构中，每个实例下允许创建多个数据库（database），每个数据库下允许创建多个模式（schema），每个模式下允许创建多个对象，比如表、函数、视图、索引等，每个表又可以依据行和列两个维度进行衡量，从而形成如下的逻辑层级：



### **openGauss数据库权限分类**

在openGauss数据库系统中权限分为两种：系统权限和对象权限。

- 系统权限是指系统规定用户使用数据库的权限，比如登录数据库、创建数据库、创建用户/角色、创建安全策略等。
- 对象权限是指在数据库、模式、表、视图、函数等数据库对象上执行特殊动作的权限，不同的对象类型与不同的权限相关联，比如数据库的连接权限，表的查看、更新、插入等权限，函数的执行权限等。基于特定的对象来描述对象权限才是有意义的。

### 授权操作示例

将对表tbl进行select的权限以及将select再赋权的权限授予用户user1

```sql
GRANT select ON TABLE tbl TO user1 WITH GRANT OPTION;
```

赋权后用户user1有权对tbl执行select操作且user1有权限将select权限再赋予其他用户

```sql
GRANT alter, drop ON TABLE tbl TO user1;
```

撤销用户user1对表tbl进行select的权限

```sql
REVOKE select ON tbl FROM user1;
```

### 权限管理部分源码解析

**授予连接权限**

```c++
void StreamNodeGroup::grantStreamConnectPermission()
{
    bool found = false;
    AutoMutexLock streamLock(&m_streamConnectSyncLock);

    HOLD_INTERRUPTS(); /* 双重安全保障 */
    streamLock.lock();
    StreamConnectSyncElement* element =
        (StreamConnectSyncElement*)hash_search(m_streamConnectSyncTbl, &u_sess->debug_query_id, HASH_ENTER, &found); //同步信号

    if (!found) {
        element->key = u_sess->debug_query_id;//没有找到用户的情况
    } else {
        streamLock.unLock();
        ereport(ERROR,
            (errmodule(MOD_STREAM),
                errcode(ERRCODE_STREAM_DUPLICATE_QUERY_ID),
                errmsg("Distribute query failed due to duplicate query id")));
    }
    streamLock.unLock();
    RESUME_INTERRUPTS();//授予权限
}
```



**收回连接权限**

```c++
void StreamNodeGroup::revokeStreamConnectPermission()
{
    AutoMutexLock streamLock(&m_streamConnectSyncLock);

    HOLD_INTERRUPTS(); /* 增加这个宏是为了双重安全 */
    streamLock.lock(); // 锁定流权限
    (void)hash_search(m_streamConnectSyncTbl, &u_sess->debug_query_id, HASH_REMOVE, NULL);
    streamLock.unLock();
    RESUME_INTERRUPTS();
}
```

