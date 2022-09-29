## 使用说明
`-components-WatermarkCamera`文件为水印相机组件，在使用到水印相机的页面中引入组件。

若需要全屏水印相机，建议在`page.json`中将`pages`中页面属性`navigationStyle`设置为`custom`

目前版本控制水印组件是用的`v-if`,后续会优化

## 参数说明
`markText`——水印文本

`markLocation`——水印位置,后续更新

`fillColor`——水印颜色，默认为`#FFFFFF`

`textSize`——文本大小，默认为30

`bgMark`——背景水印，默认不添加

`bgMarkText`——背景水印文本
