---
title: "Process Listing e Token Dumping com WinAPI"
author: H41stur
date: 2024-08-15 01:00:00 -0300
categories: [Estudos, WinAPI]
tags: [Shellcoding, Shellcode, Windows API, Kernel, WinAPI, Process Listing]
image: "/img/posts/process-listing.png"
alt: "Process Listing e Token Dumping com WinAPI"
---

![Process Listing e Token Dumping com WinAPI](/img/posts/process-listing.png)

# TL;DR

Este artigo explora técnicas para listar processos e realizar _token dumping_ em sistemas Windows utilizando diversas APIs. Abordamos métodos além da Tool Help Library, como `WTSEnumerateProcessesEx`, destacando como essas abordagens podem ser aplicadas em cenários de exploração e reconhecimento de ambientes restritivos. A compreensão e o uso estratégico dessas APIs permitem uma coleta mais profunda de informações críticas sobre o ambiente, essencial para qualquer atividade de análise ou pentesting em sistemas Windows. O artigo fornece exemplos práticos e incentiva a combinação dessas técnicas para maximizar a eficácia das operações.

# Introdução

Em ambientes Windows, independentemente do objetivo de um artefato malicioso em execução, seja ele elevação de privilégios, persistência, movimentação lateral, evasão de defesas, entre outros, um dos primeiros passos críticos para o sucesso de qualquer operação é o reconhecimento completo de tudo que está em execução no sistema. Este reconhecimento vai além da simples descoberta dos nomes dos processos e serviços ativos; envolve a obtenção de informações detalhadas sobre o contexto de execução de cada processo, como seu PID, o usuário associado e o nível de privilégio em que opera. A capacidade de coletar o máximo de informações possíveis desde o início é fundamental para uma compreensão abrangente do ambiente, permitindo uma análise mais precisa e tomada de decisões informadas.

