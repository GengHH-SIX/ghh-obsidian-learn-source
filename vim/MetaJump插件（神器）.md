
**==Normal 和  Insert 模式下都能使用==**

- 注：*常用于大范围跳转和不规则区域选择*
### 跳转

- the ==Alt+.== shortcut will trigger the metaGo.gotoAfter command, the cursor will be placed after the target character;
- the ==Alt+,== shortcut will trigger the metaGo.gotoBefore command, the cursor will be placed before the target character;
- the **==`Alt+/`==** shortcut will trigger the metaGo.gotoSmart command which intelligently set cursor position after navigation

### 选择

1. type **==`Alt+Shift+/` ==** to tell I want to _select_ to somewhere.
2. type the character(stands for location) on screen, metaGo will show you some codes encoded with character.
3. type the code characters, you will _select_ to that location.
4. repeat 1-3 to adjust your current selection.

### 添加夺光标

1. ==Ctrl+Alt+,== to add another cursor before the target-character.
2. ==Ctrl+Alt+.== to add another cursor after the target-character.
3. **==`Ctrl+Alt+/`==** to add another cursor smartly to the target-character.

    ==Ctrl+u to cancel last cursor action.==

### 删除

1. ==alt+d==: to delete from cursor to the position smartly
2. **==`alt+backspace`==**: to delete from cursor to the position before the target character
3. ==alt+delete==: to delete from cursor to the position after the target character




---

#### 跳转
- 按下“==Alt+.==”快捷键将触发metaGo.gotoAfter命令，光标将被放置在目标字符之后；
- 按下==Alt+,==快捷键将触发metaGo.gotoBefore命令，光标将被放置在目标字符之前；
- 按下==Alt+/==快捷键将触发metaGo.gotoSmart命令，该命令可在导航后智能设置光标位置
#### 选择
1. 按下Alt+Shift+/键，表示我想选择某个位置。
2. 在屏幕上输入字符（代表位置），metaGo会显示一些用该字符编码的代码。
3. 输入代码字符后，您将自动跳转到该位置。
4. 重复步骤1至3，以调整您当前的选择。
#### 添加夺光标
1. 按下Ctrl+Alt+，在目标字符前添加另一个光标。
2. 按下Ctrl+Alt+. 可在目标字符后添加另一个光标。
3. 按下Ctrl+Alt+/，即可巧妙地将另一个光标添加到目标字符处。
    ==Ctrl+u 取消上一次光标操作。==
#### 删除
1. ==alt+d==：智能删除光标到当前位置之间的内容
2. ==alt+backspace==：从光标位置删除到目标字符之前的位置
3. ==alt+delete==：从光标位置删除到目标字符后的位置