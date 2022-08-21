# pyeval.py



## 程式碼詳解

### import module

```python
import logging
from datetime import datetime

from quark import config
from quark.core.struct.registerobject import RegisterObject
from quark.core.struct.tableobject import TableObject
```

根據使用情境 import module 可分為兩個部分
1. log 訊息所需之 module
2. register table 所需之 module (RegisterObject, TableObject)

以下將逐一說明。

#### 1. log 訊息所需之 module

`import logging` 
做為處理 log 訊息之 module，用處在於追蹤 register table 建構過程所紀錄之資訊，以便開發者 debug。

> `logging` 為 python module，其主要功能為提供處理 log 訊息之 API。 

from datatime

#### 2. register table 所需之 module (RegisterObject, TableObject)


### log 訊息處理

### class PyEval