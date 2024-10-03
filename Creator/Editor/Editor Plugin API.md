
# EditorPlugin

> Last revision: 2022-11-24

## Plugin Type

1. Trigger plugin: Trigger once via shortcut or menu selection
2. Review plugin: Trigger once when running AI Review
3. Interaction enhancement plugin: Enable or disable through UI button. In the enabled state, the click and drag behavior of the Editor are all forwarded to the plugin

## Data description

### Note

The Note object is returned in the form of NoteID, and the ID is an int value, which is globally unique, but does not guarantee continuity. Other properties of Note can be queried through ID. Note or id mentioned below refer to NoteID.

### Beat

The Editor uses lots of beat values, such as 2:1/4, which means that the second beat with offset by 1/4 beat. The script uses the following structure to represent this value:

```lua
{
    beat = 2,
    numor = 1,
    denom = 4
}
```

This structure is called **beat**


### BPM
```lua
{
    beat = {beat=2, numor=0, denom=4},
    bpm = 160,
    delay = 0
}
```

This structure is called**bpm**

### Effect
```lua
{
    beat = {beat=2, numor=0, denom=4},
    type = "hs",
    value = 4
}
```

This structure is called**effect**

## Script template

```lua
PluginIcon = 'icon.png'  -- Icon used by Type-3 plugin, put the file in the same folder of the lua script, supported from 5.0.2
-- The rest variables are NEEDED
PluginName = 'Name' -- Used for display
PluginMode = 0 -- For which mode, see [Enums - Mode]
PluginType = 0 -- Plugin type, see [Plugin Type]
PluginRequire = '5.0.1' -- The minimal client version needed
-- Type-1&2 plugins use this Entry only
function Run()
end
-- Type-3 plugin use the following Entries
-- Trigger by click
function OnClick()
end
-- Trigger by drag
function OnDragStart()
end
function OnDragMove()
end
function OnDragEnd()
end


-- From 6.0.42
function OnKey(key, down)
end

-- From 6.0.42, called when plugin be selected
function OnActive()
end

-- From 6.0.42, called when selected plugin be disabled
function OnDeactive()
end
```

## API doc

The following interfaces are not specifically specified, and are supported since client side version 5.0.1

### Query API

> Using **Editor:** to Access

| Lua function name | Definition | Remarks |
| --- | --- | --- |
| SelectNotes():array | Get the selected note collection |  |
| GetNoteCount():int | Current total number of Notes |  |
| GetCurrentTime():number | Get the millisecond value of audio time |  |
| GetCurrentBeat():beat | Gets the beat value of audio time |  |
| GetCurrentDivide():int | Get the current beat divide | Returns the denominator, 1/4 returns 4 |
| GetNoteAt(int):id | Get the note id by index | Returns -1 if note is not found |
| GetNoteTime(id,bool):number | Get Note head/tail time in milliseconds | Returns -1 if note is not found |
| GetNoteBeat(id,bool):beat | Get Note head/tail time beat value | Returns all -1 when note is not found |
| GetNoteX(id):int | Get Note's horizontal unit coordinates, tracks, etc., starting from 0 | Returns -1 if note is not found |
| GetNoteType(id):int | Get Note Type | Returns -1 if note is not found |
| GetNoteFlag(id):int | Get the arrow direction and arrow type of Note, see [Enums - NoteFlag] | Returns -1 if note is not found |
| GetNoteWidth(id):int | Get Note width | Returns -1 if note is not found |
| GetNoteSlideBodyCount(id):int | Get the number of body segments of Slide | Returns -1 if note is not found |
| GetNoteSlideBodyBeat(id,int): beat | Gets the beat offset value of the segment relative to the head | Note not found, or returns -1 when the index is out of range |
| GetNoteSlideBodyX(id,int):int | Gets the X offset value of the segment relative to the head | Note not found, or returns -1 when the sequence number is out of range |
| **6.0.72** GetNoteGroup(id):int | Get the group number of note  | |
| GetClickX():int | Get the horizontal unit coordinates at the click position | Point to the left of the track to return -1. Point to the right side of the track Return track number + 1. For example, under 4K editing, return 5 |
| GetClickBeat():beat | Get the beat value at the click position | Returns all -1 when note is not found |
| **6.0.0** GetClickBeatFree():beat | Get the beat value at the click position, align to 1/32 divide | 6.0.0 |
| **6.0.0** ChartInfo(key):string        | Get chart meta                                   | - version:string<br>- creator:string<br>- title:string<br>- artist:string<br>- bpm:string(number)<br>- level:string(number)<br>- key:string(number)，key tracks |

