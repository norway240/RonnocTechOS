
-- usage:
--  taco
--  taco filename

local tArgs = {...}
local betaKey = ""
local currentFile = ""
local isSaved = true
local isChanged = true
local fileLinesArr = {""}
local curLine,curCol,lastCol=1,0,0
local menus = {}
local inMenu=false
local selectedMenu = 1
local selectedSubMenu = 1
local running = true
local config = {isFullScreen = false, clipboard=""}
local showFirstTimeMessage = false
local showAboutBox=false
local collectedInputs = {file="My Input Data"}
local collectedInputsCursor = {file=1}
local collectedInputsOffsets = {file=0}
local collectedInputsSize = {file=0}
local w,h = term.getSize()
local scrollOffsetX,scrollOffsetY = 0,0
local isBlinking = false
local state = "edit"
local message = {}
local showingMessage=false
local showingFileDialog = false
local selectedMessageOption=1
local lastBlinkTime = 0
sleep(0.1)
local timer = os.startTimer(0)

local function setColors(monitor, bg, fg, bg2, fg2)
   if monitor then
    if monitor.isColor() then
      monitor.setBackgroundColor(bg)
      monitor.setTextColor(fg)
    else
      monitor.setBackgroundColor(bg2)
      monitor.setTextColor(fg2)
    end
  end
end

local function checkLinePosition()
    local maxLines = #fileLinesArr
    if curLine < 1 then
        curLine = 1
    elseif curLine > #fileLinesArr then
        curLine = maxLines
    end
    local yOffset = 2
    if config.isFullScreen then
      yOffset = 0
    end
    if scrollOffsetY > h-yOffset-curLine then
        scrollOffsetY=h-yOffset-curLine
    elseif scrollOffsetY < 1-curLine then
        scrollOffsetY=1-curLine
    end
end
local function checkColPosition()
    if curCol < 0 then
      curCol = 0
    elseif curCol > string.len(fileLinesArr[curLine]) then
      curCol = string.len(fileLinesArr[curLine])
    end
    local total = h-3 --todo: possible bug
    if #fileLinesArr < total then
        total = #fileLinesArr
    end
    local padWidth = string.len(tostring(total))
    local workingWidth = w - padWidth - 1
    
    if scrollOffsetX >= workingWidth - curCol-1 then
        scrollOffsetX=workingWidth-curCol-1
    elseif scrollOffsetX < 0-curCol then
        scrollOffsetX=0-curCol
    end
end
local function checkMenuPositions()
  if selectedMenu < 1 then
    selectedMenu = #menus
    elseif selectedMenu > #menus then
    	selectedMenu = 1
	end
  if selectedSubMenu < 1 then
    selectedSubMenu = #menus[selectedMenu].items
	elseif selectedSubMenu > #menus[selectedMenu].items then
		selectedSubMenu = 1
	end
end
local function checkMessageOptionPositions()
  if selectedMessageOption < 1 then
    selectedMessageOption = #message.footer
	elseif selectedMessageOption > #message.footer then
		selectedMessageOption = 1
	end
end
local function loadFile(file)
    if fs.exists(file) and not fs.isDir(file) then
        local tmpArr = {}
        local h = fs.open(file, "r")
        while true do
            local data = h.readLine()
            if data == nil then break end
            table.insert(tmpArr, data)
        end
        h.close()
        return tmpArr
    end
    return null
end
local function openFile(file)
  local data = loadFile(file)
  if data then
		fileLinesArr = {""}
		currentFile = file
		isSaved = false
		isChanged = false
		scrollOffsetX,scrollOffsetY=0,0
		if data then
			fileLinesArr = data
			isSaved = true
		end
		curLine,curCol,lastCol=1,0,0
		checkLinePosition()
		checkColPosition()
		return true
	end
	return false
end
local function saveFile(file, dataArr)
    local tmpArr = {}
    local h = fs.open(file, "w")
    if h then
			for i=1, #dataArr do
					h.writeLine(dataArr[i])
			end
			h.close()
			isSaved = true
			isChanged = false
			return true
		end
    return false
end
local function loadSettings()
    if fs.exists("taco.db") then
        local cfg
        local h = fs.open("taco.db", "r")
        while true do
            data = h.readLine()
            if data == nil then break end
            if data == "--[[CONFIG:" then
                local tmpCfg = h.readAll()
                cfg=textutils.unserialize(string.sub(tmpCfg,1,string.len(tmpCfg)-2))
            end
        end
        h.close()
        if cfg then
            if cfg.isFullScreen then
                config.isFullScreen = cfg.isFullScreen
            end
            if cfg.clipboard then
                config.clipboard = cfg.clipboard
            end
        end
    else
        showFirstTimeMessage = true
    end
end
local function saveSettings()
    settingsStr=textutils.serialize(config)
    local h = fs.open("taco.db", "w")
    h.writeLine('print("This is a settings file for TACO :)")')
    h.writeLine('--[[CONFIG:')
    h.writeLine(settingsStr)
    h.writeLine(']]')
    h.close()
end
local function newFile()
    currentFile = ""
    fileLinesArr={""}
		curLine,curCol,lastCol=1,0,0
		checkLinePosition()
		checkColPosition()
    isSaved = false
    isChanged = false
end
    
table.insert(menus, {
                        name="File",
                        hotkey=keys.f,
                        items={
                                {"New", "N", "new_file", true},
                                {"Open..", "O", "open_file", true},
                                {"Save", "S", "save_file", false},
                                {"Save As..", "A", "saveas_file", false},
                                {"--"},
                                {"Revert", "R", "revert_file", false},
                                --{"Print..", "P", "print_file", false},
                                --{"Run Script", "E", "run_file", false},
                                {"--"},
                                {"Exit", "X", "exit_app", true}
                            }
                    })
