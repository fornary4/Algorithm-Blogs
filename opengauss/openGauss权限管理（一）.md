# openGauss权限管理（一）

在大型数据库中，为了保证数据的安全性，数据管理员需要为每个用户赋予不同的权限，以满足不同用户的需求

<font color = red>数据库的重要特点之一就是它的安全性，而安全性的实现则需要复杂的权限管理系统支持。</font>

openGauss数据库管理系统支持对数据库、表、字段等进行不同数据区域的`权限管理`，权限又分为查询权限、修改权限、删除权限等等不同的详细类别的精细化管理。可以说openGauss数据库是一个数据权限管理非常完备的系统，能够实现对不同用户的数据权限做到精准控制。

### 数据库权限操作场景

- 针对不同用户，可以通过权限设置分配不同的数据库，不同用户之间的作业效率互不影响，保障作业性能
- 管理员用户和数据库的所有者拥有所有权限，不需要进行权限设置且其他用户无法修改其数据库权限。

### 权限管理示例

1. **登陆服务器**： 当某个用户在客户端通过登陆命令连接服务器时，从表user中取出host、user、password进行验证，验证通过则允许登陆连接。
2. **操作数据**：某个用户通过一条命令使用数据时，按照表user、db、tables_priv、columns_priv的顺序进行查询分析。具体流程是，先检查全局权限表user，如果表user中对应的权限为Y，则此用户对所有数据库的权限都为Y，将不再检查表db, tables_priv,columns_priv。如果user中对应的权限为N，则到db表中检查此用户对应的具体数据库，并得到db中为Y的权限，如果db中为N，则检查tables_priv中此数据库对应的具体表，取得表中的权限，以此类推。

### openGauss用户权限管理

在user.cpp中，定义了用户的管理与常用操作，也包含了一些权限操作

例如grant_nodegroup_to_role是将节点组的所有权限授予角色

```c++
static void grant_nodegroup_to_role(Oid groupoid, Oid roleid, bool is_grant)
{
    char in_redis; //保存在redis中的标识
    grantNodeGroupToRole(groupoid, roleid, ACL_ALL_RIGHTS_NODEGROUP, is_grant);//进行授权操作
    in_redis = get_pgxc_group_redistributionstatus(groupoid);
    if (in_redis == PGXC_REDISTRIBUTION_SRC_GROUP) {
        Oid dest_group = PgxcGroupGetRedistDestGroupOid();
        if (OidIsValid(dest_group)) { //检查用户的合法性，然后授予权限
            grantNodeGroupToRole(dest_group, roleid, ACL_ALL_RIGHTS_NODEGROUP, is_grant);
        }
    }
}
```

