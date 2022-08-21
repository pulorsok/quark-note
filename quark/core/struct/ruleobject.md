# RuleObject.py

## module 功能
定義 RuleObject 類別用以儲存 Quark 規則之資料，主要儲存之資料包括：

- crime: 該 Quark 規則之描述
- permission: 該 Quark 規則所偵測之 permission
- api: 該 Quark 規則所偵測之 api，該筆資料固定為長度為 2 之陣列，分別為「第一次呼叫之 api」與「第二次呼叫之 api」。
- label: 該 Quark 規則之分類。 (e.g. 'camera', 'sms', etc)
- score: 該 Quark 規則之風險評分，預設為 1。

## module function 程式碼
### get_score
輸入該規則精分析後之 confidence，
並回傳經權重計算後之 score。

## module 程式碼

```python
class RuleObject:
    """RuleObject is used to store the rule from json file"""

    __slots__ = [
        "check_item",
        "_json_obj",
        "_crime",
        "_permission",
        "_api",
        "_score",
        "rule_filename",
        "_label",
    ]

    def __init__(self, ruleJson: os.PathLike, jsonData: dict = None) -> None:
        """Making detection rule a rule instance

        :param ruleJson: Path of a single Quark rule
        :param jsonData: Dictionary to directly assign to the content of this
         Quark rule, defaults to None
        """
        # the state of five stages
        self.check_item = [False, False, False, False, False]

        if jsonData is not None:
            self._json_obj = jsonData
        else:
            with open(ruleJson) as json_file:
                self._json_obj = json.loads(json_file.read())

        self._crime = self._json_obj["crime"]
        self._permission = self._json_obj["permission"]

        self._api = self._json_obj["api"]
        for index in range(len(self._api)):
            descriptor = self._api[index]["descriptor"]
            if " " not in descriptor:
                self._api[index][
                    "descriptor"
                ] = descriptor_to_androguard_format(descriptor)

        self._score = self._json_obj["score"]
        self.rule_filename = os.path.basename(ruleJson)
        self._label = self._json_obj["label"]

    def __repr__(self):
        return f"<RuleObject-{self.rule_filename}>"

    @property
    def crime(self):
        """
        Description of given crime.

        :return: a string of the crime
        """
        return self._crime

    @property
    def permission(self):
        """
        Permission requested by the apk to practice the crime.

        :return: a list of given permissions
        """
        return self._permission

    @property
    def api(self):
        """
        Key native APIs that do the action and target in order.

        :return: a list recording the APIs class_name and method_name in order
        """
        return self._api

    @property
    def label(self):
        """
        A list contains various lebels described in https://github.com/quark-engine/quark-rules/blob/master/label_desc.csv

        :return: a label list defined in rules
        """
        return self._label

    @property
    def score(self):
        """
        The value used to calculate the weighted score

        :return: integer
        """
        return self._score

    def get_score(self, confidence):
        """
        According to the state of the five stages, we calculate the weighted score based on exponential growth.
        For example, we captured the third stage in five stages, then the weighted score would be (2^3-1) / 2^4.

        2^(confidence - 1)

        :param confidence:
        :return: floating point
        """
        if confidence == 0:
            return 0
        return (2 ** (confidence - 1) * self._score) / 2**4


if __name__ == "__main__":
    pass
```