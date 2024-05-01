---
title: "Uma Jornada de Hacking de Hardware: Criando um Clonador de Sinais RF"
author: H41stur
date: 2024-05-01 01:00:00 -0300
categories: [Estudos, Hardware Hacking]
tags: [Arduino]
image: "/img/posts/replay-rf.png"
alt: "Uma Jornada de Hacking de Hardware: Criando um Clonador de Sinais RF"
---

![Uma Jornada de Hacking de Hardware: Criando um Clonador de Sinais RF](/img/posts/replay-rf.png)

# TL;DR

Este artigo detalha a construção de um dispositivo de clonagem de sinais RF para controles de portão usando **Arduino Nano** e a biblioteca **RC-Switch**. O dispositivo opera na frequência de 433 MHz e foca em sistemas de controle que utilizam códigos fixos para transmissão. O projeto explora vulnerabilidades inerentes a esses sistemas e demonstra como eles podem ser clonados e reproduzidos com facilidade. 

A motivação central do projeto foi aprender e entender as tecnologias subjacentes ao *hacking* de *hardware*. O artigo detalha a estrutura dos sinais de código fixo, as limitações de segurança associadas a eles, e fornece o passo a passo de montagem do dispositivo, incluindo *hardware*, *software* e testes práticos.

O código fonte e a montagem do *hardware* são explicados em detalhes, oferecendo *insights* para *hackers* interessados em explorar as fraquezas desses sistemas de controle. O artigo conclui com a importância de considerar sistemas de segurança mais robustos, como códigos rotativos, para proteger os acessos contra ataques de repetição.

# Motivação

Bom, este é o primeiro projeto que publico relacionado a *hardware hacking*, nada mirabolante, nada inovador e não é um dispositivo difícil de encontrar no mercado já pronto e de tamanho reduzido. Porém, comprar um pronto e usar sem ao menos entender o processo, só me tornaria um apertador botão, certo?

Sem contar que toda a diversão, a **VERDADEIRA** diversão do *hacking* seria negligenciada, os quais são os processos de aprendizado, de entendimento e por fim de subverter o sistema.

Assim como boa parte dos momentos de iluminação acontecem no meio da noite e não me deixam dormir, a ideia para montar meu próprio "clonador" de sinal de controle de abertura e fechamento de portões me surgiu no meio da noite, me acelerou e me fez já iniciar o planejamento.

O primeiro passo foi de fato entender como funciona a transmissão, o recebimento, o *encode* e a interpretação dos sinais do enviados pelo controle remoto.

O segundo passo, foi de fato decidir o que o *device* faria, o que por consequência levaria a lista de materiais e componentes que seriam utilizados no projeto, além, é claro, de como o código-fonte seria feito para atender a todos os requisitos.

Mais uma vez, esse processo foi ótimo, costumo dizer que é nele que reside o *hacking*, o *exploit* é só a consequência.

# Introdução

No contexto da segurança cibernética e do *hardware hacking*, o entendimento de tecnologias subjacentes é essencial para avaliar e explorar vulnerabilidades e subverter as defesas. Um exemplo desta aplicação encontra-se nos sistemas de controle de acesso remoto, como os controles de portões eletrônicos. Algo tão trivial no nosso dia-a-dia, mas que tem um peso enorme na segurança. Este estudo foca na análise e manipulação de sinais de RF (*Radio Frequency*) utilizados por esses controles, visando compreender suas vulnerabilidades e desenvolver métodos para subverter sistemas similares.

A frequência de 433 MHz é amplamente utilizada por oferecer um equilíbrio entre alcance, penetração de obstáculos e eficiência energética, sendo ideal para a comunicação de curto alcance em ambientes urbanos densos. A modulação e codificação dos sinais nesta frequência são as chaves para a comunicação eficaz e segura entre o controle e o receptor. Portanto, uma compreensão detalhada desses aspectos é crucial. 

Este artigo descreve a metodologia utilizada para interceptar e replicar sinais RF de 433 MHz, detalhando o *hardware* e *software* empregados, os desafios encontrados. Através deste estudo, busca-se entender melhor as falhas potenciais desses sistemas, ao ponto de criar um dispositivo para automatizar a exploração.

# *Overview* Sobre o Sistema de Controle Remoto de Portões

Os controles de abertura de portões eletrônicos, em sua maioria, operam em 433 MHz e utilizam uma combinação de frequência de rádio e codificação digital para diferenciar os sinais entre diferentes dispositivos e sistemas. Por mais que seja frequente a situação de que um controle consiga ativar diversos outros receptores, ainda existe um padrão finito de diferenciação de sinal, para que o mínimo de aleatoriedade possa existir entre os controles. Esta diferenciação, pode ser dividida em algumas características:

- **Frequência RF (Rádio Frequência)**: A frequência de 433 MHz é comumente usada para comunicação de curto alcance em dispositivos como controles remotos de portões eletrônicos, sistemas de alarme e outros dispositivos de automação residencial. Essa frequência é apenas o meio pelo qual o sinal é transmitido pelo ar.
- **Codificação e Modulação**: O aspecto mais crucial que permite que diferentes controles remotos operem portões distintos, mesmo na mesma frequência, é a codificação do sinal. O controle remoto codifica a informação que determina as instruções a serem enviadas ao portão. Essa codificação pode ser feita de várias maneiras:
	- **Código Fixo**: Em sistemas mais antigos ou mais simples, cada controle é programado com um código fixo enviado cada vez que o botão é pressionado. Este código deve corresponder ao configurado no receptor do portão para que a ação (abrir ou fechar) seja executada.
	- **Código Rotativo ou *Hopping Code***: Nos sistemas mais avançados, utiliza-se um mecanismo de código rotativo. Aqui, cada vez que o controle é usado, ele envia um novo código, gerado por um algoritmo sincronizado tanto no controle quanto no receptor. Esse código é baseado em uma sequência predeterminada ou pseudoaleatória, tornando muito mais difícil a interceptação ou duplicação do sinal.
- **Segurança Adicional**: Além da modulação e codificação, medidas de segurança adicionais podem ser implementadas, como criptografia dos dados enviados. Isso adiciona uma camada de proteção, dificultando que os sinais sejam copiados ou manipulados por agentes não autorizados.
- **Endereçamento**: Em alguns sistemas, pode-se configurar endereços específicos (semelhantes a identificadores únicos) nos controles e receptores, permitindo que múltiplos dispositivos operem na mesma frequência sem interferência, pois cada conjunto comunica-se apenas com seu par correspondente.

Estas técnicas permitem que diversos controles operem em um ambiente com muitos dispositivos sem causar interferências indesejadas, mantendo a operação segura e eficiente. 

