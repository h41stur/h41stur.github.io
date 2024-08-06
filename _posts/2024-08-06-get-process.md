---
title: "Enumerando Processos pelo Nome"
author: H41stur
date: 2024-08-06 01:00:00 -0300
categories: [Estudos, WinAPI]
tags: [Shellcoding, Shellcode, Windows API, Kernel, WinAPI]
image: "/img/posts/WinAPI.png"
alt: "Enumerando Processos pelo Nome"
---

![Enumerando Processos pelo Nome](/img/posts/WinAPI.png)


# TL;DR

Neste artigo, exploramos como utilizar as APIs do Windows para enumerar processos em execução, focando nas funções `CreateToolhelp32Snapshot`, `Process32First` e `Process32Next`. Discutimos a importância de entender como essas APIs funcionam por baixo dos panos, especialmente para desenvolvedores e profissionais de segurança cibernética. Mostramos um exemplo prático de código para obter o PID de processos pelo nome, comparando os resultados com ferramentas nativas do Windows como PowerShell e CMD.

Esta compreensão profunda das APIs permite desenvolver soluções personalizadas e fortalecer as habilidades técnicas, abrindo caminho para futuras explorações criativas e seguras no campo da segurança cibernética.

# Introdução

Seguindo com os estudos sobre como explorar as chamadas de WinAPI, achei interessante escrever sobre alguns pontos específicos que fazem muito sentido para o **meu** processo de estudos, sendo basicamente mimetizar ações que possam parecer triviais, porém com uso das APIs.

Acredito que este tipo de entendimento é essencial no contexto de **verdadeiro** entendimento de todo o processo. Uma vez que, mesmo podendo realizar ações simples utilizando ferramentas prontas e até mesmo intrínsecas do próprio Windows, podemos "abrir o capô" do SO e ver o motor funcionando. Saber como as aplicações realizam estas ações simples, reproduzir estas ações de forma primitiva e até mesmo encontrar maneiras diferentes de fazê-las.

No caso deste artigo, me foquei em estudar e utilizar as APIs do Windows para encontrar informações de processos em execução através de uma pesquisa por seu nome. Algo tão simples quanto utilizar o comando `Get-Process` no `PowerShell` ou o `tasklist` no `CMD`. Mas, e se eu não quiser utilizar `Powershell` ou `CMD`? E se eu estiver codando um *malware* que faz uso exclusivo de WinAPI? E se eu quiser criar minhas próprias chamadas customizadas? E se eu quiser entender como tudo funciona por baixo?

Tentarei esmiuçar o processo que usei para estudar e os achados deste processo que podem ser úteis não só para esta ação utilizada no artigo, mas para tantas outras.

Boa leitura e boa sorte!

# *God save the debuggers!*

