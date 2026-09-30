---
name: vatadmin-crud-generator
description: vatadmin curd 表名 plugin|app
---

***

name: "vatadmin-crud-generator"
description: "基于表结构一键生成vatadmin完整CRUD代码，包括后端控制器、模型、前端视图和菜单配置。当用户需要快速生成完整的CRUD功能时调用。"
------------------------------------------------------------------------------------

# Vatadmin一键CRUD生成器

## 概述

本工具基于Vatadmin的PagesController，帮助用户一键生成完整的CRUD代码。通过分析数据库表结构，自动生成后端控制器、模型、前端视图和菜单配置，大幅减少开发工作量。

命令格式：

**首次生成CRUD**：`vatadmin curd 表名 plugin|app`

```bash
vatadmin curd your_table_name plugin
```

```bash
vatadmin curd your_table_name app
```

**重新同步字段**：`vatadmin recurd 页面ID`

当表结构发生变更（新增字段、修改字段类型或注释等）后，调用此命令根据最新的表结构重新同步字段配置，无需重新创建页面。

```bash
vatadmin recurd 1
```

## 核心原理

Vatadmin的CRUD生成基于以下流程：

1. **表结构分析** - 读取数据库表结构和字段信息
2. **页面配置生成** - 创建vat_pages记录，包含字段配置和页面设置
3. **代码生成** - 根据配置生成后端和前端代码
4. **菜单配置** - 自动创建对应的菜单和权限
5. **字段重新同步（可选）** - 表结构变更后，通过syncField接口按最新表结构重新同步字段配置

### 构建模块

Vatadmin支持两种构建模块：

- **应用模块**（build_module=0）：生成到app目录下，适用于系统核心功能
- **插件模块**（build_module=1）：生成到plugin目录下，适用于插件功能

## 使用方法

### 1. 准备工作

1. **创建数据库表** - 首先创建符合规范的数据库表
2. **确保表结构规范** - 表名建议使用`vat_`前缀，字段名使用下划线命名法
3. **添加表注释和字段注释** - 这些注释会被自动用于生成页面名称和字段标签

### 2. 生成步骤

#### 2.1 第一步：创建页面配置

通过API调用`/basic/pages/add`接口，传入表名参数和可选的tpl_json参数：

**基本用法（生成到应用目录）**：

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d "table=your_table_name&build_module=0"
```

**插件用法（生成到插件目录）**：

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d "table=your_table_name&build_module=1&build_app_name=vatadmin"
```

**高级用法（带自定义tpl_json - 应用模式）**：

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d "table=your_table_name&build_module=0&tpl_json={%22api_list%22:{...},%22fields%22:[...]}"
```

**高级用法（带自定义tpl_json - 插件模式）**：

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d "table=your_table_name&build_module=1&build_app_name=vatadmin&tpl_json={%22api_list%22:{...},%22fields%22:[...]}"
```

**参数说明**：
- `table` - 表名（必填）
- `build_module` - 构建模块（0应用，1插件，默认0）
- `build_app_name` - 应用名称（默认admin）
- `build_menu` - 是否创建菜单（默认0）
- `build_menu_name` - 菜单名称（默认表注释）
- `tpl_json` - 自定义模板配置（可选）

**注意**：
- 请根据实际项目配置修改端口号（默认8787）
- 确保Webman服务已启动
- 确保具有相应的接口调用权限
- tpl_json参数需要进行URL编码

该接口会：

- 分析表结构和字段信息
- 生成字段配置（包括搜索、表单、表格显示等）
- 创建vatpages记录，包含目录和类名信息
- 自动生成驼峰命名的控制器、模型和视图名称
- 根据表名自动计算目录结构（如vat_member_log会生成member目录）
- 如果提供了tpl_json参数，使用自定义配置；否则使用系统自动生成的配置

#### 2.2 第二步：构建代码

通过API调用`/basic/pages/build`接口，传入页面ID和构建选项：

```bash
curl -X POST "http://localhost:8787/basic/pages/build" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "id=1&options[]=controller&options[]=model&options[]=view&options[]=json&options[]=menu"
```

**注意**：
- 请根据实际项目配置修改端口号（默认8787）
- 确保Webman服务已启动

构建选项说明：

- `controller` - 生成后端控制器
- `model` - 生成后端模型
- `view` - 生成前端视图（列表页和编辑页）
- `json` - 生成前端配置文件
- `menu` - 生成菜单和权限

