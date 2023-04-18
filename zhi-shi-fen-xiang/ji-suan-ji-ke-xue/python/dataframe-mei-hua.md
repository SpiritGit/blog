# DataFrame美化

Pandas提供了`DataFrame.style`属性，它会返回`Styler`对象，用以数据样式的美化





### DataFrame美化结果导出

美化一般是为了添加到数据分析相关报告中，除截图外，DataFrame美化结果还可导出为图片，以应对高清图片、或表格过大难以截图的场景。

```python
# 注意，dfi通过调用chrome输出图片，因此需确保系统已安装chrome
import dataframe_image as dfi

dfi.export(your_df_style, 'your_name.jpg')
```
