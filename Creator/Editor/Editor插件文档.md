# Editor插件设计
> 📍 最后修订：2023-11-24


## 插件分类

1. 触发类插件：通过快捷键或菜单选择触发⼀次
2. 审核类插件：运⾏AIReview时触发⼀次
3. 交互增强类插件：通过⼊⼝按钮开启或关闭，开启状态下，Editor的点击，拖拽⾏为全部转发给插件

## 插件⼊⼝

1. 触发类，收纳在更多功能的菜单列表⾥
2. 审核类没有⼊⼝，在执⾏AIReview时⾃动调⽤
3. 在Editor交互⾯板上有⼀个多态切换按钮，⽤⼾点击后弹出所有来⾃插件的可选交互能⼒。可⾃定义指定⼀个展⽰Icon（5.0.2起⽀持）

## 数据说明

### Note

其中Note对象以NoteID的形式返回，ID是int型，全局唯⼀，但不保证连续。通过ID可以查询Note的其他属性。

下⽂提到的Note或者id，均指NoteID。

### 节拍

Editor会⽤到⼤量的节拍值，⽐如2:1/4,意思是第⼆拍基础上偏移1/4拍。脚本使⽤如下的结构代表这个值：

```lua
{
    beat = 2,
    numor = 1,
    denom = 4
}
```

下⽂将这个结构成为**beat**

## 脚本模板

```lua
PluginIcon = 'icon.png'  -- 3类插件⽤的图标⽂件，需要放在lua同⽬录下, 5.0.2起⽀持
-- 以下都是必选字段
PluginName = '插件名称' -- ⽤于显⽰的插件名称
PluginMode = 0 -- 适⽤的游玩模式，参⻅【各类枚举值-Mode】
PluginType = 0 -- 插件类型，参⻅【插件分类】
PluginRequire = '5.0.1' -- 插件需要的客⼾端最低版本

-- 1&2类插件只实现这个接⼝
function Run()

end

-- 3类插件实现以下两组接⼝⼆选⼀，或者全部
-- 第⼀组，通过点击触发
function OnClick()

end

-- 第⼆组，通过拖动触发
-- 拖动开始
function OnDragStart()

end

-- 拖动中
function OnDragMove()

end

-- 拖动结束
function OnDragEnd()

end

-- 6.0.42起支持
function OnKey(key, down)
end

-- 6.0.42起支持，插件被启动时触发
function OnActive()
end

-- 6.0.42起支持，激活状态的插件被禁用时触发
function OnDeactive()
end
```

## 接⼝⽂档

以下接⼝没有特别指定的，都是从客⼾端5.0.1版本开始⽀持

### 查询接⼝

> 使⽤**Editor:**访问

| lua函数名                     | 定义                                       | 备注                              |
|------------------------------|-------------------------------------------|----------------------------------|
| GetSelectNotes():array       | 获取当前选中的note集合                           |                                  |
| GetNoteCount():int           | 当前Note总数                                 |                                  |
| GetCurrentTime():number      | 获取当前时刻的毫秒值                               |                                  |
| GetCurrentBeat():beat        | 获取当前时刻的节拍值                               |                                  |
| GetCurrentDivide():int       | 获取当前的节拍分度                                | 返回分⺟值，1/4返回4                    |
| GetNoteAt(int):id            | 获取按序号在第int位的noteid                        | 找不到note时返回-1                    |
| GetNoteTime(id,head):number  | 获取Note头部/尾部时间毫秒值                          | 找不到note时返回-1<br>head:bool,返回头部或尾部的值 |
| GetNoteBeat(id,head):beat    | 获取Note头部/尾部时间节拍值                          | 找不到note时返回全-1<br>head:bool,返回头部或尾部的值 |
| GetNoteX(id):int             | 获取Note的横向单位坐标，轨道等，从0开始                    | 找不到note时返回-1                    |
| GetNoteType(id):int          | 获取Note类型                                 | 找不到note时返回-1                    |
| GetNoteFlag(id):int          | 获取Note的箭头⽅向，箭头类型，参⻅【各<br>类枚举值-NoteFlag】   | 找不到note时返回-1                    |
| GetNoteWidth(id):int         | 获取Note宽度                                 | 找不到note时返回-1                    |
| GetNoteSlideBodyCount(id):int | 获得Slide类型Note的⾝体⽚段个数                      | 找不到note时返回-1                    |
| GetNoteSlideBodyBeat(id,int):beat | 获得Slide类型Note的⾝体部分指定⽚段的相<br>对头部的节拍偏移值     | 找不到note,或序号超出范围时返回-1         |
| GetNoteSlideBodyX(id,int):int | 获得Slide类型Note的⾝体部分指定⽚段的相<br>对头部的X偏移值     | 找不到note,或序号超出范围<br>时返回-1        |
| GetClickX():int              | 获得点击处的横向单位坐标                             | 点到轨道左侧返回-1<br>点到轨道右侧返回轨道数+1<br>⽐如在4K编辑下，返回5 |
| GetClickBeat():beat          | 获得点击处的节拍值,对⻬到当前分度                        |                                  |
| **6.0.0** GetClickBeatFree():beat | 获得点击处的节拍值,对⻬到1/32分度                      | 6.0.0                            |
| **6.0.0** ChartInfo(key):string        | 获取谱⾯信息                                   | 参数不区分⼤⼩写<br>- version:string<br>- creator:string<br>- title:string<br>- artist:string<br>- bpm:string(number)<br>- level:string(number)<br>- key:string(number)，轨道数 |

