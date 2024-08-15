# ui.tables 扩展示例

NiceGUI 中 `ui.table` 组件为我们提供了非常好的数据展示形式。

## 1. 使用 UI 组件渲染单元格

在渲染一些表格数据时，为了提高页面美化效果，我们常常希望将某些列渲染成更突出的元素。NiceGUI 官方提供了一个[示例](https://nicegui.io/documentation/table#conditional_formatting)，使用 `ui.badge` 组件渲染 `age` 字段，并根据字段值在颜色上做不同的突出。

![alt text](../images/table_conditional_formatting.png){ width="300"}

```python linenums="1" title="条件格式化"
from nicegui import ui

columns = [
    {'name': 'name', 'label': 'Name', 'field': 'name'},
    {'name': 'age', 'label': 'Age', 'field': 'age'},
    {'name': 'status', 'label': 'Status', 'field': 'status'},
]
rows = [
    {'name': 'Alice', 'age': 18, 'status': True},
    {'name': 'Bob', 'age': 21, 'status': False},
    {'name': 'Carol', 'age': 42, 'status': True},
]
table = ui.table(columns=columns, rows=rows, row_key='name')
table.add_slot('body-cell-age', '''
    <q-td key="age" :props="props">
        <q-badge :color="props.value < 21 ? 'red' : 'green'">
            {{ props.value }}
        </q-badge>
    </q-td>
''')
ui.run()
```

参照此方法，我们尝试下将 `status` 字段通过 `ui.switch` 组件渲染，更突出用户的活跃状态。

```python linenums="1" hl_lines="21-29" title="ui.switch组件渲染status字段"
from nicegui import ui

columns = [
    {'name': 'name', 'label': 'Name', 'field': 'name'},
    {'name': 'age', 'label': 'Age', 'field': 'age'},
    {'name': 'status', 'label': 'Status', 'field': 'status'},
]
rows = [
    {'name': 'Alice', 'age': 18, 'status': True},
    {'name': 'Bob', 'age': 21, 'status': False},
    {'name': 'Carol', 'age': 42, 'status': True},
]
table = ui.table(columns=columns, rows=rows, row_key='name')
table.add_slot('body-cell-age', '''
    <q-td key="age" :props="props">
        <q-badge :color="props.value < 21 ? 'red' : 'green'">
            {{ props.value }}
        </q-badge>
    </q-td>
''')
table.add_slot('body-cell-status', '''
    <q-td key="status" :props="props">
        <q-switch 
            v-model="props.value"
            checked-icon="check" 
            unchecked-icon="clear"
        />
    </q-td>
''')
ui.run()
```

![alt text](../images/render_with_switch.png){ width="300"}

我们发现表格确实正常渲染了，但是这种方式下 `ui.switch` 组件 却无法点击切换状态，我们需要使用 `v-model="props.row.status"` 来绑定到每行 `status` 的值，就可以正常切换状态了。

```python linenums="1" hl_lines="4" title="ui.switch组件渲染status字段"
table.add_slot('body-cell-status', '''
    <q-td key="status" :props="props">
        <q-switch 
            v-model="props.row.status"
            checked-icon="check" 
            unchecked-icon="clear"
        />
    </q-td>
''')
```

???+ example "切换状态后同步数据至数据库"

    如果我们想要功能更便捷，那么可以切换状态后，立马将更改同步至数据中，这里参考 NiceGUI 官方的表格下拉[示例](https://nicegui.io/documentation/table#table_with_drop_down_selection)，稍作改动来实现此功能。

    ```python linenums="1" hl_lines="5-7" title="同步更改后的 status 字段值到数据库中"
    from nicegui import events

    def sync2db_status(e: events.GenericEventArguments) -> None:
        for row in table.rows:
            if row['id'] == e.args['id'] and row['status'] != e.args['status']:
                # 将此条数据会更新到数据库
                user_db_table.update(row)
                row['status'] = e.args['status']
        
        ui.notify(f'Table.rows is now: {table.rows}')

    table.add_slot('body-cell-status', '''
            <q-td key="status" :props="props">
                <q-toggle v-model="props.row.status" 
                    checked-icon="check" unchecked-icon="clear" 
                    @update:model-value="(e) => $parent.$emit('sync2db_status', props.row)"
                />
            </q-td>
        ''')
    ```