- https://github.com/ggml-org/llama.cpp/releases/tag/b9433

## 启动脚本
```
@echo off
chcp 65001 >nul
title LLM 模型与多模态文件选择器（支持局域网访问）

cd /d "%~dp0"

:: --- 自动获取本机局域网真实的真实 IP ---
set "local_ip=127.0.0.1"
for /f "tokens=4 delims= " %%i in ('route print ^| findstr 0.0.0.0 ^| findstr /v "255.255.255.255"') do (
    set "local_ip=%%i"
)

:menu
cls
setlocal enabledelayedexpansion
echo ===========================================
echo      LLM 模型与多模态文件启动器
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
    echo !count!. %%~nxf
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

:: --- 核心优化：在启动前疯狂提示真实的访问地址 ---
echo.
echo ============================================================
echo   📢 服务即将启动！请复制以下真实访问地址：
echo ------------------------------------------------------------
echo   [本地访问] http://127.0.0.1:!port!/v1
echo   [局域网访问] http://%local_ip%:!port!/v1
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
            --host 0.0.0.0 --port !port!
    ) else (
        echo 输入错误，正在返回主菜单...
        pause
        goto :menu
    )
)

pause
```
