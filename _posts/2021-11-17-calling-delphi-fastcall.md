---
layout: post
title:  Calling an unexported Delphi fastcall function from C++
categories: [Reverse Engineering]
excerpt: I recently found myself needing to call an unexported function in a Delphi exe from C++. A disassembler told me that the function used the fastcall calling convention, but I soon realized that it isn't the same as the fastcall calling convention in MSVC.
---

I recently found myself needing to call an unexported function in a Delphi exe from C++. A disassembler told me that the function used the fastcall calling convention, but I soon realized that it isn't the same as the fastcall calling convention in MSVC. As described on [wikipedia](https://en.wikipedia.org/wiki/X86_calling_conventions#Borland_register){:target="_blank"}, Borland Delphi uses a bespoke register-based calling convention. While GCC and Clang can be coerced into using this convention, I found no way to get the Microsoft compiler to do it.

I had two problems to solve. First, how to call an unexported function from an exe. Second, how to leverage the Borland Register calling convention to call said function.

Solving the first problem wasn't too bad, since this function didn't have any dependencies. The plan of attack was simple: Read the exe into an array, allocate some memory for the function, create a pointer to it, and call it. The tricky bit here is to ensure that the allocated memory is flagged as executable, so you don't get an access violation when trying to call into it.

```cpp
int sub_69B790_proxy(void* a1, void* a2){
	char* startpos;
	void* funcaddr = 0;


	char* codeBuffer = 0;
	long length;
	FILE* f = fopen("target.exe", "rb");

	if (f)
	{
		fseek(f, 0, SEEK_END);
		length = ftell(f);
		fseek(f, 0, SEEK_SET);
		codeBuffer = (char*)malloc(length);
		if (codeBuffer)
		{
			fread(codeBuffer, 1, length, f);
		}
		fclose(f);
	}

	funcaddr = codeBuffer + 0x29ab90; // 0x29ab90 is the offset into the exe at which the function is located, ascertained from a disassembler


	if (codeBuffer)
	{
		SYSTEM_INFO system_info;
		GetSystemInfo(&system_info);
		auto const page_size = system_info.dwPageSize;

		
		auto const buffer = VirtualAlloc(nullptr, page_size, MEM_COMMIT, PAGE_EXECUTE_READWRITE);

		std::memcpy(buffer, funcaddr, 0x8D8); // 0x8D is the size of the function, ascertained from a disassembler

		free(codeBuffer);

		// TODO: Execute the function at buffer

		VirtualFree(buffer, 0, MEM_RELEASE);

	}
}
```

There are some obvious optimizations to the code above: The intial file read is old style C, and instead of reading the whole file, we could just read the target function and read it directly into our executable memory block. The key takeaway here though, is that we're reading a chunk of an executable from disk into executable memory, and are prepared to execute it ourselves.

Or we would be, except for the second problem: we know it's the Borland Register calling convention, and we have no way of expressing that in the Microsoft compiler. One option would be to patch the function in memory to use a different calling convention. This particular method was messy enough that I didn't want to do that. Instead, I decided to use a small bit of inline assembly to set up the registers and call the function myself:

```cpp
	__asm {
			mov eax, [a1]
			mov edx, [a2]
			call[buffer]
		}

	
```

In the Borland Register calling convention, the first argument goes into `eax`, the second in `edx`. That's exactly what I've done, copying the two function arguments into those registers, and then executing the `call` directly to the buffer memory. When the call comes back, the return value would be in `eax` if I needed it, though in this particular case I didn't.

That's all there is to it. I was lucky that the function I was calling only had two arguments, but more would not be insurmountable. I was especially lucky that this function had no absolute jumps or nested function calls, because those would not have worked with this method (at least not without patching the functions).