### Review API

> Use **Review:** to Access

| Lua function name | Definition | Remarks |
| --- | --- | --- |
| SetResult(bool,string) | Set the final result to pass, or fail. |  |

### Operating API

> Using **Editor:** to Access

The following APIs all support undo by default.

| Lua function name | Definition | Remarks |
| --- | --- | --- |
| StartBatch() | Start a group of actions, the whole group can be undo together |  |
| FinishBatch() | Close the currently open combination of actions and execute them immediately |  |
| DeleteNote(id) | Delete the specified note |  |
| AddNote(int) | Create an empty note of the specified type | After note is created, type cannot be changed |
| SetNoteBeat(id,beat,bool) | Set the beat value of the head and tail of the note | Setting millisecond value is not supported |
| SetNoteX(id,int) | Set the horizontal unit coordinates of note |  |
| SetNoteFlag(id,int) | Set arrow direction, arrow type |  |
| SetNoteWidth(id,int) | Set note width |  |
| AddNoteSlideBody(id,beat) | Append a segment to the end of the body part of a Slide |  |
| SetNoteSlideBodyX(id,int,int) | Set the X offset of the Slide segment |  |
| DeleteNoteSlideBody(id) | Clear the segments of Slide |  |
| **6.0.0** SetNoteGroup(id,int) | Set the group number of note |  |
| **6.0.72** SelectNote(id)  | Select note | |

### Timing API

> Using **Editor:** to Access

| Lua function name            | Definition             | Remarks                    |
|-----------------------------|------------------------------------|------------------------------|
| **6.0.62** AddTime(beat, val) | Add BPM point | |
| **6.0.62** DeleteTime(beat) | Remove BPM at giving beat | |
| **6.0.62** AddEffect(beat, type, val) | Add Effect point<br>- type: string，sv /hs / jump | |
| **6.0.62** DeleteEffect(beat, type) | Remove Effect at giving beat and type | |
| **6.0.72** GetTimeCount():int | Current total number of BPMs | |
| **6.0.72** GetEffectCount():int | Current total number of Effects | | 
| **6.0.72** GetTimeAt(int):bpm | Get the BPM by index | |
| **6.0.72** GetEffectAt(int):effect | Get the effect by index | |

### Graphic API

> Using **Editor:** to Access

| Lua function name                  | Definition                          | Remarks                          |
|------------------------------|----------------------------------|------------------------------|
| **6.0.0** AddSprite(name,file,parent):mod | Add Sprite to editing area, return a Editor-Specific Module   | - name：string,name for the module, Required<br>- file：string,image filename，Required<br>- parent：string,parent name of the module，Optional |
| **6.0.0** AddText(name,content):mod | Add Text to editing area, return a Editor-Specific Module   |                              |
| **6.0.0** RemoveModule(name)      | Remove module by names                       |                              |
| **6.0.42** AddSpriteUndo(name,file,parent) | Same to AddSprite, but with Undo support | |
| **6.0.42** AddTextUndo(name,content) | Same to AddText, but with Undo support | |
| **6.0.42** RemoveModuleUndo(name)      | Same to RemoveModule, but with Undo support | |
| **6.0.42** FindModule(name):mod | Find module by name | | 


### Auxiliary API

> Using **Editor:** to Access

| Lua function name | Definition | Remarks |
| --- | --- | --- |
| ShowMessage(string) | Show a short message |  |
| **5.2.6** MakeBeat(int,int,int): beat | Create a beat struct |  |
| BeatAdd(beat,beat):beat | Add two beat |  |
| BeatMinus(beat,beat):beat | Subtraction of two beat |  |
| Beat(Beat) | Scroll to the specified beat |  |
| SeekTo(number) | Scroll to the specified milliseconds |  |
| **5.4.32** CallShortcut(int) | Call internal function, see enums below |  |
| **5.4.72** GetUserInput(default, function) | Get input text from user | default: string, default text<br>function: function(string) callback |
| **6.0.52** GetUserInput(title, default, function) | Get input text from user |  |
| **6.0.0** ReadFile(name):string | Read files in chart folder |  |
| **6.0.0** WriteFile(name,string) | Write files with given name to chart folder |  |
| **6.0.22** ReadData(key, psw):string | Read cross-plugin shared data, use the same psw value if share data within a group of plugins | |
| **6.0.22** WriteData(key, psw, data) | Write value to cross-plugin shared data | |
| **6.0.42** ReadBytes(name):array   | Read binary content from files in chart folder            |                              |

