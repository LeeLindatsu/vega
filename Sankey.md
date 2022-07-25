# Vega 圖表學習-**Sankey**

https://www.elastic.co/blog/sankey-visualization-with-vega-in-kibana

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658739529366_Screenshot_from_2018-02-21_23-22-47.png)

# 1. 開頭 & "data”
    {
      $schema: https://vega.github.io/schema/vega/v3.0.json
      data: [
        {
          // query ES based on the currently selected time range and filter string
          name: rawData
          url: {
            %context%: true
            %timefield%: @timestamp
            index: logstash-*
            body: {
              size: 0
              aggs: {
                table: {
                  composite: {
                    size: 10000
                    sources: [
                      {
                        stk1: {
                          terms: {field: "geo.src"}
                        }
                      }
                      {
                        stk2: {
                          terms: {field: "geo.dest"}
                        }
                      }
                    ]
                  }
                }
              }
            }
          }
          // From the result, take just the data we are interested in
          format: {property: "aggregations.table.buckets"}
          // Convert key.stk1 -> stk1 for simpler access below
          transform: [
            {type: "formula", expr: "datum.key.stk1", as: "stk1"}
            {type: "formula", expr: "datum.key.stk2", as: "stk2"}
            {type: "formula", expr: "datum.doc_count", as: "size"}
          ]
        }
        {
          name: nodes
          source: rawData
          transform: [
            // when a country is selected, filter out unrelated data
            {
              type: filter
              expr: !groupSelector || groupSelector.stk1 == datum.stk1 || groupSelector.stk2 == datum.stk2
            }
            // Set new key for later lookups - identifies each node
            {type: "formula", expr: "datum.stk1+datum.stk2", as: "key"}
            // instead of each table row, create two new rows,
            // one for the source (stack=stk1) and one for destination node (stack=stk2).
            // The country code stored in stk1 and stk2 fields is placed into grpId field.
            {
              type: fold
              fields: ["stk1", "stk2"]
              as: ["stack", "grpId"]
            }
            // Create a sortkey, different for stk1 and stk2 stacks.
            // Space separator ensures proper sort order in some corner cases.
            {
              type: formula
              expr: datum.stack == 'stk1' ? datum.stk1+' '+datum.stk2 : datum.stk2+' '+datum.stk1
              as: sortField
            }
            // Calculate y0 and y1 positions for stacking nodes one on top of the other,
            // independently for each stack, and ensuring they are in the proper order,
            // alphabetical from the top (reversed on the y axis)
            {
              type: stack
              groupby: ["stack"]
              sort: {field: "sortField", order: "descending"}
              field: size
            }
            // calculate vertical center point for each node, used to draw edges
            {type: "formula", expr: "(datum.y0+datum.y1)/2", as: "yc"}
          ]
        }
        {
          name: groups
          source: nodes
          transform: [
            // combine all nodes into country groups, summing up the doc counts
            {
              type: aggregate
              groupby: ["stack", "grpId"]
              fields: ["size"]
              ops: ["sum"]
              as: ["total"]
            }
            // re-calculate the stacking y0,y1 values
            {
              type: stack
              groupby: ["stack"]
              sort: {field: "grpId", order: "descending"}
              field: total
            }
            // project y0 and y1 values to screen coordinates
            // doing it once here instead of doing it several times in marks
            {type: "formula", expr: "scale('y', datum.y0)", as: "scaledY0"}
            {type: "formula", expr: "scale('y', datum.y1)", as: "scaledY1"}
            // boolean flag if the label should be on the right of the stack
            {type: "formula", expr: "datum.stack == 'stk1'", as: "rightLabel"}
            // Calculate traffic percentage for this country using "y" scale
            // domain upper bound, which represents the total traffic
            {
              type: formula
              expr: datum.total/domain('y')[1]
              as: percentage
            }
          ]
        }
        {
          // This is a temp lookup table with all the 'stk2' stack nodes
          name: destinationNodes
          source: nodes
          transform: [
            {type: "filter", expr: "datum.stack == 'stk2'"}
          ]
        }
        {
          name: edges
          source: nodes
          transform: [
            // we only want nodes from the left stack
            {type: "filter", expr: "datum.stack == 'stk1'"}
            // find corresponding node from the right stack, keep it as "target"
            {
              type: lookup
              from: destinationNodes
              key: key
              fields: ["key"]
              as: ["target"]
            }
            // calculate SVG link path between stk1 and stk2 stacks for the node pair
            {
              type: linkpath
              orient: horizontal
              shape: diagonal
              sourceY: {expr: "scale('y', datum.yc)"}
              sourceX: {expr: "scale('x', 'stk1') + bandwidth('x')"}
              targetY: {expr: "scale('y', datum.target.yc)"}
              targetX: {expr: "scale('x', 'stk2')"}
            }
            // A little trick to calculate the thickness of the line.
            // The value needs to be the same as the hight of the node, but scaling
            // size to screen's height gives inversed value because screen's Y
            // coordinate goes from the top to the bottom, whereas the graph's Y=0
            // is at the bottom. So subtracting scaled doc count from screen height
            // (which is the "lower" bound of the "y" scale) gives us the right value
            {
              type: formula
              expr: range('y')[0]-scale('y', datum.size)
              as: strokeWidth
            }
            // Tooltip needs individual link's percentage of all traffic
            {
              type: formula
              expr: datum.size/domain('y')[1]
              as: percentage
            }
          ]
        }
      ]
| **Property** | **Type**                                                                                                               | **Description**                                                                                                                                                                                                   |
| ------------ | ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name         | [String](https://vega.github.io/vega/docs/types/#String)                                                               | ***必需的。***data set 的唯一名稱。                                                                                                                                                                                         |
| format       | [Format](https://vega.github.io/vega/docs/data/#format)                                                                | An object指定解析數據文件或值的格式。                                                                                                                                                                                           |
| source       | [String](https://vega.github.io/vega/docs/types/#String) | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 由一個或多個data sets的名稱用作此data set的源(the source)。<br>the source 與 轉換管道(transform pipeline) 組合使用 → 得到新數據很有用。<br><br>若是字符串值（string-valued），則表示source data set的名稱。<br>若是數組值（array-valued），則指定應合併（聯合）在一起的data source名稱的集合。 |
| url          | [String](https://vega.github.io/vega/docs/types/#String)                                                               | 從中加載 data set 的 URL。<br>使用 `*format*` 確保正確解析加載的數據。<br>如果未指定`*format*`，則假定數據採用row-oriented的 JSON 格式。                                                                                                               |
| transform    | [Transform](https://vega.github.io/vega/docs/transforms)[ ]                                                            | 要對輸入數據執行的轉換數組(transforms array)。轉換管道(transform pipeline)的輸出然後成為該數據集(data set)的值。                                                                                                                                  |

    sources: [
                      {
                        stk1: {
                          terms: {field: "geo.src"}
                        }
                      }
                      {
                        stk2: {
                          terms: {field: "geo.dest"}
                        }

“stk” → 在這個圖表中 設定 長柱條 供數據線連接的 參數。



    transform: [
            {type: "formula", expr: "datum.key.stk1", as: "stk1"}
            {type: "formula", expr: "datum.key.stk2", as: "stk2"}
            {type: "formula", expr: "datum.doc_count", as: "size"}
          ]

“transform”  :  [**”type” : ”formula”**]

                **公式**變換根據計算公式用新值擴展數據對象。
| expr              | [Expr](https://vega.github.io/vega/docs/types/#Expr)       | ***必需的。***計算得出值的公式[表達式。](https://vega.github.io/vega/docs/expressions)                                      |
| ----------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| as                | [String](https://vega.github.io/vega/docs/types/#String)   | ***必需的。***寫入公式值的輸出字段。                                                                                       |
| initonly<br>（初始化） | [Boolean](https://vega.github.io/vega/docs/types/#Boolean) | `true`：則僅在首次觀察（first observed）data object 時評估（evaluated）公式。<br>如果修改data object，公式值*不會*自動更新。<br>默認值為`false`. |



              type: filter
              expr: !groupSelector || groupSelector.stk1 == datum.stk1 || groupSelector.stk2 == datum.stk2

**“type”  :   “filter”**
根據提供的filter表達式從數據流中刪除對象。
可以和 使用者可自我操作 的參數一起使用。

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658672889564_+2022-07-24+10.27.57.png)


expr代表了以下的新”公式“：
當 非 ”groupSelector“ 
或是
”groupSelector“ 的 “stk1” 相等於 一個整個資料組的 “stk1”
或是
”groupSelector“ 的 “stk2” 相等於 一個整個資料組的 “stk2”

**“type” :  “fold”**：

1. 將一個或多個數據字段collapses（或“折疊（folds）”）為兩個屬性：一個 *key*（包含原始數據字段名稱）和一個 *value*（包含data value）。
2. 生成一個新的 data stream，其中每個數據對象由 *key* 和 *value* 以及相應輸入數據對象的所有原始字段組成。
3. 轉換`fold`僅適用於已知字段列表（使用`fields`參數設置）。如果您的數據對象包含數組類型的字段，您可能希望使用[flatten](https://vega.github.io/vega/docs/transforms/flatten)轉換。
| fields (字段) | [字段](https://vega.github.io/vega/docs/types/#Field)[ ]   | ***必需的。***指示要折疊的屬性的數據字段數組。                                |
| ----------- | -------------------------------------------------------- | --------------------------------------------------------- |
| as (作為)     | [字符串](https://vega.github.io/vega/docs/types/#String)[ ] | *折疊變換生成的key* 和 *value*的輸出字段名稱。<br>默認值為`["key", "value"]`. |

**“type”  :  “stack”**

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658673874000_+2022-07-24+10.44.30.png)

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658673950912_+2022-07-24+10.45.43.png)


**“type”  :  “aggregate”**
轉換對輸入 data stream 進行分組 和 匯總以生成得出 輸出流。
聚合轉換可用於計算數據對象組的計數、總和、平均值和其他描述性統計信息。

| **Property** | **Type**                                                    | **Description**                                                                             |
| ------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| groupby      | [Field](https://vega.github.io/vega/docs/types/#Field)[ ]   | 要分組的數據字段(data fields)。<br>未指定，將使用包含所有數據對象的單個組。                                              |
| fields       | [Field](https://vega.github.io/vega/docs/types/#Field)[ ]   | 要為其計算聚合函數的數據字段。<br>該數組應與*操作*對齊並*作為*數組。<br>若沒有指定 `*fields*` 和 `*ops*` *，*則默認使用 `count` 聚合。   |
| ops          | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 應用於*fields*的聚合操作，例如`sum`、`average`或`count`。<br>如果未指定`*ops*` *，*則默認使用 `count` 聚合             |
| as           | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 用於 fields 中每個聚合字段的輸出字段 *fields*。<br>如果未指定，將根據操作和字段名稱（例如，、）自動生成`sum_field`名稱`average_field`。 |

**“type”  :  “lookup”**
通過在 a secondary data stream 上查找值來擴展主數據流。
lookup 從主要數據流中接受一個或多個 key fields，然後在a secondary data stream的單個關鍵字段中搜索每個key fields。

| from   | [Data](https://vega.github.io/vega/docs/types/#Data)        | ***必需的。***要執行lookup的the secondary data的名稱。                                                                                                  |
| ------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| key    | [Field](https://vega.github.io/vega/docs/types/#Field)      | ***必需的。***The key field on the secondary stream。                                                                                            |
| fields | [Field](https://vega.github.io/vega/docs/types/#Field)[ ]   | ***必需的。***The data fields in the primary stream to lookup.                                                                                  |
| as     | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 寫入secondary stream中找到的數據的輸出fields。<br>如果未指定且提供了`*values*`參數，則將使用`*values*`數組(array)中的字段名稱。<br>*如果提供了多個* `*fields*` **或未指定`*values*`，則需要此參數。 |

**“type” : “linkpath”「連線」**

            {
              type: linkpath
              orient: horizontal
              shape: diagonal
              sourceY: {expr: "scale('y', datum.yc)"}
              sourceX: {expr: "scale('x', 'stk1') + bandwidth('x')"}
              targetY: {expr: "scale('y', datum.target.yc)"}
              targetX: {expr: "scale('x', 'stk2')"}
            }
![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658675480429_+2022-07-24+11.10.24.png)

| **Property** | **Type**                                                 | **Description**                                                                                                                                                                                                                                                                                         |
| ------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| sourceX      | [Field](https://vega.github.io/vega/docs/types/#Field)   | 源 x 坐標(source x-coordinate)的數據字段(field)。默認值為`source.x`                                                                                                                                                                                                                                                  |
| sourceY      | [Field](https://vega.github.io/vega/docs/types/#Field)   | 源 y 坐標(source y-coordinate)的數據字段(field)。默認值為`source.y`                                                                                                                                                                                                                                                  |
| targetX      | [Field](https://vega.github.io/vega/docs/types/#Field)   | 目標 x 坐標的數據字段(field)。默認值為`target.x`                                                                                                                                                                                                                                                                      |
| targetY      | [Field](https://vega.github.io/vega/docs/types/#Field)   | 目標 y 坐標的數據字段(field)。默認值為`target.y`                                                                                                                                                                                                                                                                      |
| orient       | [String](https://vega.github.io/vega/docs/types/#String) | 鏈接路徑的方向。(The orientation of the link path. )<br>`vertical`（默認）、`horizontal`或`radial`。<br><br>如果`radial`指定(specified)方向(orientation)，則 x 和 y 坐標參數(parameters)將分別解釋為角度（以弧度為單位）和半徑。                                                                                                                        |
| shape        | [String](https://vega.github.io/vega/docs/types/#String) | 鏈接路徑的形狀。(The shape of the link path)<br> One of `line` (default), `arc`, `curve`, `diagonal`, or `orthogonal`.                                                                                                                                                                                          |
| require      | [Signal](https://vega.github.io/vega/docs/types/#Signal) | 此變換所依賴的必需信號。（A required signal that this transform depends on.）<br><br>如果源或目標坐標值被設置為不同data stream中變換的非傳播副作用（a non-propagating side-effect of a transform）（例如[強制變換](https://vega.github.io/vega/docs/transforms/force/)），則需要此參數。<br><br>在這種情況下，上游（upstream）變換應該綁定到一個信號並且是鏈路路徑變換（linkpath transform）所需要的。 |
| as           | [String](https://vega.github.io/vega/docs/types/#String) | 鏈接路徑的輸出字段（The output field for the link path. ）<br>默認 `"path"`.                                                                                                                                                                                                                                         |

![https://vega.github.io/vega/docs/transforms/linkpath/](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658715840252_+2022-07-25+10.23.47.png)

# 2. “scales”:
      scales: [
        {
          // calculates horizontal stack positioning
          name: x
          type: band
          range: width
          domain: ["stk1", "stk2"]
          paddingOuter: 0.05
          paddingInner: 0.95
        }
        {
          // this scale goes up as high as the highest y1 value of all nodes
          name: y
          type: linear
          range: height
          domain: {data: "nodes", field: "y1"}
        }
        {
          // use rawData to ensure the colors stay the same when clicking.
          name: color
          type: ordinal
          range: category
          domain: {data: "rawData", fields: ["stk1", "stk2"]}
        }
        {
          // this scale is used to map internal ids (stk1, stk2) to stack names
          name: stackNames
          type: ordinal
          range: ["Source", "Destination"]
          domain: ["stk1", "stk2"]
        }
      ]

**“type” :  “band”**
將此 domain 映射到連續的(continuous)數字輸出範圍，例如像素(pixels)。
離散輸出值(Discrete output values)通過將連續範圍劃分為均勻*帶(*uniform bands*)*而由刻度自動計算(automatically computed by the scale)。

除了標準數值 *range* value（例如`[0, 500]`），band scales 可以為每個band指定一個固定的 step。
步長由具有*step*屬性的對象指定，該屬性以像素為單位提供步長，例如`"range": {"step": 20}`.

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658717784808_band.png)


**“type” :  “linear”**

每個範圍值 *y* 可以表示為 domain value *x*的線性函數：*y = mx + b*。

| paddingInner | [Number](https://vega.github.io/vega/docs/types/#Number) | 每個波段步內(band step)的內部填充(inner padding)（間距spacing），作為步長(step size)的一部分（默認值`0`）。<br>該值必須在 [0,1] 範圍內。            |
| ------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| paddingOuter | [Number](https://vega.github.io/vega/docs/types/#Number) | 縮放範圍末端（the ends of the scale range）的外部填充(outer padding)（間距），作為步長(step size)的一部分（默認值`0`）。<br>該值必須在 [0,1] 範圍內。 |

**“type” :  “ordinal” 「顏色」**
具有離散的域和範圍（discrete domain and range）。
例如，序數比例可能將一組命名類別 映射 到一組顏色或一組形狀。
序數尺度用作（Ordinal scales function ）從域值到範圍值的“查找表（lookup table）”。

“range”:{“scheme”:      }是 設置顏色 的部分，可點網址到 schemes 看有哪些。

![https://vega.github.io/vega/docs/schemes/](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658719486179_+2022-07-25+11.24.37.png)

# 3. “axes”
      axes: [
        {
          // x axis should use custom label formatting to print proper stack names
          orient: bottom
          scale: x
          encode: {
            labels: {
              update: {
                text: {scale: "stackNames", field: "value"}
              }
            }
          }
        }
        {orient: "left", scale: "y"}
      ]
| **Property** | **Type**                                                   | **Description**                                                                                                                                                        |
| ------------ | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| scale        | [String](https://vega.github.io/vega/docs/types/#String)   | ***必需的。*** 支持（ backing）軸組件（axis component）的比例（ scale）的名稱。 ******                                                                                                       |
| orient       | [String](https://vega.github.io/vega/docs/types/#String)   | ***必需的。*** **Axes** 的方向。                                                                                                                                               |
| encode       | [Object](https://vega.github.io/vega/docs/types/#Object)   | 自定義axis樣式的可選標記編碼。<br>（Optional mark encodings for custom axis styling. ）<br><br>Supports encoding blocks for `axis`, `ticks`, `grid`, `labels`, `title`, and `domain`. |
| labels       | [Boolean](https://vega.github.io/vega/docs/types/#Boolean) | A boolean flag 指示標籤是否應作為axis 的一部分包含在內 （indicating if labels should be included as part of the axis ）<br>(默認： `true`)                                                   |

# 4.”marks”
      marks: [
        {
          // draw the connecting line between stacks
          type: path
          name: edgeMark
          from: {data: "edges"}
          // this prevents some autosizing issues with large strokeWidth for paths
          clip: true
          encode: {
            update: {
              // By default use color of the left node, except when showing traffic
              // from just one country, in which case use destination color.
              stroke: [
                {
                  test: groupSelector && groupSelector.stack=='stk1'
                  scale: color
                  field: stk2
                }
                {scale: "color", field: "stk1"}
              ]
              strokeWidth: {field: "strokeWidth"}
              path: {field: "path"}
              // when showing all traffic, and hovering over a country,
              // highlight the traffic from that country.
              strokeOpacity: {
                signal: !groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 0.9 : 0.3
              }
              // Ensure that the hover-selected edges show on top
              zindex: {
                signal: !groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 1 : 0
              }
              // format tooltip string
              tooltip: {
                signal: datum.stk1 + ' → ' + datum.stk2 + '    ' + format(datum.size, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'
              }
            }
            // Simple mouseover highlighting of a single line
            hover: {
              strokeOpacity: {value: 1}
            }
          }
        }
        {
          // draw stack groups (countries)
          type: rect
          name: groupMark
          from: {data: "groups"}
          encode: {
            enter: {
              fill: {scale: "color", field: "grpId"}
              width: {scale: "x", band: 1}
            }
            update: {
              x: {scale: "x", field: "stack"}
              y: {field: "scaledY0"}
              y2: {field: "scaledY1"}
              fillOpacity: {value: 0.6}
              tooltip: {
                signal: datum.grpId + '   ' + format(datum.total, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'
              }
            }
            hover: {
              fillOpacity: {value: 1}
            }
          }
        }
        {
          // draw country code labels on the inner side of the stack
          type: text
          from: {data: "groups"}
          // don't process events for the labels - otherwise line mouseover is unclean
          interactive: false
          encode: {
            update: {
              // depending on which stack it is, position x with some padding
              x: {
                signal: scale('x', datum.stack) + (datum.rightLabel ? bandwidth('x') + 8 : -8)
              }
              // middle of the group
              yc: {signal: "(datum.scaledY0 + datum.scaledY1)/2"}
              align: {signal: "datum.rightLabel ? 'left' : 'right'"}
              baseline: {value: "middle"}
              fontWeight: {value: "bold"}
              // only show text label if the group's height is large enough
              text: {signal: "abs(datum.scaledY0-datum.scaledY1) > 13 ? datum.grpId : ''"}
            }
          }
        }
        {
          // Create a "show all" button. Shown only when a country is selected.
          type: group
          data: [
            // We need to make the button show only when groupSelector signal is true.
            // Each mark is drawn as many times as there are elements in the backing data.
            // Which means that if values list is empty, it will not be drawn.
            // Here I create a data source with one empty object, and filter that list
            // based on the signal value. This can only be done in a group.
            {
              name: dataForShowAll
              values: [{}]
              transform: [{type: "filter", expr: "groupSelector"}]
            }
          ]
          // Set button size and positioning
          encode: {
            enter: {
              xc: {signal: "width/2"}
              y: {value: 30}
              width: {value: 80}
              height: {value: 30}
            }
          }
          marks: [
            {
              // This group is shown as a button with rounded corners.
              type: group
              // mark name allows signal capturing
              name: groupReset
              // Only shows button if dataForShowAll has values.
              from: {data: "dataForShowAll"}
              encode: {
                enter: {
                  cornerRadius: {value: 6}
                  fill: {value: "#f5f5f5"}
                  stroke: {value: "#c1c1c1"}
                  strokeWidth: {value: 2}
                  // use parent group's size
                  height: {
                    field: {group: "height"}
                  }
                  width: {
                    field: {group: "width"}
                  }
                }
                update: {
                  // groups are transparent by default
                  opacity: {value: 1}
                }
                hover: {
                  opacity: {value: 0.7}
                }
              }
              marks: [
                {
                  type: text
                  // if true, it will prevent clicking on the button when over text.
                  interactive: false
                  encode: {
                    enter: {
                      // center text in the paren group
                      xc: {
                        field: {group: "width"}
                        mult: 0.5
                      }
                      yc: {
                        field: {group: "height"}
                        mult: 0.5
                        offset: 2
                      }
                      align: {value: "center"}
                      baseline: {value: "middle"}
                      fontWeight: {value: "bold"}
                      text: {value: "Show All"}
                    }
                  }
                }
              ]
            }
          ]
        }
      ]

“mark”
圖形**標記**使用幾何圖元（例如矩形、線條和繪圖符號）對數據進行可視化編碼。
標記是可視化的基本視覺構建塊，提供可以根據支持數據設置其屬性的基本形狀。

| **Property** | **Type**                                                   | **Description**                                                                                                                                                                                                                                                                  |
| ------------ | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type         | [String](https://vega.github.io/vega/docs/types/#String)   | 必需的。 marks 圖形類型。 <br>必須是支援的 marks 類型（[supported mark types](https://vega.github.io/vega/docs/marks/#types)）。                                                                                                                                                                     |
| clip         | [Clip](https://vega.github.io/vega/docs/marks/#clip)       | 指示是否應將 marks 剪裁為指定的形狀（默認：`false`）。<br><br>若是布爾值，則剪輯區域是封閉組(enclosing group)的寬度和高度。<br>若是object-valued，則應指定任意 SVG path string 或 用於裁剪到地球球體的製圖投影（cartographic projection）。                                                                                                           |
| encode       | [Encode](https://vega.github.io/vega/docs/marks/#encode)   | 包含一組 mark 屬性的視覺編碼規則（visual encoding rules）的對象（object）。                                                                                                                                                                                                                           |
| from         | [From](https://vega.github.io/vega/docs/marks/#from)       | 描述此 mark set 應可視化的數據的 object。<br>若未定義(undefined)，則假定包含一個空對象(empty object)的單元素數據集(a single element data set)。<br><br>`from` 屬性可以指定要使用的data set（例如，{"data": "table"}）或提供分面指令(faceting directive)以 跨(across) a set of group marks 細分（subdivide ）數據集。                                |
| zindex       | [Number](https://vega.github.io/vega/docs/types/#Number)   | 整數 z-index 指示此 mark set 相對於 other marks, axes, or legends 的分層(layering)。<br><br>默認為0，數值介於[0-1]，將造成 此 mark set 繪製（drawn on）在具有較低 z-index 值的其他 mark, axis, or legend 定義之上。<br><br>請注意，此值適用於set中的所有marks，而不是單個標記項(individual mark *items*)。要調整集合中項目的順序，請使用 zindex encoding channel。 |
| interactive  | [Boolean](https://vega.github.io/vega/docs/types/#Boolean) | 一個布爾標誌（默認為 `true`），指示 marks 是否可以用作輸入事件源。<br><br>若為 `false`，則不會生成與 marks 對應的鼠標或觸摸(touch)事件。<br>此屬性還可以採用 [Signal](https://vega.github.io/vega/docs/types/#Signal) 值來動態切換交互狀態(dynamically toggle interactive status)。                                                               |

        {
          // draw the connecting line between stacks
          type: path
          name: edgeMark
          from: {data: "edges"}
          // this prevents some autosizing issues with large strokeWidth for paths
          clip: true
          encode: {
            update: {
              // By default use color of the left node, except when showing traffic
              // from just one country, in which case use destination color.
              stroke: [
                {
                  test: groupSelector && groupSelector.stack=='stk1'
                  scale: color
                  field: stk2
                }
                {scale: "color", field: "stk1"}
              ]
              strokeWidth: {field: "strokeWidth"}
              path: {field: "path"}
              // when showing all traffic, and hovering over a country,
              // highlight the traffic from that country.
              strokeOpacity: {
                signal: !groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 0.9 : 0.3
              }
              // Ensure that the hover-selected edges show on top
              zindex: {
                signal: !groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 1 : 0
              }
              // format tooltip string
              tooltip: {
                signal: datum.stk1 + ' → ' + datum.stk2 + '    ' + format(datum.size, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'
              }
            }
            // Simple mouseover highlighting of a single line
            hover: {
              strokeOpacity: {value: 1}
            }
          }
        }

**“type” : “path”**

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658732122755_+2022-07-24+10.09.25.png)

| **Property**                                                                                                                                                                            | **Type**                                                 | **Description**                                                                                                                                                                                                                                                                        |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| path                                                                                                                                                                                    | [String](https://vega.github.io/vega/docs/types/#String) | An [SVG path string](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths) 描述 path 的幾何形狀（geometry）。                                                                                                                                                                         |
| stroke                                                                                                                                                                                  | [Color](https://vega.github.io/vega/docs/types/#Color)   | 描邊（stroke）顏色。                                                                                                                                                                                                                                                                          |
| strokeWidth                                                                                                                                                                             | [Number](https://vega.github.io/vega/docs/types/#Number) | stroke 寬度，像素（pixels）為單位。                                                                                                                                                                                                                                                               |
| strokeOpacity                                                                                                                                                                           | [Number](https://vega.github.io/vega/docs/types/#Number) | stroke 不透明度從 0（透明）到 1（不透明）。                                                                                                                                                                                                                                                            |
| tooltip<br><br>![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658733245633_+2022-07-25+3.13.47.png)<br><br><br>小字方格是工具提示 | [Any](https://vega.github.io/vega/docs/types/#Any)       | 鼠標懸停時 顯示的工具提示（tooltip）文本。<br><br>若該值是一個 object（日期或數組（array）除外），則該 object 中的所有鍵值對（key-value pairs）都將顯示在工具提示中（每行一個）<br> (e.g., `"key1: value1\nkey2: value2"`).<br><br>數組值（Array values）將顯示在括號 `[value1, value2, ...]` 中。<br>其他值將被強制轉換為字符串。<br>巢狀 （Nested ）object 值不會 遞迴（recursively）印出. |
| `hover`                                                                                                                                                                                 |                                                          | 鼠標懸停時 會使用該集。                                                                                                                                                                                                                                                                           |

    {
          // draw stack groups (countries)
          type: rect
          name: groupMark
          from: {data: "groups"}
          encode: {
            enter: {
              fill: {scale: "color", field: "grpId"}
              width: {scale: "x", band: 1}
            }
            update: {
              x: {scale: "x", field: "stack"}
              y: {field: "scaledY0"}
              y2: {field: "scaledY1"}
              fillOpacity: {value: 0.6}
              tooltip: {
                signal: datum.grpId + '   ' + format(datum.total, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'
              }
            }
            hover: {
              fillOpacity: {value: 1}
            }
          }
        }

**“type” : “rect”**
矩形標記在各種可視化中都很有用，包括條形圖和時間線。

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658732914370_+2022-07-25+2.56.19.png)

| y2          | [數字](https://vega.github.io/vega/docs/types/#Number)     | 以像素為單位的次要 y 坐標。                                                                                                                                                                                                                                                                        |
| ----------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fillOpacity | [Number](https://vega.github.io/vega/docs/types/#Number) | 填充不透明度從 0（透明）到 1（不透明）。                                                                                                                                                                                                                                                                 |
| tooltip     | [Any](https://vega.github.io/vega/docs/types/#Any)       | 鼠標懸停時 顯示的工具提示（tooltip）文本。<br><br>若該值是一個 object（日期或數組（array）除外），則該 object 中的所有鍵值對（key-value pairs）都將顯示在工具提示中（每行一個）<br> (e.g., `"key1: value1\nkey2: value2"`).<br><br>數組值（Array values）將顯示在括號 `[value1, value2, ...]` 中。<br>其他值將被強制轉換為字符串。<br>巢狀 （Nested ）object 值不會 遞迴（recursively）印出. |

    {
          // draw country code labels on the inner side of the stack
          type: text
          from: {data: "groups"}
          // don't process events for the labels - otherwise line mouseover is unclean
          interactive: false
          encode: {
            update: {
              // depending on which stack it is, position x with some padding
              x: {
                signal: scale('x', datum.stack) + (datum.rightLabel ? bandwidth('x') + 8 : -8)
              }
              // middle of the group
              yc: {signal: "(datum.scaledY0 + datum.scaledY1)/2"}
              align: {signal: "datum.rightLabel ? 'left' : 'right'"}
              baseline: {value: "middle"}
              fontWeight: {value: "bold"}
              // only show text label if the group's height is large enough
              text: {signal: "abs(datum.scaledY0-datum.scaledY1) > 13 ? datum.grpId : ''"}
            }
          }
        }

**“type” :  “text”**

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658733500352_+2022-07-24+10.52.52.png)

| yc         | [Number](https://vega.github.io/vega/docs/types/#Number)                                                               | 中心 y 坐標。與 y 和 y2 不兼容。                                                                                                                                                                                                                   |
| ---------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| align      | [String](https://vega.github.io/vega/docs/types/#String)                                                               | 水平文本對齊方式。<br>One of `left` (默認), `center`, or `right`.                                                                                                                                                                                  |
| baseline   | [String](https://vega.github.io/vega/docs/types/#String)                                                               | 垂直文本基線（baseline）。 <br>One of `alphabetic` (默認), `top`, `middle`, `bottom`, `line-top`, or `line-bottom`. <br><br>The `line-top` 和 `line-bottom` 值 **≥ 5.10** 的操作類似於  `top` and `bottom`, <br>但是是相對於 `lineHeight` 而不是單獨的 `fontSize` 計算的。 |
| fontWeight | [String](https://vega.github.io/vega/docs/types/#String) | [Number](https://vega.github.io/vega/docs/types/#Number)    | 字體(font )粗細 (e.g., `normal` or `bold`).                                                                                                                                                                                                 |
| text       | [String](https://vega.github.io/vega/docs/types/#String) | [String](https://vega.github.io/vega/docs/types/#String)[ ] | 要顯示的文本。<br>如果文本的呈現長度超過限制參數（parameter），則該文本可能會被截斷（truncated）。<br>對於 ≥ 5.7 的版本，字符串數組（string array ）指定多行文本。<br>對於 ≥ 5.10 的版本，所有文本行在渲染（rendering）之前都經過空白修剪（ white-space trimmed）。                                                           |

    {
          // Create a "show all" button. Shown only when a country is selected.
          type: group
          data: [
            // We need to make the button show only when groupSelector signal is true.
            // Each mark is drawn as many times as there are elements in the backing data.
            // Which means that if values list is empty, it will not be drawn.
            // Here I create a data source with one empty object, and filter that list
            // based on the signal value. This can only be done in a group.
            {
              name: dataForShowAll
              values: [{}]
              transform: [{type: "filter", expr: "groupSelector"}]
            }
          ]
          // Set button size and positioning
          encode: {
            enter: {
              xc: {signal: "width/2"}
              y: {value: 30}
              width: {value: 80}
              height: {value: 30}
            }
          }
          marks: [
            {
              // This group is shown as a button with rounded corners.
              type: group
              // mark name allows signal capturing
              name: groupReset
              // Only shows button if dataForShowAll has values.
              from: {data: "dataForShowAll"}
              encode: {
                enter: {
                  cornerRadius: {value: 6}
                  fill: {value: "#f5f5f5"}
                  stroke: {value: "#c1c1c1"}
                  strokeWidth: {value: 2}
                  // use parent group's size
                  height: {
                    field: {group: "height"}
                  }
                  width: {
                    field: {group: "width"}
                  }
                }
                update: {
                  // groups are transparent by default
                  opacity: {value: 1}
                }
                hover: {
                  opacity: {value: 0.7}
                }
              }

**“type” : “group”**

![](https://paper-attachments.dropbox.com/s_56487B9CCCD077D1B2D578A699B720862226AABCE60EBB7B139AFF2EAEFFA4E9_1658737356195_+2022-07-25+4.12.06.png)

| cornerRadius | [Number](https://vega.github.io/vega/docs/types/#Number) | 所有四個角的圓角矩形角的半徑（默認為 0）      |
| ------------ | -------------------------------------------------------- | -------------------------- |
| fill         | [Color](https://vega.github.io/vega/docs/types/#Color)   | 填充顏色。                      |
| stroke       | [Color](https://vega.github.io/vega/docs/types/#Color)   | 描邊（stroke）顏色。              |
| strokeWidth  | [Number](https://vega.github.io/vega/docs/types/#Number) | stroke 寬度（以像素（pixels）為單位）。 |
| height       | [Number](https://vega.github.io/vega/docs/types/#Number) | mark 的高度（以像素為單位）（如果支持）。    |
| width        | [Number](https://vega.github.io/vega/docs/types/#Number) | mark 的寬度（以像素為單位）（如果支持）。    |

# 5. ”signals”

參數化可視化並可以驅動交互行為的動態變量。
信號可以在整個 Vega 規範中使用，例如定義標記屬性或數據轉換參數。
信號值是*響應式*的（ *reactive*）：它們可以響應輸入事件流、外部 API 調用或上游信號的更改而更新。

    signals: [
        {
          // used to highlight traffic to/from the same country
          name: groupHover
          value: {}
          on: [
            {
              events: @groupMark:mouseover
              update: "{stk1:datum.stack=='stk1' && datum.grpId, stk2:datum.stack=='stk2' && datum.grpId}"
            }
            {events: "mouseout", update: "{}"}
          ]
        }
        // used to filter only the data related to the selected country
        {
          name: groupSelector
          value: false
          on: [
            {
              // Clicking groupMark sets this signal to the filter values
              events: @groupMark:click!
              update: "{stack:datum.stack, stk1:datum.stack=='stk1' && datum.grpId, stk2:datum.stack=='stk2' && datum.grpId}"
            }
            {
              // Clicking "show all" button, or double-clicking anywhere resets it
              events: [
                {type: "click", markname: "groupReset"}
                {type: "dblclick"}
              ]
              update: "false"
            }
          ]
        }
      ]

“events”
事件處理程序對象包括指示要respond 哪些 *events* *的*[事件流](https://vega.github.io/vega/docs/event-streams)定義，以及 用於設置 new signal value的 *update* 表達式，或用於 updating 與之交互的標記的*encode* set*。*

| value  | [Any](https://vega.github.io/vega/docs/types/#Any)               | signals 的初始值（默認: `undefined`）。 <br>該值是在計算 `init` 或 `update` 表達式之前分配的。                                                                       |
| ------ | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| on     | [Handler](https://vega.github.io/vega/docs/signals/#handlers)[ ] | 一組(An array)[事件流處理程序](https://vega.github.io/vega/docs/signals/#handlers)(event stream handlers )，用於updating the signal value以response輸入事件。 |
| init   | [Expression](https://vega.github.io/vega/docs/expressions)       | `**≥4.4**`signals 的初始化表達式。此表達式將被調用一次且僅一次。*init*和*update*參數是互斥的，不能一起使用。                                                                      |
| events | [EventStream](https://vega.github.io/vega/docs/event-streams)    | 必需的。 要響應(respond)的事件。                                                                                                                       |
| update | [Expression](https://vega.github.io/vega/docs/expressions)       | events 發生時評估的表達式，結果將成為新的signal value。<br> 如果未指定編碼(*encode*)，則此屬性(property)是必需的。                                                             |