### 审核接⼝

> 使⽤**Review:**访问

| lua函数名               | 定义                  | 备注             |
|----------------------|---------------------|----------------|
| SetResult(bool,string) | 设置最终结果通过，或失败。      | 设置失败是显⽰的失败原因   |

### 操作接⼝

> 使⽤**Editor:**访问

以下接⼝都默认⽀持撤销，⽆需额外实现。

但由于接⼝⽀持的能⼒被分拆的⾮常原⼦化，⽽Editor默认的撤销步数有限，所以建议按需使⽤操作组合，将多个操作聚合在⼀起执⾏。

| lua函数名                      | 定义                               | 备注                          |
|-----------------------------|------------------------------------|------------------------------|
| StartBatch()                | 开始⼀个操作组合，整个组合可以⼀起撤销               |                              |
| FinishBatch()               | 关闭⽬前打开的操作组合，并⽴刻执⾏                  |                              |
| DeleteNote(id)              | 删除指定note                          |                              |
| AddNote(type):id            | 创建⼀个指定类型的空note                     | note创建后，type不可改<br>type:int,Note类型 |
| SetNoteBeat(id,beat,bool)   | 设置note的头部、尾部节拍值                    | 不⽀持设置毫秒值                    |
| SetNoteX(id,int)            | 设置note的横向单位坐标                      |                              |
| SetNoteFlag(id,int)         | 设置箭头⽅向，箭头类型                        |                              |
| SetNoteWidth(id,int)        | 设置note宽度                           |                              |
| AddNoteSlideBody(id,beat)   | 向Slide类型Note⾝体部分尾部追加⼀个⽚<br>段，三个int为相对头部的节拍值 |                              |
| SetNoteSlideBodyX(id,int,int) | 设置Slide类型Note⾝体指定⽚段的X偏移            |                              |
| DeleteNoteSlideBody(id)     | 清空Slide类Note的⾝体数据                  |                              |
| **6.0.0** SetNoteGroup(id,int)   | 设置Note的分组编号                        |                              |
| **6.0.72** SelectNote(id)  | 选中note | |

### 时间点接口

> 使⽤**Editor:**访问

| lua函数名                      | 定义                               | 备注                          |
|-----------------------------|------------------------------------|------------------------------|
| **6.0.62** AddTime(beat, val) | 添加BPM时间点 | |
| **6.0.62** DeleteTime(beat) | 删除指定位置的时间点 | |
| **6.0.62** AddEffect(beat, type, val) | 添加效果点<br>- type: string，取值sv，hs或jump | |
| **6.0.62** DeleteEffect(beat, type) | 删除指定位置和类型的效果点 | |

### 图形接⼝

> 使⽤**Editor:**访问