table.insert(menus, {
                        name="Edit",
                        hotkey=keys.e,
                        items={
                                --{"Cut", "X", "cut_selection", false},
                                --{"Copy", "C", "copy_selection", false},
                                --{"Paste", "V", "paste_selection", false},
                                --{"--"},
                                --{"Delete", "D", "clear_selection", false},
                                --{"--"},
                                {"Cut Line", "K", "cut_line", false},
                                {"Copy Line", "J", "copy_line", false},
                                {"Paste Line", "U", "paste_line", false},
                                {"--"},
                                {"Delete Line", "R", "delete_line", false},
                                {"--"},
                                {"Clone Line", "R", "clone_line", false},
                            }
                    })
                    --[[
table.insert(menus, {
                        name="Search",
                        hotkey=keys.s,
                        items={
                                {"Find..", "F", "find_text", false},
                                {"Replace..", "R", "replace_text", false}
                            }
                    })]]
table.insert(menus, {
                        name="Options",
                        hotkey=keys.o,
                        items={
                                {"Full Screen Mode", "F", "fullscreen_toggle", false},
                                --{"Settings..", "S", "show_settings", false}
                            }
                    })
table.insert(menus, {
                        name="Help",
                        hotkey=keys.h,
                        items={
                                --{"Help", "H", "show_help", true},
                                {"Grab Updates", "U", "perform_update", true},
                                {"--"},
                                {"About TACO", "A", "show_about", true},
                            }
                    })
local function lpad(str, len, char)
    if char == nil then char = ' ' end
    local i = len - #str
    if i >= 0 then
      return string.rep(char, i) .. str
    else
      return str
    end
end
local function rpad(str, len, char)
    if char == nil then char = ' ' end
    local i = len - #str
    if i >= 0 then
      return str .. string.rep(char, i)
    else
      return str
    end
end
local function drawScreen(monitor)
    w,h = monitor.getSize()
    monitor.setBackgroundColor(colors.black)
    monitor.clear()
    --header bar
    local guiOffset = 1
    local lineTotalOffset = 2
    if config.isFullScreen then
        guiOffset = 0
        lineTotalOffset = 0
    end
    --editor area
    local ii=1
    local total = h-lineTotalOffset-scrollOffsetY+curLine
    if #fileLinesArr < total then
        total = #fileLinesArr
    end
    local padWidth = string.len(tostring(total))
    for i=1+guiOffset, h-(guiOffset/2) do
        local currentLineIndex = ii-scrollOffsetY
        local workingWidth = w - padWidth - 1
        if 1==2 then
          workingWidth=workingWidth-1 --scrollbar
        end
        monitor.setCursorPos(1,i)
        setColors(monitor, colors.blue, colors.lightGray, colors.black, colors.white)
        monitor.write(string.rep(" ", w)) --subtract the two "+"'s
        if currentLineIndex <= #fileLinesArr then
            local currentLine = fileLinesArr[currentLineIndex]
            setColors(monitor, colors.white, colors.red, colors.white, colors.black)
            monitor.setCursorPos(1,i)
            monitor.write(lpad(tostring(currentLineIndex), padWidth, " "))
            if curLine == currentLineIndex then
                setColors(monitor, colors.lightBlue, colors.white, colors.black, colors.white)
            else
                setColors(monitor, colors.blue, colors.lightGray, colors.black, colors.white)
            end
            monitor.setCursorPos(padWidth+1,ii+guiOffset)
            currentLineFit = currentLine
            if currentLineFit then
                currentLineFit = string.sub(currentLineFit, -scrollOffsetX+1, string.len(currentLineFit))
                if string.len(currentLineFit) > workingWidth then
                    currentLineFit = string.sub(currentLineFit, 1, workingWidth)
                end
                monitor.write(" "..rpad(currentLineFit, workingWidth, " ").." ")
            end
            if curLine == currentLineIndex and (isBlinking or inMenu) then
                monitor.setCursorPos(padWidth+2+curCol+scrollOffsetX,i)
                setColors(monitor, colors.white, colors.lightBlue, colors.white, colors.black)
                local msg = string.sub(currentLineFit,curCol+scrollOffsetX+1,curCol+scrollOffsetX+1)
                if msg == "" then msg = " " end
                monitor.write(msg)
            end
        end
        ii=ii+1
    end
    
    if not config.isFullScreen or inMenu then
        --footer bar
        setColors(monitor, colors.lightGray, colors.black, colors.white, colors.black)
        monitor.setCursorPos(1,h)
        monitor.write(string.rep(" ", w))
        
        monitor.setCursorPos(1,h)
        if not isSaved or isChanged then
					setColors(monitor, colors.lightGray, colors.red, colors.white, colors.black)
          monitor.write("*")
				end
        if currentFile == "" then
					setColors(monitor, colors.lightGray, colors.green, colors.white, colors.black)
          monitor.write("new_file")
        else
          monitor.write(currentFile)
        end
        
        setColors(monitor, colors.lightGray, colors.black, colors.white, colors.black)
        local footerMsg = "Ln "..curLine..":"..#fileLinesArr.."  Col "..(curCol+1) --.." XOff="..scrollOffsetX
        monitor.setCursorPos(w-string.len(footerMsg)+1,h)
        monitor.write(footerMsg)
    
        --header bar
        monitor.setCursorPos(1,1)
        setColors(monitor, colors.lightGray, colors.black, colors.white, colors.black)
        monitor.write(string.rep(" ", w))
        monitor.setCursorPos(2,1)
        for i=1, #menus do
            if inMenu and i == selectedMenu then
                setColors(monitor, colors.black, colors.white, colors.black, colors.white)
            else
                setColors(monitor, colors.lightGray, colors.black, colors.white, colors.black)
            end
            monitor.write(" "..menus[i].name.." ")
        end
        
        -- Time
        local time = os.time()
        local timeFmt = textutils.formatTime(time, false)
        local timeLen = string.len(timeFmt)
        monitor.setCursorPos(w-timeLen,1)
        setColors(monitor, colors.lightGray, colors.black, colors.white, colors.black)
        monitor.write(timeFmt)
    end

    if inMenu then
        monitor.setCursorPos(1,1)
        setColors(monitor, colors.lightGray, colors.black, colors.white, colors.black)
        local menuWidth = 1
        local menuOffset = 0
        for i=1, selectedMenu-1 do
            menuOffset=menuOffset+string.len(menus[i].name)+2
        end
        for i=1, #menus[selectedMenu].items do
            local len = string.len(menus[selectedMenu].items[i][1])+2
            if len > menuWidth then
                menuWidth = len
            end
        end
        for i=1, #menus[selectedMenu].items do
            if i == selectedSubMenu then
                setColors(monitor, colors.black, colors.white, colors.black, colors.white)
            else
                setColors(monitor, colors.lightGray, colors.black, colors.white, colors.black)
            end
            monitor.setCursorPos(menuOffset+3,1+i)
            monitor.write(string.rep(" ", menuWidth))
            monitor.setCursorPos(menuOffset+3,1+i)
            local str = menus[selectedMenu].items[i][1]
            if str == "--" then
                if monitor.isColor() then
                  monitor.setTextColor(colors.gray)
                else
                  monitor.setTextColor(colors.black)
                end
                monitor.write(string.rep("-", menuWidth))
            else
                monitor.write(" "..str.." ")
            end
        end
    end
    
