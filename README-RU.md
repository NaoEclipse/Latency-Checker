## Translations
- [English](README.md)

### Latency Checker 🚀

---

#### Описание:
Эта программа демонстрирует создание и измерение системной нагрузки на C++ с использованием многопоточности. Она создает фоновый поток, который непрерывно выполняет вычисления, требующие больших вычислительных ресурсов, в то время как основной поток измеряет точность длительностей сна.

## Код

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <chrono>
#include <atomic>
#include <windows.h>

std::atomic<bool> keep_running(true);

void create_load() {
    while (keep_running) {
        volatile int x = 0;
        for (int i = 0; i < 50000; ++i) {
            x += i * i * i;
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
}

void measure_sleep_time(std::chrono::milliseconds duration) {
    for (int i = 0; i < 1; ++i) {
        auto start_time = std::chrono::high_resolution_clock::now();
        std::this_thread::sleep_for(duration);
        auto end_time = std::chrono::high_resolution_clock::now();
        
        std::chrono::duration<double, std::milli> slept_time = end_time - start_time;
        double delta = slept_time.count() - duration.count();
        
        std::cout << "Sleep(" << duration.count() << "ms) slept " << slept_time.count() 
                  << "ms (delta: " << delta << "ms)" << std::endl;
    }
}

int main() {
    timeBeginPeriod(1);
    
    SetThreadPriority(GetCurrentThread(), THREAD_PRIORITY_TIME_CRITICAL);

    SetPriorityClass(GetCurrentProcess(), REALTIME_PRIORITY_CLASS);

    std::thread load_thread([](){
        SetThreadPriority(GetCurrentThread(), THREAD_PRIORITY_TIME_CRITICAL);
        create_load();
    });

    std::cout << "Ctrl+C to stop." << std::endl;
    
    try {
        while (true) {
            measure_sleep_time(std::chrono::milliseconds(1));
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    } catch (...) {
        std::cout << "\nStopping." << std::endl;
        keep_running = false;
        load_thread.join();
    }
    
    timeEndPeriod(1);
    
    return 0;
}
```
