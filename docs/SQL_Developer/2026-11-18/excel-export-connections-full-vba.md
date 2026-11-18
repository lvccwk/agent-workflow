完整版，已經包括之前所有改動：Type 顯示文字、新 sheet 放最後並命名、加 Excel Name 同 File Location。直接成段覆蓋原本嘅代碼就得。

```vba
Sub ExportConnections()
    Dim ws As Worksheet, cn As WorkbookConnection
    Dim r As Long, i As Long, cmd As Variant, conn As Variant
    Dim usedIn As String, mFormula As String, typeText As String
    Dim pt As PivotTable, sh As Worksheet

    ' 如果已有舊嘅匯出 sheet，先刪除
    Application.DisplayAlerts = False
    On Error Resume Next
    ThisWorkbook.Worksheets("Connections_Export").Delete
    On Error GoTo 0
    Application.DisplayAlerts = True

    ' 喺最後面新增 sheet
    Set ws = ThisWorkbook.Worksheets.Add( _
             After:=ThisWorkbook.Sheets(ThisWorkbook.Sheets.Count))
    ws.Name = "Connections_Export"

    ws.Range("A1:H1").Value = Array("Name", "Type", "Connection String", _
                                    "Command Text", "Used In", "M Formula", _
                                    "Excel Name", "File Location")
    ws.Range("A1:H1").Font.Bold = True
    r = 2

    For Each cn In ThisWorkbook.Connections
        cmd = "": conn = "": usedIn = "": mFormula = "": typeText = ""

        On Error Resume Next

        ' Command Text 同 Connection String
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

        ' Type 轉做文字
        If cn.Type >= 1 And cn.Type <= 9 Then
            typeText = Choose(cn.Type, "OLEDB", "ODBC", "XMLMAP", "TEXT", _
                              "WEB", "DATAFEED", "MODEL", "WORKSHEET", "NOCONNECTION")
        Else
            typeText = CStr(cn.Type)
        End If

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

        ' Power Query 嘅 M 代碼
        mFormula = ThisWorkbook.Queries(Replace(cn.Name, "Query - ", "")).Formula

        On Error GoTo 0

        ' 寫入工作表
        ws.Cells(r, 1).Value = cn.Name
        ws.Cells(r, 2).Value = typeText
        ws.Cells(r, 3).Value = "'" & conn
        ws.Cells(r, 4).Value = "'" & cmd
        ws.Cells(r, 5).Value = usedIn
        ws.Cells(r, 6).Value = "'" & mFormula
        ws.Cells(r, 7).Value = ThisWorkbook.Name
        ws.Cells(r, 8).Value = ThisWorkbook.Path

        r = r + 1
    Next cn

    ws.Columns("A:H").ColumnWidth = 40
    ws.Range("A:H").WrapText = False

    MsgBox "完成！共匯出 " & (r - 2) & " 個連線。", vbInformation
End Sub
```

## 使用步驟：

1. 按 Alt + F11 打開 VBA 編輯器。
2. 喺左邊揀返之前個 Module，按 Ctrl + A 全選，刪除舊代碼。
3. 貼入上面成段代碼。
4. 返回 Excel，按 Alt + F8，揀 ExportConnections，再按「執行」。

完成後會彈出提示，結果喺最後一張叫 Connections_Export 嘅 sheet。每次重新執行都會先刪除舊嗰張，再重新產生。
