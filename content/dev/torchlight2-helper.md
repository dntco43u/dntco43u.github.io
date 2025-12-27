---
date: 2025-12-15
title: torchlight2-helper
description: Torchlight II v1.25.5.6 매크로
tags:
  - autohotkey
categories:
  - dev
---

![image](/images/ahk-logo.webp#center)

## GitHub repository
[바로 가기](https://github.com/dntco43u/torchlight2_helper)

## autohotkey

### Shortcuts
| Shortcuts        | Remarks                                   |
|------------------|-------------------------------------------|
| `Ctrl+Shift+F1`  | Chat /bow                                 |
| `Ctrl+Shift+F2`  | Chat /salute                              |
| `Ctrl+Shift+F3`  | Chat /dance                               |
| `Ctrl+Shift+F4`  | Chat /drums                               |
| `Ctrl+Shift+F5`  | Chat /lute                                |
| `Ctrl+Shift+F6`  | Chat /laugh                               |
| `Ctrl+Shift+F7`  | Chat /taunt                               |
| `Ctrl+Shift+F8`  | Chat /gunshow                             |
| `Ctrl+Shift+~`   | Identify all                              |
| `Ctrl+Shift+1`   | Move item from stash inventory            |
| `Ctrl+Shift+2`   | Transmute item                            |
| `Ctrl+Shift+3`   | Restore shared stash                      |
| `Ctrl+Shift+4`   | Add enchant                               |
| `Ctrl+Shift+5`   | Disenchant and add enchant                |
| `Ctrl+Shift+6`   | Inventory loop disenchant and add enchant |
| `Ctrl+Shift+7`   | Inventory loop add enchant                |
| `Ctrl+Shift+8`   | Transmute scroll                          |
| `Ctrl+Shift+Tab` | Reroll world                              |
| `Ctrl+Shift+Q`   | Backup shared stash                       |
| `Ctrl+Shift+W`   | Backup save file                          |
| `Ctrl+Shift+E`   | Gamble                                    |
| `Ctrl+Shift+T`   | Tarroch's tomb golem soul                 |
| `Ctrl+Shift+Y`   | Restart world                             |
| `Ctrl+Shift+A`   | Auto spam toggle                          |
| `Ctrl+Shift+S`   | Custom console                            |
| `Ctrl+Shift+D`   | Buy item                                  |
| `Ctrl+Shift+F`   | Fish                                      |
| `Ctrl+Shift+G`   | Sell item                                 |
| `Ctrl+Shift+H`   | Hunt                                      |
| `Ctrl+Shift+J`   | Custom console                            |
| `Ctrl+Shift+L`   | Login game                                |
| `Ctrl+Shift+N`   | Copy item                                 |

### torchlight2_helper.properties
```ini
LOGIN_PASSWORD=W***************************************************************
```

### 구성
따로 분리한 구성 없이 소스에 하드코딩됨. 적용할 구성 외에 주석 처리할 것
- 자동 인챈트 구성
```ahk
;Configuration examples
filterStatObj := [["+", 76]]  ;lv38 (38*0.69)28x3=84, ceil(84*0.90)75.6
filterStatObj := [["+", 111]] ;lv58 (58*0.69)41x3=123, ceil(123*0.90)110.7
filterStatObj := [["+", 152]] ;lv81 (81*0.69)56x3=168, ceil(168*0.90)151.2
filterStatObj := [["+", 184]] ;lv99 (99*0.69)68*3=204, ceil(204*0.90)183.6
filterStatObj := [["+", 187]] ;lv100 (100*0.69)69*3=207, ceil(207*0.90)186.3
filterStatObj := [["+", 195]] ;lv105 (105*0.69)72*3=216, ceil(216*0.90)194.4
```

- 자동 스팸 구성
```ahk
;Configuration examples
Filter examples<br>
isAutoHealthPotion := true
isAutoManaPotion := true
isAutoPetPotion := false
isAutoSkill := true
isAutoAttack := true
isDervish6 := true
isDervish5 := true
isDervish4 := true
lowHealth90Coord := [38, 522] ;90%
lowMana90Coord := [754, 522] ;90%
lowHealth66Coord := [38, 543] ;66%
lowMana66Coord := [754, 543] ;66%
lowHealth33Coord := [38, 568] ;33%
lowMana33Coord := [754, 568] ;33%
lowPetHealthCoord := [38, 10]
lowPetManaCoord := [38, 22]
skill1Coord := [251, 575]
skillXObj := []
activeskillCoord := [606, 582]
berserkerChargeCoord := [474, 508]
embermageChargeCoord := [467, 510]
engineerChargeCoord := [467, 510]
outlanderChargeCoord := [478, 516]
```

### torchlight2_helper.ahk [^1]
```ahk
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

;-------------------------------------------------------------------------------
; global variables
;-------------------------------------------------------------------------------

global LOG_LEVEL := 2 ;current log level
global TRACE := 1     ;log level
global DEBUG := 2     ;log level
global INFO := 3      ;log level
global WARN := 4      ;log level
global ERROR := 5     ;log level
global SEVERE := 6    ;log level

global KEYDOWN := 1  ;keyboard event
global KEYUP := 2    ;keyboard event
global KEYPRESS := 3 ;keyboard event
global KEYRAW := 4   ;keyboard event

global LOG_FILE := A_ScriptDir "\log\torchlight2_helper.log"                                ;log file
global POWERFUL_FILE := A_ScriptDir "\res\powerful.bmp"                                     ;powerful image file
global SAVE_SRC_DIRECTORY := A_MyDocuments "\My Games\Runic Games\Torchlight 2\save"        ;save source directory
global SAVE_BAK_DIRECTORY := A_ScriptDir "\save"                                            ;save backup directory
global MODSAVE_SRC_DIRECTORY := A_MyDocuments "\My Games\Runic Games\Torchlight 2\modsave"  ;save source directory
global MODSAVE_BAK_DIRECTORY := A_ScriptDir "\modsave"                                      ;save backup directory
global CAPTURE2TEXT_FILE := A_ScriptDir "\Capture2Text\Capture2Text.exe"                    ;capture2text exe file, faster than Capture2Text_CLI.exe
global DEFLAGGER_FILE := A_ScriptDir "\deflag\deflag.bat"                                   ;deflag bat file
global PROPERTIES_FILE := A_ScriptDir "\" SubStr(A_ScriptName, 1, -4) ".properties"         ;properties file
global LOGIN_PASSWORD := ""                                                                 ;login password

global X := 1      ;screen coordinate x
global Y := 2      ;screen coordinate y
global WIDTH := 3  ;screen coordinate width
global HEIGHT := 4 ;screen coordinate height
global KEY_1 := 1 ;key
global KEY_2 := 2 ;key
global KEY_3 := 3 ;key
global KEY_4 := 4 ;key
global KEY_Q := 5 ;key
global KEY_W := 6 ;key
global KEY_E := 7 ;key
global KEY_R := 8 ;key
global KEY_D := 9 ;key
global KEY_F := 10 ;key

global m_IsLoading := false
global m_LastHDrinkHealth := A_TickCount - 8000
global m_LastHDrinkMana := A_TickCount - 8000
global m_LastHDrinkPetHealth := A_TickCount - 8000
global m_LastHDrinkPetMana := A_TickCount - 8000
global m_LastDervish6 := A_TickCount - 20000
global m_LastDervish5 := A_TickCount - 18000
global m_LastDervish4 := A_TickCount - 16000

;-------------------------------------------------------------------------------
; init
;-------------------------------------------------------------------------------

init() ;init

/*
init
*/
init() {
  initHotkey("initapp")
  getProperties(getTextFile(PROPERTIES_FILE))
  runAsAdmin()
  removeLogFile(8192)
}

;-------------------------------------------------------------------------------
; test
;-------------------------------------------------------------------------------

test() {
  ;getPixelColor(292, 228)
  testPixelColorInputAuto()
  ;testPixelColorInventory()
  ;testIsDisenchant()
  ;testTextFromOcr()
  ;testPixelColorocr()
  ;deflagSharedStash(MODSAVE_SRC_DIRECTORY)
}

;-------------------------------------------------------------------------------
; hotkeys
;-------------------------------------------------------------------------------

^+f1:: ;ctrl+shift+f1 chat /bow
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /bow")
  inputChat("/bow")           ;your character takes a bow
  return
}

^+f2:: ;ctrl+shift+f2 chat /salute
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /salute")
  inputChat("/salute")        ;your character salutes
  return
}

^+f3:: ;ctrl+shift+f3 chat /dance
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /dance")
  inputChat("/dance")         ;your character does a little dance until you move or do something else
  return
}

^+f4:: ;ctrl+shift+f4 chat /drums
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /drums")
  inputChat("/drums")         ;your character plays some imaginary drums
  return
}

^+f5:: ;ctrl+shift+f5 chat /lute
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /lute")
  inputChat("/lute")          ;your character plays on an imaginary lute
  return
}

^+f6:: ;ctrl+shift+f6 chat /laugh
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /laugh")
  inputChat("/laugh")         ;your character laughs
  return
}

^+f7:: ;ctrl+shift+f7 chat /taunt
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /taunt")
  inputChat("/taunt")         ;your character does a taunt
  return
}

^+f8:: ;ctrl+shift+f8 chat /gunshow
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("chat /gunshow")
  inputChat("/gunshow")       ;your character shows his guns arms
  return
}

^+~:: ;ctrl+shift+~
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("console")
  inputConsole("Identifyall") ;identify all
  return
}

^+1:: ;ctrl+shift+1
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("move item from stash inventory")
  inputMoveItemFromStashInventory()
  return
}

^+2:: ;ctrl+shift+2
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("transmute item")
  inputTransmuteItem()
  return
}

^+3:: ;ctrl+shift+3
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("restore shared stash")
  inputQuitGame()
  copySharedStash(MODSAVE_BAK_DIRECTORY, MODSAVE_SRC_DIRECTORY)
  copySharedStash(SAVE_BAK_DIRECTORY, SAVE_SRC_DIRECTORY)
  deflagSharedStash(MODSAVE_SRC_DIRECTORY)
  deflagSharedStash(SAVE_SRC_DIRECTORY)
  inputCreateGame(false)
  return
}

^+4:: ;ctrl+shift+4
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("add enchant")
  inputAddEnchant()
  return
}

^+5:: ;ctrl+shift+5
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("disenchant and add enchant")
  inputDisenchant()
  inputAddEnchant()
  return
}

^+6:: ;ctrl+shift+6
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("inventory loop disenchant and add enchant")
  ;element type
  ;filterTypeObj := ["chance", "conveys"]      ;only element damage
  ;filterTypeObj := ["chance", "+"]            ;only conveys
  ;filterTypeObj := ["chance", "conveys && +"] ;only conveys, only element damage
  ;element filter
  ;filterStatObj := [["conveys", 95]] ;lv8
  ;filterStatObj := [["conveys", 235]] ;lv17
  ;filterStatObj := [["conveys", 445]] ;lv26
  ;filterStatObj := [["conveys", 850]] ;lv39
  ;filterStatObj := [["conveys", 900]] ;lv40
  ;filterStatObj := [["conveys", 1650]] ;lv54
  ;filterStatObj := [["+", 195]] ;lv55
  ;filterStatObj := [["conveys", 2000]] ;lv60
  ;filterStatObj := [["conveys", 2400]] ;lv65
  ;filterStatObj := [["conveys", 3350]] ;lv75
  ;filterStatObj := [["conveys", 3700]] ;lv79
  ;filterStatObj := [["conveys", 4300]] ;lv84
  ;filterStatObj := [["+", 306], ["conveys", 4800]] ;lv88
  ;filterStatObj := [["+", 325], ["conveys", 5500]] ;lv93
  ;filterStatObj := [["conveys", 5600]] ;lv94
  ;filterStatObj := [["+", 327]] ;lv95
  ;filterStatObj := [["conveys", 6000]] ;lv97
  ;filterStatObj := [["+", 345], ["conveys", 6350]] ;lv99
  ;filterStatObj := [["+", 347], ["conveys", 6500]] ;lv100
  ;filterStatObj := [["+", 365], ["conveys", 7350]] ;lv105
  ;stat type
  ;filterTypeObj := ["dex", "foc", "vit"]        ;only str
  ;filterTypeObj := ["str", "foc", "vit"]        ;only dex
  filterTypeObj := ["str", "dex", "vit"]        ;only foc
  ;filterTypeObj := ["str", "dex", "foc"]        ;only vit
  ;filterTypeObj := ["dex", "vit", "str && foc"] ;only str, only foc
  ;filterTypeObj := ["str", "dex", "vit && foc"] ;only foc, only vit
  ;filterTypeObj := ["foc", "vit"]               ;str, dex
  ;filterTypeObj := ["dex", "vit"]               ;str, foc
  ;filterTypeObj := ["dex", "foc"]               ;str, vit
  ;filterTypeObj := ["str", "vit"]               ;dex, foc
  ;filterTypeObj := ["str", "dex"]               ;foc, vit
  ;filterTypeObj := ["dex"]                      ;str, foc, vit
  ;stat filter
  ;filterStatObj := [["+", 76]]  ;lv38 (38*0.69)28x3=84, ceil(84*0.90)75.6
  ;filterStatObj := [["+", 111]] ;lv58 (58*0.69)41x3=123, ceil(123*0.90)110.7
  ;filterStatObj := [["+", 152]] ;lv81 (81*0.69)56x3=168, ceil(168*0.90)151.2
  ;filterStatObj := [["+", 184]] ;lv99 (99*0.69)68*3=204, ceil(204*0.90)183.6
  ;filterStatObj := [["+", 187]] ;lv100 (100*0.69)69*3=207, ceil(207*0.90)186.3
  ;filterStatObj := [["+", 195]] ;lv105 (105*0.69)72*3=216, ceil(216*0.90)194.4
  filterStatObj := [["+", 181]]
  ;filterStatObj := [["+", 207]]
  ;filterStatObj := [["+", 999]]
  inputInventoryLoopEnchant(filterTypeObj, filterStatObj)
  return
}

^+7:: ;ctrl+shift+7
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("inventory loop add enchant")
  inputInventoryLoopAddEnchant()
  return
}

^+8:: ;ctrl+shift+8
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("transmute scroll")
  inputTransmuteScroll()
  return
}

^+tab:: ;ctrl+shift+tab
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("reroll world")
  inputQuitGame()
  deflagSharedStash(MODSAVE_SRC_DIRECTORY)
  deflagSharedStash(SAVE_SRC_DIRECTORY)
  inputCreateGame(true)
  return
}

^+q:: ;ctrl+shift+q
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("backup shared stash")
  inputSaveGame()
  deflagSharedStash(MODSAVE_SRC_DIRECTORY)
  deflagSharedStash(SAVE_SRC_DIRECTORY)
  copySharedStash(MODSAVE_SRC_DIRECTORY, MODSAVE_BAK_DIRECTORY)
  copySharedStash(SAVE_SRC_DIRECTORY, SAVE_BAK_DIRECTORY)
  playBeep(1)
  return
}

^+w:: ;ctrl+shift+w
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("backup save file")
  inputSaveGame()
  copySaveFile(MODSAVE_SRC_DIRECTORY, MODSAVE_BAK_DIRECTORY)
  copySaveFile(SAVE_SRC_DIRECTORY, SAVE_BAK_DIRECTORY)
  return
}

^+e:: ;ctrl+shift+e
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("gamble")
  inputGamble()
  return
}

^+t:: ;ctrl+shift+t
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("tarroch's tomb golem soul")
  inputConsole("item golem soul, 3") ;golem soul
  inputGetItem()
  return
}

^+y:: ;ctrl+shift+y
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("restart world")
  inputQuitGame()
  deflagSharedStash(MODSAVE_SRC_DIRECTORY)
  deflagSharedStash(SAVE_SRC_DIRECTORY)
  inputCreateGame(false)
  return
}

^+a:: ;ctrl+shift+a auto spam toggle
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("auto spam toggle " (m_IsAutoInput ? "off" : "on"))
  m_IsAutoInput := !m_IsAutoInput
  if m_IsAutoInput {
    m_IsLoading := true
    SetTimer, inputAutoTimer, 1
  } else {
    SetTimer, inputAutoTimer, Off
  }
  return
}

^+s:: ;ctrl+shift+s console
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("console")
  ;potion
  ;inputConsole("item rejuv potion 8, 1") ;ultimate rejuvenation potion lv90
  ;fish
  ;inputConsole("item fish_batfish, 1") ;warsnout
  ;inputConsole("item fish_batfish_perm, 1") ;giant warsnout
  ;spell
  ;inputConsole("item haste vi, 1") ;haste lv83
  ;inputConsole("item dervish vi, 1") ;dervish
  ;inputConsole("item dervish v, 1") ;dervish
  ;inputConsole("item dervish iv, 1") ;dervish
  ;inputConsole("item summon skeleton vi, 1") ;summon skeleton
  ;inputConsole("item summon skeleton v, 1") ;summon skeleton
  ;inputConsole("item summon skeleton iv, 1") ;summon skeleton
  ;inputConsole("item summon skeleton iii, 1") ;summon skeleton
  ;inputConsole("item haste vi, 1") ;haste lv83
  ;inputConsole("item elemental overload vi, 1") ;elemental overload lv83
  ;inputConsole("item block vi, 1") ;block lv83
  ;inputConsole("item willpower vi, 1") ;willpower lv83
  ;inputConsole("item martial weapons expertise vi, 1") ;martial weapons expertise lv83
  ;inputConsole("item heal all vi, 1") ;heal all lv83
  ;inputConsole("item summon skeleton vi, 1") ;summon skeleton lv83
  ;inputConsole("item heal all vi, 1") ;heal all
  ;inputConsole("item heal all v, 1") ;heal all
  ;inputConsole("item heal all iv, 1") ;heal all
  ;inputConsole("item heal all iii, 1") ;heal all
  ;inputConsole("item heal all ii, 1") ;heal all
  ;inputConsole("item heal all i, 1") ;heal all
  ;inputConsole("item haste vi, 1") ;haste
  ;inputConsole("item haste v, 1") ;haste
  ;inputConsole("item haste iv, 1") ;haste
  ;inputConsole("item haste iii, 1") ;haste
  ;inputConsole("item haste ii, 1") ;haste
  ;inputConsole("item haste i, 1") ;haste
  ;inputConsole("item dervish vi, 1") ;dervish
  ;inputConsole("item dervish v, 1") ;dervish
  ;inputConsole("item dervish iv, 1") ;dervish
  ;inputConsole("item dervish iii, 1") ;dervish
  ;inputConsole("item dervish ii, 1") ;dervish
  ;inputConsole("item dervish i, 1") ;dervish
  ;inputConsole("item block vi, 1") ;block
  ;inputConsole("item block v, 1") ;block
  ;inputConsole("item block iv, 1") ;block
  ;inputConsole("item block iii, 1") ;block
  ;inputConsole("item block ii, 1") ;block
  ;inputConsole("item block i, 1") ;block
  ;inputConsole("item adventurer vi, 1") ;adventurer
  ;inputConsole("item adventurer v, 1") ;adventurer
  ;inputConsole("item adventurer iv, 1") ;adventurer
  ;inputConsole("item adventurer iii, 1") ;adventurer
  ;inputConsole("item adventurer ii, 1") ;adventurer
  ;inputConsole("item adventurer i, 1") ;adventurer
  ;inputConsole("item advanced spellcasting vi, 1") ;concentration
  ;inputConsole("item advanced spellcasting v, 1") ;concentration
  ;inputConsole("item advanced spellcasting iv, 1") ;concentration
  ;inputConsole("item advanced spellcasting iii, 1") ;concentration
  ;inputConsole("item advanced spellcasting ii, 1") ;concentration
  ;inputConsole("item advanced spellcasting i, 1") ;concentration
  ;inputConsole("item summon blood zombie, 1") ;summon blood zombie
  ;inputConsole("item summon skeleton i, 1") ;summon skeleton
  ;inputConsole("item summon archers i, 1") ;summon archers
  ;inputConsole("item summon skeleton ii, 1") ;summon skeleton
  ;inputConsole("item summon skeleton i, 1") ;summon skeleton
  ;normal, magic weapons
  ;inputConsole("item staff_n00, 1") ;inert staff lv1
  ;inputConsole("item fist_n00, 1") ;berserker claws lv1
  ;inputConsole("item greathammer_n00, 1") ;railman wrench lv1
  ;inputConsole("item pistol_n00, 1") ;initiate matchlock lv1
  ;inputConsole("item rifle_n00b, 1") ;rookie shotgonne lv5
  ;rare
  ;inputConsole("item pistol_m01c, 32") ;poacher's gun lv9
  ;inputConsole("item wand_m01c, 32") ;witch wand lv9
  ;inputConsole("item zzz_sword_grom, 1") ;grom's fireblade lv38
  ;inputConsole("item zzz_sword_mc_iron, 8") ;iron sword lv42
  ;inputConsole("item zzz_sword_mc_gold, 1") ;gold sword lv42
  ;inputConsole("item wand_m04c, 32") ;steel brand lv40
  ;inputConsole("item axe_u05x, 1") ;the axe of throwing lv40
  ;inputConsole("item cannon_m05b, 32") ;prototype falconet lv47
  ;inputConsole("item wand_m07slots, 1") ;silverisle dracoduceus lv79
  ;inputConsole("item fist_m08, 32") ;kideny dagger lv86
  ;inputConsole("item fist_m08c, 32") ;dragon talon lv95
  ;inputConsole("item wand_m08slots, 1") ;bedima augur wand lv94
  ;inputConsole("item wand_m08c, 32") ;archmage scepter lv96
  ;inputConsole("item cannon_m08c, 32") ;dwarven rune howitzer lv95
  ;inputConsole("item pistol_m08c, 32") ;over-under revolver lv95
  ;rare weapon
  ;inputConsole("item staff_m01slots, 1") ;striking dragon staff lv8
  ;inputConsole("item staff_m02slots, 1") ;ember-studded staff lv17
  ;inputConsole("item staff_m03slots, 1") ;arbiter staff lv26
  ;inputConsole("item staff_m04slots, 1") ;hexing staff lv38
  ;inputConsole("item rifle_m01slots, 1") ;ember-sights shotgonne lv9
  ;inputConsole("item rifle_m02slots, 1") ;four-gauge heavy rifle lv18
  ;inputConsole("item rifle_m03slots, 1") ;imperial trench-rifle lv27
  ;inputConsole("item rifle_m04slots, 1") ;air-cooled battle rifle lv39
  ;inputConsole("item wand_m01slots, 1") ;gem-studded wand lv9
  ;inputConsole("item wand_m02slots, 1") ;shalestone rod lv18
  ;inputConsole("item wand_m03slots, 1") ;dragonclaw rod lv27
  ;inputConsole("item wand_m04slots, 1") ;ember crook lv39
  ;inputConsole("item cannon_m01slots, 1") ;baroque culverin lv8
  ;inputConsole("item cannon_m02slots, 1") ;brass-bound culverin lv17
  ;inputConsole("item cannon_m03slots, 1") ;imperial autocannon lv26
  ;inputConsole("item cannon_m04slots, 1") ;imperial battle-cannon lv38
  ;unique
  ;inputConsole("item berserker_05_pants_alt_set, 1") ;savage pants lv75, appearance different from set
  ;inputConsole("item berserker_06_boots_alt_set, 1") ;harbinger boots lv99, appearance different from set
  ;inputConsole("item caster_03_boots_alt_d, 1") ;zidders' boots of the roo lv46, limoany cannot be installed, appearance different from set
  ;inputConsole("item netherim_02_shoulders_alt_set, 1") ;incubus shoulders lv60, can't get knockback resistance, appearance different from set
  ;inputConsole("item shield_u02c, 1") ;roundhaven lv23
  ;inputConsole("item bone_01_ring_f_alt_b, 1") ;the howing flame lv26
  ;inputConsole("item zzz_immobresist, 1") ;remobilizer lv23
  ;inputConsole("item shield_u03c, 1") ;crimson guard lv38
  ;inputConsole("item z_amulet_axler, 1") ;axler's commendation lv46
  ;inputConsole("item shield_u05, 1") ;the centerwing lv58
  ;inputConsole("item hammer_u05, 1") ;hammer of the ecclesiats lv61
  ;inputConsole("item cannon_u05, 1") pot-de-fer lv60
  ;inputConsole("item cannon_u06, 1") the kingmaker lv75
  ;inputConsole("item z_shield_u07_set, 1") ;the critical factor lv75
  ;inputConsole("item clockwork_01_amulet_i_alt_b, 1") ;crackledoom lv81
  ;inputConsole("item clockwork_01_ring_p_alt_b, 1") ;turquet band lv81
  ;inputConsole("item bow_u07b, 1") ;soulpiercer lv97
  ;inputConsole("item shield_u07c, 1") ;parma's coal-burner lv98
  ;inputConsole("item caster_06_amulet_alt_set, 1") ;trancendent icon lv99
  ;inputConsole("item hammer_u07c, 1") ;hammer of retribution lv100
  ;inputConsole("item wand_u07c, 1") ;galaxy quest lv100
  ;inputConsole("item polearm_u07c, 1") ;spear-bot 2300 lv100
  ;inputConsole("item legendary2_wand06c, 1") ;netherrealm wand lv105
  ;inputConsole("item legendary2_pistol06c, 1") ;netherrealm pistol lv105
  ;inputConsole("item legendary2_hammer05c, 1") ;netherrealm hammer lv105
  ;inputConsole("item legendary2_axe05c, 1") ;netherrealm axe lv105
  ;inputConsole("item legendary2_cannon04c, 1") ;netherrealm cannon lv105
  ;inputConsole("item legendary2_staff06c, 1") ;netherrealm staff lv105
  ;inputConsole("item legendary2_rifle04c, 1") ;netherrealm shotgonne lv105
  ;inputConsole("item legendary2_fist06c, 1") ;netherrealm claw lv105
  ;inputConsole("item legendary2_hammer05c, 1") ;netherrealm hammer lv105
  ;inputConsole("item legendary2_shield05c, 1") ;netherrealm shield lv105
  ;legendary
  ;inputConsole("item legendary_staff01, 1") ;heart of glass lv54
  ;inputConsole("item legendary_pistol02, 1") ;ator the fighting eagle lv65
  ;inputConsole("item legendary_wand02, 1") ;the black glove lv65
  ;inputConsole("item legendary_greataxe02, 1") ;all the colors of the dark lv79
  ;inputConsole("item legendary_polearm02, 1") ;battle royale lv79
  ;inputConsole("item legendary_sword03, 1") ;a taste of blood lv84
  ;inputConsole("item legendary_bow03, 1") ;tenebre 84
  ;inputConsole("item legendary_wand04, 1") ;four sided triangle lv88
  ;inputConsole("item legendary_fist04, 1") ;the beast with a million eyes lv88
  ;inputConsole("item legendary_crossbow02, 1") ;bittermoon lv94
  ;inputConsole("item legendary_shield04, 1") ;old master q lv99
  ;inputConsole("item legendary_cannon03, 1") ;oldboy lv99
  ;inputConsole("item legendary_staff05, 1") ;invincible lv99
  ;inputConsole("item legendary_hammer04, 1") ;quaertermass and the pit lv99
  ;inputConsole("item legendary_greatsword03, 1") ;twitch of the death nerve lv99
  ;inputConsole("item legendary2_shield05, 1") ;escutcheon lv105
  ;inputConsole("item legendary2_shield05b, 1") ;decisive sanctuary lv105
  ;inputConsole("item legendary2_axe05, 1") ;eight-armed axe lv105
  ;inputConsole("item legendary2_axe05b, 1") ;axe of the elder one lv105
  ;inputConsole("item legendary2_fist06, 1") ;clockmaster's tooth lv105
  ;inputConsole("item legendary2_fist06b, 1") ;bonepry wedge lv105
  ;inputConsole("item legendary2_hammer05, 1") ;arcgap's vice lv105
  ;inputConsole("item legendary2_hammer05b, 1") ;smithereen lv105
  ;inputConsole("item legendary2_pistol06, 1") ;blastmaw's trigger lv105
  ;inputConsole("item legendary2_pistol06b, 1") ;dispatch lv105
  ;inputConsole("item legendary2_sword05, 1") ;cerulean nightmare lv105
  ;inputConsole("item legendary2_sword05b, 1") ;shards of cobalt lv105
  ;inputConsole("item legendary2_wand06, 1") ;the prophecy lv105
  ;inputConsole("item legendary2_wand06b, 1") ;magus scream lv105
  ;inputConsole("item legendary2_bow05, 1") ;the serpent lv105
  ;inputConsole("item legendary2_bow05b, 1") ;dragonskull's balance lv105
  ;inputConsole("item legendary2_cannon04, 1") ;riftplane destroyer lv105
  ;inputConsole("item legendary2_cannon04b, 1") ;starfall's command lv105
  ;inputConsole("item legendary2_crossbow03, 1") ;the mandlebow lv105
  ;inputConsole("item legendary2_crossbow03b, 1") ;skullfire fangbow lv105
  ;inputConsole("item legendary2_greataxe04, 1") ;foe's collapse lv105
  ;inputConsole("item legendary2_greataxe04b, 1") ;ribcleave lv105
  ;inputConsole("item legendary2_greathammer04, 1") ;quientus lv105
  ;inputConsole("item legendary2_greathammer04b, 1") ;the final embrace lv105
  ;inputConsole("item legendary2_polearm04, 1") ;impulse lv105
  ;inputConsole("item legendary2_polearm04b, 1") ;bio-agitator lv105
  ;inputConsole("item legendary2_rifle04, 1") ;tri-force catalyst lv105
  ;inputConsole("item legendary2_rifle04b, 1") ;bloom`s margin lv105
  ;inputConsole("item legendary2_staff06, 1") ;dragoncoil lv105
  ;inputConsole("item legendary2_staff06b, 1") ;ophidian`s endgame lv105
  ;inputConsole("item legendary2_greatsword04, 1") ;glyphforge lv105
  ;inputConsole("item legendary2_greatsword04b, 1") ;mortal edge lv105
  ;gem
  ;inputConsole("item tl2_venomember_rank1, 4") ;venom ember speck lv8
  ;inputConsole("item tl2_voidember_rank1, 4") ;void ember speck lv8
  ;inputConsole("item tl2_voidember_rank2, 4") ;void ember chip lv22
  ;inputConsole("item tl2_bloodember_rank1, 32") ;blood ember speck lv8
  ;inputConsole("item tl2_bloodember_rank2, 32") ;blood ember chip lv22
  ;inputConsole("item tl2_bloodember_rank3, 32") ;blood ember shard lv36
  ;inputConsole("item tl2_bloodember_rank4, 32") ;blood ember lv50
  ;inputConsole("item tl2_bloodember_rank7, 32") ;giant blood ember lv92
  ;inputConsole("item tl2_chaosember_rank1, 32") ;chaos ember speck lv8
  ;inputConsole("item tl2_chaosember_rank2, 32") ;chaos ember chip lv22
  ;inputConsole("item tl2_chaosember_rank3, 32") ;chaos ember shard lv36
  ;inputConsole("item tl2_chaosember_rank4, 32") ;chaos ember lv50
  ;inputConsole("item tl2_chaosember_rank5, 32") ;giant chaos ember lv64
  ;inputConsole("item tl2_chaosember_rank7, 32") ;giant chaos ember lv92
  ;inputConsole("item tl2_ironember_rank7, 32") ;giant iron ember lv92
  ;inputConsole("item tl2_eyeofwinterwidow, 4") ;the eye of winter widow lv11
  ;inputConsole("item tl2_eyeofstennbrun, 4") ;the eye of the stennbrun lv13
  ;inputConsole("item tl2_eyeofgrell, 4") ;eye of grell lv12
  ;inputConsole("item tl2_eyeofthewerewolf, 4") ;the eye of the grizzled alpha lv39
  ;inputConsole("item tl2_eyeofsister_aleera, 32") ;eye of aleera lv47
  ;inputConsole("item tl2_eyeofdarkalchemist, 4") ;the eye of the dark alchemist lv48
  ;inputConsole("item tl2_eyeofnetherlord, 4") ;the eye of the netherlord lv50
  ;inputConsole("item tl2_skull011, 4") ;skull of festuur lv27
  ;inputConsole("item tl2_skull021, 4") ;skull of karandir lv52
  ;inputConsole("item tl2_skull023, 4") ;skull of limoany lv55
  ;inputConsole("item tl2_skull024, 4") ;lovantine skull lv57
  ;inputConsole("item tl2_skull025, 4") ;skull of macaburg lv58
  ;inputConsole("item tl2_skull026, 4") ;meerko skull lv60
  ;inputConsole("item tl2_skull034, 4") ;qzmoda skull lv72
  ;inputConsole("item tl2_skull035, 4") ;skull of reichliu lv73
  ;inputConsole("item tl2_skull036, 4") ;rambren skull lv75
  ;inputConsole("item tl2_skull040, 4") ;tibbeek skull lv81
  ;inputConsole("item tl2_skull043, 4") ;skull of vastok lv85
  ;inputConsole("item tl2_skull044, 4") ;vellinique skull lv87
  ;inputConsole("item tl2_skull045, 4") ;skull of whorlbarb lv88
  ;inputConsole("item tl2_skull046, 4") ;wfuntir skull lv90
  ;inputConsole("item tl2_skull047, 4") ;skull of x'n!troph lv91
  ;inputConsole("item tl2_skull051, 4") ;skull of zardon lv97
  ;inputConsole("item tl2_skull052, 4") ;zardon's mighty skull lv99
  ;pet
  ;inputConsole("item petcollar_u13, 32") ;epic collar lv99
  ;inputConsole("item pettag_u13, 32") ;epic stud lv98
  ;inputConsole("item petcollar_u01, 32") ;epic collar lv4
  ;inputConsole("item pettag_u01, 32") ;epic stud lv3
  ;set valkyrie
  ;inputConsole("item light_g_chest_alt_set, 32") ;valkyrie armor lv96
  ;inputConsole("item light_g_belt_alt_set, 32") ;valkyrie belt lv96
  ;inputConsole("item light_g_boots_alt_set, 32") ;valkyrie boots lv96
  ;inputConsole("item light_g_pants_alt_set, 32") ;valkyrie pants lv96
  ;inputConsole("item light_g_ring_alt_set, 32") ;valkyrie ring lv96
  ;inputConsole("item light_g_shoulders_alt_set, 32") ;valkyrie shoulders lv96
  ;inputConsole("item light_g_amulet_alt_set, 32") ;valkyrie amulet lv96
  ;inputConsole("item light_g_gloves_alt_set, 32") ;valkyrie gloves lv96
  ;inputConsole("item light_g_helmet_alt_set, 32") ;valkyrie helmet lv96
  ;set dragonrift
  ;inputConsole("item medium_g_chest_alt_set, 1") ;dragonrift armor lv94
  ;inputConsole("item medium_g_belt_alt_set, 1") ;dragonrift belt lv94
  ;inputConsole("item medium_g_boots_alt_set, 1") ;dragonrift boots lv94
  ;inputConsole("item medium_g_pants_alt_set, 1") ;dragonrift pants lv94
  ;inputConsole("item medium_g_ring_alt_set, 1") ;dragonrift ring lv94
  ;inputConsole("item medium_g_shoulders_alt_set, 1") ;dragonrift shoulders lv94
  ;inputConsole("item medium_g_amulet_alt_set, 1") ;dragonrift amulet lv94
  ;inputConsole("item medium_g_gloves_alt_set, 1") ;dragonrift gloves lv94
  ;inputConsole("item medium_g_helmet_alt_set, 1") ;dragonrift helmet lv94
  ;set inquisitor
  ;inputConsole("item cloth_g_chest_alt_set, 32") ;inquisitor armor lv92
  ;inputConsole("item cloth_g_belt_alt_set, 32") ;inquisitor belt lv92
  ;inputConsole("item cloth_g_boots_alt_set, 32") ;inquisitor boots lv92
  ;inputConsole("item cloth_g_pants_alt_set, 32") ;inquisitor pants lv92
  ;inputConsole("item cloth_g_ring_alt_set, 32") ;inquisitor ring lv92
  ;inputConsole("item cloth_g_shoulders_alt_set, 32") ;inquisitor shoulders lv92
  ;inputConsole("item cloth_g_amulet_alt_set, 32") ;inquisitor amulet lv92
  ;inputConsole("item cloth_g_gloves_alt_set, 32") ;inquisitor gloves lv92
  ;inputConsole("item cloth_g_helmet_alt_set, 32") ;inquisitor helmet lv92
  ;set aristocrat (no amulet)
  ;inputConsole("item pimp_01_chest_alt_set, 1") ;aristocrat armor lv91
  ;inputConsole("item pimp_01_belt_alt_set, 32") ;aristocrat belt lv91
  ;inputConsole("item pimp_01_boots_alt_set, 1") ;aristocrat boots lv91
  ;inputConsole("item pimp_01_pants_alt_set, 1") ;aristocrat pants lv91
  ;inputConsole("item pimp_01_ring_alt_set, 32") ;aristocrat ring lv91
  ;inputConsole("item pimp_01_shoulders_alt_set, 1") ;aristocrat shoulders lv91
  ;inputConsole("item pimp_01_gloves_alt_set, 1") ;aristocrat gloves lv91
  ;inputConsole("item pimp_01_helmet_alt_set, 1") ;aristocrat helmet lv91
  ;set sentinel
  ;inputConsole("item vanquisher_01_chest_alt_set, 32") ;sentinel armor lv58
  ;inputConsole("item vanquisher_01_belt_alt_set, 32") ;sentinel belt lv58
  ;inputConsole("item vanquisher_01_boots_alt_set, 32") ;sentinel boots lv58
  ;inputConsole("item vanquisher_01_pants_alt_set, 32") ;sentinel pants lv58
  ;inputConsole("item vanquisher_01_ring_alt_set, 32") ;sentinel ring lv58
  ;inputConsole("item vanquisher_01_shoulders_alt_set, 32") ;sentinel shoulders lv58
  ;inputConsole("item vanquisher_01_amulet_alt_set, 32") ;sentinel amulet lv58
  ;inputConsole("item vanquisher_01_gloves_alt_set, 32") ;sentinel gloves lv58
  ;inputConsole("item vanquisher_01_helmet_alt_set, 32") ;sentinel helmet lv58
  ;set exorcist
  ;inputConsole("item varkolyn_01_chest_alt_set, 32") ;exorcist armor lv44
  ;inputConsole("item varkolyn_01_belt_alt_set, 32") ;exorcist belt lv44
  ;inputConsole("item varkolyn_01_boots_alt_set, 32") ;exorcist boots lv44
  ;inputConsole("item varkolyn_01_pants_alt_set, 32") ;exorcist pants lv44
  ;inputConsole("item varkolyn_01_ring_alt_set, 32") ;exorcist ring lv44
  ;inputConsole("item varkolyn_01_shoulders_alt_set, 32") ;exorcist shoulders lv44
  ;inputConsole("item varkolyn_01_amulet_alt_set, 32") ;exorcist amulet lv44
  ;inputConsole("item varkolyn_01_gloves_alt_set, 32") ;exorcist gloves lv44
  ;inputConsole("item varkolyn_01_helmet_alt_set, 32") ;exorcist helmet lv44
  ;set ironlord
  ;inputConsole("item stone_01_amulet_alt_set, 32") ;granite amulet lv37
  ;inputConsole("item stone_01_ring_alt_set, 32") ;granite ring lv37
  ;inputConsole("item stone_01_belt_alt_set, 32") ;granite belt lv37
  ;set janissary
  ;inputConsole("item dezrohir_02_belt_alt_set, 32") ;janissary belt lv34
  ;inputConsole("item dezrohir_02_amulet_alt_set, 32") ;janissary amulet lv34
  ;inputConsole("item dezrohir_02_ring_alt_set, 32") ;janissary ring lv34
  ;set ezrohir
  ;inputConsole("item dezrohir_01_chest_alt_set, 32") ;ezrohir armor lv30
  ;inputConsole("item dezrohir_01_belt_alt_set, 32") ;ezrohir belt lv30
  ;inputConsole("item dezrohir_01_boots_alt_set, 32") ;ezrohir boots lv30
  ;inputConsole("item dezrohir_01_pants_alt_set, 32") ;ezrohir pants lv30
  ;inputConsole("item dezrohir_01_ring_alt_set, 32") ;ezrohir ring lv30
  ;inputConsole("item dezrohir_01_shoulders_alt_set, 32") ;ezrohir shoulders lv30
  ;inputConsole("item dezrohir_01_amulet_alt_set, 32") ;ezrohir amulet lv30
  ;inputConsole("item dezrohir_01_gloves_alt_set, 32") ;ezrohir gloves lv30
  ;inputConsole("item dezrohir_01_helmet_alt_set, 32") ;ezrohir helmet lv30
  ;set clovenhoof
  ;inputConsole("item furhide_01_chest_alt_set, 32") ;clovenhoof armor lv15
  ;inputConsole("item furhide_01_belt_alt_set, 32") ;clovenhoof belt lv15
  ;inputConsole("item furhide_01_boots_alt_set, 32") ;clovenhoof boots lv15
  ;inputConsole("item furhide_01_pants_alt_set, 32") ;clovenhoof pants lv15
  ;inputConsole("item furhide_01_ring_alt_set, 32") ;clovenhoof ring lv15
  ;inputConsole("item furhide_01_shoulders_alt_set, 32") ;clovenhoof shoulders lv15
  ;inputConsole("item furhide_01_amulet_alt_set, 32") ;clovenhoof amulet lv15
  ;inputConsole("item furhide_01_gloves_alt_set, 32") ;clovenhoof gloves lv15
  ;inputConsole("item furhide_01_helmet_alt_set, 32") ;clovenhoof helmet lv15
  ;set runemaster
  ;inputConsole("item sturm_01_chest_alt_set, 32") ;runemaster armor lv7
  ;inputConsole("item sturm_01_belt_alt_set, 32") ;runemaster belt lv7
  ;inputConsole("item sturm_01_boots_alt_set, 32") ;runemaster boots lv7
  ;inputConsole("item sturm_01_pants_alt_set, 32") ;runemaster pants lv7
  ;inputConsole("item sturm_01_ring_alt_set, 32") ;runemaster ring lv7
  ;inputConsole("item sturm_01_shoulders_alt_set, 32") ;runemaster shoulders lv7
  ;inputConsole("item sturm_01_amulet_alt_set, 32") ;runemaster amulet lv7
  ;inputConsole("item sturm_01_gloves_alt_set, 32") ;runemaster gloves lv7
  ;inputConsole("item sturm_01_helmet_alt_set, 32") ;runemaster helmet lv7
  ;inputConsole("item zzz_sturm_01_helmet_alt_b, 1") ;general grell's headpiece lv9
  ;set ghastly
  ;inputConsole("item zzz_ghastly_boots, 1") ;ghastly boots lv5
  ;inputConsole("item zzz_ghastly_shoulders, 1") ;ghastly shoulders lv5
  ;inputConsole("item zzz_ghastly_gloves, 1") ;ghastly gloves lv5
  ;set emberweave
  ;inputConsole("item caster_05_chest_alt_set, 1") ;emberweave armor lv70
  ;inputConsole("item caster_05_belt_alt_set, 1") ;emberweave belt lv70
  ;inputConsole("item caster_05_boots_alt_set, 1") ;emberweave boots lv70
  ;inputConsole("item caster_05_pants_alt_set, 1") ;emberweave pants lv70
  ;inputConsole("item caster_05_ring_alt_set, 2") ;emberweave ring lv75
  ;inputConsole("item caster_05_shoulders_alt_set, 1") ;emberweave shoulders lv70
  ;inputConsole("item caster_05_amulet_alt_set, 1") ;emberweave amulet lv70
  ;inputConsole("item caster_05_gloves_alt_set, 1") ;emberweave gloves lv70
  ;inputConsole("item caster_05_helmet_alt_set, 1") ;emberweave helmet lv70
  ;set trancendent
  ;inputConsole("item caster_06_chest_alt_set, 1") ;trancendent armor lv99
  ;inputConsole("item caster_06_belt_alt_set, 1") ;trancendent belt lv99
  ;inputConsole("item caster_06_boots_alt_set, 1") ;trancendent boots lv99
  ;inputConsole("item caster_06_pants_alt_set, 1") ;trancendent pants lv99
  ;inputConsole("item caster_06_ring_alt_set, 2") ;trancendent ring lv99
  ;inputConsole("item caster_06_shoulders_alt_set, 1") ;trancendent shoulders lv99
  ;inputConsole("item caster_06_amulet_alt_set, 1") ;trancendent amulet lv99
  ;inputConsole("item caster_06_gloves_alt_set, 1") ;trancendent gloves lv99
  ;inputConsole("item caster_06_helmet_alt_set, 1") ;trancendent helmet lv99
  ;set ascendant
  ;inputConsole("item caster_x07_chest, 1") ;ascendant armor lv105
  ;inputConsole("item caster_x07_belt, 1") ;ascendant belt lv105
  ;inputConsole("item caster_x07_boots, 1") ;ascendant boots lv105
  ;inputConsole("item caster_x07_pants, 1") ;ascendant pants lv105
  ;inputConsole("item caster_x07_shoulders, 1") ;ascendant shoulders lv105
  ;inputConsole("item caster_x07_gloves, 1") ;ascendant gloves lv105
  ;inputConsole("item caster_x07_helmet, 1") ;ascendant helmet lv105
  ;set draketalon
  ;inputConsole("item berserker_x07_chest, 1") ;draketalon armor lv105
  ;inputConsole("item berserker_x07_belt, 1") ;draketalon belt lv105
  ;inputConsole("item berserker_x07_boots, 1") ;draketalon boots lv105
  ;inputConsole("item berserker_x07_pants, 1") ;draketalon pants lv105
  ;inputConsole("item berserker_x07_shoulders, 1") ;draketalon shoulders lv105
  ;inputConsole("item berserker_x07_gloves, 1") ;draketalon gloves lv105
  ;inputConsole("item berserker_x07_helmet, 1") ;draketalon helmet lv105
  ;set outercore
  ;inputConsole("item engineer_x07_chest, 1") ;outercore armor lv105
  ;inputConsole("item engineer_x07_belt, 1") ;outercore belt lv105
  ;inputConsole("item engineer_x07_boots, 1") ;outercore boots lv105
  ;inputConsole("item engineer_x07_pants, 1") ;outercore pants lv105
  ;inputConsole("item engineer_x07_shoulders, 1") ;outercore shoulders lv105
  ;inputConsole("item engineer_x07_gloves, 1") ;outercore gloves lv105
  ;inputConsole("item engineer_x07_helmet, 1") ;outercore helmet
  ;set dominion
  ;inputConsole("item wanderer_x07_chest, 1") ;dominion armor lv105
  ;inputConsole("item wanderer_x07_belt, 1") ;dominion belt lv105
  ;inputConsole("item wanderer_x07_boots, 1") ;dominion boots lv105
  ;inputConsole("item wanderer_x07_pants, 1") ;dominion pants lv105
  ;inputConsole("item wanderer_x07_shoulders, 1") ;dominion shoulders lv105
  ;inputConsole("item wanderer_x07_gloves, 1") ;dominion gloves lv105
  ;inputConsole("item wanderer_x07_helmet, 1") ;dominion helmet lv105
  ;sturmbeorn
  ;inputConsole("item sturm_01_ring_p_alt_c, 32") ;sturmbeorn ring lv9
  ;inputConsole("item sturm_01_ring_f_alt_c, 32") ;sturmbeorn ring lv9
  ;inputConsole("item sturm_01_amulet_p_alt_c, 32") ;sturmbeorn amulet lv9
  ;inputConsole("item sturm_01_chest_alt_c, 32") ;sturmbeorn armor lv9
  ;inputConsole("item sturm_01_belt_alt_c, 32") ;sturmbeorn belt lv9
  ;inputConsole("item sturm_01_boots_alt_c, 32") ;sturmbeorn boots lv9
  ;inputConsole("item sturm_01_pants_alt_c, 32") ;sturmbeorn pants lv9
  ;inputConsole("item sturm_01_shoulders_alt_c, 32") ;sturmbeorn shoulders lv9
  ;inputConsole("item sturm_01_gloves_alt_c, 32") ;sturmbeorn gloves lv9
  ;inputConsole("item sturm_01_helmet_alt_c, 32") ;sturmbeorn helmet lv9
  inputGetItem()
  return
}

^+d:: ;ctrl+shift+a buy item
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("buy item")
  inputBuyItem(99)
  return
}

^+f:: ;ctrl+shift+f
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("fish")
  inputFish()
  return
}

^+g:: ;ctrl+shift+g
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("sell item")
  inputMoveItemFromPlayerInventory(true)
  return
}

^+h:: ;ctrl+shift+h
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("hunt")
  inputHunt()
  return
}

^+j:: ;ctrl+shift+j console
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("console")
  ;monster
  inputConsole("monster boss_spider_cave, 24")
  ;inputConsole("monster boss_werewolf, 1")
  ;inputConsole("monster boss_map_artificer, 1")
  ;inputConsole("monster boss_map_robot, 1")
  ;inputConsole("monster boss_map_wraithlord, 1")
  ;inputConsole("monster boss_map_dragon, 1")
  ;inputConsole("monster boss_map_spectraldragon, 1")
  ;inputConsole("monster boss_netherim_shadow, 1")
  Sleep, 10
  pressKey("f", KEYPRESS, 10) ;use best potion
  pressKey("Shift", KEYDOWN, 10)
  pressKeyCustom("q", KEYPRESS, 2000, 10) ;use skill
  pressKey("Shift", KEYUP, 10)
  return
}

^+l:: ;ctrl+shift+l
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  initHotkey("login game")
  inputLoginGame()
  inputShowInternetGame()
  return
}

^+z:: ;ctrl+shift+tz
{
  initHotkey("test")
  test()
  return
}

^+c:: ;ctrl+shift+c
{
  initHotkey("get pixel color")
  getPixelColor("", "")
  return
}

^+v:: ;ctrl+shift+v
{
  initHotkey("is image from coordinate")
  isImageFromCoordinate(0, 0, A_ScreenWidth, A_ScreenHeight, POWERFUL_FILE)
  return
}

^+b:: ;ctrl+shift+b
{
  initHotkey("get text from ocr")
  getTextFromOcr(0, 0, A_ScreenWidth, A_ScreenHeight)
  return
}

^+n:: ;ctrl+shift+n copy item
{
  if !WinActive("ahk_exe TL2StashNinja.exe")
    return
  initHotkey("copy item")
  inputBuyItem(39)
  Sleep, 2500
  if WinExist("ahk_exe Torchlight2.exe")
      WinActivate
  Sleep, 3000
  inputCreateGame(false)
  return
}

^+p:: ;ctrl+shift+p
{
  initHotkey("pause")
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
  initHotkey("exitapp")
  ExitApp
}

;-------------------------------------------------------------------------------
; timers
;-------------------------------------------------------------------------------

inputAutoTimer: ;timer input auto
{
  if !WinActive("ahk_exe Torchlight2.exe")
    return
  if isColorFromPixel(6, 6, "0x000000") || isColorFromPixel(6, 6, "0x02051F") { ;loading or dead
    m_IsLoading := true
    return
  }
  if !isColorFromPixel(6, 6, "0x060818") { ;in game pet portrait
    m_LastDervish6 := A_TickCount - 20000
    m_LastDervish5 := A_TickCount - 18000
    m_LastDervish4 := A_TickCount - 16000
    return
  }
  showToolTip(DEBUG, "inputAutoTimer:")
  ;characterPreset := "uf1sokbz"
  ;characterPreset := "xrpjzo2c"
  ;characterPreset := "jaq3gmvf"
  ;characterPreset := "nswh4rcj"
  ;characterPreset := "cxteb5uk"
  characterPreset := "zm6ahnbf"
  ;characterPreset := "f7ybsowu"
  ;characterPreset := "dajx8vtg"
  isAutoHealthPotion := true
  isAutoManaPotion := true
  isAutoPetPotion := false
  isAutoSkill := true
  isAutoAttack := true
  isDervish6 := true
  isDervish5 := true
  isDervish4 := true
  lowHealth90Coord := [38, 522] ;90%
  lowMana90Coord := [754, 522] ;90%
  lowHealth66Coord := [38, 543] ;66%
  lowMana66Coord := [754, 543] ;66%
  lowHealth33Coord := [38, 568] ;33%
  lowMana33Coord := [754, 568] ;33%
  lowPetHealthCoord := [38, 10]
  lowPetManaCoord := [38, 22]
  skill1Coord := [251, 575]
  skillXObj := []
  activeskillCoord := [606, 582]
  berserkerChargeCoord := [474, 508]
  embermageChargeCoord := [467, 510]
  engineerChargeCoord := [467, 510]
  outlanderChargeCoord := [478, 516]
  loop, 10 {
    skillXObj.insert(skill1Coord[X] + ((A_Index - 1) * 34))
  }
  if isAutoHealthPotion {
    if !isColorFromPixel(lowHealth66Coord[X], lowHealth66Coord[Y], "0x082BFE") {
      if ((A_TickCount - m_LastHDrinkHealth) >= 8000) { ;potion cooldown 8s
        pressKeyCustom("z", KEYPRESS, 1, 1) ;use best potion
        m_LastHDrinkHealth := A_TickCount
        m_LastHDrinkHealth := A_TickCount
        showToolTip(INFO, "inputAuto() lowHealth66")
      }
    }
  }
  if isAutoManaPotion {
    if !isColorFromPixel(lowMana66Coord[X], lowMana66Coord[Y], "0xFC0536") {
      if ((A_TickCount - m_LastHDrinkMana) >= 8000) { ;potion cooldown 8s
        pressKeyCustom("x", KEYPRESS, 1, 1) ;use best potion
        m_LastHDrinkMana := A_TickCount
        showToolTip(INFO, "inputAuto() lowMana33")
      }
    }
  }
  if isAutoSkill {
    if isColorFromPixel(skillXObj[KEY_1], skill1Coord[Y], "0x6B8A9C") ;haste
      pressKeyCustom("1", KEYPRESS, 1, 1) ;use skill
    if isDervish6 && ((A_TickCount - m_LastDervish4) >= 16000) && isColorFromPixel(skillXObj[2], skill1Coord[Y], "0x6A8DA3") { ;devish vi
      pressKeyCustom("2", KEYPRESS, 1, 1) ;use skill
      m_LastDervish6 := A_TickCount
    } else if isDervish5 && ((A_TickCount - m_LastDervish6) >= 20000) && !isColorFromPixel(skillXObj[2], skill1Coord[Y], "0x6A8DA3") && isColorFromPixel(skillXObj[3], skill1Coord[Y], "0x6A8DA3") { ;devish v
      pressKeyCustom("3", KEYPRESS, 1, 1) ;use skill
      m_LastDervish5 := A_TickCount
    } else if isDervish4 && ((A_TickCount - m_LastDervish5) >= 18000) && !isColorFromPixel(skillXObj[2], skill1Coord[Y], "0x6A8DA3") && !isColorFromPixel(skillXObj[3], skill1Coord[Y], "0x6A8DA3")  && isColorFromPixel(skillXObj[4], skill1Coord[Y], "0x6A8DA3") { ;devish iv
      pressKeyCustom("4", KEYPRESS, 1, 1) ;use skill
      m_LastDervish4 := A_TickCount
    }
    if (characterPreset = "uf1sokbz") {
      ;if isColorFromPixel(embermageChargeCoord[X], embermageChargeCoord[Y], "0x416262") { ;embermage chargebar
      ;}
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        ;if !isColorFromPixel(lowMana66Coord[X], lowMana66Coord[Y], "0xFC0536") || !isColorFromPixel(lowHealth66Coord[X], lowHealth66Coord[Y], "0x082BFE") {
        ;  if isColorFromPixel(skillXObj[KEY_W], skill1Coord[Y], "0x003473") { ;death's bounty
        ;    pressKeyCustom("w", KEYPRESS, 1, 1) ;use skill
        ;  }
        ;}
        if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0xC1E6FF") ;blazing pillar
          pressKeyCustom("f", KEYPRESS, 1, 1) ;use skill
        if GetKeyState("a", "P") {
          pressKeyCustom("q", KEYPRESS, 500, "")
        } else if GetKeyState("s", "P") {
          if isColorFromPixel(skillXObj[KEY_R], skill1Coord[Y], "0xF7DF8F") ;ice prison
            pressKeyCustom("r", KEYPRESS, 1, 1) ;use skill
          if isColorFromPixel(skillXObj[KEY_W], skill1Coord[Y], "0x630608") ;hailstorm
            pressKeyCustom("w", KEYPRESS, 1, 1) ;use skill
          if isColorFromPixel(skillXObj[KEY_D], skill1Coord[Y], "0x00119D") ;firestorm
            pressKeyCustom("d", KEYPRESS, 1, 1) ;use skill
        }
      }
      if GetKeyState("c", "P") {
        if isColorFromPixel(skillXObj[KEY_E], skill1Coord[Y], "0xB34205") ;frost phase
          pressKeyCustom("e", KEYPRESS, 1, 1)
      }
    } else if (characterPreset = "xrpjzo2c") {
      if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0x3B2A21") ;iceshield
        pressKeyCustom("f", KEYPRESS, 1, 1) ;use skill
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        if isColorFromPixel(berserkerChargeCoord[X], berserkerChargeCoord[Y], "0x082F4C") ;berserker chargebar
          pressKeyCustom("w", KEYPRESS, 1, 1)
        if GetKeyState("a", "P") {
          pressKeyCustom("q", KEYPRESS, 1, "")
        } else if GetKeyState("s", "P") {
          pressKeyCustom("r", KEYPRESS, 1, "")
        }
      }
      if GetKeyState("c", "P") {
        if isColorFromPixel(skillXObj[KEY_E], skill1Coord[Y], "0x69208B") ;shadow burst
          pressKeyCustom("e", KEYPRESS, 1, 1)
      }
    } else if (characterPreset = "jaq3gmvf") {
      if m_IsLoading {
        clickMouseCustom("", "", "Left", 1, 500) ;use skill
        pressKeyCustom("y", KEYPRESS, 1, 1) ;swap active skill
        clickMouseCustom("", "", "Left", 1, 1) ;use skill
        m_IsLoading := false
      }
      if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0xFA6039") ;forcefield
        pressKeyCustom("f", KEYPRESS, 1, 1) ;use skill
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        if GetKeyState("a", "P") {
          ;if isColorFromPixel(skillXObj[KEY_W], skill1Coord[Y], "0x000C6E") ;emberquake
            pressKeyCustom("w", KEYPRESS, 1, 1) ;use skill
          pressKeyCustom("q", KEYPRESS, 1, "") ;use skill
        } else if GetKeyState("s", "P") {
          ;if isColorFromPixel(engineerChargeCoord[X], engineerChargeCoord[Y], "0x22699D") { ;engineer chargebar
          ;  if isColorFromPixel(skillXObj[KEY_R], skill1Coord[Y], "0x536C40") ;dynamo field
          ;    pressKeyCustom("r", KEYPRESS, 1, 1) ;use skill
          ;}
          ;if isColorFromPixel(skillXObj[KEY_D], skill1Coord[Y], "0xEF1862") ;emberreach
            pressKeyCustom("d", KEYPRESS, 1, 1) ;use skill
          pressKeyCustom("r", KEYPRESS, 1, 1) ;use skill
        }
      }
      if GetKeyState("c", "P") {
        if isColorFromPixel(skillXObj[KEY_E], skill1Coord[Y], "0xDB301B") ;storm burst
          pressKeyCustom("e", KEYPRESS, 1, 1)
      }
    } else if (characterPreset = "nswh4rcj") {
      if isColorFromPixel(activeskillCoord[X], activeskillCoord[Y], "0x6E1515") ;shadowmantle
        clickMouseCustom("", "", "Left", 1, 1) ;use skill ;use skil
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        if !isColorFromPixel(lowMana66Coord[X], lowMana66Coord[Y], "0xFC0536") ;auto attack when low mana
          clickMouseCustom("", "", "Right", 1, 1) ;click auto attack
        if GetKeyState("a", "P") {
          if isColorFromPixel(outlanderChargeCoord[X], outlanderChargeCoord[Y], "0x000E21") ;outlander chargebar
            pressKeyCustom("w", KEYPRESS, 500, "") ;use skill
          else
            pressKeyCustom("q", KEYPRESS, 500, "") ;use skill
        } else if GetKeyState("s", "P") {
          if isColorFromPixel(skillXObj[KEY_R], skill1Coord[Y], "0x182634") ;bramble wall
            pressKeyCustom("r", KEYPRESS, 1, 1) ;use skill
          if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0x263424") ;stone pact
            pressKeyCustom("f", KEYPRESS, 1, "") ;use skill
          if isColorFromPixel(skillXObj[KEY_D], skill1Coord[Y], "0x140D7E") ;blade pact
            pressKeyCustom("d", KEYPRESS, 1, 1) ;use skill
        }
      }
      if GetKeyState("c", "P") {
      }
    } else if (characterPreset = "cxteb5uk") {
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0xC1E6FF") ;blazing pillar
          pressKeyCustom("f", KEYPRESS, 1, 1) ;use skill
        if GetKeyState("a", "P") {
          if isColorFromPixel(skillXObj[KEY_E], skill1Coord[Y], "0xB34205") ;frost phase
            pressKeyCustom("e", KEYPRESS, 1, 1)
          pressKeyCustom("q", KEYPRESS, 500, "")
        } else if GetKeyState("s", "P") {
          if isColorFromPixel(skillXObj[KEY_R], skill1Coord[Y], "0xF7DF8F") ;ice prison
            pressKeyCustom("r", KEYPRESS, 1, 1) ;use skill
          if isColorFromPixel(skillXObj[KEY_W], skill1Coord[Y], "0x630608") ;hailstorm
            pressKeyCustom("w", KEYPRESS, 1, 1) ;use skill
          if isColorFromPixel(skillXObj[KEY_D], skill1Coord[Y], "0x00119D") ;firestorm
            pressKeyCustom("d", KEYPRESS, 1, 1) ;use skill
        }
      }
      if GetKeyState("c", "P") {
        if isColorFromPixel(skillXObj[KEY_E], skill1Coord[Y], "0xB34205") ;frost phase
          pressKeyCustom("e", KEYPRESS, 1, 1)
      }
    } else if (characterPreset = "zm6ahnbf") {
      if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0x3B2A21") ;iceshield
        pressKeyCustom("f", KEYPRESS, 1, 1) ;use skill
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        if isColorFromPixel(berserkerChargeCoord[X], berserkerChargeCoord[Y], "0x082F4C") ;berserker chargebar
          pressKeyCustom("w", KEYPRESS, 1, 1)
        if GetKeyState("a", "P") {
          pressKeyCustom("q", KEYPRESS, 1, "")
        } else if GetKeyState("s", "P") {
          pressKeyCustom("r", KEYPRESS, 1, "")
        }
      }
      if GetKeyState("c", "P") {
        if isColorFromPixel(skillXObj[KEY_E], skill1Coord[Y], "0x69208B") ;shadow burst
          pressKeyCustom("e", KEYPRESS, 1, 1)
      }
    } else if (characterPreset = "f7ybsowu") {
      if m_IsLoading {
        clickMouseCustom("", "", "Left", 1, 500) ;use skill
        pressKeyCustom("y", KEYPRESS, 1, 1) ;swap active skill
        clickMouseCustom("", "", "Left", 1, 1) ;use skill
        m_IsLoading := false
      }
      if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0xFA6039") ;forcefield
        pressKeyCustom("f", KEYPRESS, 1, 1) ;use skill
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        if GetKeyState("a", "P") {
        } else if GetKeyState("s", "P") {
          if isColorFromPixel(engineerChargeCoord[X], engineerChargeCoord[Y], "0x22699D") { ;engineer chargebar
            if isColorFromPixel(skillXObj[KEY_R], skill1Coord[Y], "0x536C40") ;dynamo field
              pressKeyCustom("r", KEYPRESS, 1, 1) ;use skill
          }
        }
      }
      if GetKeyState("c", "P") {
        if isColorFromPixel(skillXObj[KEY_E], skill1Coord[Y], "0xDB301B") ;storm burst
          pressKeyCustom("e", KEYPRESS, 1, 1)
      }
    } else if (characterPreset = "dajx8vtg") {
      if isColorFromPixel(activeskillCoord[X], activeskillCoord[Y], "0x6E1515") ;shadowmantle
        clickMouseCustom("", "", "Left", 1, 1) ;use skill ;use skil
      if GetKeyState("LButton", "P")
        return
      if GetKeyState("a", "P") || GetKeyState("s", "P") {
        if !isColorFromPixel(lowMana66Coord[X], lowMana66Coord[Y], "0xFC0536") ;auto attack when low mana
          clickMouseCustom("", "", "Right", 1, 1) ;click auto attack
        if GetKeyState("a", "P") {
          if isColorFromPixel(skillXObj[KEY_W], skill1Coord[Y], "0x014311") ;venomous hail
            pressKeyCustom("w", KEYPRESS, 1, 1) ;use skill
          pressKeyCustom("q", KEYPRESS, 500, "") ;use skill
        } else if GetKeyState("s", "P") {
          if isColorFromPixel(skillXObj[KEY_R], skill1Coord[Y], "0x182634") ;bramble wall
            pressKeyCustom("r", KEYPRESS, 1, 1) ;use skill
          if isColorFromPixel(skillXObj[KEY_F], skill1Coord[Y], "0x263424") ;stone pact
            pressKeyCustom("f", KEYPRESS, 1, "") ;use skill
          if isColorFromPixel(skillXObj[KEY_D], skill1Coord[Y], "0x140D7E") ;blade pact
            pressKeyCustom("d", KEYPRESS, 1, 1) ;use skill
        }
      }
      if GetKeyState("c", "P") {
      }
    }
  }
  if isAutoAttack {
    clickMouseCustom("", "", "Right", 1, "") ;click auto attack
  }
  if isAutoPetPotion {
    if !isColorFromPixel(lowPetHealthCoord[X], lowPetHealthCoord[Y], "0x0000DA") {
      if ((A_TickCount - m_LastHDrinkPetHealth) >= 8000) {
        pressKeyCustom("Shift", KEYDOWN, 1, 1)
        pressKeyCustom("z", KEYPRESS, 1, 1) ;use best potion
        pressKeyCustom("Shift", KEYUP, 1, 1)
        m_LastHDrinkPetHealth := A_TickCount
      }
    }
    if !isColorFromPixel(lowPetManaCoord[X], lowPetManaCoord[Y], "0xC9012F") {
      if ((A_TickCount - m_LastHDrinkPetMana) >= 8000) {
        pressKeyCustom("Shift", KEYDOWN, 1, 1)
        pressKeyCustom("x", KEYPRESS, 1, 1) ;use best potion
        pressKeyCustom("Shift", KEYUP, 1, 1)
        m_LastHDrinkPetMana := A_TickCount
      }
    }
  }
}

removeTooltipTimer: ;timer remove tooltip
{
  ToolTip
  return
}

;-------------------------------------------------------------------------------
; input functions
;-------------------------------------------------------------------------------

/*
input chat
inputChat("/help")          ;displays many commands that are gameplay related
inputChat("/blowkiss")      ;your character blows a kiss
inputChat("/bow")           ;your character takes a bow
inputChat("/breakdance")    ;your character breakdances
inputChat("/burp")          ;your character burps
inputChat("/cls")           ;clears the chat
inputChat("/clueless")      ;your character acts clueless
inputChat("/cmon")          ;your character impatiently beckons someone to follow
inputChat("/cough")         ;your character coughs
inputChat("/cry")           ;your character starts crying
inputChat("/dance")         ;your character does a little dance until you move or do something else
inputChat("/despair")       ;your character loses all hope
inputChat("/drums")         ;your character plays some imaginary drums
inputChat("/emote")         ;functions like a /me command
inputChat("/emotes")        ;displays many commands, but not all of them
inputChat("/facepalm")      ;your character facepalms
inputChat("/gunshow")       ;your character shows his guns arms
inputChat("/jump")          ;your character tries to jump but can't
inputChat("/laugh")         ;your character laughs
inputChat("/lute")          ;your character plays on an imaginary lute
inputChat("/oops")          ;your character says oops
inputChat("/roll")          ;you roll a 1d100
inputChat("/salute")        ;your character salutes
inputChat("/slap")          ;your character slaps someone's face
inputChat("/slowclap")      ;your character claps ironically
inputChat("/taunt")         ;your character does a taunt
inputChat("/victory")       ;your character acts victorious
inputChat("/wave")          ;your character waves
inputChat("/whathaveidone") ;your character falls to his knees in dispair
inputChat("/pvp")           ;toggles pvp flag
*/
inputChat(chat) {
  pressKey("Enter", KEYPRESS, 100)
  pressKey(chat, KEYRAW, 100) ;press chat
  pressKey("Enter", KEYPRESS, 100)
}

/*
input hunt
ng cycle skeleton level ng: lv55, ng+2:81, ng+3:100, ng+4: lv120, ng+5: lv120
registering a potion on a hotkey and turn on cheat engine unlimited hotbar consume
switch pet to be passive
toggle item filter
*/
inputHunt() {
  MouseGetPos, itemX, itemY
  loop {
    loop, 20 {
      clickMouse(itemX, itemY, "Right", 10) ;spawn skeleton
    }
    Sleep, 10
    pressKey("f", KEYPRESS, 10) ;use best potion
    pressKey("Shift", KEYDOWN, 10)
    moveMouse(itemX, itemY - 80, 10) ;to avoid interfering with image search
    ;pressKeyCustom("q", KEYPRESS, 500, 10) ;use skill
    pressKeyCustom("q", KEYPRESS, 500, 10) ;use skill
    pressKey("Shift", KEYUP, 10)
  }
}

/*
input buy item
*/
inputBuyItem(times) {
  MouseGetPos, itemX, itemY
  pressKey("Shift", KEYDOWN, 40) ;buy item
  loop, % times {
    clickMouse(itemX, itemY, "Left", 40) ;click buy item
  }
  pressKey("Shift", KEYUP, 40) ;buy item
}

/*
input console
*/
inputConsole(command) {
  pressKey("Insert", KEYPRESS, 40) ;open console
  pressKey(command, KEYRAW, 40) ;press command
  pressKey("Enter", KEYPRESS, 40) ;press command
  pressKey("Insert", KEYPRESS, 40) ;close console
}

/*
input get item
*/
inputGetItem() {
  pressKey("Shift", KEYDOWN, 40) ;select pet
  loop, {
    clickMouse("", "", "Right", 1) ;click get item
  }
  pressKey("Shift", KEYUP, 40) ;select pet
}

/*
input login game
*/
inputLoginGame() {
  loginCoord := [765, 587]
  loginTitleCoord := [289, 174]
  passwordCursorCoord := [309, 324]
  clickMouse(loginCoord[X], loginCoord[Y], "Left", 100) ;click mainmenu login
  clickMouse(passwordCursorCoord[X], passwordCursorCoord[Y], "Left", 100) ;click cursor to password
  pressKey(LOGIN_PASSWORD, KEYRAW, 100) ;press password
  pressKey("Enter", KEYPRESS, 100) ;press password
  sleepUntilPixelFound(false, loginTitleCoord[X], loginTitleCoord[Y], "0x0A1A39", 10, 5000) ;sleep until pixel not found
}

/*
input show internet game
*/
inputShowInternetGame() {
  mainMenuResumeCoord := [397, 553]
  internetCoord := [397, 301]
  clickMouse(mainMenuResumeCoord[X], mainMenuResumeCoord[Y], "Left", 100) ;click mainmenu resume
  clickMouse(internetCoord[X], internetCoord[Y], "Left", 100) ;click cursor to password
}

/*
input create game
*/
inputCreateGame(isReroll) {
  mainMenuResumeCoord := [397, 553]
  lanCoord := [397, 343]
  lanTitleCoord := [322, 41]
  lanCreateGameCoord := [654, 563]
  lanReRollCoord := [603, 305]
  lanCreateGameOkCoord := [639, 388]
  singlePlayerCoord := [397, 256]
  clickMouseCustom(mainMenuResumeCoord[X], mainMenuResumeCoord[Y], "Left", 100, 100) ;click mainmenu resume
  if isReroll {
    clickMouse(lanCoord[X], lanCoord[Y], "Left", 100) ;click mainmenu lan
    sleepUntilPixelFound(true, lanTitleCoord[X], lanTitleCoord[Y], "0xFFFFFF", 10, 5000) ;sleep until pixel found
    clickMouse(lanCreateGameCoord[X], lanCreateGameCoord[Y], "Left", 100) ;click mainmenu create game
    clickMouse(lanReRollCoord[X], lanReRollCoord[Y], "Left", 100) ;click mainmenu reroll
    clickMouse(lanCreateGameOkCoord[X], lanCreateGameOkCoord[Y], "Left", 100) ;click mainmenu ok
  } else {
    clickMouse(singlePlayerCoord[X], singlePlayerCoord[Y], "Left", 100) ;click mainmenu single player
  }
}

/*
input save game
*/
inputSaveGame() {
  optionCoord := [789, 5]
  optionSaveCoord := [475, 412]
  pressKey("Esc", KEYPRESS, 40) ;close
  clickMouse(optionCoord[X], optionCoord[Y], "Left", 100) ;click option
  clickMouse(optionSaveCoord[X], optionSaveCoord[Y], "Left", 100) ;click option save
}

/*
input quit game
*/
inputQuitGame() {
  optionCoord := [789, 5]
  optionQuitCoord := [399, 344]
  mainMenuBottomLeftCoord := [21, 578]
  pressKey("Esc", KEYPRESS, 40) ;close
  clickMouse(optionCoord[X], optionCoord[Y], "Left", 100) ;click option
  clickMouse(optionQuitCoord[X], optionQuitCoord[Y], "Left", 100) ;click option quit
  sleepUntilPixelFound(true, mainMenuBottomLeftCoord[X], mainMenuBottomLeftCoord[Y], "0xFFFFFF", 10, 5000) ;sleep until pixel found
  Sleep, 100
}

/*
input fish
recommend toggle item fliter
*/
inputFish() {
  startTime := A_TickCount
  fishCoord := [397, 294]
  fishDialogCoord := [567, 214]
  fishDialogOkCoord := [584, 443]
  MouseGetPos, holeX, holeY
  Loop {
    totalCount++
    clickMouseCustom(holeX, holeY, "Right", 100, 100) ;click well
    if !sleepUntilPixelFound(true, fishCoord[X], fishCoord[Y], "0x232026", 10, 3000) ;sleep until pixel found
      continue
    clickMouseCustom(fishCoord[X], fishCoord[Y], "Left", 100, 100) ;click fish button
    if !sleepUntilPixelFound(true, fishDialogCoord[X], fishDialogCoord[Y], "0x00000C", 10, 3000) ;sleep until pixel found
      continue
    clickMouseCustom(fishDialogOkCoord[X], fishDialogOkCoord[Y], "Left", 100, 100) ;click fish dialog button
    if (Mod(A_Index, 100) = 0) {
      endTime := A_TickCount - startTime
      message := "inputFish() total:" totalCount ", elapsedTime:" formatHhmmss(endTime) ", speed/case:" Round(endTime / totalCount) "ms"
      writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
      showToolTip(DEBUG, message)
    }
  }
}

/*
input move item from merchant invetory
*/
inputMoveItemFromMerchantInvetory() {
  inventoryGap := [46, 46]
  merchantInventoryCoord := [26, 55]
  merchantColorObj := ["0x081C18", "0x082421", "0x082925", "0x102C29", "0x082926", "0x082421", "0x082421", "0x081F1E", "0x081C18", "0x103029", "0x102F2B", "0x103431", "0x10332F", "0x103029", "0x082821", "0x08201B", "0x081E18", "0x102D29", "0x103431", "0x183C39", "0x113532", "0x10312B", "0x0B2A24", "0x082822", "0x082018", "0x10332E", "0x10312C", "0x10312B", "0x133734", "0x113631", "0x102E29", "0x082621", "0x081918", "0x082B29", "0x10302A", "0x103431", "0x103831", "0x10302E", "0x103029", "0x082521"]
  moveMouse(A_ScreenWidth, A_ScreenHeight, 10) ;to avoid interfering with image search
  pressKey("Shift", KEYDOWN, 10) ;move item
  loop, 5 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      merchantInventoryIndex++
      mouseX := merchantInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := merchantInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, merchantColorObj[merchantInventoryIndex])
         continue
       else
         clickMouse(mouseX, mouseY, "Left", 10) ;click stash invetory item move to player
    }
  }
  pressKey("Shift", KEYUP, 10) ;move item
  playBeep(3)
}

/*
input move item from player invetory
*/
inputMoveItemFromPlayerInventory(isSell) {
  inventoryGap := [46, 46]
  playerInventoryCoord := [443, 341]
  petInventoryCoord := [26, 341]
  petSellAllCoord := [316, 526]
  playerInventoryColorObj := ["0x132C42", "0x183C5A", "0x183F60", "0x1E4668", "0x184163", "0x214568", "0x214263", "0x1F3752", "0x18334A", "0x183C5A", "0x18405B", "0x183F60", "0x21496B", "0x194061", "0x214263", "0x15334C", "0x13304A", "0x184163", "0x214363", "0x1E4163", "0x21466B", "0x214865", "0x21415E", "0x183958", "0x102C42", "0x183852", "0x153452", "0x21415F", "0x183856", "0x1B3A57", "0x183552", "0x183452"]
  petInventoryColorObj := ["0x102C42", "0x193D5B", "0x183E5D", "0x1B4266", "0x184163", "0x214563", "0x214163", "0x1C3856", "0x18324A", "0x183C5A", "0x18415D", "0x204266", "0x214668", "0x1C4163", "0x214163", "0x183652", "0x102E46", "0x184163", "0x214363", "0x184163", "0x1E4668", "0x214968", "0x214161", "0x1F3C5A", "0x102A40", "0x183452", "0x113452", "0x21415C", "0x183A5A", "0x1A3A56", "0x133451", "0x183452"]
  moveMouse(A_ScreenWidth, A_ScreenHeight, 10) ;to avoid interfering with image search
  pressKey("Shift", KEYDOWN, 10) ;move item
  loop, 4 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      playerInventoryIndex++
      mouseX := playerInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := playerInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, playerInventoryColorObj[playerInventoryIndex])
        continue
      else
        clickMouse(mouseX, mouseY, "Left", 10) ;click player invetory item move to stash
    }
  }
  Sleep, 10
  if isSell {
    clickMouse(petSellAllCoord[X], petSellAllCoord[Y], "Left", 10) ;click sell all
    pressKey("Shift", KEYUP, 10) ;move item
    return
  }
  loop, 4 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      petInventoryIndex++
      mouseX := petInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := petInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, petInventoryColorObj[petInventoryIndex])
        continue
      else
        clickMouse(mouseX, mouseY, "Left", 10) ;click player invetory item move to stash
    }
  }
  pressKey("Shift", KEYUP, 10) ;move item
  playBeep(3)
}

/*
input move item from stash invetory
*/
inputMoveItemFromStashInventory() {
  inventoryGap := [46, 46]
  stashInventoryCoord := [26, 55]
  stashColorObj := ["0x081C18", "0x082221", "0x082821", "0x082923", "0x082623", "0x082321", "0x082321", "0x081C18", "0x081C18", "0x103029", "0x102D29", "0x103231", "0x102E29", "0x102F29", "0x082821", "0x081D18", "0x082018", "0x103029", "0x103431", "0x183C39", "0x103431", "0x103029", "0x082C29", "0x082821", "0x08221A", "0x10312D", "0x103029", "0x103431", "0x103532", "0x103331", "0x103029", "0x082821", "0x081D18", "0x0B2C29", "0x10312C", "0x103431", "0x103831", "0x103031", "0x103029", "0x082825"]
  moveMouse(A_ScreenWidth, A_ScreenHeight, 40) ;to avoid interfering with image search
  pressKey("Shift", KEYDOWN, 40) ;move item
  loop, 5 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      stashInventoryIndex++
      mouseX := stashInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := stashInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
     if isColorFromPixel(mouseX, mouseY, stashColorObj[stashInventoryIndex])
        continue
      else
        clickMouse(mouseX, mouseY, "Left", 40) ;click stash invetory item move to player
    }
  }
  pressKey("Shift", KEYUP, 40) ;move item
  pressKey("Esc", KEYPRESS, 40) ;close
  playBeep(3)
}

/*
input gamble
*/
inputGamble() {
  weaponTab := [136, 281]
  armorTab := [178, 281]
  clickMouse(weaponTab[X], weaponTab[Y], "Left", 40) ;click inventory tab
  inputMoveItemFromMerchantInvetory()
  clickMouse(armorTab[X], armorTab[Y], "Left", 40) ;click inventory tab
  inputMoveItemFromMerchantInvetory()
  inputMoveItemFromPlayerInventory(true)
}

/*
input transmute scroll
*/
inputTransmuteScroll() {
  inventoryGap := [46, 46]
  playerInventoryCoord := [443, 378]
  petInventoryCoord := [26, 379]
  playerInventoryTabCoord := [645, 522]
  petInventoryTabCoord := [213, 522]
  transmuteCoord := [271, 233]
  playerInventoryColorObj := ["0x5C8DB5", "0x84A9C6", "0x81A7C6", "0x89AAC6", "0x7BA2C6", "0x84AAC7", "0x84AAC6", "0x5A8CB5", "0x4A79AD", "0x7BA6C6", "0x81A7C6", "0x84A8C6", "0x82AAC6", "0x7BA2C6", "0x6F98BD", "0x527EA5", "0x496E97", "0x6594B6", "0x6594B5", "0x6B99B9", "0x74A2C5", "0x5D8FB6", "0x5A8BB1", "0x517BA4"]
  petInventoryColorObj := ["0x5285AD", "0x84A9C6", "0x7CA3C6", "0x8CAAC6", "0x7BA2C6", "0x84AAC9", "0x84AAC6", "0x5A8CB5", "0x4073A2", "0x7BA1C6", "0x81A7C6", "0x81A7C6", "0x84AACB", "0x7BA1C6", "0x6D97BD", "0x5284A7", "0x466B94", "0x6394B9", "0x6B95B5", "0x6899BA", "0x7BA5C6", "0x6493BD", "0x6396B5", "0x527EA6"]
  moveMouse(A_ScreenWidth, A_ScreenHeight, 40) ;to avoid interfering with image search
  clickMouse(playerInventoryTabCoord[X], playerInventoryTabCoord[Y], "Left", 40) ;click inventory tab
  clickMouse(petInventoryTabCoord[X], petInventoryTabCoord[Y], "Left", 40) ;click inventory tab
  pressKey("Shift", KEYDOWN, 40) ;move item
  loop, 3 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      playerInventoryIndex++
      mouseX := playerInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := playerInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, playerInventoryColorObj[playerInventoryIndex])
        break, 2 ;exit if the first space or middle of the inventory is empty
      clickMouse(mouseX, mouseY, "Left", 70) ;click inventory item move to transmuter
      if isColorFromPixel(transmuteCoord[X], transmuteCoord[Y], "0xB4B1B4")
        clickMouse(transmuteCoord[X], transmuteCoord[Y], "Left", 70) ;click transmute item
    }
  }
  loop, 3 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      petInventoryIndex++
      mouseX := petInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := petInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, petInventoryColorObj[petInventoryIndex])
        break, 2 ;exit if the first space or middle of the inventory is empty
      clickMouse(mouseX, mouseY, "Left", 70) ;click inventory item move to transmuter
      if isColorFromPixel(transmuteCoord[X], transmuteCoord[Y], "0xB4B1B4")
        clickMouse(transmuteCoord[X], transmuteCoord[Y], "Left", 70) ;click transmute item
    }
  }
  pressKey("Shift", KEYUP, 40) ;move item
  if isColorFromPixel(transmuteCoord[X], transmuteCoord[Y], "0xB4B1B4")
    clickMouse(transmuteCoord[X], transmuteCoord[Y], "Left", 70) ;click transmute item
  moveMouse(playerInventoryCoord[X], playerInventoryCoord[Y], 40) ;show item description
  playBeep(3)
}

/*
input transmute item
*/
inputTransmuteItem() {
  inventoryGap := [46, 46]
  playerInventoryCoord := [443, 341]
  petInventoryCoord := [26, 341]
  transmuteCoord := [271, 233]
  playerInventoryColorObj := ["0x132C42", "0x183C5A", "0x183F60", "0x1E4668", "0x184163", "0x214568", "0x214263", "0x1F3752", "0x18334A", "0x183C5A", "0x18405B", "0x183F60", "0x21496B", "0x194061", "0x214263", "0x15334C", "0x13304A", "0x184163", "0x214363", "0x1E4163", "0x21466B", "0x214865", "0x21415E", "0x183958", "0x102C42", "0x183852", "0x153452", "0x21415F", "0x183856", "0x1B3A57", "0x183552", "0x183452"]
  petInventoryColorObj := ["0x102C42", "0x193D5B", "0x183E5D", "0x1B4266", "0x184163", "0x214563", "0x214163", "0x1C3856", "0x18324A", "0x183C5A", "0x18415D", "0x204266", "0x214668", "0x1C4163", "0x214163", "0x183652", "0x102E46", "0x184163", "0x214363", "0x184163", "0x1E4668", "0x214968", "0x214161", "0x1F3C5A", "0x102A40", "0x183452", "0x113452", "0x21415C", "0x183A5A", "0x1A3A56", "0x133451", "0x183452"]
  moveMouse(A_ScreenWidth, A_ScreenHeight, 40) ;to avoid interfering with image search
  pressKey("Shift", KEYDOWN, 40) ;move item
  loop, 4 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      playerInventoryIndex++
      mouseX := playerInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := playerInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, playerInventoryColorObj[playerInventoryIndex])
        break, 2 ;exit if the first space or middle of the inventory is empty
      clickMouse(mouseX, mouseY, "Left", 70) ;click inventory item move to transmuter
      if isColorFromPixel(transmuteCoord[X], transmuteCoord[Y], "0xB4B1B4")
        clickMouse(transmuteCoord[X], transmuteCoord[Y], "Left", 70) ;click transmute item
    }
  }
  loop, 4 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      petInventoryIndex++
      mouseX := petInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := petInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, petInventoryColorObj[petInventoryIndex])
        break, 2 ;exit if the first space or middle of the inventory is empty
      clickMouse(mouseX, mouseY, "Left", 70) ;click inventory item move to transmuter
      if isColorFromPixel(transmuteCoord[X], transmuteCoord[Y], "0xB4B1B4")
        clickMouse(transmuteCoord[X], transmuteCoord[Y], "Left", 70) ;click transmute item
    }
  }
  pressKey("Shift", KEYUP, 40) ;move item
  if isColorFromPixel(transmuteCoord[X], transmuteCoord[Y], "0xB4B1B4")
    clickMouse(transmuteCoord[X], transmuteCoord[Y], "Left", 70) ;click transmute item
  moveMouse(playerInventoryCoord[X], playerInventoryCoord[Y], 40) ;show item description
  playBeep(3)
}

/*
input item to enchanter
*/
inputMovetItem(pixelX, pixelY) {
  pressKey("Shift", KEYDOWN, 40) ;move item
  clickMouse(pixelX, pixelY, "Left", 40) ;click inventory item move to enchanter
  pressKey("Shift", KEYUP, 40) ;move item
}

/*
input disenchant
*/
inputDisenchant() {
  disenchantCoord := [303, 241]
  clickMouse(disenchantCoord[X], disenchantCoord[Y], "Left", 40) ;click disenchant item
}

/*
input add enchant
*/
inputAddEnchant() {
  enchantSlotCoord := [188, 102]
  enchantCoord := [302, 184]
  enchantDialogCoord := [363, 186]
  enchantOkCoord := [493, 405]
  clickMouse(enchantCoord[X], enchantCoord[Y], "Left", 40) ;click enchant item
  clickMouse(enchantOkCoord[X], enchantOkCoord[Y], "Left", 40) ;click enchant ok
  moveMouse(enchantSlotCoord[X], enchantSlotCoord[Y], 40) ;show item description
}

/*
input inventory loop enchant
*/
inputInventoryLoopEnchant(filterTypeObj, filterStatObj) {
  inventoryGap := [46, 46]
  playerInventoryCoord := [443, 341]
  enchantSlotCoord := [188, 102]
  disenchantButtonCoord := [292, 228]
  playerInventoryColorObj := ["0x132C42", "0x183C5A", "0x183F60", "0x1E4668", "0x184163", "0x214568", "0x214263", "0x1F3752", "0x18334A", "0x183C5A", "0x18405B", "0x183F60", "0x21496B", "0x194061", "0x214263", "0x15334C", "0x13304A", "0x184163", "0x214363", "0x1E4163", "0x21466B", "0x214865", "0x21415E", "0x183958", "0x102C42", "0x183852", "0x153452", "0x21415F", "0x183856", "0x1B3A57", "0x183552", "0x183452"]
  moveMouse(A_ScreenWidth, A_ScreenHeight, 40) ;to avoid interfering with image search
  loop, 4 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      playerInventoryIndex++
      mouseX := playerInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := playerInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, playerInventoryColorObj[playerInventoryIndex])
        continue
      inputMovetItem(mouseX, mouseY)
      sleepUntilPixelFound(true, disenchantButtonCoord[X], disenchantButtonCoord[Y], "0x134968", 10, 200) ;sleep until pixel found
      inputLoopEnchant(filterTypeObj, filterStatObj)
      inputMovetItem("", "")
    }
  }
  playBeep(3)
}

/*
input inventory loop add enchant
*/
inputInventoryLoopAddEnchant() {
  inventoryGap := [46, 46]
  playerInventoryCoord := [443, 341]
  petInventoryCoord := [26, 341]
  disenchantButtonCoord := [292, 228]
  playerInventoryColorObj := ["0x132C42", "0x183C5A", "0x183F60", "0x1E4668", "0x184163", "0x214568", "0x214263", "0x1F3752", "0x18334A", "0x183C5A", "0x18405B", "0x183F60", "0x21496B", "0x194061", "0x214263", "0x15334C", "0x13304A", "0x184163", "0x214363", "0x1E4163", "0x21466B", "0x214865", "0x21415E", "0x183958", "0x102C42", "0x183852", "0x153452", "0x21415F", "0x183856", "0x1B3A57", "0x183552", "0x183452"]
  petInventoryColorObj := ["0x102C42", "0x193D5B", "0x183E5D", "0x1B4266", "0x184163", "0x214563", "0x214163", "0x1C3856", "0x18324A", "0x183C5A", "0x18415D", "0x204266", "0x214668", "0x1C4163", "0x214163", "0x183652", "0x102E46", "0x184163", "0x214363", "0x184163", "0x1E4668", "0x214968", "0x214161", "0x1F3C5A", "0x102A40", "0x183452", "0x113452", "0x21415C", "0x183A5A", "0x1A3A56", "0x133451", "0x183452"]
  moveMouse(A_ScreenWidth, A_ScreenHeight, 10) ;to avoid interfering with image search
  loop, 4 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      playerInventoryIndex++
      mouseX := playerInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := playerInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, playerInventoryColorObj[playerInventoryIndex])
        continue
      inputMovetItem(mouseX, mouseY)
      sleepUntilPixelFound(true, disenchantButtonCoord[X], disenchantButtonCoord[Y], "0x134968", 10, 200) ;sleep until pixel found
      inputAddEnchant()
      inputMovetItem("", "")
    }
  }
  loop, 4 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      petInventoryIndex++
      mouseX := petInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := petInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      if isColorFromPixel(mouseX, mouseY, petInventoryColorObj[petInventoryIndex])
        continue
      inputMovetItem(mouseX, mouseY)
      sleepUntilPixelFound(true, disenchantButtonCoord[X], disenchantButtonCoord[Y], "0x134968", 10, 200) ;sleep until pixel found
      inputAddEnchant()
      inputMovetItem("", "")
    }
  }
  playBeep(3)
}

/*
input loop enchant
*/
inputLoopEnchant(filterTypeObj, filterStatObj) {
  startTime := A_TickCount
  enchantSlotCoord := [188, 102]
  disenchantCoord := [303, 241]
  enchantCoord := [302, 184]
  enchantDialogCoord := [363, 186]
  enchantOkCoord := [493, 405]
  powerfulCoord := [489, 314]
  ocrRect := [222, 84, 142, 430]
  clickMouse(disenchantCoord[X], disenchantCoord[Y], "Left", 40) ;click disenchant item
  loop {
    clickMouse(enchantCoord[X], enchantCoord[Y], "Left", "") ;click enchant item
    sleepUntilPixelFound(true, enchantDialogCoord[X], enchantDialogCoord[Y], "0xFFFFFF", 1, 5000) ;sleep until pixel found
    enchantmentCount++
    totalCount++
    isPowerful := isImageFromCoordinate(powerfulCoord[X], powerfulCoord[Y], 1, 3, POWERFUL_FILE)
    clickMouse(enchantOkCoord[X], enchantOkCoord[Y], "Left", 40) ;click enchant ok
    if isPowerful {
      powerfulCount++
      if (enchantmentCount >= 3) {
        x3PowerfulCount++
        moveMouse(enchantSlotCoord[X] - 1, enchantSlotCoord[Y] - 1, 1) ;refresh mouse cursor
        moveMouse(enchantSlotCoord[X], enchantSlotCoord[Y], 40) ;show item description
        playBeep(1)
        ;sleepUntilPixelFound(true, ocrRect[X], ocrRect[Y], "0x02141C", 10, 5000) ;sleep until pixel found
        if isDisenchant(getTextFromOcr(ocrRect[X], ocrRect[Y], ocrRect[WIDTH], ocrRect[HEIGHT]), filterTypeObj, filterStatObj) {
          clickMouse(disenchantCoord[X], disenchantCoord[Y], "Left", 40) ;click disenchant item
          enchantmentCount := 0
        } else {
          playBeep(2)
          break
        }
      }
    } else {
      clickMouse(disenchantCoord[X], disenchantCoord[Y], "Left", 40) ;click disenchant item
      enchantmentCount := 0
    }
  }
  moveMouse(enchantSlotCoord[X], enchantSlotCoord[Y], 40) ;show item description
  endTime := A_TickCount - startTime
  message := "inputLoopEnchant() total:" totalCount ", powerful:" powerfulCount "(" formatPercent(powerfulCount, totalCount) "), x3Powerful:" x3PowerfulCount "(" formatPercent(x3PowerfulCount, totalCount) "), elapsedTime:" formatHhmmss(endTime) ", speed/case:" Round(endTime / totalCount) "ms"
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
}

;-------------------------------------------------------------------------------
; screen functions
;-------------------------------------------------------------------------------

/*
get text from ocr
ocr unrecognized case netherrealm axe
*/
getTextFromOcr(startX, startY, width, height) {
  endX := startX + width
  endY :=startY + height
  startTime := A_TickCount
  RunWait, %CAPTURE2TEXT_FILE% --s "%startX% %startY% %endX% %endY%" --clipboard, , Hide ;as the area narrows the speed increase
  endTime := A_TickCount - startTime
  message := "RunWait() CAPTURE2TEXT_FILE speed:" Round(endTime) "ms"
  writeLogFile(DEBUG, A_ThisFunc, A_LineNumber, message)
  ocrText := trim(clipboard, "`n`r`t ")
  filteredOcrText := filterOcrText(ocrText)
  message := "getTextFromOcr() " (ErrorLevel = 0 ? "succeed" : "failed") " filteredOcrText: " filteredOcrText ", ocrText: " ocrText
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
  return filteredOcrText
}

/*
is image from coordinate
*/
isImageFromCoordinate(startX, startY, width, height, imageFile) {
  endX := startX + width
  endY :=startY + height
  ImageSearch, foundX, foundY, startX, startY, endX, endY, %imageFile%
  result := ErrorLevel = 0 ? true : false
  message := "ImageSearch() " (result ? "succeed " foundX ", " foundY : "failed") ", " imageFile
  writeLogFile(DEBUG, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
  return result
}

/*
is color from pixel
*/
isColorFromPixel(startX, startY, color) {
  endX := startX + 1
  endY :=startY + 1
  PixelSearch, foundX, foundY, startX, startY, endX, endY, color, ,Fast
  result := ErrorLevel = 0 ? true : false
  message := "PixelSearch() " (result ? "succeed " foundX ", " foundY : "failed") " " color
  writeLogFile(DEBUG, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
  if result
    return true
  else
    return false
}

;-------------------------------------------------------------------------------
; etc functions
;-------------------------------------------------------------------------------

/*
is disenchant
*/
isDisenchant(enchantText, filterTypeObj, filterStatObj) {
  if (enchantText = "")
    return true
  if (filterTypeObj = "")
    return false
  for k, v in filterTypeObj {
    if InStr(filterTypeObj[A_index], "&&") {
      filterOrObj := StrSplit(filterTypeObj[A_index], "&&")
      for k, v in filterOrObj {
        if InStr(enchantText, trim(filterOrObj[A_index]), false) ;case insensitive
          filterOrCount++
      }
      if (filterOrObj.MaxIndex() = filterOrCount)
        return true
    } else {
      if InStr(enchantText, filterTypeObj[A_index], false) ;case insensitive
        return true
    }
  }
  if (filterStatObj = "")
    return false
  if InStr(enchantText, "+") {
    enchantStatObj := StrSplit(enchantText, "+")
    for k, v in enchantStatObj {
      enchantTotalStat += trim(StrSplit(enchantStatObj[A_Index], " ")[1])
    }
    for k, v in filterStatObj {
      if (filterStatObj[A_Index][1] = "+")
        filterStat := filterStatObj[A_Index][2]
    }
  } else if InStr(enchantText, "conveys", false) { ;case insensitive
    enchantTotalStat := trim(StrSplit(StrSplit(enchantText, "Conveys ")[2], " ")[1])
    for k, v in filterStatObj {
      if (filterStatObj[A_Index][1] = "conveys")
        filterStat := filterStatObj[A_Index][2]
    }
  }
  if (filterStat <= enchantTotalStat)
    return false
  else
    return true
}

/*
filter ocr text
*/
filterOcrText(ocrText) {
  if InStr(ocrText, "Enchantments: 3", false)
    return trim(StrSplit(trim(StrSplit(trim(StrSplit(ocrText, "Enchantments: 3")[2]), "Set:")[1]), "Requires")[1]) ;"Enchantments: ocrText Set: Requires"
  else ;when ocr scan fails
    return ""
}

/*
sleep until pixel found
*/
sleepUntilPixelFound(isFound, pixelX, pixelY, searchColor, delay, maxDelay) {
  loop {
    if (maxDelay <= (delay * A_Index)) {
      writeLogFile(WARN, A_ThisFunc, A_LineNumber, "maxDelay over " maxDelay "ms")
      return false
    }
    if (isFound = isColorFromPixel(pixelX, pixelY, searchColor))
      return true
    else
      Sleep, %delay%
  }
}

;-------------------------------------------------------------------------------
; test functions
;-------------------------------------------------------------------------------

/*
test text from ocr
*/
testTextFromOcr() {
  ocrRect := [222, 84, 142, 430]
  inputDisenchant()
  inputAddEnchant()
  getTextFromOcr(ocrRect[X], ocrRect[Y], ocrRect[WIDTH], ocrRect[HEIGHT])
  sleepUntilPixelFound(true, ocrRect[X], ocrRect[Y], "0x02141C", 10, 5000)
  showToolTip(INFO, "ocr complete")
}

/*
test is disenchant
*/
testIsDisenchant() {
  ;enchantText := "+211 Focus AttribL or ?h'  ,7 . &Dex ?,"
  enchantText := "+211 Focus AttribL or ?h'  ,7 . &Dex ?s"
  ;enchantText :="+297 Ice Damage "
  ;enchantText := "+86 lce Damage Conveys 2710 lce[ or_. : . . Strength 104 & Dex Legendary Claw Col Number Four of Five"
  ;filterTypeObj := ["chance", "conveys && +"] ;only conveys, only element damage
  filterTypeObj := ["str att", "dex att", "vit att"] ;only foc
  ;filterStatObj := [["+", 200], ["conveys", 3900]]
  filterStatObj := [["+", 207]]
  message := isDisenchant(enchantText, filterTypeObj, filterStatObj)
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  showToolTip(INFO, message)
}

/*
test pixel color ocr
*/
testPixelColorOcr() {
  ocrRect := [222, 84, 142, 430]
  getPixelColor(ocrRect[X], ocrRect[Y])
}

/*
test pixel color input auto
*/
testPixelColorInputAuto() {
  skill1Coord := [251, 575]
  skillXObj := []
  activeskillCoord := [606, 582]
  lowHealth90Coord := [38, 522] ;90%
  lowMana90Coord := [754, 522] ;90%
  lowHealth66Coord := [38, 543] ;66%
  lowMana66Coord := [754, 543] ;66%
  lowHealth33Coord := [38, 568] ;33%
  lowMana33Coord := [754, 568] ;33%
  lowPetHealthCoord := [38, 10]
  lowPetManaCoord := [38, 22]
  berserkerChargeCoord := [474, 508]
  embermageChargeCoord := [467, 510]
  engineerChargeCoord := [467, 510]
  outlanderChargeCoord := [478, 516]
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "in game pet portrait")
  getPixelColor(6, 6)
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "skillXCoord")
  loop, 10 {
    skillXObj.insert(skill1Coord[X] + ((A_Index - 1) * 34))
    if (A_Index = 1)
      writeLogFile(INFO, A_ThisFunc, A_LineNumber, "KEY_1234")
    if (A_Index = 5)
      writeLogFile(INFO, A_ThisFunc, A_LineNumber, "KEY_QWER")
    if (A_Index = 9)
      writeLogFile(INFO, A_ThisFunc, A_LineNumber, "KEY_DF")
    getPixelColor(skillXObj[A_Index], skill1Coord[Y])
  }
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "activeskillCoord")
  getPixelColor(activeskillCoord[X], activeskillCoord[Y])
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "lowHealthCoord90")
  getPixelColor(lowHealthCoord90[X], lowHealthCoord90[Y])
  getPixelColor(lowMana90Coord[X], lowMana90Coord[Y])
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "lowHealthCoord66")
  getPixelColor(lowHealth66Coord[X], lowHealth66Coord[Y])
  getPixelColor(lowMana66Coord[X], lowMana66Coord[Y])
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "lowHealthCoord33")
  getPixelColor(lowHealth33Coord[X], lowHealth33Coord[Y])
  getPixelColor(lowMana33Coord[X], lowMana33Coord[Y])
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "lowPetHealthCoord")
  getPixelColor(lowPetHealthCoord[X], lowPetHealthCoord[Y])
  getPixelColor(lowPetManaCoord[X], lowPetManaCoord[Y])
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, "chargeCoord")
  getPixelColor(berserkerChargeCoord[X], berserkerChargeCoord[Y])
  getPixelColor(embermageChargeCoord[X], embermageChargeCoord[Y])
  getPixelColor(engineerChargeCoord[X], engineerChargeCoord[Y])
  getPixelColor(outlanderChargeCoord[X], outlanderChargeCoord[Y])
}

