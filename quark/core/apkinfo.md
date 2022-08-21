# apkinfo.py

## module 功能
提供以 Androgaurd 為基礎，取得各項 apk/dex 資訊之 api。
該 module 首先輸入欲分析之 apk/dex 之檔案路徑，並且以 Androguard 函式庫進行分析。

## module function 程式碼
### permissions
取得 apk/dex 使用之權限。

- 輸出：輸出 python set，儲存所有定義之權限

### android_apis
取得 apk/dex 所有 Android 原生 API。

- 輸出：輸出 python set，儲存所有原生 API 之 MethodObject

### custom_methods
取得 apk/dex 所有自訂義之 API。

- 輸出：輸出 python set，儲存所有自訂義之 API 之 MethodObject

### all_methods
取得 apk/dex 所有包含 Android 原生與自訂義之 API。

- 輸出：輸出 python set，儲存所有包含 Android 原生與自訂義之 API 之 MethodObject

### find_method
取得 apk/dex 內符合輸入之 class name、method name 與 descriptor 之函式。
並將搜尋之函式轉為 MethodObject 回傳。

- 輸入：
  - class_name: 欲搜尋函式之 class 名稱。(e.g. `Lcom/google/progress/APNOperator;`)
  - method_name: 欲搜尋函式之 method 名稱。(e.g. `deleteAPN`)
  - descriptor: 欲搜尋函式之輸入參數型態，以及輸出參數型態。(e.g. `()Z`)
- 輸出：以 MethodObject 資料型態回傳

### upperfunc
取得目標函式之所有上層函式。

- 輸入：以 MethodObject 作為目標函式資料型態輸入
- 輸出：輸出 python set，儲存所有上層函式之 MethodObject

### lowerfunc
取得目標函式之所有呼叫之函式。

- 輸入：以 MethodObject 作為目標函式資料型態輸入
- 輸出：輸出 python set，儲存所有呼叫函式之 MethodObject

### get_method_bytecode
取得目標函式中所有 smali 程式碼，並以 BytecodeObject 資料型態回傳。(e.g. `['new-instance', 'v4', Lcom/google/progress/SMSHelper;]`)

- 輸入：以 MethodObject 作為目標函式資料型態輸入
- 輸出：輸出 python set，儲存所有目標函式之 BytecodeObject

### get_strings
取得 apk/dex 所有定義之 string 資料。

- 輸出：輸出 python set，儲存所有定義之字串

### _construct_bytecode_instruction
將完整之 smali byte code 字串轉為 python list 儲存。
例如：輸入 `"new-instance v4 Lcom/google/progress/SMSHelper;"`
轉為 `['new-instance', 'v4', Lcom/google/progress/SMSHelper;]`

- 輸入：輸入完整 smali byte code 指令之字串 (e.g. `"new-instance v4 Lcom/google/progress/SMSHelper;"`)
- 輸出：python list 儲存 smali byte code 指令 (e.g. `['new-instance', 'v4', Lcom/google/progress/SMSHelper;]`)

### get_wrapper_smali
建構 API1 與 API2 在函式中之執行順序。

- 輸入：
  - parent_method：兩 API 之 mutual parent function
  - first_method：第一個呼叫之 API
  - second_method：第二個呼叫之 API
- 輸出：python dict 儲存第一呼叫之 API 位置 (index)，以及第二呼叫之 API 位置 (index)

### superclass_relationships
取得 apk/dex 中所有 class 之繼承關係資料。

- 輸出：python dict 記錄每個 class 所繼承之類別

### subclass_relationships
取得 apk/dex 中所有 class 之子類別關係資料。

- 輸出：python dict 記錄每個 class 所之子類別

### _convert_to_method_object
將 Androguard 之 MethodAnalysis 資料型態轉為 MethodObject。

- 輸入：目標函式之 MethodAnalysis (Androguard)
- 輸出：目標函式之 MethodObject (Quark)

