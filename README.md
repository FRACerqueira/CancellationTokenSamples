# **CancellationToken - O que voc� precisa e deve saber antes de usar**

No mundo moderno da engenharia de software os processos , atividades e depend�ncias ocorrem a todo o momento de forma **concorrente**! Falar de **concorr�ncia** � ter consci�ncia que v�rias tarefas s�o executadas simultaneamente ou concorrentes com os ciclos/processadores dispon�veis e que estes **recursos s�o finitos e/ou tem custos associados � sua utiliza��o.**

_Obter performance � bem mais sobre saber trabalhar com concorr�ncia e como **usar de forma consciente os recursos** que sobre saber aplicar padr�es arquiteturais._

De um modo geral (minha opini�o) **falhamos miseravelmente em criar aplica��es / bibliotecas e funcionalidades que fazem bom uso de recursos! A "nuvem" at� pode ser "infinita", mas certamente o cart�o de cr�dito e sua infraestrutura N�O!**

Onde entra o **CancellationToken** neste contexto? Ent�o vamos l�..., mas antes vou alinhar, junto com voc�, alguns **conceitos fundamentais que precisam ser compreendidos e aplicados** :

_**CancellationToken** � um componente desenhado para informar que n�o � mais necess�rio continuar uma tarefa, fornecendo um mecanismo para cancelamento cooperativo de opera��es ass�ncrona e propagando a informa��o de solicita��o de cancelamento por todas as tarefas que a utilizam._

**Alguns aspectos importantes:**

#### _O cancelamento **n�o � imposto/for�ado � uma solicita��o!** As tarefas podem e devem determinar como e quando encerrar em resposta a uma solicita��o de cancelamento._

**N�o existe m�gica** , o **CancellationToken** n�o faz nada al�m de informar uma solicita��o de cancelamento, isso � muito importante.

#### _A solicita��o de **cancelamento refere-se sempre a opera��es e n�o a objetos** , em outras palavras, devemos criar "opera��es cancel�veis" para poder tirar proveito do CancellationToken._

**N�o existe m�gica** , o **CancellationToken** informa, e voc� precisa e deve saber o que fazer com esta solicita��o, isso � ainda mais importante!

#### _**opera��es cancel�veis** S�o tarefa que escolhem uma **estrat�gia de como encerrar e como responder** a uma solicita��o de cancelamento. Normalmente, executam alguma limpeza necess�ria e respondem o mais **breve poss�vel**._

**N�o existe m�gica** , o que precisa ser feito ap�s o recebimento da solicita��o de cancelamento deve estar bem definido e alinhado com as regras de neg�cio e os componentes utilizados, al�m disso a implementa��o **devem ter suporte para este tipo de opera��o**. E isso � mais que importante � fundamental!

_Como sei que foi enviado solicita��o de cancelamento? Atrav�s da propriedade **IsCancellationRequested.**_

**Notas antes de baixar:**

- Este repo mostra apenas o **CancellationToken,** o uso de tarefas ass�ncronas (Async / await) n�o est�o dentro deste escopo
- Todos os exemplos apresentados s�o **apenas did�ticos e necessariamente n�o representam as melhores pr�tica**. 

## Tornando sua Api �Cancel�vel�

Quando uma solicita��o de �request� � enviada para sua api, � mantida uma conex�o entre as duas partes envolvidas (quem est� consumindo / quem est� processando a solicita��o). 
O que acontece se a parte que solicitou abandonar a solicita��o? A conex�o � perdida e o processamento continua na sua api! (Desperd�cio de processamento , recursos de Infraestrutura/Midllewares e os custos associados financeiros e de performance).  

Toda vez que � estabelecida uma conex�o, �por baixo dos panos� � disponibilizado um par�metro de CancellationToken (opcional � deve ser declarado) em sua solicita��o associado � esta conex�o. 

Quando ocorre uma �quebra� de conex�o entre as partes � enviado uma solicita��o de cancelamento (**IsCancellationRequested = true**) para que voc� tenha a oportunidade de tomar uma a��o para n�o continuar a opera��o, uma vez que a outra parte n�o tem mais interesse no resultado da opera��o.

_**Dica : Sempre declarar e propagar o CancellationToken para as chamadas subjacentes, oferecendo a oportunidade as demais depend�ncias de tomar uma a��o para n�o continuar a opera��o o mais breve poss�vel.**_

