---
title: Ripgrep and Grep and smart operations
tag: Linux
---

```sh
rg -i fidelity -g '!imgs/'                 # case-insensitive
rg fidelity -g '!imgs/' -g '!img/'         # skip several folder names
rg fidelity -g '!imgs/' path/to/dir        # search a specific directory`
```


