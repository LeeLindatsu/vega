# Vega 圖表學習-Radial Plot

https://vega.github.io/editor/#/edited

      "scales": [
        {
          "name": "r",
          "type": "sqrt",
          "domain": {"data": "table", "field": "data"},
          "zero": true,
          "range": [20, 100]
        }
## “scales”：
    “type”：**”sqrt”**
        Square root (`sqrt`) scales，是指數為 0.5 的 power scales 的便捷簡寫，表示平方根變換。
        > Log scales (`log`) are [quantitative scales](https://vega.github.io/vega/docs/scales/#quantitative) 指數變換應用於計算輸出範圍值之前的輸入域值。
        > 每個範圍值 y 可以表示為域值 x 的多項式函數：y = mx^k + b，其中 k 是指數值。
        
    “range”：賦予”name”:”r”此的數值為[20,100]，讓之後的參數可以使用。
            若此參數要使用，會寫 {“scale”:”r”, ”field”:”data”}
            
      "marks": [
        {
          "type": "arc",
          "from": {"data": "table"},
          "encode": {
            "enter": {
              "x": {"field": {"group": "width"}, "mult": 0.5},
              "y": {"field": {"group": "height"}, "mult": 0.5},
              "startAngle": {"field": "startAngle"},
              "endAngle": {"field": "endAngle"},
              "innerRadius": {"value": 20}, //圖內中間空心大小
              "outerRadius": {"scale": "r", "field": "data"},
              "stroke": {"value": "#fff"} //外框
            },
            "update": {
              "fill": {"value": "#ccc"} //圖內顏色
            },
            "hover": {
              "fill": {"value": "pink"} //游標移到變的顏色
            }
          }
        },
        {
          "type": "text",
          "from": {"data": "table"},
          "encode": {
            "enter": {
              "x": {"field": {"group": "width"}, "mult": 0.5},
              "y": {"field": {"group": "height"}, "mult": 0.5},
              "radius": {"scale": "r", "field": "data", "offset": 8},
              "theta": {"signal": "(datum.startAngle + datum.endAngle)/2"},
              "fill": {"value": "#000"}, //字的顏色
              "align": {"value": "center"},
              "baseline": {"value": "middle"},
              "text": {"field": "data"}
            }
          }
        }
      ]

“marks”：

    “radius”:以像素(pixels)為單位的極坐標(Polar coordinate)徑向**偏移(offset)**，相對於由 x 和 y 屬性確定的原點（默認為 0）。
    
    “theta”:以弧度(radians)表示的極坐標角度(Polar coordinate angle)，相對於由 x 和 y 屬性確定的原點（默認為 0）。theta 的值遵循相同的圓弧標記約定(convention of arc marks)：角度以弧度測量，0 表示向上或“北”。
    
    “baseline”:垂直文本基線。
        One of `alphabetic` (default), `top`, `middle`, `bottom`, `line-top`, or `line-bottom`。
        The `line-top` and `line-bottom` values **≥ 5.10** 的操作類似於  `top` and `bottom`, but are calculated relative to the *lineHeight* rather than *fontSize* alone.
        
    “text”:要顯示的文本。
           如果文本的呈現長度超過限制參數，則該文本可能會被截斷。
https://www.ifreesite.com/color/

![](https://paper-attachments.dropbox.com/s_0D4B5848844C346B2BFFBAA574BB811C67F298C2718A7B139760B2AC67CF8223_1656986477182_+2022-07-05+100059.png)

![](https://paper-attachments.dropbox.com/s_0D4B5848844C346B2BFFBAA574BB811C67F298C2718A7B139760B2AC67CF8223_1656988132442_+2022-07-05+102332.png)



    {
      "$schema": "https://vega.github.io/schema/vega/v5.json",
      "description": "A basic radial plot that encodes two values as the angle and radius of an arc.",
      "width": 200,
      "height": 200,
      //拉霸
      "signals": [
        {
          "name": "startAngle", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 6.29, "step": 0.01}
        },
        {
          "name": "endAngle", "value": 6.29,
          "bind": {"input": "range", "min": 0, "max": 6.29, "step": 0.01}
        },
        {
          "name": "padAngle", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 0.1}
        },
        {
          "name": "innerRadius", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 90, "step": 1}
        },
        {
          "name": "cornerRadius", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 100, "step": 5}
        },
        {
          "name": "sort", "value": false,
          "bind": {"input": "checkbox"}
        }
      ],//
    
      "data": [
        {
          "name": "table",
          "values": [12, 23, 47, 6, 52, 19],
          "transform": [{"type": "pie", "field": "data","startAngle": {"signal": "startAngle"}, "endAngle": {"signal": "endAngle"},"sort": {"signal": "sort"}}
        
              ]
        }
        
        
      ],
      "scales": [
        {
          "name": "r",
          "type": "sqrt",
          "domain": {"data": "table", "field": "data"},
          "zero": true,
          "range": [-100, 300] //縮進去
        }
      ],
      "marks": [
        {
          "type": "arc",
          "from": {"data": "table"},
          "encode": {
            "enter": {
              "x": {"field": {"group": "width"}, "mult": 0.5},
              "y": {"field": {"group": "height"}, "mult": 0.5},
              "startAngle": {"field": "startAngle"},
              "endAngle": {"field": "endAngle"},
              "innerRadius": {"value": 20},
              "outerRadius": {"scale": "r", "field": "data"},
              "stroke": {"value": "#66B3FF"}
            },
            "update": {
              "fill": {"value": "#4DFFFF"}, //變顏色
              "startAngle": {"field": "startAngle"},
              "endAngle": {"field": "endAngle"},
              "padAngle": {"signal": "padAngle"},
              "innerRadius": {"signal": "innerRadius"},
              "cornerRadius": {"signal": "cornerRadius"}
            },
            "hover": {
              "fill": {"value": "pink"}
            }
          }
        },
        {
          "type": "text",
          "from": {"data": "table"},
          "encode": {
            "enter": {
              "x": {"field": {"group": "width"}, "mult": 0.5},
              "y": {"field": {"group": "height"}, "mult": 0.5},
              "radius": {"scale": "r", "field": "data", "offset": 8},
              "theta": {"signal": "(datum.startAngle + datum.endAngle)/2"},
              "fill": {"value": "#000"},
              "align": {"value": "center"},
              "baseline": {"value": "middle"},
              "text": {"field": "data"}
            },
            "update": {
             }
          }
        }
      ]
    }
    

