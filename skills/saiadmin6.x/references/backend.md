# Backend Development Rules

## Directory Structure (`server/plugin/<plugin_name>`)

| Directory/File | Description |
| :--- | :--- |
| `app/admin/` | Management panel logic (Controllers, Logic). |
| `app/api/` | External interface logic (Controllers, Logic). |
| `app/model/` | Shared data models. |
| `app/validate/` | Shared data validators. |

## Common Patterns (Model & Validate)

### 1. Model (`app/model/<module>/<Name>.php`)
Reference: `server/plugin/saiadmin/utils/code/stub/saiadmin/php/model.stub`

Standard models MUST extend `plugin\saiadmin\basic\think\BaseModel` and define table/pk.

**Key Features:**
- **Search Scopes**: Use `search<Field>Attr` for complex query handling (like, in, between, etc.)
- **Simple Equality**: No need to add searchers for simple `=` queries (handled by BaseLogic automatically)
- **Casting/Accessors**: Handle JSON fields or complex types.
- **Relations**: Define `belongsTo`, `hasMany` etc.

```php
<?php
namespace plugin\myplugin\app\model\test;

use plugin\saiadmin\basic\think\BaseModel;

class MyModel extends BaseModel
{
    protected $pk = 'id';
    protected $table = 'my_table';

    // Search Scope: Like (complex query)
    public function searchKeywordsAttr($query, $value)
    {
        $query->where('name', 'like', '%' . $value . '%');
    }
    
    // Note: No need for simple equality searches like status, parent_id
    // BaseLogic handles these automatically: status=1, parent_id=5, etc.
    
    // Accessor: JSON to Array
    public function getImagesAttr($value)
    {
        return json_decode($value ?? '', true);
    }

    // Mutator: Array to JSON
    public function setImagesAttr($value)
    {
        return json_encode($value, JSON_UNESCAPED_UNICODE);
    }
}
```

### 2. Validate (`app/validate/<module>/<Name>Validate.php`)
Reference: `server/plugin/saiadmin/utils/code/stub/saiadmin/php/validate.stub`

Standard validators extend `BaseValidate` (or `think\Validate`).

```php
<?php
namespace plugin\myplugin\app\validate\test;

use plugin\saiadmin\basic\BaseValidate;

class MyValidate extends BaseValidate
{
    protected $rule = [
        'name' => 'require|max:25',
    ];

    protected $message = [
        'name.require' => 'Name is required',
    ];

    protected $scene = [
        'save' => ['name'],
        'update' => ['name'],
    ];
}
```

---

## Pattern A: Regular Model (List/CRUD)

Use this pattern for standard flat data lists.

### 1. Controller (`.../controller/...`)
- **Index**: Calls `$this->logic->search()` and `$this->logic->getList()`.
- **Permissions**: Use `#[Permission('Name', 'plugin:module:controller:action')]` for all actions.
- **Import**: MUST include `use plugin\saiadmin\service\Permission;`.

```php
    use plugin\saiadmin\service\Permission;

    /**
     * Data List
     */
    #[Permission('MyLogic List', 'plugin:test:mylogic:index')]
    public function index(Request $request): Response
    {
        $where = $request->more([
            ['keywords', ''],
            ['status', ''],
        ]);
        $query = $this->logic->search($where);
        $data = $this->logic->getList($query);
        return $this->success($data);
    }

    /**
     * Read Data
     */
    #[Permission('MyLogic Read', 'plugin:test:mylogic:read')]
    public function read(Request $request): Response
    {
        $id = $request->input('id', '');
        $model = $this->logic->read($id);
        return $this->success($model);
    }

    /**
     * Save Data
     */
    #[Permission('MyLogic Save', 'plugin:test:mylogic:save')]
    public function save(Request $request): Response
    {
        $data = $request->post();
        $this->validate('save', $data);
        return $this->logic->add($data) ? $this->success('Saved') : $this->fail('Failed');
    }

    /**
     * Update Data
     */
    #[Permission('MyLogic Update', 'plugin:test:mylogic:update')]
    public function update(Request $request): Response
    {
        $data = $request->post();
        $this->validate('update', $data);
        return $this->logic->edit($data['id'], $data) ? $this->success('Updated') : $this->fail('Failed');
    }

    /**
     * Destroy Data
     */
    #[Permission('MyLogic Destroy', 'plugin:test:mylogic:destroy')]
    public function destroy(Request $request): Response
    {
        $ids = $request->post('ids', '');
        return $this->logic->destroy($ids) ? $this->success('Deleted') : $this->fail('Failed');
    }
```

### 2. Logic (`.../logic/...`)
- Extends `BaseLogic`.
- Standard `add`, `edit`, `destroy` implementation.

```php
namespace plugin\myplugin\app\admin\logic\test;

use plugin\saiadmin\basic\think\BaseLogic;
use plugin\myplugin\app\model\test\MyModel;

class MyLogic extends BaseLogic
{
    public function __construct()
    {
        $this->model = new MyModel();
    }
    
    // Use BaseLogic's default add/edit/destroy or override if needed
    // Ensure Db::startTrans() is used if overriding
}
```

---

## Pattern B: Tree Model (Hierarchy)

Use this pattern for hierarchical data (e.g., categories, departments, menus).

### 1. Controller (`.../controller/...`)
- **Index**: Calls `$this->logic->tree($where)` instead of `getList`.
- **Permissions**: Standard 5 permissions (Index, Read, Save, Update, Destroy).

```php
    /**
     * Tree List
     */
    #[Permission('Tree List', 'plugin:test:tree:index')]
    public function index(Request $request): Response
    {
        $where = $request->more([
            ['keywords', ''],
        ]);
        // Use tree method for hierarchical data
        $data = $this->logic->tree($where);
        return $this->success($data);
    }
    
    // Implement read, save, update, destroy with #[Permission] as above...
```

### 2. Logic (`.../logic/...`)
- **Tree Generation**: Implements `tree` method using `Helper::makeTree`.
- **Edit Validation**: Prevents setting parent to self.
- **Delete Validation**: Prevents deleting items with children.

```php
namespace plugin\myplugin\app\admin\logic\test;

use plugin\saiadmin\basic\think\BaseLogic;
use plugin\saiadmin\utils\Helper;
use plugin\saiadmin\exception\ApiException;
use plugin\myplugin\app\model\test\MyTreeModel;

class MyTreeLogic extends BaseLogic
{
    public function __construct()
    {
        $this->model = new MyTreeModel();
    }

    // 1. Validation in Edit
    public function edit($id, $data): mixed
    {
        if (isset($data['parent_id']) && $data['parent_id'] == $id) {
            throw new ApiException('Cannot set parent to self');
        }
        if (!isset($data['parent_id'])) {
            $data['parent_id'] = 0;
        }
        if ($data['parent_id'] == $data['id']) {
            throw new ApiException('不能设置父级为自身');
        }
        return parent::edit($id, $data);
    }

    // 2. Validation in Destroy
    public function destroy($ids): bool
    {
        $count = $this->model->whereIn('parent_id', $ids)->count();
        if ($count > 0) {
            throw new ApiException('该分类下存在子分类，请先删除子分类');
        }
        return parent::destroy($ids);
    }

    // 3. Tree Structure Generation
    public function tree($where)
    {
        $query = $this->search($where);
        $request = request();
        if ($request && $request->input('tree', 'false') === 'true') {
            $query->field('id, id as value, name as label, parent_id');
        }
        $data = $this->getAll($query);
        return Helper::makeTree($data);
    }
}
```