O .NET j� fornece diversos m�todos associados a cada recurso que j� est�o preparados para cancelar a opera��o caso recebam uma notifica��o de cancelamento.

**Lembre-se: A solicita��o de cancelamento refere-se sempre a opera��es e n�o a objetos, ent�o o fato de seu recurso(objeto) de api ou tarefa/fun��o ser ou n�o �Async� � irrelevante!**  

**ATEN��O:** Normalmente os m�todos associados a cada recurso que aceitam o cancelamento esperam uma opera��o ass�ncrona criando esta rela��o com �Async/await�, ent�o ao executar uma opera��o ass�ncrona sobre uma opera��o s�ncrona, embora seja poss�vel, deve ser feita com muito crit�rio! 

**Exemplo: Acessando outra depend�ncia de api (Tarefa n�o cancel�vel) e sem token:**

Para testar use a aplica��o �**CancellationTokenApiSample**� e chame o �Endpoint� �Call/HelloWord� , em seguida execute um refresh no browser; esta a��o interrompe a conex�o e ativa a solicita��o de cancelamento. 

Neste exemplo � feito uma chamada a outro �Endpoint� (que demora 10 segundos para recuperar a informa��o) . Como n�o tem a declara��o de cancelamento e a depend�ncia tamb�m n�o � uma tarefa cancel�vel a execu��o continua at� o final na api e na depend�ncia sem ser interrompida. (Veja os tempos)

**Exemplo: Acessando outra depend�ncia de api (Tarefa n�o cancel�vel) e com CancellationToken:**

Para testar use a aplica��o **�CancellationTokenApiSample�** e chame o �Endpoint�  �CallCT /HelloWord� em seguida execute um refresh no browser; esta a��o interrompe a conex�o e ativa a solicita��o de cancelamento. 

Neste exemplo � feito uma chamada a outro �Endpoint� (que demora 10 segundos para recuperar a informa��o) . Como tem a declara��o de cancelamento  e a depend�ncia n�o � uma tarefa cancel�vel a execu��o continua at� o final na depend�ncia sem ser interrompida e � interrompida na api. (Veja os tempos)

**Exemplo: Acessando outra depend�ncia de api (Tarefa cancel�vel) e com CancellationToken:**

Para testar use a aplica��o **�CancellationTokenApiSample�** e chame o �Endpoint�  �CallCT / HelloWordCancelation� em seguida execute um refresh no browser; esta a��o interrompe a conex�o e ativa a solicita��o de cancelamento. 

Neste exemplo e feito uma chamada a outro �Endpoint� (que demora 10 segundos para recuperar a informa��o). Como tem a declara��o de cancelamento  e a depend�ncia � uma tarefa cancel�vel a execu��o � interrompida na api e na depend�ncia. (Veja os tempos)

**Exemplo: Acessando um banco de dados (Tarefa n�o cancel�vel) e sem CancellationToken:**

Para testar use a aplica��o **�CancellationTokenApiSample�** e chame o �Endpoint�  �Call/OpenDb� em seguida execute um refresh no browser; esta a��o interrompe a conex�o e ativa a solicita��o de cancelamento. 

Neste exemplo e feito um acesso ao um banco de dados abrindo e fechando a conex�o. Para simular um atraso, a conex�o aponta para um endere�o �errado� para tentar 6 vezes.  Quando ocorre o a solicita��o de cancelamento a consulta ao banco continua .(veja o tempo)

**Exemplo: Acessando um banco de dados (Tarefa cancel�vel) e com CancellationToken:**

Para testar use a aplica��o �CancellationTokenApiSample� e chame o �Endpoint�  �CallCT/OpenDb� em seguida execute um refresh no browser; esta a��o interrompe a conex�o e ativa a solicita��o de cancelamento. 

Neste exemplo e feito um acesso ao um banco de dados abrindo e fechando a conex�o. Para simular um atraso a conex�o aponta para um endere�o �errado� para tentar 6 vezes.  Quando ocorre o a solicita��o de cancelamento a consulta ao banco � interrompida .(veja o tempo)

## Tornando uma Tarefa/Fun��o �Cancel�vel�
Nos exemplos anteriores � feito chamadas a dois �Endpoint� diferentes 
- /HelloWord (tarefa **n�o** cancel�vel)
- /HelloWordCancelation (tarefa cancel�vel)

