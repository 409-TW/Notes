# JasperSoft Studio

## 運算式 expressions

### 數字格式 (Decimal format)

可以把數字套用特定特定格式, 像是千分位點或小數下兩位之類的
就像Excel的數字格式那樣

```java
new DecimalFormat(%format%).format(Double.parseDouble(%value%))
```
- %format% 數字格式 e.g. `"#,##0.00"`
- %value% 欲格式化的值 e.g. `$F{field1}`或`$V{variable1}`

或如果資料本身型態就是`Double`的話, 也可以這樣寫:
```java
new DecimalFormat(%format%).format(%value%)
```

### Base64轉圖片 (Base64 to image)

僅會用在`Image`元素當中, 用來在報表上顯示圖片
p.s. 以往會用路徑的方式來帶入圖片, 需要先將圖片另存於伺服器較沒效率; 隨著改用MongoDB, 檔案(Binary)可以直接存入資料庫之後, 改成以Base64字串作為資料, 直接讓Jasper讀取字串, 做起來較為方便

```java
new java.io.ByteArrayInputStream(org.apache.commons.codec.binary.Base64.decodeBase64(%value%.getBytes()))
```
- %value% 要轉成圖片的Base64字串, 可以是`Field` e.g. `$F{logo}`

## 元素 elements

### 表單(Table)

建立時，Dataset Run要選擇子資料，並勾選"Use a JRDatasource expression"，然後填入下方代碼

```java
((net.sf.jasperreports.engine.data.JsonDataSource)$P{REPORT_DATA_SOURCE}).subDataSource(%dataSource%)
```

- %dataSource% 子資料於主資料中的key，比方說主資料為
```json
{	
  header: {...},
  footer: {...},
  data: [
    {...},
    {...}
  ]
}
```
  
  則%dataSource%填入"data"

### 交叉表 (Crosstab)

建立時，需要選擇Rows, Columns 以及 Measures。

- Rows - 水平向單位的欄位

- Columns - 垂直向單位的欄位

- Measures - 運算結果(值)

  |      | Column  | Column  |
  | ---- | ------- | ------- |
  | Row  | Measure | Measure |
  | Row  | Measure | Measure |

Japser 預設 Crosstab 必須有 Total(總計欄位)，但我們實務上可能不需要此欄位，然而直接刪除的話會將整個 Column 都刪掉，因此需要將所有的 Total 欄位的 width(寬度) 或 height(高度) 設為 0 來隱藏它，視情況決定調整寬或高。

若有遇到 Row 或 Column 需要跨欄置中的狀況時，請在左下角(Outline)中點開 Row(Column) Groups，並在其中找到欲跨欄置中的欄位，點擊後於右下角(Cell)中設定 Row(Column) Positoin 為 "Center" 即可。

## 設定 settings

### 令 footer 接續在 details 後方:

1. 先將 footer 的內容放置於 column footer
2. 點選左下角(Outline)中的根部(report name)
3. 接著將右下角 Report 相關設定中的 "Float Column Footer" 勾選即可

### 分欄

1. 點選左下角(Outline)中的根部(report name)
2. 接著點開右下角最下面的 Edit Page Format
3. 右下角的 Columns 都是關於分欄的設定

# JasperStarter

### 產出報表

```sh
jasperstarter process "%jasperFile%" -o "%output%" -f %outFormat% -t %inputFormat% --data-file %input%
```

- %jasperFile% jasper 檔的絕對路徑(完整檔名)
- %output% 產出檔案的絕對路徑(不用副檔名)
- %outFormat% 輸出格式(pdf, xlsx, docx, ...etc)
- %inputFormat% 輸入格式(json, csv, ...etc)
- %input% 輸入檔案的絕對路徑(完整檔名)
