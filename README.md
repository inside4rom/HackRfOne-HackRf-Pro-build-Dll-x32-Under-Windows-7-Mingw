# HackRf-One-HackRf-Pro-build-Dll-X32-Under-Windows-7-MINGW-
Инструкция: Сборка hackrf.dll для Windows 7 x86

---

# 📄 Инструкция: Сборка libhackrf.dll для Windows 7 x86 (Delphi 7 Compatible)

Эта утилита позволяет собрать современную версию библиотеки `libhackrf` (поддерживающую HackRF One и Pro) под **32-битную архитектуру** для использования в приложениях написанных в X32/X86 компиляторах типа **Delphi 7** 

> ⚠️ **Важно:** Официальные сборки от Great Scott Gadgets предназначены для Windows 10/11 x64. Данный гайд обходит это ограничение, используя старый тулчейн MinGW-w64, совместимый с legacy-CRT (`msvcrt.dll`).

## 🛠 Требования к окружению

Для сборки вам понадобятся следующие компоненты. Все пути должны быть без пробелов и кириллицы.

### 1. Компилятор: MinGW-w64 (GCC 5.3.0, i686, win32 threads)
*   **Почему эта версия?** GCC 5.3 — последняя стабильная версия, гарантированно работающая на Win7 без новых API зависимостей. Вариант `win32 threads` исключает необходимость тащить лишние DLL (`libwinpthread`).
*   **Скачать:** [SourceForge Mirror](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/5.3.0/threads-win32/dwarf/i686-5.3.0-release-win32-dwarf-rt_v4-rev0.7z/download)
*   **Установка:** Распакуйте архив в `C:\mingw32`. Добавьте `C:\mingw32\bin` в системную переменную PATH.

### 2. CMake (Версия 3.16.9)
*   **Почему эта версия?** Новее версии (3.20+) не поддерживают старые генераторы Makefiles и могут требовать Win10+ SDK.
*   **Скачать:** [Official Archive](https://cmake.org/files/v3.16/cmake-3.16.9-win32-x86.zip)
*   **Установка:** Распакуйте в `C:\cmake-3.16.9`.

### 3. libusb-1.0 (Binaries for MinGW32)
*   **Зачем нужен:** Низкоуровневый доступ к USB-устройству HackRF.
*   **Скачать:** [GitHub Releases v1.0.27](https://github.com/libusb/libusb/releases/download/v1.0.27/libusb-1.0.27-binaries.7z)
*   **Установка:**
    1. Распакуйте архив.
    2. Скопируйте содержимое папки `MinGW32` в `C:\libusb-1.0\MinGW32`.
    3. Скопируйте содержимое папки `include` в `C:\libusb-1.0\include`.
    *Итоговая структура:*
    ```text
    C:\libusb-1.0\
    ├── include\
    │   └── libusb.h       <-- Критично! Файл должен лежать здесь
    └── MinGW32\
        ├── dll\libusb-1.0.dll
        └── static\libusb-1.0.dll.a
    ```

### 4. Исходный код HackRF
*   Рекомендуется использовать тег **v2024.02.1** (первая полная поддержка HackRF Pro + стабильность для One).
*   **Скачать:** [GitHub Tag v2024.02.1 Zip](https://github.com/greatscottgadgets/hackrf/archive/refs/tags/v2024.02.1.zip)
*   **Установка:** Распакуйте в `C:\hackrf-2024.02.1`.

---

## 🚀 Процесс сборки

### Шаг 1: Конфигурация проекта (CMake)

Создайте скрипт `build_config.bat` со следующим содержимым:

```batch
@echo off
cd /d C:\hackrf-2024.02.1\host\libhackrf
if exist build rmdir /s /q build
mkdir build
cd build

set CMAKE=C:\cmake-3.16.9\bin\cmake.exe
set GCC=C:/mingw32/bin/gcc.exe
set MAKE=C:/mingw32/bin/mingw32-make.exe

REM Пути к зависимостям
set USB_INC_PATH=C:/libusb-1.0/include
set USB_LIB=C:/libusb-1.0/MinGW32/static/libusb-1.0.dll.a

%CMAKE% .. -G "MinGW Makefiles" ^
  -DCMAKE_BUILD_TYPE=Release ^
  -DCMAKE_C_COMPILER=%GCC% ^
  -DCMAKE_MAKE_PROGRAM=%MAKE% ^
  -DLIBUSB_INCLUDE_DIR=%USB_INC_PATH% ^
  -DLIBUSB_LIBRARIES=%USB_LIB% ^
  -DBUILD_SHARED_LIBS=ON ^
  REM --kill-at убирает декорацию имен функций (@N), важно для Delphi stdcall/cdecl
  -DCMAKE_SHARED_LINKER_FLAGS="-Wl,--kill-at" ^
  REM Принудительно указываем стандарт C11 и путь к заголовкам libusb
  -DCMAKE_C_FLAGS="-I%USB_INC_PATH% -std=gnu11"

pause
```

Запустите этот скрипт. Успешным результатом является сообщение:
`-- Build files have been written to: ...`

### Шаг 2: Компиляция

Создайте скрипт `build_make.bat`:

```batch
@echo off
cd /d C:\hackrf-2024.02.1\host\libhackrf\build
C:\mingw32\bin\mingw32-make.exe VERBOSE=1
pause
```

Запустите его. Если сборка прошла успешно, вы увидите:
`[100%] Built target hackrf`

Готовый файл будет находиться по пути:
`C:\hackrf-2024.02.1\host\libhackrf\build\src\libhackrf.dll`



## 📜 Лицензия
Данный гайд распространяется под лицензией MIT. Исходный код HackRF доступен под GPL v2+.