#### 2.3 第三步（可选）：重新同步字段

当数据库表结构发生变更后（例如新增字段、修改字段类型、调整字段注释等），无需重新执行add创建页面，可以通过调用`/basic/pages/syncField`接口，根据最新的表结构重新同步该页面的字段配置。

```bash
curl -X POST "http://localhost:8787/basic/pages/syncField" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d "id=1"
```

**参数说明**：
- `id` - vat_pages表的页面ID（必填，即add成功后返回的页面ID）

**适用场景**：
- 表新增了字段，需要同步到页面配置中
- 字段类型或注释被修改，需要更新字段配置
- 字段配置丢失或异常，需要根据表结构重新生成

**注意**：
- 同步字段后，如需要重新生成代码文件，仍需再次调用`/basic/pages/build`接口
- 请根据实际项目配置修改端口号（默认8787），并确保Webman服务已启动
- 已手动自定义的字段配置可能会被覆盖，操作前请注意备份

### 3. 表结构规范

为了获得最佳生成效果，必须遵循以下表结构规范：

#### 3.1 表名规范

- 必须使用`vat_`前缀
- 使用下划线命名法，如`vat_user`、`vat_product_category`
- **目录生成规则**（自动存储到vat_pages表的build_controller、build_model、build_view字段，动态分析）：
  - 逻辑：移除`vat_`前缀后，按下划线分割表名
  - 所有表名都生成目录结构：第一个单词作为目录，整个表名转换为驼峰命名作为类名
  - 示例：
    - `vat_member` -> 存储值：`member/Member`
    - `vat_member_log` -> 存储值：`member/MemberLog`
    - `vat_brand_city` -> 存储值：`brand/BrandCity`
    - `vat_user` -> 存储值：`user/User`

#### 3.2 必有的字段（建表规范）

| 字段名 | 类型 | 约束 | 默认值 | 注释 |
|-------|------|------|--------|------|
| `id` | `int(11)` | `NOT NULL AUTO_INCREMENT` | - | `ID编号` |
| `status` | `tinyint(1)` | `NOT NULL` | `0` | `状态（0正常，1禁用）` |
| `created_at` | `datetime` | `DEFAULT CURRENT_TIMESTAMP` | 当前时间 | `创建时间` |
| `updated_at` | `datetime` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 当前时间 | `更新时间` |

**注意**：
- 必须设置`PRIMARY KEY (`id`)`
- 能设置`NOT NULL`的字段尽量设置，并添加默认值
- 不能设置默认值的字段不要强制设置

#### 3.3 其他字段规范

| 字段类型   | 命名建议                      | 自动处理           |
| ------ | ------------------------- | -------------- |
| 名称/标题  | `name`、`title`            | 自动设为模糊搜索       |
| 类型/状态  | `type`、`status`           | 自动设为下拉选择       |
| 密码     | `password`                | 自动在表格中隐藏       |
| 图标     | `icon`                    | 自动显示为图标        |
| 头像     | `avatar`                  | 自动显示为头像        |
| 图片     | `image`、`img`、`cover`     | 自动设为图片上传       |
| 多图片    | `imgs`、`images`           | 自动设为多图片上传      |
| JSON字段 | `json`结尾                  | 自动设为JSON编辑器    |
| 日期字段   | `date`类型                  | 自动设为日期选择器      |
| 整数字段   | `int`、`tinyint`等          | 自动设为数字输入       |

#### 3.4 注释规范

- 表注释会作为页面名称和菜单名称
- 字段注释会作为表单标签和表格列标题
- 建议为所有字段添加清晰的注释

## 生成结果

### 1. 后端文件

#### 1.1 控制器

生成路径：`webman/app/{build_app_name}/controller/{build_controller}Controller.php`

示例：

```php
<?php
declare(strict_types=1);

namespace app\admin\controller;

use plugin\vatadmin\app\controller\BaseController;
use app\model\User;

class UserController extends BaseController
{
    protected $model = User::class;
    protected $confine = 0; //如果需要才加
    protected $tableCode = 0; //如果已经有了，需要加，但还是不能一样
}
```

#### 1.2 模型

生成路径：`webman/app/model/{build_model}.php`

示例：

