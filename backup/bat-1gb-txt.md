通过代码命令快速创建指定大小容量的文件。

## 创建1GB文本文件
```
fsutil file createnew f:\1GB.txt 1073741824
```
## 换算单位
```
byte (B):1073741824
kilobyte (kB):1048576
megabyte (MB):1024
gigabyte (GB):	1
```
## 一件生成大文件
```
@echo off
chcp 65001 >nul
title 真实GB文件生成工具

echo ================================
echo   真实写入文件（占用磁盘空间）
echo ================================
echo.

set /p size=请输入大小（GB）： 
set /p drive=请输入盘符（例如 F:\）： 

set filename=%drive%%size%GB.bin

echo.
echo 正在生成 %size%GB 文件（真实写入数据）...
echo 文件：%filename%
echo 请稍等...

powershell -NoProfile -Command ^
"$size=%size%; $path='%filename%'; $buffer = New-Object byte[](1MB); $fs=[System.IO.File]::OpenWrite($path); for($i=0;$i -lt $size*1024;$i++){ $fs.Write($buffer,0,$buffer.Length) }; $fs.Close()"

echo.
echo 完成！
pause
```

## 复制文件速度对比
 K70 三星U盘 百度网盘
![ScreenShot_2026-07-05_132445_931.png](https://i.imgur.com/BLmbzkG.png)