end

function updateAllTheThings()
    shell.run("market get gjdh1m taco "..betaKey.." y")
end
local function resetBlink()
    lastBlinkTime = os.clock()+.5
    isBlinking = true
end
local function centerText(monitor, tY, tText)
    local offX = (w+1)/2 - (string.len(tText)-1)/2
    monitor.setCursorPos(offX, tY)
    monitor.write(tText)
end
local function centerTextWidth(monitor, tX, tY, width, tText)
    local offX = (width+1)/2 - (string.len(tText)-1)/2
    monitor.setCursorPos(offX+tX, tY)
    monitor.write(tText)
end
local logo = {
"     1111111111  ";
"  11115e454e5e11 ";
" 11545e5111111111";
"1154e411111111111";
"1e455111111111111";
"15ec5111111111111";
"1cccc1111111111  ";
"1cccc111111      ";
" 111111          ";
}
--[[Stolen Shamelessly from nPaintPro - http://pastebin.com/4QmTuJGU]]
local function getColourOf(hex)
  local value = tonumber(hex, 16)
  if not value then return nil end
  value = math.pow(2,value)
  return value
end
local function drawPictureTable(mon, image, xinit, yinit, alpha)
  if not alpha then alpha = 1 end
  for y=1,#image do
    for x=1,#image[y] do
      mon.setCursorPos(xinit + x-1, yinit + y-1)
      local col = getColourOf(string.sub(image[y], x, x))
      if not col then col = alpha end
      if term.isColor() then
        mon.setBackgroundColour(col)
      else
        local pixel = string.sub(image[y], x, x)
        if pixel == "c" or pixel == "5" or pixel == " " then
          mon.setBackgroundColour(colors.white)
        else
          mon.setBackgroundColour(colors.black)
        end
      end
      mon.write(" ")
		end
	end
end
--[[End Theft]]
local function trySaveFileFromDialog()
  currentFile = collectedInputs.file
	if saveFile(currentFile, fileLinesArr) then
		showingFileDialog=false
  else
		messageBox("Error!", {"Couldn't save file!", "Try another name.."}, {{"OK",null}})
  end
end
local function tryOpenFileFromDialog()
  if openFile(collectedInputs.file) then
		showingFileDialog=false
  else
		messageBox("Error!", {"Couldn't open file!"}, {{"OK",null}})
  end
end
local function hideFileDialog()
  showingFileDialog=false
end
local fileDialog={}
fileDialog.title="Open File"
fileDialog.basePath="/"
fileDialog.currentTab=1
fileDialog.currentTabMax=4
fileDialog.dirScrollOffset=0
fileDialog.fileScrollOffset=0
fileDialog.dirSelected=1
fileDialog.fileSelected=1
fileDialog.currentPathFiles={}
fileDialog.currentPathFolders={}
fileDialog.selectedFooterOption=1
fileDialog.footer={}
local function addSlashIfNeeded(path)
  local len = string.len(path)
  if string.sub(path, len, len) ~= "/" then
    return path.."/"
  else
    return path
  end
end
local function addSlashIfNeededLeft(path)
  if string.sub(path, 1,1) ~= "/" then
    return "/"..path
  else
    return path
  end
end
local function getFilePath(file)
    local fName = fs.getName(file)
    local lenBpath = string.len(file)
    local lenFname = string.len(fName)
    return string.sub(file, 1, lenBpath - lenFname-1)
end
local function prepareDialog()
  local tmpList = fs.list(fileDialog.basePath)
  local tmpFileArr,tmpFolderArr={},{}
  fileDialog.basePath = addSlashIfNeeded(fileDialog.basePath)
  if fileDialog.basePath ~= "/" then
      table.insert(tmpFolderArr, {"..", getFilePath(fileDialog.basePath)})
  end
  for key, file in ipairs(tmpList) do
    local len = string.len(fileDialog.basePath)
    file = addSlashIfNeeded(fileDialog.basePath)..file
    if fs.isDir(file) then
      table.insert(tmpFolderArr, {fs.getName(file), file.."/"})
    else
      table.insert(tmpFileArr, {fs.getName(file), file})
    end
  end
  fileDialog.currentPathFiles = tmpFileArr
  fileDialog.currentPathFolders = tmpFolderArr
  fileDialog.dirScrollOffset=0
  fileDialog.fileScrollOffset=0
  checkDialogLimits()
end
function dialogReset()
  fileDialog.currentTab=1
  fileDialog.dirScrollOffset=0
  fileDialog.fileScrollOffset=0
  fileDialog.dirSelected=1
  fileDialog.fileSelected=1
end
local dirFileListHeight = 1
function checkDialogLimits()
  if fileDialog.selectedFooterOption < 1 then
    fileDialog.selectedFooterOption = #fileDialog.footer
  end
  if fileDialog.selectedFooterOption > #fileDialog.footer then
    fileDialog.selectedFooterOption = 1
  end

  if fileDialog.dirSelected < 1 then
    fileDialog.dirSelected = 1
  end
  if fileDialog.fileSelected < 1 then
    fileDialog.fileSelected = 1
  end
  if fileDialog.fileSelected > #fileDialog.currentPathFiles then
    fileDialog.fileSelected = #fileDialog.currentPathFiles
  end
  if fileDialog.dirSelected > #fileDialog.currentPathFolders then
    fileDialog.dirSelected = #fileDialog.currentPathFolders
  end
  
  if fileDialog.dirScrollOffset > dirFileListHeight-fileDialog.dirSelected+1 then
      fileDialog.dirScrollOffset=dirFileListHeight-fileDialog.dirSelected+1
  elseif fileDialog.dirScrollOffset < 1-fileDialog.dirSelected then
      fileDialog.dirScrollOffset=1-fileDialog.dirSelected
  end
  
  if fileDialog.fileScrollOffset > dirFileListHeight-fileDialog.fileSelected+1 then
      fileDialog.fileScrollOffset=dirFileListHeight-fileDialog.fileSelected+1
  elseif fileDialog.fileScrollOffset < 1-fileDialog.fileSelected then
      fileDialog.fileScrollOffset=1-fileDialog.fileSelected
  end
  
  
  
  if collectedInputsCursor.file < 0 then
      collectedInputsCursor.file = 0
  elseif collectedInputsCursor.file > string.len(collectedInputs.file) then
      collectedInputsCursor.file = string.len(collectedInputs.file)
  end
  if collectedInputsOffsets.file >= collectedInputsSize.file - collectedInputsCursor.file-1 then
      collectedInputsOffsets.file=collectedInputsSize.file-collectedInputsCursor.file-1
  elseif collectedInputsOffsets.file < 0-collectedInputsCursor.file then
      collectedInputsOffsets.file=0-collectedInputsCursor.file
  end
end
function showFileSaveAsDialog()
  if fs.exists(currentFile) and not fs.isDir(currentFile) then
    fileDialog.basePath=addSlashIfNeededLeft(shell.resolve(getFilePath(currentFile)))
  else
    --fileDialog.basePath=addSlashIfNeededLeft(shell.resolve(""))
  end
  prepareDialog()
  dialogReset()
  fileDialog.title="Save File As.."
  collectedInputs.file = addSlashIfNeededLeft(shell.resolve(currentFile))
  collectedInputsCursor.file = string.len(collectedInputs.file)
  collectedInputsOffsets.file = 0
  collectedInputsSize.file = 0
  fileDialog.footer={{"Save",trySaveFileFromDialog},{"Cancel",hideFileDialog}}
  showingFileDialog = true
end
function showFileOpenDialog()
  if fs.exists(currentFile) and not fs.isDir(currentFile) then
    fileDialog.basePath=addSlashIfNeededLeft(shell.resolve(getFilePath(currentFile)))
  else
    --fileDialog.basePath=addSlashIfNeededLeft(shell.resolve(""))
  end
  prepareDialog()
  dialogReset()
  fileDialog.title="Open File"
  collectedInputs.file = addSlashIfNeededLeft(shell.resolve(currentFile))
  collectedInputsCursor.file = string.len(collectedInputs.file)
  collectedInputsOffsets.file = 0
  collectedInputsSize.file = 0
  fileDialog.footer={{"Open",tryOpenFileFromDialog},{"Cancel",hideFileDialog}}
  showingFileDialog = true
end
function drawFileDialog(monitor)
    w,h = monitor.getSize()
    local height = h-2
    local width = w-4
    local offX = (w+1)/2 - (width-1)/2
    local offY = (h+1)/2 - (height-1)/2
    local footerMsg = ""
    for o=1, #fileDialog.footer do
      local item = fileDialog.footer[o][1]
      if fileDialog.selectedFooterOption == o then
        item = "["..item.."]"
      else
        item = " "..item.." "
      end
      if o > 1 then
        item = " "..item
      end
      footerMsg=footerMsg..item
    end
    if monitor.isColor() then
      monitor.setBackgroundColor(colors.cyan)
    else
      monitor.setBackgroundColor(colors.black)
    end
    monitor.clear()
    for i=1, height do
        setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
        monitor.setCursorPos(offX, offY+i-1)
        if i == 1 or i == height then
            monitor.write("+")
            monitor.write(string.rep("-", width-2))
            monitor.write("+")
            if i == 1 then
                setColors(monitor, colors.black, colors.yellow, colors.white, colors.black)
                centerText(monitor, offY+i-1, " "..fileDialog.title.." ")
                setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
            elseif i == height then
                if fileDialog.currentTab == 4 then
                  setColors(monitor, colors.black, colors.white, colors.black, colors.white)
                else
                  setColors(monitor, colors.lightGray, colors.white, colors.white, colors.black)
                end
                centerText(monitor, offY+i-1, " "..footerMsg.." ")
                setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
            end
        else
            setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
            monitor.write("|")
            setColors(monitor, colors.white, colors.black, colors.white, colors.black)
            monitor.write(string.rep(" ", width-2))
            setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
            monitor.write("|")
        end
    end
    setColors(monitor, colors.white, colors.gray, colors.white, colors.black)
    monitor.setCursorPos(offX + 2, offY + 2)
    monitor.write("File: ")
    if fileDialog.currentTab == 1 then
      setColors(monitor, colors.black, colors.white, colors.black, colors.white)
    else
      setColors(monitor, colors.lightGray, colors.white, colors.black, colors.white)
    end
    local workingWidth = width - 10
    local textWidth = workingWidth
    collectedInputsSize.file = textWidth
    monitor.write(string.rep(" ", workingWidth))
    monitor.setCursorPos(offX + 8, offY + 2)    

    currentLineFit = collectedInputs.file
    inputCursor = collectedInputsCursor.file
    inputOffsetX = collectedInputsOffsets.file
    if currentLineFit then
        currentLineFit = string.sub(currentLineFit, -inputOffsetX+1, string.len(currentLineFit))
        if string.len(currentLineFit) > textWidth then
            currentLineFit = string.sub(currentLineFit, 1, textWidth)
        end
        monitor.write(rpad(currentLineFit, textWidth, " "))
    end
    if fileDialog.currentTab == 1 and isBlinking then
        monitor.setCursorPos(offX+8+inputCursor+inputOffsetX,offY + 2) 
        setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
        local msg = string.sub(currentLineFit,inputCursor+inputOffsetX+1,inputCursor+inputOffsetX+1)
        if msg == "" then msg = " " end
        monitor.write(msg)
    end
            
    
    local fileStart = 15
    setColors(monitor, colors.white, colors.gray, colors.white, colors.black)
    monitor.setCursorPos(offX + 2, offY + 4)
    monitor.write("Dir: ")
    monitor.setCursorPos(offX + fileStart, offY + 4)
    monitor.write("Files: "..fileDialog.basePath)
    
    dirFileListHeight = height - offY - 6
    for i=0, dirFileListHeight do
      local dirIndex = i+1-fileDialog.dirScrollOffset
      local fileIndex = i+1-fileDialog.fileScrollOffset
        
      if fileDialog.currentTab == 2 then
        setColors(monitor, colors.black, colors.white, colors.black, colors.white)
      else
        setColors(monitor, colors.lightGray, colors.white, colors.black, colors.white)
      end
      monitor.setCursorPos(offX + 2, offY + 5+i)
      monitor.write(string.rep(" ", fileStart - offX))
      if fileDialog.currentPathFolders[dirIndex] then
        if fileDialog.currentTab == 2 and dirIndex == fileDialog.dirSelected then
          setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
        end
        monitor.setCursorPos(offX + 2, offY + 5+i)
        monitor.write(fileDialog.currentPathFolders[dirIndex][1])
      end
      
              
      if fileDialog.currentTab == 3 then
        setColors(monitor, colors.black, colors.white, colors.black, colors.white)
      else
        setColors(monitor, colors.lightGray, colors.white, colors.black, colors.white)
      end
      monitor.setCursorPos(offX + fileStart, offY + 5+i)
      monitor.write(string.rep(" ", width - fileStart - 2))
      if fileDialog.currentPathFiles[fileIndex] then
        if fileDialog.currentTab == 3 and fileIndex == fileDialog.fileSelected then
          setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
        end
        monitor.setCursorPos(offX + fileStart, offY + 5+i)
        monitor.write(fileDialog.currentPathFiles[fileIndex][1])
      end
    end
    
    --gui debug
    --monitor.setCursorPos(1,1)
    --monitor.write("ICur="..collectedInputsCursor.file.." IOff="..collectedInputsOffsets.file.." ISize="..collectedInputsSize.file.." IValSize="..string.len(collectedInputs.file))
end
function drawAbout(monitor)
    w,h = monitor.getSize()
    local height = 13
    local width = 39
    local offX = (w+1)/2 - (width-1)/2
    local offY = (h+1)/2 - (height-1)/2
    if monitor.isColor() then
      monitor.setBackgroundColor(colors.cyan)
    else
      monitor.setBackgroundColor(colors.black)
    end
    monitor.clear()
    for i=1, height do
        setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
        monitor.setCursorPos(offX, offY+i-1)
        if i == 1 or i == height then
            monitor.write("+")
            monitor.write(string.rep("-", width-2))
            monitor.write("+")
            if i == 1 then
                setColors(monitor, colors.black, colors.yellow, colors.black, colors.white)
                centerText(monitor, offY+i-1, " About TACO BETA ")
                setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
            end
        else
            setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
            monitor.write("|")
            setColors(monitor, colors.white, colors.black, colors.white, colors.black)
            monitor.write(string.rep(" ", width-2))
            setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
            monitor.write("|")
        end
    end
    drawPictureTable(monitor, logo, offX + 2, offY + 2, colours.white)
    setColors(monitor, colors.white, colors.orange, colors.white, colors.black)
    centerTextWidth(monitor, offX + 14, offY+3, width - ((offX*2) + 10), "=TACO BETA=")
    setColors(monitor, colors.white, colors.gray, colors.white, colors.black)
    centerTextWidth(monitor, offX + 14, offY+4, width - ((offX*2) + 10), "Edit with flavor!")
    setColors(monitor, colors.white, colors.lightGray, colors.white, colors.black)
    centerTextWidth(monitor, offX + 14, offY+6, width - ((offX*2) + 10), "By: da404lewzer")
    centerTextWidth(monitor, offX + 14, offY+8, width - ((offX*2) + 10), "TurtleScripts.com")
    centerTextWidth(monitor, offX + 14, offY+9, width - ((offX*2) + 10), "Project #gjdh01")
end
function drawMessage(monitor)
  w,h = monitor.getSize()
  local height = #message.body + 2 + 2 --2 for header/footer, 2 for border
  local width = 1
  for i=1, #message.body do
    if string.len(message.body[i])+4 > width then
        width = string.len(message.body[i])+4
    end
  end
  local footerMsg = ""
  for o=1, #message.footer do
    local item = message.footer[o][1]
    if selectedMessageOption == o then
      item = "["..item.."]"
    else
      item = " "..item.." "
    end
    if o > 1 then
      item = " "..item
    end
    footerMsg=footerMsg..item
  end
  if string.len(message.title)+4 > width then
      width = string.len(message.title)+4
  end
  if string.len(footerMsg)+4 > width then
      width = string.len(footerMsg)+4
  end
  local offX = (w+1)/2 - (width-1)/2
  local offY = (h+1)/2 - (height-1)/2
  for i=1, height do
    setColors(monitor, colors.orange, colors.black, colors.black, colors.white)
    monitor.setCursorPos(offX, offY+i-1)
    if i == 1 or i == height then
        monitor.write("+")
        monitor.write(string.rep("-", width-2))
        monitor.write("+")
        if i == 1 then
            setColors(monitor, colors.black, colors.yellow, colors.black, colors.white)
            centerText(monitor, offY+i-1, " "..message.title.." ")
            setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
        elseif i == height then
            setColors(monitor, colors.black, colors.white, colors.black, colors.white)
            centerText(monitor, offY+i-1, " "..footerMsg.." ")
            setColors(monitor, colors.orange, colors.black, colors.white, colors.black)
        end
    else
        setColors(monitor, colors.orange, colors.black, colors.black, colors.white)
        monitor.write("|")
        setColors(monitor, colors.white, colors.black, colors.white, colors.black)
        monitor.write(string.rep(" ", width-2))
        setColors(monitor, colors.orange, colors.black, colors.black, colors.white)
        monitor.write("|")
        if i-2 > 0 and i-2 <= #message.body then
            setColors(monitor, colors.white, colors.black, colors.white, colors.black)
            centerText(monitor, offY+i-1, message.body[i-2])
        end
        
    end
  end
end
function messageBox(title, bodyArr, footer)
  message.title = title
  message.body = bodyArr
  message.footer = footer
  showingMessage = true
end
local function trySaveFileAs()
  showFileSaveAsDialog()
end
local function stopEditor()
  running = false
end
local function trySaveFile()
  if currentFile == ""then
    trySaveFileAs()
  else
    saveFile(currentFile, fileLinesArr)
  end
end
local function trySaveFileThenNew()
  if currentFile == ""then
    trySaveFileAs()
  else
    saveFile(currentFile, fileLinesArr)
    if isSaved and not isChanged then
      newFile()
    end
  end
end
local function trySaveFileThenOpen()
  if currentFile == ""then
    trySaveFileAs()
  else
    saveFile(currentFile, fileLinesArr)
    if isSaved and not isChanged then
      showFileOpenDialog()
    end
  end
end
local function trySaveFileThenExit()
  if currentFile == ""then
    trySaveFileAs()
  else
    saveFile(currentFile, fileLinesArr)
    if isSaved and not isChanged then
      stopEditor()
    end
  end
end
local function tryExitEditor()
  if isSaved and not isChanged then
    stopEditor()
  else
    messageBox( "Exit Editor?", 
                {
                  "The current file isn't saved, save it first?"
                }, 
                {{"YES", trySaveFileThenExit}, {"NO", stopEditor}, {"CANCEL", null}}
              )
  end
end
local function tryOpenFile()
  if isSaved and not isChanged then
    showFileOpenDialog()
  else
    messageBox( "Create New File..", 
                {
                  "The current file isn't saved, save it first?"
                }, 
                {{"YES", trySaveFileThenOpen}, {"NO", showFileOpenDialog}, {"CANCEL", null}}
              )
  end
end
local function tryNewFile()
  if isSaved and not isChanged then
    newFile()
  else
    messageBox( "Create New File..", 
                {
                  "The current file isn't saved, save it first?"
                }, 
                {{"YES", trySaveFileThenNew}, {"NO", newFile}, {"CANCEL", null}}
              )
  end
end
local function revertFile()
  if currentFile == "" then
    fileLinesArr = {""}
    isChanged = false
  else
		if not openFile(currentFile) then
			messageBox("Error!", {"Couldn't revert file!"}, {{"OK",null}})
		end
  end
end
local function tryRevertFile()
  messageBox( "Revert file?", 
              {
                "This cannot be done, sure?"
              }, 
              {{"YES", revertFile}, {"NO", null}}
            )
end
local function doFileDialogDefaultOption()
  local item = fileDialog.footer[1][2]
  if item then
    item()
  end
  fileDialog.selectedFooterOption = 1
end
local function doFileDialogMessageOption()
  local item = fileDialog.footer[fileDialog.selectedFooterOption][2]
  if item then
    item()
  end
  fileDialog.selectedFooterOption = 1
end
local function doMessageOption()
  local item = message.footer[selectedMessageOption][2]
  if item then
    item()
  end
  selectedMessageOption = 1
end
local function doMenuAction(action)
    if action == "exit_app" then
        tryExitEditor()
    elseif action == "new_file" then
        tryNewFile()
    elseif action == "open_file" then
        tryOpenFile()
    elseif action == "revert_file" then
        tryRevertFile()
    elseif action == "save_file" then
        trySaveFile()
    elseif action == "saveas_file" then
        trySaveFileAs()
    elseif action == "paste_line" then
        table.insert(fileLinesArr, curLine, config.clipboard)
        isChanged=true
    elseif action == "clone_line" then
        table.insert(fileLinesArr, curLine, fileLinesArr[curLine])
        isChanged=true
    elseif action == "copy_line" then
        config.clipboard = fileLinesArr[curLine]
    elseif action == "cut_line" then
        config.clipboard = fileLinesArr[curLine]
        table.remove(fileLinesArr, curLine)
        if #fileLinesArr == 0 then
          fileLinesArr = {""}
        end
        isChanged=true
    elseif action == "delete_line" then
        table.remove(fileLinesArr, curLine)
        if #fileLinesArr == 0 then
          fileLinesArr = {""}
        end
        isChanged=true
    elseif action == "perform_update" then
        setColors(temp, colors.black, colors.white, colors.black, colors.white)
        term.clear()
        term.setCursorPos(1,1)
        updateAllTheThings()
        messageBox("Downloaded Latest", 
                    {
                        "Restart TACO to complete :)"
                    }, 
                    {{"OK", null}})
      timer = os.startTimer(0.01)
    elseif action == "show_about" then
        showAboutBox = true
    elseif action == "fullscreen_toggle" then
        config.isFullScreen = not config.isFullScreen
        checkLinePosition()
        checkColPosition()
    else
        --show_help
        --show_settings
        --replace_text
        --find_text
        --delete_line
        --paste_line
        --copy_line
        --cut_line
        --clear_selection
        --paste_selection
        --cut_selection
        --copy_selection
        --print_file
        --revert_file
        --open_file
        
    end
    selectedMenu = 1
    selectedSubMenu = 1
end


loadSettings()

if #tArgs > 0 then
  --openFile(tArgs[1])
	if not openFile(addSlashIfNeededLeft(shell.resolve(getFilePath(tArgs[1])))) then
		messageBox("Error!", {"Couldn't open file!"}, {{"OK",null}})
	end
end
prepareDialog()
if showFirstTimeMessage then
    messageBox("Welcome to TACO BETA", 
                {
                    "Welcome to the editor with flavor!",
                    "This is BETA sorry if you find bugs :(",
                    "",
                    "Presse ENTER to continue,",
                    "Press CONTROL to access menus :)"
                }, 
                {{"OK", null}})
end
while running do
  local event, p1,p2,p3 = os.pullEvent()
  if event == "mouse_scroll" then 
    if not showingMessage and not showAboutBox and not inMenu then
      curLine=curLine+p1
      checkLinePosition()
      checkColPosition()
      resetBlink()
    end
  elseif event == "mouse_click" then 
    if not showingMessage and not showAboutBox and not inMenu then
      local padWidth = string.len(tostring(#fileLinesArr))
      local offsetY = 1
      if config.isFullScreen then offsetY = 0 end
      if p1 == 1 then
        curCol=p2-padWidth-scrollOffsetX
        curLine=p3-offsetY-scrollOffsetY
        checkLinePosition()
        checkColPosition()
      end
      resetBlink()
    end
  elseif event == "key" then 
		if showingMessage then
      if p1 == keys.enter then
        showingMessage = false
        doMessageOption()
      elseif p1 == keys.left or p1 == keys.up then
        selectedMessageOption=selectedMessageOption-1
        checkMessageOptionPositions()
      elseif p1 == keys.right or p1 == keys.down then
        selectedMessageOption=selectedMessageOption+1
        checkMessageOptionPositions()
      end
    elseif showAboutBox then
      if p1 == keys.enter then
        showAboutBox = false
      end
    elseif showingFileDialog then
      if p1 == keys.enter then
        if fileDialog.currentTab == 1 then
          doFileDialogDefaultOption()
        elseif fileDialog.currentTab == 2 then
          fileDialog.basePath = fileDialog.currentPathFolders[fileDialog.dirSelected][2]          
          prepareDialog()
        elseif fileDialog.currentTab == 3 then
          collectedInputs.file = fileDialog.currentPathFiles[fileDialog.fileSelected][2]
          fileDialog.currentTab = 1
          prepareDialog()          
        elseif fileDialog.currentTab == 4 then
          doFileDialogMessageOption()
        end
      elseif p1 == keys.tab then
        fileDialog.currentTab=fileDialog.currentTab+1
        if fileDialog.currentTab > fileDialog.currentTabMax then
          fileDialog.currentTab = 1
        end
      elseif p1 == keys.home then
        if fileDialog.currentTab == 1 then
          collectedInputsCursor.file=0
        end
        checkDialogLimits()
      elseif p1 == keys["end"] then
        if fileDialog.currentTab == 1 then
          collectedInputsCursor.file=string.len(collectedInputs.file)+1
        end
        checkDialogLimits()

      elseif p1 == keys.backspace then
        if fileDialog.currentTab == 1 then
          if collectedInputsCursor.file > 0 then
            local leftSide = string.sub(collectedInputs.file, 1, collectedInputsCursor.file-1)
            local rightSide = string.sub(collectedInputs.file, collectedInputsCursor.file+1)
            collectedInputs.file = leftSide..rightSide
            collectedInputsCursor.file=collectedInputsCursor.file-1
          end
        end
        checkDialogLimits()
      elseif p1 == keys.delete then
        if fileDialog.currentTab == 1 then
          if collectedInputsCursor.file <= string.len(collectedInputs.file) then
            local leftSide = string.sub(collectedInputs.file, 1, collectedInputsCursor.file)
            local rightSide = string.sub(collectedInputs.file, collectedInputsCursor.file+2)
            collectedInputs.file = leftSide..rightSide
          end
        end
        checkDialogLimits()
      elseif p1 == keys.up or p1 == keys.left then
        if fileDialog.currentTab == 1 then
          collectedInputsCursor.file=collectedInputsCursor.file-1
        elseif fileDialog.currentTab == 2 then
          fileDialog.dirSelected=fileDialog.dirSelected-1
        elseif fileDialog.currentTab == 3 then
          fileDialog.fileSelected=fileDialog.fileSelected-1
        elseif fileDialog.currentTab == 4 then
          fileDialog.selectedFooterOption=fileDialog.selectedFooterOption-1
        end
        checkDialogLimits()
      elseif p1 == keys.down or p1 == keys.right then
        if fileDialog.currentTab == 1 then
          collectedInputsCursor.file=collectedInputsCursor.file+1
        elseif fileDialog.currentTab == 2 then
          fileDialog.dirSelected=fileDialog.dirSelected+1
        elseif fileDialog.currentTab == 3 then
          fileDialog.fileSelected=fileDialog.fileSelected+1
        elseif fileDialog.currentTab == 4 then
          fileDialog.selectedFooterOption=fileDialog.selectedFooterOption+1
        end
        checkDialogLimits()
      end
    else
      if state == "edit" then
        if p1 == 29 or p1 == 157 then
          inMenu = not inMenu
        end
        if inMenu then
          if p1 == keys.up then
            selectedSubMenu=selectedSubMenu-1
            checkMenuPositions()
            if menus[selectedMenu].items[selectedSubMenu][1] == "--" then
              selectedSubMenu=selectedSubMenu-1
              checkMenuPositions()
            end
          elseif p1 == keys.down then
            selectedSubMenu=selectedSubMenu+1
            checkMenuPositions()
            if menus[selectedMenu].items[selectedSubMenu][1] == "--" then
              selectedSubMenu=selectedSubMenu+1
              checkMenuPositions()
            end
          elseif p1 == keys.left then
            selectedSubMenu = 1
            selectedMenu=selectedMenu-1
            checkMenuPositions()
          elseif p1 == keys.right then
            selectedSubMenu = 1
            selectedMenu=selectedMenu+1
            checkMenuPositions()
          elseif p1 == keys.enter then
            doMenuAction(menus[selectedMenu].items[selectedSubMenu][3])
            inMenu = false
          end
        else
          if p1 == keys.up then
            curLine=curLine-1
          elseif p1 == keys.down then
            curLine=curLine+1
          elseif p1 == 201 then -- page up
            curLine=curLine-(h-2)
          elseif p1 == 209 then --page down
            curLine=curLine+(h-2)
          elseif p1 == keys.left then
            curCol=curCol-1
          elseif p1 == keys.right then
            curCol=curCol+1
          elseif p1 == keys.home then
            curCol=0
          elseif p1 == keys["end"] then
            curCol=string.len(fileLinesArr[curLine])+1
          elseif p1 == keys.tab then
            isChanged=true
            local leftSide = string.sub(fileLinesArr[curLine], 1, curCol)
            local rightSide = string.sub(fileLinesArr[curLine], curCol+1)
            fileLinesArr[curLine] = leftSide.."  "..rightSide
            curCol=curCol+2
            resetBlink()
            checkLinePosition()
            checkColPosition()
          elseif p1 == keys.enter then
            isChanged=true
            if curCol == string.len(fileLinesArr[curLine]) then
              curLine=curLine+1
              curCol=0
              table.insert(fileLinesArr, curLine, "")
            elseif curCol == 0 then
              table.insert(fileLinesArr, curLine, "")  
              curLine=curLine+1
            else                    
              local leftSide = string.sub(fileLinesArr[curLine], 1, curCol)
              local rightSide = string.sub(fileLinesArr[curLine], curCol+1)
              fileLinesArr[curLine] = leftSide
              table.insert(fileLinesArr, curLine+1, rightSide)   
              curLine=curLine+1
              curCol=0
            end
          elseif p1 == keys.backspace then
            isChanged=true
            if curCol > 0 then
              local leftSide = string.sub(fileLinesArr[curLine], 1, curCol-1)
              local rightSide = string.sub(fileLinesArr[curLine], curCol+1)
              fileLinesArr[curLine] = leftSide..rightSide
              curCol=curCol-1
            elseif curCol == 0 and curLine > 1 then
              local newCurCol = string.len(fileLinesArr[curLine-1])
              fileLinesArr[curLine-1] = fileLinesArr[curLine-1] .. fileLinesArr[curLine]
              table.remove(fileLinesArr, curLine)
              curLine = curLine -1
              curCol = newCurCol
            end
          elseif p1 == keys.delete then
            isChanged=true
            if curCol < string.len(fileLinesArr[curLine]) then
              local leftSide = string.sub(fileLinesArr[curLine], 1, curCol)
              local rightSide = string.sub(fileLinesArr[curLine], curCol+2)
              fileLinesArr[curLine] = leftSide..rightSide
            elseif curCol == string.len(fileLinesArr[curLine]) and curLine < #fileLinesArr then
              fileLinesArr[curLine] = fileLinesArr[curLine] .. fileLinesArr[curLine+1]
              table.remove(fileLinesArr, curLine+1)
            end
          end
          checkLinePosition()
          checkColPosition()
        end          
        resetBlink()
      elseif state == "msg" then
        if p1 == keys.enter then
          state = "edit"
        end 
      end
    end
  elseif event == "char" then
    if showingMessage then
    elseif showAboutBox then
    elseif showingFileDialog then
      if fileDialog.currentTab == 1 then
        local leftSide = string.sub(collectedInputs.file, 1, collectedInputsCursor.file)
        local rightSide = string.sub(collectedInputs.file, collectedInputsCursor.file+1)        
        collectedInputs.file=leftSide..p1..rightSide
        collectedInputsCursor.file=collectedInputsCursor.file+1
        checkDialogLimits()
      end
    else
      if state == "edit" then
        isChanged=true
        local leftSide = string.sub(fileLinesArr[curLine], 1, curCol)
        local rightSide = string.sub(fileLinesArr[curLine], curCol+1)
        fileLinesArr[curLine] = leftSide..p1..rightSide
        curCol=curCol+1
        resetBlink()
        checkLinePosition()
        checkColPosition()
      end
    end
  elseif event == "timer" and p1 == timer then
    if state == "edit" then
      drawScreen(term)
    elseif state == "additem" then
      --displayAddItem()
    elseif state == "msg" then
      --displayMsg(currentMessage)
    end
    if showingFileDialog then
      drawFileDialog(term)
    end
    if showingMessage then
      drawMessage(term)
    end
    if showAboutBox then
      drawAbout(term)
    end
    timer = os.startTimer(0.01)
  end
  if os.clock() - lastBlinkTime > 0.5 then
    lastBlinkTime = os.clock()
    isBlinking = not isBlinking
  end
end

saveSettings()
setColors(term, colors.black, colors.white, colors.black, colors.white)
term.clear()
term.setCursorPos(1,1)