---
title: Flutter SSL Pinning Bypass
author: H41stur
date: 2024-01-16 01:00:00 -0300
categories: [Estudos, Mobile]
tags: [Mobile, Android, iOS, Flutter, SSL Pinning]
image: "/img/posts/flutter.png"
alt: "Bypass Flutter SSL Pinning"
---

![Bypass Flutter SSL Pinning](/img/posts/flutter.png)

# TL-DR

Os apps criados utilizando o *framework* **Flutter** processam as conexões seguras e obedecem às definições de *proxy* de forma distinta se comparados aos apps programados em DEX. Uma biblioteca denominada `libflutter.so` contém as funcionalidades necessárias e sua própria CA *store* utilizando a biblioteca [Mozilla NSS](https://hg.mozilla.org/mozilla-central/raw-file/tip/security/nss/lib/ckfw/builtins/certdata.txt){:target="\_blank"} para estabelecer conexões remotas.

Por conta disso, os aplicativos Flutter têm características exclusivas que tornam o *bypass* do SSL *pinning* mais desafiador. No entanto, existem várias técnicas disponíveis.

Neste post, abordaremos uma delas, envolvendo a análise da biblioteca `libflutter.so` para criação de um *hooking* personalizado.

# Motivação

Recentemente, durante um trabalho, acabei me deparando com um aplicativo feito em Flutter, não era um aplicativo recente, portanto foi feito com versões não atualizadas do *framework*.

Ao estudar sobre as técnicas de *bypass* de SSL *Pinning*, não houve problemas em encontrar uma forma de fazê-lo. Porém, me despertou a curiosidade de entender mais a fundo o processo.

Ao concluir o projeto, decidi estudar as possibilidades em projetos feitos com a versão mais atual do Flutter. O que me levou a um problema: nenhum artigo que encontrei, se tratava das versões mais atuais (logo, logo este post também se enquadrará nessa categoria), e nenhum aplicativo já disponibilizado para laboratório era de fato atualizado com arquiteturas e versões atuais.

O primeiro passo foi ler sobre o Flutter, sua própria documentação (ótima por sinal) o suficiente para codar um aplicativo simples, que me permitisse testar o *bypass* de SSL *pinning*.

O segundo passo, foi quebrar muito a cabeça para entender como tudo funciona com as versões atuais, tanto do *framework*, quanto da arquitetura do dispositivo Android utilizado (ARMv8).

Acredito que todo o processo foi muito gratificante e deu embasamento mais profundo sobre o funcionamento, não só do Flutter, como dos próprios dispositivos.

# Introdução

O *bypass* do SSL *Pinning* no Flutter não é algo novo, mas é algo que sai dos padrões mais utilizados quando se trata de métodos. Obviamente, por conta do tempo de existência do *framework* já existem formas que automatizam o processo em versões anteriores, porém, **entender** como as coisas funcionam é melhor do que **ver** as coisas funcionando.

Esta tarefa exige um pouco mais de trabalho, e análise, só que, ao meu ponto de vista, é muito mais prazeroso e divertido, além de garantir embasamento sobre alguns assuntos.

Portanto, aproveite o processo.

## Flutter

O Flutter é um *framework* de código aberto criado pelo Google para o desenvolvimento de aplicações móveis. Ele permite a criação de aplicativos nativos de alta qualidade para iOS e Android a partir de uma única base de código. Isso significa que, com o Flutter, você pode desenvolver um aplicativo que funcione tanto em dispositivos iOS quanto Android, sem a necessidade de escrever códigos específicos para cada plataforma.

Uma de suas principais características é o uso do Dart como linguagem de programação, alterando alguns dos comportamentos normalmente encontrados em aplicativos Android — ou seja, os aplicativos Android Flutter não respeitam as configurações de *proxy* do Android nem confiam no **Android TrustManager**.

A imagem abaixo mostra a arquitetura de uma aplicação Flutter que pode ser consultada [aqui](https://docs.flutter.dev/resources/architectural-overview){:target="\_blank"}.

![Arquitetura Flutter](/img/posts/Pasted%20image%2020240117181023.png)

## *Bypass* de SSL *Pinning*

O *bypass* de SSL *pinning* é uma técnica particularmente relevante no domínio da análise de segurança de aplicações móveis. Para compreender completamente o que isso significa, é importante primeiro entender o que é o SSL *pinning*.

**SSL *Pinning*:** É uma prática de segurança que envolve "fixar" um ou mais certificados específicos dentro de um aplicativo. Isso significa que o aplicativo é configurado para rejeitar todas as conexões, exceto aquelas que usam um certificado específico ou emitidos por uma Autoridade Certificadora específica. O objetivo é prevenir ataques *man-in-the-middle* (MITM), onde um invasor intercepta e potencialmente altera as comunicações entre o cliente e o servidor.

***Bypass* de SSL *Pinning*:** Esta técnica é usada para desativar ou contornar o mecanismo de *pinning* de SSL em um aplicativo.

# Ambiente de Laboratório

- Qualquer distribuição Linux, ou Windows, ou Mac (isso é irrelevante)
- Aparelho Android com *root* (usarei um Samsung A04e ARMv8 com Android 13)
- ADB
- BurpSuite (usarei o Pro, mas pode ser o Community)
- APK Flutter (codei um exemplo, deixarei o código-fonte no post)
- [Frida](https://github.com/frida/frida/releases){:target="\_blank"}
- [Ghidra](https://ghidra-sre.org/){:target="\_blank"}
- [APK Tool](https://apktool.org/docs/install){:target="\_blank"}


# *Hooking* de Verificação de Certificado

O "*hooking*" de funções é uma técnica utilizada para modificar ou estender o comportamento de um aplicativo durante a execução. Especificamente, no caso do *bypass* de SSL *pinning*, o *hooking* é usado para interceptar e manipular chamadas a funções específicas responsáveis pela implementação do SSL *pinning*.

**Funções de SSL *Pinning***: Em um aplicativo móvel, o SSL *pinning* é implementado através de funções específicas que verificam se o certificado SSL apresentado pelo servidor é o esperado. Se o certificado não corresponder ao que está 'fixado' no aplicativo, a conexão é rejeitada.

***Hooking* de Funções**: O *hooking* envolve o uso de ferramentas ou bibliotecas para interceptar chamadas a essas funções específicas de SSL *pinning*. Quando uma função é “*hookada*”, o controle é temporariamente transferido para um código personalizado antes de, opcionalmente, retornar ao fluxo normal da aplicação. Essa técnica permite alterar o comportamento da função, modificar seus parâmetros ou o valor retornado, ou mesmo impedir que a função original seja executada.

## Configurando o *Setup*

Como vimos anteriormente, os aplicativos desenvolvidos em Flutter não respeitam as configurações de *proxy* do Android nem confiam no **Android TrustManager**, o que significa que a simples configuração de *proxy* nas conexões não é suficiente. É necessário algum recurso externo como o uso do `iptables` ou algum aplicativo de *proxy*.

Em versões mais antigas do Android, era possível utilizar o aplicativo [ProxyDroid](https://play.google.com/store/apps/details?id=org.proxydroid&hl=en&gl=US&pli=1){:target="\_blank"}, porém, ele não roda no Android 13, neste caso eu encontrei o [Super Proxy](https://play.google.com/store/apps/details?id=com.scheler.superproxy&hl=en&gl=US){:target="\_blank"} que atende muito bem.

Para o tráfego poder passar do dispositivo físico para o BurpSuite, é preciso configurar a interface de rede nas configurações de *proxy*, do Burp para ouvir em todas as interfaces.

![Configuração do BurpSuite](/img/posts/Pasted%20image%2020240118150221.png)

Após isso, o **Super Proxy** pode ser configurado para apontar para a máquina onde o BurpSuite está ouvindo na porta **8080**.

![Configuração Super Proxy](/img/posts/Pasted%20image%2020240118150355.png)

## O Aplicativo

Como este processo foi estudado durante testes de um aplicativo real, me deparei com o problema de que não poderia fazer uma publicação expondo o aplicativo sem permissão (também não queria pedir a permissão).

Ao procurar por aplicativos disponíveis na internet para testar, me deparei com outra situação: só encontramos aplicativos mais antigos, feitos com versões antigas do Flutter.

A solução foi codar um aplicativo por mim mesmo, com a versão mais atual do Flutter (**3.16.8** no momento) para poder exemplificar o processo.

Deixarei abaixo, o código-fonte utilizado no `main.dart`.

```dart
import 'package:flutter/material.dart';  
import 'package:http/http.dart' as http;  
  
void main() {
	runApp(MyApp());  
}  
  
class MyApp extends StatelessWidget {
	@override  
	Widget build(BuildContext context) {  
		return MaterialApp(  
			title: 'HTTP e HTTPS Intercept',  
			home: MyHomePage(),  
		);  
	}  
}  
  
class MyHomePage extends StatefulWidget {  
	@override  
	_MyHomePageState createState() => _MyHomePageState();  
}  
  
class _MyHomePageState extends State<MyHomePage> {  
	String _response = '';  

	Future<void> _makeHttpCall(String url) async {  
		try {  
			final response = await http.get(Uri.parse(url));  
			setState(() {  
				_response = 'Requisição para: ' + url + '\nStatus Code: ${response.statusCode}';  
			});
		} catch (e) {  
			setState(() {  
				_response = 'Erro ao fazer o request: $e';  
				print(e);  
			});  
		}  
	}  

	@override  
	Widget build(BuildContext context) {  
		return Scaffold(  
			appBar: AppBar(  
				title: Text('HTTP e HTTPS Intercept'),  
			),
			body: Center(  
				child: Column(  
					mainAxisAlignment: MainAxisAlignment.center,  
					children: <Widget>[  
						ElevatedButton(  
							onPressed: () => _makeHttpCall('http://www.pentest-standard.org/index.php/Main_Page'), 
							child: Text('HTTP'),  
							style: ElevatedButton.styleFrom(  
								backgroundColor: Colors.blue,  
							),  
						),  
						SizedBox(height: 20),  
						ElevatedButton(  
							onPressed: () => _makeHttpCall('https://h41stur.com'),  
							child: Text(  
								'HTTPS',  
								style: TextStyle(color: Colors.white),  
							),
							style: ElevatedButton.styleFrom(  
								backgroundColor: Colors.red,  
							),  
						),  
						Padding(  
							padding: EdgeInsets.all(20),  
							child: Text(_response),  
						),
					],  
				),  
			),  
		);  
	}  
}
```

Lembrando que o arquivo `pubspec.yml` deve conter a dependência `HTTP`.

```yml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.3
```

E o `AndroidManifest.xml` deve conter a permissão para uso da `internet`.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Ao abrir o aplicativo de testes após sua compilação e instalação no *device*, temos dois botões, um para um *request* **HTTP** que não valida o certificado, e outro para um *request* **HTTPS** que passa pela validação.

Quando ligamos o *Interceptor* do BurpSuite e fazemos o *request* **HTTP** o tráfego é interceptado normalmente.

![Tráfego interceptado](/img/posts/Pasted%20image%2020240119160800.png)


Já quando fazemos o *request* **HTTPS** o tráfego é negado, e não é realizado, uma vez que o certificado do BurpSuite não faz parte da CA *store* do Flutter.

![Erro na requisição](/img/posts/Pasted%20image%2020240119160910.png)


<span style="color: #1cce43">É importante lembrar que como a CA *store* do Flutter está compilada nas bibliotecas do aplicativo, e não confia nos certificados do dispositivo, de nada adianta importar o certificado do BurpSuite no aparelho.</span>

## Encontrando os Gatilhos

Uma vez que o comportamento foi identificado, foi iniciada a parte mais interessante do processo, que é identificar os gatilhos para a validação de certificados.

Neste caso em específico, eu codei o aplicativo para imprimir em tela o erro que causou a interrupção da *request*.

![Erro](/img/posts/Pasted%20image%2020240119161107.png)

Porém, isso também pode ser checado com o uso do `logcat`. Durante a requisição HTTPS, podemos ver o erro `CERTIFICATE_VERIFY_FAILED`.

![CERTIFICATE_VERIFY_FAILED](/img/posts/Pasted%20image%2020240119162953.png)

Uma informação importante sobre o erro está no ponto `handshake.cc:393` que mostra o exato ponto no código em que o erro foi invocado.

Sabendo que o Dart utiliza o `BoringSSL` do **Google** para lidar com tudo relacionado a SSL, e que ambos são `open source`, podemos consultar diretamente no código-fonte. 

Analisando a [handshake.cc](https://github.com/google/boringssl/blob/master/ssl/handshake.cc#L393){:target="\_blank"} na linha **393** (o número da linha pode variar conforme a versão analisada), podemos ver o trecho que invoca o erro.

![Validação de certificado](/img/posts/Pasted%20image%2020240118155813.png)

Este trecho de código, faz parte da função `ssl_verify_peer_cert`, ao analisarmos o trecho de código, veremos que ele está em uma condicional que depende da variável `ret`. Um pouco mais acima no código, especificamente na linha [386](https://github.com/google/boringssl/blob/master/ssl/handshake.cc#L386){:target="\_blank"}, veremos que a variável `ret` retorna o *enum* `ssl_verify_result_t` definido na [ssl.h](https://github.com/google/boringssl/blob/083f72d726097b4abb67982315adc5f7ceb5a69a/include/openssl/ssl.h#L2530){:target="\_blank"} na linha **2530**.

![ssl.h](/img/posts/Pasted%20image%2020240118160317.png)

Este enum, compreende 3 possibilidades: `ssl_verify_ok` (0), `ssl_verify_invalid` (1) e `ssl_verify_retry` (2).

Analisando este cenário, podemos chegar a conclusão de que se o valor da variável `ret` for **1** (`ssl_verify_invalid`), o código pulará para a linha 393 e então a função `OPENSSL_PUT_ERROR` será invocada. Caso o valor de `ret` seja **0** (`ssl_verify_ok`), a execução pulará para o fim, na linha **413** e a função `ssl_verify_peer_cert` retornará o valor **0**.

Isso potencialmente significa que o se valor de retorno da função `ssl_verify_peer_cert` for alterado para `ssl_verify_ok` (0) em tempo de execução, podemos tratar os *requests* sempre como válidos. 

Um desafio nesta parte, foi identificar o exato ponto para fazer o *hooking*, uma vez que o certificado é verificado por mais de um método na função `ssl_verify_peer_cert`. Podemos ver nas linhas [368 e 369](https://github.com/google/boringssl/blob/master/ssl/handshake.cc#L368){:target="\_blank"} que o método `custom_verify_callback` faz a checagem, caso passe por este método, o `session_verify_cert_chain` ainda é invocado na linha [386](https://github.com/google/boringssl/blob/master/ssl/handshake.cc#L386){:target="\_blank"}.

Após algumas horas de análise, percebi que com o uso do Frida, podemos facilmente fazer o *hooking* de uma função inteira e alterar seu valor, neste caso, mesmo que existam vários métodos checando e validando o certificado, como a condição válida da função `ssl_verify_peer_cert` é **0**, podemos fazer o *bypass* de dos métodos que ela invoca e fazê-la retornar `ssl_verify_ok`.

O código abaixo, recebe o endereço de uma função e modifica seu valor de retorno.

```javascript
function hooking_ssl_verify(address)
{
    Interceptor.replace(address, new NativeCallback((pathPtr, flags) => {
        console.log("[+] Validação de certificado desabilitada");
        return 0;
    }, 'int', ['pointer', 'int']));
}
```

O próximo desafio, é encontrar a localização desta função em tempo de execução.

## Criando o *Hooking*

Agora que uma função foi encontrada para ser "*hookada*", podemos analisar a biblioteca `libflutter.so` contida no aplicativo, para encontrá-la mediante engenharia reversa como `Ghidra`.

A primeira coisa a se fazer é descompilar o aplicativo com o `apktool`.

```bash
apktool d h41stur.apk
```

![Descompilando o apk](/img/posts/Pasted%20image%2020240119163418.png)

Após a descompilação, a estrutura de diretórios que compõe o aplicativo será criado, no diretório `lib`, haverão mais 3 diretórios, cada um contendo a `libflutter.so` correspondente a uma arquitetura, neste caso, trabalharemos com a `arm64-v8a` correspondente ao meu *device*.

![libflutter.so](/img/posts/Pasted%20image%2020240122150953.png)

Podemos importar a `libflutter.so` no Ghidra e esperar (um bom tempo) pela análise. O *disassembly* da biblioteca, pode nos dar uma análise comparativa para encontrar a função `ssl_verify_peer_cert` e o seu *offset*.

Existem várias formas de fazer esta busca, nem todas serão efetivas, portanto tive que trabalhar com as informações obtidas. No erro reportado pelo aplicativo, tivemos a dica de que a validação falhou na linha **393** da `handshake.cc`. Quando analisamos esta linha no GitHub, percebemos que foi no método `OPENSSL_PUT_ERROR`.

Uma vez que não podemos fazer buscas por nomes de função no *disassembly*, podemos tentar fazer a busca **escalar**. O primeiro passo é investigar a classe `handshake.cc`, para isso fazemos a busca por *strings* no Ghidra (*Search -> For Strings...*). Na busca, procuramos pela classe `handshake.cc`.

![Busca pela handshake.cc](/img/posts/Pasted%20image%2020240122154110.png)

![Resultado da busca](/img/posts/Pasted%20image%2020240122154150.png)

O resultado da busca, nos deixa claro que, como o método `ssl_verify_peer_cert` está contido em alguma das referências cruzadas, e a função `OPENSSL_PUT_ERROR` está dentro dele, ela deve estar em algum endereço na casa do `0x6e0000`.

Com esta informação, podemos fazer a busca escalar (*Search -> For Scalars...*). Sabendo que a `OPENSSL_PUT_ERROR` é invocada na linha 393 da `handshake.cc` (`0x189` em hexa), buscamos por esta referência.

![Busca escalar](/img/posts/Pasted%20image%2020240122154834.png)

Entre todas as referências encontradas, a função que está selecionada (no caso da minha biblioteca, a função `FUN_006eea4c`), é uma excelente candidata, uma vez que seu endereço está na casa de `0x6e0000` e sua estrutura é bem parecida com a estrutura da função `ssl_verify_peer_cert`.

![Função encontrada](/img/posts/Pasted%20image%2020240122155404.png)

Ao nos direcionarmos até ela, podemos identificar os *bytes* que a compõe.

![Função](/img/posts/Pasted%20image%2020240122160707.png)

Estes bytes compreendem o *pattern* da função, esta sequência específica, identifica a função durante a execução. Desta forma, podemos identificá-la na biblioteca `libflutter.so` durante a execução para fazermos o *hooking* da seguinte forma:

```javascript
var m = Process.findModuleByName("libflutter.so"); // localiza o módulo em tempo de execução
var pattern = "fe 0f 1c f8 f8 5f 01 a9 f6 57 02 a9 f4 4f 03 a9 91 dd 0b 94 68 1a 40 f9 15 e9 40 f9"; // pattern da função ssl_verify_peer_cert
var ranges = m.enumerateRanges('r-x'); // localiza as áreas com permissão de leitura e execução dentro da libflutter.so

// faz um loop em cada área r-x encontrada e busca pelo pattern dentro dela
ranges.forEach(range => {
	Memory.scan(range.base, range.size, pattern, {
		onMatch: function(address, size) {
			console.log("[+] ssl_verify_peer_cert encontrada em: 0x" + (address - m.base).toString(16));
			hooking_ssl_verify(address); // invoca o hooking quando encontra o pattern
			}})
})

// hooking
function hooking_ssl_verify(address)
{
    Interceptor.replace(address, new NativeCallback((pathPtr, flags) => {
        console.log("[+] Validação de certificado desabilitada");
        return 0;
    }, 'int', ['pointer', 'int']));
}

```


Este script, encontrará a `libfluuter.so` em tempo de execução e varrerá a memória do dispositivo para encontrar seu endereço base e seu tamanho. Em seguida, encontrará as áreas com permissão de leitura e execução na memória, e para cada uma delas, procurará o *pattern* da função `ssl_verify_peer_cert`, quando a encontrar, fará o *hooking*, injetando `0x00` em seu valor de retorno, que corresponnde a `ssl_verify_ok`.

Para executar, utilizamos o seguinte comando:

```bash
frida -U -f com.example.h41stur.flutter_interceptor -l flutter_ssl_pinning.js
```

Aqui eu me deparei com o primeiro ponto de atenção: o script não encontrou a `libflutter.so` durante a execução do aplicativo e retornou erro.

![Erro](/img/posts/Pasted%20image%2020240122190417.png)

Como por padrão a `libflutter.so` é carregada ao abrir os aplicativos nativos em Flutter, decidi escrever outro script para verificar o endereço de memória de cada módulo carregado ao abrir o aplicativo e validar este comportamento.

```javascript
Process.enumerateModules({
    onMatch: function(module) {
        console.log('Module name: ' + module.name + " - Base Address: " + module.base.toString());
        },
    onComplete: function(){}
    }
);
```

Para executar, utilizamos o seguinte comando:

```bash
frida -U -f com.example.h41stur.flutter_interceptor -l find_modules.js | grep libflutter.so
```

![Erro](/img/posts/Pasted%20image%2020240119171756.png)

E novamente a `libflutter.so` aparentemente não foi carregada. Isso me levantou a dúvida de que talvez exista um *delay* entre a chamada do aplicativo, e a invocação da biblioteca, então um `setTimeout` de 1 segundo foi adicionado ao script para validar.

```javascript
function process() {
	Process.enumerateModules({
		onMatch: function(module) {
			console.log('Module name: ' + module.name + " - Base Address: " + module.base.toString());
		},
		onComplete: function(){}
		}
	);
}

setTimeout(process, 1000);
```

![Endereço da libflutter](/img/posts/Pasted%20image%2020240119185417.png)

E desta vez, foi possível, portanto, o `setTimeout` também foi adicionado ao script anterior.

```javascript
function findPattern() {
    var m = Process.findModuleByName("libflutter.so"); // localiza o módulo em tempo de execução
    var pattern = "fe 0f 1c f8 f8 5f 01 a9 f6 57 02 a9 f4 4f 03 a9 91 dd 0b 94 68 1a 40 f9 15 e9 40 f9"; // pattern da função ssl_verify_peer_cert
    var ranges = m.enumerateRanges('r-x'); // localiza as áreas com permissão de leitura e execução dentro da libflutter.so

    // faz um loop em cada área r-x encontrada e busca pelo pattern dentro dela
    ranges.forEach(range => {
        Memory.scan(range.base, range.size, pattern, {
            onMatch: function(address, size) {
                console.log("[+] ssl_verify_peer_cert encontrada em: 0x" + (address - m.base).toString(16));
                hooking_ssl_verify(address); // invoca o hooking quando encontra o pattern
            }
        })
    })
}

// hooking
function hooking_ssl_verify(address)
{
    Interceptor.replace(address, new NativeCallback((pathPtr, flags) => {
        console.log("[+] Validação de certificado desabilitada");
        return 0;
    }, 'int', ['pointer', 'int']));
}

setTimeout(findPattern, 1000); // delay de 1 segundo
```

![Offset encontrado](/img/posts/Pasted%20image%2020240122191818.png)

Desta vez, não só o script foi executado sem erros, como encontrou o *offset* da função `ssl_verify_peer_cert`.

Ao interceptarmos o tráfego HTTPS com o BurpSuite, conseguimos o *bypass* do SSL *Pinning* com sucesso.

![Bypass do SSL Pinning](/img/posts/Pasted%20image%2020240122192325.png)

![Bypass do SSL Pinning](/img/posts/Pasted%20image%2020240122192419.png)

Este processo analisou especificamente o aplicativo sendo testado para entendê-lo e criar um *exploit* customizado para esta situação. Porém, nada impede que o processo possa ser melhorado para que o *exploit* seja atualizado para atender qualquer aplicativo utilizando a versão atual do Flutter.

O intuito aqui, não foi criar algo para que todos possam usar, mas sim o entendimento para que todos possam fazê-lo.

## Conclusão

Muito provavelmente existem outras formas de se obter o mesmo resultado, e até mesmo formas de automatizar este processo, que talvez eu explore em breve.

Porém, o melhor de tudo documentado neste post, foi o **processo de aprendizado** que de fato é o que torna nossa área tão divertida e dinâmica para trabalhar.

# Referências

- [https://docs.flutter.dev/resources/architectural-overview](https://docs.flutter.dev/resources/architectural-overview){:target="\_blank"}
- [https://firefox-source-docs.mozilla.org/security/nss/index.html](https://firefox-source-docs.mozilla.org/security/nss/index.html){:target="\_blank"}