Comparando os dois c�digos, voc� pode notar que existem algumas diferen�as:
- Recebe um par�metro CancellationToken (Propaga��o! lembra da dica no in�cio?)
- Aproveita o Token para executar o Delay:  �token.WaitHandle.WaitOne(10000);� (Vamos falar disso agora)

O c�digo original usa o comando �Thread.Sleep(10000)�. Este comando interrompe a execu��o Thread por um tempo determinado para simular um trabalhado demorado/ custoso de um recurso.  

Queremos que  isso aconte�a, por�m somente quando n�o houver uma solicita��o de cancelamento, ent�o precisamos de uma outra estrat�gia para fazer isso e responder o mais breve poss�vel quando ocorre o cancelamento.

A propriedade **�WaitHandle�** disponibiliza o manipulador do token que � respons�vel por controlar a sincroniza��o entre threads e aplicar os sinalizadores (sem�foros) para que isso aconte�a. 

O Legal da  propriedade **�WaitHandle�**  � que possui alguns m�todos para interromper a execu��o at� receber um sinal e isso pode ser um par�metro de tempo. Quando ocorre uma solicita��o de cancelamento esta **temporiza��o � interrompida** e �voil��!  temos agora uma estrat�gia eficiente e r�pida!

**Dica : Utilize sempre que poss�vel WaitHandle  e seus m�todos como  sinalizadores de tempo(WaitOne ou seus semelhantes).  Quando ocorre uma solicita��o de cancelamento seus m�todos tamb�m s�o avisados e interrompem a espera/a��o programada**

## Tornando uma tarefa em background �Cancel�vel� em um host web

Uma tarefa em �background� normalmente � iniciada junto com aplica��o e permanece ativa durante o ciclo de vida dela, sobre um �loop� baseado em uma condi��o/regra qualquer. 

Em cen�rios atuais, onde �container � vida� e os conceitos de **�HPA"  (�upscale� / �downscale�) s�o necess�rios para fazem uso consciente de recursos** as tarefas em �background� precisam saber a hora de parar de forma controlada evitando �travamentos� e fazendo as limpezas necess�rias durante este processo. Isso � v�lido mesmo sem o uso de �container�.

O uso de CancellationToken � um bom �aliado� para garantir uma sa�da deste �loop� de uma forma controlada (�Graceful Shutdown�). 

**Exemplo: Tarefa em background (Tarefa cancel�vel) e com CancellationToken:**

Para testar use a aplica��o **�CancellationTokenBackGroudServiceSample�** e acompanhe o log por cerca de 30 segundos e depois chame o �Endpoint� **�StopApplication�** ; esta a��o envia uma solicita��o para finalizar a aplica��o que por sua vez dispara uma solicita��o de cancelamento. 

Analisando o c�digo, voc� poder� ver um loop temporizado pela classe **�PeriodicTimer�**. Antes de executar a a��o desejada � feita uma valida��o de neg�cio �AnyRule� que quando atendida executa uma **tarefa cancel�vel �DoWork�**.  Os resultados podem ser vistos e avaliados pelo log saindo de forma controlada (Graceful Shutdown).

Uma boa pr�tica observada ,como j� dito, � a propaga��o do CancellationToken para as depend�ncias subjacentes , que no nosso caso � o m�todo �DoWork�. 

Outra boa pr�tica � o uso do comando �ThrowIfCancellationRequested()� que notifica ao chamador uma solicita��o de cancelamento abortando a execu��o. 

**TrowIfCancellationRequested** : Gera um OperationCanceledException se esse token tiver tido o cancelamento solicitado. � equivalente ao c�digo abaixo :
```
if (token.IsCancellationRequested)   
    throw new OperationCanceledException(token);
```

## Assumindo o controle do �Cancelamento�

Sempre temos dispon�vel pela Infraestrutura do .NET o CancellationToken de encerramento da aplica��o, por�m  outras  necessidades aparecem em cen�rios que queremos cancelar alguma tarefa com base em um evento, estado ou regra de neg�cio. Sendo assim precisamos assumir o controle! Nesta hora entra em cena outro componente: 

**CancellationTokenSource**:  � uma classe desenhada para ser uma fonte de notifica��o de cancelamento controlada pela aplica��o. 

Adivinha qual � a principal propriedade desta classe? O CancellationToken!