Como na esmagadora das vezes, nos deparamos no dia-a-dia com sinais de código fixo, todo este projeto é baseado neste tipo de sinal.

## Modulação de Código Fixo

O sistema de código fixo é um dos métodos mais simples e, por isso, também um dos mais vulneráveis em termos de segurança para sistemas de controle remoto como os utilizados em portões eletrônicos. Seu funcionamento tem uma estrutura simplificada:

### Importância da Frequência de 433 MHz

A frequência de 433 MHz é amplamente utilizada para dispositivos de comunicação de curto alcance devido ao equilíbrio entre alcance, penetração de obstáculos e eficiência energética. É uma das bandas ISM (Indústria, Ciência e Medicina), que são faixas de frequência reservadas internacionalmente para esses usos. Sua popularidade resulta na vasta gama de dispositivos disponíveis no mercado que operam nessa frequência, tornando-se uma escolha comum para dispositivos de controle remoto de portões, sistemas de alarme e automação residencial.

### Estrutura do Código Fixo

No sistema de código fixo, cada controle remoto possui um código único pré-programado, que é transmitido toda vez que o botão é pressionado. Este código geralmente consiste em uma sequência de bits que pode representar comandos simples como abrir ou fechar.

### Composição do Código

O código é geralmente uma combinação binária (por exemplo, 0101001110110), e sua extensão pode variar, mas encontra-se comumente entre 8 a 12 bits, permitindo assim um número finito, mas amplo, de combinações possíveis. Este código binário é transmitido com a frequência portadora de 433 MHz usando uma forma de modulação.

### Modulação

A modulação mais comum para estes sinais é a ***Amplitude Shift Keying*** (ASK), uma forma de modulação onde a presença de um sinal em determinada amplitude representa um bit '1' e sua ausência ou uma amplitude diferente representa um bit '0'. Outra forma comum é a ***Frequency Shift Keying*** (FSK), onde diferentes frequências são usadas para representar os bits '0' e '1'.

### Transmissão e Recepção

Quando o usuário pressiona o botão no controle remoto, o código binário fixo é modulado com a frequência portadora e transmitido pelo ar. O receptor no portão eletrônico é configurado para reconhecer e responder apenas ao seu código específico. Quando recebe um sinal, ele demodula a frequência portadora para extrair o código binário. Se o código recebido corresponde ao código esperado, o receptor executa a ação correspondente (por exemplo, abrir ou fechar o portão).

### Implementações Típicas

Controles de código fixo são comuns em sistemas mais antigos ou mais baratos onde a segurança não é uma preocupação primordial (ou deveria ser, mas não é). Com a evolução das tecnologias de segurança, muitos sistemas migraram para códigos rotativos ou criptografados para evitar essas vulnerabilidades.

### Exemplo de Análise de um Sistema de Código Fixo

Imaginemos um cenário onde o controle remoto de um portão eletrônico utiliza modulação AM com código fixo. Este sistema tem uma codificação de 10 bits para o código do controle e uma frequência portadora de 433 MHz.

#### Estrutura do Sinal

- **Código Fixo**: Cada controle é programado com um código de 10 bits, como `1011010010`.
- **Modulação**: O sinal de RF usa ASK, onde um nível de amplitude alta indica '1' e baixa indica '0'.
- **Taxa de Transmissão**: O sinal é transmitido a 1.000 bits por segundo (bps).

#### Transmissão e Recepção

Quando o botão do controle é pressionado, o código `1011010010` é modulado na frequência de 433 MHz. Cada bit é representado pela amplitude da onda de RF, onde um pulso alto é '1' e um pulso baixo é '0'. O receptor do portão, configurado para reconhecer este código, aciona o portão ao identificar o padrão `1011010010`.

#### Segurança e Vulnerabilidade

A segurança do sistema é baixa devido ao uso de código fixo. Um atacante pode usar um dispositivo de captura de sinais RF para interceptar o sinal enquanto o controle é usado. O atacante então pode retransmitir esse sinal, conhecido como ataque de *replay*, para ganhar acesso ao portão.

## Decodificação do Código Fixo

A decodificação de um sinal de código fixo transmitido por controle remoto RF geralmente envolve várias partes distintas que podem ser analisadas e entendidas. As principais, que o tornam únicos, são:

### Protocolo

O protocolo é o conjunto de regras que definem como os dados são formatados e transmitidos. Em controles remotos de código fixo, o protocolo especificará:

- **Formato do sinal**: Como os bits são representados no sinal.
- **Estrutura da mensagem**: A sequência e significado dos bits (por exemplo, bits de início, dados, *checksum*, bits de parada).

### Tamanho do Pulso

O tamanho do pulso refere-se à duração de cada sinal transmitido que representa um bit. Em modulações como ASK ou FSK, o tamanho do pulso pode ser fixo para todos os bits ou variar dependendo do bit ser '0' ou '1':

- **Pulso longo**: Geralmente representa um '1'.
- **Pulso curto**: Geralmente representa um '0'.

### Taxa de Transmissão

Esta é a velocidade com que os bits são transmitidos, geralmente medida em bits por segundo (bps). Em controles remotos, essa taxa não precisa ser muito alta, pois os comandos são simples e o volume de dados é pequeno.

### Modulação

A técnica de modulação usada para transmitir o sinal:

- **ASK (*Amplitude Shift Keying*)**: Na modulação ASK, a amplitude da onda portadora é alterada para representar bits '1' e '0'. Um sinal de amplitude maior pode representar um '1', enquanto um sinal de amplitude menor representa '0'. É uma técnica simples de implementar, mas pode ser suscetível a ruídos e interferências.
- **FSK (*Frequency Shift Keying*)**: No FSK, a frequência da onda portadora é alterada para representar diferentes bits. Uma frequência pode representar um '1', enquanto uma frequência diferente pode representar '0'. Esta técnica é mais robusta contra interferências do que o ASK, mas é mais complexa de implementar.

### Intervalo entre Transmissões

Em muitos controles, há um intervalo definido entre a repetição do código para evitar colisão de sinal e permitir que o receptor processe o comando corretamente.

### *Checksum*

Algumas implementações de código fixo podem incluir um *checksum* ao final do código para verificar a integridade do sinal recebido. O *checksum* é uma soma de verificação que ajuda a identificar erros na transmissão.

### Padrão de Codificação

Alguns sistemas de código fixo podem usar uma codificação específica ou um padrão para aumentar a redundância e reduzir a possibilidade de interferência de outros dispositivos.

### Dados de Identificação do Dispositivo

Inclui um código único para cada dispositivo ou usuário, permitindo que o receptor identifique qual dispositivo está enviando o comando.

