---
date: 2025-12-15
title: rename-helper
description: 파일명 base64 인코딩/디코딩
tags:
  - autohotkey
categories:
  - dev
---

![image](/images/ahk-logo.webp#center)

## GitHub repository
[바로 가기](https://github.com/dntco43u/rename_helper)

## autohotkey

### Shortcuts
| Shortcuts      | Remarks              |
|----------------|----------------------|
| `Ctrl+Shift+P` | Pause                |
| `Ctrl+Shift+R` | Reload               |
| `Ctrl+Shift+X` | Exit                 |
| `Ctrl+Shift+A` | 파일명 base64 인코딩 |
| `Ctrl+Shift+S` | 파일명 base64 디코딩 |

### rename_helper.properties
```ini
LOG_LEVEL=INFO
```

### Common.ahk [^1]
```ahk
#include <Base64>

#SingleInstance force
#NoEnv
#MaxHotkeysPerInterval 99000000
#HotkeyInterval 99000000
#KeyHistory 0
#InstallKeybdHook
#InstallMouseHook
#UseHook

ListLines Off
Process, Priority, , A
SetTitleMatchMode, 2
SetTitleMatchMode, Fast
SetBatchLines, -1
SetKeyDelay, -1, -1, Play
SetMouseDelay, -1
SetDefaultMouseSpeed, 0
SetWinDelay, -1
SetControlDelay, -1
SendMode Input
CoordMode, Mouse, Client
CoordMode, Pixel, Client
CoordMode, ToolTip, Screen

FileEncoding, UTF-8

;-------------------------------------------------------------------------------
; global variables
;-------------------------------------------------------------------------------

global CURRENT_LOG_LEVEL := 0
global LOG_LEVEL_ARRAY := [["TRACE", 1], ["DEBUG", 2], ["INFO", 3], ["WARN", 4], ["ERROR", 5], ["SEVERE", 6]]
global LOG_FILE := A_ScriptDir "\log\" SubStr(A_ScriptName, 1, -4) ".log" ;log file
global PROPERTIES_FILE := A_ScriptDir "\" SubStr(A_ScriptName, 1, -4) ".properties" ;properties file

;-------------------------------------------------------------------------------
; init
;-------------------------------------------------------------------------------

init()
;init

init() {
  getProperties(PROPERTIES_FILE) ;getProperties
  runAsAdmin()
  removeLogFile(8192)
}

;-------------------------------------------------------------------------------
; screen functions
;-------------------------------------------------------------------------------

getPixelColor(pixelX, pixelY) {
  if (pixelX = "") || (pixelY = "") {
    MouseGetPos, pixelX, pixelY
  }
  PixelGetColor, color, pixelX, pixelY
  message := "PixelGetColor() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " pixelX ", " pixelY " " color
  writeLogFile("INFO", A_ThisFunc, A_LineNumber, message)
  showToolTip("INFO", message)
  return color
}

isColorFromPixel(startX, startY, color, variation) {
  endX := startX + 1
  endY :=startY + 1
  PixelSearch, foundX, foundY, startX, startY, endX, endY, color, variation, Fast
  result := ErrorLevel = 0 ? true : false
  message := "PixelSearch() " (result ? "SUCCEED " foundX ", " foundY : "FAILED ") " " color
  writeLogFile("DEBUG", A_ThisFunc, A_LineNumber, message)
  showToolTip("WARN", message)
  if result
    return true
  else
    return false
}

;-------------------------------------------------------------------------------
; file functions
;-------------------------------------------------------------------------------

removeFile(file) {
  FileDelete, %file%
  message := "FileDelete() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " file
  writeLogFile("DEBUG", A_ThisFunc, A_LineNumber, message)
}

getTextFile(textFile) {
  FileRead, outText, %textFile%
  message := "FileRead() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " textFile
  writeLogFile("DEBUG", A_ThisFunc, A_LineNumber, message)
  message := "getTextFile() " outText " (" StrLen(outText) ")"
  writeLogFile("DEBUG", A_ThisFunc, A_LineNumber, message)
  showToolTip("DEBUG", message)
  return outText
}

setTextFile(outText, textFile) {
  FileAppend, % outText "`r`n", %textFile%
  message := "FileAppend() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " textFile
  writeLogFile("DEBUG", A_ThisFunc, A_LineNumber, message)
  message := "setTextFile() " outText " (" StrLen(outText) ")"
  writeLogFile("DEBUG", A_ThisFunc, A_LineNumber, message)
  showToolTip("DEBUG", message)
}

;-------------------------------------------------------------------------------
; common functions
;-------------------------------------------------------------------------------

getProperties(propertiesFile) {
  FileRead, properties, %propertiesFile%
  if errorlevel
    return
  oProperties := StrSplit(properties, "`r`n")
  loop, % oProperties.MaxIndex() {
    if (InStr(oProperties[A_Index], "LOG_LEVEL")) {
      logLevelProperties := StrSplit(oProperties[A_Index], "=")[2]
      loop, % LOG_LEVEL_ARRAY.MaxIndex() {
        if (logLevelProperties = LOG_LEVEL_ARRAY[A_Index][1]) {
          CURRENT_LOG_LEVEL := LOG_LEVEL_ARRAY[A_Index][2]
        }
      }
    }
  }
}

runAsAdmin() {
  message := "A_IsAdmin " (A_IsAdmin = 1 ? "SUCCEED" : "FAILED ")
  writeLogFile("DEBUG", A_ThisFunc, A_LineNumber, message)
  showToolTip("DEBUG", message)
  if A_IsAdmin
    return
  Run *RunAs "%A_ScriptFullPath%"
  ExitApp
}

writeLogFile(levelName, functionName, lineNumber, message) {
  level := 0
  loop, % LOG_LEVEL_ARRAY.MaxIndex() {
    if (levelName = LOG_LEVEL_ARRAY[A_Index][1]) {
      level := LOG_LEVEL_ARRAY[A_Index][2]
    }
  }
  if (CURRENT_LOG_LEVEL > level)
    return
  ;write log file
  FileAppend, % A_YYYY "-" A_MM  "-" A_DD " " A_Hour ":" A_Min ":" A_Sec "." A_MSec " " levelName " " A_ScriptName "." functionName "() Line " lineNumber " " message "`n", %LOG_FILE%
}

removeLogFile(thresholdSize) {
  FileGetSize, fileSize, %LOG_FILE%, K
  message := "FileGetSize() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " fileSize "KB " LOG_FILE
  writeLogFile("WARN", A_ThisFunc, A_LineNumber, message)
  if (fileSize < thresholdSize)
    return
  FileDelete, %LOG_FILE%
  message := "FileDelete() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " LOG_FILE
  writeLogFile("WARN", A_ThisFunc, A_LineNumber, message)
  showToolTip("WARN", message)
}

showToolTip(levelName, message) {
  level := 0
  loop, % LOG_LEVEL_ARRAY.MaxIndex() {
    if (levelName = LOG_LEVEL_ARRAY[A_Index][1]) {
      level := LOG_LEVEL_ARRAY[A_Index][2]
    }
  }
  if (CURRENT_LOG_LEVEL > level)
    return
  ToolTip, %message%, 5, 5
  SetTimer, removeTooltipTimer, -7000
}

removeTooltipTimer:
{
  ToolTip
  return
}

initHotkey(description) {
  if GetKeyState("Ctrl") {
    Send, {Ctrl Up}
    Sleep, 1
  }
  if GetKeyState("Alt") {
    Send, {Alt Up}
    Sleep, 1
  }
  if GetKeyState("Shift") {
    Send, {Shift Up}
    Sleep, 1
  }
  if GetKeyState("LButton") {
    Send, {LButton Up}
    Sleep, 1
  }
  if GetKeyState("RButton") {
    Send, {RButton Up}
    Sleep, 1
  }
  KeyWait, Ctrl
  KeyWait, Alt
  KeyWait, Shift
  Sleep, 20
  showToolTip("INFO", A_ScriptName "." description)
}

;playBeep(1, 7902, 80) ;SUCCEED
;playBeep(1, 2489, 80) ;FAILED
playBeep(times, freq, dur) {
  loop, % times {
    SoundBeep, freq, dur
    Sleep, 10
  }
}
```

### rename_helper.ahk [^2]
```ahk
#include <Common>

;-------------------------------------------------------------------------------
; global variables
;-------------------------------------------------------------------------------

;-------------------------------------------------------------------------------
; init
;-------------------------------------------------------------------------------

;-------------------------------------------------------------------------------
; hotkeys
;-------------------------------------------------------------------------------

^+p:: ;ctrl+shift+p
{
  initHotkey("Pause")
  Pause
  return
}

^+r:: ;ctrl+shift+r
{
  Reload
  return
}

^+x:: ;ctrl+shift+x
{
  ExitApp
}

^+`:: ;ctrl+shift+`
{
  initHotkey("getPixelColor")
  getPixelColor("", "")
  return
}

^+z:: ;ctrl+shift+z
{
  initHotkey("test")
  test()
  return
}

^+1:: ;ctrl+shift+1
{
  initHotkey("testEncodeBase64")
  renameEncodeBase64()
  renameDecodeBase64()
  return
}

^+a:: ;ctrl+shift+a
{
  initHotkey("renameEncodeBase64")
  renameEncodeBase64()
  return
}

^+s:: ;ctrl+shift+s
{
  initHotkey("renameDecodeBase64")
  renameDecodeBase64()
  return
}

;-------------------------------------------------------------------------------
; test
;-------------------------------------------------------------------------------

test() {
  message := decodeString(encodeString("rename_helper")) " -> " encodeString("rename_helper")
  writeLogFile("WARN", A_ThisFunc, A_LineNumber, message)
}

;-------------------------------------------------------------------------------
; biz functions
;-------------------------------------------------------------------------------

renameEncodeBase64() {
  ;FileSelectFolder, selectedPath, *C:\Users\%A_UserName%\Downloads\gzmo5ajq
  FileSelectFolder, selectedPath, *C:\Users\%A_UserName%\Downloads\1
  if ErrorLevel
    return
  renameEncodeFile := selectedPath "\" "renameCrypt.enc"
  renameDecodeFile := selectedPath "\" "renameCrypt.dec"
  removeFile(renameEncodeFile)
  removeFile(renameDecodeFile)
  rootPath := selectedPath
  loop Files, %selectedPath%\*.*, DFR
  {
    SplitPath, A_LoopFileFullPath, , outDir, outExt, outNameNoExt
    loopPath := StrReplace(A_LoopFileFullPath, selectedPath "\")
    cryptLoopPath := ""
    loopPathAttrib = ""
    oPathString := StrSplit(loopPath, "\")
    if (InStr(A_LoopFileAttrib, "D")) {
      loopPathAttrib := "D"
      loop, % oPathString.MaxIndex() {
        cryptLoopPath := cryptLoopPath "\" encodeString(oPathString[A_Index])
      }
    } else {
      loopPathAttrib := "F"
      loop, % oPathString.MaxIndex() {
        if (A_Index = oPathString.MaxIndex()) {
          cryptLoopPath := cryptLoopPath "\" encodeString(outNameNoExt) "." outExt
        } else {
          cryptLoopPath := cryptLoopPath "\" encodeString(oPathString[A_Index])
        }
      }
    }
    cryptLoopPath := SubStr(cryptLoopPath, 2, StrLen(cryptLoopPath))
    encodeSource := A_LoopFileFullPath
    encodeTarget := rootPath "\" cryptLoopPath
    setTextFile(loopPathAttrib "█" encodeSource "█" encodeTarget, renameEncodeFile)
  }
  if (copySourceFile(renameEncodeFile))
    removeSourceFile(renameEncodeFile)
  removeFile(renameEncodeFile)
  playBeep(1, 7902, 80) ;SUCCEED
}

renameDecodeBase64() {
  FileSelectFolder, selectedPath, *C:\Users\%A_UserName%\Downloads\gzmo5ajq
  if ErrorLevel
    return
  renameEncodeFile := selectedPath "\" "renameCrypt.enc"
  renameDecodeFile := selectedPath "\" "renameCrypt.dec"
  removeFile(renameEncodeFile)
  removeFile(renameDecodeFile)
  rootPath := selectedPath
  loop Files, %selectedPath%\*.*, DFR
  {
    SplitPath, A_LoopFileFullPath, , outDir, outExt, outNameNoExt
    loopPath := StrReplace(A_LoopFileFullPath, selectedPath "\")
    cryptLoopPath := ""
    loopPathAttrib = ""
    oPathString := StrSplit(loopPath, "\")
    if (InStr(A_LoopFileAttrib, "D")) {
      loopPathAttrib := "D"
      loop, % oPathString.MaxIndex() {
        cryptLoopPath := cryptLoopPath "\" decodeString(oPathString[A_Index])
      }
    } else {
      loopPathAttrib := "F"
      loop, % oPathString.MaxIndex() {
        if (A_Index = oPathString.MaxIndex()) {
          cryptLoopPath := cryptLoopPath "\" decodeString(outNameNoExt) "." outExt
        } else {
          cryptLoopPath := cryptLoopPath "\" decodeString(oPathString[A_Index])
        }
      }
    }
    cryptLoopPath := SubStr(cryptLoopPath, 2, StrLen(cryptLoopPath))
    decodeSource := A_LoopFileFullPath
    decodeTarget := rootPath "\" cryptLoopPath
    setTextFile(loopPathAttrib "█" decodeSource "█" decodeTarget, renameDecodeFile)

  }
  if (copySourceFile(renameDecodeFile))
    removeSourceFile(renameDecodeFile)
  removeFile(renameDecodeFile)
  playBeep(1, 7902, 80) ;SUCCEED
}

copySourceFile(renameCryptFile) {
  copyErrorLevel := 0
  FileRead, cryptString, %renameCryptFile%
  if ErrorLevel
    return false
  oCryptString := StrSplit(cryptString, "`r`n")
  loop, % oCryptString.MaxIndex() {
    if (InStr(oCryptString[A_Index], "█")) {
      cryptType := StrSplit(oCryptString[A_Index], "█")[1]
      cryptSource := StrSplit(oCryptString[A_Index], "█")[2]
      cryptTarget := StrSplit(oCryptString[A_Index], "█")[3]
      message := ""
      if (cryptType = "D") {
        FileCreateDir, %cryptTarget%
        copyErrorLevel := ErrorLevel
        message := "FileCreateDir() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " cryptSource " -> " cryptTarget
      } else if (cryptType = "F") {
        FileCopy, %cryptSource%, %cryptTarget%, 1
        copyErrorLevel := ErrorLevel
        message := "FileCopy() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " cryptSource " -> " cryptTarget
      }
      writeLogFile("INFO", A_ThisFunc, A_LineNumber, message)
      showToolTip("INFO", message)
      if copyErrorLevel {
        MsgBox, %cryptSource%
        return false
      }
    }
  }
  return true
}

removeSourceFile(renameCryptFile) {
  FileRead, cryptString, %renameCryptFile%
  if ErrorLevel
    return false
  oCryptString := StrSplit(cryptString, "`r`n")
  loop, % oCryptString.MaxIndex() {
    if (InStr(oCryptString[A_Index], "█")) {
      cryptType := StrSplit(oCryptString[A_Index], "█")[1]
      cryptSource := StrSplit(oCryptString[A_Index], "█")[2]
      message := ""
      if (cryptType = "D") {
        FileRemoveDir , %cryptSource%, 1
        message := "FileRemoveDir() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " cryptSource
      } else if (cryptType = "F") {
        FileDelete, %cryptSource%
        message := "FileDelete() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " cryptSource
      }
      writeLogFile("INFO", A_ThisFunc, A_LineNumber, message)
      showToolTip("INFO", message)
    }
  }
}

;XXX: Remove CRLF and use CRYPT_STRING_HEXRAW Crypt / Case of Base64, file name contains special characters that cannot be used
encodeString(str, formatName := "CRYPT_STRING_HEXRAW", encoding := "UTF-8", NOCRLF := true) {
  chars := StrPut(str, encoding) - 1
  VarSetCapacity(buff, size := chars << (encoding = "utf-16" || encoding = "cp1200"), 0)
  StrPut(str, &buff, chars, encoding)
  return cryptBinaryToString(&buff, size, formatName, NOCRLF)
}

decodeString(encodedStr, formatName := "CRYPT_STRING_HEXRAW", encoding := "UTF-8") {
  size := cryptStringToBinary(encodedStr, data, formatName)
  return StrGet(&data, size, encoding)
}

cryptBinaryToString(pData, size, formatName := "CRYPT_STRING_HEXRAW", NOCRLF := true) {
  static formats := { CRYPT_STRING_BASE64: 0x1, CRYPT_STRING_HEX: 0x4, CRYPT_STRING_HEXRAW: 0xC }, CRYPT_STRING_NOCRLF := 0x40000000
  fmt := formats[formatName] | (NOCRLF ? CRYPT_STRING_NOCRLF : 0)
  if !DllCall("Crypt32\CryptBinaryToString", "Ptr", pData, "UInt", size, "UInt", fmt, "Ptr", 0, "UIntP", chars)
    throw "CryptBinaryToString failed. LastError: " . A_LastError
  VarSetCapacity(outData, chars << !!A_IsUnicode)
  DllCall("Crypt32\CryptBinaryToString", "Ptr", pData, "UInt", size, "UInt", fmt, "Str", outData, "UIntP", chars)
  return outData
}

cryptStringToBinary(string, ByRef outData, formatName := "CRYPT_STRING_HEXRAW")
{
  static formats := { CRYPT_STRING_BASE64: 0x1, CRYPT_STRING_HEX: 0x4, CRYPT_STRING_HEXRAW: 0xC }
  fmt := formats[formatName]
  chars := StrLen(string)
  if !DllCall("Crypt32\CryptStringToBinary", "Ptr", &string, "UInt", chars, "UInt", fmt, "Ptr", 0, "UIntP", bytes, "UIntP", 0, "UIntP", 0)
    throw "CryptStringToBinary failed. LastError: " . A_LastError
   VarSetCapacity(outData, bytes)
   DllCall("Crypt32\CryptStringToBinary", "Ptr", &string, "UInt", chars, "UInt", fmt, "Str", outData, "UIntP", bytes, "UIntP", 0, "UIntP", 0)
   return bytes
}
```

### Dependencies
| Libraries                                                                           | Remrks            |
|-------------------------------------------------------------------------------------|-------------------|
| [Base64.ahk](https://github.com/cocobelgica/AutoHotkey-Util/blob/master/Base64.ahk) | base64 라이브러리 |

[^1]: https://github.com/dntco43u/rename_helper/blob/main/lib/Common.ahk
[^2]: https://github.com/dntco43u/rename_helper/blob/main/rename_helper.ahk