_**Nota: Esta classe implementa a interface IDisposable ent�o lembre-se de utilizar o m�todo Dispose() quando n�o for mais necess�rio ou encapsular sua utiliza��o em um bloco de �using�.**_

O CancellationTokenSource possui alguns m�todos para atender ao seu prop�sito e vamos abordar alguns deles (Existem v�rios outros m�todos, abordaremos adiante) 

- Cancel 
- CancelAfter
- Register

### O m�todo Cancel

O que faz o m�todo Cancel? Comunica uma solicita��o de cancelamento, em outras palavras, tornam a propriedade **IsCancellationRequested = true**!

**Exemplo: CancellationTokenSource � M�todo Cancel** 

Para testar use a aplica��o **�CancellationTokenCancelConsole�** , siga as instru��es que apareceram no console e acompanhe o log. 

Neste exemplo temos um servi�o em background que inicia duas tarefas ass�ncronas: 
- TaskUI: Respons�vel  pela intera��o do console para enviar um comando de processar mensagens  ou uma solicita��o de cancelamento e encerrar a aplica��o. 
- TaskProcess: Respons�vel  por processar uma mensagem. 

Analisando o c�digo , voc� pode observar o uso do CancellationTokenSource com o m�todo Cancel  na �TaskUI� para enviar uma solicita��o de cancelamento para Task �TaskProcess�. 

Existe um problema nesta abordagem: Quando ocorre um **Ctrl-C** a Task �TaskProcess� n�o � encerrada automaticamente pois ele apenas observa o **CancellationTokenSource** sendo preciso enviar outro **Cancel** durante o m�todo **�StopAsync�** e aguarda o t�rmino da Task para termos uma sa�da controlada (Graceful Shutdown) e um fluxo correto de sua l�gica(executar o comando ap�s o loop) .

### O m�todo CancelAfter

O que faz o m�todo CancelAfter(TimeSpan)/ CancelAfter(Int32)? Comunica uma solicita��o de cancelamento ap�s o n�mero especificado de tempo. 

Um exemplo pr�tico para este m�todo � o uso em regras de **TIMEOUT** para execu��es de **qualquer** atividade.

**Exemplo: CancellationTokenSource � M�todo CancelAfter**

Para testar use a aplica��o **�CancellationTokenCancelAfterConsole�** siga as instru��es que apareceram no console e acompanhe o log. 

Neste exemplo temos um servi�o em background que inicia uma tarefa ass�ncrona que chama outra tarefa cancel�vel para processar a mensagem com um �flag� para ficar em espera  al�m do tempo m�ximo definido ,demostrando um controle de timeout (ap�s um tempo pr�-definido a tarefa � cancelada). 

Observe que o par�metro de CancellationToken � um CancellationTokenSource que foi usado o m�todo CancelAfter(10000) antes da chamada.

### O m�todo Register

O que faz o m�todo Register()? Registra um �action� que ser� chamado quando este CancellationToken for cancelado. Se este token j� estiver no estado cancelado, a action ser� executado imediatamente e de forma s�ncrona. Qualquer exce��o gerada pelo action ser� propagada a partir desta chamada de m�todo.

**Exemplo: CancellationTokenSource � M�todo Register** 

Para testar use a aplica��o **�CancellationTokenRegisterConsole�** siga as instru��es que apareceram no console e acompanhe o log. 

Neste exemplo temos um servi�o em background que inicia uma tarefa ass�ncrona que processa a mensagem. Antes de iniciar o �loop�  � feito um registro de uma �action� para ser executada  quando ocorrer o cancelamento. Esta �action� executa o fim da aplica��o que por sua vez envia uma solicita��o de cancelamento e encerra o �loop�.

## Tornando uma tarefa �Cancel�vel� associada a uma ou mais regras de neg�cio / solicita��es de cancelamentos

No mundo real nem tudo � t�o simples e muita das vezes precisamos implementar regras mais complexas que requer **mais de uma �nica fonte de cancelamento**. Vamos a um cen�rio real:

_Voc� possui um m�todo que j� recebe um CancellationToken vindo de um �request� . Isso � �timo , voc� pode controlar quando a conex�o � perdida e interromper o processo. 
Agora imagina que este m�todo al�m de interromper quando a conex�o � perdida interrompa tamb�m quando ultrapasse um tempo limite ou ainda pior quando uma regra de neg�cio n�o seja atendida durante seu processamento, o que acontecer primeiro. �Sacou o drama ?�, nesta hora entra em cena outra classe :_