## module 程式碼
```python
class AndroguardImp(BaseApkinfo):
    """Information about apk based on androguard analysis"""

    __slots__ = ("apk", "dalvikvmformat", "analysis")

    def __init__(self, apk_filepath: Union[str, PathLike]):
        super().__init__(apk_filepath, "androguard")

        if self.ret_type == "APK":
            # return the APK, list of DalvikVMFormat, and Analysis objects
            self.apk, self.dalvikvmformat, self.analysis = AnalyzeAPK(apk_filepath)
        elif self.ret_type == "DEX":
            # return the sha256hash, DalvikVMFormat, and Analysis objects
            _, _, self.analysis = AnalyzeDex(apk_filepath)
        else:
            raise ValueError("Unsupported File type.")

    @property
    def permissions(self) -> List[str]:
        if self.ret_type == "APK":
            return self.apk.get_permissions()

        if self.ret_type == "DEX":
            return []

    @property
    def android_apis(self) -> Set[MethodObject]:
        apis = set()

        for external_cls in self.analysis.get_external_classes():
            for meth_analysis in external_cls.get_methods():
                if meth_analysis.is_android_api():
                    apis.add(meth_analysis)

        return {self._convert_to_method_object(api) for api in apis}

    @property
    def custom_methods(self) -> Set[MethodObject]:
        return {
            self._convert_to_method_object(meth_analysis)
            for meth_analysis in self.analysis.get_methods()
            if not meth_analysis.is_external()
        }

    @property
    def all_methods(self) -> Set[MethodObject]:
        return {
            self._convert_to_method_object(meth_analysis)
            for meth_analysis in self.analysis.get_methods()
        }

    @functools.lru_cache()
    def find_method(
        self,
        class_name: Optional[str] = ".*",
        method_name: Optional[str] = ".*",
        descriptor: Optional[str] = ".*",
    ) -> MethodObject:
        regex_class_name = re.escape(class_name)
        regex_method_name = f"^{re.escape(method_name)}$"
        regex_descriptor = re.escape(descriptor)

        method_result = self.analysis.find_methods(
            classname=regex_class_name,
            methodname=regex_method_name,
            descriptor=regex_descriptor,
        )

        result = next(method_result, None)
        return self._convert_to_method_object(result) if result else None

    @functools.lru_cache()
    def upperfunc(self, method_object: MethodObject) -> Set[MethodObject]:
        method_analysis = method_object.cache
        return {
            self._convert_to_method_object(call)
            for _, call, _ in method_analysis.get_xref_from()
        }

    def lowerfunc(self, method_object: MethodObject) -> Set[MethodObject]:
        method_analysis = method_object.cache
        return {
            (self._convert_to_method_object(call), offset)
            for _, call, offset in method_analysis.get_xref_to()
        }

    def get_method_bytecode(self, method_object: MethodObject) -> Set[MethodObject]:
        method_analysis = method_object.cache
        try:
            for (
                _,
                ins,
            ) in method_analysis.get_method().get_instructions_idx():
                bytecode_obj = None
                reg_list = []

                # count the number of the registers.
                length_operands = len(ins.get_operands())
                if length_operands == 0:
                    # No register, no parameter
                    bytecode_obj = BytecodeObject(
                        ins.get_name(),
                        None,
                        None,
                    )
                else:
                    index_of_parameter_starts = None
                    for i in range(length_operands - 1, -1, -1):
                        if not isinstance(ins.get_operands()[i][0], Operand):
                            index_of_parameter_starts = i
                            break

                    if index_of_parameter_starts is not None:
                        parameter = ins.get_operands()[index_of_parameter_starts]
                        parameter = (
                            parameter[2] if len(parameter) == 3 else parameter[1]
                        )

                        for i in range(index_of_parameter_starts):
                            reg_list.append(
                                "v" + str(ins.get_operands()[i][1]),
                            )
                    else:
                        parameter = None
                        for i in range(length_operands):
                            reg_list.append(
                                "v" + str(ins.get_operands()[i][1]),
                            )

                    bytecode_obj = BytecodeObject(ins.get_name(), reg_list, parameter)

                yield bytecode_obj
        except AttributeError:
            # TODO Log the rule here
            pass

    def get_strings(self) -> str:
        return {
            str(string_analysis.get_orig_value())
            for string_analysis in self.analysis.get_strings()
        }

    @functools.lru_cache()
    def _construct_bytecode_instruction(self, instruction):
        """
        Construct a list of strings from the given bytecode instructions.

        :param instruction: instruction instance from androguard
        :return: a list with bytecode instructions strings
        """
        instruction_list = [instruction.get_name()]
        reg_list = []

        # count the number of the registers.
        length_operands = len(instruction.get_operands())
        if length_operands == 0:
            # No register, no parameter
            return instruction_list

        elif length_operands == 1:
            # Only one register

            reg_list.append(f"v{instruction.get_operands()[length_operands - 1][1]}")

            instruction_list.extend(reg_list)

            return instruction_list
        elif length_operands >= 2:
            # the last one is parameter, the other are registers.

            parameter = instruction.get_operands()[length_operands - 1]
            for i in range(length_operands - 1):
                reg_list.append(
                    "v" + str(instruction.get_operands()[i][1]),
                )
            parameter = parameter[2] if len(parameter) == 3 else parameter[1]
            instruction_list.extend(reg_list)
            instruction_list.append(parameter)

            return instruction_list

    @functools.lru_cache()
    def get_wrapper_smali(
        self,
        parent_method: MethodObject,
        first_method: MethodObject,
        second_method: MethodObject,
    ) -> Dict[str, Union[BytecodeObject, str]]:
        method_analysis = parent_method.cache

        result = {
            "first": None,
            "first_hex": None,
            "second": None,
            "second_hex": None,
        }

        first_method_pattern = (
            f"{first_method.class_name}"
            f"->{first_method.name}{first_method.descriptor}"
        )
        second_method_pattern = (
            f"{second_method.class_name}"
            f"->{second_method.name}{second_method.descriptor}"
        )

        for _, ins in method_analysis.get_method().get_instructions_idx():
            if first_method_pattern in str(ins):
                result["first"] = self._construct_bytecode_instruction(ins)
                result["first_hex"] = ins.get_hex()
            if second_method_pattern in str(ins):
                result["second"] = self._construct_bytecode_instruction(ins)
                result["second_hex"] = ins.get_hex()

        return result

    @property
    def superclass_relationships(self) -> Dict[str, Set[str]]:
        hierarchy_dict = defaultdict(set)

        for _class in self.analysis.get_classes():
            hierarchy_dict[str(_class.name)].add(str(_class.extends))
            hierarchy_dict[str(_class.name)].union(
                str(implements) for implements in _class.implements
            )

        return hierarchy_dict

    @property
    def subclass_relationships(self) -> Dict[str, Set[str]]:
        hierarchy_dict = defaultdict(set)

        for _class in self.analysis.get_classes():
            class_name = str(_class.name)
            hierarchy_dict[str(_class.extends)].add(class_name)
            for implements in _class.implements:
                hierarchy_dict[str(implements)].add(class_name)

        return hierarchy_dict

    @staticmethod
    @functools.lru_cache
    def _convert_to_method_object(
        method_analysis: MethodAnalysis,
    ) -> MethodObject:
        return MethodObject(
            access_flags=method_analysis.access,
            class_name=str(method_analysis.class_name),
            name=str(method_analysis.name),
            descriptor=str(method_analysis.descriptor),
            cache=method_analysis,
        )

```