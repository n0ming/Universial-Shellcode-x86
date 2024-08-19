# Universial-Shellcode-x86
WinExec로 Hello.exe를 실행하는 방법은 다음 아래 과정과 같습니다.
첫번째로 수정해야할 점은 Compute_Hash 함수 call 이후 add해주는 값입니다.
현재 CreateProcessA와 달리 WinExec는 0x8 크기이기에 0x10에서 0x8로 바꿔야합니다.

그리고 WinExec함수는 다음 아래의 형태를 구성하고 있습니다. 
    UINT WINAPI WinExec(
      _In_ LPCSTR lpCmdLine,
      _In_ UINT   uCmdShow
    );
IpCmdLine은 Hello.exe의 주소가 들어가야하며, uCmdShow는 SW_SHOWNORMAL으로 1을 넣었습니다. 이에 맞춰 작성한 코드가 아래입니다.
    push 1 // SW_SHOWNORMAL
    push DWORD PTR[ebp - 0x3C] // lpCmdLine

그렇기에 CreateProcessA의 인자로 작성한 아래 코드를 모두 WinExec함수 인자로 바꿔줘야 합니다.
아래가 CreateProcessA의 형태입니다.
    BOOL CreateProcessA(
      [in, optional]      LPCSTR                lpApplicationName,
      [in, out, optional] LPSTR                 lpCommandLine,
      [in, optional]      LPSECURITY_ATTRIBUTES lpProcessAttributes,
      [in, optional]      LPSECURITY_ATTRIBUTES lpThreadAttributes,
      [in]                BOOL                  bInheritHandles,
      [in]                DWORD                 dwCreationFlags,
      [in, optional]      LPVOID                lpEnvironment,
      [in, optional]      LPCSTR                lpCurrentDirectory,
      [in]                LPSTARTUPINFOA        lpStartupInfo,
      [out]               LPPROCESS_INFORMATION lpProcessInformation
    );
그 아래가 바로 멘토님의 코드에서 CreateProcessA의 인자 관련 코드입니다.
    mov DWORD PTR [ebp-0x48], 0x44 //StartupInfo cb
    lea eax, [ebp-0x48]  //eax: StartupInfo addr
    push eax //pProcessInfo
    push eax //pStartupInfo
    push 0x0 //starting directory
    push 0x0 //Environment block
    push 0x0 //Create flags
    push 0x0 //bInherit Handle
    push 0x0 //lpThreadAttribute
    push 0x0 //lpProcessAttribute
    push DWORD PTR [ebp-0x4C] //lpCommandLine
    push 0x0 //lpApplicationName


번외로 저는 기존의 공간 할당한 크기값을 좀더 줄여서 진행하여 0x40을 마지막에 다시 add해주었습니다.
    add esp, 0x40