**CreateLinkedToken**:  � uma classe desenhada para ser uma **fonte de notifica��o de cancelamento** controlado pela aplica��o que **unifica v�rios tokens de cancelamento em uma �nica solicita��o de cancelamento**. 

_**Nota: Esta classe implementa a interface IDisposable ent�o lembre-se de utilizar o m�todo Dispose() quando n�o for mais necess�rio ou encapsular sua utiliza��o em um bloco de �using�.**_

**Exemplo: Acessando um Endpoint(Tarefa cancel�vel com CreateLinkedToken):** 

O mais legal desta classe � que al�m de simplificar a codifica��o (basta verificar a propriedade IsCancellationRequested  independente que que a originou) � que ainda � poss�vel saber qual token de cancelamento foi a origem (caso seja preciso tomar a��es diferentes para tokens distintos).

Para testar use a aplica��o **�CancellationTokenApiCreateLinkedToken�** e chame o �Endpoint�  �Call/HelloWordWithTimeout?timeout=true�  e aguarde o resultado (ser� um timeout com status 499  em seguida chame o mesmo �Endpoint�   e execute um refresh no browser; esta a��o interrompe a conex�o e ativa a solicita��o de cancelamento. 

Neste exemplo � feito uma chamada a outro �Endpoint� (que demora 10 segundos para recuperar a informa��o ) . antes da execu��o  � criando um CancellationTokenSource  e um CreateLinkedTokenSource  utilizando o CancellationToken do request e o CancellationTokenSource.Token.  � simulado uma espera maior de que o tempo m�ximo definido para demostrar o  timeout. 

Observe que o c�digo faz isso utilizando CreateLinkedTokenSource!  Isso significa que caso ocorra um cancelamento pelo cliente ou o timeout a tarefa ser� cancelada! (Veja os logs)

## Tornando os retornos de um API �Cancel�vel� rastre�veis

Para finalizar quero de outro aspecto fundamental e muito relevante que propositalmente n�o citei no in�cio:

#### Mantenha sempre os retornos coerentes de sua API quando ocorre um cancelamento por parte do cliente. 

Mas porque isso � importante se foi dito que quando ocorreu cancelamento o cliente n�o tem mais interesse , e n�o recebe o resultado ?

Lembra que tamb�m falei tamb�m que minha opini�o falhamos miseravelmente em criar aplica��es / bibliotecas e funcionalidades que fazem bom uso de recursos? 

N�o � apenas cancelar e se preocupar em performance e custo � **fazer bom uso de recursos existentes tamb�m!** vamos detalhar isso melhor criando um cen�rio:

_Voc� trabalha em uma empresa de com�rcio eletr�nico que possui um gateway interno para fazer a comunica��es entre as diversas apis e al�m de rotear estas comunica��es � gerado um log de infraestrutura com os tempos e status de cada requisi��o para uma an�lise pelo time de produtos e estrat�gia da empresa.  Perda de impulso de compras � um indicador relevante, e talvez somente  o tempo de espera das requisi��es n�o seja o �nico meio para isso . Quando ocorre um cancelamento por parte do cliente se voc� **n�o escolher um status correto as m�tricas de status ficar�o �polu�das�** mesmo que funcionalmente n�o tenha erros._

A aplica��o **�CancellationTokenApiCreateLinkedToken�** (**SampleController.cs**) faz este tratamento retornando um status n�o padr�o (499 � Mesmo do nginx) . 

_**Nota : Para verificar que isso acontece de fato � necess�rio colocar um breakpoint e acompanhar a execu��o.**_

## Conclus�o
**CancellationToken** � um instrumento poderoso para otimizar o uso de recursos , tornando sua aplica��o muito mais resiliente e r�pida, aproveita corretamente os recursos de Infraestrutura/Midllewares. 

Existem muitos outros cen�rios (e m�todos) que podem ser explorados , mas tornaria bem mais extenso este artigo que o desejado. Espero que com este �overview� e com os c�digos disponibilizados no projeto exemplo,  voc� possa aproveitar melhor o CancellationToken!


_**Fernando Cerequeira**_

## Refer�ncias

**CancellationToken** :
https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken?view=net-8.0

**CancellationTokenSource** : 
https://learn.microsoft.com/pt-br/dotnet/api/system.threading.cancellationtokensource?view=net-8.0

