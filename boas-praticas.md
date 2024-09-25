# Organização de aplicações Salesforce

*Faaaaaala Lendas! Este documento é um guia sugerido de boas práticas para desenvolvimento em Salesforce. Fique à vontade parar sugerir e criticar o conteúdo. Só bora!*

<br>

## Pacotes 📦
Sempre que possível, devemos priorizar o desenvolvimento em formato de Apps instaláveis no lugar de um emaranhado de customizações espalhadas na org. Para isso, iremos utilizar:

|Recurso|Descrição|
|-|-|
|Unlocked Packages|Para orgs novas, onde não há desenvolvimentos existentes, pode-se criar este tipo de pacote e trabalhar com **scratch orgs**. É possível quebrar a aplicação em vários pacotes e criar dependências entre eles.|
|Unlocked Packages - Org Dependent|Em orgs onde já há desenvolvimentos anteriores, usamos este tipo de pacote. Pode-se quebrar a aplicação em vários pacotes, porém todos vão apontar suas dependências para a org de destino onde será instalado. Para este tipo de pacote é necessário trabalhar com **sandboxes**.|
|Permission Sets|Ao trabalhar com pacotes, deve-se ter em mente que **Profiles** não são empacotáveis. Todo pacote deve conter seu próprio **Permission Set** e as permissões dos metadados atribuídas através dele.|
|GitHub|Cada desenvolvedor terá seu próprio ambiente de desenvolvimento, seja scratch org ou sandbox. Todo metadado gerado, será versionado com a ajuda do **SFDX Source Tracking** através do VSCode. A partir do repositório do Git serão geradas novas versões dos aplicativos.|

<br>

