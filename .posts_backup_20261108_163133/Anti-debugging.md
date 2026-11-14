---
title: Anti-debugging
date: 2026-11-08 15:47:18
tags: [反调试, 动态调试]
categories: 笔记
---

## 第一课：认识反调试

反调试 = 程序不想让你用调试器分析它，所以设置了各种"陷阱"来检测你有没有在调试。

- 花指令 = 房间里故意堆满垃圾，让你看不清路（干扰静态分析，你之前学过）
- 反调试 = 房间里装了摄像头和警报器，一旦发现你（调试器），就锁门或自毁（干扰动态分析）

| 方式 | 通俗解释 | 例子 |
| --- | --- | --- |
| **查内存** | 翻自己的"档案"，看有没有被调试器动过手脚 | 看 `BeingDebugged` 标志 |
| **设陷阱** | 故意做一件会触发警报的事，看有没有人处理 | 故意触发异常，看调试器是否拦截 |

## 第二课：API 反调试 — IsDebuggerPresent

### 一句话

程序直接打电话给操作系统："帮我查一下，有没有人在调试我？"

### 具体过程

```c
// 程序里的代码
if (IsDebuggerPresent()) {    // ← 打电话问系统
    exit(1);                   // ← 有调试器？退出！
}
```

### 这个电话打给谁？

- 打给 kernel32.dll（Windows 的一个系统文件）
- 具体打给里面的 IsDebuggerPresent 函数

### 系统怎么回答？

系统去翻一个内部档案（PEB 结构体），看里面 BeingDebugged 这个字段：

- BeingDebugged = 1 → 有人调试 → 返回 1（真）
- BeingDebugged = 0 → 没人调试 → 返回 0（假）

### 你在 IDA 里怎么找到它？

```text
方法1：Shift+F12 搜字符串 "Debugger"
方法2：Imports 窗口搜 "IsDebuggerPresent"
方法3：x64dbg 运行，程序退出了，重新单步跟踪找退出点
```

### 找到了怎么办？三种绕过方法

| 方法 | 你做什么 | 效果 |
| --- | --- | --- |
| Patch | 把 if (v1) 改成永远走"无调试器"分支 | 改文件，永久有效 |
| Hook | 截住 IsDebuggerPresent 的返回，强制改 0 | 不改文件，运行时欺骗 |
| 插件 | x64dbg + ScyllaHide，一键勾选 | 自动处理 |

## 第三课：CheckRemoteDebuggerPresent 和 PEB 反调试

### 先讲 CheckRemoteDebuggerPresent

#### 一句话

IsDebuggerPresent 只能查"我自己"有没有被调试，CheckRemoteDebuggerPresent 可以查"别的进程"有没有被调试。

#### 区别

| API | 查谁 | 参数 |
| --- | --- | --- |
| IsDebuggerPresent() | 当前进程 | 无参数 |
| CheckRemoteDebuggerPresent(hProcess, &result) | 指定进程 | 进程句柄 + 结果指针 |

#### 代码长什么样

```c
BOOL result;
CheckRemoteDebuggerPresent(GetCurrentProcess(), &result);
if (result) {
    exit(1);
}
```

GetCurrentProcess() = 拿到自己的进程句柄，所以实际上还是在查自己。

#### 怎么绕过？

和 IsDebuggerPresent 一样：

- Patch 跳转
- Hook API 强制返回 0
- ScyllaHide

### 再讲 PEB 反调试

#### 一句话

IsDebuggerPresent 本质上就是去读 PEB 里的 BeingDebugged 字段。但程序也可以不打电话，直接去翻 PEB 这个"档案"。

#### PEB 是什么？

Process Environment Block = 进程环境块，是 Windows 给每个进程维护的一个内存里的结构体，里面存了很多进程信息。

#### 怎么找到 PEB？

```text
FS:[0x30]  →  指向 PEB 的地址
（在 64 位程序里是 GS:[0x60]）
```

#### PEB 里和反调试相关的三个字段

