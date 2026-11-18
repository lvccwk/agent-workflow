開一個獨立嘅「工具」Excel 放呢段代碼，佢會自動行勻 report_template 同入面所有 sub folder，逐個 Excel 打開、抽資料、再閂返（唔會儲存），最後將全部結果集中喺一張 sheet。

## 使用步驟

1. 開一個全新空白 Excel。
2. 按 Alt + F11 → Insert → Module，貼入下面成段代碼。
3. 另存為 .xlsm，例如 Desktop\Export_Tool.xlsm（唔好存喺 report_template 入面）。
4. 按 Alt + F8 → 揀 ExportAllConnections → 執行。
5. 想改路徑，就改頂部 ROOT_FOLDER 嗰行。

```vba
Option Explicit

Const ROOT_FOLDER As String = "C:\Users\abc\Desktop\report_template"

Dim gRow As Long, gFiles As Long, gErr As Long

Sub ExportAllConnections()
    Dim ws As Worksheet, fso As Object, oldSec As Long

    Set fso = CreateObject("Scripting.FileSystemObject")
    If Not fso.FolderExists(ROOT_FOLDER) Then
        MsgBox "搵唔到資料夾：" & ROOT_FOLDER, vbExclamation
        Exit Sub
    End If

    ' 準備結果 sheet（舊嘅會先刪除）
    Application.DisplayAlerts = False
    On Error Resume Next
    ThisWorkbook.Worksheets("Connections_Export").Delete
    On Error GoTo 0
    Set ws = ThisWorkbook.Worksheets.Add( _
             After:=ThisWorkbook.Sheets(ThisWorkbook.Sheets.Count))
    ws.Name = "Connections_Export"
    ws.Range("A1:H1").Value = Array("Excel Name", "File Location", "Connection Name", _
                                    "Type", "Connection String", "Command Text", _
                                    "Used In", "M Formula")
    ws.Range("A1:H1").Font.Bold = True

    ' 關閉提示、唔行其他檔案嘅 macro、唔更新外部連結
    Application.ScreenUpdating = False
    Application.EnableEvents = False
    Application.AskToUpdateLinks = False
    oldSec = Application.AutomationSecurity
    Application.AutomationSecurity = 3   ' 停用被打開檔案嘅 macro

    gRow = 2: gFiles = 0: gErr = 0
    ProcessFolder fso.GetFolder(ROOT_FOLDER), ws

    ' 還原設定
    Application.AutomationSecurity = oldSec
    Application.AskToUpdateLinks = True
    Application.EnableEvents = True
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
    Application.StatusBar = False

    ws.Columns("A:H").ColumnWidth = 40
    ws.Range("A1:H1").AutoFilter
    ws.Activate

    MsgBox "完成！" & vbCrLf & _
           "成功處理：" & gFiles & " 個檔案" & vbCrLf & _
           "開唔到：" & gErr & " 個檔案", vbInformation
End Sub

' 逐個資料夾（包括 sub folder）處理
Private Sub ProcessFolder(fld As Object, ws As Worksheet)
    Dim f As Object, sf As Object, ext As String, wb As Workbook

    For Each f In fld.Files
        ext = LCase(Mid(f.Name, InStrRev(f.Name, ".") + 1))
        If (ext = "xlsx" Or ext = "xlsm" Or ext = "xlsb" Or ext = "xls") _
           And Left(f.Name, 2) <> "~$" _
           And f.Path <> ThisWorkbook.FullName Then

            Application.StatusBar = "處理中：" & f.Path
            DoEvents

            Set wb = Nothing
            On Error Resume Next
            Set wb = Workbooks.Open(Filename:=f.Path, UpdateLinks:=0, _
                                    ReadOnly:=True, Password:="", _
                                    IgnoreReadOnlyRecommended:=True)
            On Error GoTo 0

            If wb Is Nothing Then
                ws.Cells(gRow, 1).Value = f.Name
                ws.Cells(gRow, 2).Value = fld.Path
                ws.Cells(gRow, 3).Value = "* 開唔到（可能有密碼或檔案損壞）*"
                gRow = gRow + 1
                gErr = gErr + 1
            Else
                ExtractConnections wb, ws
                wb.Close SaveChanges:=False
                gFiles = gFiles + 1
            End If
        End If
    Next f

    For Each sf In fld.SubFolders
        ProcessFolder sf, ws
    Next sf
End Sub

' 抽出單一 workbook 嘅連線資料
Private Sub ExtractConnections(wb As Workbook, ws As Worksheet)
    Dim cn As WorkbookConnection, sh As Worksheet, pt As PivotTable
    Dim cmd As Variant, conn As Variant
    Dim usedIn As String, mFormula As String, typeText As String, pcName As String
    Dim i As Long, rc As Long

    If wb.Connections.Count = 0 Then
        ws.Cells(gRow, 1).Value = wb.Name
        ws.Cells(gRow, 2).Value = wb.Path
        ws.Cells(gRow, 3).Value = "(冇連線)"
        gRow = gRow + 1
        Exit Sub
    End If

    For Each cn In wb.Connections
        cmd = "": conn = "": usedIn = "": mFormula = "": typeText = ""

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

        If cn.Type >= 1 And cn.Type <= 9 Then
            typeText = Choose(cn.Type, "OLEDB", "ODBC", "XMLMAP", "TEXT", _
                              "WEB", "DATAFEED", "MODEL", "WORKSHEET", "NOCONNECTION")
        Else
            typeText = CStr(cn.Type)
        End If

        ' Used In：Table / QueryTable
        rc = 0
        rc = cn.Ranges.Count
        For i = 1 To rc
            usedIn = usedIn & cn.Ranges(i).Parent.Name & "!" & _
                     cn.Ranges(i).Address(False, False) & "; "
        Next i

        ' Used In：PivotTable
        For Each sh In wb.Worksheets
            For Each pt In sh.PivotTables
                pcName = ""
                pcName = pt.PivotCache.WorkbookConnection.Name
                If pcName = cn.Name Then
                    usedIn = usedIn & sh.Name & "!" & pt.Name & " (Pivot); "
                End If
            Next pt
        Next sh

        ' Power Query M 代碼
        mFormula = wb.Queries(Replace(cn.Name, "Query - ", "")).Formula

        On Error GoTo 0

        ws.Cells(gRow, 1).Value = wb.Name
        ws.Cells(gRow, 2).Value = wb.Path
        ws.Cells(gRow, 3).Value = cn.Name
        ws.Cells(gRow, 4).Value = typeText
        ws.Cells(gRow, 5).Value = SafeText(CStr(conn))
        ws.Cells(gRow, 6).Value = SafeText(CStr(cmd))
        ws.Cells(gRow, 7).Value = usedIn
        ws.Cells(gRow, 8).Value = SafeText(mFormula)
        gRow = gRow + 1
    Next cn
End Sub

' 避免超過儲存格字數上限，同埋防止被當成公式
Private Function SafeText(s As String) As String
    SafeText = "'" & Left(s, 32000)
End Function
```

## 結果

全部寫入工具檔案入面嘅 Connections_Export sheet，每行係一個連線，A、B 欄係嗰個 Excel 嘅檔名同所在資料夾。

冇連線嘅檔案都會列一行「(冇連線)」，方便你確認佢有被掃描到。

有密碼或者開唔到嘅檔案會標示「開唔到」，唔會令成個程序停低。

執行期間，Excel 底部狀態列會顯示正在處理邊個檔案。

## 要留意

檔案係用唯讀方式打開，閂嘅時候唔會儲存，所以唔會改動原本嘅檔案。

如果某啲檔案設定咗「開啟檔案時重新整理」，打開時可能會自動 refresh，令速度變慢，甚至彈出登入視窗；遇到嘅話，輸入或者取消就會繼續。

幾十個檔案大約要幾分鐘，執行期間唔好郁 Excel。
