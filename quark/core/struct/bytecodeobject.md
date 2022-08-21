# bytecodeobject.py

## module 功能解說
定義 BytecodeObject 類別用以儲存 Dalvik Bytecode 指令之三種資料：

-  mnemonic: Dalvik bytecode 指令集 (e.g. 'invoke-virtual')
-  registers: 該段指令使用之暫存器名稱 (e.g. 'v0')
-  parameter: 該段指令執行之程式碼，一般來說為呼叫之韓式名稱 (e.g. 'Lcom/google/progress/APNOperator;->deleteAPN()Z')

## module 程式碼
```python
class BytecodeObject:
    """BytecodeObject is used to store the instructions in smali, including mnemonic, registers, parameter"""

    __slots__ = ["_mnemonic", "_registers", "_parameter"]

    def __init__(self, mnemonic, registers, parameter):
        """
        ['invoke-virtual', 'v3', 'Lcom/google/progress/APNOperator;->deleteAPN()Z']

        :param mnemonic:
        :param registers:
        :param parameter:
        """
        self._mnemonic = mnemonic
        self._registers = registers
        self._parameter = parameter

    def __repr__(self):
        return f"<BytecodeObject-mnemonic:{self._mnemonic}, registers:{self._registers}, parameter:{self._parameter}>"

    def __eq__(self, obj):
        return (
            isinstance(obj, BytecodeObject)
            and obj.mnemonic == self.mnemonic
            and obj.registers == self.registers
            and obj.parameter == self.parameter
        )

    @property
    def mnemonic(self):
        """
        Dalvik bytecode instructions set, for example 'invoke-virtual'.

        :return: a string of mnemonic
        """
        return self._mnemonic

    @property
    def registers(self):
        """
        Registers used in Dalvik instructions, for example '[v3]'.

        :return: a list containing all the registers used
        """
        return self._registers

    @property
    def parameter(self):
        """
        Commonly used for functions called by invoke-kind instructions, for example
        'Lcom/google/progress/APNOperator;->deleteAPN()Z'.

        :return: a string of the function name
        """
        return self._parameter

```