| 字段 | 偏移 | 正常值 | 被调试时的值 |
| --- | --- | --- | --- |
| BeingDebugged | +0x002 | 0 | 1 |
| ProcessHeap.Flags | +0x018 再 +0x0C | 2 | 变 |
| ProcessHeap.ForceFlags | +0x018 再 +0x10 | 0 | 变 |
| NtGlobalFlag | +0x068 | 0 | 0x70 |

#### 程序怎么自己查 PEB？

```asm
; 汇编代码
mov eax, fs:[0x30]      ; eax = PEB 地址
movzx eax, byte ptr [eax+2]  ; eax = BeingDebugged 的值 (0 或 1)
test eax, eax
jnz debugger_detected   ; 如果是1，说明有调试器
```

#### 你在 IDA 里怎么识别 PEB 反调试？

看伪代码里有没有：

- fs:30h 或 gs:60h
- 对某个地址 +2、-0x68 的读取
- 常量 0x70（NtGlobalFlag 被调试时的值）

### 常见反调试症状速查

| 现象 | 为什么怀疑反调试 | 验证方法 |
| --- | --- | --- |
| **程序在 x64dbg 里直接退出，双击正常运行** | 最经典的反调试症状 | 在 `ExitProcess` 下断点，看调用栈 |
| **下断点就崩溃** | 可能检测了断点（INT3）或硬件断点 | 换硬件断点试试，或静态分析 |
| **单步执行极慢，但直接运行飞快** | 时间检测反调试 | 在 `GetTickCount`/`QueryPerformanceCounter` 下断点 |
| **输入正确答案但显示错误** | 程序走了错误分支 | 静态分析检查逻辑，看有没有条件跳转被反调试控制 |
| **程序启动瞬间消失，看不到窗口** | TLS 回调里退出了 | x64dbg 设置暂停于"系统断点"或"TLS回调" |
| **换台电脑/换调试器就好了** | 进程名/窗口名检测 | 改调试器进程名，或隐藏窗口 |

## 第四课：TLS 反调试 — 在 main 之前就动手

### 一句话

正常反调试在 main() 里执行，TLS 反调试在 main() 之前就执行了，让你防不胜防。

### 为什么 main() 之前还能执行代码？

程序启动流程：

```text
操作系统加载程序 → 执行 TLS 回调 → 执行 main()
                    ↑
                 在这里！
```

TLS = Thread Local Storage（线程局部存储），本来是给每个线程存私有数据的机制。

但 Windows 设计了一个回调机制：每次创建线程（包括程序启动时的主线程），都会先调用你指定的 TLS 回调函数。

### 代码长什么样

```c
// TLS 回调函数 — 在 main 之前执行！
void NTAPI TLS_Callback(PVOID DllHandle, DWORD Reason, PVOID Reserved) {
    if (Reason == DLL_PROCESS_ATTACH) {      // 程序刚加载
        if (IsDebuggerPresent()) {           // 查调试器
            ExitProcess(1);                   // 有调试器？直接退出！
        }
    }
}

// 把这个函数注册到 TLS 回调表
#pragma const_seg(".CRT$XLB")
EXTERN_C const PIMAGE_TLS_CALLBACK pTLS_CALLBACK = TLS_Callback;
#pragma const_seg()
```

### 关键问题：你在调试器里怎么发现它？

| 情况 | 现象 | 怎么办 |
| --- | --- | --- |
| 正常反调试 | 在 main 里下断点，能断住，然后发现检测逻辑 | 直接 Patch |
| TLS 反调试 | 还没走到 main，程序就退出了 | 要找到 TLS 回调，在那里下断点 |

### 怎么找 TLS 回调？

#### 方法1：IDA 里看

```text
IDA → View → Open subviews → Segments
→ 找 .tls 段
→ 或者看 Directory Entry: TLS
```

#### 方法2：x64dbg 里看

```text
x64dbg → 选项 → 事件 → 设置第一次暂停于 "系统断点" 或 "TLS 回调"
→ 这样程序会在 TLS 回调处停下，而不是 main
```