```php
<?php
declare(strict_types=1);

namespace app\model;

use think\Model;

/**
 * 用户表
 * @field id int ID
 * @field name varchar 名称
 * @field email varchar 邮箱
 * @field created_at datetime 创建时间
 */
class User extends Model
{
    protected $table = 'vat_user';
    protected $primaryKey = 'id';
    protected $autoWriteTimestamp = true;
}
```

### 2. 前端文件

#### 2.1 视图文件

生成路径：`vatadmin-naive/src/views/{build_view}/`

- `index.vue` - 列表页
- `edit.vue` - 编辑页

示例（index.vue）：

```vue
<template>
  <div class="page-container">
    <VatPage
      :page-config="pageConfig"
      :api-path="'/admin/user'"
      :has-edit="true"
      :has-delete="true"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import VatPage from '@/components/VatPage.vue'

const pageConfig = ref({
  columns: [
    { key: 'id', title: 'ID', width: 80 },
    { key: 'name', title: '名称', width: 180 },
    { key: 'email', title: '邮箱', width: 180 },
    { key: 'created_at', title: '创建时间', width: 180 },
  ],
  search: [
    { key: 'name', label: '名称', type: 'input' },
    { key: 'email', label: '邮箱', type: 'input' },
  ],
})
</script>
```

#### 2.2 配置文件

生成路径：`vatadmin-naive/src/vat/pages/{table}.json`

包含字段配置、搜索选项、表单设置等信息。

### 3. 菜单配置

自动在`vat_admin_menu`表中创建：

- 主菜单（页面名称）
- 子菜单（列表、添加、编辑、删除等操作）
- 对应的权限路由

## 高级功能

### 1. 自定义 tpl_json 配置

您可以通过 `tpl_json` 参数来自定义页面配置，包括API接口、字段配置、工具按钮等。

#### tpl_json 结构说明

```json
{
  "api_list": {
    "list": {"url": "/app/vatadmin/basic/Controller/list", "method": "get"},
    "edit": {"url": "/app/vatadmin/basic/Controller/edit", "method": "post"},
    "lock": {"url": "/app/vatadmin/basic/Controller/lock", "method": "post"},
    "delete": {"url": "/app/vatadmin/basic/Controller/delete", "method": "post"},
    "import": {"url": "/app/vatadmin/basic/Controller/import", "method": "post"},
    "download": {"url": "/app/vatadmin/basic/Controller/download", "method": "get"}
  },
  "joins": [],
  "select_fields": "*",
  "fields": [
    {
      "field": "id",
      "table_alias": "",
      "alias": "",
      "comment": "ID",
      "type": "int",
      "search": true,
      "table_display": "",
      "table_column": true,
      "table_order": 0,
      "form": false,
      "form_required": false,
      "form_order": 0,
      "sorter": true,
      "condition": "=",
      "search_view": "input",
      "form_view": "input_number",
      "rules": "",
      "width": 80,
      "default": null,
      "config": {}
    }
    // 其他字段配置...
  ],
  "tools": {
    "add": {"show": true, "permission_key": "table_add"},
    "edit": {"show": true, "permission_key": "table_edit"},
    "lock": {"show": false, "permission_key": "table_lock"},
    "unlock": {"show": false, "permission_key": "table_unlock"},
    "delete": {"show": false, "permission_key": "table_delete"},
    "import": {"show": false, "permission_key": "table_import"},
    "batch": {"show": false, "permission_key": "table_batch"},
    "refresh": {"show": true, "permission_key": "table_refresh"},
    "download": {"show": true, "permission_key": "table_download"},
    "search": {"show": true, "permission_key": "table_search"}
  },
  "setting": {}
}
```

#### 字段配置说明

| 字段 | 类型 | 说明 |
|------|------|------|
| field | string | 字段名 |
| table_alias | string | 表别名 |
| alias | string | 字段别名 |
| comment | string | 字段注释 |
| type | string | 字段类型 |
| search | boolean | 是否支持搜索 |
| table_display | string | 表格显示类型（如switch、image、avatar等） |
| table_column | boolean | 是否在表格中显示 |
| table_order | number | 表格列顺序 |
| form | boolean | 是否在表单中显示 |
| form_required | boolean | 是否必填 |
| form_order | number | 表单字段顺序 |
| sorter | boolean | 是否支持排序 |
| condition | string | 搜索条件（如=、like、between等） |
| search_view | string | 搜索组件类型 |
| form_view | string | 表单组件类型 |
| rules | string | 验证规则 |
| width | number | 表格列宽度 |
| default | any | 默认值 |
| config | object | 组件配置 |

