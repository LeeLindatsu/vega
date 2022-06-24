# Vega 圖表學習-Nested Bar Chart 
[+Grafana 與 Vega 預定學習流程](https://paper.dropbox.com/doc/Grafana-Vega-IGfmlTUqRodWXVl3GLK1O) 


https://vega.github.io/editor/#/examples/vega/bar-chart


**"width" & “padding” & “autosize”:**

![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655879231898_.PNG)
!["autosize": "pad"、”none”、”fit”、”fit-x”、”fit-y”](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655879337139_+2022-06-22+141811.png)

![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655878403618_+2022-06-22+141201.png)



    {
      "$schema": "https://vega.github.io/schema/vega/v5.json",
      "description": "A nested bar chart example, with bars grouped by category.",
      "width": 300,
      "padding": 5,
      "autosize": "pad",


## "signals"

參數可視化 & 交互行為的動態變量


- “value” : 一開始圖出現時的初始值設定
- “bind” : 後面參數可**外部調整輸入** => 讓圖表外觀由操作者調整，如列表、拉霸等
        - “input”:   `checkbox` ,  `radio` ,  `range` ,  `select`  , `button`  , `number`
            
                        網址內有其他元素可使用:
https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input

        - “min” : 人為能設的最小值數值(拉霸最左邊)
        - “max” : 人為能設的最大值數值(拉霸最右邊)
        - “step” : 移動一格增加的數值
        - “update” : 
                - trellisExtent[1]:正常
                - trellisExtent[0]:最下面的長條圖和長度表會移到圖中間


![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655880347747_2.PNG)

      "signals": [
        {
          "name": "rangeStep", "value": 20,
          "bind": {"input": "range", "min": 5, "max": 50, "step": 1}
    
        },
        {
          "name": "innerPadding", "value": 0.1,
          "bind": {"input": "range", "min": 0, "max": 0.7, "step": 0.01}
        },
        {
          "name": "outerPadding", "value": 0.2,
          "bind": {"input": "range", "min": 0, "max": 0.4, "step": 0.01}
        },
        {
          "name": "height",
          "update": "trellisExtent[1]"
        }
      ],


## “data”:
- “values”:
        - “a”(數字):數值相同時（如：都為0）會分為同一組顏色
        
        - “b”(英文數字):在此**相同顏色**的組內，代表的英文字為何
            (若  “a”的數字  和  “b”的英文字  相同 ＝> “c”的數值會  相加  後  除  以個數)
            
        - “c”(數值): 長條圖的長度
![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655906134863_+2022-06-22+9.47.40.png)

![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655908903402_+2022-06-22+10.41.35.png)

- “transform”:
        - "type": "aggregate" => 表示數值要進行 分組 和 匯總
        
        - "groupby" : ["a", "b"] => 要分組的
                (理解此組別非數字，而是字串)
                
        - "fields" :  ["c"] => 計算數字的
        
        - ”ops” : `sum`, `average` , `count` (默認：`count`）
            字的分組執行（和“c”相關）
            `count` ：同組(“a”)內相同代表英文字(“b”)的數量
            `sum` : 同組(“a”)內相同英文字(“b”)的代表數字(“c”)總和
            `average`：同組(“a”)內相同英文字(“b”)代表數字(“c”)平均
            
        - “as”：用於輸出的名稱 (無會自動生成：sum_field等)
![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655946858428_+2022-06-23+8.53.36.png)
![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655946854250_+2022-06-23+9.02.07.png)

      "data": [
        {
          "name": "tuples",
          "values": [
            {"a": 0, "b": "a", "c": 6.3},
            {"a": 0, "b": "a", "c": 4.2},
            {"a": 0, "b": "b", "c": 6.8},
            {"a": 0, "b": "c", "c": 5.1},
            {"a": 1, "b": "b", "c": 4.4},
            {"a": 2, "b": "b", "c": 3.5},
            {"a": 2, "b": "c", "c": 6.2}
          ],
          "transform": [
            {
              "type": "aggregate",
              "groupby": ["a", "b"],
              "fields": ["c"],
              "ops": ["average"],
              "as": ["c"]
            }
          ]
        },
        {
          "name": "trellis",
          "source": "tuples",
          "transform": [
            {
              "type": "aggregate",
              "groupby": ["a"]
            },
            {
              "type": "formula", "as": "span",
              "expr": "rangeStep * bandspace(datum.count, innerPadding, outerPadding)"
            },
            {
              "type": "stack",
              "field": "span"
            },
            {
              "type": "extent",
              "field": "y1",
              "signal": "trellisExtent"
            }
          ]
        }
      ],


- **“type”:**
        - **"****formula" ：新值擴展的數字**
            -   “as": “span"：***必需。***寫入公式值的輸出字。
            -   "expr" :***必需。表達*** 計算衍生值的公式。
                - **bandspace** (*count*[, *paddingInner*, *paddingOuter*])**：**元素的計數及內部和外部填充值。有助於確定圖表大小。
                        - datum：數據轉換和事件處理程序表達，JavaScript : `datum.value` or `datum['My Value']`
                        
        - **"stack" ： 堆疊、柱**
            - “field" :  定 stack 高度。


        -  **"extent" :** 轉換計算數字的最小值和最大值
            -  “field”：***必需。***要計算範圍的數字。
            -  “signal”：計算的範圍的數字結合到設定完成名稱的signal
![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655950362362_+2022-06-23+10.11.15.png)




https://vega.github.io/editor/#/edited



      "scales": [
        {
          "name": "xscale",
          "domain": {"data": "tuples", "field": "c"},
          "nice": true,
          "zero": true,
          "round": true,
          "range": "width"
        },
        {
          "name": "color",
          "type": "ordinal",
          "range": "category",
          "domain": {"data": "trellis", "field": "a"}
        }
      ],
      "axes": [
        { "orient": "bottom", "scale": "xscale", "domain": true }
      ],
    
## “scales”
    將 數據值（數字、日期、類別*等）變成* 圖的視覺值（像素、顏色、大小）。
    
- "domain"
        - “data”：***必需。***放 包含此數據 的data名。
        - “field”：***必需。***數據名稱（例如，`"price"`）
        - "nice"(布林)：修改比例的數，只將邊界擴展到最接近的捨入值。(數字四捨五入)
        - "zero"(布林)：將最低數值設為起點，調整其他長條圖
        - "round"(布林)：`false`將數字輸出值四捨五入為整數
        - "range"：指定比例範圍 ，  `width` 由  signal 確定空間範圍
- “type”
        - "ordinal"：離散的域和範圍
        - “category”：類別使用順序比例
## “axes”
    使用刻度、網格線和標籤可視化空間比例。
        - “orient” : ***必需。***axes 的方向
        - “scale” : ***必需。***支持axes組件的比例的名稱。
        - “domain”：指示是否應將域（axes基線）作為axes的一部分包含在內（默認`true`）
        
    "marks": [
        {
          "type": "group",
          "from": {
            "data": "trellis",
            "facet": {
              "name": "faceted_tuples",
              "data": "tuples",
              "groupby": "a"
            }
          },
          "encode": {
            "enter": {
              "x": {"value": 0},
              "width": {"signal": "width"}
            },
            "update": {
              "y": {"field": "y0"},
              "y2": {"field": "y1"}
            }
          },
![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655968092737_+2022-06-23+3.05.50.png)

## "marks"
    使用幾何圖型（例如矩形、線條和繪圖符號）對數據進行可視化編碼。
        - "type"：***必需。***圖形標記類型。
        - "from"：描述此 mark set 應可視化的數據的對象。
                - "data"：從中提取的 data set 的名稱。
                - “groupby”：For data-driven facets，數據進行分區的名稱。
        - “encode”：標記屬性的視覺編碼規則。
            三個主要 property sets: `*enter*`, `*update*`, `*exit*`
                    `enter` 首次實例化標記項時調用該集合。
                    `update` 只要數據或顯示屬性更新，就會調用該集合。
                    `exit` 當支持標記項的數據值被刪除時調用該集合。



    
          "scales": [
            {
              "name": "yscale",
              "type": "band",
              "paddingInner": {"signal": "innerPadding"},
              "paddingOuter": {"signal": "outerPadding"},
              "round": true,
              "domain": {"data": "faceted_tuples", "field": "b"},
              "range": {"step": {"signal": "rangeStep"}}
            }
          ],
          "axes": [
            { "orient": "left", "scale": "yscale",
              "ticks": false, "domain": false, "labelPadding": 4 }
          ],
          "marks": [
            {
              "type": "rect",
              "from": {"data": "faceted_tuples"},
              "encode": {
                "enter": {
                  "x": {"value": 0},
                  "x2": {"scale": "xscale", "field": "c"},
                  "fill": {"scale": "color", "field": "a"},
                  "strokeWidth": {"value": 2}
                },
                "update": {
                  "y": {"scale": "yscale", "field": "b"},
                  "height": {"scale": "yscale", "band": 1},
                  "stroke": {"value": null},
                  "zindex": {"value": 0}
                },
                "hover": {
                  "stroke": {"value": "firebrick"},
                  "zindex": {"value": 1}
                }
              }
            }
          ]
        }
      ]
    }
    
- **"scales":**
            - "paddingInner”：內部填充（間距），該值必須在 [0,1]
            - “paddingOuter”：外部填充（間距），該值必須在 [0,1]
![](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655970710905_+2022-06-23+3.51.45.png)

            - ”range”：The range of the scale，表示一組視覺值。
![x x2 xc y y2 yc](https://paper-attachments.dropbox.com/s_F0E7F2DDFB7CA6EC3A7605B500A21615BE0DCCDDE2A46CCD472009039104B869_1655977030334_+2022-06-23+5.32.12.png)

        
            - "marks" : 
                - "enter"
                    - "x" :  "value": 0 → 表示 長條圖起始，數字越大消失的範圍越大
                    - “strokeWidth”：筆劃寬度
                    - “fill”:填充顏色(填滿顏色)
                    - “stroke”：填充顏色（只有邊框）
                    
                    *fill* 屬性控制填色， *stroke* 屬性控制邊框
                    **stroke**指的是**線**、**文字**、及**元素外框**的"**顏色**"
                    - “zindex”：當元素之間重疊，z-index 較大的元素會覆蓋較小的元素在上層進行顯示。默認值為`0`
                    
                    
                    

