---
title: Linux operation
tag: Linux
---

```sh
# list the file numbers under the folder in the current directory
for d in */; do echo "$(find "$d" -type f | wc -l) $d"; done | sort -n
```
