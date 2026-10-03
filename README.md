# EldenRing-PerformanceTweaks

![Version](https://img.shields.io/badge/version-1.1.0-blue)
![Platform](https://img.shields.io/badge/platform-Windows%20x64-lightgrey)
![License](https://img.shields.io/badge/license-MIT-green)

[English](#english) | [Русский](#русский)

## English

A small DLL for Elden Ring that changes process-level Windows settings after the game starts: priority, timer resolution, power throttling, sleep prevention and, optionally, CPU affinity and working set size. It does not patch game code or game memory.

Written in Rust, about 300 KB, no dependencies besides Windows API. Tested on `eldenring.exe` 2.7.1 with me3 0.13.0 and The Convergence.

> [!WARNING]
> DLL mods do not load with Easy Anti-Cheat running. Play offline with EAC disabled (Mod Engine and me3 do this for you), or use Seamless Co-op.

### What it does

The mod starts one worker thread, waits for the game window and applies the settings once. The frame loop is not touched.

Always applied:

- `SetErrorMode` to suppress system error dialogs.
- Raised memory priority and high I/O priority hint for the process.
- A system beep when everything has been applied.

Controlled by the config:

- Process priority class (`PriorityLevel`).
- Timer resolution of 0.5 ms through `NtSetTimerResolution`, falling back to `timeBeginPeriod(1)` (`HighPrecisionTimer`).
- Power throttling off for the process (`DisableThrottling`).
- No sleep or display off while the game runs (`PreventSleep`).
- Affinity mask without core 0 (`BypassCore0`).
- Minimum working set (`OptimizeWorkingSet`, `WorkingSetMinMB`).
- MMCSS registration of the mod's worker thread under the chosen profile (`MMCSSProfile`). Game threads are not registered.

How much this helps depends on the machine. Expect smoother frame pacing on systems where the game was throttled or shared a core with background work, and no visible change elsewhere.

### Installation

Download `elden_ring_performance_tweaks.dll` and `er_performance_tweaks_config.ini` from the [`release`](release) folder.

**me3**: put both files into the `dll` folder of your mod and add the DLL to the profile:

```toml
[[natives]]
path = './../mod/dll/elden_ring_performance_tweaks.dll'
```

**Mod Engine 2**: add the DLL path to `external_dlls` in `config_eldenring.toml`.

**[Elden Mod Loader](https://www.nexusmods.com/eldenring/mods/117)**: put both files into `ELDEN RING\Game\mods`.

The config is looked up next to the DLL first, then next to `eldenring.exe`. If it is missing, the defaults from the table below are used.

After a launch, check `er_performance_tweaks_log.log` next to the DLL. The file is rewritten on every start and lists each applied setting.

### Configuration

| Section | Key | Default | Description |
| --- | --- | --- | --- |
| `General` | `EnableLogging` | `false` | Adds the final summary line to the log. The log itself is always written. |
| `General` | `SmartWait` | `true` | Wait for the game window (up to 40 s) and apply the settings one second later. |
| `General` | `InitDelay` | `3` | Delay in seconds, used only when `SmartWait` is off. |
| `General` | `WindowTitle` | empty | Window title to wait for. Empty means `ELDEN RING`. |
| `CPU` | `PriorityLevel` | `1` | 0 Normal, 1 Above Normal, 2 High, 3 Realtime. Realtime can freeze the system. |
| `CPU` | `BypassCore0` | `false` | Keep the game off core 0. Leave `false` on AMD Ryzen. |
| `CPU` | `PreferPCores` | `true` | Reserved. The value is read, but 1.1.0 does nothing with it. |
| `Optimization` | `HighPrecisionTimer` | `true` | 0.5 ms timer resolution. |
| `Optimization` | `MMCSSProfile` | `Games` | MMCSS task name: `Games` or `Pro Audio`. |
| `Optimization` | `OptimizeWorkingSet` | `false` | Raise the minimum working set to reduce paging. |
| `Optimization` | `WorkingSetMinMB` | `0` | Minimum working set in MB. 0 picks 8, 4, 2 or 1 GB by installed RAM. |
| `Power` | `DisableThrottling` | `true` | Turn off power throttling for the process. |
| `Power` | `PreventSleep` | `true` | Keep the system and display awake. |

The shipped ini sets `EnableLogging = true` and `PreferPCores = false`; the rest matches the defaults.

### Building

Rust with the `x86_64-pc-windows-msvc` or `x86_64-pc-windows-gnu` target:

```
cargo build --release
```

The DLL appears in `target\release\elden_ring_performance_tweaks.dll`.

### License

MIT, see [LICENSE](LICENSE). Not affiliated with FromSoftware or Bandai Namco.

## Русский

Небольшая DLL для Elden Ring. После запуска игры меняет настройки процесса в Windows: приоритет, разрешение таймера, троттлинг питания, запрет сна, по желанию привязку к ядрам и размер рабочего набора. Код и память игры не трогает.

Написана на Rust, около 300 КБ, зависит только от Windows API. Проверена на `eldenring.exe` 2.7.1 с me3 0.13.0 и The Convergence.

> [!WARNING]
> С включённым Easy Anti-Cheat DLL-моды не загружаются. Играйте офлайн с отключённым EAC (Mod Engine и me3 делают это сами) или через Seamless Co-op.

### Что делает

Мод запускает один рабочий поток, ждёт окно игры и применяет настройки один раз. В покадровый цикл не вмешивается.

Применяется всегда:

- `SetErrorMode` — системные окна ошибок не показываются.
- Повышенный приоритет памяти и высокий приоритет ввода-вывода для процесса.
- Системный звуковой сигнал, когда всё применено.

Управляется конфигом:

- Класс приоритета процесса (`PriorityLevel`).
- Разрешение таймера 0,5 мс через `NtSetTimerResolution`, запасной вариант — `timeBeginPeriod(1)` (`HighPrecisionTimer`).
- Отключение троттлинга питания для процесса (`DisableThrottling`).
- Запрет сна и отключения экрана, пока идёт игра (`PreventSleep`).
- Маска ядер без ядра 0 (`BypassCore0`).
- Минимальный рабочий набор памяти (`OptimizeWorkingSet`, `WorkingSetMinMB`).
- Регистрация рабочего потока мода в MMCSS с выбранным профилем (`MMCSSProfile`). Потоки игры не регистрируются.

Эффект зависит от машины. Там, где игру притормаживала система или она делила ядро с фоновыми задачами, кадры идут ровнее; в остальных случаях разницы может не быть.

### Установка

Возьмите `elden_ring_performance_tweaks.dll` и `er_performance_tweaks_config.ini` из папки [`release`](release).

**me3**: положите оба файла в папку `dll` мода и добавьте DLL в профиль:

```toml
[[natives]]
path = './../mod/dll/elden_ring_performance_tweaks.dll'
```

**Mod Engine 2**: добавьте путь к DLL в `external_dlls` в `config_eldenring.toml`.

**[Elden Mod Loader](https://www.nexusmods.com/eldenring/mods/117)**: положите оба файла в `ELDEN RING\Game\mods`.

Конфиг ищется сначала рядом с DLL, потом рядом с `eldenring.exe`. Если его нет, берутся значения по умолчанию из таблицы ниже.

После запуска посмотрите `er_performance_tweaks_log.log` рядом с DLL. Файл перезаписывается при каждом старте, в нём перечислена каждая применённая настройка.

### Настройка

| Раздел | Ключ | По умолчанию | Описание |
| --- | --- | --- | --- |
| `General` | `EnableLogging` | `false` | Добавляет в лог итоговую строку. Сам лог пишется всегда. |
| `General` | `SmartWait` | `true` | Ждать окно игры (до 40 с) и применить настройки через секунду. |
| `General` | `InitDelay` | `3` | Задержка в секундах, работает только при выключенном `SmartWait`. |
| `General` | `WindowTitle` | пусто | Заголовок окна, которое ждать. Пусто — `ELDEN RING`. |
| `CPU` | `PriorityLevel` | `1` | 0 обычный, 1 выше среднего, 2 высокий, 3 реального времени. С последним система может зависнуть. |
| `CPU` | `BypassCore0` | `false` | Не пускать игру на ядро 0. На AMD Ryzen оставьте `false`. |
| `CPU` | `PreferPCores` | `true` | Зарезервирован. Значение читается, но в 1.1.0 не используется. |
| `Optimization` | `HighPrecisionTimer` | `true` | Разрешение таймера 0,5 мс. |
| `Optimization` | `MMCSSProfile` | `Games` | Имя задачи MMCSS: `Games` или `Pro Audio`. |
| `Optimization` | `OptimizeWorkingSet` | `false` | Поднять минимальный рабочий набор, чтобы меньше уходило в подкачку. |
| `Optimization` | `WorkingSetMinMB` | `0` | Минимальный рабочий набор в МБ. 0 — 8, 4, 2 или 1 ГБ в зависимости от объёма ОЗУ. |
| `Power` | `DisableThrottling` | `true` | Отключить троттлинг питания для процесса. |
| `Power` | `PreventSleep` | `true` | Не давать системе и экрану засыпать. |

В поставляемом ini стоит `EnableLogging = true` и `PreferPCores = false`, остальное совпадает со значениями по умолчанию.

### Сборка

Нужен Rust с целью `x86_64-pc-windows-msvc` или `x86_64-pc-windows-gnu`:

```
cargo build --release
```

DLL появится в `target\release\elden_ring_performance_tweaks.dll`.

### Лицензия

MIT, см. [LICENSE](LICENSE). Проект не связан с FromSoftware и Bandai Namco.
