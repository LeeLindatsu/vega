# Vega 圖表學習-job-voyager

https://vega.github.io/editor/#/examples/vega/job-voyager

    {
      "$schema": "https://vega.github.io/schema/vega/v5.json",
      "description": "A searchable, stacked area chart of U.S. occupations from 1850 to 2000.",
      "width": 800,
      "height": 500,
      "padding": 5,
      "signals": [
        {
          "name": "sex", "value": "all",
          "bind": {"input": "radio", "options": ["men", "women", "all"]}
        },
        {
          "name": "query", "value": "",
          "on": [
            {"events": "area:click!", "update": "datum.job"},
            {"events": "dblclick!", "update": "''"}
          ],
          "bind": {"input": "text", "placeholder": "search", "autocomplete": "off"}
        }
      ],


- "signals"：參數可視化並可驅動交互行為的動態變量

(define a mark property or data transform parameter)<下圖右>

![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1656468471167_+2022-06-29+100711.png)
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1656465404403_+2022-06-29+091622.png)

        - “value"：signals 的初始值（默認`undefined`）。*該值在 init* 或 *update* 前出現。
                > “init” : `**≥4.4**`初始化表達式。只用僅一次。
                > 
                > “update” ：signals的更新值，
                > (當 react 屬性 ≠ `false` )自動更新以反應上面行數的signals變化。
                > init ↔ update (互斥)，不能一起使用。
                
        - “bind”： signals 綁定在可視化外定義的輸入元素。Vega 將生成新的 HTML 表單元素並設置雙向綁定：對輸入元素的更改將更新signals
                > “input”：**必需。**
                > **有效值：**`**checkbox**`(勾選項) **,** `**radio**`(圓圈選項) **,**  `**range**`(拉霸) **,**  `**select**` (清單選擇) 或是 **有效的**[**HTML 表單輸入類型**](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input)**。** 
                > 
                > “options”：(選項)  **<上圖左>** **必需。**
                > 可供選擇的選項。
                > 
                > “placeholder”：**有效值：**`password` , `search` , `tel` , `text` , `url`
                > Text that appears in the form control when it has no value set
                > 有關字段中預期的信息類型的簡短提示。一個詞或短語，提供有關預期數據類型的提示，而不是解釋或提示。
                > 
                > “autocomplete”（**不是**布爾）：
                > Hint for form autofill feature填寫表單字段值時必須提供的自動幫助
                > 例如，瀏覽器可能會讓用戶保存他們的姓名、地址、電話號碼和電子郵件地址以用於自動完成。
                > The `autocomplete` attribute is valid on `hidden`, `text`, `search`, `url`, `tel`, `email`, `date`, `month`, `week`, `time`, `datetime-local`, `number`, `range`, `color`, and `password`. 
                > 
                > 
                
        - “on”：更新signals值以回應輸入事件。
                > “events”：**必需。**
                > 要反應的事件。(The events to respond to.)
                > 
                > "update"：
                > 當事件發生時，結果會成為新的 signal 。
                > 若未具體說明 → update就是**必需**


      "data": [
        {
          "name": "jobs",
          "url": "data/jobs.json",
          "transform": [
            {
              "type": "filter",
              "expr": "(sex === 'all' || datum.sex === sex) && (!query || test(regexp(query,'i'), datum.job))"
            },
            {
              "type": "stack",
              "field": "perc",
              "groupby": ["year"],
              "sort": {
                "field": ["job", "sex"],
                "order": ["descending", "descending"]
              }
            }
          ]
        },
        {
          "name": "series",
          "source": "jobs",
          "transform": [
            {
              "type": "aggregate",
              "groupby": ["job", "sex"],
              "fields": ["perc", "perc"],
              "ops": ["sum", "argmax"],
              "as": ["sum", "argmax"]
            }
          ]
        }
      ],


- “data”：
    資料集(data set)  解釋和轉換(definitions and transforms)  定義  要加載的數據  及   如何處理它。
        - “name”：***必需。***
        - "url"：從URL中加載資料集。用 `format` 屬性確保正確解析加載的數據。未指定：假定數據採 JSON 格式。
        - “transform”：處理資料流 → 過濾數據，計算新的領域(fields)或導出新的資料流。