As APIs do Windows compreendem uma infinidade de funcionalidades e possibilidades dentro do SO, uma simples olhada na documentação da [Win32 API](https://learn.microsoft.com/en-us/windows/win32/api/) já revela sua magnitude. Isso facilita muito o processo para os desenvolvedores do próprio Windows e de desenvolvedores de *softwares* em geral.

Entre as diversas bibliotecas possíveis, existem algumas em específico que tem potencial para qualquer uso, inclusive usos maliciosos (eu diria que qualquer biblioteca poderia ter uso malicioso). Porém, a depender da criatividade, algumas delas podem ser uma pérola. É o caso da [*Tool Help Library*](https://learn.microsoft.com/en-us/windows/win32/toolhelp/tool-help-library).

Conforme a própria documentação, esta biblioteca fornece funções "que foram projetadas para simplificar a criação de ferramentas, especificamente *debuggers*."

Uma vez que *debuggers* precisam de informações completas sobre processos, *threads*, módulos, memória, entre outras informações, temos a nossa disposição uma biblioteca que nos entrega muita coisa que pode ser utilizada para enumeração e possível exploração.

## *Tool Help Library*

Quando utilizamos a *Tool Help Library* para enumerar os processos em memória, esta biblioteca faz um *snapshot* somente leitura da memória que pode conter os processos, *threads*, módulos ou a *heap*, ou todos eles.

O fato de usar um *snapshot* e não consultar a memória em tempo real, se dá pelo fato de que cada processo iniciado ou encerrado, cada *thread* iniciada ou encerrada, cada módulo executado ou encerrado, cada *heap* criada ou destruída, altera a estrutura da memória em tempo real, e um processo que fizesse consultas nesta estrutura, poderia corrompê-la facilmente.

Portanto, o "*core*" da *Tool Help Library*, é a função [*CreateToolhelp32Snapshot*](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot) que cria este *snapshot* e o retorna em um *handle* que nada mais é que uma ou várias listas contendo as informações solicitadas.

```cpp
HANDLE CreateToolhelp32Snapshot(
  [in] DWORD dwFlags,
  [in] DWORD th32ProcessID
);
```

Onde:

- `[in] dwFlags`: Indica as partes do sistema a serem incluídas no *snapshot*. Podendo ser um ou mais valores.

![](/img/posts/Pasted%20image%2020240805220814.png)

- `[in] th32ProcessID`: O identificador de processo do processo a ser incluído no *snapshot*. Este parâmetro pode ser zero para indicar o processo atual.

No contexto deste artigo, o que nos interessa é enumerar os processos, portanto, utilizaremos o argumento `TH32CS_SNAPPROCESS`.

Uma vez que temos o *handle* que contém a lista de todos os processos em execução com suas respectivas particularidades, podemos navegar por esta lista utilizando outras funções.

A primeira delas é a função [Process32First](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32first).

```cpp
BOOL Process32First(
  [in]      HANDLE           hSnapshot,
  [in, out] LPPROCESSENTRY32 lppe
);
```

Esta função exige como argumento o *handle* extraído com a lista dos processos, e um ponteiro para uma estrutura do tipo [PROCESSENTRY32](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-processentry32) onde ela descarregará as informações sobre o primeiro processo encontrado na lista.

A estrutura `PROCESSENTRY32` tem o seguinte formato:

```cpp
typedef struct tagPROCESSENTRY32 {
  DWORD     dwSize;
  DWORD     cntUsage;
  DWORD     th32ProcessID;
  ULONG_PTR th32DefaultHeapID;
  DWORD     th32ModuleID;
  DWORD     cntThreads;
  DWORD     th32ParentProcessID;
  LONG      pcPriClassBase;
  DWORD     dwFlags;
  CHAR      szExeFile[MAX_PATH];
} PROCESSENTRY32;
```

Uma informação importante vinda da própria documentação, é que antes de invocarmos a função `Process32First()` temos que inicializar o parâmetro `dwSize` da estrutura PROCESSENTRY32 com o valor `sizeof(PROCESSENTRY32)`, caso contrário o processo falhará.

![](/img/posts/Pasted%20image%2020240805222649.png)

Uma vez que a função `Process32First()` foi executada com sucesso, temos todas as informações sobre o primeiro processo do *snapshot* contidos na *struct* PROCESSENTRY32 e podemos consultá-la e obtermos as informações que precisamos.

Para continuar a navegação no *snapshot* a partir deste ponto, utilizamos a função [Process32Next](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32next) que tem basicamente os mesmos argumentos da anterior.

```cpp
BOOL Process32Next(
  [in]  HANDLE           hSnapshot,
  [out] LPPROCESSENTRY32 lppe
);
```

Após sua execução, a estrutura PROCESSENTRY32 passa a conter os dados do próximo processo do *snapshot*. A partir deste ponto, podemos percorrer toda a lista de processos contidos no *snapshot* obtendo as informações necessárias para nossos objetivos.

Este processo, em nosso contexto, será utilizado com um *snapshot* focado nos processos em execução, porém, para cada outro tipo de informação possível de se extrair, temos funções equivalentes, como estrutura `THREADENTRY32` e funções `Thread32Frist` e `Thread32Next`, estrutura `MODULEENTRY32` e funções `Module32First` e `Module32Next` e assim por diante.

Na própria documentação, temos um [script de exemplo ](https://learn.microsoft.com/en-us/windows/win32/toolhelp/taking-a-snapshot-and-viewing-processes) que dá uma série de detalhes sobre processos, *threads* e módulos em execução.

```cpp
#include <windows.h>
#include <tlhelp32.h>
#include <tchar.h>
#include <stdio.h>

//  Forward declarations:
BOOL GetProcessList( );
BOOL ListProcessModules( DWORD dwPID );
BOOL ListProcessThreads( DWORD dwOwnerPID );
void printError( TCHAR const* msg );

int main( void )
{
  GetProcessList( );
  return 0;
}

BOOL GetProcessList( )
{
  HANDLE hProcessSnap;
  HANDLE hProcess;
  PROCESSENTRY32 pe32;
  DWORD dwPriorityClass;

  // Take a snapshot of all processes in the system.
  hProcessSnap = CreateToolhelp32Snapshot( TH32CS_SNAPPROCESS, 0 );
  if( hProcessSnap == INVALID_HANDLE_VALUE )
  {
    printError( TEXT("CreateToolhelp32Snapshot (of processes)") );
    return( FALSE );
  }

  // Set the size of the structure before using it.
  pe32.dwSize = sizeof( PROCESSENTRY32 );

  // Retrieve information about the first process,
  // and exit if unsuccessful
  if( !Process32First( hProcessSnap, &pe32 ) )
  {
    printError( TEXT("Process32First") ); // show cause of failure
    CloseHandle( hProcessSnap );          // clean the snapshot object
    return( FALSE );
  }

  // Now walk the snapshot of processes, and
  // display information about each process in turn
  do
  {
    _tprintf( TEXT("\n\n=====================================================" ));
    _tprintf( TEXT("\nPROCESS NAME:  %s"), pe32.szExeFile );
    _tprintf( TEXT("\n-------------------------------------------------------" ));

    // Retrieve the priority class.
    dwPriorityClass = 0;
    hProcess = OpenProcess( PROCESS_ALL_ACCESS, FALSE, pe32.th32ProcessID );
    if( hProcess == NULL )
      printError( TEXT("OpenProcess") );
    else
    {
      dwPriorityClass = GetPriorityClass( hProcess );
      if( !dwPriorityClass )
        printError( TEXT("GetPriorityClass") );
      CloseHandle( hProcess );
    }

    _tprintf( TEXT("\n  Process ID        = 0x%08X"), pe32.th32ProcessID );
    _tprintf( TEXT("\n  Thread count      = %d"),   pe32.cntThreads );
    _tprintf( TEXT("\n  Parent process ID = 0x%08X"), pe32.th32ParentProcessID );
    _tprintf( TEXT("\n  Priority base     = %d"), pe32.pcPriClassBase );
    if( dwPriorityClass )
      _tprintf( TEXT("\n  Priority class    = %d"), dwPriorityClass );

    // List the modules and threads associated with this process
    ListProcessModules( pe32.th32ProcessID );
    ListProcessThreads( pe32.th32ProcessID );

  } while( Process32Next( hProcessSnap, &pe32 ) );

  CloseHandle( hProcessSnap );
  return( TRUE );
}


BOOL ListProcessModules( DWORD dwPID )
{
  HANDLE hModuleSnap = INVALID_HANDLE_VALUE;
  MODULEENTRY32 me32;

  // Take a snapshot of all modules in the specified process.
  hModuleSnap = CreateToolhelp32Snapshot( TH32CS_SNAPMODULE, dwPID );
  if( hModuleSnap == INVALID_HANDLE_VALUE )
  {
    printError( TEXT("CreateToolhelp32Snapshot (of modules)") );
    return( FALSE );
  }

  // Set the size of the structure before using it.
  me32.dwSize = sizeof( MODULEENTRY32 );

  // Retrieve information about the first module,
  // and exit if unsuccessful
  if( !Module32First( hModuleSnap, &me32 ) )
  {
    printError( TEXT("Module32First") );  // show cause of failure
    CloseHandle( hModuleSnap );           // clean the snapshot object
    return( FALSE );
  }

  // Now walk the module list of the process,
  // and display information about each module
  do
  {
    _tprintf( TEXT("\n\n     MODULE NAME:     %s"),   me32.szModule );
    _tprintf( TEXT("\n     Executable     = %s"),     me32.szExePath );
    _tprintf( TEXT("\n     Process ID     = 0x%08X"),         me32.th32ProcessID );
    _tprintf( TEXT("\n     Ref count (g)  = 0x%04X"),     me32.GlblcntUsage );
    _tprintf( TEXT("\n     Ref count (p)  = 0x%04X"),     me32.ProccntUsage );
    _tprintf( TEXT("\n     Base address   = 0x%08X"), (DWORD) me32.modBaseAddr );
    _tprintf( TEXT("\n     Base size      = %d"),             me32.modBaseSize );

  } while( Module32Next( hModuleSnap, &me32 ) );

  CloseHandle( hModuleSnap );
  return( TRUE );
}

BOOL ListProcessThreads( DWORD dwOwnerPID ) 
{ 
  HANDLE hThreadSnap = INVALID_HANDLE_VALUE; 
  THREADENTRY32 te32; 
 
  // Take a snapshot of all running threads  
  hThreadSnap = CreateToolhelp32Snapshot( TH32CS_SNAPTHREAD, 0 ); 
  if( hThreadSnap == INVALID_HANDLE_VALUE ) 
    return( FALSE ); 
 
  // Fill in the size of the structure before using it. 
  te32.dwSize = sizeof(THREADENTRY32); 
 
  // Retrieve information about the first thread,
  // and exit if unsuccessful
  if( !Thread32First( hThreadSnap, &te32 ) ) 
  {
    printError( TEXT("Thread32First") ); // show cause of failure
    CloseHandle( hThreadSnap );          // clean the snapshot object
    return( FALSE );
  }

  // Now walk the thread list of the system,
  // and display information about each thread
  // associated with the specified process
  do 
  { 
    if( te32.th32OwnerProcessID == dwOwnerPID )
    {
      _tprintf( TEXT("\n\n     THREAD ID      = 0x%08X"), te32.th32ThreadID ); 
      _tprintf( TEXT("\n     Base priority  = %d"), te32.tpBasePri ); 
      _tprintf( TEXT("\n     Delta priority = %d"), te32.tpDeltaPri ); 
      _tprintf( TEXT("\n"));
    }
  } while( Thread32Next(hThreadSnap, &te32 ) ); 

  CloseHandle( hThreadSnap );
  return( TRUE );
}

void printError( TCHAR const* msg )
{
  DWORD eNum;
  TCHAR sysMsg[256];
  TCHAR* p;

  eNum = GetLastError( );
  FormatMessage( FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_IGNORE_INSERTS,
         NULL, eNum,
         MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT), // Default language
         sysMsg, 256, NULL );

  // Trim the end of the line and terminate it with a null
  p = sysMsg;
  while( ( *p > 31 ) || ( *p == 9 ) )
    ++p;
  do { *p-- = 0; } while( ( p >= sysMsg ) &&
                          ( ( *p == '.' ) || ( *p < 33 ) ) );

  // Display the message
  _tprintf( TEXT("\n  WARNING: %s failed with error %d (%s)"), msg, eNum, sysMsg );
}
```

Podemos compilar este programa:

```bash
i686-w64-mingw32-gcc snapProcess.cpp -o snapProcess.exe
```

Ao executarmos no Windows, ele começará a enumerar todas as informações a partir de um *snapshot*.

![](/img/posts/Pasted%20image%2020240806072245.png)

Obviamente este programa de exemplo trás muitas informações em uma resposta bastante completa, como o objeto deste estudo é somente enumerarmos os processos, podemos utilizá-lo como base para criarmos um programa mais simples, como um que nos retorne o **PID** de um processo, pesquisando pelo nome.

Uma vez que o PID é a principal forma de identificar um processo no ponto de vista do SO, e do kernel, obter esta informação é de extrema importância em um contexto <i><span style="color:red;">red team</span></i>.

Aproveitando a lógica do programa de exemplo, podemos criar a estrutura `PROCESSENTRY32` que receberá as informações do processo durante a iteração com o *snapshot*. Um dos parâmetros da estrutura é o `szExeFile` que contém o nome do executável. Durante o *loop* pelos processos, podemos comparar este parâmetro com o que procuramos, caso o encontre, o *loop* pára e nos retorna o parâmetro `th32ProcessID` que contém o PID do processo.

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

Podemos compilar o programa:

```bash
i686-w64-mingw32-gcc getPID.cpp -o getPID.exe
```

Para fins de teste, podemos iniciar um executável qualquer, como o `Notepad.exe` e enumerar seu PID tanto com o nosso programa quanto com o `PowerShell` para comparação.

![](/img/posts/Pasted%20image%2020240806085211.png)

Como podemos ver, ambos trouxeram o mesmo resultado, mostrando que conseguimos enumerar os processos utilizando as APIS do Windows sem grandes dificuldades. Este exercício nos proporciona uma compreensão mais profunda do funcionamento interno do sistema operacional.

Conforme dito no início deste artigo, esta é uma ação trivial que pode ser feita com qualquer outra ferramenta, porém nos da entendimento sobre como o processo funciona de forma mais primitiva dentro do SO.

Sabendo do potencial de informações possíveis utilizando este método, podemos, em artigos futuros, nos aproveitar da visão <i><span style="color:red;">red teamer</span></i> para utilizar este conhecimento de forma mais criativa.

# Conclusão

Neste artigo, exploramos detalhadamente como as APIs do Windows podem ser utilizadas para enumerar processos em execução, focando no uso de chamadas específicas como `CreateToolhelp32Snapshot`, `Process32First` e `Process32Next`. Compreendemos que, embora existam ferramentas prontas no próprio sistema operacional, a implementação dessas funcionalidades diretamente através da WinAPI nos proporciona uma visão mais profunda e um controle maior sobre o funcionamento interno do Windows.

Através do exemplo prático e das explicações fornecidas, vimos como é possível replicar essas funcionalidades de forma primitiva, demonstrando que, com o conhecimento adequado, podemos desenvolver soluções personalizadas e entender melhor o comportamento do sistema operacional. Este exercício é especialmente útil para desenvolvedores, profissionais de segurança e entusiastas que desejam fortalecer suas habilidades e conhecimentos técnicos.

Ao aprofundarmos nosso entendimento sobre como os processos funcionam e como podem ser manipulados, abrimos portas para inúmeras possibilidades, como desenvolvimento de novas ferramentas. Este artigo é apenas um ponto de partida, e esperamos explorar mais técnicas e abordagens nos próximos textos, sempre com a visão de red teamer, buscando formas criativas e inovadoras de aplicar esse conhecimento.

Agradeço por acompanhar este estudo e convido todos a continuar explorando e aprendendo sobre as infinitas possibilidades que as APIs do Windows oferecem. Boa sorte em seus estudos e desenvolvimento!

# Referências

- [https://learn.microsoft.com/en-us/windows/win32/api/](https://learn.microsoft.com/en-us/windows/win32/api/)
- [https://learn.microsoft.com/en-us/windows/win32/toolhelp/tool-help-library](https://learn.microsoft.com/en-us/windows/win32/toolhelp/tool-help-library)
- [https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot)
- [https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32first](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32first)
- [https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-processentry32](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-processentry32)
- [https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32next](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32next)