Embora listar processos seja uma tarefa trivial ao ter acesso direto a um terminal `PowerShell` ou `CMD`, o desafio se intensifica significativamente em cenários onde essas ferramentas não estão disponíveis ou onde o ambiente é restritivo. Nessas situações, o conhecimento das diversas APIs do Windows se torna inestimável. Em meus artigos anteriores, explorei técnicas como a [Tool Help Library](https://learn.microsoft.com/en-us/windows/win32/toolhelp/tool-help-library) para a enumeração de processos, conforme discutido no [Enumerando Processos pelo Nome](https://h41stur.com/posts/get-process/). No entanto, a *Tool Help Library* representa apenas uma das muitas abordagens possíveis ao utilizar as APIs do Windows.

Neste artigo, exploraremos alternativas adicionais para a listagem de processos, utilizando técnicas variadas que podem ser particularmente úteis em contextos de exploração avançada. Ao entender e dominar essas diferentes técnicas, não só ampliamos nosso arsenal de ferramentas, mas também aumentamos nossa capacidade de adaptar nossas estratégias às condições específicas de cada ambiente. A fusão dessas técnicas pode levar a resultados mais eficazes, proporcionando *insights* valiosos que seriam inacessíveis através de métodos convencionais.

Boa leitura e boa sorte!

# WTSEnumerateProcessesEx

Tendo em vista o objetivo deste artigo, a função [WTSEnumerateProcessesEx](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsenumerateprocessesexa) tem um nome bem sugestivo. Na verdade, é uma função criada justamente para listar processos, porém com uma característica muito interessante: **ela pode enumerar processos de sessões *desktop* remotas**.

No caso do nosso exemplo, usaremos para enumerar os processos locais, sua sintaxe é:

```cpp
BOOL WTSEnumerateProcessesExA(
  [in]      HANDLE hServer,
  [in, out] DWORD  *pLevel,
  [in]      DWORD  SessionId,
  [out]     LPSTR  *ppProcessInfo,
  [out]     DWORD  *pCount
);
```

Onde: 

- `[in] hServer` é um *handle* para um *Remote Desktop Session*. Este *handle* pode ser obtido com a função [WTSOpenServer](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsopenservera), porém, como vamos enumerar os processos locais, podemos preenchê-lo com `WTS_CURRENT_SERVER_HANDLE` indicando que o servidor é onde o programa está sendo executado;
- `[in, out] pLevel` é um ponteiro para uma variável do tipo `DWORD` que especifica o tipo de informação queremos enumerar. Quando setado para `0`, retorna uma estrutura do tipo [WTS_PROCESS_INFO](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/ns-wtsapi32-wts_process_infoa), quando setado para `1` retorna uma estrutura do tipo [WTS_PROCESS_INFO_EX](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/ns-wtsapi32-wts_process_info_exa), esta segunda, é muito mais completa e será a utilizada em nossa enumeração;
- `[in] SessionId` especifica a sessão da qual queremos enumerar. Para enumerar todas as sessões (que pode ser usado em nosso exemplo), podemos setá-la como `WTS_ANY_SESSION`;
- `[out] ppProcessInfo` um ponteiro para uma variável que recebe um ponteiro para a estrutura que receberá os dados (`WTS_PROCESS_INFO_EX`). Esta variável tem duas características importantes: quando não for mais usada, deve ser liberada com a função [WTSFreeMemoryEx](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsfreememoryexa), e, em seguida, esta variável deve ser setada como `NULL`;
- `[out] pCount` um ponteiro para uma variável que receberá o número de estruturas recebidas na enumeração.

Sobre a função `WTSFreeMemoryEx` utilizada para liberar a estrutura `WTS_PROCESS_INFO_EX` após seu uso, tem a seguinte sintaxe:

```cpp
BOOL WTSFreeMemoryExA(
  [in] WTS_TYPE_CLASS WTSTypeClass,
  [in] PVOID          pMemory,
  [in] ULONG          NumberOfEntries
);
```

Onde:

- `[in] WTSTypeClass` é um valor do tipo `WTS_TYPE_CLASS` que especifica o tipo de estrutura a ser liberada, seus valores são:

![](/img/posts/Pasted%20image%2020240813215002.png)

- `[in] pMemory` um ponteiro para o *buffer* que será liberado;
- `[in] NumberOfEntries` o número de estruturas que o *buffer* contém.

## Enumeração básica

Somente com estas duas funções é possível obter uma enumeração básica dos processos em execução em uma máquina. Basta invocar a função `WTSEnumerateProcessesEx` e imprimir as informações do objeto `PWTS_PROCESS_INFO_EX`.

```cpp
#include <Windows.h>
#include <tchar.h>
#include <WtsApi32.h>
#include <sddl.h>
#include <stdio.h>

#pragma comment(lib, "wtsapi32")
#pragma comment(lib, "Advapi32")

int main(void) {

	DWORD level = 1; // variável que indica o tipo da estrutura PWTS_PROCESS_INFO_EX
	PWTS_PROCESS_INFO_EX processes = NULL; // variável que contém uma estrutura PWTS_PROCESS_INFO_EX vazia
	DWORD processCounter = 0; // variável que conterá a quantidade de estruturas enumeradas

	// enumerando os processos
	if (!WTSEnumerateProcessesEx(
		WTS_CURRENT_SERVER_HANDLE,
		&level,
		WTS_ANY_SESSION,
		(LPTSTR*)&processes,
		&processCounter)) {
		return 0;
	}

	// imprimindo dados das estruturas
	_tprintf(_T("Processos encontrados: %d\n\n"), processCounter);
	PWTS_PROCESS_INFO_EX processesPtr = processes;

	_tprintf(_T("#\tPID\tHandles\tThreads\tProcess Name\n\n"));

	// iterando sobre as estruturas
	for (DWORD counter = 1; counter <= processCounter; counter++) {

		_tprintf(_T("%d\t"), counter);
		_tprintf(_T("%d\t"), processes->ProcessId);
		_tprintf(_T("%d\t"), processes->HandleCount);
		_tprintf(_T("%d\t"), processes->NumberOfThreads);
		_tprintf(_T("%s\n"), processes->pProcessName);

		processes++;
	}

	// liberando a memória da estrutura após o uso
	if (!WTSFreeMemoryEx(WTSTypeProcessInfoLevel1, processesPtr, processCounter)) {
		return 0;
	}
	processes = NULL;

	_tprintf(_T("\n\nPressione qualquer tecla para finalizar()\n"));

	getchar();
	return 0;
}
```

Ao executarmos o programa, temos todos os processos enumerados em sua saída:

![](/img/posts/Pasted%20image%2020240813220357.png)

Se compararmos a saída do programa com o ***Process Explorer*** do [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/), veremos algo basicamente igual.

![](/img/posts/Pasted%20image%2020240813221338.png)

## Implementando usuário e domínio

É possível deixar esta listagem de processos ainda mais completa, trazendo o domínio e o usuário que o executa. Estas informações são importantes, pois dizem sobre o contexto no qual um processo está em execução.

As funções utilizadas até o momento não trazem estas informações, mas tem a base necessária para buscá-las com outras funções. A estrutura `WTS_PROCESS_INFO_EX` tem o seguinte formato:

```cpp
typedef struct _WTS_PROCESS_INFO_EXA {
  DWORD         SessionId;
  DWORD         ProcessId;
  LPSTR         pProcessName;
  PSID          pUserSid;
  DWORD         NumberOfThreads;
  DWORD         HandleCount;
  DWORD         PagefileUsage;
  DWORD         PeakPagefileUsage;
  DWORD         WorkingSetSize;
  DWORD         PeakWorkingSetSize;
  LARGE_INTEGER UserTime;
  LARGE_INTEGER KernelTime;
} WTS_PROCESS_INFO_EXA, *PWTS_PROCESS_INFO_EXA;
```

Entre os membros da estrutura, temos o `pUserSid` que armazena o `SID` do usuário executando o processo.

### SID

O SID (*Security Identifier*) é um identificador exclusivo que o Windows usa para identificar objetos de segurança, como contas de usuário, grupos, contas de computadores, e até mesmo entidades como sessões e serviços. O SID é uma peça fundamental da infraestrutura de segurança do Windows, permitindo ao sistema operacional realizar controle de acesso e gerenciar permissões de maneira eficiente.

#### Estrutura do SID

Um SID é uma estrutura binária composta por várias partes que, juntas, garantem a unicidade e a hierarquia do identificador. A estrutura geral de um SID é a seguinte:

- ***Revision***: Um *byte* que indica a versão do SID. Atualmente, o valor é 1.
- ***SubAuthority Count***: Um *byte* que especifica o número de subautoridades (*sub authorities*) presentes no SID.
- ***Identifier Authority***: Um valor de 48 bits que identifica a autoridade emissora do SID. Ele define o escopo no qual o SID é único.
- ***SubAuthorities***: Uma série de valores de 32 bits que representam identificadores únicos adicionais, que tornam o SID único dentro do domínio da autoridade.

#### Formato do SID (Texto)

Quando representado em formato de texto, um SID é exibido de forma legível da seguinte maneira:

```
S-1-5-21-3623811015-3361044348-30300820-1013
```

Esse exemplo pode ser quebrado da seguinte maneira:

- **S**: Indica que a sequência representa um SID.
- **1**: Indica a versão do SID (no caso, versão 1).
- **5**: Representa a autoridade emissora. No caso, "5" refere-se à autoridade de segurança NT (*Security Authority*).
- **21**: Indica que o SID foi emitido por um controlador de domínio ou por uma autoridade emissora local.
- **3623811015-3361044348-30300820**: Esses números são as sub autoridades, e indicam geralmente o domínio ou a máquina onde a conta foi criada.
- **1013**: O identificador relativo (RID - *Relative Identifier*), que especifica o objeto de segurança específico (como um usuário ou grupo) no domínio ou máquina.

#### Tipos Comuns de SIDs

Alguns SIDs são bem conhecidos porque são usados para identificar grupos e usuários padrão do Windows. Aqui estão alguns exemplos:

- **S-1-5-18**: Representa a conta ***Local System***.
- **S-1-5-19**: Representa a conta ***Local Service***.
- **S-1-5-20**: Representa a conta ***Network Service***.
- **S-1-5-32-544**: Representa o grupo ***Administrators***.
- **S-1-5-32-545**: Representa o grupo ***Users***.
- **S-1-5-32-546**: Representa o grupo ***Guests***.

#### Função do SID no Controle de Acesso

No Windows, o SID é usado em diversas partes do sistema para controle de acesso. Algumas de suas funções mais importantes incluem:

1. **ACLs (*Access Control Lists*)**:
    
    - **DACL (*Discretionary Access Control List*)**: A lista que especifica quais SIDs têm permissões de acesso a um objeto, como um arquivo ou pasta. Cada entrada na DACL, chamada de ACE (*Access Control Entry*), contém um SID e as permissões associadas a esse SID.
    - **SACL (*System Access Control List*)**: Usada para auditoria, especificando quais SIDs devem ser auditados para acesso a um objeto.
2. **Sessões de Login**:
    
    - Quando um usuário faz login, o Windows usa o SID associado à conta de usuário para identificar o usuário em todo o sistema. Esse SID é associado a um *token* de segurança que contém todos os SIDs relevantes (como os grupos dos quais o usuário faz parte) e é usado para verificar permissões sempre que o usuário tenta acessar recursos.
3. **Identificação de Contas e Grupos**:
    
    - Cada conta de usuário ou grupo no Windows tem um SID único. Isso significa que, mesmo se o nome de uma conta de usuário for alterado, o SID permanece o mesmo, garantindo que o sistema possa rastrear consistentemente as permissões e associações de conta.
4. **Migração de Contas**:
    
    - Quando contas de usuário são migradas entre domínios ou sistemas, os SIDs antigos são preservados usando um conceito chamado ***SID History***. Isso garante que o usuário ainda tenha acesso aos recursos que ele podia acessar anteriormente, mesmo que o SID do novo domínio seja diferente.


Uma vez com o `SID` do usuário, podemos tomar algumas ações, primeiramente podemos usar a função [ConvertSidToStringSid](https://learn.microsoft.com/en-us/windows/win32/api/sddl/nf-sddl-convertsidtostringsida) para convertê-lo em *string*, assim podendo imprimir seu valor em tela. Esta função tem somente 2 parâmetros:

```cpp
BOOL ConvertSidToStringSidA(
  [in]  PSID  Sid,
  [out] LPSTR *StringSid
);
```

- `[in] Sid` o próprio SID;
- `[out] StringSid` o ponteiro para uma variável que recebe um ponteiro para uma *string* terminada em `NULL`. Este *buffer* deve ser liberado após seu uso com a função [LocalFree](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-localfree).

Além do `SID` em *string* para impressão, podemos usar a função [LookupAccountSid](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-lookupaccountsida) para obtermos o nome associado a conta que está executando o processo e o primeiro domínio associado ao SID. Esta função tem a seguinte sintaxe:

```cpp
BOOL LookupAccountSidA(
  [in, optional]  LPCSTR        lpSystemName,
  [in]            PSID          Sid,
  [out, optional] LPSTR         Name,
  [in, out]       LPDWORD       cchName,
  [out, optional] LPSTR         ReferencedDomainName,
  [in, out]       LPDWORD       cchReferencedDomainName,
  [out]           PSID_NAME_USE peUse
);
```

Onde:

- `[in, optional] lpSystemName` é um ponteiro para uma *string* que contém o computador alvo. Se for setado como `NULL` tem um comportamento interessante: primeiro tentará traduzir o nome na máquina local, caso não consiga, tentará resolver nos *domain controllers* de confiança do sistema local;
- `[in] Sid` um ponteiro para o SID a ser pesquisado;
- `[out, optional] Name` um ponteiro para um *buffer* que receberá o *account name*;
- `[in, out] cchName` especifica o tamanho em `TCHARs` do *buffer* que receberá o *account name*;
- `[out, optional] ReferencedDomainName` um ponteiro para um *buffer* que receberá o *domain name*;
- `[in, out] cchReferencedDomainName` especifica o tamanho em `TCHARs` do *buffer* que receberá o *domain name*;
- `[out] peUse` um ponteiro para uma estrutura [SID_NAME_USE](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ne-winnt-sid_name_use) que receberá todos os SIDs vinculados ao SID pesquisado.

Estas duas novas funções podem ser implementadas no *loop for* que itera sobre a lista `PWTS_PROCESS_INFO_EX` que geramos.

A implementação final do programa fica:

```cpp
#include <Windows.h>
#include <tchar.h>
#include <WtsApi32.h>
#include <sddl.h>
#include <stdio.h>

#pragma comment(lib, "wtsapi32")
#pragma comment(lib, "Advapi32")

#define MAX_ACCOUNTNAME_LEN 1024
#define MAX_DOMAINNAME_LEN 1024

int main(void) {

	DWORD level = 1; // variável que indica o tipo da estrutura PWTS_PROCESS_INFO_EX
	PWTS_PROCESS_INFO_EX processes = NULL; // variável que contém uma estrutura PWTS_PROCESS_INFO_EX vazia
	DWORD processCounter = 0; // variável que conterá a quantidade de estruturas enumeradas

	// enumerando os processos
	if (!WTSEnumerateProcessesEx(
		WTS_CURRENT_SERVER_HANDLE,
		&level,
		WTS_ANY_SESSION,
		(LPTSTR*)&processes,
		&processCounter)) {
		return 0;
	}

	// imprimindo dados das estruturas
	_tprintf(_T("Processos encontrados: %d\n\n"), processCounter);
	LPTSTR stringSID = NULL; // ponteiro do tipo string que receberá o SID do usuário que executa o processo
	PWTS_PROCESS_INFO_EX processesPtr = processes;

	_tprintf(_T("#\tPID\tHandles\tThreads\tProcess Name\tSID\tAccount\n\n"));

	// iterando sobre as estruturas
	for (DWORD counter = 1; counter <= processCounter; counter++) {

		_tprintf(_T("%d\t"), counter);
		_tprintf(_T("%d\t"), processes->ProcessId);
		_tprintf(_T("%d\t"), processes->HandleCount);
		_tprintf(_T("%d\t"), processes->NumberOfThreads);
		_tprintf(_T("%s\t"), processes->pProcessName);

		// convertendo o SID para string
		if (!ConvertSidToStringSid(processes->pUserSid, &stringSID)) {
			_tprintf(_T("-\t")); // imprime um traço, caso não consiga converter ou obter o SID
		}
		else {
			_tprintf(_T("%s\t"), stringSID);
			LocalFree((HLOCAL)stringSID); // libera a memória do ponteiro após seu uso
		}

		TCHAR accountName[MAX_ACCOUNTNAME_LEN];
		DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
		TCHAR domainName[MAX_DOMAINNAME_LEN];
		DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
		SID_NAME_USE peUse;

		// procura dados da conta através do SID
		if (!LookupAccountSid(
			NULL,
			processes->pUserSid,
			accountName,
			&accountNameBuffer,
			domainName,
			&domainNameBuffer,
			&peUse)) {
			_tprintf(_T("\n"));
		}
		else {
			_tprintf(_T("%s\\%s\n"), domainName, accountName);
		}

		processes++;
	}

	// liberando a memória da estrutura após o uso
	if (!WTSFreeMemoryEx(WTSTypeProcessInfoLevel1, processesPtr, processCounter)) {
		return 0;
	}
	processes = NULL;

	_tprintf(_T("\n\nPressione qualquer tecla para finalizar()\n"));

	getchar();
	return 0;
}
```

Ao executarmos o programa, temos impresso em tela todos os SIDs e usuários que foram possíveis de identificar.

![](/img/posts/Pasted%20image%2020240814072641.png)

Porém, temos uma situação: vários processos não tiveram seus usuários e domínio enumerados. Isso ocorre por uma situação de **extrema importância** em um cenário de exploração: **o programa lista os processos no contexto de quem os está executando**. Ou seja, se o usuário a executar o programa, não tem privilégios administrativos, logo, não terá permissão para idtentificar processos executados pelo `NT AUTHORITY`.

Podemos confirmar isso executando o `Process Explorer` do *Sysinternals* como usuário normal, veremos que **não temos esta permissão**.


![](/img/posts/Pasted%20image%2020240814073029.png)

Se executarmos nosso programa como **administrador** teremos uma visão mais ampla dos processos em execução:

![](/img/posts/Pasted%20image%2020240814073935.png)

Assim como ao executar o `Process Explorer` como administrador, também nos retornará algo similar:

![](/img/posts/Pasted%20image%2020240814074032.png)

Entretanto, ainda existe uma questão incompleta neste processo, se analisarmos o *output* do programa comparado ao *output* do `Process Explorer`, veremos que alguns processos ainda não tiveram seu contexto enumerado mesmo executando como administrador:

![](/img/posts/Pasted%20image%2020240814074234.png)

Isso ocorre pelo fado de que, mesmo executando nosso programa com privilégios de administrador, existem privilégios mais elevados que por padrão o Windows não implementa por motivos de segurança. Privilégios esses implementados no `Process Explorer`, porém não em nosso programa. 

O principal envolvido neste processo é o `SeDebugPrivilege`, se analisarmos nosso programa em execução com o `Process Explorer` e formos na aba `Security`, veremos que o `SeDebugPrivilege` está desabilitado mesmo o executando como **administrador**:

![](/img/posts/Pasted%20image%2020240814080259.png)

Isto pode ser implementado em nosso programa para termos uma visão **total** dos processos.

## Habilitando o SeDebugPrivilege

Habilitar o `SeDebugPrivilege` em um programa é uma tarefa relativamente simples, porém, para entendimento de como tudo funciona, é preciso montar um *background* sólido do processo.

### Tokens de acesso

Um **token de acesso** no Windows é uma estrutura de dados essencial para a segurança do sistema, representando a identidade de um processo ou *thread*. Ele contém informações sobre os direitos e privilégios de segurança de um usuário, grupo ou dispositivo que está executando o processo ou *thread*, sendo usado pelo sistema operacional para controlar o acesso a recursos do sistema, como arquivos, pastas, serviços, e outros objetos de segurança.

#### O Que é um Token de Acesso?

O token de acesso é fundamental para a segurança do Windows, funcionando como um "passe" que carrega todas as informações de segurança relevantes de um usuário ou processo. Quando um usuário faz login no Windows, o sistema autentica o usuário e gera um token de acesso. Esse token é associado a qualquer processo que o usuário inicie, como um aplicativo ou um serviço.

#### Conteúdo de um Token de Acesso

Um token de acesso contém várias informações cruciais para o controle de acesso:

1. **SID (*Security Identifier*)**:
    
    - Cada token contém o SID do usuário ou grupo que possui o processo. O SID identifica exclusivamente o usuário no sistema.
2. **SIDs de Grupos**:
    
    - O token também contém os SIDs de todos os grupos aos quais o usuário pertence. Isso inclui grupos de segurança como `Administradores`, `Usuários`, ou grupos específicos da organização.
3. **Privilégios**:
    
    - Os privilégios são permissões adicionais que um processo pode ter. Eles podem incluir direitos especiais como `SeDebugPrivilege`, que permite depurar qualquer processo no sistema.
4. **Lista de Controle de Acesso (ACLs)**:
    
    - Embora as ACLs sejam usadas para definir permissões em objetos, o token de acesso carrega as permissões que um processo tem com base nas ACLs aplicadas ao usuário.
5. **Tipo de Token**:
    
    - Os tokens podem ser de dois tipos principais: `Primary` e `Impersonation`. Tokens primários são usados para criar novos processos, enquanto tokens de impersonação são usados para representar o contexto de segurança de outro usuário, permitindo que um processo "faça-se passar" por outro usuário.
6. **Nível de Impersonação**:
    
    - Define o grau de acesso que um processo tem ao usar o token de outro usuário. Os níveis incluem `Anonymous`, `Identification`, `Impersonation`, e `Delegation`.
7. **Nível de Integridade**:
    
    - O Windows usa um sistema de níveis de integridade para proteger objetos sensíveis de processos menos confiáveis. O token de acesso contém um SID de nível de integridade que determina as permissões que o processo tem sobre objetos protegidos. Níveis comuns incluem `Low`, `Medium`, `High`, e `System`.
8. **Tempo de Criação e Expiração**:
    
    - Registra quando o token foi criado e, em alguns casos, quando ele expira.

#### Como um Token de Acesso é Criado e Usado?

1. **Autenticação do Usuário**:
    
    - Quando um usuário faz login no sistema, seja localmente ou através de uma rede, o Windows autentica o usuário usando credenciais como senha ou certificado. Após a autenticação bem-sucedida, o sistema gera um token de acesso primário.
2. **Atribuição do Token a Processos**:
    
    - Esse token de acesso é então associado a todos os processos que o usuário inicia. Por exemplo, quando o usuário abre um aplicativo, o Windows copia o token de acesso do usuário para o novo processo, garantindo que o processo herde todas as permissões e restrições do usuário.
3. **Verificação de Acesso**:
    
    - Quando um processo tenta acessar um recurso (como abrir um arquivo ou modificar uma chave de registro), o Windows verifica o token de acesso associado ao processo contra as ACLs do recurso. Se o SID do usuário ou de qualquer grupo ao qual ele pertence tiver permissão nas ACLs do recurso, o acesso é concedido; caso contrário, é negado.
4. **Impersonação**:
    
    - Em alguns casos, um processo pode querer agir em nome de outro usuário. Isso é feito usando tokens de impersonação. Por exemplo, um serviço do Windows pode usar um token de impersonação para executar ações em nome de um usuário que está interagindo com o serviço.

#### Manipulação de Tokens no Windows

No desenvolvimento e administração de sistema Windows, é comum precisar manipular tokens de acesso. Algumas funções da API do Windows que permitem a manipulação de tokens de acesso são:

- [OpenProcessToken()](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken):
    
    - Abre o token de acesso associado a um processo existente.
- [GetTokenInformation()](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation):
    
    - Obtém informações específicas de um token, como privilégios, SIDs ou o tipo de token.
- [SetTokenInformation()](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-settokeninformation):
    
    - Modifica informações em um token de acesso, como definir um nível de integridade.
- [AdjustTokenPrivileges()](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges):
    
    - Habilita ou desabilita privilégios no token de acesso.
- [DuplicateTokenEx()](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-duplicatetokenex):
    
    - Cria uma cópia de um token, permitindo a criação de tokens de impersonação ou novos tokens primários com modificações.
- [ImpersonateLoggedOnUser()](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-impersonateloggedonuser):
    
    - Permite que um processo use um token de um usuário logado para realizar operações em nome desse usuário.


Como os tokens de acesso são uma parte central da arquitetura de segurança do Windows. Se um atacante conseguir manipular ou criar tokens de acesso com permissões elevadas, ele pode efetuar um **Privilege Escalation**, ganhando controle sobre partes significativas ou até mesmo todo o sistema.

Em outras palavras: quem controla os tokens, controla o SO.

### LUID

Um **LUID (*Locally Unique Identifier*)**, é um identificador de 64 bits usado no sistema operacional para identificar de maneira única um objeto ou evento em um contexto local, como dentro de um computador ou uma sessão de sistema operacional. Isso significa que, enquanto o computador estiver em execução, qualquer LUID gerado por ele será único. Após um reinício, no entanto, o mesmo LUID pode ser gerado novamente, o que não é um problema já que ele é único somente na instância atual do sistema. O LUID é utilizado principalmente em questões relacionadas à segurança, especialmente no contexto de privilégios e tokens de acesso.

#### Estrutura

O LUID é um valor de 64 bits (`unsigned long long` ou `DWORDLONG`) e, por isso, pode representar uma vasta quantidade de identificadores únicos. No Windows, ele é representado pela estrutura `LUID`, que é composta por dois valores de 32 bits (`LowPart` e `HighPart`).

#### Onde o LUID é Usado?

Os LUIDs são usados em várias partes do sistema operacional Windows, mas são particularmente relevantes em questões de segurança e gestão de privilégios. Eles aparecem em contextos como:

1. **Privilégios de Segurança**:
    
    - No Windows, privilégios são operações ou capacidades especiais que podem ser concedidas a processos ou usuários. Cada privilégio tem um LUID associado a ele. Quando um processo ou *thread* precisa habilitar ou verificar um privilégio, o Windows usa o LUID para identificar o privilégio específico.
        
    - Por exemplo, o privilégio `SeDebugPrivilege` tem um LUID específico que pode ser consultado e usado em operações que manipulam privilégios, como `AdjustTokenPrivileges()`.
        
2. **Auditoria e Segurança**:
    
    - O LUID é usado em auditoria de segurança e eventos para garantir que eventos ou privilégios possam ser identificados de maneira única durante a execução do sistema.
3. **Tokens de Acesso**:
    
    - Dentro de tokens de acesso, que representam a identidade e privilégios de um processo, os LUIDs são usados para definir privilégios específicos que o token possui ou que podem ser ajustados.

#### Funcionamento e Geração do LUID

##### Geração do LUID

No Windows, LUIDs são gerados utilizando uma função do sistema chamada `AllocateLocallyUniqueId()`. Esta função garante que cada LUID gerado seja único no contexto do sistema operacional atual. A geração do LUID é eficiente, o que é importante para manter o desempenho do sistema, mesmo quando LUIDs são necessários com frequência.

##### Estrutura do LUID (em C++)

```cpp
typedef struct _LUID {
  DWORD LowPart;
  LONG  HighPart;
} LUID, *PLUID;
```

Onde:

- **`LowPart`**: Contém os 32 bits menos significativos do LUID;
- **`HighPart`**: Contém os 32 bits mais significativos do LUID.

### SeDebugPrivilege

O **SeDebugPrivilege** é um privilégio de segurança no Windows que concede a um usuário ou processo a capacidade de depurar e interagir com processos pertencentes a outros usuários, incluindo aqueles que pertencem ao sistema. Este privilégio é extremamente poderoso e, por isso, é geralmente restrito a contas administrativas ou a processos específicos que necessitam realizar tarefas de diagnóstico ou depuração ao nível de sistema.

Privilégios como este são particularmente críticos porque concedem capacidades que, se mal utilizadas, podem comprometer completamente a segurança e a integridade de um sistema. Devido à natureza sensível desse privilégio, ele é geralmente concedido apenas a contas administrativas e, em muitos casos, apenas é habilitado temporariamente e de maneira controlada.

#### Principais Funções do SeDebugPrivilege

1. ***Debugging* de Processos**:
    
    - O `SeDebugPrivilege` permite que um usuário ou processo anexe um *debugger* a qualquer processo em execução no sistema, independentemente do usuário proprietário do processo. Isso inclui processos executados com privilégios elevados, como serviços do sistema ou processos do núcleo do sistema operacional.
2. ***Bypass* de Controle de Acesso**:
    
    - Um dos aspectos mais poderosos desse privilégio é que ele permite ao titular ignorar as restrições de segurança que normalmente impedem um processo de acessar ou manipular processos de outros usuários. Por exemplo, um administrador com `SeDebugPrivilege` pode terminar, modificar ou injetar código em processos protegidos.
3. **Análise e Diagnóstico de Sistemas**:
    
    - Esse privilégio é frequentemente utilizado em ferramentas de diagnóstico e análise de segurança, como *debuggers* e analisadores de memória, que precisam acessar profundamente o sistema para monitorar ou manipular processos.
4. **Caminho de Exploração em Ataques**:
    
    - Devido ao poder que ele concede, o `SeDebugPrivilege` é frequentemente alvo de explorações. Um atacante que consegue obter `SeDebugPrivilege` em um sistema pode essencialmente ganhar controle total sobre o sistema, podendo terminar processos de segurança, injetar *malware* em processos confiáveis ou extrair informações sensíveis de processos em execução.

#### Funcionamento

Quando um processo é iniciado, ele recebe um token de acesso que especifica os privilégios que o processo possui. Privilégios como `SeDebugPrivilege` não estão habilitados por padrão, mesmo para processos que têm direito a eles. Isso é uma medida de segurança para minimizar o risco de abuso.

#### Habilitação de SeDebugPrivilege:

Um processo que precisa usar `SeDebugPrivilege` deve primeiro habilitá-lo explicitamente, o que pode ser feito por meio de chamadas à API do Windows, como `AdjustTokenPrivileges`. 

1. **Obtenção do Token do Processo**:
    
    - O processo chama `OpenProcessToken()` para obter o token de acesso associado ao seu próprio processo.
2. **Ajuste de Privilégios**:
    
    - Em seguida, o processo usa `LookupPrivilegeValue()` para obter o identificador do `SeDebugPrivilege` e `AdjustTokenPrivileges()` para habilitar esse privilégio no token.
3. **Uso do Privilégio**:
    
    - Após a habilitação, o processo pode usar APIs como `OpenProcess()`, `TerminateProcess()`, ou `WriteProcessMemory()` para interagir com processos que normalmente estariam fora do alcance devido a restrições de segurança.

#### Uso Legítimo vs. Abuso

Sendo um privilégio muito delicado, pode-se considerá-lo uma faca de dois gumes.

- **Uso Legítimo**:
    
    - Ferramentas de desenvolvimento e *debugging*, como o Visual Studio, frequentemente precisam habilitar `SeDebugPrivilege` para depurar processos do sistema ou serviços.
    - Em ambientes de TI, administradores podem habilitar esse privilégio para investigar problemas de desempenho, travamentos de processos, ou comportamento anômalo em sistemas.
- **Abuso e Exploração**:
    
    - **Escalação de Privilégios**: Se um atacante obtiver acesso a uma conta de usuário com baixos privilégios, mas conseguir ativar `SeDebugPrivilege`, ele pode escalar seus privilégios para acessar processos de sistema críticos.
    - **Injeção de Código**: Um invasor com `SeDebugPrivilege` pode injetar código malicioso em processos de alta integridade, como antivírus ou serviços de sistema, comprometendo essencialmente a segurança do sistema de forma invisível.
    - **Desativação de Segurança**: Ferramentas de segurança, como antivírus, executadas com privilégios elevados, podem ser terminadas ou manipuladas por um atacante com `SeDebugPrivilege`.


### Implementando o código

Uma vez que a base para a implementação está construída, podemos criar uma função que irá alterar a propriedade `SeDebugPrivilege` no token pertencente ao nosso programa.

O primeiro passo é usar a função [LookupPrivilegeValue](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-lookupprivilegevaluea) para obter o LUID do `SeDebugPrivilege`. Esta função tem a seguinte sintaxe:

```cpp
BOOL LookupPrivilegeValueA(
  [in, optional] LPCSTR lpSystemName,
  [in]           LPCSTR lpName,
  [out]          PLUID  lpLuid
);
```

Onde:

- `[in, optional] lpSystemName` é um ponteiro para uma *string* que contém o nome do sistema onde o privilégio será recuperado. Se for passado o valor `NULL`, a função tenta recuperar o privilégio no sistema local;
- `[in] lpName` um ponteiro para a *string* que contém qual privilégio queremos recuperar;
- `[out] lpLuid` o ponteiro para uma variável que receberá o LUID após a pesquisa.

No código, teremos algo como:

```cpp
LUID privilegeLuid;
if (!LookupPrivilegeValue(NULL, _T("SeDebugPrivilege"), &privilegeLuid)) {
	return 0;
}
```

Uma vez com o LUID do `SeDebugPrivilege`, podemos criar uma estrutura do tipo [TOKEN_PRIVILEGES](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-token_privileges) para manipular o token original do programa. Esta estrutura tem o seguinte formato:

```cpp
typedef struct _TOKEN_PRIVILEGES {
  DWORD               PrivilegeCount;
  LUID_AND_ATTRIBUTES Privileges[ANYSIZE_ARRAY];
} TOKEN_PRIVILEGES, *PTOKEN_PRIVILEGES;
```

Onde:

- `PrivilegeCount` especifica a quantidade de privilégios que a *array* da estrutura conterá, em nosso caso **1**;
- `Privileges[ANYSIZE_ARRAY]` especifica um *array* de estruturas [LUID_AND_ATTRIBUTES](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-luid_and_attributes), este *array* conterá o LUID do privilégio e o atributo que ele receberá, de acordo com estes [atributos](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-privilege_set), utilizaremos o `SE_PRIVILEGE_ENABLED`:

![](/img/posts/Pasted%20image%2020240814140324.png)

No código, teremos algo como:

```cpp
// criando a estrutura do token
TOKEN_PRIVILEGES tkPrivs;

tkPrivs.PrivilegeCount = 1; // quantidade de privilégios a serem configurados
tkPrivs.Privileges[0].Luid = privilegeLuid; // LUID do primeiro e único privilégio (SeDebugPrivilege)
tkPrivs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED; // atributo que o privilégio receberá
```

Com esta base configurada, temos que criar um *handle* para o token do processo atual, no caso nosso programa, com os direitos de alteração de privilégios. Faremos isso com a função [OpenProcessToken](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken).

Esta função exige um *handle* para o processo atual, uma máscara de acesso que informará os direitos de acesso que queremos nesse token e um ponteiro para um *handle* que receberá o token aberto. Os [direitos de acesso](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-rights-for-access-token-objects) seguem a seguinte tabela:

![](/img/posts/Pasted%20image%2020240814141148.png)

Logo, em nosso caso, utilizaremos `TOKEN_ADJUST_PRIVILEGES`, uma vez que queremos habilitar um privilégio.

No código, teremos algo como:

```cpp
HANDLE currentProcessHandle = GetCurrentProcess();
HANDLE processToken;

if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES, &processToken)) {
	return 0;
}
```

Por fim, utilizaremos a função [AdjustTokenPrivileges](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges) que serve para habilitar ou desabilitar privilégios em tokens. Esta função receberá o *handle* para o token que geramos anteriormente, e um ponteiro para a estrutura `TOKEN_PRIVILEGES` que montamos. No código teremos algo como:

```cpp
if (!AdjustTokenPrivileges(processToken, false, &tkPrivs, 0, NULL, NULL)) {
	return 0;
}
```

Todos estes passos podem ser enfileirados em uma função dentro do nosso código, que será invocada logo no início da `main`. O código completo fica:

```cpp
#include <Windows.h>
#include <tchar.h>
#include <WtsApi32.h>
#include <sddl.h>
#include <stdio.h>

#pragma comment(lib, "wtsapi32")
#pragma comment(lib, "Advapi32")

#define MAX_ACCOUNTNAME_LEN 1024
#define MAX_DOMAINNAME_LEN 1024

// função para habilitar SeDebugPrivilege
BOOL EnableSeDebug(void) {

	LUID privilegeLuid;
	if (!LookupPrivilegeValue(NULL, _T("SeDebugPrivilege"), &privilegeLuid)) {
		return 0;
	}

	// criando a estrutura do token
	TOKEN_PRIVILEGES tkPrivs;

	tkPrivs.PrivilegeCount = 1; // quantidade de privilégios a serem configurados
	tkPrivs.Privileges[0].Luid = privilegeLuid; // LUID do primeiro e único privilégio (SeDebugPrivilege)
	tkPrivs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED; // atributo que o privilégio receberá

	HANDLE currentProcessHandle = GetCurrentProcess();
	HANDLE processToken;

	if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES, &processToken)) {
		return 0;
	}

	if (!AdjustTokenPrivileges(processToken, false, &tkPrivs, 0, NULL, NULL)) {
		return 0;
	}

	return TRUE;
}

int main(void) {

	EnableSeDebug();

	DWORD level = 1; // variável que indica o tipo da estrutura PWTS_PROCESS_INFO_EX
	PWTS_PROCESS_INFO_EX processes = NULL; // variável que contém uma estrutura PWTS_PROCESS_INFO_EX vazia
	DWORD processCounter = 0; // variável que conterá a quantidade de estruturas enumeradas

	// enumerando os processos
	if (!WTSEnumerateProcessesEx(
		WTS_CURRENT_SERVER_HANDLE,
		&level,
		WTS_ANY_SESSION,
		(LPTSTR*)&processes,
		&processCounter)) {
		return 0;
	}

	// imprimindo dados das estruturas
	_tprintf(_T("Processos encontrados: %d\n\n"), processCounter);
	LPTSTR stringSID = NULL; // ponteiro do tipo string que receberá o SID do usuário que executa o processo
	PWTS_PROCESS_INFO_EX processesPtr = processes;

	_tprintf(_T("#\tPID\tHandles\tThreads\tProcess Name\tSID\tAccount\n\n"));

	// iterando sobre as estruturas
	for (DWORD counter = 1; counter <= processCounter; counter++) {

		_tprintf(_T("%d\t"), counter);
		_tprintf(_T("%d\t"), processes->ProcessId);
		_tprintf(_T("%d\t"), processes->HandleCount);
		_tprintf(_T("%d\t"), processes->NumberOfThreads);
		_tprintf(_T("%s\t"), processes->pProcessName);

		// convertendo o SID para string
		if (!ConvertSidToStringSid(processes->pUserSid, &stringSID)) {
			_tprintf(_T("-\t")); // imprime um traço, caso não consiga converter ou obter o SID
		}
		else {
			_tprintf(_T("%s\t"), stringSID);
			LocalFree((HLOCAL)stringSID); // libera a memória do ponteiro após seu uso
		}

		TCHAR accountName[MAX_ACCOUNTNAME_LEN];
		DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
		TCHAR domainName[MAX_DOMAINNAME_LEN];
		DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
		SID_NAME_USE peUse;

		// procura dados da conta através do SID
		if (!LookupAccountSid(
			NULL,
			processes->pUserSid,
			accountName,
			&accountNameBuffer,
			domainName,
			&domainNameBuffer,
			&peUse)) {
			_tprintf(_T("\n"));
		}
		else {
			_tprintf(_T("%s\\%s\n"), domainName, accountName);
		}

		processes++;
	}

	// liberando a memória da estrutura após o uso
	if (!WTSFreeMemoryEx(WTSTypeProcessInfoLevel1, processesPtr, processCounter)) {
		return 0;
	}
	processes = NULL;

	_tprintf(_T("\n\nPressione qualquer tecla para finalizar()\n"));

	getchar();
	return 0;
}
```

Agora, quando executamos o programa com privilégios de administrador, podemos ver no `Process Explorer` que o privilégio `SeDebugPrivilege` está habilitado, e os processos antes não enumerados agora trazem seu contexto.

![](/img/posts/Pasted%20image%2020240814142341.png)

## Melhorando o Código

O programa criado para listar os processos está funcionando normalmente, porém ele não está preparado para alguns cenários específicos. Uma vez que nem sempre o executaremos com privilégios administrativos, é preciso saber se é possível habilitar o `SeDebugPrivilege`.

Quando executamos o programa com privilégios de usuário, ele não retorna erro, mesmo quando invocamos a função que habilita os privilégios, o único comportamento é o de não trazer as informações que não tem acesso.

Isso corre, pois a função `AdjustTokenPrivileges` não adiciona novos privilégios a um token, e sim habilita ou desabilita um privilégio já existente. Se executarmos o programa com privilégios de usuário e checarmos com o `Process Explorer`, veremos que neste contexto o `SeDebugPrivilege` nem está presente.

![](/img/posts/Pasted%20image%2020240814163148.png)

Isso significa que a função não consegue habilitar ou desabilitar o privilégio, porém, ela não falha. Ao invés de falhar, a função simplesmente ignora os privilégios que não existem e continua sua execução.

Para termos um vislumbre do que está, acontecendo, podemos invocar a função [GetLastError](https://learn.microsoft.com/en-us/windows/win32/api/errhandlingapi/nf-errhandlingapi-getlasterror) para identificar o que está havendo. Existe um [código de exemplo](https://learn.microsoft.com/en-us/windows/win32/Debug/retrieving-the-last-error-code) fornecido pela própria Microsoft de como usar esta função. Abaixo o código:

```cpp
#include <windows.h>
#include <strsafe.h>

void ErrorExit(LPCTSTR lpszFunction) 
{ 
    // Retrieve the system error message for the last-error code

    LPVOID lpMsgBuf;
    LPVOID lpDisplayBuf;
    DWORD dw = GetLastError(); 

    FormatMessage(
        FORMAT_MESSAGE_ALLOCATE_BUFFER | 
        FORMAT_MESSAGE_FROM_SYSTEM |
        FORMAT_MESSAGE_IGNORE_INSERTS,
        NULL,
        dw,
        MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
        (LPTSTR) &lpMsgBuf,
        0, NULL );

    // Display the error message and exit the process

    lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT, 
        (lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR)); 
    StringCchPrintf((LPTSTR)lpDisplayBuf, 
        LocalSize(lpDisplayBuf) / sizeof(TCHAR),
        TEXT("%s failed with error %d: %s"), 
        lpszFunction, dw, lpMsgBuf); 
    MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK); 

    LocalFree(lpMsgBuf);
    LocalFree(lpDisplayBuf);
    ExitProcess(dw); 
}

void main()
{
    // Generate an error

    if(!GetProcessId(NULL))
        ErrorExit(TEXT("GetProcessId"));
}
```

Podemos fazer algumas alterações e implementá-lo em nosso código, substituiremos primeiramente o `MessaageBox` com o erro por um `_printf()` e colocaremos uma condição para encerrar o programa no qual escolheremos seu comportamento. A função fica da seguinte forma:

```cpp
void PrintError(LPCTSTR lpszFunction, bool exitProcess)
{
	// Retrieve the system error message for the last-error code

	LPVOID lpMsgBuf;
	LPVOID lpDisplayBuf;
	DWORD dw = GetLastError();

	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM |
		FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL,
		dw,
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf,
		0, NULL);

	// Display the error message and exit the process

	lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT,
		(lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR));
	StringCchPrintf((LPTSTR)lpDisplayBuf,
		LocalSize(lpDisplayBuf) / sizeof(TCHAR),
		TEXT("%s failed with error %d: %s"),
		lpszFunction, dw, lpMsgBuf);
	//MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK);
	_tprintf(TEXT("Error: %s\n"), (LPTSTR)lpDisplayBuf);

	LocalFree(lpMsgBuf);
	LocalFree(lpDisplayBuf);
	if (exitProcess) ExitProcess(dw);
}
```
Agora podemos invocá-la logo após chamarmos a função `EnableSeDebug()`.

```cpp
EnableSeDebug();
PrintError(_T("EnableSeDebug()"), true);
```

Quando executamos o programa, podemos ver o motivo da não enumeração dos processos.

![](/img/posts/Pasted%20image%2020240814164605.png)

Como podemos ver, os privilégios necessários não estão presentes no processo.

O problema disso, é que, muitas vezes perdemos falhas de *misconfiguration*, pois, como um processo herda os privilégios de quem o executa, existe a possibilidade de cairmos em um contexto onde nosso acesso tem os privilégios necessários, mesmo não sendo administrador.

Uma implementação que podemos fazer na função `EnableSeDebug`, é a checagem se o `SeDebugPrivilege` realmente existe no contexto de execução, antes de tentar habilitá-lo.

Para extrairmos esta informação de um token, usaremos a função [GetTokenInformation](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation) que tem a seguinte sintaxe:

```cpp
BOOL GetTokenInformation(
  [in]            HANDLE                  TokenHandle,
  [in]            TOKEN_INFORMATION_CLASS TokenInformationClass,
  [out, optional] LPVOID                  TokenInformation,
  [in]            DWORD                   TokenInformationLength,
  [out]           PDWORD                  ReturnLength
);
```

Onde:

- `[in] TokenHandle` é um *handle* de um token de acesso para onde as informações serão enviadas, este *nadle* é criado na função `OpenProcesssToken` e deve ter o tipo de acesso `TOKEN_QUERY`;
- `[in] TokenInformationClass` especifica qual valor queremos extrair da estrutura [TOKEN_INFORMATION_CLASS](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ne-winnt-token_information_class), no nosso caso é o `TokenPrivileges` que contém uma estrutura `TOKEN_PRIVILEGES` que já conhecemos.

![](/img/posts/Pasted%20image%2020240814170415.png)

- `[out, optional] TokenInformation` o ponteiro para um *buffer* que receberá as informações coletadas, pode ser `NULL`, nesse caso, nenhuma informação é retornada (isso é importante para os próximos passos);
- `[in] TokenInformationLength` o tamanho em *bytes* do *buffer* que receberá as informações, se o *buffer* for configurado como `NULL` este parâmetro precisa ser zero;
- `[out] ReturnLength` um ponteiro para uma variável que receberá o tamanho em *bytes* das informações coletadas;

O uso desta função tem uma particularidade, para que ela salve as informações em um *buffer*, é necessário especificar o tamanho em *bytes* desse *buffer*, porém, não sabemos o tamanho da resposta na primeira execução.

No entanto, como esta função também retorna o tamanho da resposta em *bytes*, a forma mais simples de resolver este problema, é invocá-la uma vez passando o *buffer* como `NULL` somente para armazenar o tamanho da resposta em uma variável. E, logo após, invocá-la novamente, passando esta variável como tamanho do *buffer* criado.

No código teremos:

```cpp
HANDLE currentProcessHandle = GetCurrentProcess();
HANDLE processToken;

if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &processToken)) {
	PrintError(_T("OpenProcessToken()"), true);
}

// coletando o tamanho da estrutura
DWORD firstStructSize;
GetTokenInformation(processToken, TokenPrivileges, NULL, 0, &firstStructSize);

// invocando GetTokenInformation() com todos os parâmetros
DWORD secondStructSize; // variável que receberá o tamanho da resposta na segunda execução
PTOKEN_PRIVILEGES processTokenPriv; // ponteiro para estrutura TOKEN_PRIVILEGES que receberá a resposta

processTokenPriv = (PTOKEN_PRIVILEGES)malloc(firstStructSize); // alocando o buffer

if (!GetTokenInformation(processToken, TokenPrivileges, processTokenPriv, firstStructSize, &secondStructSize)) {
	PrintError(_T("GetTokenInformation()"), true);
}
```

Com a estrutura `TOKEN_PRIVILEGES` completa do token, que contém um *array* de LUIDs e Atributos, podemos iterar sobre ela, comparando cada LUID com o LUID que extraímos com a função `LookupPrivilegeValue`.

O que temos que ter em mente na comparação feita durante a iteração, é que a [estrutura LUID](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-luid) separa seu valor em duas partes:

```cpp
typedef struct _LUID {
  DWORD LowPart;
  LONG  HighPart;
} LUID, *PLUID;
```

Logo, nossa comparação deve ser dividida em duas condições.

```cpp
// iterando sobre a estrutura TOKEN_PRIVILEGES
PLUID_AND_ATTRIBUTES luidStruct; // ponteiro para estrutura que receberá cada membro do array na iteração
bool seDebug = false; // variável de controle caso o privilégio exista

for (DWORD i = 0; i < processTokenPriv->PrivilegeCount; i++) {

	luidStruct = &processTokenPriv->Privileges[i];

	if ((luidStruct->Luid.LowPart == privilegeLuid.LowPart) && (luidStruct->Luid.HighPart == privilegeLuid.HighPart)) {
		_tprintf(_T("[+] SeDebugPrivilege encontrada para habilitar!\n\n"));
		seDebug = true;
		break;
	}
}

// caso não encontre o SeDebugPrivilege
if (!seDebug) {
	_tprintf(_T("[-] SeDebugPrivilege nao encontrado\nExecute com privilegios administrativos para enumerar todos os processos!\n\n"));
	return FALSE;
}
```

O código completo fica da seguinte forma:

```cpp
#include <Windows.h>
#include <tchar.h>
#include <WtsApi32.h>
#include <sddl.h>
#include <stdio.h>
#include <strsafe.h>

#pragma comment(lib, "wtsapi32")
#pragma comment(lib, "Advapi32")

#define MAX_ACCOUNTNAME_LEN 1024
#define MAX_DOMAINNAME_LEN 1024

void PrintError(LPCTSTR lpszFunction, bool exitProcess)
{
	// Retrieve the system error message for the last-error code

	LPVOID lpMsgBuf;
	LPVOID lpDisplayBuf;
	DWORD dw = GetLastError();

	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM |
		FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL,
		dw,
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf,
		0, NULL);

	// Display the error message and exit the process

	lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT,
		(lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR));
	StringCchPrintf((LPTSTR)lpDisplayBuf,
		LocalSize(lpDisplayBuf) / sizeof(TCHAR),
		TEXT("%s failed with error %d: %s"),
		lpszFunction, dw, lpMsgBuf);
	//MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK);
	_tprintf(TEXT("Erro: %s\n"), (LPTSTR)lpDisplayBuf);

	LocalFree(lpMsgBuf);
	LocalFree(lpDisplayBuf);
	if (exitProcess) ExitProcess(dw);
}

// função para habilitar SeDebugPrivilege
BOOL EnableSeDebug(void) {

	LUID privilegeLuid;
	if (!LookupPrivilegeValue(NULL, _T("SeDebugPrivilege"), &privilegeLuid)) {
		PrintError(_T("LookupPrivilegeValue()"), true);
	}

	// criando a estrutura do token
	TOKEN_PRIVILEGES tkPrivs;

	tkPrivs.PrivilegeCount = 1; // quantidade de privilégios a serem configurados
	tkPrivs.Privileges[0].Luid = privilegeLuid; // LUID do primeiro e único privilégio (SeDebugPrivilege)
	tkPrivs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED; // atributo que o privilégio receberá

	HANDLE currentProcessHandle = GetCurrentProcess();
	HANDLE processToken;

	if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &processToken)) {
		PrintError(_T("OpenProcessToken()"), true);
	}

	// coletando o tamanho da estrutura
	DWORD firstStructSize;
	GetTokenInformation(processToken, TokenPrivileges, NULL, 0, &firstStructSize);

	// invocando GetTokenInformation() com todos os parâmetros
	DWORD secondStructSize; // variável que receberá o tamanho da resposta na segunda execução
	PTOKEN_PRIVILEGES processTokenPriv; // ponteiro para estrutura TOKEN_PRIVILEGES que receberá a resposta

	processTokenPriv = (PTOKEN_PRIVILEGES)malloc(firstStructSize); // alocando o buffer

	if (!GetTokenInformation(processToken, TokenPrivileges, processTokenPriv, firstStructSize, &secondStructSize)) {
		PrintError(_T("GetTokenInformation()"), true);
	}

	// iterando sobre a estrutura TOKEN_PRIVILEGES
	PLUID_AND_ATTRIBUTES luidStruct; // ponteiro para estrutura que receberá cada membro do array na iteração
	bool seDebug = false; // variável de controle caso o privilégio exista

	for (DWORD i = 0; i < processTokenPriv->PrivilegeCount; i++) {

		luidStruct = &processTokenPriv->Privileges[i];

		if ((luidStruct->Luid.LowPart == privilegeLuid.LowPart) && (luidStruct->Luid.HighPart == privilegeLuid.HighPart)) {
			_tprintf(_T("[+] SeDebugPrivilege encontrada para habilitar!\n\n"));
			seDebug = true;
			break;
		}
	}

	// caso não encontre o SeDebugPrivilege
	if (!seDebug) {
		_tprintf(_T("[-] SeDebugPrivilege nao encontrado\nExecute com privilegios administrativos para enumerar todos os processos!\n\n"));
		return FALSE;
	}

	if (!AdjustTokenPrivileges(processToken, false, &tkPrivs, 0, NULL, NULL)) {
		PrintError(_T("AdjustTokenPrivileges()"), true);
	}

	return TRUE;
}

int main(void) {

	EnableSeDebug();

	DWORD level = 1; // variável que indica o tipo da estrutura PWTS_PROCESS_INFO_EX
	PWTS_PROCESS_INFO_EX processes = NULL; // variável que contém uma estrutura PWTS_PROCESS_INFO_EX vazia
	DWORD processCounter = 0; // variável que conterá a quantidade de estruturas enumeradas

	// enumerando os processos
	if (!WTSEnumerateProcessesEx(
		WTS_CURRENT_SERVER_HANDLE,
		&level,
		WTS_ANY_SESSION,
		(LPTSTR*)&processes,
		&processCounter)) {
		return 0;
	}

	// imprimindo dados das estruturas
	_tprintf(_T("Processos encontrados: %d\n\n"), processCounter);
	LPTSTR stringSID = NULL; // ponteiro do tipo string que receberá o SID do usuário que executa o processo
	PWTS_PROCESS_INFO_EX processesPtr = processes;

	_tprintf(_T("#\tPID\tHandles\tThreads\tProcess Name\tSID\tAccount\n\n"));

	// iterando sobre as estruturas
	for (DWORD counter = 1; counter <= processCounter; counter++) {

		_tprintf(_T("%d\t"), counter);
		_tprintf(_T("%d\t"), processes->ProcessId);
		_tprintf(_T("%d\t"), processes->HandleCount);
		_tprintf(_T("%d\t"), processes->NumberOfThreads);
		_tprintf(_T("%s\t"), processes->pProcessName);

		// convertendo o SID para string
		if (!ConvertSidToStringSid(processes->pUserSid, &stringSID)) {
			_tprintf(_T("-\t")); // imprime um traço, caso não consiga converter ou obter o SID
		}
		else {
			_tprintf(_T("%s\t"), stringSID);
			LocalFree((HLOCAL)stringSID); // libera a memória do ponteiro após seu uso
		}

		TCHAR accountName[MAX_ACCOUNTNAME_LEN];
		DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
		TCHAR domainName[MAX_DOMAINNAME_LEN];
		DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
		SID_NAME_USE peUse;

		// procura dados da conta através do SID
		if (!LookupAccountSid(
			NULL,
			processes->pUserSid,
			accountName,
			&accountNameBuffer,
			domainName,
			&domainNameBuffer,
			&peUse)) {
			_tprintf(_T("\n"));
		}
		else {
			_tprintf(_T("%s\\%s\n"), domainName, accountName);
		}

		processes++;
	}

	// liberando a memória da estrutura após o uso
	if (!WTSFreeMemoryEx(WTSTypeProcessInfoLevel1, processesPtr, processCounter)) {
		return 0;
	}
	processes = NULL;

	_tprintf(_T("\n\nPressione qualquer tecla para finalizar()\n"));

	getchar();
	return 0;
}
```

Quando executamos o programa com privilégios de usuário, temos a mensagem do `SeDebugPrivilege` não encontrado e os processos enumerados conforme possível.

![](/img/posts/Pasted%20image%2020240814190142.png)

Já quando executamos com privilégios administrativos (ou em um caso real onde o usuário tem tais permissões), temos a mensagem positiva e os processos enumerados.

![](/img/posts/Pasted%20image%2020240814190250.png)

## Requisitando Elevação

Uma vez que conseguimos descobrir se o usuário que executa o programa tem privilégios necessários, e, caso não tenha sabemos que precisamos executá-lo com privilégios, é possível fazer com que o próprio programa solicite execução como Administrador.

Obviamente, esta requisição irá ativar o UAC (*User Account Control*) que por sua vez solicitará ao usuário a permissão para executar como administrador, porém existem infinitas formas de se aproveitar disso, uma vez que o *malware* pode ser executado por uma vítima que acredita na integridade do programa.

O processo é basicamente simples, seguindo o fluxo do que foi feito até o momento, o programa é lançado, checa se os privilégios existem no token, caso não, ele simplesmente vai se "relançar" solicitando privilégios de administrador.

Para isso, usaremos a função [ShellExecuteEx](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexa) que tem como objetivo efetuar uma operação em determinado arquivo. Sua sintaxe é simples, e contém um único parâmetro.

```cpp
BOOL ShellExecuteExA(
  [in, out] SHELLEXECUTEINFOA *pExecInfo
);
```

Este parâmetro é um ponteiro para uma estrutura [SHELLEXECUTEINFO](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/ns-shellapi-shellexecuteinfoa) que contém e também irá receber informações sobre a aplicação executada.

Esta estrutura contém várias informações:

```cpp
typedef struct _SHELLEXECUTEINFOA {
  DWORD     cbSize;
  ULONG     fMask;
  HWND      hwnd;
  LPCSTR    lpVerb;
  LPCSTR    lpFile;
  LPCSTR    lpParameters;
  LPCSTR    lpDirectory;
  int       nShow;
  HINSTANCE hInstApp;
  void      *lpIDList;
  LPCSTR    lpClass;
  HKEY      hkeyClass;
  DWORD     dwHotKey;
  union {
    HANDLE hIcon;
    HANDLE hMonitor;
  } DUMMYUNIONNAME;
  HANDLE    hProcess;
} SHELLEXECUTEINFOA, *LPSHELLEXECUTEINFOA;
```

Não serão todos os parâmetros a ser usados no processo, porém alguns são obrigatórios e outros necessários:

- `cbSize` deve conter o tamanho em *bytes* da própria estrutura, necessário para sua inicialização;
- `fMask` contém um parâmetro que valida outras estruturas caso sejam utilizadas, em nosso caso não será necessário e pode ser setado para `SEE_MASK_DEFAULT`;
- `lpVerb` uma *string* que especifica a ação a ser tomada, e é essa a variável mais importante neste processo, pois, entre a lista de ações que ela aceita, temos a `runas` que executa uma aplicação como Administrador;

![](/img/posts/Pasted%20image%2020240814203034.png)

- `lpFile` uma *string* que contém o *path* do programa a ser executado, como podemos não ter o controle de onde o programa será armazenado, podemos usar a função [GetModuleFileName](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulefilenamea) para obter o *path* do processo atual; esta função exige um ponteiro para um *buffer* onde o *path* será armazenado e o tamanho deste *buffer* em *bytes*;
- `lpParameters` um ponteiro para uma variável que contenha parâmetros de execução da aplicação; se não houver parâmetros, deve ser setado como `NULL`;
- `lpDirectory` o ponteiro para uma variável que contenha o diretório de trabalho da aplicação; se for setado como `NULL` a função considera o diretório de execução;
- `nShow` a *flag* que especifica como a aplicação será exibida, segue a mesma tabela da função [ShowWindow](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-showwindow), em nosso caso usaremos `SW_SHOWNORMAL`.

Podemos então criar uma função que conterá este fluxo:

```cpp
void Relaunch(void) {

	SHELLEXECUTEINFO execInfo;
	WCHAR filePath[MAXFILEPATHLEN];
	DWORD filePathLen = MAXFILEPATHLEN;

	// coletando o path do programa
	GetModuleFileName(NULL, filePath, filePathLen);

	// preenchendo a estrutura SHELLEXECUTEINFO
	execInfo.cbSize = sizeof(SHELLEXECUTEINFO);
	execInfo.fMask = SEE_MASK_DEFAULT;
	execInfo.lpVerb = _T("runas");
	execInfo.lpFile = filePath;
	execInfo.lpParameters = NULL;
	execInfo.lpDirectory = NULL;
	execInfo.nShow = SW_SHOWNORMAL;

	ShellExecuteEx(&execInfo);
}
```

E em nossa função `main` quando invocamos a `EnableSeDebug`, podemos criar uma condição onde, caso o privilégio não seja encontrado no reconhecimento inicial, ele invoca a função `Relaunch`. A implementação fica da seguinte forma:

```cpp
#include <Windows.h>
#include <tchar.h>
#include <WtsApi32.h>
#include <sddl.h>
#include <stdio.h>
#include <strsafe.h>
#include <shellapi.h>
#include <winnt.h>

#pragma comment(lib, "wtsapi32")
#pragma comment(lib, "Advapi32")

#define MAX_ACCOUNTNAME_LEN 1024
#define MAX_DOMAINNAME_LEN 1024
#define	MAXFILEPATHLEN	5000

void Relaunch(void) {

	SHELLEXECUTEINFO execInfo;
	WCHAR filePath[MAXFILEPATHLEN];
	DWORD filePathLen = MAXFILEPATHLEN;

	// coletando o path do programa
	GetModuleFileName(NULL, filePath, filePathLen);

	// preenchendo a estrutura SHELLEXECUTEINFO
	execInfo.cbSize = sizeof(SHELLEXECUTEINFO);
	execInfo.fMask = SEE_MASK_DEFAULT;
	execInfo.lpVerb = _T("runas");
	execInfo.lpFile = filePath;
	execInfo.lpParameters = NULL;
	execInfo.lpDirectory = NULL;
	execInfo.nShow = SW_SHOWNORMAL;

	ShellExecuteEx(&execInfo);
}

void PrintError(LPCTSTR lpszFunction, bool exitProcess)
{
	// Retrieve the system error message for the last-error code

	LPVOID lpMsgBuf;
	LPVOID lpDisplayBuf;
	DWORD dw = GetLastError();

	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM |
		FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL,
		dw,
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf,
		0, NULL);

	// Display the error message and exit the process

	lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT,
		(lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR));
	StringCchPrintf((LPTSTR)lpDisplayBuf,
		LocalSize(lpDisplayBuf) / sizeof(TCHAR),
		TEXT("%s failed with error %d: %s"),
		lpszFunction, dw, lpMsgBuf);
	//MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK);
	_tprintf(TEXT("Erro: %s\n"), (LPTSTR)lpDisplayBuf);

	LocalFree(lpMsgBuf);
	LocalFree(lpDisplayBuf);
	if (exitProcess) ExitProcess(dw);
}

// função para habilitar SeDebugPrivilege
BOOL EnableSeDebug(void) {

	LUID privilegeLuid;
	if (!LookupPrivilegeValue(NULL, _T("SeDebugPrivilege"), &privilegeLuid)) {
		PrintError(_T("LookupPrivilegeValue()"), true);
	}

	// criando a estrutura do token
	TOKEN_PRIVILEGES tkPrivs;

	tkPrivs.PrivilegeCount = 1; // quantidade de privilégios a serem configurados
	tkPrivs.Privileges[0].Luid = privilegeLuid; // LUID do primeiro e único privilégio (SeDebugPrivilege)
	tkPrivs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED; // atributo que o privilégio receberá

	HANDLE currentProcessHandle = GetCurrentProcess();
	HANDLE processToken;

	if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &processToken)) {
		PrintError(_T("OpenProcessToken()"), true);
	}

	// coletando o tamanho da estrutura
	DWORD firstStructSize;
	GetTokenInformation(processToken, TokenPrivileges, NULL, 0, &firstStructSize);

	// invocando GetTokenInformation() com todos os parâmetros
	DWORD secondStructSize; // variável que receberá o tamanho da resposta na segunda execução
	PTOKEN_PRIVILEGES processTokenPriv; // ponteiro para estrutura TOKEN_PRIVILEGES que receberá a resposta

	processTokenPriv = (PTOKEN_PRIVILEGES)malloc(firstStructSize); // alocando o buffer

	if (!GetTokenInformation(processToken, TokenPrivileges, processTokenPriv, firstStructSize, &secondStructSize)) {
		PrintError(_T("GetTokenInformation()"), true);
	}

	// iterando sobre a estrutura TOKEN_PRIVILEGES
	PLUID_AND_ATTRIBUTES luidStruct; // ponteiro para estrutura que receberá cada membro do array na iteração
	bool seDebug = false; // variável de controle caso o privilégio exista

	for (DWORD i = 0; i < processTokenPriv->PrivilegeCount; i++) {

		luidStruct = &processTokenPriv->Privileges[i];

		if ((luidStruct->Luid.LowPart == privilegeLuid.LowPart) && (luidStruct->Luid.HighPart == privilegeLuid.HighPart)) {
			_tprintf(_T("[+] SeDebugPrivilege encontrada para habilitar!\n\n"));
			seDebug = true;
			break;
		}
	}

	// caso não encontre o SeDebugPrivilege
	if (!seDebug) {
		_tprintf(_T("[-] SeDebugPrivilege nao encontrado\nExecutando com privilegios administrativos!\n\n"));
		return FALSE;
	}

	if (!AdjustTokenPrivileges(processToken, false, &tkPrivs, 0, NULL, NULL)) {
		PrintError(_T("AdjustTokenPrivileges()"), true);
	}



	return TRUE;
}

int main(void) {

	if (!EnableSeDebug()) {
		Relaunch();
		ExitProcess(-1);
	}
	else {
		_tprintf(_T("Adicionando privilegio de debug!\n\n"));
	}

	DWORD level = 1; // variável que indica o tipo da estrutura PWTS_PROCESS_INFO_EX
	PWTS_PROCESS_INFO_EX processes = NULL; // variável que contém uma estrutura PWTS_PROCESS_INFO_EX vazia
	DWORD processCounter = 0; // variável que conterá a quantidade de estruturas enumeradas

	// enumerando os processos
	if (!WTSEnumerateProcessesEx(
		WTS_CURRENT_SERVER_HANDLE,
		&level,
		WTS_ANY_SESSION,
		(LPTSTR*)&processes,
		&processCounter)) {
		return 0;
	}

	// imprimindo dados das estruturas
	_tprintf(_T("Processos encontrados: %d\n\n"), processCounter);
	LPTSTR stringSID = NULL; // ponteiro do tipo string que receberá o SID do usuário que executa o processo
	PWTS_PROCESS_INFO_EX processesPtr = processes;

	_tprintf(_T("#\tPID\tHandles\tThreads\tProcess Name\tSID\tAccount\n\n"));

	// iterando sobre as estruturas
	for (DWORD counter = 1; counter <= processCounter; counter++) {

		_tprintf(_T("%d\t"), counter);
		_tprintf(_T("%d\t"), processes->ProcessId);
		_tprintf(_T("%d\t"), processes->HandleCount);
		_tprintf(_T("%d\t"), processes->NumberOfThreads);
		_tprintf(_T("%s\t"), processes->pProcessName);

		// convertendo o SID para string
		if (!ConvertSidToStringSid(processes->pUserSid, &stringSID)) {
			_tprintf(_T("-\t")); // imprime um traço, caso não consiga converter ou obter o SID
		}
		else {
			_tprintf(_T("%s\t"), stringSID);
			LocalFree((HLOCAL)stringSID); // libera a memória do ponteiro após seu uso
		}

		TCHAR accountName[MAX_ACCOUNTNAME_LEN];
		DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
		TCHAR domainName[MAX_DOMAINNAME_LEN];
		DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
		SID_NAME_USE peUse;

		// procura dados da conta através do SID
		if (!LookupAccountSid(
			NULL,
			processes->pUserSid,
			accountName,
			&accountNameBuffer,
			domainName,
			&domainNameBuffer,
			&peUse)) {
			_tprintf(_T("\n"));
		}
		else {
			_tprintf(_T("%s\\%s\n"), domainName, accountName);
		}

		processes++;
	}

	// liberando a memória da estrutura após o uso
	if (!WTSFreeMemoryEx(WTSTypeProcessInfoLevel1, processesPtr, processCounter)) {
		return 0;
	}
	processes = NULL;

	_tprintf(_T("\n\nPressione qualquer tecla para finalizar()\n"));

	getchar();
	return 0;
}
```

Quando executamos o programa com usuário comum, temos a mensagem de aviso:

![](/img/posts/Pasted%20image%2020240814211408.png)

Logo em seguida, o UAC é acionado:

![](/img/posts/Pasted%20image%2020240814211434.png)

Quando a execução é aceita, temos a enumeração completa dos processos.

![](/img/posts/Pasted%20image%2020240814211518.png)

## *Token Dumper*

Com tudo apresentado até o momento, já é níitido que as informações que podemos obter dos tokens de acesso são tão, se não mais, valiosas quanto as informações dos processos.

Quando implementamos a função `EnableSeDebug` para habilitar o privilégio `SeDebugPrivilege`, vimos que, ao usar a função `GetTokenInformation` precisamos informar o valor que queremos obter da classe `TOKEN_INFORMATION_CLASS`.

Uma simples observada nesta classe nos mostra a quantidade de informações que podemos extrair a partir dela:

```cpp
typedef enum _TOKEN_INFORMATION_CLASS {
  TokenUser = 1,
  TokenGroups,
  TokenPrivileges,
  TokenOwner,
  TokenPrimaryGroup,
  TokenDefaultDacl,
  TokenSource,
  TokenType,
  TokenImpersonationLevel,
  TokenStatistics,
  TokenRestrictedSids,
  TokenSessionId,
  TokenGroupsAndPrivileges,
  TokenSessionReference,
  TokenSandBoxInert,
  TokenAuditPolicy,
  TokenOrigin,
  TokenElevationType,
  TokenLinkedToken,
  TokenElevation,
  TokenHasRestrictions,
  TokenAccessInformation,
  TokenVirtualizationAllowed,
  TokenVirtualizationEnabled,
  TokenIntegrityLevel,
  TokenUIAccess,
  TokenMandatoryPolicy,
  TokenLogonSid,
  TokenIsAppContainer,
  TokenCapabilities,
  TokenAppContainerSid,
  TokenAppContainerNumber,
  TokenUserClaimAttributes,
  TokenDeviceClaimAttributes,
  TokenRestrictedUserClaimAttributes,
  TokenRestrictedDeviceClaimAttributes,
  TokenDeviceGroups,
  TokenRestrictedDeviceGroups,
  TokenSecurityAttributes,
  TokenIsRestricted,
  TokenProcessTrustLevel,
  TokenPrivateNameSpace,
  TokenSingletonAttributes,
  TokenBnoIsolation,
  TokenChildProcessFlags,
  TokenIsLessPrivilegedAppContainer,
  TokenIsSandboxed,
  TokenIsAppSilo,
  TokenLoggingInformation,
  MaxTokenInfoClass
} TOKEN_INFORMATION_CLASS, *PTOKEN_INFORMATION_CLASS;
```

Cada membro desta classe tem um formato, alguns são inteiros, outros *strings*, *arrays*, estruturas, etc. Para cada um que queremos obter, um tratamento específico deve ser feito.

Para deixar nosso programa de *process listing* mais completo possível, após a enumeração dos processos em execução, podemos solicitar o PID de um processo específico e enumerar várias informações relevantes do seu token.

Como vimos anteriormente, temos que criar um *handle* para o token passando o tipo de permissão que queremos, que no caso foi `MAXIMUM_ALLOWED`. Também vimos que precisamos de um *handle* para o processo do qual queremos extrair o token, no caso feito até o momento, utilizamos a função `GetCurrentProcess`, pois estávamos extraindo o token do pŕoprio programa.

Para criar um *handle* de outro processo em execução, podemos usar a função `OpenProcess` bastante utilizada nos artigos anteriores dessa série.

Basicamente, a implementação no código para solicitar o PID, criar um *handle* para um processo baseado no PID e em seguida criar um *handle* para o token do processo é:

```cpp
// ********************************** inicio do dump do token
int pid;

_tprintf(_T("\n\nFazer o dump do token para qual PID?: "));
scanf_s("%d[^\n]", &pid);
_tprintf(_T("\n[+] Fazendo dump do token para o PID: %d\n"), pid);

// criando um handle para o processo do qual extrairemos o token
HANDLE processHandle;
if (NULL == (processHandle = OpenProcess(MAXIMUM_ALLOWED, false, pid))) {

	PrintError(_T("OpenProcess()"), false);
}

// criando um handle para o token
HANDLE processToken;

if (!OpenProcessToken(processHandle, MAXIMUM_ALLOWED, &processToken)) {

	PrintError(_T("OpenProcessToken()"), false);
}
```

Com o *handle* dp token em mãos, podemos utilizar a função `GetTokenInformation` para coletar qualquer informação listada na classe `TOKEN_INFORMATION_CLASS`.

Neste ponto existe algo a se considerar: existem muitas informações nessa classe, algumas mais relevantes que outras, a depender do contexto de enumeração e exploração. Para fins deste artigo, decidi enumerar as seguintes informações:

- `TokenOwner`
- `TokenUser`
- `TokenPrimaryGroup`
- `TokenGroups`
- `TokenPrivileges`
- `TokenSource`
- `TokenType`
- `TokenElevation`

Para cada um destes membros da classe, temos um tipo, portanto foi criada uma função para o *parsing* de cada uma, as funções ficaram da seguinte forma:

```cpp
void GetTokenOwner(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenOwner);
	DWORD structLen;

	PTOKEN_OWNER tokenInfo = (PTOKEN_OWNER)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenOwner, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN OWNER: \n"));
	PrintSidInfo(tokenInfo->Owner);

	free(tokenInfo);


}

void GetTokenUser(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenUser);
	DWORD structLen;

	PTOKEN_USER tokenInfo = (PTOKEN_USER)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenUser, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN USER: \n"));
	PrintSidInfo(tokenInfo->User.Sid);

	free(tokenInfo);


}

void GetTokenPrimaryGroup(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenPrimaryGroup);
	DWORD structLen;

	PTOKEN_PRIMARY_GROUP tokenInfo = (PTOKEN_PRIMARY_GROUP)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenPrimaryGroup, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN PRIMARY GROUP: \n"));
	PrintSidInfo(tokenInfo->PrimaryGroup);

	free(tokenInfo);


}

void GetTokenGroups(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenGroups);
	DWORD structLen;

	PTOKEN_GROUPS tokenInfo = (PTOKEN_GROUPS)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenGroups, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN GROUPS: \n"));
	for (DWORD i = 0; i < tokenInfo->GroupCount; i++) {

		PrintSidInfo(tokenInfo->Groups[i].Sid);
	}
	

	free(tokenInfo);


}

void GetTokenPrivileges(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenPrivileges);
	DWORD structLen;

	PTOKEN_PRIVILEGES tokenInfo = (PTOKEN_PRIVILEGES)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenPrivileges, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	TCHAR privName[MAX_PRIV_LEN];
	DWORD privNameSize = sizeof(privName);

	_tprintf(_T("************************ TOKEN PRIVILEGES: \n"));
	for (DWORD i = 0; i < tokenInfo->PrivilegeCount; i++) {

		privNameSize = sizeof(privName);

		if (!LookupPrivilegeName(NULL, &tokenInfo->Privileges[i].Luid, privName, &privNameSize)) {
			PrintError(_T("LookupPrivilegeName()"), false);
		}
		else {
			if (tokenInfo->Privileges[i].Attributes & SE_PRIVILEGE_ENABLED) {
				_tprintf(_T("%3d. Enabled        %s\n"), i + 1, privName);
			}
			else {
				_tprintf(_T("%3d. Disabled       %s\n"), i + 1, privName);
			}
		}
	}
	_tprintf(_T("\n"));

	free(tokenInfo);


}

void GetTokenSource(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_SOURCE tokenInfo = (PTOKEN_SOURCE)malloc(sizeof(TOKEN_SOURCE));

	if (!GetTokenInformation(processToken, TokenSource, (LPVOID)tokenInfo, sizeof(TOKEN_SOURCE), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN SOURCE: \n"));
	for (DWORD i = 0; i < 8; i++) {

		if (tokenInfo->SourceName[i]) {
			_tprintf(_T("%c"), tokenInfo->SourceName[i]);
		}
	}
	_tprintf(_T("\n\n"));

	free(tokenInfo);


}

void GetTokenType(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_TYPE tokenInfo = (PTOKEN_TYPE)malloc(sizeof(PTOKEN_TYPE));

	if (!GetTokenInformation(processToken, TokenType, (LPVOID)tokenInfo, sizeof(PTOKEN_TYPE), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN TYPE: \n"));

	if (*tokenInfo == TokenPrimary) {

		_tprintf(_T("TokenPrimary"));
	}
	else {
		_tprintf(_T("TokenImpersonation"));

		// procurando o tipo de impersonation
		PSECURITY_IMPERSONATION_LEVEL tokenImpLevel = (PSECURITY_IMPERSONATION_LEVEL)malloc(sizeof(SECURITY_IMPERSONATION_LEVEL));
		if (!GetTokenInformation(processToken, TokenImpersonationLevel, (LPVOID)tokenImpLevel, sizeof(TOKEN_TYPE), &structLen)) {

			PrintError(_T("GetTokenInformation()"), false);
		}
		else {
			switch (*tokenImpLevel) {

			case SecurityAnonymous:
				_tprintf(_T("Impersonation Level: SecurityAnonymous\n"));
					break;

			case SecurityIdentification:
				_tprintf(_T("Impersonation Level: SecurityIdentification\n"));
				break;

			case SecurityImpersonation:
				_tprintf(_T("Impersonation Level: SecurityImpersonation\n"));
				break;

			case SecurityDelegation:
				_tprintf(_T("Impersonation Level: SecurityDelegation\n"));
				break;
			}
		}
	}
	_tprintf(_T("\n\n"));

	free(tokenInfo);


}

void GetTokenElevation(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_ELEVATION tokenInfo = (PTOKEN_ELEVATION)malloc(sizeof(TOKEN_ELEVATION));

	if (!GetTokenInformation(processToken, TokenElevation, (LPVOID)tokenInfo, sizeof(TOKEN_ELEVATION), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN ELEVATION: \n"));
	if (tokenInfo->TokenIsElevated) {

		_tprintf(_T("Token Elevated: TRUE\n"));
	}
	else {
		_tprintf(_T("Token Elevated: FALSE\n"));
	}
	_tprintf(_T("\n"));

	free(tokenInfo);


}
```

Na função `main` do programa, eu invoco cada uma destas funções de *parsing* passando o *handle* do token e, ao final, eu fecho os *handles* abertos.

Abaixo o código completo:

```cpp
#include <Windows.h>
#include <tchar.h>
#include <WtsApi32.h>
#include <sddl.h>
#include <stdio.h>
#include <strsafe.h>
#include <shellapi.h>
#include <winnt.h>

#pragma comment(lib, "wtsapi32")
#pragma comment(lib, "Advapi32")

#define MAX_ACCOUNTNAME_LEN 1024
#define MAX_DOMAINNAME_LEN 1024
#define	MAX_FILE_PATH_LEN	5000
#define MAX_PRIV_LEN	5000

void PrintError(LPCTSTR lpszFunction, bool exitProcess)
{
	// Retrieve the system error message for the last-error code

	LPVOID lpMsgBuf;
	LPVOID lpDisplayBuf;
	DWORD dw = GetLastError();

	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM |
		FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL,
		dw,
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf,
		0, NULL);

	// Display the error message and exit the process

	lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT,
		(lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR));
	StringCchPrintf((LPTSTR)lpDisplayBuf,
		LocalSize(lpDisplayBuf) / sizeof(TCHAR),
		TEXT("%s failed with error %d: %s"),
		lpszFunction, dw, lpMsgBuf);
	//MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK);
	_tprintf(TEXT("Erro: %s\n"), (LPTSTR)lpDisplayBuf);

	LocalFree(lpMsgBuf);
	LocalFree(lpDisplayBuf);
	if (exitProcess) ExitProcess(dw);
}

// coleta o tamanho da struct
DWORD GetTokenInfoLen(HANDLE processToken, TOKEN_INFORMATION_CLASS info) {

	DWORD structSize;

	GetTokenInformation(processToken, info, NULL, 0, &structSize);

	return structSize;
}

void SIDType(SID_NAME_USE type) {

	switch (type) {

	case SidTypeUser:

		_tprintf(_T("SID Type: SidTypeUser: SID de usuario.\n"));
		break;

	case SidTypeGroup:
		_tprintf(_T("SID Type: SidTypeGroup: SID de grupo\n"));
		break;

	case SidTypeDomain:
		_tprintf(_T("SID Type: SidTypeDomain: SID de dominio\n"));
		break;

	case SidTypeAlias:
		_tprintf(_T("SID Type: SidTypeAlias: SID alias\n"));
		break;

	case SidTypeWellKnownGroup:
		_tprintf(_T("SID Type: SidTypeWellKnownGroup: SID de grupo well-known\n"));
		break;

	case SidTypeDeletedAccount:
		_tprintf(_T("SID Type: SidTypeDeletedAccount: SID de conta deletada\n"));
		break;

	case SidTypeInvalid:
		_tprintf(_T("SID Type: SidTypeInvalid: SID invalido\n"));
		break;

	case SidTypeUnknown:
		_tprintf(_T("SID Type: SidTypeUnknown: SID nao conhecido\n"));
		break;

	case SidTypeComputer:
		_tprintf(_T("SID Type: SidTypeComputer: SID de computador\n"));
		break;

	case SidTypeLabel:
		_tprintf(_T("SID Type: SidTypeLabel: SID mandatory integrity label\n"));
		break;
	}
}

void PrintSidInfo(PSID psid) {

	LPTSTR stringSID = NULL;

	if (!ConvertSidToStringSid(psid, &stringSID)) {
		PrintError(_T("ConvertSidToStringSid()"), false);
	}

	TCHAR accountName[MAX_ACCOUNTNAME_LEN];
	DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
	TCHAR domainName[MAX_DOMAINNAME_LEN];
	DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
	SID_NAME_USE peUse;

	if (!LookupAccountSid(NULL, psid, accountName, &accountNameBuffer, domainName, &domainNameBuffer, &peUse)) {
		PrintError(_T("LookupAccountSid()"), false);
		return;
	}

	_tprintf(_T("SID: %s\n"), stringSID);
	SIDType(peUse);
	_tprintf(_T("Conta: %s\\%s\n\n"), domainName, accountName);
}

void GetTokenOwner(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenOwner);
	DWORD structLen;

	PTOKEN_OWNER tokenInfo = (PTOKEN_OWNER)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenOwner, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN OWNER: \n"));
	PrintSidInfo(tokenInfo->Owner);

	free(tokenInfo);


}

void GetTokenUser(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenUser);
	DWORD structLen;

	PTOKEN_USER tokenInfo = (PTOKEN_USER)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenUser, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN USER: \n"));
	PrintSidInfo(tokenInfo->User.Sid);

	free(tokenInfo);


}

void GetTokenPrimaryGroup(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenPrimaryGroup);
	DWORD structLen;

	PTOKEN_PRIMARY_GROUP tokenInfo = (PTOKEN_PRIMARY_GROUP)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenPrimaryGroup, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN PRIMARY GROUP: \n"));
	PrintSidInfo(tokenInfo->PrimaryGroup);

	free(tokenInfo);


}

void GetTokenGroups(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenGroups);
	DWORD structLen;

	PTOKEN_GROUPS tokenInfo = (PTOKEN_GROUPS)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenGroups, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN GROUPS: \n"));
	for (DWORD i = 0; i < tokenInfo->GroupCount; i++) {

		PrintSidInfo(tokenInfo->Groups[i].Sid);
	}
	

	free(tokenInfo);


}

void GetTokenPrivileges(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenPrivileges);
	DWORD structLen;

	PTOKEN_PRIVILEGES tokenInfo = (PTOKEN_PRIVILEGES)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenPrivileges, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	TCHAR privName[MAX_PRIV_LEN];
	DWORD privNameSize = sizeof(privName);

	_tprintf(_T("************************ TOKEN PRIVILEGES: \n"));
	for (DWORD i = 0; i < tokenInfo->PrivilegeCount; i++) {

		privNameSize = sizeof(privName);

		if (!LookupPrivilegeName(NULL, &tokenInfo->Privileges[i].Luid, privName, &privNameSize)) {
			PrintError(_T("LookupPrivilegeName()"), false);
		}
		else {
			if (tokenInfo->Privileges[i].Attributes & SE_PRIVILEGE_ENABLED) {
				_tprintf(_T("%3d. Enabled        %s\n"), i + 1, privName);
			}
			else {
				_tprintf(_T("%3d. Disabled       %s\n"), i + 1, privName);
			}
		}
	}
	_tprintf(_T("\n"));

	free(tokenInfo);


}

void GetTokenSource(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_SOURCE tokenInfo = (PTOKEN_SOURCE)malloc(sizeof(TOKEN_SOURCE));

	if (!GetTokenInformation(processToken, TokenSource, (LPVOID)tokenInfo, sizeof(TOKEN_SOURCE), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN SOURCE: \n"));
	for (DWORD i = 0; i < 8; i++) {

		if (tokenInfo->SourceName[i]) {
			_tprintf(_T("%c"), tokenInfo->SourceName[i]);
		}
	}
	_tprintf(_T("\n\n"));

	free(tokenInfo);


}

void GetTokenType(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_TYPE tokenInfo = (PTOKEN_TYPE)malloc(sizeof(PTOKEN_TYPE));

	if (!GetTokenInformation(processToken, TokenType, (LPVOID)tokenInfo, sizeof(PTOKEN_TYPE), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN TYPE: \n"));

	if (*tokenInfo == TokenPrimary) {

		_tprintf(_T("TokenPrimary"));
	}
	else {
		_tprintf(_T("TokenImpersonation"));

		// procurando o tipo de impersonation
		PSECURITY_IMPERSONATION_LEVEL tokenImpLevel = (PSECURITY_IMPERSONATION_LEVEL)malloc(sizeof(SECURITY_IMPERSONATION_LEVEL));
		if (!GetTokenInformation(processToken, TokenImpersonationLevel, (LPVOID)tokenImpLevel, sizeof(TOKEN_TYPE), &structLen)) {

			PrintError(_T("GetTokenInformation()"), false);
		}
		else {
			switch (*tokenImpLevel) {

			case SecurityAnonymous:
				_tprintf(_T("Impersonation Level: SecurityAnonymous\n"));
					break;

			case SecurityIdentification:
				_tprintf(_T("Impersonation Level: SecurityIdentification\n"));
				break;

			case SecurityImpersonation:
				_tprintf(_T("Impersonation Level: SecurityImpersonation\n"));
				break;

			case SecurityDelegation:
				_tprintf(_T("Impersonation Level: SecurityDelegation\n"));
				break;
			}
		}
	}
	_tprintf(_T("\n\n"));

	free(tokenInfo);


}

void GetTokenElevation(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_ELEVATION tokenInfo = (PTOKEN_ELEVATION)malloc(sizeof(TOKEN_ELEVATION));

	if (!GetTokenInformation(processToken, TokenElevation, (LPVOID)tokenInfo, sizeof(TOKEN_ELEVATION), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN ELEVATION: \n"));
	if (tokenInfo->TokenIsElevated) {

		_tprintf(_T("Token Elevated: TRUE\n"));
	}
	else {
		_tprintf(_T("Token Elevated: FALSE\n"));
	}
	_tprintf(_T("\n"));

	free(tokenInfo);


}

void Relaunch(void) {

	SHELLEXECUTEINFO execInfo;
	WCHAR filePath[MAX_FILE_PATH_LEN];
	DWORD filePathLen = MAX_FILE_PATH_LEN;

	// coletando o path do programa
	GetModuleFileName(NULL, filePath, filePathLen);

	// preenchendo a estrutura SHELLEXECUTEINFO
	execInfo.cbSize = sizeof(SHELLEXECUTEINFO);
	execInfo.fMask = SEE_MASK_DEFAULT;
	execInfo.lpVerb = _T("runas");
	execInfo.lpFile = filePath;
	execInfo.lpParameters = NULL;
	execInfo.lpDirectory = NULL;
	execInfo.nShow = SW_SHOWNORMAL;

	ShellExecuteEx(&execInfo);
}

// função para habilitar SeDebugPrivilege
BOOL EnableSeDebug(void) {

	LUID privilegeLuid;
	if (!LookupPrivilegeValue(NULL, _T("SeDebugPrivilege"), &privilegeLuid)) {
		PrintError(_T("LookupPrivilegeValue()"), true);
	}

	// criando a estrutura do token
	TOKEN_PRIVILEGES tkPrivs;

	tkPrivs.PrivilegeCount = 1; // quantidade de privilégios a serem configurados
	tkPrivs.Privileges[0].Luid = privilegeLuid; // LUID do primeiro e único privilégio (SeDebugPrivilege)
	tkPrivs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED; // atributo que o privilégio receberá

	HANDLE currentProcessHandle = GetCurrentProcess();
	HANDLE processToken;

	if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &processToken)) {
		PrintError(_T("OpenProcessToken()"), true);
	}

	// coletando o tamanho da estrutura
	DWORD firstStructSize = GetTokenInfoLen(processToken, TokenPrivileges);

	// invocando GetTokenInformation() com todos os parâmetros
	DWORD secondStructSize; // variável que receberá o tamanho da resposta na segunda execução
	PTOKEN_PRIVILEGES processTokenPriv; // ponteiro para estrutura TOKEN_PRIVILEGES que receberá a resposta

	processTokenPriv = (PTOKEN_PRIVILEGES)malloc(firstStructSize); // alocando o buffer

	if (!GetTokenInformation(processToken, TokenPrivileges, processTokenPriv, firstStructSize, &secondStructSize)) {
		PrintError(_T("GetTokenInformation()"), true);
	}

	// iterando sobre a estrutura TOKEN_PRIVILEGES
	PLUID_AND_ATTRIBUTES luidStruct; // ponteiro para estrutura que receberá cada membro do array na iteração
	bool seDebug = false; // variável de controle caso o privilégio exista

	for (DWORD i = 0; i < processTokenPriv->PrivilegeCount; i++) {

		luidStruct = &processTokenPriv->Privileges[i];

		if ((luidStruct->Luid.LowPart == privilegeLuid.LowPart) && (luidStruct->Luid.HighPart == privilegeLuid.HighPart)) {
			_tprintf(_T("[+] SeDebugPrivilege encontrada para habilitar!\n\n"));
			seDebug = true;
			break;
		}
	}

	// caso não encontre o SeDebugPrivilege
	if (!seDebug) {
		_tprintf(_T("[-] SeDebugPrivilege nao encontrado\nExecutando com privilegios administrativos!\n\n"));
		return FALSE;
	}

	if (!AdjustTokenPrivileges(processToken, false, &tkPrivs, 0, NULL, NULL)) {
		PrintError(_T("AdjustTokenPrivileges()"), true);
	}



	return TRUE;
}

int main(void) {

	if (!EnableSeDebug()) {
		Relaunch();
		ExitProcess(-1);
	}
	else {
		_tprintf(_T("Adicionando privilegio de debug!\n\n"));
	}

	DWORD level = 1; // variável que indica o tipo da estrutura PWTS_PROCESS_INFO_EX
	PWTS_PROCESS_INFO_EX processes = NULL; // variável que contém uma estrutura PWTS_PROCESS_INFO_EX vazia
	DWORD processCounter = 0; // variável que conterá a quantidade de estruturas enumeradas

	// enumerando os processos
	if (!WTSEnumerateProcessesEx(
		WTS_CURRENT_SERVER_HANDLE,
		&level,
		WTS_ANY_SESSION,
		(LPTSTR*)&processes,
		&processCounter)) {
		return 0;
	}

	// imprimindo dados das estruturas
	_tprintf(_T("Processos encontrados: %d\n\n"), processCounter);
	LPTSTR stringSID = NULL; // ponteiro do tipo string que receberá o SID do usuário que executa o processo
	PWTS_PROCESS_INFO_EX processesPtr = processes;

	_tprintf(_T("#\tPID\tHandles\tThreads\tProcess Name\t\tAccount\n\n"));

	// iterando sobre as estruturas
	for (DWORD counter = 1; counter <= processCounter; counter++) {

		_tprintf(_T("%d\t"), counter);
		_tprintf(_T("%d\t"), processes->ProcessId);
		_tprintf(_T("%d\t"), processes->HandleCount);
		_tprintf(_T("%d\t"), processes->NumberOfThreads);
		_tprintf(_T("%s\t\t"), processes->pProcessName);

		TCHAR accountName[MAX_ACCOUNTNAME_LEN];
		DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
		TCHAR domainName[MAX_DOMAINNAME_LEN];
		DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
		SID_NAME_USE peUse;

		// procura dados da conta através do SID
		if (!LookupAccountSid(
			NULL,
			processes->pUserSid,
			accountName,
			&accountNameBuffer,
			domainName,
			&domainNameBuffer,
			&peUse)) {
			_tprintf(_T("\n"));
		}
		else {
			_tprintf(_T("%s\\%s\n"), domainName, accountName);
		}

		processes++;
	}

	// liberando a memória da estrutura após o uso
	if (!WTSFreeMemoryEx(WTSTypeProcessInfoLevel1, processesPtr, processCounter)) {
		return 0;
	}
	processes = NULL;

	// ********************************** inicio do dump do token
	int pid;

	_tprintf(_T("\n\nFazer o dump do token para qual PID?: "));
	scanf_s("%d[^\n]", &pid);
	_tprintf(_T("\n[+] Fazendo dump do token para o PID: %d\n"), pid);

	// criando um handle para o processo do qual extrairemos o token
	HANDLE processHandle;
	if (NULL == (processHandle = OpenProcess(MAXIMUM_ALLOWED, false, pid))) {
		PrintError(_T("OpenProcess()"), false);

	}


	// criando um handle para o token
	HANDLE processToken;

	if (!OpenProcessToken(processHandle, MAXIMUM_ALLOWED, &processToken)) {
		PrintError(_T("OpenProcessToken()"), false);
	}

	// parsing das informações do token
	_tprintf(_T("[+] Parsing das informacoes do token:\n"));

	_tprintf(_T("\n*********************INFORMACOES DO TOKEN**********************\n\n"));
	GetTokenOwner(processToken);
	GetTokenUser(processToken);
	GetTokenPrimaryGroup(processToken);
	GetTokenGroups(processToken);
	GetTokenPrivileges(processToken);
	GetTokenSource(processToken);
	GetTokenType(processToken);
	GetTokenElevation(processToken);

	CloseHandle(processHandle);
	CloseHandle(processToken);

	_tprintf(_T("\n\nPressione qualquer tecla para finalizar()\n"));

	getchar();
	getchar();
	return 0;
}
```

Quando executamos o programa como um usuário, ele continua solicitando elevação para administrador e enumera tosos os processos, porém, no final, ele solicita o PID do processo alvo para o *dump* do token:

![](/img/posts/Pasted%20image%2020240815132229.png)

Quando inserimos um PID válido, ele enumera as informações do token.

![](/img/posts/Pasted%20image%2020240815132347.png)

### Ajustando detalhes

A enumeração ainda tem uma pequena falha, mesmo o executando como administrador, ainda existem processos que retornam erro ao enumarar. No exemplo abaixo, tentaremos enumerar o PID **92** do processo ***Registry***.

![](/img/posts/Pasted%20image%2020240815134748.png)

![](/img/posts/Pasted%20image%2020240815134829.png)

O programa retorna falha em todas as tentativas, pois não conseguiu abrir um *handle* para o processo.

Isso ocorre, pois estamos utilizando os direitos `MAXIMUM_ALLOWED` na função `OpenProcess`, porém, existem processos protegidos pelo próprio sistema que não permitem o uso destes direitos.

Se olharmos a documentação do [*Process Security and Access Rights*](https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights) veremos que direitos gerais e de modificação não são permitidos para processos protegidos, neste caso, podemos utilizar os direitos `PROCESS_QUERY_LIMITED_INFORMATION` para enumerá-los.

![](/img/posts/Pasted%20image%2020240815135154.png)

Assim, podemos continuar utilizando o `MAXIMUM_ALLOWED` inicialmente, porém criar uma condição de erro que altere para `PROCESS_QUERY_LIMITED_INFORMATION` quando não for possível criar o *handle*. O código final com a implementação fica da seguinte forma:

```cpp
#include <Windows.h>
#include <tchar.h>
#include <WtsApi32.h>
#include <sddl.h>
#include <stdio.h>
#include <strsafe.h>
#include <shellapi.h>
#include <winnt.h>

#pragma comment(lib, "wtsapi32")
#pragma comment(lib, "Advapi32")

#define MAX_ACCOUNTNAME_LEN 1024
#define MAX_DOMAINNAME_LEN 1024
#define	MAX_FILE_PATH_LEN	5000
#define MAX_PRIV_LEN	5000

void PrintError(LPCTSTR lpszFunction, bool exitProcess)
{
	// Retrieve the system error message for the last-error code

	LPVOID lpMsgBuf;
	LPVOID lpDisplayBuf;
	DWORD dw = GetLastError();

	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM |
		FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL,
		dw,
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf,
		0, NULL);

	// Display the error message and exit the process

	lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT,
		(lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR));
	StringCchPrintf((LPTSTR)lpDisplayBuf,
		LocalSize(lpDisplayBuf) / sizeof(TCHAR),
		TEXT("%s failed with error %d: %s"),
		lpszFunction, dw, lpMsgBuf);
	//MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK);
	_tprintf(TEXT("Erro: %s\n"), (LPTSTR)lpDisplayBuf);

	LocalFree(lpMsgBuf);
	LocalFree(lpDisplayBuf);
	if (exitProcess) ExitProcess(dw);
}

// coleta o tamanho da struct
DWORD GetTokenInfoLen(HANDLE processToken, TOKEN_INFORMATION_CLASS info) {

	DWORD structSize;

	GetTokenInformation(processToken, info, NULL, 0, &structSize);

	return structSize;
}

void SIDType(SID_NAME_USE type) {

	switch (type) {

	case SidTypeUser:

		_tprintf(_T("SID Type: SidTypeUser: SID de usuario.\n"));
		break;

	case SidTypeGroup:
		_tprintf(_T("SID Type: SidTypeGroup: SID de grupo\n"));
		break;

	case SidTypeDomain:
		_tprintf(_T("SID Type: SidTypeDomain: SID de dominio\n"));
		break;

	case SidTypeAlias:
		_tprintf(_T("SID Type: SidTypeAlias: SID alias\n"));
		break;

	case SidTypeWellKnownGroup:
		_tprintf(_T("SID Type: SidTypeWellKnownGroup: SID de grupo well-known\n"));
		break;

	case SidTypeDeletedAccount:
		_tprintf(_T("SID Type: SidTypeDeletedAccount: SID de conta deletada\n"));
		break;

	case SidTypeInvalid:
		_tprintf(_T("SID Type: SidTypeInvalid: SID invalido\n"));
		break;

	case SidTypeUnknown:
		_tprintf(_T("SID Type: SidTypeUnknown: SID nao conhecido\n"));
		break;

	case SidTypeComputer:
		_tprintf(_T("SID Type: SidTypeComputer: SID de computador\n"));
		break;

	case SidTypeLabel:
		_tprintf(_T("SID Type: SidTypeLabel: SID mandatory integrity label\n"));
		break;
	}
}

void PrintSidInfo(PSID psid) {

	LPTSTR stringSID = NULL;

	if (!ConvertSidToStringSid(psid, &stringSID)) {
		PrintError(_T("ConvertSidToStringSid()"), false);
	}

	TCHAR accountName[MAX_ACCOUNTNAME_LEN];
	DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
	TCHAR domainName[MAX_DOMAINNAME_LEN];
	DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
	SID_NAME_USE peUse;

	if (!LookupAccountSid(NULL, psid, accountName, &accountNameBuffer, domainName, &domainNameBuffer, &peUse)) {
		PrintError(_T("LookupAccountSid()"), false);
		return;
	}

	_tprintf(_T("SID: %s\n"), stringSID);
	SIDType(peUse);
	_tprintf(_T("Conta: %s\\%s\n\n"), domainName, accountName);
}

void GetTokenOwner(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenOwner);
	DWORD structLen;

	PTOKEN_OWNER tokenInfo = (PTOKEN_OWNER)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenOwner, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN OWNER: \n"));
	PrintSidInfo(tokenInfo->Owner);

	free(tokenInfo);


}

void GetTokenUser(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenUser);
	DWORD structLen;

	PTOKEN_USER tokenInfo = (PTOKEN_USER)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenUser, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN USER: \n"));
	PrintSidInfo(tokenInfo->User.Sid);

	free(tokenInfo);


}

void GetTokenPrimaryGroup(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenPrimaryGroup);
	DWORD structLen;

	PTOKEN_PRIMARY_GROUP tokenInfo = (PTOKEN_PRIMARY_GROUP)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenPrimaryGroup, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN PRIMARY GROUP: \n"));
	PrintSidInfo(tokenInfo->PrimaryGroup);

	free(tokenInfo);


}

void GetTokenGroups(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenGroups);
	DWORD structLen;

	PTOKEN_GROUPS tokenInfo = (PTOKEN_GROUPS)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenGroups, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN GROUPS: \n"));
	for (DWORD i = 0; i < tokenInfo->GroupCount; i++) {

		PrintSidInfo(tokenInfo->Groups[i].Sid);
	}
	

	free(tokenInfo);


}

void GetTokenPrivileges(HANDLE processToken) {

	DWORD infoLen = GetTokenInfoLen(processToken, TokenPrivileges);
	DWORD structLen;

	PTOKEN_PRIVILEGES tokenInfo = (PTOKEN_PRIVILEGES)malloc(infoLen);

	if (!GetTokenInformation(processToken, TokenPrivileges, (LPVOID)tokenInfo, infoLen, &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	TCHAR privName[MAX_PRIV_LEN];
	DWORD privNameSize = sizeof(privName);

	_tprintf(_T("************************ TOKEN PRIVILEGES: \n"));
	for (DWORD i = 0; i < tokenInfo->PrivilegeCount; i++) {

		privNameSize = sizeof(privName);

		if (!LookupPrivilegeName(NULL, &tokenInfo->Privileges[i].Luid, privName, &privNameSize)) {
			PrintError(_T("LookupPrivilegeName()"), false);
		}
		else {
			if (tokenInfo->Privileges[i].Attributes & SE_PRIVILEGE_ENABLED) {
				_tprintf(_T("%3d. Enabled        %s\n"), i + 1, privName);
			}
			else {
				_tprintf(_T("%3d. Disabled       %s\n"), i + 1, privName);
			}
		}
	}
	_tprintf(_T("\n"));

	free(tokenInfo);


}

void GetTokenSource(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_SOURCE tokenInfo = (PTOKEN_SOURCE)malloc(sizeof(TOKEN_SOURCE));

	if (!GetTokenInformation(processToken, TokenSource, (LPVOID)tokenInfo, sizeof(TOKEN_SOURCE), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN SOURCE: \n"));
	for (DWORD i = 0; i < 8; i++) {

		if (tokenInfo->SourceName[i]) {
			_tprintf(_T("%c"), tokenInfo->SourceName[i]);
		}
	}
	_tprintf(_T("\n\n"));

	free(tokenInfo);


}

void GetTokenType(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_TYPE tokenInfo = (PTOKEN_TYPE)malloc(sizeof(PTOKEN_TYPE));

	if (!GetTokenInformation(processToken, TokenType, (LPVOID)tokenInfo, sizeof(PTOKEN_TYPE), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN TYPE: \n"));

	if (*tokenInfo == TokenPrimary) {

		_tprintf(_T("TokenPrimary"));
	}
	else {
		_tprintf(_T("TokenImpersonation"));

		// procurando o tipo de impersonation
		PSECURITY_IMPERSONATION_LEVEL tokenImpLevel = (PSECURITY_IMPERSONATION_LEVEL)malloc(sizeof(SECURITY_IMPERSONATION_LEVEL));
		if (!GetTokenInformation(processToken, TokenImpersonationLevel, (LPVOID)tokenImpLevel, sizeof(TOKEN_TYPE), &structLen)) {

			PrintError(_T("GetTokenInformation()"), false);
		}
		else {
			switch (*tokenImpLevel) {

			case SecurityAnonymous:
				_tprintf(_T("Impersonation Level: SecurityAnonymous\n"));
					break;

			case SecurityIdentification:
				_tprintf(_T("Impersonation Level: SecurityIdentification\n"));
				break;

			case SecurityImpersonation:
				_tprintf(_T("Impersonation Level: SecurityImpersonation\n"));
				break;

			case SecurityDelegation:
				_tprintf(_T("Impersonation Level: SecurityDelegation\n"));
				break;
			}
		}
	}
	_tprintf(_T("\n\n"));

	free(tokenInfo);


}

void GetTokenElevation(HANDLE processToken) {

	DWORD structLen;

	PTOKEN_ELEVATION tokenInfo = (PTOKEN_ELEVATION)malloc(sizeof(TOKEN_ELEVATION));

	if (!GetTokenInformation(processToken, TokenElevation, (LPVOID)tokenInfo, sizeof(TOKEN_ELEVATION), &structLen)) {
		PrintError(_T("GetTokenInformation()"), false);
		return;
	}

	_tprintf(_T("************************ TOKEN ELEVATION: \n"));
	if (tokenInfo->TokenIsElevated) {

		_tprintf(_T("Token Elevated: TRUE\n"));
	}
	else {
		_tprintf(_T("Token Elevated: FALSE\n"));
	}
	_tprintf(_T("\n"));

	free(tokenInfo);


}

void Relaunch(void) {

	SHELLEXECUTEINFO execInfo;
	WCHAR filePath[MAX_FILE_PATH_LEN];
	DWORD filePathLen = MAX_FILE_PATH_LEN;

	// coletando o path do programa
	GetModuleFileName(NULL, filePath, filePathLen);

	// preenchendo a estrutura SHELLEXECUTEINFO
	execInfo.cbSize = sizeof(SHELLEXECUTEINFO);
	execInfo.fMask = SEE_MASK_DEFAULT;
	execInfo.lpVerb = _T("runas");
	execInfo.lpFile = filePath;
	execInfo.lpParameters = NULL;
	execInfo.lpDirectory = NULL;
	execInfo.nShow = SW_SHOWNORMAL;

	ShellExecuteEx(&execInfo);
}

// função para habilitar SeDebugPrivilege
BOOL EnableSeDebug(void) {

	LUID privilegeLuid;
	if (!LookupPrivilegeValue(NULL, _T("SeDebugPrivilege"), &privilegeLuid)) {
		PrintError(_T("LookupPrivilegeValue()"), true);
	}

	// criando a estrutura do token
	TOKEN_PRIVILEGES tkPrivs;

	tkPrivs.PrivilegeCount = 1; // quantidade de privilégios a serem configurados
	tkPrivs.Privileges[0].Luid = privilegeLuid; // LUID do primeiro e único privilégio (SeDebugPrivilege)
	tkPrivs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED; // atributo que o privilégio receberá

	HANDLE currentProcessHandle = GetCurrentProcess();
	HANDLE processToken;

	if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &processToken)) {
		PrintError(_T("OpenProcessToken()"), true);
	}

	// coletando o tamanho da estrutura
	DWORD firstStructSize = GetTokenInfoLen(processToken, TokenPrivileges);

	// invocando GetTokenInformation() com todos os parâmetros
	DWORD secondStructSize; // variável que receberá o tamanho da resposta na segunda execução
	PTOKEN_PRIVILEGES processTokenPriv; // ponteiro para estrutura TOKEN_PRIVILEGES que receberá a resposta

	processTokenPriv = (PTOKEN_PRIVILEGES)malloc(firstStructSize); // alocando o buffer

	if (!GetTokenInformation(processToken, TokenPrivileges, processTokenPriv, firstStructSize, &secondStructSize)) {
		PrintError(_T("GetTokenInformation()"), true);
	}

	// iterando sobre a estrutura TOKEN_PRIVILEGES
	PLUID_AND_ATTRIBUTES luidStruct; // ponteiro para estrutura que receberá cada membro do array na iteração
	bool seDebug = false; // variável de controle caso o privilégio exista

	for (DWORD i = 0; i < processTokenPriv->PrivilegeCount; i++) {

		luidStruct = &processTokenPriv->Privileges[i];

		if ((luidStruct->Luid.LowPart == privilegeLuid.LowPart) && (luidStruct->Luid.HighPart == privilegeLuid.HighPart)) {
			_tprintf(_T("[+] SeDebugPrivilege encontrada para habilitar!\n\n"));
			seDebug = true;
			break;
		}
	}

	// caso não encontre o SeDebugPrivilege
	if (!seDebug) {
		_tprintf(_T("[-] SeDebugPrivilege nao encontrado\nExecutando com privilegios administrativos!\n\n"));
		return FALSE;
	}

	if (!AdjustTokenPrivileges(processToken, false, &tkPrivs, 0, NULL, NULL)) {
		PrintError(_T("AdjustTokenPrivileges()"), true);
	}



	return TRUE;
}

int main(void) {

	if (!EnableSeDebug()) {
		Relaunch();
		ExitProcess(-1);
	}
	else {
		_tprintf(_T("Adicionando privilegio de debug!\n\n"));
	}

	DWORD level = 1; // variável que indica o tipo da estrutura PWTS_PROCESS_INFO_EX
	PWTS_PROCESS_INFO_EX processes = NULL; // variável que contém uma estrutura PWTS_PROCESS_INFO_EX vazia
	DWORD processCounter = 0; // variável que conterá a quantidade de estruturas enumeradas

	// enumerando os processos
	if (!WTSEnumerateProcessesEx(
		WTS_CURRENT_SERVER_HANDLE,
		&level,
		WTS_ANY_SESSION,
		(LPTSTR*)&processes,
		&processCounter)) {
		return 0;
	}

	// imprimindo dados das estruturas
	_tprintf(_T("Processos encontrados: %d\n\n"), processCounter);
	LPTSTR stringSID = NULL; // ponteiro do tipo string que receberá o SID do usuário que executa o processo
	PWTS_PROCESS_INFO_EX processesPtr = processes;

	_tprintf(_T("#\tPID\tHandles\tThreads\tProcess Name\t\tAccount\n\n"));

	// iterando sobre as estruturas
	for (DWORD counter = 1; counter <= processCounter; counter++) {

		_tprintf(_T("%d\t"), counter);
		_tprintf(_T("%d\t"), processes->ProcessId);
		_tprintf(_T("%d\t"), processes->HandleCount);
		_tprintf(_T("%d\t"), processes->NumberOfThreads);
		_tprintf(_T("%s\t\t"), processes->pProcessName);

		TCHAR accountName[MAX_ACCOUNTNAME_LEN];
		DWORD accountNameBuffer = MAX_ACCOUNTNAME_LEN;
		TCHAR domainName[MAX_DOMAINNAME_LEN];
		DWORD domainNameBuffer = MAX_DOMAINNAME_LEN;
		SID_NAME_USE peUse;

		// procura dados da conta através do SID
		if (!LookupAccountSid(
			NULL,
			processes->pUserSid,
			accountName,
			&accountNameBuffer,
			domainName,
			&domainNameBuffer,
			&peUse)) {
			_tprintf(_T("\n"));
		}
		else {
			_tprintf(_T("%s\\%s\n"), domainName, accountName);
		}

		processes++;
	}

	// liberando a memória da estrutura após o uso
	if (!WTSFreeMemoryEx(WTSTypeProcessInfoLevel1, processesPtr, processCounter)) {
		return 0;
	}
	processes = NULL;

	// ********************************** inicio do dump do token
	int pid;

	_tprintf(_T("\n\nFazer o dump do token para qual PID?: "));
	scanf_s("%d[^\n]", &pid);
	_tprintf(_T("\n[+] Fazendo dump do token para o PID: %d\n"), pid);

	// criando um handle para o processo do qual extrairemos o token
	HANDLE processHandle;
	if (NULL == (processHandle = OpenProcess(MAXIMUM_ALLOWED, false, pid))) {
		_tprintf(_T("[-] Falha ao usar OpenProcess() com MAXIMUM_ALLOWED, este processo deve estar protegido!\n"));
		_tprintf(_T("[-] Tentando novamente com com PROCESS_QUERY_LIMITED_INFORMATION!\n"));

		if (NULL == (processHandle = OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION, false, pid))) {
			PrintError(_T("OpenProcess()"), false);
		}
	}


	// criando um handle para o token
	HANDLE processToken;

	if (!OpenProcessToken(processHandle, MAXIMUM_ALLOWED, &processToken)) {
		PrintError(_T("OpenProcessToken()"), false);
	}

	// parsing das informações do token
	_tprintf(_T("[+] Parsing das informacoes do token:\n"));

	_tprintf(_T("\n*********************INFORMACOES DO TOKEN**********************\n\n"));
	GetTokenOwner(processToken);
	GetTokenUser(processToken);
	GetTokenPrimaryGroup(processToken);
	GetTokenGroups(processToken);
	GetTokenPrivileges(processToken);
	GetTokenSource(processToken);
	GetTokenType(processToken);
	GetTokenElevation(processToken);

	CloseHandle(processHandle);
	CloseHandle(processToken);

	_tprintf(_T("\n\nPressione qualquer tecla para finalizar()\n"));

	getchar();
	getchar();
	return 0;
}
```

Agora quando enumeramos o PID 92, conseguimos obter as informações.

![](/img/posts/Pasted%20image%2020240815135623.png)

O passo-a-passo na criação deste programa, fornece uma boa base para o *process listing* e o *token dumping* utilizando as APIs do Windows. Obviamente o desafio é utilizar a criatividade para explorar cada vez mais possibilidades.

# CreateToolhelp32Snapshot

O uso da [CreateToolhelp32Snapshot](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot) para *process listing* foi exemplificado no artigo anterior "[Enumerando Processos pelo Nome](https://h41stur.com/posts/get-process/)" onde a utilizamos para encontrar processos em execução pesquisando pelo nome do executável.

O seu uso é muito simples, uma vez que só exige dois parâmetros: o tipo do *snapshot* e o PID de um processo específico que pode ser setado como zero, para listar todos os processos.

```cpp
HANDLE CreateToolhelp32Snapshot(
  [in] DWORD dwFlags,
  [in] DWORD th32ProcessID
);
```

O tipo de *snapshot* pode ser desde os processos em execução, ou *threads* e até mesmo um *snapshot* completo, seguindo as opções da tabela abaixo:

![](/img/posts/Pasted%20image%2020240815164903.png)

## Vantagens do uso

Utilizar esta API para listar processos tem como sua maior vantagem que seu uso não é considerado malicioso por sistemas antivírus.

Isso ocorre por uma série de fatores, tais como: não é preciso ter o privilégio `SeDebugPrivilege` para obter informações sobre processos; ele não consulta diretamente a memória, mas cria um *snapshot* para ser analisado, sem interferir em nenhum outro processo em execução; é utilizado por uma infinidade de sistemas de monitoramento, portanto seu uso é considerado "normal"; entre outros.

Inclusive, por esse motivo, é uma técnica utilizada por alguns *malwares* famosos como:

[Zeus](https://ioactive.com/pdfs/ZeusSpyEyeBankingTrojanAnalysis.pdf)

![](/img/posts/Pasted%20image%2020240815170034.png)

E o [PoS RAM Scraper](https://documents.trendmicro.com/assets/wp/wp-pos-ram-scraper-malware.pdf)

![](/img/posts/Pasted%20image%2020240815170130.png)

## Desvantagens do uso

A maior, e nem tão impactante, desvantagem de usar a `CreateToolhelp32Snapshot` também está no fato de trabalhar com *snapshots*, ou seja, é um "retrato" de um momento no sistema, durante a análise deste momento, novos processos podem ser inicializados, finalizados, modificados, novas *threads* podem ser executadas alterando o contexto do sistema em questão.

Do ponto de vista <i><span style="color:red;">red team</span></i> é uma desvantagem aceitável, comparado com as vantagens.

## Desenvolvendo o *Process Listing*

Para listarmos os processos, podemos criar um *snapshot* semente destes, passando como tipo, o valor `TH32CS_SNAPPROCESS`. A função `CreateToolhelp32Snapshot` enviará o *snapshot* para um *handle*.

A partir deste ponto, podemos trabalhar com a lista de processos gerada no *snapshot* com o uso da função `Process32First`.

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

Após sua execução, a estrutura `PROCESSENTRY32` passa a conter os dados do próximo processo do *snapshot*. A partir deste ponto, podemos percorrer toda a lista de processos contidos no *snapshot* obtendo as informações necessárias para nossos objetivos.

É possível notar que a estrutura `PROCESSENTRY32` por si só, já contém informações relevantes.

Um simples programa de *process listing* pode ser criado da seguinte maneira:

```cpp
#include <Windows.h>
#include <tchar.h>
#include <TlHelp32.h>
#include <strsafe.h>


void PrintError(LPCTSTR lpszFunction, bool exitProcess)
{
	// Retrieve the system error message for the last-error code

	LPVOID lpMsgBuf;
	LPVOID lpDisplayBuf;
	DWORD dw = GetLastError();

	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM |
		FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL,
		dw,
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf,
		0, NULL);

	// Display the error message and exit the process

	lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT,
		(lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR));
	StringCchPrintf((LPTSTR)lpDisplayBuf,
		LocalSize(lpDisplayBuf) / sizeof(TCHAR),
		TEXT("%s failed with error %d: %s"),
		lpszFunction, dw, lpMsgBuf);
	//MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK);
	_tprintf(TEXT("Erro: %s\n"), (LPTSTR)lpDisplayBuf);

	LocalFree(lpMsgBuf);
	LocalFree(lpDisplayBuf);
	if (exitProcess) ExitProcess(dw);
}

// função para parsing
void ParseSnapshot(LPPROCESSENTRY32 process) {

	_tprintf(_T("PID: %d\tThreads: %d\tExe File: %s\n"), process->th32ProcessID, process->cntThreads, process->szExeFile);

}

int main(void) {

	// handle para o snapshot
	HANDLE processes;

	// extraindo o snapshot
	processes = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

	if (INVALID_HANDLE_VALUE == processes) {
		PrintError(_T("CreateToolhelp32Snapshot"), true);
	}

	// estrutura PROCESSENTRY32
	PROCESSENTRY32 processEntry;
	// inicializando a estrutura PROCESSENTRY32
	processEntry.dwSize = sizeof(PROCESSENTRY32);

	// coletando o primeiro processo da lista
	if (!Process32First(processes, &processEntry)) {
		PrintError(_T("Process32First"), true);
	}

	// iterando a lista
	do {
		// enviando para o parsing
		ParseSnapshot(&processEntry);

	} while (Process32Next(processes, &processEntry));

	CloseHandle(processes);

	getchar();
	return 0;
}
```

Ao executarmos, temos a listagem de processos:

![](/img/posts/Pasted%20image%2020240815184407.png)

Obviamente este é um exemplo bem simples, porém as possibilidades documentadas de informações passíveis de extração, nos dá um leque muito grande de informações que podem ser coletadas.

# EnumProcesses

A [EnumProcesses](https://learn.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-enumprocesses) é uma função da [PSAPI](https://learn.microsoft.com/en-us/windows/win32/api/_psapi/) (que por sinal, é cheia de excelentes funções) que visa trazer um *array* com **todos** os PIDs de processos em execução no sistema. Sua sintaxe é bem simples:

```cpp
BOOL EnumProcesses(
  [out] DWORD   *lpidProcess,
  [in]  DWORD   cb,
  [out] LPDWORD lpcbNeeded
);
```

Onde: 

- `[out] lpidProcess` é um ponteiro para uma *array* que receberá os PIDs;
- `[in] cb` é o tamanho da *array* em *bytes*;
- `[out] lpcbNeeded` o ponteiro para a variável que receberá a quantidade em *bytes* da resposta obtida.

Tecnicamente é uma função simples, com uma resposta simples, porém, como já vimos durante este artigo, saber o PID de um processo é essencial no *process listing*, pois, uma vez que temos este valor, podemos utilizar várias outras funções de API para trazer informações relevantes.

Por exemplo, podemos extrair a *array* com todos os PIDs, e, durante a iteração com a lista, podemos usar a função `OpenProcess` para criar um *handle* para cada processo, e a partir daí usar a função [GetProcessImageFileName](https://learn.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-getprocessimagefilenamea) para obter o *path* de cada processo.

Porém, assim como a API `WTSEnumerateProcessesEx`, ela exige o privilégio `SeDebugPrivilege` para enumerar processos de privilégio elevado.

Uma simples implementação seria:

```cpp
#include <Windows.h>
#include <stdio.h>
#include <strsafe.h>
#include <Psapi.h>
#include <tchar.h>

#define MAX_PIDS			4000
#define MAX_FILENAME_LEN	2000

void PrintError(LPCTSTR lpszFunction, bool exitProcess)
{
	// Retrieve the system error message for the last-error code

	LPVOID lpMsgBuf;
	LPVOID lpDisplayBuf;
	DWORD dw = GetLastError();

	FormatMessage(
		FORMAT_MESSAGE_ALLOCATE_BUFFER |
		FORMAT_MESSAGE_FROM_SYSTEM |
		FORMAT_MESSAGE_IGNORE_INSERTS,
		NULL,
		dw,
		MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT),
		(LPTSTR)&lpMsgBuf,
		0, NULL);

	// Display the error message and exit the process

	lpDisplayBuf = (LPVOID)LocalAlloc(LMEM_ZEROINIT,
		(lstrlen((LPCTSTR)lpMsgBuf) + lstrlen((LPCTSTR)lpszFunction) + 40) * sizeof(TCHAR));
	StringCchPrintf((LPTSTR)lpDisplayBuf,
		LocalSize(lpDisplayBuf) / sizeof(TCHAR),
		TEXT("%s failed with error %d: %s"),
		lpszFunction, dw, lpMsgBuf);
	//MessageBox(NULL, (LPCTSTR)lpDisplayBuf, TEXT("Error"), MB_OK);
	_tprintf(TEXT("Erro: %s\n"), (LPTSTR)lpDisplayBuf);

	LocalFree(lpMsgBuf);
	LocalFree(lpDisplayBuf);
	if (exitProcess) ExitProcess(dw);
}

// função para habilitar SeDebugPrivilege
BOOL EnableSeDebug(void) {

	LUID privilegeLuid;
	if (!LookupPrivilegeValue(NULL, _T("SeDebugPrivilege"), &privilegeLuid)) {
		PrintError(_T("LookupPrivilegeValue()"), true);
	}

	// criando a estrutura do token
	TOKEN_PRIVILEGES tkPrivs;

	tkPrivs.PrivilegeCount = 1; // quantidade de privilégios a serem configurados
	tkPrivs.Privileges[0].Luid = privilegeLuid; // LUID do primeiro e único privilégio (SeDebugPrivilege)
	tkPrivs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED; // atributo que o privilégio receberá

	HANDLE currentProcessHandle = GetCurrentProcess();
	HANDLE processToken;

	if (!OpenProcessToken(currentProcessHandle, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &processToken)) {
		PrintError(_T("OpenProcessToken()"), true);
	}

	// coletando o tamanho da estrutura
	DWORD firstStructSize;
	GetTokenInformation(processToken, TokenPrivileges, NULL, 0, &firstStructSize);

	// invocando GetTokenInformation() com todos os parâmetros
	DWORD secondStructSize; // variável que receberá o tamanho da resposta na segunda execução
	PTOKEN_PRIVILEGES processTokenPriv; // ponteiro para estrutura TOKEN_PRIVILEGES que receberá a resposta

	processTokenPriv = (PTOKEN_PRIVILEGES)malloc(firstStructSize); // alocando o buffer

	if (!GetTokenInformation(processToken, TokenPrivileges, processTokenPriv, firstStructSize, &secondStructSize)) {
		PrintError(_T("GetTokenInformation()"), true);
	}

	// iterando sobre a estrutura TOKEN_PRIVILEGES
	PLUID_AND_ATTRIBUTES luidStruct; // ponteiro para estrutura que receberá cada membro do array na iteração
	bool seDebug = false; // variável de controle caso o privilégio exista

	for (DWORD i = 0; i < processTokenPriv->PrivilegeCount; i++) {

		luidStruct = &processTokenPriv->Privileges[i];

		if ((luidStruct->Luid.LowPart == privilegeLuid.LowPart) && (luidStruct->Luid.HighPart == privilegeLuid.HighPart)) {
			_tprintf(_T("[+] SeDebugPrivilege encontrada para habilitar!\n\n"));
			seDebug = true;
			break;
		}
	}

	// caso não encontre o SeDebugPrivilege
	if (!seDebug) {
		_tprintf(_T("[-] SeDebugPrivilege nao encontrado\nExecute com privilegios administrativos para enumerar todos os processos!\n\n"));
		return FALSE;
	}

	if (!AdjustTokenPrivileges(processToken, false, &tkPrivs, 0, NULL, NULL)) {
		PrintError(_T("AdjustTokenPrivileges()"), true);
	}



	return TRUE;
}

int main(void) {

	DWORD	pids[MAX_PIDS];
	DWORD	cbNeeded;
	DWORD	numberProc;
	HANDLE	processHandle;
	TCHAR	fileName[MAX_FILENAME_LEN];

	// habilitando SeDebugPrivileges
	if (!EnableSeDebug()) {
		_tprintf(_T("[-] Nao foi possivel habilitar privilegio de debug!\n[-] Alguns processos nao serao enumerados!\n\n"));
	} else {
		_tprintf(_T("[+] Adicionando privilegio de debug!\n\n"));
	}

	// Obtendo os PIDs
	if (EnumProcesses(pids, sizeof(pids), &cbNeeded) == 0) {
		PrintError(_T("EnumProcesses()"), true);
	}

	numberProc = cbNeeded / sizeof(DWORD);

	_tprintf(_T("#\tPID\tProcess Image\n\n"));

	for (DWORD counter = 0; counter < numberProc; counter++) {

		// pulando PID 0
		if (pids[counter] == 0) continue;

		_tprintf(_T("%d\t%d\t"), counter + 1, pids[counter]);

		// obtendo o handle do processo
		processHandle = OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION, FALSE, pids[counter]);

		if (processHandle == NULL) {
			PrintError(_T("OpenProcess()"), false);
		}
		else {

			// obtendo o nome do processo
			if (GetProcessImageFileName(processHandle, fileName, MAX_FILENAME_LEN) == 0) {

				PrintError(_T("GetProcessImageFileName()"), false);
				_tprintf(_T("-\n"));
			}
			else {

				// pulando PID 0
				if (pids[counter] != 0) {

					_tprintf(_T("%s\n"), fileName);
				}
			}
		}
	}

	getchar();
	return 0;
}
```

Quando executamos o programa como usuário comum, vemos que não foi possível habilitar o `SeDebugPrivilege` e houve erro na enumeração de processos elevados.

![](/img/posts/Pasted%20image%2020240815211153.png)

Já quando executamos como administrador, o `SeDebugPrivilege` é habilitado e quase todos os processos retornam o *Process Image*.

![](/img/posts/Pasted%20image%2020240815211358.png)

O fato do erro ter acontecido ao tentar enumerar o *Process Image* neste processo, não é necessariamente um erro. Se analisarmos o **PID 4** com o *Process Explorer* veremos que o processo é o *System*.

![](/img/posts/Pasted%20image%2020240815211632.png)

Quando expandimos as propriedades deste processo, vemos que ele não tem um *path*.

![](/img/posts/Pasted%20image%2020240815211721.png)

Logo, o erro retornado, não é um erro de código de fato, ela não conseguiu trazer o *path*, pois ele realmente não existe. nada que não possa ser tratado para melhorar a resposta do programa.

# Conclusão

Neste artigo, exploramos diversas técnicas para listagem de processos e _token dumping_ utilizando APIs do Windows, destacando a flexibilidade e o poder dessas abordagens no contexto de exploração e análise de sistemas. Através do uso de funções como `WTSEnumerateProcessesEx` e outros métodos abordados, vimos como é possível obter informações detalhadas sobre os processos em execução em um ambiente Windows, identificando não apenas o contexto em que eles operam, mas também as permissões associadas e os tokens de segurança em uso.

Essas técnicas, quando aplicadas de forma estratégica, podem ser combinadas para criar ferramentas mais robustas e eficientes, capazes de superar restrições impostas pelo ambiente alvo. A compreensão detalhada do funcionamento dessas APIs não só amplia o leque de possibilidades durante um exercício de pentesting ou análise forense, mas também destaca a importância de uma abordagem criativa e adaptável ao enfrentarmos diferentes cenários.

Além disso, ao utilizar diferentes abordagens para realizar tarefas aparentemente simples, como a listagem de processos, garantimos uma maior resiliência das ferramentas desenvolvidas, uma vez que a diversidade de métodos torna mais difícil para mecanismos de defesa automatizados preverem ou bloquearem tais ações. Esta capacidade de adaptação é crucial em um cenário de segurança cibernética onde as técnicas e contramedidas estão em constante evolução.

O domínio das APIs discutidas não apenas facilita a execução de tarefas essenciais para reconhecimento e exploração, mas também abre caminho para a criação de soluções customizadas e inovadoras que podem fazer a diferença em situações críticas. A exploração contínua dessas e outras funções oferecidas pelo Windows é essencial para qualquer profissional que busca aprofundar seus conhecimentos e habilidades em cibersegurança.


# Referências

- [https://learn.microsoft.com/en-us/windows/win32/toolhelp/tool-help-library](https://learn.microsoft.com/en-us/windows/win32/toolhelp/tool-help-library)
- [https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsenumerateprocessesexa](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsenumerateprocessesexa)
- [https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsopenservera](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsopenservera)
- [https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/ns-wtsapi32-wts_process_infoa](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/ns-wtsapi32-wts_process_infoa)
- [https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/ns-wtsapi32-wts_process_info_exa](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/ns-wtsapi32-wts_process_info_exa)
- [https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsfreememoryexa](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsfreememoryexa)
- [https://learn.microsoft.com/en-us/sysinternals/](https://learn.microsoft.com/en-us/sysinternals/)
- [https://learn.microsoft.com/en-us/windows/win32/api/sddl/nf-sddl-convertsidtostringsida](https://learn.microsoft.com/en-us/windows/win32/api/sddl/nf-sddl-convertsidtostringsida)
- [https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-localfree](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-localfree)
- [https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-lookupaccountsida](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-lookupaccountsida)
- [https://learn.microsoft.com/en-us/windows/win32/api/winnt/ne-winnt-sid_name_use](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ne-winnt-sid_name_use)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken)
- [https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation)
- [https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-settokeninformation](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-settokeninformation)
- [https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges)
- [https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-duplicatetokenex](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-duplicatetokenex)
- [https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-impersonateloggedonuser](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-impersonateloggedonuser)
- [https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-lookupprivilegevaluea](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-lookupprivilegevaluea)
- [https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-token_privileges](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-token_privileges)
- [https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-luid_and_attributes](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-luid_and_attributes)
- [https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-privilege_set](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-privilege_set)
- [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken)
- [https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-adjusttokenprivileges)
- [https://learn.microsoft.com/en-us/windows/win32/api/errhandlingapi/nf-errhandlingapi-getlasterror](https://learn.microsoft.com/en-us/windows/win32/api/errhandlingapi/nf-errhandlingapi-getlasterror)
- [https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation)
- [https://learn.microsoft.com/en-us/windows/win32/api/winnt/ne-winnt-token_information_class](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ne-winnt-token_information_class)
- [https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexa](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexa)
- [https://learn.microsoft.com/en-us/windows/win32/api/shellapi/ns-shellapi-shellexecuteinfoa](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/ns-shellapi-shellexecuteinfoa)
- [https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulefilenamea](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulefilenamea)
- [https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights](https://learn.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights)
- [https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot)
- [https://ioactive.com/pdfs/ZeusSpyEyeBankingTrojanAnalysis.pdf](https://ioactive.com/pdfs/ZeusSpyEyeBankingTrojanAnalysis.pdf)
- [https://documents.trendmicro.com/assets/wp/wp-pos-ram-scraper-malware.pdf](https://documents.trendmicro.com/assets/wp/wp-pos-ram-scraper-malware.pdf)
- [https://learn.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-enumprocesses](https://learn.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-enumprocesses)
- [https://learn.microsoft.com/en-us/windows/win32/api/_psapi/](https://learn.microsoft.com/en-us/windows/win32/api/_psapi/)
- [https://learn.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-getprocessimagefilenamea](https://learn.microsoft.com/en-us/windows/win32/api/psapi/nf-psapi-getprocessimagefilenamea)