| - `[aggregate](https://vega.github.io/vega/docs/transforms/aggregate)` - Group and summarize a data stream.<br>    - `[bin](https://vega.github.io/vega/docs/transforms/bin)` - Discretize numeric values into uniform bins.<br>    - `[collect](https://vega.github.io/vega/docs/transforms/collect)` - Collect and sort all data objects in a stream.<br>    - `[countpattern](https://vega.github.io/vega/docs/transforms/countpattern)` - Count the frequency of patterns in text strings.<br>    - `[cross](https://vega.github.io/vega/docs/transforms/cross)` - Perform a cross-product of a data stream with itself.<br>    - `[density](https://vega.github.io/vega/docs/transforms/density)` - Generate values drawn from a probability distribution.<br>    - `[dotbin](https://vega.github.io/vega/docs/transforms/dotbin)` - Perform density binning for dot plot construction. **≥ 5.7**<br>    - `[extent](https://vega.github.io/vega/docs/transforms/extent)` - Compute minimum and maximum values over a data stream.<br>    - `[**filter**](https://vega.github.io/vega/docs/transforms/filter)` **-** **Filter a data stream using a predicate expression.(**敘述表達**)**<br>    - `[flatten](https://vega.github.io/vega/docs/transforms/flatten)` - Map array-typed fields to data objects, one per array entry. **≥ 3.1**<br>    - `[fold](https://vega.github.io/vega/docs/transforms/fold)` - Collapse selected data fields into *key* and *value* properties.<br>    - `[formula](https://vega.github.io/vega/docs/transforms/formula)` - Extend data objects with derived fields using a formula expression.<br>    - `[identifier](https://vega.github.io/vega/docs/transforms/identifier)` - Assign unique key values to data objects.<br>    - `[kde](https://vega.github.io/vega/docs/transforms/kde)` - Estimate smoothed densities for numeric values. **≥ 5.4**<br>    - `[impute](https://vega.github.io/vega/docs/transforms/impute)` - Perform imputation of missing values.<br>    - `[joinaggregate](https://vega.github.io/vega/docs/transforms/joinaggregate)` - Extend data objects with calculated aggregate values.<br>    - `[loess](https://vega.github.io/vega/docs/transforms/loess)` - Fit a smoothed trend line using local regression. **≥ 5.4**<br>    - `[lookup](https://vega.github.io/vega/docs/transforms/lookup)` - Extend data objects by looking up key values on another stream.<br>    - `[pivot](https://vega.github.io/vega/docs/transforms/pivot)` - Pivot unique values to new aggregate fields. **≥ 3.2**<br>    - `[project](https://vega.github.io/vega/docs/transforms/project)` - Generate derived data objects with a selected set of fields.<br>    - `[quantile](https://vega.github.io/vega/docs/transforms/quantile)` - Calculate sample quantile values over an input data stream. **≥ 5.7**<br>    - `[regression](https://vega.github.io/vega/docs/transforms/regression)` - Fit regression models to smooth and predict values. **≥ 5.4**<br>    - `[sample](https://vega.github.io/vega/docs/transforms/sample)` - Randomly sample data objects in a stream.<br>    - `[sequence](https://vega.github.io/vega/docs/transforms/sequence)` - Generate a new stream containing a sequence of numeric values.<br>    - `[timeunit](https://vega.github.io/vega/docs/transforms/timeunit)` - Discretize date-time values into time unit bins. **≥ 5.8**<br>    - `[window](https://vega.github.io/vega/docs/transforms/window)` - Calculate over ordered groups, including ranking and running totals. |

![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658648153915_+2022-07-24+3.35.48.png)

    > “expr”：**必需。**用以過濾data，計算結果是`false`就會是過濾不呈現的對象。
    
    "expr": "(sex === 'all' || datum.sex === sex) && (!query || test(regexp(query,'i'), datum.job))"
    當 （sex選擇是”all” 或是 data一整筆sex 等於 sex的選項）而且 （非 query 或是 嘗試 尋找“query”輸入的字是否匹配“job”內的data）
    
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658649658452_+2022-07-24+4.00.51.png)

    > datum():
    > 將「一筆」資料綁定給選擇集當中的每一個元素當中。
    > 
    > Regex (正規表達式) :
    > 1.尋找匹配的字串
    > 2. 取代匹配的字串
    > 3. 驗證使用者輸入資料欄位
    > 4.擷取某段想要的資訊
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658651217305_+2022-07-24+4.26.51.png)

    > 
    > **“type”:”stack”:**
    > 通過堆疊值組來計算佈局。最常見的用例是創建堆疊圖(stacked graphs)，包括堆疊條形圖(stacked bar charts)和流圖(stream graphs)。
     "sort": {
                "field": ["job", "sex"],
                "order": ["descending", "descending"] }
    > “descending"：表示依照大小，大方塊（大的）在下依序往上堆疊
    > “ascending”：則相反，小方塊在下。
    > 
    > **"type": "aggregate"**
