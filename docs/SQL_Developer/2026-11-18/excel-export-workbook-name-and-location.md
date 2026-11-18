用 ThisWorkbook.Name 同 ThisWorkbook.Path 就得，加兩欄 G、H。

## 1. 標題改成：

```vba
ws.Range("A1:H1").Value = Array("Name", "Type", "Connection String", _
                                "Command Text", "Used In", "M Formula", _
                                "Excel Name", "File Location")
```

## 2. 喺 ws.Cells(r, 5).Value = usedIn 下面加：

```vba
ws.Cells(r, 7).Value = ThisWorkbook.Name      ' 例如 Report.xlsm
ws.Cells(r, 8).Value = ThisWorkbook.Path      ' 例如 C:\Users\xxx\Documents
```

## 3. 欄寬改成 A:H：

```vba
ws.Columns("A:H").ColumnWidth = 40
```

## 一啲補充：

- 想要包埋檔名嘅完整路徑，可以用 ThisWorkbook.FullName（例如 C:\...\Report.xlsm）。
- 如果檔案喺 OneDrive / SharePoint，Path 會顯示成 https://... 網址，唔係本機路徑，呢個正常。
- 如果檔案未儲存過，Path 會係空白。
- 如果你要嘅其實係連線資料來源嘅檔案（例如 Power Query 讀緊邊個 Excel / CSV），呢個資料喺 F 欄 M Formula 嘅 File.Contents("C:\...") 入面，或者 C 欄 Connection String 嘅 Data Source=...。有需要嘅話，我可以加一欄專門抽出嗰個路徑。