## O Problema do Processo de Clonagem do Sinal

Como um dos passos cruciais para o meu dispositivo era armazenar os sinais interceptados para que pudessem ser reproduzidos a qualquer momento, o processo n!ao é tão simples quanto receber, armazenar e reproduzir.

É preciso receber, decodificar, armazenar as partes importantes para reprodução, e tudo isso deve ser feito de uma forma que eu consiga ler em um *display* e escolher qual sinal reproduzir posteriormente.

Felizmente, existe a biblioteca **RC-Switch** para o **Arduino** que lida com sinais RF muito bem, com ela, é possível, no momento da interceptação, decodificar sinais de código fixo em suas principais partes:

### Protocolo

A biblioteca rc-switch pode reconhecer vários protocolos diferentes, cada um com sua própria configuração de tamanho de pulso, ordem de bits e estrutura de dados. 

### Tamanho do Pulso

O tamanho do pulso é crucial porque diferentes protocolos podem usar diferentes durações de pulsos para representar bits '0' e '1'. A precisão na medição do tamanho do pulso é fundamental para a decodificação correta dos dados transmitidos.

### Valor Recebido

Esta é a informação bruta capturada pelo receptor. Normalmente, é apresentada como um valor binário, decimal ou hexadecimal. Este valor representa diretamente os dados enviados pelo transmissor. No contexto de um controle remoto de portão, por exemplo, esse valor poderia ser um código único que ativa o mecanismo de abertura ou fechamento do portão.

Para reprodução do sinal, este valor tem pouca relevância, porém, por se tratar de um valor decimal, acredito ser um bom valor para identificar os sinais posteriormente.

### Comprimento

O comprimento é o número total de bits do sinal recebido. Esta informação é vital porque diz quantos bits compõem a mensagem completa. Saber o comprimento ajuda na análise do sinal, permitindo separar e interpretar cada parte da mensagem corretamente, garantindo que toda a informação seja considerada durante a decodificação.

### Código Decimal

Embora a informação possa ser capturada e manipulada em forma binária ou hexadecimal, a rc-switch também fornece uma conversão para código decimal. Isso simplifica a visualização e a análise do sinal, especialmente para aqueles que podem não estar familiarizados com a leitura direta de valores binários ou hexadecimais. O código decimal é essencialmente outra forma de representar o mesmo valor, mas em um sistema numérico que é mais comumente usado em aplicações diárias.

# O Hardware

Para este projeto, escolhi utilizar uma **Arduino Nano** para ser o *core* do dispositivo, pelo seu tamanho reduzido em relação a espaço de armazenamento, além da facilidade de encontrar e o baixo custo (comprando da China fica ainda muito mais barato).

![Arduino Nano](/img/posts/Pasted%20image%2020240430210122.png)

Como um dos requisitos do projeto é armazenar os sinais armazenados, eu precisava de um *storage*, optei por utilizar um módulo leitor de cartão micro SD e um cartão de 500 mb (era o único que eu tinha disponível, não precisava de tanto).

![Modulo Leitor Cartão Micro Sd](/img/posts/Pasted%20image%2020240430210449.png)

Para interceptar e reproduzir sinais, utilizei um par de módulos receptor e transmissor de RF 433 MHz.

![Módulos RF 433MHz Transmissor e Receptor](/img/posts/Pasted%20image%2020240430210725.png)

Para visualizar, mesmo que de forma básica, tudo que está sendo feito, além de administrar os sinais interceptados e transmitidos, eu precisava de uma tela. Existem infinitas possibilidades no mercado, porém, optei pelo mais simples, um *display* LCD de 16x2 já com uma placa I2C integrada.

![Display LCD 16x2 com Backlight Azul e I2C](/img/posts/Pasted%20image%2020240430211034.png)

E mais alguns componentes menores, como botões *pull-up*, transistores e capacitores, conectores de bateria e fonte, *protoboard* para montar o protótipo, placa ilhada para montar o projeto final, barras de pinos e um bocado de diversão.

## Ligação dos Componentes

A lista de componentes apresentada é o suficiente para montar o *hardware*, porém um problema surgiu: a Arduino funciona com uma tensão de **5V**, e eu só tinha disponível uma fonte ou uma bateria de **9V**.

Sim, eu sei que existem infinitos módulos reguladores de tensão no mercado que funcionam basicamente *plug and play*, mas esse tipo de circuito é tão simples, que me faria sentir um completo inutil se eu não fizesse o meu próprio.

