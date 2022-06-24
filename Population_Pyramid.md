# Vega 圖表學習-Population Pyramid 

https://vega.github.io/editor/#/examples/vega/population-pyramid

    {
      "$schema": "https://vega.github.io/schema/vega/v5.json",
      "description": "A population pyramid showing U.S. demographics from 1850 to 2000.",
      "height": 400,
      "padding": 5,
      "signals": [
        { "name": "chartWidth", "value": 300 },
        { "name": "chartPad", "value": 20 },
        { "name": "width", "update": "2 * chartWidth + chartPad" },
        { "name": "year", "value": 2000,
          "bind": {"input": "range", "min": 1850, "max": 2000, "step": 10} }
      ],
![](https://paper-attachments.dropbox.com/s_9C20C735456B085724893500E200D82545A9B439B18F53E4F0FAEC5BADE8F6B5_1655995049680_+2022-06-23+10.14.50.png)

- “signals”:
        - “bind”：將 signals 綁定到外部輸入元素，例如滑塊、選擇列表或單選按鈕組。


      "data": [
        {
          "name": "population",
          "url": "data/population.json"
        },
        {
          "name": "popYear",
          "source": "population",
          "transform": [
            {"type": "filter", "expr": "datum.year == year"}
          ]
        },
        {
          "name": "males",
          "source": "popYear",
          "transform": [
            {"type": "filter", "expr": "datum.sex == 1"}
          ]
        },
        {
          "name": "females",
          "source": "popYear",
          "transform": [
            {"type": "filter", "expr": "datum.sex == 2"}
          ]
        },
        {
          "name": "ageGroups",
          "source": "population",
          "transform": [
            { "type": "aggregate", "groupby": ["age"] }
          ]
        }
      ],
- “data”：
        - “name”：***必需。***數據集的名稱。
        
        - “url”：從中加載數據集的 URL。使用 format(確保正確加載)的。 未指定格式屬性，採用JSON 格式。
        
        - “source”：此數據集一個或多個數據集的名稱。
        
        字符串：表示源數據集的名稱。數值：則指定應合併在一起的數據源名稱的集合。
        
        - “transform”：處理數據以過濾數據、計算新字或衍生新數據。
        
            - “type”:”filter” ：將數據中包含 expr 的數值去除。
            
                - “expr”：***必需。***過濾數據。若算式結果為`false` → 視為 數據要過濾的對象。
                
                - “datum.”：You can use a [datum definition](https://vega.github.io/vega-lite/docs/encoding.html#datum-def) to map a constant data value to an [encoding channel](https://vega.github.io/vega-lite/docs/encoding.html#channels) via an underlying scale by setting the `datum` property.
                                **突出顯示特定數據值。**
                                
            - “type”:”aggregate”：對輸入數據進行分組和匯總以生成輸出。
                    可用於計算數據的次數、總和、平均值和其他描述性統計信息。
                
                - “groupby”：要分組的數據段落。未指定，則使用包含所有數據的單一群組。
                


      "scales": [
        {
          "name": "y",
          "type": "band",
          "range": [{"signal": "height"}, 0],
          "round": true,
          "domain": {"data": "ageGroups", "field": "age"}
        },
        {
          "name": "c",
          "type": "ordinal",
          "domain": [1, 2],
          "range": ["#d5855a", "#6c4e97"]
        }
      ],
- “scales”：數據值（數字、日期、類別*等） →* 視覺值（像素、顏色、大小）。


        - “range”：引用 signal 的範圍值，如 {"signal": "myRange"}`。 <上面>
                        調色配置。例如，`{"scheme": "blueorange"}`。  <下面>
                        
        - "round"(布林)：（默認為 `false`）數字輸出值四捨五入為整數。助於pixel grid。
        
        - "domain"：設 domain 的數字，如`[0, 500]`或`['a', 'b', 'c']`
                        signal 設定 domain value  `{"signal": "myDomain"}`
                        
                - “data”：***必需。***含domain values的數據集名稱。
                
                - “field”：資料欄位(data field)名稱 (e.g., `"price"`)
                
      "marks": [
        {
          "type": "text",
          "interactive": false,
          "from": {"data": "ageGroups"},
          "encode": {
            "enter": {
              "x": {"signal": "chartWidth + chartPad / 2"},
              "y": {"scale": "y", "field": "age", "band": 0.5},
              "text": {"field": "age"},
              "baseline": {"value": "middle"},
              "align": {"value": "center"},
              "fill": {"value": "#000"}
            }
          }
        },
        {
          "type": "group",
          "encode": {
            "update": {
              "x": {"value": 0},
              "height": {"signal": "height"}
            }
          },
          "scales": [
            {
              "name": "x",
              "type": "linear",
              "range": [{"signal": "chartWidth"}, 0],
              "nice": true, "zero": true,
              "domain": {"data": "population", "field": "people"}
            }
          ],
          "axes": [
            {"orient": "bottom", "scale": "x", "format": "s", "title": "Females"}
          ],



- "marks"：圖型（例如矩形、線條和繪圖符號）對數據進行可視化。
    - "type"：
![](https://paper-attachments.dropbox.com/s_9C20C735456B085724893500E200D82545A9B439B18F53E4F0FAEC5BADE8F6B5_1656046489938_+2022-06-24+125145.png)

        
    - “interactive”(布林)：（默認`true`） " marks" 是否可作為輸入事件源。
                                `false` 不會生成與標記對應的鼠標或觸摸事件。
                                還可以採用 Signal 來動態切換交互狀態。
                                
    - “from”：標記應可視化的數據的對象。
    
    - "encode”：Top-Level Mark ，含一組 "marks" 屬性的視覺編碼規則的對象。
    
        - "enter"：enter、update 和 exit 這三個元素可以處理當畫面元素 ( Elements ) 和資料數量 ( data ) 不相等的情形。
    利用 enter 放入資料，資料多少筆就會在背景「預先」產生多少個元素，當 enter 指令就會把這些預先產生的元素放到畫面裏，而資料和元素數量相等的部分，我們就稱之為「update」，
    
    資料比元素多，「enter」就會自動生成對應數量的元素來因應，如果資料比元素少，就可以用 exit 指令來列出多的元素，再利用 remove 來移除，這就是 enter 到 update 到 exit 的標準流程。
    ( enter 就是多出來的，update 就是相等，exit 就是少掉的 )
![](https://paper-attachments.dropbox.com/s_9C20C735456B085724893500E200D82545A9B439B18F53E4F0FAEC5BADE8F6B5_1656048868487_svg-d3-18-enter-update-exit.jpg)



- “scales”：決定了視覺編碼的性質


          "marks": [
            {
              "type": "rect",
              "from": {"data": "females"},
              "encode": {
                "enter": {
                  "x": {"scale": "x", "field": "people"},
                  "x2": {"scale": "x", "value": 0},
                  "y": {"scale": "y", "field": "age"},
                  "height": {"scale": "y", "band": 1, "offset": -1},
                  "fillOpacity": {"value": 0.6},
                  "fill": {"scale": "c", "field": "sex"}
                }
              }
            }
          ]
        },
        {
          "type": "group",
          "encode": {
            "update": {
              "x": {"signal": "chartWidth + chartPad"},
              "height": {"signal": "height"}
            }
          },
          "scales": [
            {
              "name": "x",
              "type": "linear",
              "range": [0, {"signal": "chartWidth"}],
              "nice": true, "zero": true,
              "domain": {"data": "population", "field": "people"}
            }
          ],
          "axes": [
            {"orient": "bottom", "scale": "x", "format": "s", "title": "Males"}
          ],
          "marks": [
            {
              "type": "rect",
              "from": {"data": "males"},
              "encode": {
                "enter": {
                  "x": {"scale": "x", "field": "people"},
                  "x2": {"scale": "x", "value": 0},
                  "y": {"scale": "y", "field": "age"},
                  "height": {"scale": "y", "band": 1, "offset": -1},
                  "fillOpacity": {"value": 0.6},
                  "fill": {"scale": "c", "field": "sex"}
                }
              }
            }
          ]
        }
      ]
    }
![](https://paper-attachments.dropbox.com/s_9C20C735456B085724893500E200D82545A9B439B18F53E4F0FAEC5BADE8F6B5_1656049722256_.PNG)



- "marks": 
    - "type": "rect" 具有給定位置、寬度和高度的矩形。 矩形標記在各種可視化中都很有用，包括條形圖和時間線。
    
    - “fillOpacity”：從 0（透明）到 1（不透明）的填充不透明度。
    
- "scales"： (默認 `linear`)
> 線性刻度 ( `linear`) 是保留比例差異的定量刻度，每個範圍值y可以表示為域值x的線性函數：y = mx + b。


    - "axes"：使用刻度(ticks)、網格線(gridlines)和標籤(labels)進行可視化比例對照
    
        - "orient"：*方向*參數
| left   | 沿圖表的左邊緣放置一個 y 軸。   |
| ------ | ------------------ |
| right  | 沿圖表的右邊緣放置一個 y 軸。   |
| top    | 沿著圖表的上邊緣放置一個 x 軸。  |
| bottom | 沿著圖表的底部邊緣放置一個 x 軸。 |

        

