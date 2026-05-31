
1、下载工具和模型
```
https://github.com/ggml-org/llama.cpp/releases/download/b9437/llama-b9437-bin-win-cuda-13.3-x64.zip
https://modelscope.cn/models/Qwen/Qwen3.6-35B-A3B
```
2、解压工具包
```
llama-b9437-bin-win-cuda-13.3-x64.zip
```
在目录下创建 models 文件夹
把下载的模型放进去

3、在目录文件夹下创建 run.bat 
```
@echo off
chcp 65001 >nul
title LLM 模型与多模态文件选择器（完美穿透局域网版）

cd /d "%~dp0"

:: --- 自动获取本机局域网真实的真实 IP（防止被虚拟机干扰） ---
set "local_ip=127.0.0.1"

:: 优先寻找 192.168.x.x 格式的真实物理网卡 IP
for /f "tokens=4 delims= " %%i in ('route print ^| findstr 0.0.0.0 ^| findstr "192.168."') do (
    set "local_ip=%%i"
)

:: 如果没找到 192 段，再尝试寻找 10.x.x.x 段
if "%local_ip%"=="127.0.0.1" (
    for /f "tokens=4 delims= " %%i in ('route print ^| findstr 0.0.0.0 ^| findstr " 10."') do (
        set "local_ip=%%i"
    )
)

:: 万一还是没找到（比如 172 纯物理内网），则使用保底兼容逻辑
if "%local_ip%"=="127.0.0.1" (
    for /f "tokens=4 delims= " %%i in ('route print ^| findstr 0.0.0.0 ^| findstr /v "255.255.255.255"') do (
        set "local_ip=%%i"
    )
)

:menu
cls
setlocal enabledelayedexpansion
echo ===========================================
echo       LLM 模型与多模态文件启动器
echo ===========================================
echo.

:: --- 第一步：扫描并选择主模型文件 ---
echo === [第一步] 请选择主模型文件 ===
set model_count=0
for %%f in (models\*.gguf) do (
    echo "%%~nxf" | findstr /i /v "mmproj" >nul
    if !errorlevel! equ 0 (
        set /a model_count+=1
        set "model_file[!model_count!]=%%~nxf"
        echo !model_count!. %%~nxf
    )
)
echo.
set /p m_choice=请输入模型编号：
if not defined model_file[%m_choice%] goto :menu
set "selected_model=!model_file[%m_choice%]!"

:: --- 第二步：扫描并选择多模态映射文件 ---
echo.
echo === [第二步] 请选择 mmproj 文件 (输入 0 跳过) ===
set mm_count=0
for %%f in (models\*mmproj*.gguf) do (
    set /a mm_count+=1
    set "mm_file[!mm_count!]=%%~nxf"
    echo !mm_count!. %%~nxf
)
echo 0. 不使用多模态文件
echo.
set /p mm_choice=请输入 mmproj 编号：

:: --- 第三步：自定义服务端口 ---
echo.
echo === [第三步] 请设置服务端口 ===
set port=8080
set /p user_port=请输入端口号 (直接回车默认使用 8080)：
if not "!user_port!"=="" set port=!user_port!

:: --- 🎛️ 核心新增：全自动动态放行防火墙端口 ---
echo.
echo 🛡️ 正在检查并自动优化防火墙设置，确保局域网顺畅访问...
netsh advfirewall firewall delete rule name="LLM_Server_AutoPort" >nul 2>&1
netsh advfirewall firewall add rule name="LLM_Server_AutoPort" dir=in action=allow protocol=TCP localport=!port! >nul 2>&1
if !errorlevel! equ 0 (
    echo  [成功] 已自动为您开放局域网进站端口: !port!
) else (
    echo  [提示] 自动开放端口失败。若局域网无法访问，请尝试【右键-以管理员身份运行】此脚本。
)

:: --- 核心展示：在启动前疯狂提示真实的访问地址 ---
echo.
echo ============================================================
echo   📢 服务即将启动！请复制以下真实访问地址：
echo ------------------------------------------------------------
echo   🔗 [浏览器直接访问（网页端）]
echo   • 本地访问：http://127.0.0.1:!port!
echo   • 局域网访问：http://%local_ip%:!port!
echo.
echo   🛠️ [第三方软件集成（API 接口）]
echo   • 本地接口：http://127.0.0.1:!port!/v1
echo   • 局域网接口：http://%local_ip%:!port!/v1
echo ============================================================
echo   (注：下方 llama-server 提示的 0.0.0.0 代表正在监听上述所有地址)
echo ============================================================
echo.
timeout /t 3 >nul

if "%mm_choice%"=="0" (
    llama-server.exe ^
        -m "models\%selected_model%" ^
        -ngl 999 -c 131072 -np 1 -n 8192 ^
        --host 0.0.0.0 --port !port!
) else (
    if defined mm_file[%mm_choice%] (
        set "selected_mm=!mm_file[%mm_choice%]!"
        llama-server.exe ^
            -m "models\%selected_model%" ^
            --mmproj "models\!selected_mm!" ^
            -ngl 999 -c 131072 -n 8192 ^
            --image-min-tokens 1024 ^
            --host 0.0.0.0 --port !port!
    ) else (
        echo 输入错误，正在返回主菜单...
        pause
        goto :menu
    )
)

pause
```
4、点击 run.bat 启动脚本

5、测试效果
```
机器配置信息
处理器：i7-12700H
内存：32.0 GB
显卡：NVIDIA RTX 3060 Laptop GPU 6GB
速度：17-18t/s
```

6、遇到的问题

简单计算都要思考无法关闭，暂无解决方法。