#### 使用示例

**示例：创建带自定义配置的会员管理页面**

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d "table=vat_member&tpl_json={%22api_list%22:{%22list%22:{%22url%22:%22/app/vatadmin/basic/Member/list%22,%22method%22:%22get%22},%22edit%22:{%22url%22:%22/app/vatadmin/basic/Member/edit%22,%22method%22:%22post%22},%22lock%22:{%22url%22:%22/app/vatadmin/basic/Member/lock%22,%22method%22:%22post%22},%22delete%22:{%22url%22:%22/app/vatadmin/basic/Member/delete%22,%22method%22:%22post%22},%22import%22:{%22url%22:%22/app/vatadmin/basic/Member/import%22,%22method%22:%22post%22},%22download%22:{%22url%22:%22/app/vatadmin/basic/Member/download%22,%22method%22:%22get%22}},{%22joins%22:[],%22select_fields%22:%22*%22,%22fields%22:[{%22field%22:%22id%22,%22table_alias%22:%22%22,%22alias%22:%22%22,%22comment%22:%22ID%22,%22type%22:%22int%22,%22search%22:true,%22table_display%22:%22%22,%22table_column%22:true,%22table_order%22:0,%22form%22:false,%22form_required%22:false,%22form_order%22:0,%22sorter%22:true,%22condition%22:%22=%22,%22search_view%22:%22input%22,%22form_view%22:%22input_number%22,%22rules%22:%22%22,%22width%22:80,%22default%22:null,%22config%22:{}}],%22tools%22:{%22add%22:{%22show%22:true,%22permission_key%22:%22vat_member_add%22},%22edit%22:{%22show%22:true,%22permission_key%22:%22vat_member_edit%22},%22lock%22:{%22show%22:false,%22permission_key%22:%22vat_member_lock%22},%22unlock%22:{%22show%22:false,%22permission_key%22:%22vat_member_unlock%22},%22delete%22:{%22show%22:false,%22permission_key%22:%22vat_member_delete%22},%22import%22:{%22show%22:false,%22permission_key%22:%22vat_member_import%22},%22batch%22:{%22show%22:false,%22permission_key%22:%22vat_member_batch%22},%22refresh%22:{%22show%22:true,%22permission_key%22:%22vat_member_refresh%22},%22download%22:{%22show%22:true,%22permission_key%22:%22vat_member_download%22},%22search%22:{%22show%22:true,%22permission_key%22:%22vat_member_search%22}},%22setting%22:{}}
```

### 2. 字段类型自动识别

系统会根据字段类型和名称自动设置：

- **整数类型** - 数字输入框
- **日期类型** - 日期选择器
- **状态字段** - 开关或下拉选择
- **图片字段** - 图片上传组件
- **JSON字段** - JSON编辑器

### 2. 数据验证规则

自动根据字段名称生成验证规则：

- `mobile` - 手机号验证
- `email` - 邮箱验证
- `url` - URL验证
- `idcard` - 身份证验证
- `date` - 日期验证

### 3. 搜索条件自动设置

- `name`、`title`字段 - 模糊搜索
- `status`、`type`字段 - 精确匹配
- 日期字段 - 范围搜索

### 4. 表格显示自动配置

- `id`字段 - 固定宽度80px，支持排序
- `status`字段 - 显示为开关
- `avatar`字段 - 显示为头像
- `image`字段 - 显示为图片
- `password`字段 - 表格中隐藏

## 常见问题

### 1. 生成的代码在哪里？

- 后端代码：`webman/app/`目录下
- 前端代码：`vatadmin-naive/src/views/`目录下
- 配置文件：`vatadmin-naive/src/vat/pages/`目录下

### 2. 如何修改生成的代码？

- 后端代码：直接编辑生成的控制器和模型文件
- 前端代码：编辑生成的Vue文件
- 页面配置：通过Vatadmin后台的页面管理功能修改

### 3. 如何添加自定义字段类型？

在`PagesController.php`的`applyFieldRules`方法中添加新的字段规则。

### 4. 如何修改生成的菜单？

- 通过Vatadmin后台的菜单管理功能修改
- 或直接修改`vat_admin_menu`表

## 示例

### 示例1：生成用户管理CRUD

1. **创建表结构**：

```sql
CREATE TABLE `vat_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(255) NOT NULL COMMENT '名称',
  `email` varchar(255) NOT NULL COMMENT '邮箱',
  `mobile` varchar(20) NOT NULL COMMENT '手机号',
  `status` tinyint(1) DEFAULT 1 COMMENT '状态',
  `created_at` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='用户表';
```