嘗試  移動字(text) & 根據數值大小顏色(fill)漸變
 移動字
 https://vega.github.io/editor/#/examples/vega/packed-bubble-chart


數值大小顏色改變
參考  Grouped Bar Chart 
https://vega.github.io/editor/#/edited

    {
      "$schema": "https://vega.github.io/schema/vega/v5.json",
      "description": "A basic radial plot that encodes two values as the angle and radius of an arc.",
      "width": 200,
      "height": 200,
     
    
      "data": [
        {
          "name": "table",
          "values": [12, 23, 47, 6, 52, 19],
            
          "transform": [{"type": "pie", "field": "data","startAngle": {"signal": "startAngle"}, "endAngle": {"signal": "endAngle"},"sort": {"signal": "sort"}}
              ]
        }
        
        
      ],
      //拉霸
      "signals": [
        
        {"name": "cx", "update": "width /2 "},
        {"name": "cy", "update": "height /2"},
        {
          "name": "startAngle", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 6.29, "step": 0.01}
        },
        {
          "name": "endAngle", "value": 6.29,
          "bind": {"input": "range", "min": 0, "max": 6.29, "step": 0.01}
        },
        {
          "name": "padAngle", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 0.1}
        },
        {
          "name": "innerRadius", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 90, "step": 1}
        },
        {
          "name": "cornerRadius", "value": 0,
          "bind": {"input": "range", "min": 0, "max": 100, "step": 5}
        },
        {
          "name": "sort", "value": true,
          "bind": {"input": "checkbox"}
        },//
      ],
      "scales": [
        {
          "name": "r",
          "type": "sqrt",
          "domain": {"data": "table", "field": "data"},
          "zero": true,
          "range": [-100, 300] //縮進去
        }
        
      ],
      "marks": [
        {
          "type": "arc",
          "from": {"data": "table"},
          "encode": {
            "enter": {
              "x": {"field": {"group": "width"}, "mult": 0.5},
              "y": {"field": {"group": "height"}, "mult": 0.5},
              "startAngle": {"field": "startAngle"},
              "endAngle": {"field": "endAngle"},
              "innerRadius": {"value": 20},
              "outerRadius": {"scale": "r", "field": "data"},
              "stroke": {"value": "#66B3FF"},
              "fill":{"value": "#4DFFFF"}
              //"fill": [{"contrast('#66B3FF', datum.data) > contrast('#66B3FF', datum.data) ", "value": "white"}, {"value": "black"}]
            },
            "update": {
              "fill":{"value": "#4DFFFF"},
               //"fill":[{"test":"contrast(datum.table.values) > contrast(10)", "value": "red"},
               //    {"value": "black"}],  //變顏色
              "startAngle": {"field": "startAngle"},
              "endAngle": {"field": "endAngle"},
              "padAngle": {"signal": "padAngle"},
              "innerRadius": {"signal": "innerRadius"},
              "cornerRadius": {"signal": "cornerRadius"}
            },
            "hover": {
              "fill": {"value": "pink"}
            }
          },
         "transform": [
            {
               "type": "force",
              "iterations": 100,
              "static": false,
              "forces": [
                {
                  "force": "collide",
                  "iterations": 2,
                  "radius": {"expr": "sqrt(datum.size) / 2"}
                },
              
                {"force": "x", "x": "xfocus", "strength": {"signal": "sort"}},
                {"force": "y", "y": "yfocus", "strength": {"signal": "sort"}}
              ]
        }]
        },
        {
     "type": "text",
          "from": {"data": "table"},
          "encode": {
            "enter": {
              "x": {"field": {"group": "width"}, "mult": 0.5},
              "y": {"field": {"group": "height"}, "mult": 0.5},
              "radius": {"scale": "r", "field": "data", "offset": 8},
              "theta": {"signal": "(datum.startAngle + datum.endAngle)/2"},
              "fill": {"value": "#000"},
              "align": {"value": "center"},
              
              "baseline": {"value": "middle"},
              "text": {"field": "data"}
            },
            "update": {"x": {"field": "x"}, "y": {"field": "y"},
            "startAngle": {"field": "startAngle"},
              "endAngle": {"field": "endAngle"},
              "padAngle": {"signal": "padAngle"},
              "innerRadius": {"signal": "innerRadius"},
              "cornerRadius": {"signal": "cornerRadius"}}
        }
        
        }
          
        
      ]
    }
    

