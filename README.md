1. The HTML Form: transfer_form.hta
<html>
<head>
<title>File Transfer Tool</title>
<HTA:APPLICATION
    ID="oTransfer"
    APPLICATIONNAME="FileTransferTool"
    BORDER="dialog"
    INNERBORDER="no"
    SINGLEINSTANCE="yes"
    WINDOWSTATE="normal"
    SCROLL="no"
    MAXIMIZEBUTTON="no"
    MINIMIZEBUTTON="yes"
>
</head>
<style>
body { font-family: Arial, sans-serif; font-size: 14px; margin: 15px; }
.container { width: 400px; margin: auto; }
h3 { margin-top: 0; }
.form-group { margin-bottom: 10px; }
.form-group label { display: block; font-weight: bold; }
.form-group input[type="text"] { width: 95%; padding: 5px; border: 1px solid #ccc; }
.form-group input[type="button"] { padding: 8px 15px; background-color: #007bff; color: white; border: none; cursor: pointer; }
.form-group input[type="button"]:hover { background-color: #0056b3; }
#status { margin-top: 20px; font-weight: bold; color: #333; }
</style>

<body>
<div class="container">
    <h3>Automated File Transfer</h3>
    <div class="form-group">
        <label for="msgId">Enter New Message ID:</label>
        <input type="text" id="msgId" name="msgId">
    </div>
    <div class="form-group">
        <label for="sentBy">Enter New SentBy:</label>
        <input type="text" id="sentBy" name="sentBy">
    </div>
    <div class="form-group">
        <label for="sendTo">Enter New SendTo:</label>
        <input type="text" id="sendTo" name="sendTo">
    </div>
    <div class="form-group">
        <input type="button" value="Generate & Transfer" onclick="runScript()">
    </div>
    <div id="status"></div>
</div>

<script language="VBScript">
Sub runScript()
    Dim newMsgId, newSentBy, newSendTo, password
    
    newMsgId = document.getElementById("msgId").value
    newSentBy = document.getElementById("sentBy").value
    newSendTo = document.getElementById("sendTo").value

    password = InputBox("Please enter the password:", "Password Required")
    
    If newMsgId = "" Or newSentBy = "" Or newSendTo = "" Or password = "" Then
        document.getElementById("status").innerText = "Error: All fields are required."
        Exit Sub
    End If

    Dim oShell
    Set oShell = CreateObject("WScript.Shell")

    ' Pass all four values (msgId, sentBy, sendTo, password) to the VBScript
    oShell.Run "mshta.exe """ & "C:\path\to\your\run_script.vbs" & """ """ & newMsgId & """ """ & newSentBy & """ """ & newSendTo & """ """ & password & """", 0, True
    
    document.getElementById("status").innerText = "File transfer process started."
End Sub
</script>
</body>
</html>


2. The VBScript: run_script.vbs
'----------------------------------------------------------------------------------
' VBScript to handle XML file modification using regular expressions and transfer.
' This script is called by the HTA application.
'----------------------------------------------------------------------------------

' Get the arguments passed from the HTA file
Dim newMsgId, newSentBy, newSendTo, password
newMsgId = WScript.Arguments(0)
newSentBy = WScript.Arguments(1)
newSendTo = WScript.Arguments(2)
password = WScript.Arguments(3)

' --- User Configuration ---
Dim localFilePath
localFilePath = "C:\Users\sg22906\Documents\MQMessage_Sayantam_j_LCH.txt"

Dim remoteUser, remoteServer, remotePath
remoteUser = "sg22906"
remoteServer = "hydcsapdw.nam.nsroot.net"
remotePath = "/tmp"

' --- File Modification Logic ---
Dim fso, fileContent, newContent
Set fso = CreateObject("Scripting.FileSystemObject")

If Not fso.FileExists(localFilePath) Then
    WScript.Echo "Error: File not found at " & localFilePath
    WScript.Quit
End If

fileContent = fso.OpenTextFile(localFilePath, 1).ReadAll

' Create regular expression objects for each value
Dim reMsgId, reSentBy, reSendTo
Set reMsgId = New RegExp
Set reSentBy = New RegExp
Set reSendTo = New RegExp

' Set patterns to find the tags and capture their content
' The pattern is adjusted to handle the attributes inside the tags
reMsgId.Pattern = "(<messageId.*?>)(.*?)(</messageId>)"
reMsgId.IgnoreCase = True
reMsgId.Global = True

reSentBy.Pattern = "(<sentBy.*?>)(.*?)(</sentBy>)"
reSentBy.IgnoreCase = True
reSentBy.Global = True

reSendTo.Pattern = "(<sendTo.*?>)(.*?)(</sendTo>)"
reSendTo.IgnoreCase = True
reSendTo.Global = True

' Use the regular expressions to replace the values
newContent = reMsgId.Replace(fileContent, "$1" & newMsgId & "$3")
newContent = reSentBy.Replace(newContent, "$1" & newSentBy & "$3")
newContent = reSendTo.Replace(newContent, "$1" & newSendTo & "$3")

Set fileHandle = fso.OpenTextFile(localFilePath, 2)
fileHandle.Write newContent
fileHandle.Close

WScript.Echo "File updated with new messageId, sentBy, and sendTo."

' --- File Transfer with SendKeys (Insecure) ---
Dim oShell
Set oShell = CreateObject("WScript.Shell")

Dim scpCommand
scpCommand = "scp """ & fso.GetAbsolutePathName(localFilePath) & """ " & remoteUser & "@" & remoteServer & ":" & remotePath

oShell.Run "C:\Program Files\Git\bin\bash.exe", 1, False
WScript.Sleep 2000
oShell.AppActivate("Git Bash")
oShell.SendKeys scpCommand
oShell.SendKeys "{ENTER}"

WScript.Sleep 5000
oShell.SendKeys password
oShell.SendKeys "{ENTER}"

WScript.Echo "Transfer complete. Check Git Bash window for final status."