#### 方法3：PE 工具看

```text
用 CFF Explorer 打开 exe
→ Data Directories → TLS Directory
→ 看 AddressOfCallBacks 指向的函数地址
```

### 怎么绕过？

和之前一样，找到 TLS 回调里的检测逻辑，Patch 掉。

但难点是：你要先找到它在哪里，因为它不在 main 里。

## 第五课：进程名反调试 — 程序扫描你电脑里有没有开调试器

### 一句话

程序不查自己，而是查"整个电脑里有没有人在跑调试器"。

### 具体过程

```c
// 程序里的代码
bool IsDebuggerProcessRunning() {
    // 已知的调试器名单
    const char* debuggerNames[] = { 
        "ollydbg.exe",      // OllyDbg
        "x64dbg.exe",       // x64dbg
        "ida.exe",          // IDA
        "windbg.exe"        // WinDbg
    };

    // 拍一张"当前所有进程"的快照
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

    // 逐个遍历进程
    PROCESSENTRY32 pe32;
    Process32First(hSnapshot, &pe32);
    do {
        // 对比进程名
        for (每个 debuggerNames) {
            if (进程名 == 调试器名) {
                return true;  // 找到了！有调试器！
            }
        }
    } while (Process32Next(hSnapshot, &pe32));

    return false;  // 没找到
}

int main() {
    if (IsDebuggerProcessRunning()) {
        ExitProcess(1);  // 有调试器？退出！
    }
    // 正常逻辑...
}
```

### 关键 API

| API | 干嘛的 |
| --- | --- |
| CreateToolhelp32Snapshot | 给当前所有进程拍一张"快照" |
| Process32First | 拿到第一个进程的信息 |
| Process32Next | 拿到下一个进程的信息 |
| pe32.szExeFile | 进程的可执行文件名 |

### 你在 IDA 里怎么识别？

看 Imports 里有没有 CreateToolhelp32Snapshot，然后看伪代码里是不是在遍历数组、对比字符串。

### 怎么绕过？

| 方法 | 具体操作 |
| --- | --- |
| 改调试器进程名 | 把 x64dbg.exe 改成 abc.exe |
| 隐藏调试器进程 | 用插件或驱动级工具 |
| Patch 程序 | 找到对比逻辑，NOP 掉或改跳转 |
| Hook API | 让 Process32First/Process32Next 跳过调试器进程 |

---

## 第六课：窗口名反调试 — 扫描窗口标题

### 一句话

不查进程名，查窗口标题。调试器窗口开着，标题里可能有 "x64dbg"、"OllyDbg" 等字样。

### 关键 API

| API | 干嘛的 |
| --- | --- |
| EnumWindows | 遍历所有顶层窗口 |
| GetWindowTextA | 拿到窗口标题 |
| strstr | 检查标题里是否包含调试器名字 |

### 怎么绕过？

和进程名一样：

- 改调试器窗口标题
- Patch 程序
- Hook EnumWindows 让它永远返回"没找到"

---

## 第七课：时间戳反调试 — 检测"你太慢了"

### 一句话

调试器单步执行会让程序变慢。程序记录两条指令之间的时间，如果差值太大，说明有人在慢慢看。

### 三种计时方式

| 方式 | 精度 | API/指令 |
| --- | --- | --- |
| rdtsc | 纳秒级 | CPU 指令，读时间戳计数器 |
| QueryPerformanceCounter | 微秒级 | Windows API |
| GetTickCount | 毫秒级 | Windows API |

### 代码长什么样

```c
DWORD time1 = GetTickCount();   // 记录开始时间

// 要检测的代码（比如一段空操作或关键逻辑）
__asm { nop; nop; nop; }

DWORD time2 = GetTickCount();   // 记录结束时间

if (time2 - time1 > 0x10) {     // 如果超过 16 毫秒
    ExitProcess(0);              // 太慢了！有调试器！
}
```

### 正常执行 vs 调试执行

