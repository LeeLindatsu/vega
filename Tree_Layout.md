# Vega 圖表學習-Tree Layout


https://vega.github.io/editor/#/examples/vega/tree-layout

## 關聯圖
![](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658580824206_+2022-07-23+8.53.37.png)



1.   "signals"
![](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658584382061_+2022-07-23+8.56.43.png)

    "name": "separation", "value": false,
    "bind": {"input": "checkbox"}
    “separation”:
        用於使用節點是否要呈現分離狀態。
        如果`true`（默認），表親節點將比兄弟節點放置得更遠。
        如果`false`，節點將被均勻分離，就像在標準樹狀圖中一樣。

可以看以下網址和”separation”相關：
https://vega.github.io/vega/docs/transforms/tree/



2. data
      "data": [
        {
          "name": "tree",
          "url": "data/flare.json",
          "transform": [
            {
              "type": "stratify",
              "key": "id",
              "parentKey": "parent"
            },
            {
              "type": "tree",
              "method": {"signal": "layout"},
              "size": [{"signal": "height"}, {"signal": "width - 100"}],
              "separation": {"signal": "separation"},
              "as": ["y", "x", "depth", "children"]
            }
          ]
        },
        {
          "name": "links",
          "source": "tree",
          "transform": [
            { "type": "treelinks" },
            {
              "type": "linkpath",
              "orient": "horizontal",
              "shape": {"signal": "links"}
            }
          ]
        }
      ],

設立一個準備要導入的值名叫“tree”，並且把資料都倒入（“url”），進行資料轉換（“transform”）→要對輸入數據執行的轉換數組。轉換輸出然後成為該數據集的值。

**以下都為“transform”內的值：**


- "type": "stratify”
| ## key       | **必要的。** ****包含每個節點的唯一鍵（標識符）的數據字段。                                                                                 |
| ------------ | ------------------------------------------------------------------------------------------------------------------ |
| ## parentKey | **必要的。**<br>層次結構中每個節點的父節點的鍵值的數據字段。<br>要指示 root node of the tree， **<br>*parentKey*值必須是`null`、`undefined`或空字符串`''`。 |

- "type": "tree”
              "method": {"signal": "layout"},
              "size": [{"signal": "height"}, {"signal": "width - 100"}],
              "separation": {"signal": "separation"}

**其中 {“signal”:”   …  ”} 表示引用在 signal 參數所設定的值。**

