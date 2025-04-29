# SQL Converter 工具

## 简介

SQL Converter 是一款用于跨数据库SQL语句转换的Java工具，支持多种主流数据库类型（MySQL、Oracle、Dameng等），可将源数据库的SQL语句转换为目标数据库兼容的语法格式。适用于数据迁移、跨数据库开发等场景。

## 功能特性

- **多数据库支持** 支持 MySQL、Oracle、Dameng 等数据库类型互转
- **SQL类型识别** 自动区分 DML（INSERT/UPDATE/DELETE）和 DDL（CREATE/ALTER/DROP/TRUNCATE/COMMENT）
- **异构DDL转换** 提取表结构元数据，生成目标数据库兼容的DDL
- **配置灵活** 支持大小写敏感、目标库命名规则等高级配置
- **统一接口** 提供简洁的API接口，快速集成到Java项目

## 使用说明

### 1. 核心转换接口

```java
public static List<String> convertSql(
    String sql, 
    DataBaseType sourceDBType, 
    DataBaseType targetDBType,
    String targetSchema, 
    String targetTable, 
    HashMap<String,String> config
) throws Exception
```

**参数说明**

| 参数         | 类型                   | 必选 | 说明                                          |
| ------------ | ---------------------- | ---- | --------------------------------------------- |
| sql          | String                 | 是   | 原始SQL语句                                   |
| sourceDBType | DataBaseType           | 是   | 源数据库类型（枚举值：MYSQL/ORACLE/DAMENG等） |
| targetDBType | DataBaseType           | 是   | 目标数据库类型                                |
| targetSchema | String                 | 是   | 目标模式名（如Oracle的Schema）                |
| targetTable  | String                 | 是   | 目标表名                                      |
| config       | HashMap<String,String> | 否   | 高级配置参数（见下方配置说明）                |

**返回值** 返回转换后的SQL语句列表（可能包含多条语句）

### 2. 通过模板生成CREATE语句

```java
public static List<String> convertCreateByTable(
    Template template, 
    String sourceDBType, 
    String targetDBType, 
    String targetSchema, 
    String targetTable, 
    HashMap<String,String> config
) throws Exception
```

适用于从预定义的表结构模板生成DDL

### 3. SQL类型识别接口

```java


public static SQLType getTypeBySQL(String sql, String sourceDBType) throws Exception
```

**返回值** 返回SQLType枚举：DML/CREATE/ALTER/DROP/TRUNCATE/COMMENT

## 配置参数

通过`config` HashMap传递高级参数：

| 参数键            | 示例值  | 说明                                                         |
| ----------------- | ------- | ------------------------------------------------------------ |
| caseSensitive     | "true"  | 是否启用大小写敏感（默认false）                              |
| targetDefaultCase | "upper" | caseSensitive为false时设置目标库标识符默认大小写规则（upper/lower） |

示例配置：

```
JavaHashMap<String, String> config = new HashMap<>();
config.put("caseSensitive", "true");
config.put("targetDefaultCase", "upper");
```

## 使用示例

### 示例1：转换ALTER语句

```java
String sql = "ALTER TABLE `test`.`employees` DROP COLUMN `phone`;";
HashMap<String, String> config = new HashMap<>();
config.put("caseSensitive", "false");

List<String> converted = SQLConverter.convertSql(
    sql, 
    DataBaseType.MYSQL, 
    DataBaseType.ORACLE, 
    "DXP", 
    "EMPLOYEES", 
    config
);

// 输出结果示例：
// ALTER TABLE DXP.EMPLOYEES DROP COLUMN PHONE;
```

### 示例2：识别SQL类型

```java
SQLType type = SQLConverter.getTypeBySQL(
    "TRUNCATE TABLE orders", 
    "MYSQL"
);
// 返回 SQLType.TRUNCATE
```

## 注意事项

1. **DDL转换限制** 部分数据库特有的DDL语法（如存储过程）可能无法完全转换
2. **异常处理** 遇到无法解析的语法会抛出Exception，建议使用try-catch块捕获

