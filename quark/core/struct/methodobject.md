# methodobject.py

## module 功能
定義 MethodObject 類別用以儲存 dex file 中之 method，主要用以建構 Quark 之 register table，儲存資料包括：

-  class_name: method 之類別名稱 (e.g. )
-  name: method 之函式名稱 (e.g. )
-  descriptor: method 之輸入參數型態，以及回傳值型態 (e.g. )
-  access_flags: 
-  cache: 

## module function 程式碼
### is_android_api

參考 https://developer.android.com/reference/packages 之資料，
從 method 類別名稱中判斷該 method 是否為 android 原生 API。
若是
    return True
否則
    return False

## module 程式碼
```python
@dataclass(unsafe_hash=True)
class MethodObject(object):
    """
    Information about a method in a dex file.
    """

    class_name: str
    name: str
    descriptor: str
    access_flags: str = field(compare=False, default="")
    cache: object = field(compare=False, default=None, repr=False)

    @property
    def full_name(self) -> str:
        return self.__str__()

    def is_android_api(self) -> bool:
        # Packages found at https://developer.android.com/reference/packages
        api_list = [
            "Landroid/",
            "Lcom/google/android/",
            "Ldalvik/",
            "Ljava/",
            "Ljavax/",
            "Ljunit/",
            "Lorg/apache/",
            "Lorg/json/",
            "Lorg/w3c/",
            "Lorg/xml/",
            "Lorg/xmlpull/",
        ]

        return any(self.class_name.startswith(prefix) for prefix in api_list)

    def __str__(self) -> str:
        return f"{self.class_name} {self.name} {self.descriptor}"
```