| **Property**   | **Type**                                                    | **Description**                                                                                                             |
| -------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| ## field       | [Field](https://vega.github.io/vega/docs/types/#Field)      | 對應於節點數值的數據字段。<br>a node and all its descendants的值的總和在節點對像上作為`value`屬性可用。<br>如果未指定，將改為計算leaf nodes的數量。                       |
| ## **sort**    | [Compare](https://vega.github.io/vega/docs/types/#Compare)  | 用於對兄弟節點進行排序的比較器。comparator的輸入是or are tree node objects，而不是輸入 data objects。.                                                 |
| ## **method**  | [String](https://vega.github.io/vega/docs/types/#String)    | 要使用的佈局顯示方法。`tidy`（默認）或`cluster`。                                                                                            |
| **separation** | [Boolean](https://vega.github.io/vega/docs/types/#Boolean)  | 指示是否應應用節點分離邏輯的標誌。<br>`true`（默認），表親節點將比兄弟節點放置得更遠。<br>`false`，節點將被均勻分離，就像在標準樹狀圖中一樣。                                           |
| ## size        | [Number](https://vega.github.io/vega/docs/types/#Number)[ ] | 整體佈局的大小，以 [width, height] 數組的形式提供。                                                                                          |
| ## nodeSize    | [Number](https://vega.github.io/vega/docs/types/#Number)[ ] | 每個節點的大小，以 [width, height] 數組的形式提供。                                                                                          |
| ## as          | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 寫入佈局結果的輸出字段。<br>默認值為`["x", "y", "depth", "children"]`，其中`x`和`y`是佈局坐標，<br>`depth`：tree depth，<br>`children`：tree 中節點的子節點的計數。 |



3. “scales”

將map data values（數字、日期、類別）**縮放**為視覺值（像素、顏色、大小）。


      "scales": [
        {
          "name": "color",
          "type": "linear",
          "range": {"scheme": "magma"},
          "domain": {"data": "tree", "field": "depth"},
          "zero": true
        }
      ],

“type”: “linear” : 
線性刻度 ( `linear`) 是保留比例差異的[定量刻度。](https://vega.github.io/vega/docs/scales/#quantitative)
每個範圍值*y*可以表示為域值*x*的線性函數：*y = mx + b*。

“name” : “color”  & “range”：{”scheme”:”magama”} → 設定整個圖表的配色

![color scheme: https://vega.github.io/vega/docs/schemes/](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658628146487_+2022-07-24+9.58.27.png)

| **Property** | **Type**                                                   | **Description**                                                                                                                                                                     |
| ------------ | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name         | [String](https://vega.github.io/vega/docs/types/#String)   | ***必需的。***scales唯一名稱。<br>Scales and [projections](https://vega.github.io/vega/docs/projections) 共享相同的namespace；名稱在兩者中必須是唯一的。                                                        |
| type         | [String](https://vega.github.io/vega/docs/types/#String)   | scale 的類型（默認`linear`）。                                                                                                                                                              |
| domain       | [Domain](https://vega.github.io/vega/docs/scales/#domain)  | The domain of input data values for the scale。<br>For quantitative data，這可以採用具有最小值和最大值的二元數組的形式。<br> For ordinal or categorical data，這可能是有效輸入值的數組。                                   |
| range        | [Range](https://vega.github.io/vega/docs/scales/#range)    | scale 範圍，表示 the set of visual values。<br>For numeric values，範圍通常採用具有最小值和最大值的二元素數組的形式。<br>For ordinal or quantized data，範圍可以是所需輸出值的數組，這些值mapped to elements in the specified domain。 |
| zero         | [Boolean](https://vega.github.io/vega/docs/types/#Boolean) | scale domain是否應包含零的布爾標誌。<br>默認值為`true` for `linear`, `sqrt` and `pow`, and `false`                                                                                                  |

        
4. “marker”：


      "marks": [
        {
          "type": "path",
          "from": {"data": "links"},
          "encode": {
            "update": {
              "path": {"field": "path"},
              "stroke": {"value": "#ccc"}
            }
          }
        },
        {
          "type": "symbol",
          "from": {"data": "tree"},
          "encode": {
            "enter": {
              "size": {"value": 100},
              "stroke": {"value": "#fff"}
            },
            "update": {
              "x": {"field": "x"},
              "y": {"field": "y"},
              "fill": {"scale": "color", "field": "depth"}
            }
          }
        },
        {
          "type": "text",
          "from": {"data": "tree"},
          "encode": {
            "enter": {
              "text": {"field": "name"},
              "fontSize": {"value": 9},
              "baseline": {"value": "middle"}
            },
            "update": {
              "x": {"field": "x"},
              "y": {"field": "y"},
              "dx": {"signal": "datum.children ? -7 : 7"},
              "align": {"signal": "datum.children ? 'right' : 'left'"},
              "opacity": {"signal": "labels ? 1 : 0"}
            }
          }
        }
      ]
    }
    


- "type": "path"
## 關聯線外型 設定

和 圖形外觀 相關， 可參考：https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths
正方形：

![](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658629616748_+2022-07-24+10.26.10.png)


波形：

![](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658629645279_+2022-07-24+10.26.21.png)


弧形：

![](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658629671673_+2022-07-24+10.26.27.png)

![https://vega.github.io/vega/docs/marks/path/](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658629526226_+2022-07-24+10.09.25.png)

    "stroke": {"value": "#ccc"}

**表示圖示內連接線的顏色**

“encode”:

    - `enter`首次實例化標記項時調用該集合。
    - 除非另有說明，否則`update`只要數據或顯示屬性更新，就會調用該集合。
    - `exit`當支持標記項的數據值被刪除時調用該集合。


## 關聯點外型 設定

"type": "symbol"

![https://vega.github.io/vega/docs/marks/symbol/](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658630680492_+2022-07-24+10.38.10.png)



    "enter": {
              "size": {"value": 100},
              "stroke": {"value": "#fff"}}

圖形 外框設定


    "update": {
              "x": {"field": "x"},
              "y": {"field": "y"},
              "fill": {"scale": "color", "field": "depth"}

圖形內顏色 套用  “scales” 設定的 ， 範圍是 “data” 的 "as": ["y", "x", "depth", "children"] 


## 字外觀 設定

“type”:”text”


![https://vega.github.io/vega/docs/marks/text/](https://paper-attachments.dropbox.com/s_EE678A34139CA760EAF89E54B70C45FED473909ECB5D4CE25A3523AB8620509A_1658631969623_+2022-07-24+10.52.52.png)



    "opacity": {"signal": "labels ? 1 : 0"}

透明度：0 (transparent) to 1 (opaque).
隨著”signal”的“labels”設定（由使用者操作）：

    "signals": [
        {
          "name": "labels", "value": true,
          "bind": {"input": "checkbox"}
        },

