# TableObject.py

## module 功能
定義 TableObject 類別用以儲存 Register Table 所需之資料，以及操作 Table 所需之函式 (insert, pop)，Register Table 用以記錄 Register 之

該資料包括：

- hash_table: 以二維陣列儲存 Regiester 數值交換記錄，第一層陣列記錄時間點，第二層陣列記錄 Register 之儲存值

## module function 程式碼
### insert
新增 RegisterObject 至指定之暫存器。

輸入 `index`: 儲存之暫存器名稱。
輸入 `var_obj`: 輸入欲新增之 RegisterObject。

### pop
刪除指定暫存器之最後一位資料。

輸入 `index`: 欲刪除資料之暫存器名稱。

### get_obj_list
取得指定暫存器之所有資料。

輸入 `index`: 欲取得資料之暫存器名稱。
輸出一陣列其儲存暫存器之資料。

### get_table
取得完整 RegisterTable 之資料。


## module 程式碼

```python
class TableObject:
    """This table is used to track the usage of variables in the register"""

    __slots__ = ["hash_table"]

    def __init__(self, count_reg):
        """
        This table used to store the variable object, which uses a hash table
        with a stack-based list to generate the bytecode variable tracker table.

        :param count_reg: the maximum number of register to initialize
        """
        self.hash_table = [[] for _ in range(count_reg)]

    def __repr__(self):
        return f"<TableObject-{self.hash_table}>"

    def insert(self, index, var_obj):
        """
        Insert VariableObject into the nested list in the hashtable.

        :param index: the index to insert to the table
        :param var_obj: instance of VariableObject
        :return: None
        """
        try:
            self.hash_table[index].append(var_obj)
        except IndexError:
            pass

    def get_obj_list(self, index):
        """
        Return the list which contains the VariableObject.

        :param index: the index to get the corresponding VariableObject
        :return: a list containing VariableObject
        """
        try:
            return self.hash_table[index]
        except IndexError:
            return None

    def get_table(self):
        """
        Get the entire hash table.

        :return: a two-dimensional list
        """
        return self.hash_table

    def pop(self, index):
        """
        Override the built-in pop function, to get the top element, which
        is VariableObject on the stack while not delete it.

        :param index: the index to get the corresponding VariableObject
        :return: VariableObject
        """
        return self.hash_table[index][-1]


if __name__ == "__main__":
    pass

```