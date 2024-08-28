# NiceGUI 教程


```python title="ref.py"

class Ref:
    def __init__(self, **kwargs):
        self._subject = Subject()
        self._dict = {}

        for k, v in kwargs.items():
            self.set_value(k, v)

    def __getattr__(self, key):
        # 允许通过 a.key 访问属性值
        if key in self._dict:
            return self.get_value(key)

        raise AttributeError(f"No attribute named '{key}' exists.")

    def __setattr__(self, key, value):
        # 允许通过 a.key = value 设置属性值
        if key.startswith("_"):  # 处理私有属性
            super().__setattr__(key, value)
        else:
            self.set_value(key, value)

    def __str__(self):
        return f"{self.__class__.__name__}(" + ", ".join(f"{k}={v}" for k, v in self._dict.items()) + ")"

    def set_value(self, key, value):
        old_value = self._dict.get(key)
        self._dict[key] = value

        if old_value != value:
            self._subject.on_next((key, old_value, value))

    def update_from_dict(self, d: dict):
        for k, v in d.items():
            self.set_value(k, v)

    def get_value(self, key):
        return self._dict.get(key)

    def remove(self, key):
        if key in self._dict:
            old_value = self._dict.pop(key)
            self._subject.on_next((key, old_value, None))

    def get_keys(self):
        return self._dict.keys()

    def subscribe(self, on_next):
        return self._subject.subscribe(on_next)
```


```python title="button.py"
from nicegui import ui
from ref import Ref
from typing import Any


class Button(ui.button):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.ref = None

    def bind_ref(self, ref: Ref):
        if isinstance(ref, Ref):
            self.ref = ref
            self.ref.subscribe(self.bind_prop)

            for k, v in self.ref._dict.items():
                self.props(f"{k}={v}")

        return self

    def bind_prop(self, *args):
        for k, ov, nv in args:
            if k == "text":
                self._props["label"] = nv
            else:
                self._props[k] = nv

            self.update()

        return self


@ui.page("/")
def index():
    btn_ref = Ref(text="amy", icon="list")
    btn = Button(
        text="abc",
        on_click=lambda: (btn_ref.update_from_dict({"size": "lg", "color": "red", "icon": "home"}), print(btn_ref))
    ).bind_ref(btn_ref)

    btn2 = ui.button(on_click=lambda: btn_ref.set_value("text", "two"))


ui.run()

```