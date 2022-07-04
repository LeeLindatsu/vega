# Vega_pie-chart

https://vega.github.io/editor/#/examples/vega/pie-chart

      "$schema": "https://vega.github.io/schema/vega/v5.json",
      "description": "A basic pie chart example.",
      "width": 200,
      "height": 200,
      "autosize": "none",

“autosize”:

    Sets how the visualization size should be determined.
    如果是 string ,會是以下其中之一： `pad` (默認), `fit`, `fit-x`, `fit-y`, or `none`. 
    Object values 可以另外指定內容大小和自動調整大小的參數。


![](https://paper-attachments.dropbox.com/s_B98F0F6B55F6FA4E7AC30E6F1BF7D1C43DCFEEEE8197E4107030E6E7A7D125D3_1656774582914_+2022-07-02+11.07.16.png)

![](https://paper-attachments.dropbox.com/s_B98F0F6B55F6FA4E7AC30E6F1BF7D1C43DCFEEEE8197E4107030E6E7A7D125D3_1656774744640_+2022-07-02+11.10.36.png)


設定顏色：

      "scales": [
        {
          "name": "color",
          "type": "ordinal",
          "domain": {"data": "table", "field": "id"},
          "range": {"scheme": "category20"}
        }
      ],
## "scales"
    “domain”：將前面設定圓餅圖數值的“data“帶入“scales”的參數。
                The domain of input data values for the scale. 
                For quantitative data, this can take the form of a two-element array with minimum and maximum values. 
                For ordinal or categorical data，可能是有效輸入值的數組。
    
    **“type”** ：
![](https://paper-attachments.dropbox.com/s_B98F0F6B55F6FA4E7AC30E6F1BF7D1C43DCFEEEE8197E4107030E6E7A7D125D3_1656775762946_+2022-07-02+11.29.08.png)

    - Quantitative Scales：<定量>
            - A quantitative scale maps a continuous domain (numbers or dates) to a continuous output range (pixel locations, sizes, colors). 
            
            - All quantitative scales（except for `time` and `utc` ）use a 默認 domain of [0, 1] and 默認 unit range [0, 1].
            
            - All quantitative scales support color-valued ranges, defined either as an array of color strings or as a scheme specification.
                >  If the domain includes two color values, a sequential color scale is used.
                >  If the domain includes three color values, a diverging color scale is used. 



    - Discrete Scales：<離散 domain range>
            - Discrete scales map values from a discrete domain to a discrete range. 
        
            - In the case of band and point scales, the range is determined by discretizing a continuous numeric range.


    - Discretizing Scales：<連續domain → 離散>
            - break up a continuous domain into discrete segments, and then map values in each segment to a range value.




    “range”：
    在此是作為顏色設定的參數，例如，`{"scheme": "blueorange"}`。
    Color schemes provide a set of named color palettes for both discrete and continuous color encodings.
        > Vega顏色配置： [Cynthia Brewer](https://en.wikipedia.org/wiki/Cynthia_Brewer) and the [ColorBrewer](http://colorbrewer2.org/) project,
        >                                 or by [Maureen Stone](https://research.tableau.com/user/maureen-stone) of Tableau Software.


    **Discrete** **color schemes** may be used directly with scales that have discrete (or discretizing) domains, such as `[ordinal](https://vega.github.io/vega/docs/scales/#ordinal)`, `[quantize](https://vega.github.io/vega/docs/scales/#quantize)`, and `[quantile](https://vega.github.io/vega/docs/scales/#quantile)` scales. 


    **Continuous** **color schemes** can be used directly with continuous scales (such as `[linear](https://vega.github.io/vega/docs/scales/#linear)`, `[log](https://vega.github.io/vega/docs/scales/#log)`, and `[sqrt](https://vega.github.io/vega/docs/scales/#sqrt)` scales), and – by specifying a scheme `count` property – can also be used to generate discrete color schemes.
| **Property** | **Description**                                                                                                     |
| ------------ | ------------------------------------------------------------------------------------------------------------------- |
| category     | Default [color scheme](https://vega.github.io/vega/docs/schemes) for categorical data.                              |
| diverging    | Default [color scheme](https://vega.github.io/vega/docs/schemes) for diverging quantitative ramps.（分散漸層）            |
| heatmap      | Default [color scheme](https://vega.github.io/vega/docs/schemes) for quantitative heatmaps.（熱圖）                     |
| ordinal      | Default [color scheme](https://vega.github.io/vega/docs/schemes) for rank-ordered data. （排序）                        |
| ramp         | Default [color scheme](https://vega.github.io/vega/docs/schemes) for sequential quantitative ramps. （連續漸層）          |
| symbol       | Array of [symbol](https://vega.github.io/vega/docs/marks/symbol) names or paths for the default shape palette. （預設） |

    https://vega.github.io/vega/docs/schemes/


      "marks": [
        {
          "type": "arc",
          "from": {"data": "table"},
          "encode": {
            "enter": {
              "fill": {"scale": "color", "field": "id"},
              "x": {"signal": "width / 2"},
              "y": {"signal": "height / 2"}
            },
            "update": {
              "startAngle": {"field": "startAngle"},
              "endAngle": {"field": "endAngle"},
              "padAngle": {"signal": "padAngle"},
              "innerRadius": {"signal": "innerRadius"},
              "outerRadius": {"signal": "width / 2"},
              "cornerRadius": {"signal": "cornerRadius"}

“marks”：

    - “from”：
    An object describing the data this mark set should visualize.
    如果未定義，則假定包含一個 empty object的 single element data set。
    from 屬性可以指定要使用的data set（例如，{"data": "table"}）或提供分面指令以跨一組組標記細分數據集。
    - “encode”：
    The name of a mark property encoding set to re-evaluate for the mark item that was the source of the input event. 
    如果未指定 *update*，則此屬性是必需的。




![](https://paper-attachments.dropbox.com/s_B98F0F6B55F6FA4E7AC30E6F1BF7D1C43DCFEEEE8197E4107030E6E7A7D125D3_1656855006579_+2022-07-03+2.50.22.png)

![](https://paper-attachments.dropbox.com/s_B98F0F6B55F6FA4E7AC30E6F1BF7D1C43DCFEEEE8197E4107030E6E7A7D125D3_1656898284747_+2022-07-04+093108.png)