## Enums

### NoteType

* Key/Catch/Pad/Slide/Cubemode Normal type = 1
* Catchmode Rain type = 8
* Key/Pad/Ring/CubeMode Hold Type = 128
* Wipe type for Slide/Live/Cube mode = 1024
* Slide type for Slide/Live mode = 2048
* LiveMode Flick Type = 16384

### Mode

* Key = 0
* Catch = 3
* Pad = 4
* Taiko = 5
* Ring = 6
* Slide = 7
* Live = 8
* Cube = 9

### NoteFlag

* No direction = 0
* Flick tag = 1
* Right = 2
* Up = 4
* Left = 8
* Down = 16
* Own direction = 32

### Internal Functions

* Speed up playrate = 7
* Slow down playrate = 8
* Increase grid scale = 39
* Decrease grid scale = 40

## Others

### Editor-Specific Module
Editor-Specific Module is inhertied from Module without animation support, see [Skin-Module](../Skin/Skin%20Script%20API%20Basic.md), The differences are below:

| Param         | Definition                            | Remarks      |
| ---------------- | --------------------------------------- | ------------ |
| X                | Percent value of X                      |              |
| Y                | Invalid                 |              |
| Beat             | Beat value of Y
| Width            | Percent value of Width                       |              |
| Height           | Unit value of Height                        |              |
| HeightBeat       | Beat value of Height

## Demo

### Trigger plugin

```lua
-- Delete selected notes
function Run()
    local notes = Editor:GetSelectNotes()
    Editor:StartBatch()
    for i=0,notes.Length-1 do
        Editor:DeleteNote(notes[i])
    end
    Editor:FinishBatch()
end

-- Move selected note one beat later
function Run()
    local notes = Editor:GetSelectNotes()
    local nid = notes[0]
    local beat = Editor:GetNoteBeat(nid)
    beat.beat = beat.beat + 1
    Editor:SetNoteBeat(nid, beat)
end

-- Add a slide with 3 segments
function Run()
    Editor:StartBatch()
    local nid = Editor:AddNote(2048)
    Editor:SetNoteX(nid, 100)
    Editor:SetNoteWidth(nid, 100)
    local beat = {beat=2, numor=0, denom=4}
    Editor:SetNoteBeat(nid, beat, true)
    beat.numor = 1
    Editor:AddNoteSlideBody(nid, beat)
    Editor:SetNoteSlideBodyX(nid, 0, -20)
    beat.numor = 2
    Editor:AddNoteSlideBody(nid, beat)
    Editor:SetNoteSlideBodyX(nid, 1, 20)
    beat.numor = 3
    Editor:AddNoteSlideBody(nid, beat)
    Editor:SetNoteSlideBodyX(nid, 2, -30)
    Editor:FinishBatch()
end
```

### Review plugin

```lua
PluginName = "Overmap"
PluginMode = 0
PluginType = 1
PluginRequire = '5.0.1'

function Run()
    local count = Editor:GetNoteCount()
    if count > 100 then
        Review:SetResult(false, "Overmap!")
    end
    -- Return nothing if no error
end
```

### Interactive class plugin

```lua
-- Add a note at click point
function OnClick()
    local beat = Editor:GetCurrentBeat()
    local x = Editor:GetClickX()
    if x < 0 then
        return
    end
    Editor:StartBatch()
    local nid = Editor:CreateNote(1)
    Editor:SetNoteX(nid, x)
    Editor:SetNoteBeat(nid, beat, true)
    Editor:FinishBatch()
end
```

```lua
PluginName = 'Add Pattern'
PluginMode = 0
PluginType = 2
PluginRequire = '5.0.1'

function OnClick()
    local x = Editor:GetClickX()
    local beat = Editor:GetClickBeat()
    local div = Editor:GetCurrentDivide()
    Editor:StartBatch()
    local nid = Editor:AddNote(1)
    Editor:SetNoteX(nid, x)
    Editor:SetNoteBeat(nid, beat, true)
    for i=x+1,3 do
        nid = Editor:AddNote(1)
        Editor:SetNoteX(nid, i)
        local delta = {beat =0, numor = 1, denom=div}
        beat = Editor:BeatAdd(beat, delta)
        Editor:SetNoteBeat(nid, beat, true)
    end
    Editor:FinishBatch()
end
```