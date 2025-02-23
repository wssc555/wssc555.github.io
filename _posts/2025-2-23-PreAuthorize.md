---
title: @PreAuthorize注解自定义
tags: Java

---
# @PreAuthorize注解自定义

#### 写在前面

@[PreAuthorize](https://so.csdn.net/so/search?q=PreAuthorize&spm=1001.2101.3001.7020)注解会在方法执行前进行权限验证，支持Spring EL表达式，它是基于方法注解的权限解决方案。只有当@EnableGlobalMethodSecurity(prePostEnabled=true)的时候，@PreAuthorize才可以使用，@EnableGlobalMethodSecurity注解在SPRING安全中心进行设置，如下：

```java
/**
 * SPRING安全中心
 * @author ROCKY
 */
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}

```

#### 如何使用

###### ①. 注解如何使用？

```java
/**
 * 删除用户
 */
@PreAuthorize("@ss.hasPermi('system:user:remove')")
@Log(title = "用户管理", businessType = BusinessType.DELETE)
@DeleteMapping("/{userIds}")
public Result remove(@PathVariable Long[] userIds) {
    if (ArrayUtils.contains(userIds, getUserId())) {
        return error("当前用户不能删除");
    }
    return toAjax(userService.deleteUserByIds(userIds));
}
123456789101112
```

###### ②. 自定义权限实现

@PreAuthorize(“@ss.hasPermi(‘system:user:remove’)”)的意思是什么？

A. ss 是一个注册在 Spring容器中的BEAN，对应的类是cn.hadoopx.framework.web.service.PermissionService；
B. hasPermi 是PermissionService类中定义的方法；
C.当Spring EL 表达式返回TRUE，则权限校验通过；
D. PermissionService.java的定义如下：

```java
/**
 * 自定义权限实现，ss取自SpringSecurity首字母
 * @author ROCKY
 */
@Service("ss")
public class PermissionService {
    /**
     * 所有权限标识
     */
    private static final String ALL_PERMISSION = "*:*:*";

    /**
     * 管理员角色权限标识
     */
    private static final String SUPER_ADMIN = "admin";
    private static final String ROLE_DELIMETER = ",";
    private static final String PERMISSION_DELIMETER = ",";

    /**
     * 验证用户是否具备某权限
     * @param permission 权限字符串
     * @return 用户是否具备某权限
     */
    public boolean hasPermi(String permission) {
        if (StringUtils.isEmpty(permission)) {
            return false;
        }
        LoginUser loginUser = SecurityUtils.getLoginUser();
        if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getPermissions())) {
            return false;
        }
        return hasPermissions(loginUser.getPermissions(), permission);
    }

    /**
     * 验证用户是否不具备某权限，与 hasPermi逻辑相反
     * @param permission 权限字符串
     * @return 用户是否不具备某权限
     */
    public boolean lacksPermi(String permission) {
        return hasPermi(permission) != true;
    }

    /**
     * 验证用户是否具有以下任意一个权限
     * @param permissions 以 PERMISSION_NAMES_DELIMETER 为分隔符的权限列表
     * @return 用户是否具有以下任意一个权限
     */
    public boolean hasAnyPermi(String permissions) {
        if (StringUtils.isEmpty(permissions)) {
            return false;
        }
        LoginUser loginUser = SecurityUtils.getLoginUser();
        if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getPermissions())) {
            return false;
        }
        Set<String> authorities = loginUser.getPermissions();
        for (String permission : permissions.split(PERMISSION_DELIMETER)) {
            if (permission != null && hasPermissions(authorities, permission)) {
                return true;
            }
        }
        return false;
    }

    /**
     * 判断用户是否拥有某个角色
     * @param role 角色字符串
     * @return 用户是否具备某角色
     */
    public boolean hasRole(String role) {
        if (StringUtils.isEmpty(role)) {
            return false;
        }
        LoginUser loginUser = SecurityUtils.getLoginUser();
        if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getUser().getRoles())) {
            return false;
        }
        for (SysRole sysRole : loginUser.getUser().getRoles()) {
            String roleKey = sysRole.getRoleKey();
            if (SUPER_ADMIN.equals(roleKey) || roleKey.equals(StringUtils.trim(role))) {
                return true;
            }
        }
        return false;
    }

    /**
     * 验证用户是否不具备某角色，与 isRole逻辑相反。
     * @param role 角色名称
     * @return 用户是否不具备某角色
     */
    public boolean lacksRole(String role) {
        return hasRole(role) != true;
    }

    /**
     * 验证用户是否具有以下任意一个角色
     * @param roles 以 ROLE_NAMES_DELIMETER 为分隔符的角色列表
     * @return 用户是否具有以下任意一个角色
     */
    public boolean hasAnyRoles(String roles) {
        if (StringUtils.isEmpty(roles)) {
            return false;
        }
        LoginUser loginUser = SecurityUtils.getLoginUser();
        if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getUser().getRoles())) {
            return false;
        }
        for (String role : roles.split(ROLE_DELIMETER)) {
            if (hasRole(role)) {
                return true;
            }
        }
        return false;
    }

    /**
     * 判断是否包含权限
     * @param permissions 权限列表
     * @param permission  权限字符串
     * @return 用户是否具备某权限
     */
    private boolean hasPermissions(Set<String> permissions, String permission) {
        return permissions.contains(ALL_PERMISSION) || permissions.contains(StringUtils.trim(permission));
    }
}

```

###### 