| 场景 | 时间差 |
| --- | --- |
| 正常跑 | 几微秒，远小于 16ms |
| 单步调试 | 你按一次 F8 可能隔几秒，远大于 16ms |

### 怎么绕过？

| 方法 | 具体操作 |
| --- | --- |
| 下断点改返回值 | 在 GetTickCount 下断点，第二次调用时改成第一次的值 + 一个很小的数 |
| Hook API | 让 GetTickCount 永远返回一个固定值，或返回差值很小的值 |
| Patch | 把 if (差值 > 0x10) 改成 if (0) 或改跳转 |
| 用硬件断点 | 不在检测代码里下软件断点，避免触发时间差 |

---

## 第八课：硬件断点检测 — 查调试器的"隐形钩子"

### 一句话

x64dbg 的硬件断点存在 CPU 的调试寄存器里（DR0~DR7）。程序可以直接读这些寄存器，发现有值就说明有硬件断点。

### 关键代码

```c
CONTEXT ctx = {};
ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;  // 我要读调试寄存器
GetThreadContext(GetCurrentThread(), &ctx);   // 拿到当前线程上下文

if (ctx.Dr0 || ctx.Dr1 || ctx.Dr2 || ctx.Dr3) {
    // DR0~DR3 存的是硬件断点地址，有值 = 有硬件断点
    ExitProcess(0);
}
```

### 怎么绕过？

| 方法 | 具体操作 |
| --- | --- |
| 不用硬件断点 | 改用软件断点（INT3） |
| Patch | 找到 GetThreadContext 调用，NOP 掉或改跳转 |
| Hook | 让 GetThreadContext 返回的 Dr0~Dr3 都是 0 |

---

## 第九课：异常处理反调试 — 故意触发异常看有没有人接

### 一句话

程序故意触发一个异常（比如 int 3），如果没调试器，操作系统会按正常流程处理；如果有调试器，调试器会先拦截。程序通过观察"谁处理了异常"来判断有没有调试器。

### 代码长什么样

```c
__try {
    __asm { int 3 }   // 故意触发断点异常
} 
__except (EXCEPTION_EXECUTE_HANDLER) {
    // 如果走到这里，说明异常被程序自己的处理函数捕获了
    // = 没有调试器拦截
    printf("No debugger\n");
}
```

### 正常情况 vs 调试情况

| 场景 | 谁处理 int 3 |
| --- | --- |
| 无调试器 | 程序的 __except 捕获，继续执行 |
| 有调试器 | 调试器先拦截，停在断点处 |

### 怎么绕过？

在 x64dbg 里设置异常处理策略：

```text
选项 → 异常 → 把 "INT3" 和 "BreakPoint" 设为 "传递给程序"
```

让调试器不要拦截，交给程序自己处理。

---

## 第十课：单步检测反调试 — 查 CPU 的"单步模式"

### 一句话

CPU 有个标志叫 Trap Flag（TF），设为 1 时每执行一条指令就触发一次调试中断。调试器单步执行时会设这个标志，程序可以检测它。

### 关键代码

```asm
pushfd          ; 把 EFLAGS 寄存器压栈
pop eax         ; 弹到 eax
or eax, 0x100   ; 把第 8 位（TF）设为 1
push eax        ; 压回栈
popfd           ; 恢复 EFLAGS，TF=1

nop             ; 执行一条指令

pushfd          ; 再看 EFLAGS
pop eax
test eax, 0x100 ; TF 还是 1 吗？
jz debugger_detected  ; 如果变成 0 了，说明调试器清掉了它
```

### 逻辑

| 情况 | TF 变化 | 原因 |
| --- | --- | --- |
| 无调试器 | TF 保持 1 | 没人动它 |
| 有调试器单步 | TF 被清 0 | 调试器处理单步异常后复位 TF |

### 怎么绕过？

在 x64dbg 里：

```text
选项 → 异常 → 把 "Single Step" 设为 "传递给程序"
```

或者 Patch 掉检测逻辑。