| **Property** | **Type**                                                    | **Description**                                                                                                                      |
| ------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| groupby      | [Field](https://vega.github.io/vega/docs/types/#Field)[ ]   | 要分組的數據字段。(The data fields to group by）<br>如果未指定，將使用包含所有數據對象的單個組。                                                                     |
| fields       | [Field](https://vega.github.io/vega/docs/types/#Field)[ ]   | 要為其計算聚合函數（aggregate functions）的The data fields。<br>該數組應與*ops* and *as* *為*數組。如果沒有指定no *fields* and *ops*，則默認使用a `count` aggregation。 |
| ops          | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 應用於*fields*的aggregation operations，例如`sum`、`average`或`count`。<br>如果未指定*操作*`count`，則默認使用aggregation。                                  |
| as           | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 用於 fields 中每個each aggregated field的輸出字段 *fields*。<br>如果未指定，將根據操作和字段名稱<br>（例如，、）自動生成`sum_field`名稱`average_field`。                     |

"scales"：

        {
          "name": "alpha",
          "type": "linear", "zero": true,
          "domain": {"data": "series", "field": "sum"},
          "range": [0.9, 0.1]
        },
    > 也和顏色相關，調整range可以改變深淺。
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658652918015_+2022-07-24+4.48.27.png)
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658652904842_+2022-07-24+4.48.17.png)

    > **“type”:”sqrt”**
    > 平方根 (sqrt) 刻度是指數為 0.5 的 [power scales](https://vega.github.io/vega/docs/scales/#pow) 便捷簡寫，
    > 表示平方根變換。
    > 
    > 可以放在字型作為設定
       {
          "name": "font",
          "type": "sqrt",
          "range": [0, 40], "round": true, "zero": true,
          "domain": {"data": "series", "field": "argmax.perc"}
        },
    > 更改“range”可以改變 **字型大小**
    > 
    > **"type": "quantile"**
    > 分位數
    > 分位數比例 ( `quantile`) 將輸入域值的樣本映射到基於計算的[分位數](https://en.wikipedia.org/wiki/Quantile)邊界的離散範圍。域被認為是連續的，因此規模將接受任何合理的輸入值；但是，域被指定為一組離散的樣本值。
      {
          "name": "opacity",
          "type": "quantile",
          "range": [0, 0, 0, 0, 0, 0.1, 0.2, 0.4, 0.7, 1.0],
          "domain": {"data": "series", "field": "argmax.perc"}
        },
    > 不透明度
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658654282346_+2022-07-24+5.06.37.png)

    > **"type": "quantize"**
    > 量化比例 ( `quantize`) 類似於[線性比例](https://vega.github.io/vega/docs/scales/#linear)，除了它們使用離散範圍而不是連續範圍。連續輸入域根據輸出範圍內的值的數量（ the cardinality of）被劃分為均勻的段。每個範圍值y可以表示為域值x的量化線性函數：y = m round(x) + b。
    > 量化比例對於創建具有固定數量的輸出值的顏色或大小編碼特別有用。


- “axes”

**Axes** visualize spatial [scale](https://vega.github.io/vega/docs/scales) mappings using ticks, grid lines and labels.

      "axes": [
        {
          "orient": "bottom", "scale": "x", "format": "d", "tickCount": 15
        },
        {
          "orient": "right", "scale": "y", "format": "%",
          "grid": true, "domain": false, "tickSize": 12,
          "encode": {
            "grid": {"enter": {"stroke": {"value": "#ccc"}}},
            "ticks": {"enter": {"stroke": {"value": "#ccc"}}}
          }
        }
      ],
| **Property** | **Type**                                                                                                                                                                       | **Description**                                                                                                                                                                                                                                                                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| scale        | [String](https://vega.github.io/vega/docs/types/#String)                                                                                                                       | ***必需的。***The name of the scale backing the axis component.                                                                                                                                                                                                                                                                                                                              |
| orient       | [String](https://vega.github.io/vega/docs/types/#String)                                                                                                                       | ***必需的。*** axis的方向。 <br> [axis orientation reference](https://vega.github.io/vega/docs/axes/#orientation).                                                                                                                                                                                                                                                                               |
| format       | [String](https://vega.github.io/vega/docs/types/#String) | [TimeMultiFormat](https://vega.github.io/vega/docs/types/#TimeMultiFormat)                                          | axis labels格式說明pattern。<br>For 數值（numerical）數值, 必須符合 [d3-format](https://github.com/d3/d3-format#locale_format) 。<br>For 日期時間 數值, 必須符合 [d3-time-format](https://github.com/d3/d3-time-format#locale_format) 。                                                                                                                                                                            |
| tickCount    | [Number](https://vega.github.io/vega/docs/types/#Number) | [String](https://vega.github.io/vega/docs/types/#String) | [Object](https://vega.github.io/vega/docs/types/#Object) | 所需的ticks數，用於可視化定量scales的axes。<br><br>可使用的字符串值：<br>`"millisecond"`, `"second"`, `"minute"`, `"hour"`, `"day"`, `"week"`, `"month"`, 和`"year"`。<br>就數字不同，值可能會是“不錯的”（2、5、10 的倍數）並且位於基礎規模的範圍內。<br> 對於時間或 utc 類型的刻度，the tick count可以改為時間間隔說明符。<br><br>或者，{"interval": "month", "step": 3} 形式的對象值區間說明符包括所需數量的區間步長（interval steps）。<br>  在這裡為each quarter（1 月、4 月、7 月、10 月）邊界（boundary）生成ticks。 |
| tickSize     | [Number](https://vega.github.io/vega/docs/types/#Number)                                                                                                                       | 像素（pixels）長度 of axis ticks。                                                                                                                                                                                                                                                                                                                                                              |
| grid （格線）    | [Boolean](https://vega.github.io/vega/docs/types/#Boolean)                                                                                                                     | 是否應將 grid lines 視為 axis 的一部分。<br>默認為 `false`。                                                                                                                                                                                                                                                                                                                                            |
| ticks（刻度）    | [Boolean](https://vega.github.io/vega/docs/types/#Boolean)                                                                                                                     | 是否應將 ticks 視為 axis 的一部分。<br>默認為 `true`。                                                                                                                                                                                                                                                                                                                                                  |

- “mark”：
    
      "marks": [
        {
          "type": "group",
          "from": {
            "data": "series",
            "facet": {
              "name": "facet",
              "data": "jobs",
              "groupby": ["job", "sex"]
            }
          },
          "marks": [
            {
              "type": "area",
              "from": {"data": "facet"},
              "encode": {
                "update": {
                  "x": {"scale": "x", "field": "year"},
                  "y": {"scale": "y", "field": "y0"},
                  "y2": {"scale": "y", "field": "y1"},
                  "fill": {"scale": "color", "field": "sex"},
                  "fillOpacity": {"scale": "alpha", "field": {"parent": "sum"}}
                },
                "hover": {
                  "fillOpacity": {"value": 0.2}
                }
              }
            }
          ]
        },
        {
          "type": "text",
          "from": {"data": "series"},
          "interactive": false,
          "encode": {
            "update": {
              "x": {"scale": "x", "field": "argmax.year"},
              "dx": {"scale": "offset", "field": "argmax.year"},
              "y": {"signal": "scale('y', 0.5 * (datum.argmax.y0 + datum.argmax.y1))"},
              "fill": {"value": "#000"},
              "fillOpacity": {"scale": "opacity", "field": "argmax.perc"},
              "fontSize": {"scale": "font", "field": "argmax.perc", "offset": 5},
              "text": {"field": "job"},
              "align": {"scale": "align", "field": "argmax.year"},
              "baseline": {"value": "middle"}
            }
          }
        }
      ]
    }
    
    > “type”:"group"
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658666362189_+2022-07-24+8.38.25.png)

    > "type": "area”
![](https://paper-attachments.dropbox.com/s_DA5D2FB48570875AD4949C6F43FC5F65E011F042FD7AD8A48EE55AB47401F89C_1658667554395_+2022-07-24+8.53.39.png)

    > “fillOpacity”： 不透明度調整， 0（透明）到 1（不透明）
    > **“hover“：則將在鼠標懸停時調用該集。**

