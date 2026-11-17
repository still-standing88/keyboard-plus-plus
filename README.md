# Keyboard++

Keyboard++ is a C++17 header-only keyboard library modeled on Python's
`keyboard` package. Add `SRC` to your include path and include
`keyboard.hpp`. The headers can be included from several translation units.

```cpp
#include "keyboard.hpp"
#include <iostream>

int main() {
    const int shortcut = Keyboard::add_hotkey(L"ctrl+shift+k", [] {
        std::cout << "Shortcut pressed\n";
    });

    Keyboard::wait(L"esc");
    Keyboard::remove_hotkey(shortcut);
    Keyboard::listener().stop();
    Keyboard::listener().rethrow_error();
}
```

Use `Keyboard::send(L"ctrl+shift+k")` for a keystroke or chord and
`Keyboard::write(L"Hello")` for text. A comma separates successive hotkey
steps, as in `L"ctrl+k, ctrl+c"`. Key names use wide strings. Key codes
are platform-specific and can be retrieved with `key_to_scan_codes`.

| Platform | Build | Runtime |
| --- | --- | --- |
| Windows 10 version 1607 or newer | MSVC `cl /std:c++17 /utf-8 /EHsc /I SRC example.cpp` links User32 automatically. With MinGW use `-std=c++17 -I SRC example.cpp -luser32`. | Hooks and injection operate in the caller's interactive desktop session. |
| Linux | `c++ -std=c++17 -pthread -I SRC example.cpp $(pkg-config --cflags --libs xkbcommon)` | Capture requires read access to keyboard devices under `/dev/input`. Injection requires write access to `/dev/uinput`. Linux uses the XKB rule defaults or the `XKB_DEFAULT_*` environment variables for layout selection. Unicode fallback uses the Ctrl+Shift+U input method and depends on application support. |
| macOS 10.15 or newer | `clang++ -std=c++17 -I SRC example.cpp -framework ApplicationServices -framework Carbon -framework CoreFoundation -framework CoreGraphics` | Grant Input Monitoring for observation. Grant Accessibility to inject or suppress input. |

`hook` and `hook_key` return IDs accepted by `unhook`. Their normal
callbacks run on the listener's processing thread. Returning `true` stops
dispatch to subsequent normal callbacks for that event. For a suppressing
hook, return `false` to discard the event. Suppressing callbacks run
synchronously and should return promptly. Windows supports suppressing
hooks on the default listener. On macOS, construct a
`Keyboard::KeyboardListener(Keyboard::KeyboardManager::instance(), true)`
to enable them. Linux evdev capture is observation-only.

`is_pressed` reports key state tracked by the listener as it receives
events. Input synthesized by the library itself is excluded from its hooks.
A listener starts on demand. Call `stop()` to close it and join its worker
threads. A callback can request a stop, but its listener must outlive the
callback. Startup errors are thrown by the registering call.
`rethrow_error()` reports asynchronous capture or callback failures.
