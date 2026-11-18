Excel 冇內建「匯出」按鈕，但可以用 VBA 將所有 connection 的 Command Text 同 Used In 範圍列到一張新工作表。

## VBA 做法

按 Alt + F11 → Insert → Module，貼入以下代碼後執行 ExportConnections：

```vba
Sub ExportConnections()
    Dim ws As Worksheet, cn As WorkbookConnection
    Dim r As Long, i As Long, cmd As Variant, conn As Variant
    Dim usedIn As String, pc As PivotCache, pt As PivotTable, sh As Worksheet

    Set ws = ThisWorkbook.Worksheets.Add
    ws.Range("A1:F1").Value = Array("Name", "Type", "Connection String", _
                                    "Command Text", "Used In", "M Formula")
    r = 2

    For Each cn In ThisWorkbook.Connections
        cmd = "": conn = "": usedIn = ""
        On Error Resume Next
        Select Case cn.Type
            Case xlConnectionTypeOLEDB
                cmd = cn.OLEDBConnection.CommandText
                conn = cn.OLEDBConnection.Connection
            Case xlConnectionTypeODBC
                cmd = cn.ODBCConnection.CommandText
                conn = cn.ODBCConnection.Connection
        End Select
        If IsArray(cmd) Then cmd = Join(cmd, "")
        If IsArray(conn) Then conn = Join(conn, "")

        ' Used In：Table / QueryTable 範圍
        For i = 1 To cn.Ranges.Count
            usedIn = usedIn & cn.Ranges(i).Parent.Name & "!" & _
                     cn.Ranges(i).Address(False, False) & "; "
        Next i

        ' Used In：PivotTable
        For Each sh In ThisWorkbook.Worksheets
            For Each pt In sh.PivotTables
                If pt.PivotCache.WorkbookConnection.Name = cn.Name Then
                    usedIn = usedIn & sh.Name & "!" & pt.Name & " (Pivot); "
                End If
            Next pt
        Next sh

        ws.Cells(r, 1).Value = cn.Name
        ws.Cells(r, 2).Value = cn.Type
        ws.Cells(r, 3).Value = "'" & conn
        ws.Cells(r, 4).Value = "'" & cmd
        ws.Cells(r, 5).Value = usedIn

        ' Power Query 的 M 代碼
        ws.Cells(r, 6).Value = "'" & ThisWorkbook.Queries( _
            Replace(cn.Name, "Query - ", "")).Formula
        On Error GoTo 0
        r = r + 1
    Next cn

    ws.Columns("A:F").ColumnWidth = 40
End Sub
```

## 要留意

- Power Query 連線：Command Text 通常只係 SELECT * FROM [Query1]，真正嘅查詢邏輯喺 M 代碼，所以加咗 F 欄經 ThisWorkbook.Queries(...).Formula 讀出。
- 只載入 Data Model 嘅連線，Used In 會係空白，呢個正常。
- 單一儲存格最多 32,767 字元，好長嘅 SQL 會被截斷；如有需要，可以改為寫入 .txt 檔。
- 要保留代碼，請另存為 .xlsm。

## 唔用 VBA 嘅方法

將 .xlsx 複製一份，副檔名改為 .zip 再解壓：

- `xl/connections.xml` → 每個連線嘅 command= 就係 Command Text
- `xl/queryTables/`、`xl/tables/`、`xl/pivotCache/` → 可以對到 Used In，不過要自己逐個對應，麻煩啲。

如果你上傳個檔案，我可以直接幫你抽出嚟整理成表。
