# Database Table Standards

All plugin tables MUST follow these standards:

## Primary Key
- Every table MUST have `id` as the primary key
- Type: `int(11) unsigned NOT NULL AUTO_INCREMENT`

## Required Standard Fields
All tables MUST include the following fields:

```sql
`status` tinyint(1) unsigned DEFAULT '1' COMMENT '状态',
`created_by` int(11) DEFAULT NULL COMMENT '创建者',
`updated_by` int(11) DEFAULT NULL COMMENT '更新者',
`create_time` datetime DEFAULT NULL COMMENT '创建时间',
`update_time` datetime DEFAULT NULL COMMENT '修改时间',
`delete_time` datetime DEFAULT NULL COMMENT '删除时间',
```

## Complete Table Example

```sql
CREATE TABLE `sa_example_table` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `name` varchar(100) NOT NULL COMMENT '名称',
  `description` text COMMENT '描述',
  `status` tinyint(1) unsigned DEFAULT '1' COMMENT '状态',
  `created_by` int(11) DEFAULT NULL COMMENT '创建者',
  `updated_by` int(11) DEFAULT NULL COMMENT '更新者',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '修改时间',
  `delete_time` datetime DEFAULT NULL COMMENT '删除时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='示例表';
```

## Field Descriptions

| Field | Type | Description |
|:---|:---|:---|
| `status` | tinyint(1) | Record status (1=enabled, 0=disabled) |
| `created_by` | int(11) | User ID who created the record |
| `updated_by` | int(11) | User ID who last updated the record |
| `create_time` | datetime | Record creation timestamp |
| `update_time` | datetime | Record last update timestamp |
| `delete_time` | datetime | Soft delete timestamp (NULL=not deleted) |

## Naming Conventions

- Table names: Use `sa_<plugin_name>_<table_name>` format
- Field names: Use lowercase with underscores (snake_case)
- All fields MUST have COMMENT for documentation
- Use appropriate data types and lengths based on actual needs

## Field Type Standards

### Image Fields
- Image fields (cover, avatar, thumbnail, etc.) MUST use `varchar(1000)`
- Supports storing multiple image URLs or long CDN paths
- Examples: `cover_image`, `avatar`, `thumbnail`, `banner_image`

### Boolean Fields
- Boolean fields (is_xxx) MUST use `tinyint(1) unsigned`
- Value definition: `1` = Yes/True, `2` = No/False
- Default value can be `1` (Yes) or `2` (No) based on business logic
- Examples: `is_top`, `is_recommend`, `is_published`, `is_enabled`

## Menu Insert Standards

When creating plugin menus, follow this hierarchical structure:

### Menu Structure
1. **Plugin Root Menu** (type=1): Main plugin entry
2. **Management Center** (type=2): Management section
3. **Feature Modules** (type=2): Specific feature pages
4. **Permissions** (type=3): CRUD operation permissions

### Menu Insert Template
```sql
-- Plugin Root Menu
INSERT INTO `sa_system_menu` VALUES(NULL, 0, 'PluginName', 'PluginName', '', 1, '/plugin-path', '', NULL, 'plugin-icon', 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
SET @id := LAST_INSERT_ID();

-- Management Center
INSERT INTO `sa_system_menu` VALUES(NULL, @id, '管理中心', 'PluginManage', '', 2, 'manage', '', NULL, 'ri:command-fill', 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
SET @parent_id := LAST_INSERT_ID();

-- Feature Module
INSERT INTO `sa_system_menu` VALUES(NULL, @parent_id, 'ModuleName', 'plugin/module/path', '', 2, 'module/path', '/plugin/pluginname/module/path/index', NULL, 'module-icon', 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
SET @parent_one := LAST_INSERT_ID();

-- CRUD Permissions
INSERT INTO `sa_system_menu` VALUES(NULL, @parent_one, '列表', NULL, 'plugin:module:controller:index', 3, NULL, NULL, NULL, NULL, 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
INSERT INTO `sa_system_menu` VALUES(NULL, @parent_one, '保存', NULL, 'plugin:module:controller:save', 3, NULL, NULL, NULL, NULL, 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
INSERT INTO `sa_system_menu` VALUES(NULL, @parent_one, '更新', NULL, 'plugin:module:controller:update', 3, NULL, NULL, NULL, NULL, 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
INSERT INTO `sa_system_menu` VALUES(NULL, @parent_one, '读取', NULL, 'plugin:module:controller:read', 3, NULL, NULL, NULL, NULL, 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
INSERT INTO `sa_system_menu` VALUES(NULL, @parent_one, '删除', NULL, 'plugin:module:controller:destroy', 3, NULL, NULL, NULL, NULL, 100, '', 2, 2, 2, 2, 2, 0, 'plugin\pluginname\app\api\controller\IndexController', 1, '', 1, 1, '2026-01-01 11:11:11', '2026-01-01 11:11:11', NULL);
```

### Field Descriptions
- `parent_id`: Parent menu ID (0 for root, use variables for hierarchy)
- `title`: Menu display name
- `name`: Menu identifier/route name
- `path`: Frontend route path
- `component`: Frontend component path
- `redirect`: Redirect path (optional)
- `meta`: Menu metadata (JSON)
- `type`: Menu type (1=directory, 2=menu, 3=button/permission)
- `icon`: Menu icon
- `orderNum`: Sort order
- `viewPath`: Backend view path
- `keepalive`: Keep alive status
- `hidden`: Hidden status
- `status`: Enable status
