# Vega問題week2
題目：

1. 不明所以的語法
    1. Stacked Bar Chart 
        1. 這一段是修改什麼的地方，與後續什麼地方聯動
      "scales": [
        ...
        {
          "name": "color",
          "type": "ordinal",
          "range": "category",
          "domain": {"data": "table", "field": "c"}
        }
      ],

**"type": "ordinal"****：**
Ordinal scales (`ordinal`)： have a discrete domain(離散域) and range
an ordinal scale 可將一組命名類別 (a set of named categories) 對應 到 一組顏色 或 一組形狀。

改顏色：<下方左圖>

    改成："range": {"scheme": "category20"} 就可以自己設定顏色
    
    內有其他顏色組合的名字：
    https://vega.github.io/vega/docs/schemes/#categorical

**“range”：”category”：**

    指定range範圍的方式：
    - 作為範圍值的數字。例如，`[0, 500]`或`['a', 'b', 'c']`。
    - 範圍值數的signal引用，例如，`{"signal": "myRange"}`。
    - 調色板的配色，例如，`{"scheme": "blueorange"}`。
    - **僅適用**`[**ordinal**](https://vega.github.io/vega/docs/scales/#ordinal)`，一組不同字段值的[數據參考](https://vega.github.io/vega/docs/scales/#dataref)。例如，`{"data": "table", "field": "value"}`。
    - 僅適用於`[band](https://vega.github.io/vega/docs/scales/#band)`和`[point](https://vega.github.io/vega/docs/scales/#point)`，每個範圍帶的step。例如，`{"step": 20}`。
    - 表示預定義[比例範圍默認值](https://vega.github.io/vega/docs/scales/#range-defaults)的字符串。例如`"width"`、`"symbol"`或`"diverging"`。
| `"category"` | 用於默認數據的[分類配色方案。](https://vega.github.io/vega/docs/schemes/#categorical) |

    [#](https://vega.github.io/vega/docs/types/#Field) **Field**
    Accepts a string indicating the name of a data field. For example: `"amount"`, `"source.x"`, `"target['x']"`.
    Alternatively, accepts an object with a string-valued `field` parameter. For example: `{"field": "amount"}`, `{"field": "source.x"}`. In addition, the `as` parameter can be used to specify a different output name for a field. For example: `{"field": "inputName", "as": "outputName"}`.
    
    接受指示數據字段名稱的字符串。 例如：`"amount"`、`"source.x"`、`"target['x']"`。
    或者，接受一個帶有字符串值 `field` 參數的對象。 例如：`{"field": "amount"}`、`{"field": "source.x"}`。 此外，`as` 參數可用於為字段指定不同的輸出名稱。 例如：`{"field": "inputName", "as": "outputName"}`。

**“****domain": {"data": "table", "field": "c"}**



![](https://paper-attachments.dropbox.com/s_0FFA9623E72EB24E6494BDFC6AF478A032BF192BF2D5F711F678EA654AE2983D_1656490445421_+2022-06-29+161344.png)
![](https://paper-attachments.dropbox.com/s_0FFA9623E72EB24E6494BDFC6AF478A032BF192BF2D5F711F678EA654AE2983D_1656490439837_+2022-06-29+160846.png)




        1. zindex 是什麼
    zindex

整數 z-index 在 axis group 中，關係到其他的`axis`, `mark`, and `legend` groups. 
默認值為 0 。
當zindex為0，軸和網格線 畫在任何marks後，定義在同一規範級別(specification level)中
當zindex為１，將在marks上方繪製軸和網格線。

整數 z-index 在 axis group 中，關係到其他的`axis`, `mark`, and `legend` groups. 
默認值為 0 。
當zindex為0，axes and grid lines are drawn *behind* any marks defined in the same specification level. 
當zindex為１，will cause axes and grid lines to be drawn on top of marks.

        1. fill 的用途是什麼?
    "fill": {"scale": "color", "field": "c"}

"marks"的type是"rect"，會呈現可設定位置、寬度和高度的矩形。Rect marks 在各可視化圖形中都很有用，包含條形圖和時間線。
其中 fill 和顏色相關，用以填充圖內顏色。

        1. update 的作用是什麼?
            "update": {
              "fillOpacity": {"value": 1}
            },

每個mark都支持一組視覺屬性編碼( visual encoding properties)，這些屬性設定 mark 實例（instance）的位置和外觀。
共有三個主要 property sets： `*enter*`, `*update*`, `*exit*`.這三個元素可以處理當畫面元素 ( Elements ) 和資料數量 ( data ) 不相等的情形。

    > The "update" properties are evaluated for all existing (non-exiting) mark instances.




補充知識：

1.  名目資料(nominal data)：
        名目資料能區分不同組別，例如：將「性別」區分成「男」、「女」。以下是名目資料的特性：
    - 名目內容（如：「男」、「女」）本身具有意義，但編碼後 （如「男」為「1」、「女」為「0」） 的數字大小，並不代表任何意義（如，不能說1大於0）。
    - 編碼後的數字不能排序，但在統計處理時，可以累加次數(頻率數，也就是符合的人數)，例如男性156人、女性182人，或按次數多寡依序排列找出最高數值(最多人選擇的選項次數)
![](https://paper-attachments.dropbox.com/s_0FFA9623E72EB24E6494BDFC6AF478A032BF192BF2D5F711F678EA654AE2983D_1656492087713_1_SSHsD0yOIeAuRXBVLGKqSA.png)



2. Grouped Bar Chart
        1. range 為什麼需要 scheme?
          "range": {"scheme": "category20"}

 使用scheme去套用檔案中catagory20的顏色方案，也可以使用{”value”:”red”}的方式去設定顏色，或以陣列[“red”,”white”]方式設定複數的值。


        1. 總共有幾種 type
      "marks": [
        {
          "type": "group",
          
![](https://paper-attachments.dropbox.com/s_EA4DFCEA44269F3D33A9A648A287AF4ED62018B15DAEF89F4E1CE3A0C1369C8A_1656637989155_2022-07-01+090620.png)


 Supported Mark Types: `arc`, `area`, `image`, `line`, `path`, `rect`, `rule`, `shape`, `symbol`, `symbol`,`text`, `trail`. 每個標記都支持一組視覺化編碼屬性，這些屬性確定標記實例的位置和外觀，標記類型，必須是受支持的類型。如這裡最上方marks的type預設為`group` 因為這張圖還分為三個子圖表以A、B、C作為三個子圖。這裡還是定義整張圖表屬性。

![](https://paper-attachments.dropbox.com/s_EA4DFCEA44269F3D33A9A648A287AF4ED62018B15DAEF89F4E1CE3A0C1369C8A_1656639623391_2022-07-01+094008.png)


在下方marks為圖表要以哪種視覺化的顯現方式，這裡預設為`rect`代表要以矩形的方式作呈現這還可以用`group`做代替，若用`line`或`area`的話圖型會以面積的方式呈現而且沒辦法以顏色做分類。









        1. 怎麼設定讓字從黑色變成白色 (這段是什麼意思)
                    {"test": "contrast('white', datum.fill) > contrast('black', datum.fill)", "value": "white"},

 比較色彩的對比度，若白色數值大於黑色，則顯示字體為白色，類似if的條件式，後方的value改為black即可將目前表單的白字改為黑色

        1. align 在什麼情況下才會有作用? 
                  "align": {"value": "right"},

 設定文字的水平對齊方式，這邊設為靠右對齊，可將值置換為`left`或`center`

        1. baseline 是設定什麼的
    "baseline": {"value": "middle"},

 設定文字的垂直基線
 
 


1. 改設定
    1. Stacked Bar Chart 
        1. 增加對應數字
        2. 把疊加兩個柱狀圖改為並列
        3. 依照數值修改顏色 (大於50 修改顏色)
        4. 添加控制條
    2. Grouped Bar Chart
        1. 把圖豎過來
        2. 數字改到Bar 右側
        3. 改成並列改成疊加
        4. Bar 中數字顏色 設定為 3 個 顏色(黑，白，紅)，而不是現在的黑白
        5. 添加控制條
2. 新圖