/*
test pixel color inventory
*/
testPixelColorInventory() {
  inventoryGap := [46, 46]
  playerInventoryCoord := [443, 378]
  petInventoryCoord := [26, 379]
  loop, 3 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      mouseX := playerInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := playerInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      getPixelColor(mouseX, mouseY)
    }
  }
  loop, 3 { ;inventory y
    yIndex := A_index + 0 ;to number
    loop, 8 { ;inventory x
      mouseX := petInventoryCoord[X] + ((A_index - 1) * inventoryGap[X])
      mouseY := petInventoryCoord[Y] + ((yIndex - 1) * inventoryGap[Y])
      getPixelColor(mouseX, mouseY)
    }
  }
}

;-------------------------------------------------------------------------------
; file functions
;-------------------------------------------------------------------------------

getTextFile(textFile) {
  FileRead, outText, %textFile%
  message := "FileRead() " (ErrorLevel = 0 ? "SUCCEED" : "FAILED ") " " textFile
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  message := "getTextFile() " outText " (" StrLen(outText) ")"
  writeLogFile(DEBUG, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
  return outText
}

/*
deflag shared stash
*/
deflagSharedStash(targetDirectory) {
  startTime := A_TickCount
  RunWait, %DEFLAGGER_FILE% "%targetDirectory%\sharedstash_v2.bin" , , Hide
  endTime := A_TickCount - startTime
  message := "RunWait() DEFLAGGER_FILE speed:" Round(endTime) "ms"
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
}

/*
copy save file
*/
copySaveFile(sourceDirectory, targetDirectory) {
  FileCopyDir, %sourceDirectory%, % targetDirectory "\save_backup\" A_Now
  message := "FileCopyDir() " (ErrorLevel = 0 ? "succeed" : "failed") " " sourceDirectory ", " targetDirectory "\save_backup\" A_Now
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  showToolTip(INFO, message)
  Sleep, 100
}

/*
copy shared stash
*/
copySharedStash(sourceDirectory, targetDirectory) {
  if !InStr(FileExist(targetDirectory), "D") {
    FileCreateDir, %targetDirectory%
    message := "FileCreateDir() " (ErrorLevel = 0 ? "succeed" : "failed") " " targetDirectory
    writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
    showToolTip(INFO, message)
  }
  sourceFile := sourceDirectory "\sharedstash_v2.bin"
  targetFile := targetDirectory "\sharedstash_v2.bin"
  FileCopy, %sourceFile%, %targetFile%, true
  message := "FileCopy() " (ErrorLevel = 0 ? "succeed" : "failed") " " sourceFile ", " targetFile
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  showToolTip(INFO, message)
  Sleep, 100
}

;-------------------------------------------------------------------------------
; common functions
;-------------------------------------------------------------------------------

getProperties(outText) {
  if errorlevel
    return
  oProperties := StrSplit(outText, "`n")
  loop, % oProperties.MaxIndex() {
    varName := StrSplit(oProperties[A_Index], "=")[1]
    varValue := StrSplit(oProperties[A_Index], "=")[2]
    switch (varName) {
      case "LOGIN_PASSWORD":
        LOGIN_PASSWORD := varValue
      default:
    }
  }
}

/*
run as admin
*/
runAsAdmin() {
  message := "A_IsAdmin " (A_IsAdmin = 1 ? "succeed" : "failed")
  writeLogFile(DEBUG, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
  if A_IsAdmin
    return
  Run *RunAs "%A_ScriptFullPath%"
  ExitApp
}

/*
write log file
*/
writeLogFile(level, functionName, lineNumber, message) {
  if (LOG_LEVEL > level)
    return
  ;write log file
  levelName := ""
  switch (level) {
    case TRACE:
      levelName := "TRACE"
    case DEBUG:
      levelName := "DEBUG"
    case INFO:
      levelName := "INFO"
    case WARN:
      levelName := "WARN"
    case ERROR:
      levelName := "ERROR"
    default:
  }
  FileAppend, % A_YYYY "-" A_MM  "-" A_DD " " A_Hour ":" A_Min ":" A_Sec "." A_MSec " " levelName " " A_ScriptName "." functionName "() Line " lineNumber " " message "`n", %LOG_FILE%
}

/*
remove log file
*/
removeLogFile(thresholdSize) {
  FileGetSize, fileSize, %LOG_FILE%, K
  message := "FileGetSize() " (ErrorLevel = 0 ? "succeed" : "failed") " " fileSize "KB " LOG_FILE
  writeLogFile(DEBUG, A_ThisFunc, A_LineNumber, message)
  if (fileSize < thresholdSize)
    return
  FileDelete, %LOG_FILE%
  message := "FileDelete() " (ErrorLevel = 0 ? "succeed" : "failed") " " LOG_FILE
  writeLogFile(DEBUG, A_ThisFunc, A_LineNumber, message)
  showToolTip(DEBUG, message)
}

/*
show tooltip
*/
showToolTip(level, message) {
  if (LOG_LEVEL > level)
    return
  ToolTip, %message%, 0, 0
  clipboard := message
  SetTimer, removeTooltipTimer, -2500
}

/*
init hotkey
*/
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
  Sleep, 100
  showToolTip(INFO, "initHotkey() " description)
}

/*
press keyboard
*/
pressKey(key, event, delay) {
  switch (event) {
    case KEYDOWN:
      Send, {%key% Down}
      Sleep, 40
    case KEYUP:
      Send, {%key% Up}
      Sleep, 40
    case KEYPRESS:
      Send, {%key% Down}
      Sleep, 40
      Send, {%key% Up}
      if (delay = "")
        return
      Sleep, %delay%
    case KEYRAW:
      SendRaw, %key%
      if (delay = "")
        return
      Sleep, %delay%
    default:
  }
}

/*
press keyboard fast
*/
pressKeyCustom(key, event, pressingTime, delay) {
  switch (event) {
    case KEYDOWN:
      Send, {%key% Down}
      Sleep, %pressingTime%
    case KEYUP:
      Send, {%key% Up}
      Sleep, %pressingTime%
    case KEYPRESS:
      Send, {%key% Down}
      Sleep, %pressingTime%
      Send, {%key% Up}
      if (delay = "")
        return
      Sleep, %delay%
    default:
  }
}

/*
click mouse
*/
clickMouse(coordX, coordY, button, delay) {
  Send, {Click, %coordX%, %coordY%, %button%, Down}
  Sleep, 40
  Send, {Click, %coordX%, %coordY%, %button%, Up}
  if (delay = "")
    return
  Sleep, %delay%
}

/*
click mouse fast
*/
clickMouseCustom(coordX, coordY, button, pressingTime, delay) {
  Send, {Click, %coordX%, %coordY%, %button%, Down}
  Sleep, %pressingTime%
  Send, {Click, %coordX%, %coordY%, %button%, Up}
  if (delay = "")
    return
  Sleep, %delay%
}

/*
move mouse
*/
moveMouse(coordX, coordY, delay) {
  MouseMove, %coordX%, %coordY%
  Sleep, %delay%
}

/*
speak tts
*/
speakTts(message) {
  sapi := ComObjCreate("SAPI.SpVoice")
  sapi.Rate := -1
  sapi.Voice := sapi.GetVoices().Item(1)
  sapi.speak(message)
}

/*
play beep
*/
playBeep(times) {
  loop, % times {
    SoundBeep, 7902, 120
    Sleep, 10
  }
}

/*
formatPercent
*/
formatPercent(numerator, denominator) {
  return Round((numerator / denominator) * 100, 2) "%"
}

/*
format hh:mm:ss
*/
formatHhmmss(ms)
{
  VarSetCapacity(hhmmss,  256), DllCall("GetDurationFormat", "uint", 2048, "uint", 0, "ptr", 0, "int64", ms*10000, "wstr", "d' days 'h':'mm':'ss", "wstr", hhmmss, "int", 256)
  return hhmmss
}

/*
get pixel color
*/
getPixelColor(pixelX, pixelY) {
  if (pixelX = "") || (pixelY = "") {
    MouseGetPos, pixelX, pixelY
  }
  PixelGetColor, color, pixelX, pixelY
  message := "PixelGetColor() " (ErrorLevel = 0 ? "succeed" : "failed") " " pixelX ", " pixelY " " color
  writeLogFile(INFO, A_ThisFunc, A_LineNumber, message)
  showToolTip(INFO, message)
  return color
}
```

[^1]: https://github.com/dntco43u/torchlight2_helper/blob/main/torchlight2_helper.ahk