| lua函数名                       | 定义                              | 备注                           |
|------------------------------|----------------------------------|------------------------------|
| **6.0.0** AddSprite(name,file,parent):mod | 向编辑区添加⾃定义图像，返回Editor定制Module<br>对象。参见后文定制Module一节说明。   | - name：string,要添加的图像名，必填<br>- file：string,要添加的图像⽂件名，必填<br>- parent：string,可以指定添加的⽗级，可选 |
| **6.0.0** AddText(name,content):mod | 向编辑区添加⾃定义⽂本，<br>返回Editor定制Module<br>对象。参见后文定制Module一节说明。  |                              |
| **6.0.0** RemoveModule(name)      | 删除⼀个⾃定义对象                       |                              |
| **6.0.42** AddSpriteUndo(name,file,parent) | 功能同AddSprite，但支持撤销 | |
| **6.0.42** AddTextUndo(name,content) | 功能同AddText，但支持撤销 | |
| **6.0.42** RemoveModuleUndo(name)      | 功能同RemoveModule，但支持撤销 | |
| **6.0.42** FindModule(name):mod | 根据name查找对象 | | 

### 辅助接⼝
> 使⽤**Editor:**访问

| lua函数名                       | 定义                            | 备注                          |
|------------------------------|-------------------------------|------------------------------|
| ShowMessage(string)          | 显⽰⼀条短消息                       |                              |
| **5.2.6** MakeBeat(int,int,int):beat | 创建⼀个beat结构                    |                              |
| BeatAdd(beat,beat):beat      | 两个beat结构相加                    |                              |
| BeatMinus(beat,beat):beat    | 两个beat结构相减                    |                              |
| SeekTo(beat)                 | 滚动到指定节拍                       |                              |
| SeekTo(number)               | 滚动到指定毫秒                       |                              |
| **5.4.32** CallShortcut(int)      | 调⽤内置功能，对应关系⻅枚举定义              |                              |
| **5.4.72** GetUserInput(default, function) | 获取⼀个⽤⼾输⼊                      | default:string,默认⽂字<br>function:function(string)类型回调 |
| **6.0.52** GetUserInput(title, default, function) | 获取⼀个⽤⼾输⼊ |  |
| **6.0.0** ReadFile(name):string   | 读取谱⾯⽬录下指定⽂件                  |                              |
| **6.0.0** WriteFile(name,string)  | 写⼊内容到⽬录下指定⽂件                 |                              |
| **6.0.22** ReadData(key, psw):string | 读取一个全局共享数据，不同插件访问相同数据时使用相同psw值 | |
| **6.0.22** WriteData(key, psw, data) | 写入一个全局共享数据 | |
| **6.0.42** ReadBytes(name):array   | 读取谱⾯⽬录下指定⽂件的二进制内容                  |                              |

## 各类枚举值

### Note类型

- Key/Catch/Pad/Slide/Cube模式普通类型=1
- Catch模式Rain类型=8
- Key/Pad/Ring/Cube模式的Hold类型=128
- Slide/Live/Cube模式的Wipe类型=1024
- Slide/Live模式的Slide类型=2048
- Live模式Flick类型=16384

### Mode

- Key=0
- Catch=3
- Pad=4
- Taiko=5
- Ring=6
- Slide=7
- Live=8
- Cube=9

### NoteFlag

- ⽆⽅向=0
- Flick标记=1
- 向右=2
- 向上=4
- 向左=8
- 向下=16
- ⾃有⽅向=32

### 内置功能序号

- 增加播放速度=7
- 降低播放速度=8
- 增加缩放=39
- 降低缩放=40

## 其他说明

### 定制Module对象
定制对象继承通用Module的基础属性，参见[皮肤文档-Module](../Skin/皮肤脚本文档-基础.md)，但不支持动画。与通用Module的区别如下：

| 属性名           | 定义                                    | 备注         |
| ---------------- | --------------------------------------- | ------------ |
| X                | X 坐标的百分比值                       |              |
| Y                | 无效                          |              |
| Beat             | Y方向的Beat值
| Width            | 宽度的百分比值                        |              |
| Height           | 高度的单位值                            |              |
| HeightBeat       | 高度的Beat值

## 主要API调⽤演⽰

### 触发类插件

```lua
-- 删除选中的notes
function Run()
    local notes = Editor:GetSelectNotes()
    Editor:StartBatch()
    for i=0,notes.Length-1 do
        Editor:DeleteNote(notes[i])
    end
    Editor:FinishBatch()
end

-- 将选择的note往后移⼀拍
function Run()
    local notes = Editor:GetSelectNotes()
    local nid = notes[0]
    local beat = Editor:GetNoteBeat(nid)
    beat.beat = beat.beat + 1
    Editor:SetNoteBeat(nid, beat)
end

-- 画⼀条由3个⽚段组成的slide
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
end
```