# analysis.py

## module 功能
儲存 Quark 進行樣本分析之分析記錄與分析結果，以及提供包括：初始化分析表格，清除資料等函式。

## module function 程式碼
### init_pretty_table
初始化 PrettyTable 之表格，該表格主要儲存 summary report 之結果，資料包括：規則檔案名稱 (Filename)、規則描述 (Rule)、信心指數 (Confidence)、風險評分 (score) 與權重 (weight)。

輸出：分析結果之 Pretty Table 資料

### init_label_report_table
初始化 PrettyTable 之表格，該表格主要儲存 label 比較分析之結果，資料包括：分類名稱 (Label)、規則描述 (Description)、規則之數量 (Number of rules)、最高信心指數 (MAX Confidence %)。

輸出：分析結果之 Pretty Table 資料。

### clean_result
清除 1-5 階段之分析結果。

## module 程式碼
```python
def init_pretty_table():
    # Pretty Table Output
    tb = PrettyTable()
    tb.field_names = ["Filename", "Rule", "Confidence", "Score", "Weight"]
    tb.align = "l"
    return tb


def init_label_report_table():
    # Pretty Table Output
    tb = PrettyTable()
    tb.field_names = [
        "Label",
        "Description",
        "Number of rules",
        "MAX Confidence %",
    ]
    tb.align = "l"
    tb.sortby = "Number of rules"
    tb.reversesort = True
    return tb


class QuarkAnalysis:
    __slots__ = [
        "crime_description",
        "first_api",
        "second_api",
        "level_1_result",
        "level_2_result",
        "level_3_result",
        "level_4_result",
        "level_5_result",
        "json_report",
        "weight_sum",
        "score_sum",
        "summary_report_table",
        "label_report_table",
        "call_graph_analysis_list",
        "parent_wrapper_mapping",
    ]

    def __init__(self):
        self.crime_description = ""
        self.first_api = None
        self.second_api = None
        self.level_1_result = []
        self.level_2_result = []
        self.level_3_result = []
        self.level_4_result = []
        self.level_5_result = []

        # Json report
        self.json_report = []
        # Sum of the each weight
        self.weight_sum = 0
        # Sum of the each rule
        self.score_sum = 0
        self.summary_report_table = init_pretty_table()
        # label report
        self.label_report_table = init_label_report_table()
        # Call graph analysis
        self.call_graph_analysis_list = []

        # Mapping between the parent function and the wrapper method
        self.parent_wrapper_mapping = defaultdict(str)

    def clean_result(self):
        self.level_1_result.clear()
        self.level_2_result.clear()
        self.level_3_result.clear()
        self.level_4_result.clear()
        self.level_5_result.clear()


if __name__ == "__main__":
    pass
```