Para isso, utilizei um regulador de tensão modelo **LM7805**. Consultando os [*datasheets*](https://www.alldatasheet.com/view.jsp?Searchword=Lm7805&gad_source=1&gclid=CjwKCAjwrcKxBhBMEiwAIVF8rF3yOAA21bEhTRJay8lU52VRQT67N8D9iNfUheP3NcvrmDAgo4FSshoCsc0QAvD_BwE) dos fabricantes, para saber qual dos capacitores utilizar na estabilização, vi que os de **25V 10uF** que já tinha disponíveis serviriam perfeitamente. O circuito fica desta forma:

![Regulador de tensão](/img/posts/Pasted%20image%2020240430220349.png)

Com o problema da alimentação resolvido, as conexões entre os módulos e a Arduino Nano ficaram desta forma:

- **Arduino Nano**:
	- **Alimentação**: Suprimento de 5V conectado ao **VIN** e terra ao **GND**.
- **Leitor de Cartão SD**:
	- **MISO**: PIN 12 do Arduino.
	- **MOSI**: PIN 11 do Arduino.
	- **SCK**: PIN 13 do Arduino.
	- **CS**: PIN 10 do Arduino.
	- **VCC**: 5V.
	- **GND**: GND
- **Receptor RF 433 MHz**:
	- **DATA**: PIN 2 do Arduino.
	- **VCC**: 5V.
	- **GND**: GND.
- **Transmissor RF 433 MHz**:
	- **DATA**: PIN 3 do Arduino.
	- **VCC**: 5V.
	- **GND**: GND.
- **Display LCD 16x2 com Backlight Azul e I2C**
	- **VCC**: 5V.
	- **GND**: GND.
	- **SDA**: PIN A4 do Arduino.
	- **SCL**: PIN A5 do Arduino.
- **Botões**:
	- Um lado de cada botão ao `GND` e o outro lado aos PINs 4, 5, 6, 7 e 8.

Ficou algo parecido com esta bagunça na *protoboard*:

![Circuito ligado na *protoboard*.](/img/posts/Pasted%20image%2020240430221455.png)

## Testando a Interceptação de *Decode* do Sinal

Uma vez com os componentes conectados, precisei testar a interceptação de sinais. A biblioteca RC-Switch possui diversos métodos que fazem o *decode* do sinal e já entregam partes relevantes. 

A princípio, o código abaixo foi criado:

```cpp
#include <RCSwitch.h>

RCSwitch mySwitch = RCSwitch(); // Cria um objeto mySwitch da classe RCSwitch para controlar o transmissor/receptor RF.

void setup() {
	Serial.begin(9600); // Inicia a comunicação serial.
	mySwitch.enableReceive(0); // Ativa a recepção de dados no pino 2 do Arduino (interrupção 0).
}

void loop() {
	// Verifica se há dados disponíveis para leitura pelo receptor RF.
	if (mySwitch.available()) {
		long receivedValue = mySwitch.getReceivedValue(); // Lê o valor recebido.
		long bitLength = mySwitch.getReceivedBitlength(); // Lê o número de bits do valor recebido.
		long pulseLength = mySwitch.getReceivedDelay(); // Lê o comprimento do pulso do sinal recebido.
		long protocol = mySwitch.getReceivedProtocol(); // Lê o protocolo do sinal recebido.
		
		// Imprime no monitor serial as informações do sinal recebido.
		Serial.print(" PulseLength: ");
		Serial.print(pulseLength);
		Serial.print(" Received ");
		Serial.print(receivedValue);
		Serial.print("/ ");
		Serial.print(bitLength);
		Serial.print("bit ");
		Serial.print("Protocol: ");
		Serial.println(protocol);
		mySwitch.resetAvailable(); // Limpa o status disponível para receber o próximo sinal.
	}
}
```

Este código, a princípio trouxe tudo que precisamos, com um pouco de organização, conseguimos transformar tudo em uma *string* para ser armazenado no cartão SD. Porém, ainda existe um detalhe: a função que transmite o sinal, precisa de uma *string* binária, e este valor é calculado conforme o valor decimal recebido e com comprimento do sinal.

Para isso, a função `dec2bin()` foi implementada, ela recebe estes dois valores e após algumas rotações, devolve a *string* binária. Com sua implementação o código fica desta maneira:

```cpp
#include <RCSwitch.h>

RCSwitch mySwitch = RCSwitch(); // Cria um objeto mySwitch da classe RCSwitch para controlar o transmissor/receptor RF.

void setup() {
	Serial.begin(9600); // Inicia a comunicação serial.
	mySwitch.enableReceive(0); // Ativa a recepção de dados no pino 2 do Arduino (interrupção 0).
}

// Declaração de uma função estática para converter um número decimal em uma cadeia de caracteres binários.
static char * dec2bin(unsigned long Dec, unsigned int bitLength);

void loop() {
	// Verifica se há dados disponíveis para leitura pelo receptor RF.
	if (mySwitch.available()) {
		long receivedValue = mySwitch.getReceivedValue(); // Lê o valor recebido.
		long bitLength = mySwitch.getReceivedBitlength(); // Lê o número de bits do valor recebido.
		long pulseLength = mySwitch.getReceivedDelay(); // Lê o comprimento do pulso do sinal recebido.
		long protocol = mySwitch.getReceivedProtocol(); // Lê o protocolo do sinal recebido.
		const char* b = dec2bin(receivedValue, bitLength);  // Converte o valor recebido para binário usando a função dec2bin.
    
	    // Imprime no monitor serial as informações do sinal recebido.
	    Serial.print("Binary: ");
	    Serial.print(b);		
		Serial.print(" PulseLength: ");
		Serial.print(pulseLength);
		Serial.print(" Received ");
		Serial.print(receivedValue);
		Serial.print("/ ");
		Serial.print(bitLength);
		Serial.print("bit ");
		Serial.print("Protocol: ");
		Serial.println(protocol);
		mySwitch.resetAvailable(); // Limpa o status disponível para receber o próximo sinal.
	}
}

// Função que converte um número decimal em uma string binária preenchida com zeros até alcançar o comprimento de bits especificado.
static char * dec2bin(unsigned long Dec, unsigned int bitLength) {
	static char bin[64];  // Array estático para armazenar a string binária.
	unsigned int i = 0;  // Índice para construir a string binária.
	
	// Converte o número decimal em binário, armazenando de trás para frente.
	while (Dec > 0) {
		bin[32 + i++] = ((Dec & 1) > 0) ? '1' : '0';  // Adiciona '1' ou '0' ao array.
		Dec >>= 1;  // Desloca o número um bit para a direita.
	}
	
	// Preenche o restante da string binária com zeros até o comprimento especificado.
	for (unsigned int j = 0; j < bitLength; j++) {
		if (j >= bitLength - i) {
			bin[j] = bin[31 + i - (j - (bitLength - i))];  // Inverte a ordem dos bits já armazenados.
		} else {
			bin[j] = '0';  // Completa com zeros.
		}
	}
	
	bin[bitLength] = '\0';  // Adiciona o terminador de string.
	
	return bin;  // Retorna a string binária.
}
```


Ao executarmos o programa, e utilizarmos um controle de portão próximo ao circuito, temos os dados impressos no monitor serial da forma que foi programado.

![Sinal interceptado do controle.](/img/posts/Pasted%20image%2020240424061900.png)

# O Software

Desenvolver o código para este projeto foi o desafio mais gratificante de todo o processo, exigiu entendimento, *debug* e implementações ao longo de todo o percurso. E tudo isso ainda me preocupando em não cobrir todo o espaço de memória do Arduino Nano.

Conforme o padrão do Arduino, são duas as funções obrigatórias em um programa: a `void setup()` que é executada somente uma vez, no início do programa, e geralmente carrega configurações iniciais, e a `void loop()` que é executada continuamente enquanto o microcontrolador estiver em funcionamento.

Porém, nada impede que mais funções sejam criadas e utilizadas na execução do programa.

Seguindo com os requisitos do projeto, detalharei cada parte do código, e em seguida juntamos tudo.

## Importando Bibliotecas e Configurando Variáveis Globais

A primeira parte do código, se trata simplesmente da importação das bibliotecas e configurações de variáveis utilizadas ao longo do programa.

```cpp
#include <SPI.h>  // Inclui a biblioteca SPI, usada para comunicação com dispositivos como o cartão SD.
#include <SD.h>  // Inclui a biblioteca para operar com o cartão SD.
#include <Wire.h>  // Inclui a biblioteca para comunicação I2C, usada aqui para o display LCD.
#include <LiquidCrystal_I2C.h>  // Inclui a biblioteca para operar o display LCD I2C.
#include <RCSwitch.h>  // Inclui a biblioteca para controlar transmissão e recepção de sinais RF.

LiquidCrystal_I2C lcd(0x27, 16, 2);  // Cria um objeto lcd para o display, especificando endereço I2C, 16 colunas e 2 linhas.

RCSwitch mySwitch = RCSwitch();  // Cria um objeto para controlar a recepção e transmissão de sinais RF.

File controls;  // Variável para manipular arquivos no cartão SD.

bool displayUpdate = true;  // Flag para controlar a atualização do display.
int lastSignal = -1;  // Guarda o último sinal selecionado para evitar atualizações desnecessárias.

// Declara uma função para converter números decimais em strings binárias.
static char * dec2bin(unsigned long Dec, unsigned int bitLength);
```

## Função `setup()`

A função inicial neste programa, não difere de outros, servindo para carregar configurações iniciais e imprimir algo na tela LCD.

```cpp
void setup() {
  Serial.begin(9600);  // Inicia comunicação serial a 9600 bps.

  lcd.init();  // Inicializa o display LCD.
  lcd.backlight();  // Ativa a luz de fundo do LCD.
  lcd.clear();  // Limpa o display.
  lcd.print("Iniciando cartao SD...");  // Exibe uma mensagem inicial.
  delay(5000);  // Espera 5 segundos.

  if (!SD.begin(10)) {  // Tenta iniciar o cartão SD no pino CS 10.
    lcd.clear();
    lcd.print("Falha no cartao SD");  // Mostra uma mensagem de erro se falhar.
    while(1);  // Trava o programa se não conseguir iniciar o SD.
  }

  // Configura os pinos dos botões como entradas com resistores de pull-up.
  pinMode(4, INPUT_PULLUP);
  pinMode(5, INPUT_PULLUP);
  pinMode(6, INPUT_PULLUP);
  pinMode(7, INPUT_PULLUP);
  pinMode(8, INPUT_PULLUP);

  mySwitch.enableReceive(digitalPinToInterrupt(2));  // Habilita a recepção de RF no pino 2 (interrupção 0).
  mySwitch.enableTransmit(3);  // Habilita a transmissão de RF no pino 3.
}
```

## Função `dec2bin()`

Esta função é a mesma criada no programa para testar a interceptação, servindo para fazer a conversão do valor decimal do sinal, juntamente com seu comprimento em uma representação binária.

```cpp
// Função para converter o valor decimal de um sinal e seu comprimento em uma representação binária.
static char * dec2bin(unsigned long Dec, unsigned int bitLength) {
  static char bin[64];  // Array estático para armazenar a representação binária.
  unsigned int i=0;

  while (Dec > 0) {
    bin[32+i++] = ((Dec & 1) > 0) ? '1' : '0';  // Converte cada bit de Dec para '1' ou '0'.
    Dec = Dec >> 1;  // Desloca Dec para a direita.
  }

  for (unsigned int j = 0; j < bitLength; j++) {
    if (j >= bitLength - i) {
      bin[j] = bin[31 + i - (j - (bitLength - i))];  // Reorganiza os bits na ordem correta.
    } else {
      bin[j] = '0';  // Preenche os espaços restantes com '0'.
    }
  }
  bin[bitLength] = '\0';  // Adiciona o caractere nulo no final para indicar o término da string.
  
  return bin;  // Retorna a string binária.
}
```

## Funções `loop()` e `handler()` 

A função `void loop()` que por si só é executada repetidamente enquanto o microcontrolador estiver em operação, chama a função criada `void handler()` que também tem um *loop* infinito. Esta redundância de *loops* se fez necessária para eu lidar com as atualizações do painel LCD, porém acredito haver uma solução melhor e mais inteligente que possa ser explorada. A princípio está funcional.

A função `void handler()` de fato controla o fluxo da interceptação dos sinais emitidos ao redor, assim como lida com outras funções que manipulam outras funcionalidades operadas pelos botões.

O primeiro ponto a se destacar é a inicialização do método `mySwitch.available()` que verifica se o receptor interceptou algum sinal ao redor, caso haja um sinal, inicia-se o fluxo de *decode* deste sinal, enviando cada parte relevante para uma variável além da conversão do sinal decimal e comprimento em uma representação binária com a função `dec2bin()`.

Em seguida, a função abre o arquivo **data.txt** no cartão SD, e grava uma *string* com todos os valores separados por vírgula. Este ponto é importante, pois depois precisaremos separar esta *string* novamente, e a vírgula nos ajudara nesse processo.

Após a gravação, o arquivo é fechado e as funções que lidam com *display* e botões são invocadas.

```cpp
void handler() {
  static int selectedSignal = 0;  // Mantém o índice do sinal selecionado.

  while(1) {  // Loop infinito dentro do handler.
    
    if (mySwitch.available()) {  // Verifica se há um sinal RF recebido.
  
      long receivedValue = mySwitch.getReceivedValue();  // Recebe o valor do sinal.
      long bitLength = mySwitch.getReceivedBitlength();  // Recebe o comprimento do bit do sinal.
      long pulseLength = mySwitch.getReceivedDelay();  // Recebe o comprimento do pulso do sinal.
      long protocol = mySwitch.getReceivedProtocol();  // Recebe o protocolo do sinal.
      const char* b = dec2bin(receivedValue, bitLength);  // Converte o valor recebido para binário.
      
      if (receivedValue != 0) {  // Se o valor recebido for válido (diferente de zero).
        lcd.clear();
        lcd.print("Gravando no SD card...");  // Informa que está gravando no SD.
        delay(1000);  // Espera 1 segundo.
        controls = SD.open("data.txt", FILE_WRITE);  // Abre o arquivo "data.txt" para escrita.
        if (controls) {  // Se o arquivo foi aberto corretamente.
          controls.print(receivedValue);  // Escreve o valor recebido.
          controls.print(",");
          controls.print(b);  // Escreve o valor binário.
          controls.print(",");
          controls.print(pulseLength);  // Escreve o comprimento do pulso.
          controls.print(",");
          controls.println(protocol);  // Escreve o protocolo e uma nova linha.
          
          controls.close();  // Fecha o arquivo.
          displayUpdate = true;  // Seta a flag para atualizar o display.
        }
      }
      mySwitch.resetAvailable();  // Reseta o status de disponibilidade do RCSwitch.
    }
    handleButtons(&selectedSignal);  // Chama a função para manipular os botões.

    if (displayUpdate || selectedSignal != lastSignal) {  // Se necessário, atualiza o display.
      displaySignals(selectedSignal);  // Chama a função para exibir os sinais.
      lastSignal = selectedSignal;  // Atualiza o último sinal exibido.
      displayUpdate = false;  // Reseta a flag de atualização.
    }
  }
}
```


## Função `handleButtons()`

Conforme as funcionalidades planejadas, eu preferi o uso de **5** botões para administrá-las para fazer o mínimo possível de reuso de botões. E para segmentar bem o código, preferi criar esta função unicamente para controlar o fluxo da aplicação conforme os botões são pressionados. Basicamente os botões seguem esta ordem da esquerda para direita na placa:

- **Botão 1**: Movimenta para cima o menu de escolha sinais para reprodução;
- **Botão 2**: Movimenta para baixo o menu de escolha sinais para reprodução;
- **Botão 3**: Seleciona o sinal e inicia a transmissão;
- **Botão 4**: Apaga todos os sinais do cartão SD;
- **Botão 5**: Encerra a transmissão do sinal.

```cpp
void handleButtons(int *selectedSignal) {
  // Verifica o estado dos botões e ajusta o sinal selecionado ou outras ações.
  if (digitalRead(4) == LOW) {
    (*selectedSignal)++;  // Incrementa o sinal selecionado.
    displayUpdate = true;  // Seta a flag para atualizar o display.
    delay(200);  // Delay para debouncing.
  } else if (digitalRead(5) == LOW) {
    (*selectedSignal)--;  // Decrementa o sinal selecionado.
    displayUpdate = true;
    delay(200); 
  } else if (digitalRead(6) == LOW) {
    transmitSignal(*selectedSignal);  // Transmite o sinal selecionado.
    delay(200);  
  } else if (digitalRead(7) == LOW) {
    resetSDCard();  // Reseta o cartão SD.
    displayUpdate = true;
    delay(200);
  } else if (digitalRead(8) == LOW) {
    lcd.clear();
    lcd.print("Transmissao encerrada");  // Exibe mensagem de transmissão encerrada.
    delay(1000);
    displayUpdate = true;
  }
}
```

## Função `getValue()`

Conforme vimos na função `handler()` todas as informações capturadas que representam as propriedades do sinal, são salvas em uma *string* separados por vírgulas no arquivo **data.txt** no cartão SD, portanto, quando fizermos a leitura de cada linha no arquivo, teremos uma nova *string* com estas informações.

Neste caso, para utilizarmos separadamente, precisamos separar estas propriedades novamente, e é neste ponto que a função `getValue()` entra, funcionando como uma implementação de `split`.

Esta função recebe uma *string*, um separador, e um índice, e por consequência, retorna o valor contido no índice após a separação da *string.*

```cpp
// Função para extrair um valor específico de uma string delimitada por um separador.
String getValue(String data, char separator, int index) {
  int found = 0;
  int strIndex[] = { 0, -1 };
  int maxIndex = data.length() - 1;

  for (int i = 0; i <= maxIndex && found <= index; i++) {
    if (data.charAt(i) == separator || i == maxIndex) {
      found++;
      strIndex[0] = strIndex[1] + 1;
      strIndex[1] = (i == maxIndex) ? i+1 : i;
    }
  }
  return found > index ? data.substring(strIndex[0], strIndex[1]) : "";
}
```


## Função `displaySignals()`

Ainda seguindo com a segmentação e organização do código, preferi criar uma função exclusiva para administrar as opções do menu responsáveis pela escolha dos sinais armazenados no cartão SD, para reprodução.

Esta função basicamente nos permite escolher um sinal entre os armazenados. Com o uso da função `fetValue()` para capturar somente o valor decimal do sinal na *string* para servir de referência, ela adiciona o caractere ">" no sinal a ser selecionado, facilitando o entendimento do menu.

```cpp
void displaySignals(int selectedSignal) {
  // Exibe os sinais no display.
  lcd.clear();
  
  controls = SD.open("data.txt");  // Abre o arquivo "data.txt".
  int lineNum = 0;
  while (controls.available() && lineNum < 2) {  // Lê até duas linhas do arquivo.
    lcd.setCursor(0, lineNum);
    String line = controls.readStringUntil('\n');  // Lê uma linha.
    String sign = getValue(line, ',', 0);  // Obtém o valor da linha.
    if (lineNum == selectedSignal % 2) {
      lcd.print(">");  // Marca o sinal selecionado.
    }
    lcd.print(sign);  // Exibe o sinal.
    lineNum++;
  }
  controls.close();  // Fecha o arquivo.
}
```

## Função `transmitSignal()`

Esta é a função mais crítica do processo, pois ela faz a leitura do sinal selecionado no cartão SD. Todos os dados vêm como *string*, portanto, cada propriedade lida, precisa ser convertida em seus devidos tipos antes de serem utilizados na configuração do sinal a ser reproduzido.

Basicamente esta função recebe o sinal que deve ser lido, separa sua *string* utilizando a função `getValue()` para obter os valores de **valor decimal do sinal**, **a representação binária do sinal**, **o comprimento do pulso** e o **protocolo** a ser utilizado.

Em seguida, a conversão em seus tipos é feita, gerando novas variáveis utilizadas na pré-configuração do sinal na biblioteca **RC-Switch**.

Após a configuração, a transmissão se inicia, eu preferi fazer um *loop* infinito com uma condição de parada utilizando o **botão 5** e alguns *delays* entre cada transmissão. Esta decisão veio depois de alguns testes onde a distância entre o transmissor e o receptor, me obrigou a insistir muitas vezes no botão de transmissão antes de conseguir uma resposta positiva. Automatizar este *loop* e inserir a condição de parada, me pareceu um processo menos custoso do que insistir em apertar o botão até conseguir.

```cpp
void transmitSignal(int signalIndex) {
  // Função para transmitir o sinal selecionado.
  controls = SD.open("data.txt");  // Abre o arquivo "data.txt".
  int lineNum = 0;
  while (controls.available()) {
    String line = controls.readStringUntil('\n');  // Lê uma linha do arquivo.
    if (lineNum == signalIndex) {  // Verifica se a linha corresponde ao sinal selecionado.
      lcd.clear();
      lcd.print("Transmitindo sinal: ");  // Informa que está transmitindo.

      String sign = getValue(line, ',', 0);  // Obtém o valor do sinal.
      String binary = getValue(line, ',', 1);  // Obtém a representação binária.
      String pulseLength = getValue(line, ',', 2);  // Obtém o comprimento do pulso.
      String protocol = getValue(line, ',', 3);  // Obtém o protocolo.

      const char* b = binary.c_str();  // Converte a string binária para const char*.
      unsigned long pl = pulseLength.toInt();  // Converte o comprimento do pulso para inteiro.
      unsigned long p = protocol.toInt();  // Converte o protocolo para inteiro.

      mySwitch.setProtocol(p);  // Configura o protocolo no RCSwitch.
      mySwitch.setPulseLength(pl);  // Configura o comprimento do pulso no RCSwitch.

      lcd.print(sign);  // Exibe o valor do sinal.

      while (1) {
        delay(1000);  // Delay entre as transmissões.
        if (digitalRead(8) == LOW) {
          lcd.clear();
          lcd.print("Transmissao encerrada.");  // Informa o fim da transmissão.
          delay(1000);
          break;
        }
        mySwitch.send(b);  // Envia o sinal.
        
        delay(2000);
      }

      if (digitalRead(8) == LOW) {
        lcd.clear();
        lcd.print("Transmissao encerrada.");
        delay(1000);
        break;
      }
    } 
    lineNum++;
  }
  controls.close();  // Fecha o arquivo.
}
```

## Função `resetSDCard()`

Por último, mas não menos importante, decidi pelo uso de uma função que apagasse todo o conteúdo do cartão SD diretamente pelo dispositivo. Isso ajudou muito na fase de testes, onde eu precisava gravar vários sinais e depois limpar a sujeira armazenada. Decidi deixá-la em produção pela utilidade.

Esta função basicamente checa se o arquivo **data.txt** existe no cartão SD, se sim, remove o arquivo.

```cpp
void resetSDCard() {
  // Reseta o cartão SD.
  controls = SD.open("data.txt", FILE_WRITE);  // Abre o arquivo "data.txt" para escrita.
  if (controls) {
    controls.close();  // Fecha o arquivo.
  }
  SD.remove("data.txt");  // Remove o arquivo "data.txt" do cartão SD.

  lcd.clear();
  lcd.print("Cartao resetado.");  // Exibe mensagem de cartão resetado.
  delay(1000);
}
```

## Juntando tudo

Ao final, temos o programa completo que segue o fluxo de trabalho planejado e lida com as necessidades de interceptação e reprodução dos sinais.

```cpp
#include <SPI.h>
#include <SD.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <RCSwitch.h>

LiquidCrystal_I2C lcd(0x27, 16, 4);

RCSwitch mySwitch = RCSwitch();

File controls;

bool displayUpdate = true;
int lastSignal = -1;
static char * dec2bin(unsigned long Dec, unsigned int bitLength);

void setup() {
  Serial.begin(9600);

  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.print("Iniciando cartao SD...");
  delay(5000);

  if (!SD.begin(10)) {
    lcd.clear();
    lcd.print("Falha no cartao SD");
    while(1);
  }

  pinMode(4, INPUT_PULLUP);
  pinMode(5, INPUT_PULLUP);
  pinMode(6, INPUT_PULLUP);
  pinMode(7, INPUT_PULLUP);
  pinMode(8, INPUT_PULLUP);

  mySwitch.enableReceive(digitalPinToInterrupt(2));
  mySwitch.enableTransmit(3);

}

void loop() {
  handler();
}

void handler() {
  static int selectedSignal = 0;
  
  while(1) {
    
    if (mySwitch.available()) {
  
      long receivedValue = mySwitch.getReceivedValue();
      long bitLength = mySwitch.getReceivedBitlength();
      long pulseLenght = mySwitch.getReceivedDelay();
      long protocol = mySwitch.getReceivedProtocol();
      const char* b = dec2bin(receivedValue, bitLength);    
      
      
      if (receivedValue != 0) {
        lcd.clear();
        lcd.print("Gravando no SD card...");
        delay(1000);
        lcd.clear();
        controls = SD.open("data.txt", FILE_WRITE);
        if (controls) {
          controls.print(receivedValue);
          controls.print(",");
          controls.print(b);
          controls.print(",");
          controls.print(pulseLenght);
          controls.print(",");
          controls.println(protocol);
          
          controls.close();
          displayUpdate = true;
        }
      }
      mySwitch.resetAvailable();
    }
    handleButtons(&selectedSignal);

    if (displayUpdate || selectedSignal != lastSignal) {
      displaySignals(selectedSignal);
      lastSignal = selectedSignal;
      displayUpdate = false;
    }
  }
}

void handleButtons(int *selectedSignal) {
  if (digitalRead(4) == LOW) {
    (*selectedSignal)++;
    displayUpdate = true;
    delay(200);
    } else if (digitalRead(5) == LOW) {
    (*selectedSignal)--; 
    displayUpdate = true;
    delay(200); 
    } else if (digitalRead(6) == LOW) {
    transmitSignal(*selectedSignal);
    delay(200);  
    } else if (digitalRead(7) == LOW) {
    resetSDCard();
    displayUpdate = true;
    delay(200);
    } else if (digitalRead(8) == LOW) {
      lcd.clear();
      lcd.print("Transmissao encerada");
      delay(1000);
      displayUpdate = true;
    }
  }

void displaySignals(int selectedSignal) {
  lcd.clear();
  
  controls = SD.open("data.txt");
  int lineNum = 0;
  while (controls.available() && lineNum < 2) {
    lcd.setCursor(0, lineNum);
    String line = controls.readStringUntil('\n');
    String sign = getValue(line, ',', 0);
    if (lineNum == selectedSignal % 2) {
      lcd.print(">");
    }
    lcd.print(sign);
    lineNum++;
  }
  controls.close();
}

void transmitSignal(int signalIndex) {
  controls = SD.open("data.txt");
  int lineNum = 0;
  while (controls.available()) {
    String line = controls.readStringUntil('\n');
    if (lineNum == signalIndex) {
      lcd.clear();
      lcd.print("Transmitindo sinal: ");

      String sign = getValue(line, ',', 0);
      String binary = getValue(line, ',', 1);
      String pulseLength = getValue(line, ',', 2);
      String protocol = getValue(line, ',', 3);

      const char* b = binary.c_str();
      unsigned long pl = pulseLength.toInt();
      unsigned long p = protocol.toInt();

      mySwitch.setProtocol(p);
      mySwitch.setPulseLength(pl);

      lcd.print(sign);

      while (1) {
        delay(1000);
        if (digitalRead(8) == LOW) {
          lcd.clear();
          lcd.print("Transmissao encerrada.");
          delay(1000);
          break;
        }
        mySwitch.send(b);
        
        delay(2000);
      }

      if (digitalRead(8) == LOW) {
        lcd.clear();
        lcd.print("Transmissao encerrada.");
        delay(1000);
        break;
      }
    } 
    lineNum++;
  }
  controls.close();
 }

void resetSDCard() {
  controls = SD.open("data.txt", FILE_WRITE);
  if (controls) {
    controls.close();
    }
  SD.remove("data.txt");

  lcd.clear();
  lcd.print("Cartao resetado.");
  delay(1000);
  }

// Parsing da string
String getValue(String data, char separator, int index) {
  int found = 0;
  int strIndex[] = { 0, -1 };
  int maxIndex = data.length() - 1;

  for (int i = 0; i <= maxIndex && found <= index; i++) {
    if (data.charAt(i) == separator || i == maxIndex) {
      found++;
      strIndex[0] = strIndex[1] + 1;
      strIndex[1] = (i == maxIndex) ? i+1 : i;
    }
  }
  return found > index ? data.substring(strIndex[0], strIndex[1]) : "";
}

// Converte o valor do sinal e o tamanho em um binario
 static char * dec2bin(unsigned long Dec, unsigned int bitLength) {
  static char bin[64]; 
  unsigned int i=0;

  while (Dec > 0) {
    bin[32+i++] = ((Dec & 1) > 0) ? '1' : '0';
    Dec = Dec >> 1;
  }

  for (unsigned int j = 0; j< bitLength; j++) {
    if (j >= bitLength - i) {
      bin[j] = bin[ 31 + i - (j - (bitLength - i)) ];
    } else {
      bin[j] = '0';
    }
  }
  bin[bitLength] = '\0';
  
  return bin;
}
```

## Teste de funcionalidade

Uma vez com todo o *hardware* montado e o *software* carregado na Arduino Nano, eu carreguei mais uma placa Arduino Nano, com o programa de interceptação de sinal, para verificar se a transmissão estava acontecendo.

O vídeo abaixo (de péssima qualidade por sinal) mostra o funcionamento na *protoboard*.

<video muted controls autoplay loop>
    <source src="/img/posts/poc-proto.mp4" type="video/mp4">
</video>


No monitor serial da segunda Arduino, temos exatamente o mesmo sinal interceptado anteriormente pelo controle remoto.

![Sinal interceptado do dispositivo.](/img/posts/Pasted%20image%2020240424061900.png)

# Montando o Projeto

Para finalmente montar o dispositivo e retirá-lo da fase de protótipo, decidi pela maior simplicidade, praticidade e facilidade possível. Não tenho a disponibilidade nem o material para montar uma PCB (*Printed Circuit Board* ou Placa de Circuito Impresso), portanto decidi pela boa e velha Placa Universal 10x10.

![Placa universal](/img/posts/Pasted%20image%2020240501075415.png)

O trabalho feito com ela não é dos mais bonitos, porém é funcional e atende as expectativas deste projeto.

Outra característica que decidi utilizar nesta montagem, foi a de não soltar os módulos diretamente na placa, optando por utilizar barras de pinos macho e fêmea para poder fazer algo mais "modular" (interprete como, poder retirar um módulo caso precise usar em outro projeto), o que me obrigou a usar alguns *jumpers* no projeto final.

![Barra de pino.](/img/posts/Pasted%20image%2020240501075735.png)

Durante a criação das trilhas, percebi que estou um pouco enferrujado na arte da solta, mas com o passar do tempo fui me recuperando, então não ficou das melhores, mas ainda repito que funcional.

![Montagem das trilhas](/img/posts/Pasted%20image%2020240501080033.png)

No final das contas, o dispositivo ficou apresentável:

![Projeto](/img/posts/Pasted%20image%2020240501080321.png)

# Testando *In the Wild*

Uma vez com o projeto montado, chegou a hora de testar, obviamente é um processo delicado, sair por aí abrindo portões arbitrariamente não é um hábito saudável (vai que um cachorro foge!).

Brincadeiras a parte, o teste ocorreu em ambiente controlado, mais precisamente em minha casa, no meu próprio portão. O teste consistiu em:

1. Desligar a energia do motor do portão, para poder usar meu controle e interceptar o sinal sem abri-lo;
2. Checar se o sinal foi gravado no dispositivo;
3. Religar a energia do motor;
4. Usar o dispositivo para replicar o sinal armazenado.

E o resultado foi esse:

<video muted controls autoplay loop>
    <source src="/img/posts/teste.mp4" type="video/mp4">
</video>


E temos um dispositivo de *replay* de sinal funcional. Fizemos testes em outros ambientes?

**Não saberão**, mas algumas operações de *Red Team* já estão garantidas.


# Conclusão

Este projeto de clonagem de sinais RF para controles de portão exemplifica a engenhosidade e a curiosidade inerentes ao *hacking*. Desde a motivação inicial até o teste prático, exploramos todo o processo de engenharia inversa e replicação de sinais RF, elucidando as vulnerabilidades inerentes dos sistemas de código fixo.

O uso do **Arduino Nano** como núcleo do dispositivo foi uma escolha acertada, dado seu tamanho compacto e versatilidade. A biblioteca **RC-Switch** demonstrou ser uma ferramenta robusta para manipular sinais RF, facilitando a interceptação e transmissão de códigos fixos. A integração de componentes como o *display* LCD, o módulo de leitura de cartão SD e os módulos RF 433 MHz, mostrou que é possível construir um dispositivo funcional com recursos limitados e relativamente acessíveis.

O processo de desenvolvimento revisitou a importância de compreender a tecnologia subjacente, desde a modulação ASK até os protocolos de comunicação. As várias etapas, desde o planejamento até a execução, evidenciaram a complexidade envolvida na engenharia reversa de sinais e na manipulação de *hardware* e *software* de forma integrada.

Este projeto é um lembrete da importância da ética no hacking. Para profissionais de segurança e entusiastas de *hardware hacking*, o dispositivo construído pode servir como uma ferramenta educacional valiosa para demonstrar os riscos e as possibilidades associadas à tecnologia RF.

Em suma, esta jornada revelou não apenas as nuances técnicas do *hacking* de hardware, mas também a necessidade contínua de aprendizado e aprimoramento nas técnicas de segurança para garantir a resiliência contra ameaças emergentes.


# Referências

- [How does Modulation Work?](https://www.taitradioacademy.com/topic/how-does-modulation-work-1-1/)
- [Remote Control Switching System Based on Wireless Technology](https://www.researchgate.net/publication/339444195_Remote_Control_Switching_System_Based_on_Wireless_Technology)
- [Understanding Radio Frequency Identification (RFID) and Its Impact on the Supply Chain](https://www.researchgate.net/publication/249901917_Understanding_Radio_Frequency_Identification_RFID_and_Its_Impact_on_the_Supply_Chain)
- [Guidelines for Securing Radio Frequency Identification (RFID) Systems](https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-98.pdf)
- [The Impact of Radio Frequency (RF) Attacks on Security and Privacy: A Comprehensive Review](https://dl.acm.org/doi/abs/10.1145/3607720.3607771)