1. **创建页面配置**：

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "table=vat_user"
```

1. **构建代码**：

```bash
curl -X POST "http://localhost:8787/basic/pages/build" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "id=1&options[]=controller&options[]=model&options[]=view&options[]=json&options[]=menu"
```

1. **结果**：

- 后端控制器：`webman/app/admin/controller/UserController.php`
- 后端模型：`webman/app/model/User.php`
- 前端视图：`vatadmin-naive/src/views/User/`
- 前端配置：`vatadmin-naive/src/vat/pages/vat_user.json`
- 菜单：自动添加到系统菜单中

### 示例2：生成产品分类CRUD

1. **创建表结构**：

```sql
CREATE TABLE `vat_product_category` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(255) NOT NULL COMMENT '分类名称',
  `parent_id` int(11) DEFAULT 0 COMMENT '父分类ID',
  `sort` int(11) DEFAULT 0 COMMENT '排序',
  `status` tinyint(1) DEFAULT 1 COMMENT '状态',
  `created_at` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='产品分类表';
```

1. **创建页面配置**：

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "table=vat_product_category"
```

1. **构建代码**：

```bash
curl -X POST "http://localhost:8787/basic/pages/build" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "id=2&options[]=controller&options[]=model&options[]=view&options[]=json&options[]=menu"
```

1. **结果**：

- vat_pages表存储值：
  - `build_controller`: `product/ProductCategory`
  - `build_model`: `product/ProductCategory`
  - `build_view`: `product/ProductCategory`
- 后端控制器：`webman/app/admin/controller/product/ProductCategoryController.php`
- 后端模型：`webman/app/model/product/ProductCategory.php`
- 前端视图：`vatadmin-naive/src/views/product/ProductCategory/`
- 前端配置：`vatadmin-naive/src/vat/pages/vat_product_category.json`
- 菜单：自动添加到系统菜单中

### 示例3：生成会员日志CRUD（带目录生成）

1. **创建表结构**：

```sql
CREATE TABLE `vat_member_log` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'ID编号',
  `member_id` int(11) NOT NULL DEFAULT 0 COMMENT '会员ID',
  `action` varchar(50) NOT NULL COMMENT '操作类型',
  `content` text COMMENT '操作内容',
  `ip` varchar(20) NOT NULL COMMENT '操作IP',
  `status` tinyint(1) NOT NULL DEFAULT 0 COMMENT '状态（0正常，1禁用）',
  `created_at` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='会员日志表';
```

1. **创建页面配置**：

```bash
curl -X POST "http://localhost:8787/basic/pages/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "table=vat_member_log"
```

1. **构建代码**：

```bash
curl -X POST "http://localhost:8787/basic/pages/build" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "id=3&options[]=controller&options[]=model&options[]=view&options[]=json&options[]=menu"
```

1. **结果**：

- vat_pages表存储值：
  - `build_controller`: `member/MemberLog`
  - `build_model`: `member/MemberLog`
  - `build_view`: `member/MemberLog`
- 后端控制器：`webman/app/admin/controller/member/MemberLogController.php`
- 后端模型：`webman/app/model/member/MemberLog.php`
- 前端视图：`vatadmin-naive/src/views/member/MemberLog/`
- 前端配置：`vatadmin-naive/src/vat/pages/vat_member_log.json`
- 菜单：自动添加到系统菜单中

## 总结

Vatadmin的一键CRUD生成功能大大简化了开发流程，通过遵循表结构规范，可以快速生成完整的前后端代码。生成的代码包含了基本的CRUD操作、数据验证、权限控制等功能，开发者可以在此基础上进行定制和扩展。

使用本工具可以：

- 减少70%以上的重复代码编写工作
- 确保代码结构的一致性和规范性
- 快速构建功能完整的管理界面
- 专注于业务逻辑的实现，而非基础代码的编写