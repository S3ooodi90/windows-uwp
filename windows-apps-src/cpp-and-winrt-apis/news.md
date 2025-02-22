---
description: News and changes to C++/WinRT.
title: What's new in C++/WinRT
ms.date: 04/23/2019
ms.topic: article
keywords: windows 10, uwp, standard, c++, cpp, winrt, projection, news, what's, new
ms.localizationpriority: medium
ms.custom: RS5
---

# What's new in C++/WinRT

## News, and changes, in C++/WinRT 2.0

For more info about the [C++/WinRT Visual Studio Extension (VSIX)](https://aka.ms/cppwinrt/vsix), the [Microsoft.Windows.CppWinRT NuGet package](https://www.nuget.org/packages/Microsoft.Windows.CppWinRT/), and the the `cppwinrt.exe` tool&mdash;including how to acquire and install them&mdash;see [Visual Studio support for C++/WinRT, XAML, the VSIX extension, and the NuGet package](intro-to-using-cpp-with-winrt.md#visual-studio-support-for-cwinrt-xaml-the-vsix-extension-and-the-nuget-package).

### Changes to the C++/WinRT Visual Studio Extension (VSIX) for version 2.0

- The debug visualizer now supports Visual Studio 2019; as well as continuing to support Visual Studio 2017.
- Numerous bug fixes have been made.

### Changes to the Microsoft.Windows.CppWinRT NuGet package for version 2.0

- The `cppwinrt.exe` tool is now included in the Microsoft.Windows.CppWinRT NuGet package, and the tool generates platfom projection headers for each project on demand. Consequently, the `cppwinrt.exe` tool no longer depends on the Windows SDK (although, the tool still ships with the SDK for compatibility reasons).
- `cppwinrt.exe` now generates projection headers under each platform/configuration-specific intermediate folder ($IntDir) to enable parallel builds.
- The C++/WinRT build support (props/targets) is now fully documented, in case you want to manually customize your project files. See [Microsoft.Windows.CppWinRT NuGet Package](https://github.com/Microsoft/xlang/blob/master/src/package/cppwinrt/nuget/readme.md).
- Numerous bug fixes have been made.

### Changes to C++/WinRT for version 2.0

#### Open source

The `cppwinrt.exe` tool takes a Windows Runtime metadata (`.winmd`) file, and generates from it a header-file-based standard C++ library that *projects* the APIs described in the metadata. That way, you can consume those APIs from your C++/WinRT code.

This tool is now an entirely open source project, available on GitHub. Visit [Microsoft\/xlang](https://github.com/Microsoft/xlang), and then click in to **src** > **tool** > **cppwinrt**.

#### xlang libraries

A completely portable header-only library (for parsing the ECMA-335 metadata format used by the Windows Runtime) forms the basis of all the Windows Runtime and xlang tooling going forward. Notably, we also rewrote the `cppwinrt.exe` tool from the ground up using the xlang libraries. This provides far more accurate metadata queries, solving a few long-standing issues with the C++/WinRT language projection.

#### Fewer dependencies

Due to the xlang metadata reader, the `cppwinrt.exe` tool itself has fewer dependencies. This makes it far more flexible, as well as being usable in more scenarios&mdash;especially in constrained build environments. Notably, it no longer relies on `RoMetadata.dll`.
 
These are the dependencies for `cppwinrt.exe` 2.0.
 
- api-ms-win-core-processenvironment-l1-1-0.dll
- api-ms-win-core-libraryloader-l1-2-0.dll
- XmlLite.dll
- api-ms-win-core-memory-l1-1-0.dll
- api-ms-win-core-handle-l1-1-0.dll
- api-ms-win-core-file-l1-1-0.dll
- SHLWAPI.dll
- ADVAPI32.dll
- KERNEL32.dll
- api-ms-win-core-rtlsupport-l1-1-0.dll
- api-ms-win-core-processthreads-l1-1-0.dll
- api-ms-win-core-heap-l1-1-0.dll
- api-ms-win-core-console-l1-1-0.dll
- api-ms-win-core-localization-l1-2-0.dll

Contrasting with these dependencies, which `cppwinrt.exe` 1.0 has.

- ADVAPI32.dll
- SHELL32.dll
- api-ms-win-core-file-l1-1-0.dll
- XmlLite.dll
- api-ms-win-core-libraryloader-l1-2-0.dll
- api-ms-win-core-processenvironment-l1-1-0.dll
- RoMetadata.dll
- SHLWAPI.dll
- KERNEL32.dll
- api-ms-win-core-rtlsupport-l1-1-0.dll
- api-ms-win-core-heap-l1-1-0.dll
- api-ms-win-core-timezone-l1-1-0.dll
- api-ms-win-core-console-l1-1-0.dll
- api-ms-win-core-localization-l1-2-0.dll
- OLEAUT32.dll
- api-ms-win-core-winrt-error-l1-1-0.dll
- api-ms-win-core-winrt-error-l1-1-1.dll
- api-ms-win-core-winrt-l1-1-0.dll
- api-ms-win-core-winrt-string-l1-1-0.dll
- api-ms-win-core-synch-l1-1-0.dll
- api-ms-win-core-threadpool-l1-2-0.dll
- api-ms-win-core-com-l1-1-0.dll
- api-ms-win-core-com-l1-1-1.dll
- api-ms-win-core-synch-l1-2-0.dll 

#### The Windows Runtime `noexcept` attribute

The Windows Runtime has a new `[noexcept]` attribute, which you may use to decorate your methods and properties in [MIDL 3.0](/uwp/midl-3/predefined-attributes). The presence of the attribute indicates to supporting tools that your implementation doesn't throw an exception (nor return a failing HRESULT). This allows language projections to optimize code-generation by avoiding the exception-handling overhead that's required to support application binary interface (ABI) calls that can potentially fail.

C++/WinRT takes advantage of this by producing C++ `noexcept` implementations of both the consuming and authoring code. If you have API methods or properties that are fail-free, and you're concerned about code size, then you can investigate this attribute.

#### Optimized code-generation

C++/WinRT now generates even more efficient C++ source code (behind the scenes) so that the C++ compiler can produce the smallest and most efficient binary code possible. Many of the improvements are geared toward reducing the cost of exception-handling by avoiding unnecessary unwind information. Binaries that use large amounts of C++/WinRT code will see roughly a 4% reduction in code size. The code is also more efficient (it runs faster) due to the reduced instruction count.

These improvements rely on a new interop feature that's available to you, as well. All of the C++/WinRT types that are resource owners now include a constructor for taking ownership directly, avoiding the previous two-step approach.

```cppwinrt
ABI::Windows::Foundation::IStringable* raw = ...

IStringable projected(raw, take_ownership_from_abi);

printf("%ls\n", projected.ToString().c_str());
```

#### Optimized exception-handling (EH) code-generation

This change complements work that has been done by the Microsoft C++ optimizer team to reduce the cost of exception-handling. If you use application binary interfaces (ABIs) (such as COM) heavily in your code, then you'll observe a lot of code following this pattern.

```cpp
int32_t Function() noexcept
{
    try
    {
        // code here constitutes unique value.
    }
    catch (...)
    {
        // code here is always duplicated.
    }
}
```

C++/WinRT itself generates this pattern for every API that's implemented. With thousands of API functions, any optimization here can be significant. In the past, the optimizer wouldn't detect that those catch blocks are all identical, so it was duplicating a lot of code around each ABI (which in turn contributed to the belief that using exceptions in system code produces large binaries). However, from Visual Studio 2019 on, the C++ compiler folds all of those catch funclets, and only stores those that are unique. The result is a further and overall 18% reduction in code size for binaries that rely heavily on this pattern. Not only is EH code now more efficient than using return codes, but also the concern about larger binaries is now a thing of the past.

#### Incremental build improvements

The `cppwinrt.exe` tool now compares the output of a generated header/source file against the contents of any existing file on disk, and it only writes out the file if the file has in fact changed. This saves considerable time with disk I/O, and it ensures that the files are not considered "dirty" by the C++ compiler. The result is that recompilation is avoided, or reduced, in many cases.

#### Generic interfaces are now all generated

Due to the xlang metadata reader, C++/WinRT now generates all parameterized, or generic, interfaces from metadata. Interfaces such as [Windows::Foundation::Collections::IVector\<T\>](/uwp/api/windows.foundation.collections.ivector_t_) are now generated from metadata rather than hand-written in `winrt/base.h`. The result is that the size of `winrt/base.h` has been cut in half, and that optimizations are generated right into the code (which was tricky to do with the hand-rolled approach).

> [!IMPORTANT]
> Interfaces such as the example given now appear in their respective namespace headers, rather than in `winrt/base.h`. So, if you have not already done so, you'll have to include the appropriate namespace header in order to use the interface.

#### Component optimizations

This update adds support for several additional opt-in optimizations for C++/WinRT, described in the sections below. Because these optimizations are breaking changes (which you may need to make minor changes to support), you'll need to turn them on explicitly using the `cppwinrt.exe` tool's `-opt` flag.

A new project (from a project template) will use `-opt` by default.

##### Uniform construction, and direct implementation access

These two optimizations allow your component direct access to its own implementation types, even when it's only using the projected types. There's no need to use [**make**](/uwp/cpp-ref-for-winrt/make), [**make_self**](/uwp/cpp-ref-for-winrt/make-self), nor [**get_self**](/uwp/cpp-ref-for-winrt/get-self) if you simply want to use the public API surface. Your calls will compile down to direct calls into the implementation, and those might even be entirely inlined.

##### Type-erased factories

This optimization avoids the #include dependencies in `module.g.cpp` so that it need not be recompiled every time any single implementation class happens to change. The result is improved build performance.

#### Smarter and more efficient `module.g.cpp` for large projects with multiple libs

The `module.g.cpp` file now also contains two additional composable helpers, named **winrt_can_unload_now**, and **winrt_get_activation_factory**. These have been designed for larger projects where a DLL is composed of a number of libs, each with its own runtime classes. In that situation, you need to manually stitch together the DLL's **DllGetActivationFactory** and **DllCanUnloadNow**. These helpers make it much easier for you to do that, by avoiding spurious origination errors. The `cppwinrt.exe` tool's `-lib` flag may also be used to give each individual lib its own preamble (rather than `winrt_xxx`) so that each lib's functions may be individually named, and thus combined unambiguously.

#### Coroutine support

Coroutine support is included automatically. Previously, the support resided in multiple places, which we felt was too limiting. And then temporarily for v2.0, a `winrt/coroutine.h` header file was necessary, but that's no longer needed. Since the Windows Runtime async interfaces are now generated, rather than hand-written, they now reside in `winrt/Windows.Foundation.h`. Apart from being more maintainable and supportable, it means that coroutine helpers such as [**resume_foreground**](/uwp/cpp-ref-for-winrt/resume-foreground) no longer have to be tacked on to the end of a specific namespace header. Instead, they can more naturally include their dependencies. This further allows **resume_foreground** to support not only resuming on a given [**Windows::UI::Core::CoreDispatcher**](/uwp/api/windows.ui.core.coredispatcher), but it can now also support resuming on a given [**Windows::System::DispatcherQueue**](/uwp/api/windows.system.dispatcherqueue). Previously, only one could be supported; but not both, since the definition could only reside in one namespace.

Here's an example of the **DispatcherQueue** support.

```cppwinrt
...
#include <winrt/Windows.System.h>
using namespace Windows::System;
...
fire_and_forget Async(DispatcherQueueController controller)
{
    bool queued = co_await resume_foreground(controller.DispatcherQueue());
    assert(queued);

    // This is just to simulate queue failure...
    co_await controller.ShutdownQueueAsync();

    queued = co_await resume_foreground(controller.DispatcherQueue());
    assert(!queued);
}
```

The coroutine helpers are now also decorated with `[[nodiscard]]`, thereby improving their usability. If you forget to (or don't realize you have to) `co_await` them for them to work then, due to `[[nodiscard]]`, such mistakes now produce a compiler warning.

#### Help with diagnosing stack allocations

Since the projected and implementation class names are (by default) the same, and only differ by namespace, it's possible to mistake the one for the other, and to accidentally create an implementation on the stack, rather than using the [**make**](/uwp/cpp-ref-for-winrt/make) family of helpers. This can be hard to diagnose in some cases, because the object may be destroyed while outstanding references are still in flight. An assertion now picks this up, for debug builds. While the assertion doesn't detect stack allocation inside a coroutine, it's nevertheless helpful in catching most such mistakes.

#### Improved capture helpers, and variadic delegates

This update fixes the limitation with the capture helpers by supporting projected types as well. This comes up now and then with the Windows Runtime interop APIs, when they return a projected type.

This update also adds support for [**get_strong**](/uwp/cpp-ref-for-winrt/implements#implementsget_strong-function) and [**get_weak**](/uwp/cpp-ref-for-winrt/implements#implementsget_weak-function) when creating a variadic (non-Windows Runtime) delegate.

#### Support for deferred destruction and safe QI during destruction

A XAML application can get itself into difficulty due to its need to perform a [**QueryInterface**](/windows/desktop/api/unknwn/nf-unknwn-iunknown-queryinterface(q_)) (QI) in a destructor, in order to call some cleanup implementation up or down the hierarchy. But, that call involves a QI after the object's reference count has already reached zero. This update adds support for debouncing the reference count, ensuring that once it reaches zero it can never be resurrected; while still allowing QI for any temporary that's required during destruction. This procedure is unavoidable in certain XAML applications/controls, and C++/WinRT is now resilient to it.

Destruction can be deferred by providing a static **final_release** function, and moving ownership of the **unique_ptr** to some other context.

```cppwinrt
struct Sample : implements<Sample, IStringable>
{
    hstring ToString()
    {
        return L"Sample";
    }

    ~Sample()
    {
        // Called when the unique_ptr below is reset.
    }

    static void final_release(std::unique_ptr<Sample> ptr) noexcept
    {
        // Move 'ptr' as needed to delay destruction.
    }
};
```

In the example below, once the **MainPage** is released (for the final time), **final_release** is called. That function spends five seconds waiting (on the thread pool), and then it resumes using the page's **Dispatcher** (which requires QI/AddRef/Release to work). It then clears the **unique_ptr**, which causes the **MainPage** destructor to actually get called. Even here, **DataContext** is called, which requires a QI for **IFrameworkElement**. Obviously, you don't have to implement your **final_release** as a coroutine. But that does work, and it makes it very simple to move destruction to a different thread.

```cppwinrt
struct MainPage : PageT<MainPage>
{
    MainPage()
    {
    }

    ~MainPage()
    {
        DataContext(nullptr);
    }

    static IAsyncAction final_release(std::unique_ptr<MainPage> ptr)
    {
        co_await 5s;

        co_await resume_foreground(ptr->Dispatcher());

        ptr = nullptr;
    }
};
```

#### Improved support for COM-style single interface inheritance

As well as for Windows Runtime programming, C++/WinRT is also used to author and consume COM-only APIs. This update makes it possible to implement a COM server where there exists an interface hierarchy. This isn't required for the Windows Runtime; but it is required for some COM implementations.

#### Correct handling of `out` params

It can be tricky to work with `out` params; particularly Windows Runtime arrays. With this update, C++/WinRT is considerably more robust and resilient to mistakes when it comes to `out` params and arrays; whether those parameters arrive via a language projection, or from a COM developer who's using the raw ABI, and who's making the mistake of not initializing variables consistently. In either case, C++/WinRT now does the right thing when it comes to handing off projected types to the ABI (by remembering to release any resources), and when it comes to zeroing out or clearing out parameters that arrive across the ABI.

#### Events now handle invalid tokens reliably

The [**winrt::event**](/uwp/cpp-ref-for-winrt/event) implementation now gracefully handles the case where its **remove** method is called with an invalid token value (a value that's not present in the array).

#### Coroutine locals are now destroyed before the coroutine returns

The traditional way of implementing a coroutine type may allow locals within the coroutine to be destroyed *after* the coroutine returns/completes (rather than prior to final suspension). The resumption of any waiter is now deferred until final suspension, in order to avoid this problem and to accrue other benefits.

## News, and changes, in Windows SDK version 10.0.17763.0 (Windows 10, version 1809)

The table below contains news and changes for C++/WinRT in the Windows SDK version 10.0.17763.0 (Windows 10, version 1809).

| New or changed feature | More info |
| - | - |
| **Breaking change**. For it to compile, C++/WinRT doesn't depend on headers from the Windows SDK. | See [Isolation from Windows SDK header files](#isolation-from-windows-sdk-header-files), below. |
| The Visual Studio project system format has changed. | See [How to retarget your C++/WinRT project to a later version of the Windows SDK](#how-to-retarget-your-cwinrt-project-to-a-later-version-of-the-windows-sdk), below. |
| There are new functions and base classes to help you pass a collection object to a Windows Runtime function, or to implement your own collection properties and collection types. | See [Collections with C++/WinRT](collections.md). |
| You can use the [{Binding}](/windows/uwp/xaml-platform/binding-markup-extension) markup extension with your C++/WinRT runtime classes. | For more info, and code examples, see [Data binding overview](/windows/uwp/data-binding/data-binding-quickstart). |
| Support for canceling a coroutine allows you to register a cancellation callback. | For more info, and code examples, see [Canceling an asychronous operation, and cancellation callbacks](concurrency.md#canceling-an-asychronous-operation-and-cancellation-callbacks). |
| When creating a delegate pointing to a member function, you can establish a strong or a weak reference to the current object (instead of a raw *this* pointer) at the point where the handler is registered. | For more info, and code examples, see the **If you use a member function as a delegate** sub-section in the section [Safely accessing the *this* pointer with an event-handling delegate](weak-references.md#safely-accessing-the-this-pointer-with-an-event-handling-delegate). |
| Bugs are fixed that were uncovered by Visual Studio's improved conformance to the C++ standard. The LLVM and Clang toolchain is also better leveraged to validate C++/WinRT's standards conformance. | You'll no longer encounter the issue described in [Why won't my new project compile? I'm using Visual Studio 2017 (version 15.8.0 or higher), and SDK version 17134](faq.md#why-wont-my-new-project-compile-im-using-visual-studio-2017-version-1580-or-higher-and-sdk-version-17134) |

Other changes.

- **Breaking change**. [**winrt::get_abi(winrt::hstring const&)**](/uwp/cpp-ref-for-winrt/get-abi) now returns `void*` instead of `HSTRING`. You can use `static_cast<HSTRING>(get_abi(my_hstring));` to get an HSTRING.
- **Breaking change**. [**winrt::put_abi(winrt::hstring&)**](/uwp/cpp-ref-for-winrt/put-abi) now returns `void**` instead of `HSTRING*`. You can use `reinterpret_cast<HSTRING*>(put_abi(my_hstring));` to get an HSTRING*.
- **Breaking change**. HRESULT is now projected as **winrt::hresult**. If you need an HRESULT (to do type checking, or to support type traits), then you can `static_cast` a **winrt::hresult**. Otherwise, **winrt::hresult** converts to HRESULT, as long as you include `unknwn.h` before you include any C++/WinRT headers.
- **Breaking change**. GUID is now projected as **winrt::guid**. For APIs that you implement, you must use **winrt::guid** for GUID parameters. Otherwise, **winrt::guid** converts to GUID, as long as you include `unknwn.h` before you include any C++/WinRT headers.
- **Breaking change**. The [**winrt::handle_type constructor**](/uwp/cpp-ref-for-winrt/handle-type#handle_typehandle_type-constructor) has been hardened by making it explicit (it's now harder to write incorrect code with it). If you need to assign a raw handle value, call the [**handle_type::attach function**](/uwp/cpp-ref-for-winrt/handle-type#handle_typeattach-function) instead.
- **Breaking change**. The signatures of **WINRT_CanUnloadNow** and **WINRT_GetActivationFactory** have changed. You mustn't declare these functions at all. Instead, include `winrt/base.h` (which is automatically included if you include any C++/WinRT Windows namespace header files) to include the declarations of these functions.
- For the [**winrt::clock struct**](/uwp/cpp-ref-for-winrt/clock), **from_FILETIME/to_FILETIME** are deprecated in favor of **from_file_time/to_file_time**.
- APIs that expect **IBuffer** parameters are simplified. Although most APIs prefer collections or arrays, enough APIs rely on **IBuffer** that it needed to be easier to use such APIs from C++. This update provides direct access to the data behind an **IBuffer** implementation, using the same data naming convention used by the C++ Standard Library containers. This also avoids colliding with metadata names that conventionally begin with an uppercase letter.
- Improved code generation: various improvements to reduce code size, improve inlining, and optimize factory caching.
- Removed unnecessary recursion. When the command-line refers to a folder, rather than to a specific `.winmd`, the `cppwinrt.exe` tool no longer searches recursively for `.winmd` files. The `cppwinrt.exe` tool also now handles duplicates more intelligently, making it more resilient to user error, and to poorly-formed `.winmd` files.
- Hardened smart pointers. Formerly, the event revokers failed to revoke when move-assigned a new value. This helped uncover an issue where smart pointer classes weren't reliably handling self-assignment; rooted in the [**winrt::com_ptr struct template**](/uwp/cpp-ref-for-winrt/com-ptr). **winrt::com_ptr** has been fixed, and the event revokers fixed to handle move semantics correctly so that they revoke upon assignment.

> [!IMPORTANT]
> Important changes were made to the [C++/WinRT Visual Studio Extension (VSIX)](https://aka.ms/cppwinrt/vsix), both in version 1.0.181002.2, and then later in version 1.0.190128.4. For details of these changes, and how they affect your existing projects, [Visual Studio support for C++/WinRT](intro-to-using-cpp-with-winrt.md#visual-studio-support-for-cwinrt-xaml-the-vsix-extension-and-the-nuget-package) and  [Earlier versions of the VSIX extension](intro-to-using-cpp-with-winrt.md#earlier-versions-of-the-vsix-extension).

### Isolation from Windows SDK header files

This is potentially a breaking change for your code.

For it to compile, C++/WinRT no longer depends on header files from the Windows SDK. Header files in the C run-time library (CRT) and the C++ Standard Template Library (STL) also don't include any Windows SDK headers. And that improves standards compliance, avoids inadvertent dependencies, and greatly reduces the number of macros that you have to guard against.

This independence means that C++/WinRT is now more portable and standards compliant, and it furthers the possibility of it becoming a cross-compiler and cross-platform library. It also means that the C++/WinRT headers aren't adversely affected macros.

If you previously left it to C++/WinRT to include any Windows headers in your project, then you'll now need to include them yourself. It is, in any case, always best practice to explicitly include the headers that you depend on, and not leave it to another library to include them for you.

Currently, the only exceptions to Windows SDK header file isolation are for intrinsics, and numerics. There are no known issues with these last remaining dependencies.

In your project, you can re-enable interop with the Windows SDK headers if you need to. You might, for example, want to implement a COM interface (rooted in [**IUnknown**](https://docs.microsoft.com/windows/desktop/api/unknwn/nn-unknwn-iunknown)). For that example, include `unknwn.h` before you include any C++/WinRT headers. Doing so causes the C++/WinRT base library to enable various hooks to support classic COM interfaces. For a code example, see [Author COM components with C++/WinRT](author-coclasses.md). Similarly, explicitly include any other Windows SDK headers that declare types and/or functions that you want to call.

### How to retarget your C++/WinRT project to a later version of the Windows SDK

The method for retargeting your project that's likely to result in the fewest compiler and linker issue is also the most labor-intensive. That method involves creating a new project (targeting the Windows SDK version of your choice), and then copying files over to your new project from your old. There will be sections of your old `.vcxproj` and `.vcxproj.filters` files that you can just copy over to save you adding files in Visual Studio.

However, there are two other ways to retarget your project in Visual Studio.

- Go to project property **General** \> **Windows SDK Version**, and select **All Configurations** and **All Platforms**. Set **Windows SDK Version** to the version that you want to target.
- In **Solution Explorer**, right-click the project node, click **Retarget Projects**, choose the version(s) you wish to target, and then click **OK**.

If you encounter any compiler or linker errors after using either of these two methods, then you can try cleaning the solution (**Build** > **Clean Solution** and/or manually delete all temporary folders and files) before trying to build again.

If the C++ compiler produces "*error C2039: 'IUnknown': is not a member of '\`global namespace''*", then add `#include <unknwn.h>` to the top of your `pch.h` file (before you include any C++/WinRT headers).

You may also need to add `#include <hstring.h>` after that.

If the C++ linker produces "*error LNK2019: unresolved external symbol _WINRT_CanUnloadNow@0 referenced in function _VSDesignerCanUnloadNow@0*", then you can resolve that by adding `#define _VSDESIGNER_DONT_LOAD_AS_DLL` to your `pch.h` file.