### Setup do ambiente de desenvolvimento 💻
1. Criar conta [GitHub](https://github.com/) com seu e-mail `@platformbuilders.io`
2. Download [Git](https://git-scm.com/downloads)
3. Download client de Git de sua preferência, exemplos:
	- [Fork](https://git-fork.com/)
	- [GitHub Desktop](https://desktop.github.com/)
	- [Sourcetree](https://www.sourcetreeapp.com/)
	- [TortoiseGit](https://tortoisegit.org/download/) *Windows Only
4. Download [Salesforce CLI](https://developer.salesforce.com/tools/sfdxcli)
5. Download [Java SDK](https://www.oracle.com/java/technologies/downloads/)
6. Download [VSCode](https://code.visualstudio.com/download)
   - Instalar [extensões](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode) do Salesforce
   - Configurar o path do Java SDK, abrindo o atalho `Preferences: Open User Settings (JSON)`, exemplo:
     -  ``` "salesforcedx-vscode-apex.java.home" : "C:/Program Files/Java/jdk1.8.0_281"```

<br>

O fluxo de trabalho ficará mais ou menos assim:

- Desenvolvedor/Analista baixa o projeto do git para seu computador
- Abre o Visual Studio Code com os plugins do SF instalados
- Conecta na sua sandbox pessoal (ex: sandbox Mago01)
- Faz o sfDeploy do conteúdo que baixou do git para sua sandbox
- Executa suas tarefas, criando códigos ou parametrizações, sincronizando o computador local com sfDeploy e sfRetrieve
- Faz o git commit e git push
- Outros magos podem baixar estas alterações e repetir o ciclo acima.


<br>

### Módulos 📁
- O desenvolvimento de um projeto pode ser quebrado em diferentes pacotes, isolando domínios de negócio para melhor organização e futuras manutenções.
- Dentro de cada pacote, é possível organizar os metadados de um subtema em pastas, representando módulos daquele pacote.

Exemplo de organização de pacotes e módulos:
```
 📦 imoveis-app
	-📂 aluguel
		-📁 classes
		-📁 lwc
	-📂 venda
		-📁 triggers
	-📂 estoque
		-📁 layouts
 📦 financiamento-app
	-📂 calculo
		-📁 classes
		-📁 flows
		-📁 objects
	-📂 taxas
		-📁 classes
		-📁 layouts
	-📂 parcelas
		-📁 flexipages
```

# Boas práticas de código em Apex
O Pattern que nós utilizamos, se trata de uma estrutura em camadas, porém tendo como base o 'DDD'.
Ainda que tenha como base o DDD, poderia muito bem ser aplicado para 'Clean Architecture' sem nenhum problema.

Abaixo segue um resumo de como separamos cada camada:

|Camada|Descrição|
|---|---|
|Camada de Interfarce com o usuário|Recebe comandos do usuário e também exibe informaçōes ao mesmo|
|Camada de Aplicaçāo|Realiza o gerenciamento de transaçōes, coordena atividades de uma aplicaçāo, executa comandos do usuário e acessa dominios. Aqui nāo deve ter regras negócio. Esta camada é responsável apenas por orquestrar o comandos necessários para uma transaçāo|
|Camada de Domínio|Responsável pelas regras de negócio e objetos de negócio. Contém estado e comportamento de um domínio|
|Camada de Infraestrutura|Responsável por expor itens de caráter mais técnico como acesso a banco dados e Web Services. Adaptadores e Frameworks, também deverāo fazer parte desta camada. Esta camada deverá suportar as outras|

A tabela acima explica de uma forma genérica o pattern utilizado. Porém, como nós implementamos?
Abaixo segue como usamos este pattern:

|Camada|Classe|
|---|---|
|Camada de Interfarce com o usuário|LWC, AURA, Visualforce, Apex Controller, Apex REST|
|Camada de Aplicaçāo|Schema, Service|
|Camada de Domínio|Classe que normalmente recebe o nome do módulo, representando uma agregaçāo|
|Camada de Infraestrutura|Repository, Factory|

Agora, vamos entender no detalhe o que seria cada classe dessas que representam as camadas.

## Tipos de classes presentes nos módulos
#### Controller
- Recebe os comandos de um usuário através de um LWC, AURA ou Visualforce. Uma vez o comando interpretado e o resultado retornado pelas camadas inferiores, este resultado é retornado ao usuário.
#### REST
- Responsável por expor transações e dados no Salesforce via API. Funcionalidade e funcionamento semelhante ao Controller, porém aqui o público alvo vai ser um ambiente externo.
#### Schema
- Classe estrutural, sem comportamento. Utilizada para gerar a estrutura de dados de uma agregaçāo como um todo. Mesma ideia de uma TO ou DTO.
#### Service
- Responsável por receber comandos transpassados por Controllers, RESTs e TriggerHandlers. Os seus metodos nāo devem ter regras de negócio e sim orquestrar todas as chamadas de negócio entre outras chamadas necessárias para se concluir um processo ou comando.
Nāo entender como algo relacionado a 'Web Service'. Aqui 'Service', é no sentido de executar uma açāo de valor em um módulo. Normalmente funciona como orquestrador de processos, executando uma ou varias execuçōes de agregaçāo para no fim realizar um retorno unificado.
#### Negócio/Agregaçāo
- Onde se concentram as regras de negócio. Deve ser escrita em um formato bem previsível, ou seja com paramêtros de entrada e saída, evitando métodos void e sem paramêtros de entrada. O nome desse tipo de classe, vai ser o nome do próprio módulo, nāo tendo assim um sulfixo como as demais classes.

*Curiosidade: As agregaçōes deveriam ser agrupamentos de Entidades (Entity). Essas classes seriam muito parecidas com as classes "BOs". Como até entāo nāo houve necessidade de abstrair até a este ponto, nāo estamos considerando 'Entity'.*
#### Repository
- Camada de acesso à dados.
Nāo importa se o dado esteja em um objeto, metadado ou em uma API externa. Esta classe, será responsável por buscar os dados e retorná-los já transformados em alguma estrutura (se haver esta necessidade).

*Importante: no caso de DML em objetos do Salesforce, isto é feito na Agregaçāo para o nosso tipo de implementaçāo. Nāo é regra do pattern, mas sim uma forma de facilitar o uso.*
#### Factory
- Camada de transformaçāo de dados. Qualquer tipo de transformaçāo seja de um objeto para Schema ou vice-versa, deve ser feito nesta classe.

*Importante: apesar do nome, nāo confudir com o Design Pattern chamado 'Factory Method'. Apesar do nome igual, sāo coisas distintas. Para este pattern, limita-se a ser uma classe de tranformaçāo. Pode ser chamada por um Repository, Service e em alguns casos pela Agregaçāo se muito necessário.*

## Exemplo de utilizaçāo do pattern
Agora que já entendemos o Pattern utilizado com detalhes, abaixo vamos ver exemplos tendo como premissa um caso de estudo.

Vamos imaginar que temos um módulo responsável por agregar as funcionalidades total de um processo de pedido. Este módulo é responsável por consultar, criar, faturar e cancelar pedidos. No nosso exemplo, vamos implementar apenas o processo (comando) de 'cancelamento de pedido'. Para isso, vamos abstrair que temos uma 'Quick Action' chamada 'Cancelar pedido', que por usa vez chamará um LWC e assim por diante.

```java
// Controller utilizado pelo LWC. Contém tratamentos de erro e faz a chamada a classe de 'Service'
public class PedidoController {
   @AuraEnabled(Cacheable=false)
    public static String cancelarPedido(String pedidoId) {
        PedidoService service = new PedidoService();

        try {
            PedidoSchema.RetornoTransacao resultado = service.cancelarPedido(pedidoId);

            return JSON.serialize(resultado);
        } catch (Exception ex) {
            throw new AuraHandledException('Erro cancelar pedido: ' +  ex.getMessage());
        }
   }
}
```

```java
//  Service contendo toda a orquestaçāo necessária para executar o processo de cancelamento de pedido.
public class PedidoService {
    //Executa processo/fluxo necessário para se completar a açāo (comando) de cancelamento de pedido
    public PedidoSchema.RetornoTransacao cancelarPedido(Id pedidoId) {
        PedidoSchema.RetornoTransacao retorno;
        Pedido pedido = new Pedido();
        PedidoRepository repository = new PedidoRepository();
        PedidoFactory factory = new PedidoFactory();

        List<Fatura__c> faturas = repository.buscarFaturas(pedidoId);
        Order order = repository.buscarPedido(pedidoId);
        Boolean pedidoCancelado = pedido.cancelarPedido(order, faturas);

        retorno = factory.gerarRetornoCancelamento(pedidoCancelado);

        return retorno;
    }
}
```

```java
// Estrutura de dados para o módulo de Pedidos.
public class PedidoSchema {
    public class Pedido {
        public String codigo {get; set;}
        public String produto {get; set;}
        public String usuarioId {get; set;}
        public Decima valor {get; set;}
    }

    public class RetornoTransacao {
        public Boolean sucesso {get; set;}
        public String mensagem {get; set;}
    }
}
```

```java
// Agregaçāo do módulo de Pedido. No nosso exemplo, para cancelar um pedido precisa obedecer a uma regra de negócio.
public class Pedido {
    public Boolean cancelarPedido(Order pedido, List<Fatura__c> faturas) {
        Boolean pedidoFaturado = false;

        for(Fatura__c fatura : faturas){
            if(fatura.Pedido__c == pedido.Id && fatura.Status__ = 'Faturado') {
                pedidoFaturado = true;
                break;
            }
        }

        if(!pedidoFaturado) {
            pedido.Status = 'Cancelado';
            update pedido;
        }
        else {
            throw new PedidoException('Este pedido nāo pode ser cancelado, pois já está faturado.');
        }
    }

    public Boolean criarPedido(PedidoSchema.Pedido pedido) {
        //TODO:
    }

    public class PedidoException extends Exception {}
}
```

```java
//Repository do módulo de Pedidos. Contém todos os retornos de dados dos objetos necessário. Sejam eles SObject ou nāo.
public class PedidoRepository {
    public List<Fatura__c> buscarFaturas(Id pedidoId){
        return [
            SELECT
                Id,
                Pedido__c,
                Status__c
            FROM
                Fatura__c
            WHERE
                Pedido__c =: pedidoId
        ];
    }

    public Order buscarPedido(Id pedidoId){
        return [
            SELECT
                Id,
                Status
            FROM
                Order
            WHERE
                Id =: pedidoId
        ];
    }

    public List<PedidoSchema.Pedido> buscarPedidosUsuario(Id usuarioId) {
        //IMPORTANTE:

        //Poderia ser usado como busca para um componente, usando um Schema como retorno.
        //Neste caso, poderia ser buscado os dados no Salesforce e transformado via Factory aqui mesmo nesta chamada
        return new List<PedidoSchema.Pedido>();
    }
}

```

```java
//Factory tendo como exemplo a geraçāo de uma estrura de retorno para a chamada.
public class PedidoFactory {
    public PedidoSchema.RetornoTransacao gerarRetornoCancelamento(Boolean pedidoCancelado) {
        PedidoSchema.RetornoTransacao retorno = new PedidoSchema.RetornoTransacao();
        retorno.sucesso = pedidoCancelado;
        retorno.mensagem = pedidoCancelado ? 'Pedido cancelado com sucesso' : 'Pedido nāo cancelado';

        return retorno;
    }
}
```

<br>

## Recomendações gerais
<br>

### Escreva códigos auto descritivos
 - Reflita bem antes de definir o nome das variáveis, métodos e classes.
 - Quanto mais claro e bem pensado essas nomenclaturas, menor a necessidade de escrever comentários.
 - Divida as rotinas em quantos métodos forem necessários.
   - Cada método deve ter um objetivo único.
   - Se um método está ficando grande demais, provavelmente está executando mais de um objetivo.
 - Evite comentários desnecessários ou redundantes quando o código já for claro o suficiente.
 - Métodos e variáveis devem ser no padrão `lowerCamelCase`.
 - Constantes devem usar `UPPERCASE` com palavras separadas por underlines, exemplos: `MINIMUM_SIZE`, `SALES_MAX_QUANTITY`

<br>

### Não use notação húngara para nomear variáveis

Prática comum no passado que atualmente não é recomendada. Prefira escrever nomes de variáveis claros o suficiente para que seja possível entender do que se trata sem utilização de prefixos.

```java
// Evite ❌
String strMensagem = '';
Boolean blnAtivo = false;
Decimal decSalario = 1000.20;
List<String> lstNome = new List<String>();
private String fcnSalvarDetalhes() { return ''; }
public interface iAuthProvider {}
```
```java
//Prefira ✅
String mensagem = '';
Boolean ativo = false;
Decimal salario = 1000.20;
List<String> nomes = new List<String>();
private String salvarDetalhes() { return ''; }
public interface AuthProvider {}
```

Evite ❌
|prefix|where to use|
|-|-|
|v|verb|
|adj|adjectives|
|n|noun|

```
💩 vUsing adjHungarian nNotation vMakes nReading nCode adjDifficult.
```

<br>


### Defina variáveis para condições complexas
Procure sempre deixar o código o mais claro possível, em condições complexas, você pode usar a própria variável `Boolean` como explicação do que está sendo testado:
```java
// Evite ❌
if ((!conta.ParentId != null && conta.Segmento__c != oldConta.Segmento__c) || (conta.TabelaTaxaVigente__c != oldConta.TabelaTaxaVigente__c) && oldConta.TabelaTaxaVigente__c != null) {
	contasProcessar.add(conta);
}
```
```java
// Prefira ✅
Boolean contaPai = conta.ParentId != null;
Boolean trocouSegmento = conta.Segmento__c != oldConta.Segmento__c;
Boolean trocouTabelaVigente = conta.TabelaTaxaVigente__c != oldConta.TabelaTaxaVigente__c && oldConta.TabelaTaxaVigente__c != null;

if ((contaPai && trocouSegmento) || trocouTabelaVigente) {
	contasProcessar.add(conta);
}
```

<br>

### Métodos no infinitivo
O nome do método deve utilizar um verbo no infinitivo, indicando uma ação:
```java
// Evite ❌
public void copiaArquivo() {}
public void listaRegistros() {}
```
```java
//Prefira ✅
public void copiarArquivo() {}
public void listarRegistros() {}
```
<br>

### Métodos devem conter poucos parâmetros
Se seu método está recebendo mais de 3 parâmetros, considere criar uma estrutura:
```java
// Evite ❌
public void salvarPessoa(String nome, String sobrenome, Date nascimento, Decimal altura, Decimal peso) {}
```
```java
//Prefira ✅
public void salvarPessoa(Pessoa pessoa) {}
```

```java
public class Pessoa {
	public String nome;
	public String sobrenome;
	public Date nascimento;
	public Decimal altura;
	public Decimal peso
}
```

<br>

### Regras para nome de classes

|Sufixo|Descrição|Exemplo|
|---|---|---|
|REST|Endpoints Rest|Usuario**REST**|
|Controller|Controller|Usuario**Controller**|
|Batch|Batch|ProcessarNotas**Batch**|
|Trigger|Trigger|Account**Trigger**|
|TriggerHandler|Handler para triggers|Account**TriggerHandler**|
|WorkRun|Handler para WorkControls|AtualizarHierarquia**WorkRun**|

<br>

### Nunca colocar regra de negocio em Controller, Rest, Batch, Scheduler, WorkRun
Essas classes devem ser apenas a ponte que liga uma ação ao processo em si. Elas devem conter apenas códigos referentes aos detalhes daquele contexto. Toda execução de regras de negócio devem estar em classes separadas.

Exemplo usando uma classe REST:
```java
@RestResource(urlMapping='/v1/exemplo-servico/*')
global class ExemploServicoREST {
	@HttpPost
	global static void processarRequest(){
		RestRequest request = RestContext.request;
		RestResponse response = RestContext.response;

		String originalBody = request.requestBody.toString();

		// Chamada para regras de negócio aqui
		ClasseNegocio.executarRegra(originalBody);

		response.statusCode = 200;
		response.responseBody = Blob.valueOf(JSON.serialize('Recebido com sucesso!'));
		response.addHeader('Content-Type', 'application/json');
	}
}
```

<br>

### Classes de Teste
Todo código escrito em Apex deve ser testado. Tente escrever os testes o quanto antes, pois eles podem te ajudar a garantir que a funcionalidade não seja quebrada ao fazer um refactoring durante a construção do programa.

<br>

Utilize o padrão A.A.A:
- **A**rrange
  - Prepare todos os dados necessários para iniciar o teste. Dados que são comuns para vários testes na mesma classe, devem ser colocados no método `@TestSetup`
  - Utilize um TestFactory para facilitar a geração dos dados. Nossa recomendação atual: https://github.com/dhoechst/Salesforce-Test-Factory
- **A**ct
  - Execute as ações entre `Test.startTest()` e `Test.stopTest();`
  - Ao testar uma trigger de INSERT, provavelmente você não terá o passo Act, pois no passo Arrange, já serão disparadas as triggers ao criar os registros.
- **A**ssert
  - Valide a execução utilizando o `System.AssertEquals()` ou `System.Assert()`.

<br>

💡 Evite ir e voltar nesses 3 passos. Se sentir necessidade de fazer novos **Act** após um **Assert**, pense em quebrar o teste em métodos menores.

Exemplo:
```java
// Evite ❌

@isTest
static void testarBloqueioDesbloqueioCotacao() {
	// Arrange
	Quote quote = [SELECT Id FROM Quote];

	// Act
	Test.startTest();
	quote.ApprovalStatus__c = 'Approved';
	update quote;
	Test.stopTest();

	// Assert
	System.assertEquals(true, Approval.isLocked(quote.Id));

	// Act again ❌
	quote.ApprovalStatus__c = 'Pending';
	update quote;

	// Assert again ❌
	System.assertEquals(false, Approval.isLocked(quote.Id));

	// Act again ❌
	quote.ApprovalStatus__c = 'Cancelled';
	update quote;

	// Assert again ❌
	System.assertEquals(false, Approval.isLocked(quote.Id));
}
```
```java
//Prefira ✅

@isTest
static void testarBloqueioCotacao() {
	// Arrange
	Quote quote = [SELECT Id FROM Quote];

	// Act
	Test.startTest();
	quote.ApprovalStatus__c = 'Approved';
	update quote;
	Test.stopTest();

	// Assert
	System.assertEquals(true, Approval.isLocked(quote.Id));
}
```

<br>

### Tratamento de erros
 - Sempre que identificar uma situação previsível de erro, faça o tratamento e exiba uma mensagem clara, principalmente em cenários de integração e o programa possa falhar por falta de algum dado teoricamente obrigatório.
 - Não mascare erros ao fazer o tratamento. É preferível uma mensagem de erro detalhada (mesmo que técnica demais para um usuário) no lugar de uma mensagem do tipo "Erro ao processar informação", que tanto o usuário quanto os desenvolvedores ficarão perdidos ao visualizá-la.

<br>

### Refatoração
Pense que seu programa não fica pronto quando apenas "funcionou". A definição de um código pronto é **funcionando**, **testado** e **refatorado**. Seu primeiro objetivo é de fato fazer funcionar. Resolva o seu problema de negócio/técnico em primeiro lugar, execute testes, refatore.

**Pense nisso como um ciclo constante até o final do desenvolvimento:**

---------

➡️ Codificar ➡️ Testar ➡️ Refatorar ➡️ Repetir ↩️

ou:

➡️ Testar ➡️ Codificar ➡️ Refatorar ➡️ Repetir ↩️

---------
<br>

Não deixe o refactoring como último passo da sua tarefa, tente fazer isso em pequenos ciclos. É comum e esperado fazer um refactoring final, porém pequenos refactorings ajudam no processo de construção. É mais difícil trabalhar no meio da bagunça, certo?

Um pintor não pinta uma casa e vai embora, deixando tinta e pincéis espalhados, piso sujo e escada jogada. Pelo menos não deveria. Pense a mesma coisa com o seu código. Limpe a sua sala após terminar a pintura, mas faça pequenas limpezas (refactoring) durante o processo (ciclo) para não tropeçar na bagunça.

<br>
<img src="https://thumbs.dreamstime.com/b/m%C3%A3o-do-pintor-com-rolo-de-pintura-93838520.jpg" width="250">

<br>

---


| Platform Builders  - 02/2024 |
|    :----:   |