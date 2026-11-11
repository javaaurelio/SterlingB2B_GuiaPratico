<div class="hero">

<div class="hero-inner">

<span class="tag">Parte 1</span> <span class="tag">IBM Sterling B2B
Integrator</span> <span class="tag">Business Process (BPML)</span>
<span class="tag">Performance</span>

# Parte 1 – Guia prático: como fazer seus Business Processes rodarem mais rápido

Boas práticas de desenvolvimento de BPs no ISBI: persistência,
comunicação, loops, ProcessData, xPath e sub-BPs. Cada tópico traz o
**porquê**, um exemplo e o que evitar.

</div>

</div>

<div class="layout">

**Neste guia** [Antes de começar](#intro) [1. Nomenclatura de
BPs](#nomenclatura) [2. BP com tempo ruim](#tempoexec) [3.
Persistência](#persistencia) [4. End Session](#endsession) [5.
Loops](#loops) [6. ProcessData enxuto](#processdata) [7. Cuidado com o
xPath](#xpath) [8. Sub-BPs (Invoke)](#invoke) [9. File System
Adapter](#filesystem) [10. Java Task](#javatask) [11. Sleep](#sleep)
[12. BPs de longa duração](#longos) [13. Mailbox](#mailbox) [14. Routing
rules](#routing) [15. Queue (1 a 9)](#filas) [16. Otimize
chamadas](#chamadas) [Checklist final](#checklist)
[Glossário](#glossario) [Referências](#referencias)

<div role="main">

<div id="intro" class="section card">

<span id="intro"></span>

## Antes de começar

Ao desenvolver Business Processes (BPs), há decisões que parecem
pequenas, mas mudam muito a velocidade de execução: **quanto o BP grava
no banco**, **quais serviços e adapters ele usa** e **como o BPML é
escrito**. Usadas corretamente, essas opções melhoram o desempenho
geral.

Este guia reúne boas práticas de desenvolvimento: um blog técnico e a
documentação oficial do ISBI 6.2.0 sobre tempo de execução de BP, mais
orientações sobre File System Adapter, Java Task, Sleep, BPs de longa
duração, uso de Mailbox, routing rules, distribuição de BPs entre queues
e otimização de chamadas. O foco é o **desenho e a codificação do BP**.

<div class="box info">

<span class="t">Como usar este guia</span>

Comece pelo tópico 1 para definir a nomenclatura dos BPs e pelo tópico 2
para identificar se o seu BP está ficando mais lento e quais são as
causas de desenho mais comuns. Depois consulte os tópicos seguintes e
use o [checklist](#checklist) antes de publicar um BP. Termos técnicos
estão no [glossário](#glossario).

</div>

| Tópico                                   | O que ganho?                                                                     |
|------------------------------------------|----------------------------------------------------------------------------------|
| 1\. Nomenclatura de BPs                  | Achar, entender e dar manutenção nos BPs com rapidez                             |
| 2\. BP com tempo ruim                    | Saber onde olhar quando um BP fica mais lento                                    |
| 3\. Persistência                         | Menos gravação no banco, BP mais rápido                                          |
| 4\. End Session                          | Comunicações (FTP/sFTP) sem sessões abertas                                      |
| 5 a 8. Loops, ProcessData, xPath, Invoke | BPs leves e rápidos                                                              |
| 9\. File System Adapter                  | Evitar listar e filtrar diretórios enormes                                       |
| 10\. Java Task                           | Código customizado sem custo de compilação a cada execução                       |
| 11 e 12. Sleep e BPs longos              | Não prender threads e manter cada execução curta                                 |
| 13\. Mailbox                             | Evitar mailbox inchada, atrasos de roteamento e erros de mensagem não encontrada |
| 14\. Routing rules                       | Escolher entre avaliação automática e manual                                     |
| 15\. Queue (1 a 9)                       | Distribuir a carga entre as queues e separar BPs críticos dos de lote            |
| 16\. Otimize chamadas                    | Menos chamadas redundantes, aproveitando os steps do fluxo                       |

</div>

<div id="nomenclatura" class="section card">

<span id="nomenclatura"></span>

## <span class="num">1</span> Nomenclatura de Business Processes

<span class="nivel">Básico</span>

Antes de criar o primeiro BP, defina como os BPs serão nomeados. A
documentação do ISBI traz o capítulo **Design Guidelines for Business
Process Models**, que trata de nomenclatura, reuso, estrutura e
gerenciamento contínuo dos modelos de BP. Ele está disponível nas
versões 6.0.2, 6.1.0, 6.1.2, 6.2.1 e 6.2.2.

<div class="box info">

<span class="t">Sobre a fonte</span>

A documentação trata nomenclatura como parte do desenho dos modelos e dá
princípios gerais, mas não impõe um padrão de nome. As regras e os
exemplos detalhados estão no documento completo de Design Guidelines
(seção de boas práticas de modelagem de BP). O formato sugerido mais
abaixo é uma proposta nossa, que você pode aplicar à sua organização.

</div>

### Princípios

| Princípio           | O que significa                                                        |
|---------------------|------------------------------------------------------------------------|
| Consistência        | Use o mesmo padrão de nome em todos os BPs.                            |
| Clareza             | O nome descreve a função ou o propósito do BP.                         |
| Manutenibilidade    | O nome facilita o gerenciamento e a análise de problemas no dia a dia. |
| Facilidade de busca | O padrão permite localizar o BP rapidamente.                           |

### Quando definir

- Na fase de planejamento da estrutura dos modelos de BP.
- Antes de criar novos BPs.
- Ao alterar modelos de BP existentes.

### Aplique a mesma ideia a outros componentes

A nomenclatura não vale só para BPs. O mesmo raciocínio (padrão
consistente, nome que descreve a função e prefixo por projeto) também
pode ser aplicado a outros componentes do ambiente, como Mailboxes,
Routing Rules, adapters e serviços configurados e outros itens que você
cria no ambiente. Assim, quem procura um item em qualquer tela encontra
tudo organizado pelo mesmo critério.

### Vantagens de usar nomes padronizados

- **Localização rápida:** a busca por prefixo ou por trecho do nome (na
  tela de BPs, no Performance Statistics Report ou nos logs) traz só o
  que interessa.
- **Menos tempo de análise:** no `wf.log` e nos relatórios, o nome já
  diz de que sistema, projeto e função é o BP.
- **Ordenação natural:** BPs do mesmo projeto aparecem agrupados nas
  listagens.
- **Menos duplicidade e reuso:** fica fácil ver se já existe um BP com a
  função que você precisa.
- **Manutenção e transição de equipe:** quem chega ao projeto entende a
  estrutura sem depender de quem a criou.
- **Estatísticas por projeto:** com o `<PROJETO>` no nome, dá para
  separar e contabilizar a quantidade de execuções de cada projeto (veja
  a seção abaixo).
- **Automação e governança:** scripts, filtros de monitoramento e
  revisões de código podem se apoiar no padrão.

### Sugestão de formato

    <SIGLA>_<PROJETO>_<TIPO>_<FUNCIONALIDADE>_<DESCRICAO>[_<VERSAO>]

| Parte               | Descrição                                                                                                                   | Exemplo                     |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------|-----------------------------|
| `SIGLA`             | Sigla da corporação ou da área dona do BP                                                                                   | `ACME`                      |
| `PROJETO`           | Projeto, sistema ou parceiro                                                                                                | `PEDIDOS`                   |
| `TIPO`              | Papel do BP: principal (`MAIN`), sub-BP (`SUB`), tratamento de erro (`ERR`), utilitário (`UTIL`) ou rotina agendada (`JOB`) | `MAIN`                      |
| `FUNCIONALIDADE`    | Grande função de negócio ou técnica                                                                                         | `ENVIO`, `RECEBE`, `VALIDA` |
| `DESCRICAO`         | Detalhe curto que distingue o BP dos demais                                                                                 | `NF_SFTP`                   |
| `VERSAO` (opcional) | Versão, só quando for necessário manter mais de uma ao mesmo tempo                                                          | `V2`                        |

<div class="compare">

<div class="ruim">

<span class="rot">✗ Nomes que atrapalham</span>

    teste
    BP_novo
    envia2
    Copy of ENVIO_FINAL
    sftp_joao

</div>

<div class="bom">

<span class="rot">✓ Nomes padronizados</span>

    ACME_PEDIDOS_MAIN_ENVIO_NF_SFTP
    ACME_PEDIDOS_SUB_VALIDA_XML_NF
    ACME_PEDIDOS_ERR_ENVIO_NOTIFICA
    ACME_PEDIDOS_JOB_LIMPEZA_MAILBOX

</div>

</div>

### Estatísticas de volume e governança por projeto

Como o nome do projeto fica sempre na mesma posição do nome do BP, é
possível **separar e contabilizar a quantidade de execuções de cada
`<PROJETO>`** filtrando pelo prefixo. Isso vale para os relatórios do
próprio ISBI, como o Performance Statistics Report e o `wf.log`, e
também para o **IBM Control Center Monitor**.

| Filtro por prefixo   | O que você consegue medir                                                                                  |
|----------------------|------------------------------------------------------------------------------------------------------------|
| `ACME_`              | Volume total de execuções da corporação ou da área                                                         |
| `ACME_PEDIDOS_`      | Volume de execuções de um projeto específico                                                               |
| `ACME_PEDIDOS_MAIN_` | Quantidade de processamentos de negócio do projeto, sem contar sub-BPs, e portanto sem duplicar a contagem |
| `ACME_PEDIDOS_ERR_`  | Quantidade de tratamentos de erro do projeto, um indicador de qualidade                                    |
| `ACME_PEDIDOS_JOB_`  | Rotinas agendadas do projeto                                                                               |

<div class="box dica">

<span class="t">Uma ótima forma de gerar estatísticas e
governança</span>

Esses números ajudam a:

- **Quantificar** o uso da plataforma por projeto, área ou parceiro,
  como base para planejamento de capacidade e prestação de contas.
- Identificar **quais projetos mais consomem** recursos e acompanhar a
  evolução do volume ao longo do tempo.
- Comparar execuções principais com erros e achar projetos com taxa de
  falha acima do esperado.
- Apoiar a **governança**: saber quem é o dono de cada BP, identificar
  BPs fora do padrão e apoiar a revisão de BPs sem uso.

<figure>
<p>[Gráfico ilustrativo: execuções por projeto nos últimos 30 dias. Os
dados são fictícios.]</p>
<figcaption>Exemplo ilustrativo com dados fictícios: quantidade de
execuções por projeto, obtida filtrando os BPs pelo prefixo
<code>&lt;SIGLA&gt;_&lt;PROJETO&gt;_</code>. Os valores reais vêm do seu
ambiente.</figcaption>
</figure>

</div>

<div class="box atencao">

<span class="t">Valide no seu ambiente</span>

A forma de filtrar por trecho do nome varia conforme a tela e a versão
(ISBI ou Control Center Monitor). Teste o filtro por prefixo no seu
ambiente antes de montar relatórios ou painéis em cima dele.

</div>

<div class="box dica">

<span class="t">Dicas para o padrão funcionar</span>

Use letras maiúsculas, números e `_` como separador; evite espaços,
acentos e caracteres especiais. Documente a lista de siglas e de tipos
em um lugar acessível ao time, e mantenha os nomes curtos o bastante
para caber nas telas e nos logs. Não coloque nomes de pessoas nem datas
no nome do BP.

</div>

<div class="box atencao">

<span class="t">Cuidado ao renomear</span>

Um BP existente pode ser chamado por outros BPs (Invoke), por routing
rules, por agendamentos e por configurações de adapters. Antes de
renomear, levante essas dependências e teste em ambiente não produtivo.

</div>

</div>

<div id="tempoexec" class="section card">

<span id="tempoexec"></span>

## <span class="num">2</span> Quando o tempo de execução do BP começa a piorar

<span class="nivel">Diagnóstico</span>

O foco aqui é o **tempo que o BP leva para executar** e que vem
aumentando. Veja como perceber o problema e quais erros de desenho
costumam estar por trás dele.

### Sintomas

- O **Performance Statistics Report** (*Operations \> System \>
  Performance \> Statistics*) mostra os tempos *Min (ms)*, *Max (ms)* e
  *Average (ms)* crescendo para BPs, atividades e atividades internas do
  sistema.
- O `wf.log`, em `install_dir/logs`, mostra o tempo de execução dos BPs
  aumentando.

### Causa típica: BP mal desenhado

Descobrir a causa pode exigir investigar a forma como o BP foi desenhado
e escrito. Estes são os problemas mais comuns:

### O que significa "BP mal desenhado"?

| Problema                             | Por que atrapalha                                                    | Onde ver neste guia              |
|--------------------------------------|----------------------------------------------------------------------|----------------------------------|
| Persistência alta demais             | Dados demais são gravados no banco.                                  | [Tópico 3](#persistencia)        |
| Invocações de serviço desnecessárias | Consomem recursos que outros BPs e componentes poderiam usar.        | Revise o desenho do BP           |
| Loops desnecessários                 | Gastam recursos que outros BPs poderiam usar.                        | [Tópico 5](#loops)               |
| xPath mal escrito                    | Consome recursos; começar com `//` faz percorrer todo o ProcessData. | [Tópico 7](#xpath)               |
| Serviços e adapters antigos          | Os mais novos podem reunir a função de vários antigos em um só.      | Veja o exemplo de revisão abaixo |

### Como determinar a causa

1.  **Leia o `wf.log`** em `install_dir/logs`: procure datas e horários
    em que o tempo de execução aumentou e os erros de cada BP.
2.  **Consulte o Performance Statistics Report** para ver invocações e
    tempo de processamento de cada BP e atividade do sistema.
3.  **Revise os BPs que ficaram mais lentos**: desenho e xPath.

<div class="box info">

<span class="t">Exemplo de revisão</span>

Usar o *Command Line adapter* para obter dados gasta mais recursos que o
*File System adapter*, que em alguns casos faz o mesmo. Em outros casos,
um xPath em um serviço *Assign* substitui a necessidade dos dois
adapters. Use também o *Multi Assign*, que reúne várias atribuições em
um único serviço.

</div>

<div class="box dica">

<span class="t">Mas vale delegar tarefas pesadas ao SO via Command
Line</span>

Nem todo uso do Command Line é um problema. Quando a operação é custosa
para o BP e o sistema operacional já tem uma ferramenta nativa e
otimizada para ela, muitas vezes **vale submeter o processo ao SO via
Command Line**. O ganho costuma aparecer com arquivos grandes. Exemplos
sugeridos (não vêm da documentação IBM; adapte ao seu SO):

- **Split** de arquivos grandes em partes menores (ex.:
  `split -l 100000`).
- **Parse/conversão Base64** (ex.: `base64` e `base64 -d`; no Windows,
  `certutil -encode` e `-decode`).
- **Filtrar, ordenar ou transformar texto** em grande volume (ex.:
  `grep`, `sort`, `awk`, `sed`).
- **Calcular checksum/hash** (ex.: `sha256sum`).
- **Juntar ou concatenar** arquivos (ex.: `cat`).

**Cuidados ao usar:**

- Valide os parâmetros (nomes e caminhos de arquivo) para evitar injeção
  de comando.
- Os comandos mudam entre Linux/AIX e Windows. O BP passa a depender do
  SO e dos binários instalados no servidor (em cluster, em todos os
  nodes).
- Trate o retorno e os erros do comando (código de saída e mensagens de
  erro) e limpe os arquivos temporários.
- Prefira uma chamada que faça o trabalho completo a várias chamadas
  pequenas.
- Teste com volume real em ambiente não produtivo e alinhe as permissões
  com a equipe de infraestrutura.

</div>

- <a
  href="https://www.ibm.com/docs/en/b2b-integrator/6.2.0?topic=troubleshooting-symptoms-causes-poor-business-process-execution-time"
  target="_blank">Página original: Symptoms and Causes of Poor Business
  Process Execution Time</a>

</div>

<div id="persistencia" class="section card">

<span id="persistencia"></span>

## <span class="num">3</span> Níveis de persistência

<span class="nivel">Básico</span>

**Persistência** é o quanto da execução do BP é gravado no banco de
dados. Com *full* (padrão), **cada step** executado é gravado. Reduzir a
persistência faz o BP rodar mais rápido e ocupar menos espaço no banco,
porque menos steps são gravados.

<figure>
<img src="assets/imagem-01.jpg" width="885"
alt="Opções de nível de persistência do BP" />
<figcaption>Opções de persistência na configuração do BP.</figcaption>
</figure>

<div class="box atencao">

<span class="t">Cuidado</span>

Alguns serviços podem **não funcionar corretamente** com persistência
muito baixa. Teste seus BPs bem para achar o nível que atende à sua
necessidade.

</div>

Documentação sobre a persistência de cada BP:

- <a
  href="http://www-01.ibm.com/support/knowledgecenter/SS3JSW_5.2.0/com.ibm.help.bp_modeling.doc/SI_BusinessProcessPersistenceOptions.html?lang=en"
  target="_blank">Opções de persistência do BP</a>

### Um serviço quebrou com persistência baixa? Sobrescreva só ele

Em vez de voltar o BP inteiro para *full*, use o parâmetro
`WF_RUNTIME_OVERRIDE_PERSISTENCE_LEVEL` para mudar a persistência apenas
daquele serviço ou adapter, em tempo de execução. No exemplo, o *Get
Document Info* é forçado para full:

    <operation name="GetDocumentInfoService">
      <participant name="GetDocumentInfoService"/>
      <output message="xout">
        <assign to="WF_RUNTIME_OVERRIDE_PERSISTENCE_LEVEL">PERSISTENCE_FULL</assign>
        <assign to="." from="*"/>
      </output>
      <input message="xin">
        <assign to="." from="*"/>
      </input>
    </operation>

- <a
  href="https://supportcontent.ibm.com/support/pages/sterling-b2b-integrator-how-override-persistence-level-specific-service-bpml"
  rel="noopener" target="_blank">Como sobrescrever o nível de persistência
  de um serviço específico no BPML</a>

</div>

<div id="endsession" class="section card">

<span id="endsession"></span>

## <span class="num">4</span> Sempre use "End Session" em BPs de comunicação

<span class="nivel">Intermediário</span>

Toda sessão aberta por qualquer protocolo precisa ser fechada. Com
conjuntos de adapters "Client" (como o FTP Client), sempre termine a
conexão com um **Client End Session**.

<div class="box perigo">

<span class="t">Erro comum</span>

Esquecer de tratar o **erro**. Se algo falhar durante a transferência, o
BP precisa entrar em um `onFault` que também execute o *Client End
Session*, senão a sessão fica aberta.

</div>

<div class="box atencao">

<span class="t">Verifique se a sessão existe antes de encerrá-la</span>

As imagens abaixo **não mostram** essa verificação, mas ela é
importante. Antes de executar o *FTP Client End Session* (ou o
equivalente do protocolo em uso), confirme que a sessão **de fato
existe**. No `onFault` isso é ainda mais necessário: o erro pode ter
acontecido *antes* de a sessão ser aberta (por exemplo, falha ao
conectar), e aí não há sessão para encerrar. Chamar o End Session sem
sessão gera um novo erro dentro do próprio tratamento de falha.

**Como aplicar:** guarde no ProcessData o identificador ou um indicador
de que o *Client Begin Session* foi bem-sucedido, e use um *Choice*
(condição) para só chamar o End Session quando esse indicador existir.

</div>

<div class="box atencao">

<span class="t">Nomes de Sequence Start devem ser únicos dentro do
BP</span>

Na imagem, o fluxo principal e o `onFault` aparecem com o mesmo nome em
"Sequence Start". Isso **não deve ser reproduzido**: dentro de um mesmo
BP, cada *Sequence Start* precisa de um nome **único**. Use nomes que
identifiquem o papel de cada bloco, por exemplo:

    Sequence Start  -> SEQ_FTP_PRINCIPAL      (fluxo principal)
    Sequence Start  -> SEQ_FTP_ONFAULT        (tratamento de falha)

</div>

O exemplo abaixo mostra um BP de FTP com o *FTP Client End Session* no
final do fluxo normal **e** outro dentro do `onFault` da sequência que
faz os steps de FTP.

*Atenção: as imagens são apenas ilustrativas. Elas não mostram a
verificação da sessão e repetem o nome do Sequence Start; ajuste os dois
pontos ao implementar.*

<figure>
<img src="assets/imagem-02.jpg"
alt="BP de FTP com End Session ao final do fluxo" />
<figcaption>Fluxo principal terminando com End Session.</figcaption>
</figure>

<figure>
<img src="assets/imagem-03.jpg" alt="onFault com End Session" />
<figcaption>Bloco <code>onFault</code> também fechando a
sessão.</figcaption>
</figure>

</div>

<div id="loops" class="section card">

<span id="loops"></span>

## <span class="num">5</span> Evite loops com muitos steps

<span class="nivel">Básico</span>

Ao usar loop, evite repetir uma grande quantidade de steps. Coloque um
**limite no contador** e deixe um próximo BP continuar o processamento.

### O que é um step?

Os **Services** e **Adapters** é que fazem o trabalho em um BP, e cada
um deles é um **step**. Como são programas como quaisquer outros, eles
seguem o paradigma clássico de **3 partes**:

1.  **Input (entrada):** a mensagem que o step recebe.
2.  **Transformation (transformação):** o trabalho que o step executa.
3.  **Output (saída):** a mensagem que o step devolve.

A entrada e a saída são as **mensagens** do step, que aparecem no BPML
como `<output message="...">` (o que o BP envia ao serviço) e
`<input message="...">` (o que o serviço devolve ao BP).

<figure>
<p>[Gráfico ilustrativo: execuções por projeto nos últimos 30 dias. Os
dados são fictícios.]</p>
<figcaption>Ilustração do paradigma Input, Transformation e
Output.</figcaption>
</figure>

<div class="box info">

<span class="t">Por que isso importa para os loops</span>

Cada volta do loop executa vários steps, e **cada step tem a sua
entrada, a sua transformação e a sua saída** (e, conforme a
persistência, grava o resultado). Por isso o custo de um loop é o número
de steps multiplicado pelo número de itens.

</div>

<div class="box info">

<span class="t">Faça a conta</span>

100 itens × 20 steps por iteração = **no mínimo 2000 steps** em um único
BP. Com muitos itens, o BP cresce rápido.

</div>

### Como seria o ideal: processar em lotes

Em vez de um BP que percorre todos os itens, o BP processa um **lote com
limite** (no exemplo, 25 itens), remove do ProcessData o que já foi
feito e, se ainda restam itens, **inicia o próximo BP** com o restante.
Cada execução fica curta e o ProcessData não cresce.

<figure>
<p>[Gráfico ilustrativo: execuções por projeto nos últimos 30 dias. Os
dados são fictícios.]</p>
<figcaption>Desenho lógico ilustrativo. Cada execução trata no máximo 25
itens, e o próximo BP recebe apenas o que falta.</figcaption>
</figure>

|                        | ✗ Um BP com tudo             | ✓ Lotes com limite                                  |
|------------------------|------------------------------|-----------------------------------------------------|
| Itens (exemplo)        | 100 itens em 1 execução      | 100 itens em 4 execuções de 25                      |
| Steps por execução     | ≈ 2000                       | ≈ 500 (25 × 20)                                     |
| Tamanho do ProcessData | Cresce até o fim da execução | Pequeno em cada lote, pois o item pronto é removido |
| Se algo falhar         | O BP todo é afetado          | Só o lote que falhou é reprocessado                 |

### Representação lógica (pseudocódigo)

    // Pseudocódigo para ilustrar a ideia. Não é BPML.
    LIMITE   = 25
    contador = 0

    enquanto (contador < LIMITE e existe item na lista) {
        item = primeiro(lista)
        processar(item)              // os ~20 steps do item
        release(item)                // tira o item do ProcessData
        contador = contador + 1
    }

    se (existe item na lista) {
        // passa só o restante ao próximo BP (veja message_to_child)
        iniciar_proximo_BP(restante = lista)
    }
    encerrar

<div class="box dica">

<span class="t">Pontos de atenção na implementação</span>

- O **LIMITE** não é um número mágico: ajuste com testes, de acordo com
  o tamanho do ProcessData e o tempo de cada item.
- Use o *Release* a cada volta e passe ao próximo BP só o necessário com
  `message_to_child` (veja os tópicos [6](#processdata) e [8](#invoke)).
- Para iniciar o próximo BP, uma opção é o serviço *Invoke Sub-Process*
  em modo assíncrono. Valide na documentação do serviço qual modo atende
  ao seu caso.
- Evite que o BP reinicie sem fim: garanta que o próximo BP só é chamado
  quando **ainda existem itens** e que cada execução remove ao menos um
  item.
- Alinhe com o tópico de [BPs de longa duração](#longos): lotes menores
  também reduzem o tempo que cada BP segura a thread.

</div>

</div>

<div id="processdata" class="section card">

<span id="processdata"></span>

## <span class="num">6</span> Mantenha o ProcessData enxuto

<span class="nivel">Intermediário</span>

O **ProcessData** (Instance Data) é a "memória de trabalho" do BP.
Quanto maior ele fica, mais lento o BP. Procure mantê-lo o menor
possível. Seis dicas:

<div class="box perigo">

<span class="t">Por que o tamanho importa: o ProcessData é persistido no
banco de dados</span>

O ProcessData é gravado no **banco de dados** durante a execução do BP,
conforme o nível de persistência. Quando o ProcessData é grande, cada
gravação e leitura move mais dados, o que **eleva o uso de I/O** e o
**uso do cache do banco** (além de rede, memória e CPU), e atrapalha
também os outros BPs que usam o mesmo banco. Quanto maior o ProcessData
e quanto mais steps o BP tem, maior o custo.

</div>

### 6.1 Limpe o ProcessData a cada iteração do loop

Use o serviço **Release** para remover o que não é mais necessário. Para
acelerar, use sempre a **primeira entrada** da lista no loop e, no fim
de cada volta, remova essa entrada com o Release. O ProcessData vai
encolhendo conforme os itens são processados.

Exemplo: o Release abaixo remove o conteúdo do nó `config`.

    <operation name="Release Service">
      <participant name="ReleaseService"/>
      <output message="ReleaseServiceTypeInputMessage">
        <assign to="TARGET">/ProcessData/test/config</assign>
        <assign to="." from="*"></assign>
      </output>
      <input message="inmsg">
        <assign to="." from="*"></assign>
      </input>
    </operation>

### 6.2 Grave só o que precisa: filtre a saída do serviço

Use *Message from Service* com *Allow Message write* para filtrar, via
xPath, o que vai para o ProcessData. Em serviços como o *Get Document
Info*, dá para gravar só as informações necessárias.

<div class="compare">

<div class="ruim">

<span class="rot">✗ Grava tudo</span>

    <input message="xin">
      <assign to="." from="*"/>
    </input>

</div>

<div class="bom">

<span class="rot">✓ Grava só o necessário</span>

    <input message="xin">
      <assign to="." from="BodyName/text()"/>
      <assign to="." from="BodyLength/text()"/>
    </input>

</div>

</div>

Ver o ProcessData resultante em cada caso

**Grava tudo** (muitos campos que você talvez nem use):

    <ProcessData>
      <PrimaryDocument SCIObjectID="rhel65_templ:node1:150cd007207:3054018"/>
      <BodyName>input_document.txt</BodyName>
      <BodyLength>193</BodyLength>
      <DocumentBodyLength>193</DocumentBodyLength>
      <DocumentLength>193</DocumentLength>
      <DocumentName>input_document.txt</DocumentName>
      <DocumentId>rhel65_templ:node1:150cd007207:3054016</DocumentId>
      <DocumentSubject/>
      <DocumentCreateTime>2015-11-06 12:34:03.202</DocumentCreateTime>
      <DocumentModifiedTime>2015-11-06 12:34:03.202</DocumentModifiedTime>
      <DocumentLifeSpan>0 Minutes</DocumentLifeSpan>
      <DocumentInitialId>rhel65_templ:node1:150cd007207:3054016</DocumentInitialId>
      <DocumentMaxInlineBodySize>102400</DocumentMaxInlineBodySize>
      <DocumentPurgeAfter>2015-11-06 12:34:03.213</DocumentPurgeAfter>
      <DocumentStorageType>DATABASE</DocumentStorageType>
      <DocumentWorkflowId>1761983</DocumentWorkflowId>
      <Encrypted>false</Encrypted>
      <DocumentType>0</DocumentType>
      <DocumentContentType>application/octet-stream</DocumentContentType>
    </ProcessData>

**Grava só nome e tamanho**:

    <ProcessData>
      <PrimaryDocument SCIObjectID="rhel65_templ:node1:150cd007207:3056496"/>
      <BodyName>input_document.txt</BodyName>
      <BodyLength>193</BodyLength>
    </ProcessData>

### 6.3 Não grave resultados na raiz do ProcessData

Use sempre um **nó específico** para o resultado de um serviço ou
adapter. Alguns nós gravados na raiz podem interferir em serviços
executados depois. Como bônus, fica fácil limpar: um Release do nó
remove tudo que está dentro dele.

<div class="compare">

<div class="ruim">

<span class="rot">✗ Na raiz</span>

    <assign to="." from="*"/>

</div>

<div class="bom">

<span class="rot">✓ Dentro de um nó</span>

    <assign to="GetDocInfoResults" from="*"/>

</div>

</div>

Ver o ProcessData com o nó GetDocInfoResults

    <ProcessData>
      <PrimaryDocument SCIObjectID="rhel65_templ:node1:150cd007207:3054018"/>
      <GetDocInfoResults>
        <BodyName>input_document.txt</BodyName>
        <BodyLength>193</BodyLength>
        <DocumentBodyLength>193</DocumentBodyLength>
        <DocumentLength>193</DocumentLength>
        <DocumentName>input_document.txt</DocumentName>
        <DocumentId>rhel65_templ:node1:150cd007207:3054016</DocumentId>
        <DocumentSubject/>
        <DocumentCreateTime>2015-11-06 12:34:03.202</DocumentCreateTime>
        <DocumentModifiedTime>2015-11-06 12:34:03.202</DocumentModifiedTime>
        <DocumentLifeSpan>0 Minutes</DocumentLifeSpan>
        <DocumentInitialId>rhel65_templ:node1:150cd007207:3054016</DocumentInitialId>
        <DocumentMaxInlineBodySize>102400</DocumentMaxInlineBodySize>
        <DocumentPurgeAfter>2015-11-06 12:34:03.213</DocumentPurgeAfter>
        <DocumentStorageType>DATABASE</DocumentStorageType>
        <DocumentWorkflowId>1761983</DocumentWorkflowId>
        <Encrypted>false</Encrypted>
        <DocumentType>0</DocumentType>
        <DocumentContentType>application/octet-stream</DocumentContentType>
      </GetDocInfoResults>
    </ProcessData>

### 6.4 DocToDOM: traga só o necessário

Ao usar o **DocToDOM** para ler o documento primário, se não precisar do
conteúdo inteiro, **não** o coloque todo no ProcessData: recupere apenas
a informação de que precisa.

- <a href="https://www.ibm.com/support/pages/node/1121853"
  target="_blank">Trabalhando com a função xPath do DocToDOM</a>

### 6.5 Prefira o PrimaryDocument para conteúdo grande

Para arquivos grandes ou conteúdo binário, deixe o dado no
**PrimaryDocument**, que é mais performático. Reserve o ProcessData para
metadados pequenos, variáveis de controle e configurações.

| PrimaryDocument                                                                                                                                                                          | ProcessData                                                                             |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| Guarda no ProcessData só uma **referência** (`SCIObjectID`) ao documento, que fica armazenado fora do XML do processo (no banco de dados ou no disco, conforme o tipo de armazenamento). | Guarda o **dado em si** dentro da estrutura XML do processo.                            |
| Não carrega o conteúdo inteiro do documento na memória durante a execução do BP.                                                                                                         | É carregado por inteiro na memória durante a execução.                                  |
| Mais eficiente para arquivos grandes ou conteúdo binário.                                                                                                                                | Pode aumentar muito o consumo de memória com volumes grandes de dados.                  |
| Menor uso de memória durante a execução do BP.                                                                                                                                           | Mais indicado para metadados pequenos, variáveis de controle e valores de configuração. |

### 6.6 Onde ficam o PrimaryDocument e o ProcessData

- **PrimaryDocument:** fica **estacionado no disco** (filesystem) quando
  o armazenamento padrão de documentos está configurado como
  `defaultDocumentStorageType=FS`. Nesse caso, os **binários do arquivo
  são persistidos no disco** e as **referências a eles ficam no banco de
  dados**.
- **ProcessData:** é **trafegado e persistido no banco de dados a cada
  step**.

<div class="box dica">

<span class="t">Benefício de separar os dois</span>

Pode haver ganho de performance ao guardar o PrimaryDocument no
filesystem e o ProcessData no banco de dados, principalmente em cenários
com documentos grandes. Essa configuração reduz a necessidade de
armazenamento no banco e pode melhorar a vazão (throughput) de fluxos
que trabalham com muitos documentos.

O tamanho do ganho depende do tamanho dos documentos, da frequência de
acesso, da configuração da infraestrutura e da carga geral do sistema.

</div>

</div>

<div id="xpath" class="section card">

<span id="xpath"></span>

## <span class="num">7</span> Cuidado com o xPath: nunca comece com `//`

<span class="nivel">Básico</span>

Um xPath iniciado por `//` faz o motor **percorrer todo o ProcessData**
para achar o nó. Com ProcessData grande, isso deixa a busca muito mais
lenta.

<div class="compare">

<div class="ruim">

<span class="rot">✗ Começa com // (percorre tudo)</span>

    <assign to="test"
      from="//type/text()"></assign>

</div>

<div class="bom">

<span class="rot">✓ Caminho direto até o nó</span>

    <assign to="test"
      from="/ProcessData/customer/type/text()"></assign>

</div>

</div>

<div class="box atencao">

<span class="t">As fontes divergem em um detalhe</span>

O blog técnico recomenda o caminho completo
(`/ProcessData/customer/type/text()`). Já a documentação do IBM Docs
6.2.0 recomenda caminho relativo, por exemplo `PurchaseOrder/text` em
vez de `/ProcessData/PurchaseOrder/text()`. O ponto em comum é: **nunca
comece com `//`**. Entre as duas formas, valide no seu ambiente
comparando os tempos no Performance Statistics Report.

</div>

</div>

<div id="invoke" class="section card">

<span id="invoke"></span>

## <span class="num">8</span> Controle o ProcessData ao invocar sub-BPs

<span class="nivel">Intermediário</span>

Ao chamar outro BP com o serviço **Invoke**, por padrão o filho pode
receber o ProcessData inteiro do pai. Use a função `message_to_child`
para passar **somente o necessário**.

<div class="box info">

<span class="t">Como funciona</span>

Quando a tag `message_to_child` existe no ProcessData, o BP invocado
recebe **apenas os nós que estão dentro dela**.

</div>

- <a
  href="http://www-01.ibm.com/support/knowledgecenter/SS3JSW_5.2.0/com.ibm.help.svcs_adpts_a_l.doc/Invoke_BusProcess_svc.html?lang=en"
  target="_blank">Documentação do serviço Invoke Sub-Process</a>

</div>

<div id="filesystem" class="section card">

<span id="filesystem"></span>

## <span class="num">9</span> File System Adapter: `attachFile` contra `collectionFolder` + `filter`

<span class="nivel">Intermediário</span>

O File System Adapter pode obter arquivos de duas formas. A escolha muda
bastante o custo quando o diretório tem muitos arquivos (na casa de 100
mil ou mais).

<div class="compare">

<div class="ruim">

<span class="rot">✗ Coleta por diretório + filtro</span>

    <collectionFolder>/directory/path</collectionFolder>
    <filter>filename_pattern</filter>

</div>

<div class="bom">

<span class="rot">✓ Caminho exato do arquivo</span>

    <attachFile>/exact/path/to/file.txt</attachFile>

</div>

</div>

### Como cada um trabalha

- **`collectionFolder` + `filter`:** o adapter lista todos os arquivos
  do diretório, percorre um a um e aplica o filtro para achar os que
  combinam. Em diretórios com muitos arquivos isso consome muitos
  recursos.
- **`attachFile`:** o adapter vai direto ao arquivo informado (por
  exemplo `/directory/FILE.TXT`), sem listar o diretório nem filtrar.

<div class="box info">

<span class="t">Em resumo</span>

- Com `filter` e `collectionFolder`, o adapter lista todos os arquivos
  do diretório e percorre os itens até encontrar os que combinam.
- O parâmetro `attachFile` evita a varredura do diretório, porque
  referencia o arquivo diretamente.
- Em ambientes com muitos arquivos (100.000 ou mais), o acesso direto ao
  arquivo pode trazer ganho de performance.

</div>

| Parâmetro                     | Comportamento                            | Performance                 | Deleta o arquivo |
|-------------------------------|------------------------------------------|-----------------------------|------------------|
| `collectionFolder` + `filter` | Lista todos os arquivos e filtra um a um | Menor em diretórios grandes | Sim (por padrão) |
| `attachFile`                  | Vai direto ao arquivo especificado       | Melhor                      | Não              |

<div class="box dica">

<span class="t">Quando usar `attachFile`</span>

Quando o BP já conhece o **nome exato** do arquivo de antemão.

</div>

<div class="box atencao">

<span class="t">Teste antes de adotar</span>

`attachFile` pode melhorar a performance, mas em alguns cenários pode
causar **outras falhas**. Faça testes de performance no seu ambiente e
confirme que o seu caso de uso não tem problemas. Dependendo do fluxo, o
filtro ainda pode ser necessário.

</div>

<div class="box info">

<span class="t">Mantenha o diretório limpo</span>

O File System Adapter não se comporta como um GET de FTP/SFTP. Se o
diretório acumula arquivos já processados, mantenha nele **apenas os
arquivos que precisam ser processados**. Isso ajuda a performance com
qualquer um dos parâmetros.

</div>

</div>

<div id="javatask" class="section card">

<span id="javatask"></span>

## <span class="num">10</span> Boas práticas com Java Task

<span class="nivel">Intermediário</span>

O Java Task é uma conveniência para lógica customizada **simples e pouco
frequente**. Em produção, com requisitos de performance, quase sempre
existe uma opção melhor.

### 10.1 Não faça chamadas HTTP direto no Java Task

O ISBI já tem componentes próprios para HTTP, que cuidam de pool de
conexões, tratamento de erros e log. Código HTTP escrito no Java Task
ignora essa infraestrutura e o monitoramento.

<div class="box perigo">

<span class="t">Sem rastro no monitoramento</span>

Uma chamada HTTP feita direto no Java Task **não é registrada nas
estatísticas do Communication Session nem no IBM Control Center
Monitor**. Quem acompanha o ambiente não enxerga essa comunicação.

</div>

<div class="box atencao">

<span class="t">Chamadas HTTPS exigem importar o CA</span>

Se a chamada for **HTTPS**, o certificado da CA precisa ser importado
**no Dashboard do ISBI** e **nos arquivos de segurança do Java**. Depois
da importação é necessária a **reciclagem (restart) do ambiente** para
ela valer.

</div>

<div class="compare">

<div class="ruim">

<span class="rot">✗ Evite</span>

Implementar a chamada HTTP no código do Java Task.

</div>

<div class="bom">

<span class="rot">✓ Prefira</span>

Usar o **HTTP Client Adapter** ou o **HTTP Server Adapter** no BP.

</div>

</div>

### 10.2 Se precisar do Java Task, use JARs pré-compilados

O código do Java Task (inline ou em arquivo-fonte) é **compilado a cada
execução**. Para reduzir esse custo:

1.  Compile a maior parte da lógica Java em arquivos JAR.
2.  Adicione os JARs ao classpath com a funcionalidade
    `install3rdpartyJars`.
3.  Deixe no Java Task apenas o mínimo: imports, instanciação, chamadas
    de método e `return`.

Isso fica mais rápido que código inline ou em arquivo-fonte.

### 10.3 Documentos no Java Task

Todo documento processado por um BP é gravado imediatamente no sistema
de arquivos ou no banco, conforme a configuração. Os documentos **não
ficam apenas em memória** durante a execução.

### 10.4 Quando o Java Task faz sentido

- Existe uma necessidade específica que nenhum serviço existente atende.
- A frequência de execução é baixa.

<div class="box atencao">

<span class="t">Escopo do suporte IBM</span>

O desenvolvimento de código customizado está fora do escopo do suporte.
O suporte ajuda com exemplos de uso de API, referências de documentação,
orientação sobre funcionalidades do produto e troubleshooting de
serviços entregues pelo produto. Desenvolvimento customizado, benchmark
de performance e consultoria de desenho exigem o IBM Services (via seu
representante de vendas ou gerente de conta).

</div>

### Resumo

| Prática                  | Recomendação                                                                                                                                              |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| Chamadas HTTP            | Usar HTTP Client/Server Adapter, e não código direto no Java Task (chamada direta não aparece no Communication Session nem no IBM Control Center Monitor) |
| Chamadas HTTPS           | CA importado no Dashboard e nos arquivos de segurança do Java, com reciclagem do ambiente                                                                 |
| Performance do Java Task | Lógica em JARs pré-compilados e código inline mínimo                                                                                                      |

</div>

<div id="sleep" class="section card">

<span id="sleep"></span>

## <span class="num">11</span> Uso do Sleep em Business Processes

<span class="nivel">Intermediário</span>

Em um BP, o Sleep deixa a execução parada por um tempo definido:

    <operation name="Sleep Service">
      <participant name="SleepService"/>
      <output message="SleepServiceInputMessage">
        <assign to="SLEEP_INTERVAL">60</assign>
        <assign to="." from="*">
        </assign>
      </output>
      <input message="inmsg">
        <assign to="." from="*">
        </assign>
      </input>
    </operation>

### Quando pode fazer sentido

- Atraso intencional entre tentativas de retry
- Espera por janela de disponibilidade de um sistema externo
- Limitação de ritmo (rate limiting) para não sobrecarregar um sistema
  de destino
- Sincronização com horários de processamento em lote

### Desvantagens

| Área           | Problema                                                                                                                   |
|----------------|----------------------------------------------------------------------------------------------------------------------------|
| Performance    | Consome recursos de thread durante a pausa, reduz o throughput e vira gargalo quando vários BPs usam Sleep ao mesmo tempo. |
| Contenção      | A thread continua alocada, mas ociosa, e pode impedir outros BPs de executar se o conjunto de threads se esgotar.          |
| Escalabilidade | Não escala bem em alto volume, pode causar acúmulo na queue e atrasos e levar a timeouts em processos dependentes.         |

### Alternativas ao Sleep

- Padrões orientados a eventos, em vez de polling com Sleep
- Mensageria assíncrona, sempre que possível
- Serviços de agendamento (*scheduled BPs*) para execução em horário
  definido
- Valores de timeout adequados, em vez de atrasos arbitrários
- Padrões de suspender/retomar o workflow para esperas longas

<div class="box dica">

<span class="t">Recomendação</span>

Minimize o Sleep em BPs de produção. Quando precisar de atraso, avalie
se a lógica pode ser redesenhada de forma assíncrona ou orientada a
eventos.

</div>

</div>

<div id="longos" class="section card">

<span id="longos"></span>

## <span class="num">12</span> BPs de longa duração (ex.: vários segundos)

<span class="nivel">Intermediário</span>

Um BP que fica vários segundos em execução (no exemplo abaixo, 45
segundos) prejudica a escalabilidade e o desempenho do ambiente,
principalmente quando há muitos BPs ao mesmo tempo e em ambientes em
cluster.

### Principais problemas

- **Consumo de threads:** BPs longos ocupam threads por muito tempo,
  sobram menos threads para novos BPs e a `queueDepth` (BPs aguardando
  na queue) aumenta.
- **Impacto no banco:** as conexões ficam abertas durante toda a
  execução, e locks e sincronizações podem bloquear outros processos.
- **Menor throughput:** menos transações por unidade de tempo e maior
  latência geral.
- **Escala em cluster:** a performance é limitada pela granularidade do
  trabalho, pela interação entre processos (locks e sincronização), pelo
  banco e pelo arranjo dos adapters.

### Exemplo: a conta que não fecha

| Cenário                                       | Valor                                      |
|-----------------------------------------------|--------------------------------------------|
| Volume                                        | 1000 transações/minuto ≈ 16,67 BPs/segundo |
| Duração média do BP                           | 45 segundos                                |
| Threads ocupadas ao mesmo tempo               | 16,67 × 45 = **≈ 750 threads**             |
| Threads disponíveis no exemplo (2 nodes × 50) | 100                                        |
| Resultado                                     | Déficit de 650 threads (87% insuficiente)  |

<div class="box info">

<span class="t">Regra prática</span>

Threads ocupadas ≈ **BPs por segundo × duração do BP em segundos**.
Quanto mais longo o BP, mais threads ele segura. Reduzir a duração reduz
diretamente a necessidade de threads.

</div>

### Sintomas

- A `queueDepth` (BPs aguardando na queue) só cresce, e o backlog
  aumenta continuamente.
- Contenção: locks de tabela forçam execução sequencial e as consultas
  ficam lentas pela alta concorrência.
- Degradação progressiva: a latência aumenta rapidamente e começam a
  ocorrer timeouts.

### Agravantes de desenho

<div class="box perigo">

<span class="t">Sleep dentro do BP</span>

Um Sleep longo (no exemplo, 60 segundos) mantém a thread **totalmente
ociosa**: ela não faz trabalho útil, só consome recurso.

</div>

### Como redesenhar

| Ação                    | Como                                                                                                                                                                                                                               |
|-------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Reduzir a granularidade | Dividir o BP em processos menores, processar dados em lotes menores e usar sub-processos assíncronos. Isso fala do *tamanho do BP*; dentro de cada parte, continue reduzindo chamadas redundantes (veja o tópico [16](#chamadas)). |
| Eliminar o Sleep        | Trocar por eventos ou mensageria, usar BPs agendados para retry e implementar o padrão de callback.                                                                                                                                |

<div class="box atencao">

<span class="t">Em cluster, o impacto se multiplica</span>

Com 2 nodes executando o dobro de BPs simultâneos, a carga no banco e
nos recursos compartilhados também dobra. Por isso é melhor reduzir a
duração do BP no desenho do que depender de mais infraestrutura. Para
operações longas, considere uma arquitetura assíncrona.

</div>

</div>

<div id="mailbox" class="section card">

<span id="mailbox"></span>

## <span class="num">13</span> Boas práticas de uso de Mailbox

<span class="nivel">Intermediário</span>

É comum guardar arquivos já entregues em pastas da mailbox (como
`/TRANSMITIDOS` ou `/JA_ENTREGUE`) como evidência. A intenção é
compreensível, mas **não é recomendada como prática de longo prazo**.

### Por que não manter arquivos processados na mailbox

- Degradação de performance pelo aumento da quantidade de mensagens na
  mailbox.
- Atraso nas respostas do banco, afetando as operações de roteamento.
- Erros *"Mailbox Message Not Found"* / *"Message ID not found"* quando
  o sistema está com volume alto.
- Problemas de timing: o step de roteamento não encontra a mensagem por
  causa de atraso no commit do banco.

### Como guardar a evidência sem manter o arquivo na mailbox

| Abordagem           | Como fazer                                                                                                                                                                                                                                                                                            |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Arquivamento por BP | Roteie os arquivos entregues com sucesso para um local de arquivamento **fora da estrutura ativa da mailbox**, no sistema de arquivos, em diretórios por data (ex.: `/archive/YYYY/MM/DD/`). Defina a retenção conforme a sua necessidade de compliance e configure permissões e controles de acesso. |
| Trilha de auditoria | Use os recursos nativos de rastreamento e relatório do ISBI e registre o status da transmissão nos metadados do workflow. Os registros do banco comprovam a entrega sem reter o arquivo na mailbox.                                                                                                   |
| Limpeza da mailbox  | Agende o BP **MailboxDeleteService** com regularidade, defina períodos de retenção para mensagens transitórias e remova as mensagens processadas depois que o arquivamento terminar.                                                                                                                  |

<div class="box dica">

<span class="t">Ciclo de vida do documento</span>

Arquivos processados com sucesso devem ser movidos para o armazenamento
de arquivamento. Defina limites de retenção para os caminhos ativos da
mailbox e acompanhe a quantidade de mensagens para evitar acúmulo.

</div>

<div class="box atencao">

<span class="t">Retenção padrão de 10 anos</span>

Quando uma mensagem é criada na mailbox, o **lifespan** dela é de **10
anos**. O *Document Tracking* dos BPs e as sessões associadas também são
mantidos por **10 anos ou até que a mensagem seja deletada da mailbox**.
Ou seja, se a mensagem não for removida, ela e o rastreamento dela
continuam ocupando o banco por muito tempo. Por isso a limpeza regular e
o arquivamento fora da mailbox são importantes.

Manter o banco de dados mais enxuto também deixa mais rápidas as rotinas
de backup, de manutenção da base e outras rotinas que dependem do
tamanho do banco.

</div>

### Mensagens da mailbox também são eventos

As mensagens da mailbox podem ser vistas como **eventos**, e não somente
como documentos. A mailbox permite trabalhar com o **conceito de
eventos**: a chegada de uma mensagem pode ser o gatilho que dispara um
BP, por meio das routing rules (veja o [tópico 14](#routing)).

- Em vez de um BP ficar em polling (ou em Sleep) esperando um arquivo,
  deixe a chegada da mensagem na mailbox iniciar o processamento.
- Isso combina com o que foi dito no tópico sobre [Sleep](#sleep):
  padrões orientados a eventos no lugar de espera ativa.
- Tratar a mensagem como evento também ajuda a decidir quando ela pode
  ser removida: depois de processada e arquivada, o evento cumpriu o
  papel.

### Passo a passo de implementação

1.  **Revise a estrutura atual:** identifique as pastas que guardam
    arquivos já processados e avalie a quantidade de mensagens e os
    períodos de retenção.
2.  **Desenhe o fluxo de arquivamento:** crie BPs que movam os arquivos
    entregues para o local de arquivamento, com estrutura de diretórios
    por data.
3.  **Configure o MailboxDeleteService:** defina os parâmetros de
    retenção, agende a execução e teste primeiro em ambiente não
    produtivo.
4.  **Monitore e ajuste:** acompanhe a quantidade de mensagens da
    mailbox e ajuste as políticas de retenção conforme necessário.

Notas técnicas de apoio (da documentação IBM):

- *Why schedule MailboxDeleteService Business Process*: explica por que
  a limpeza regular da mailbox é necessária.
- *Understanding Mailbox Message Life Cycle*: descreve o comportamento
  esperado e o gerenciamento das mensagens.
- *Understanding criteria for payload purge in /documents folder*:
  esclarece a limpeza no sistema de arquivos.

</div>

<div id="routing" class="section card">

<span id="routing"></span>

## <span class="num">14</span> Routing rules da mailbox: automática ou manual

<span class="nivel">Intermediário</span>

O ISBI tem dois modos de avaliar as routing rules de uma mailbox.
Escolher o modo certo evita mensagens perdidas ou roteadas mais de uma
vez.

<div class="compare">

<div class="bom">

<span class="rot">Automática</span>

Feita pelo serviço **Mailbox Evaluate All Automatic Rules**, agendado e
controlado pela interface da aplicação. A mensagem é avaliada **uma
única vez**, quando é adicionada à mailbox, e depois sai da tabela
"needs to be routed". Os BPs da regra iniciam logo após a avaliação, sem
esperar o término.

**Use para:** processar mensagens novas, cada uma avaliada uma única
vez.

</div>

<div class="ruim">

<span class="rot">Manual</span>

Feita pelo serviço **Mailbox Evaluate Routing Rule**, sob demanda. Toda
mensagem que atende ao padrão é roteada **a cada avaliação**, e a
mensagem continua elegível enquanto estiver na mailbox. Não há garantia
de entrega e, após uma queda do sistema, o roteamento **não retoma
sozinho**.

**Use para:** reprocessar mensagens existentes ou ter agendamento
próprio.

</div>

</div>

### Três formas de avaliar manualmente

1.  Dentro de um BP, chamando uma configuração do serviço *Mailbox
    Evaluate Routing Rule*.
2.  Por uma configuração agendada do serviço *Mailbox Evaluate Routing
    Rule*.
3.  De forma interativa, pela interface da aplicação.

### Comparação

| Característica                    | Automática                              | Manual                        |
|-----------------------------------|-----------------------------------------|-------------------------------|
| Serviço                           | Mailbox Evaluate All Automatic Rules    | Mailbox Evaluate Routing Rule |
| Frequência de avaliação           | Uma vez por mensagem, quando adicionada | Várias vezes, sob demanda     |
| Elegibilidade da mensagem         | Somente uma vez                         | Contínua                      |
| Agendamento                       | Controlado pelo sistema                 | Controlado pelo usuário       |
| Recuperação após queda do sistema | Automática                              | Reinício manual necessário    |

<div class="box atencao">

<span class="t">Regras automáticas simultâneas podem se bloquear</span>

Segundo a documentação, várias routing rules automáticas rodando ao
mesmo tempo podem bloquear umas às outras. A recomendação é usar regras
manuais ou implementar uma lógica de lock em torno das regras
automáticas.

</div>

<div class="box dica">

<span class="t">Boa prática</span>

Ao usar roteamento automático, evite manter na mailbox os arquivos já
processados (como `/TRANSMITIDOS` ou `/JA_ENTREGUE`), o que combina com
o modelo de avaliação única (veja o [tópico 13](#mailbox)). Se precisar
reavaliar as mesmas mensagens, use regras manuais.

</div>

</div>

<div id="filas" class="section card">

<span id="filas"></span>

## <span class="num">15</span> Queue (1 a 9): distribuição dos BPs entre as queues

<span class="nivel">Intermediário</span>

O ISBI executa os BPs em queues numeradas de **1 a 9**. Distribuir os
BPs entre elas, em vez de deixar tudo na queue padrão, melhora o uso das
threads e evita que um tipo de carga atrapalhe outro. Isso faz parte da
arquitetura de queues do ASI Engine e do Workflow.

### O problema de deixar tudo na queue padrão

Por padrão, todos os BPs padrão do sistema rodam na **queue 4**, e todo
BP novo também recebe a queue 4. Se você sempre mantiver o padrão,
conforme implementa mais BPs a queue 4 vai acumulando trabalho (*BP
queue build up*) e a execução dos seus BPs sofre atrasos quando muitos
BPs rodam ao mesmo tempo.

<figure>
<img src="assets/imagem-04.jpg" width="886"
alt="Tela de configuração da fila do BP" />
<figcaption>Tela onde a queue do BP é definida.</figcaption>
</figure>

<div class="box dica">

<span class="t">O que fazer</span>

Revise a configuração das queues e das threads atribuídas a cada uma, e
atribua seus BPs personalizados às queues mais adequadas à execução
deles.

</div>

### Por que distribuir

| Benefício              | O que acontece                                                                                                                                                                                                                  |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Performance            | O número de threads de cada queue (em cada node do cluster) afeta diretamente a performance. Queues diferentes podem ter alocações de threads diferentes, de acordo com a carga.                                                |
| Balanceamento de carga | Em cluster, os BPs são distribuídos entre os nodes conforme a profundidade da queue e as threads disponíveis. Isso depende também de quantos steps o BP executa antes de ser reagendado e, possivelmente, enviado a outro node. |
| Gestão de recursos     | BPs críticos ou de alta prioridade podem ficar em queues dedicadas, com mais threads. BPs de baixa prioridade ou em lote usam outras queues, sem prejudicar as operações críticas.                                              |
| Segregação de carga    | Um tipo de carga não monopoliza os recursos, e fica mais fácil monitorar e analisar cada categoria de BP.                                                                                                                       |

### Como escolher a queue de um BP

- Prioridade de processamento
- Tempo de execução esperado
- Necessidade de recursos
- Criticidade para o negócio

<div class="box info">

<span class="t">Quem define a queue de um BP</span>

Quem cria ou edita o BP no **Process Editor**. No passo 8 do assistente
de criação, preenchem-se os parâmetros de configuração do BP, entre
eles:

- Document Tracking
- Set onfault processing
- **Set Queue**, onde se informa o nível da queue
- Enable Async Start Mode
- Use BP Queuing (recomendado)
- Enable Transaction

</div>

<div class="box atencao">

<span class="t">Pontos de atenção</span>

O nível padrão do *Set Queue* é **4**. A queue escolhida precisa **já
estar alocada** (existir e ter threads) antes de ser atribuída a um BP.
Em instalações em cluster, a configuração de queues vale para todos os
nodes.

</div>

<div class="box dica">

<span class="t">Combine com a infraestrutura</span>

Definir a queue é papel do desenvolvedor, mas dimensionar as threads de
cada queue e acompanhar a profundidade das queues é tarefa de quem
administra o ambiente. Combine com essa equipe a divisão das queues
antes de publicar um BP novo.

O desenvolvedor é quem conhece o fluxo de execução dos BPs. Por isso,
ele consegue identificar qual seria a melhor distribuição entre as
queues: qual BP interfere em qual, quais projetos estão em cada queue e
qual queue recebe a maior demanda de execuções. Leve esse conhecimento
para a conversa com a equipe de infraestrutura.

</div>

</div>

<div id="chamadas" class="section card">

<span id="chamadas"></span>

## <span class="num">16</span> Otimize chamadas e aproveite os steps do fluxo

<span class="nivel">Intermediário</span>

**Prefira uma chamada que execute o trabalho completo a várias chamadas
pequenas e fragmentadas.**

Sempre que possível, aproveite os steps que já existem no fluxo para
**preparar, validar, enriquecer e organizar** os dados usados nas etapas
seguintes. O objetivo é evitar chamadas adicionais desnecessárias,
reduzir processamento redundante e deixar o fluxo mais simples e
performático.

**Regra de bolso:** junte o que é do *mesmo tipo e do mesmo escopo* (por
exemplo, transformar uma lista inteira com um único XSLT); separe o que
tem *responsabilidade, volume ou ciclo de vida diferente*. O foco é
**reduzir chamadas redundantes**, e não concentrar tudo em um único
step.

<div class="box info">

<span class="t">Exemplo prático</span>

Em vez de criar uma nova chamada só para consultar ou preparar uma
informação que será usada depois, aproveite um step que já é executado
para obter esse dado e disponibilizá-lo às próximas etapas.

</div>

<div class="box atencao">

<span class="t">Princípio, com equilíbrio</span>

Faça cada chamada trabalhar o máximo possível, **sem comprometer** a
modularidade, a legibilidade, a manutenção e a responsabilidade de cada
componente.

**Este tópico não contradiz os tópicos 5, 6 e 12; ele complementa:**

- **Tópicos [5](#loops) e [12](#longos):** falam do *tamanho do BP*
  (quantidade de steps e duração). Aqui o tema é o *número de chamadas
  dentro do trabalho*. Um BP pode processar em lotes (tópico 5) e,
  dentro de cada lote, usar poucas chamadas para a lista inteira (tópico
  16).
- **Tópico [6](#processdata):** juntar o trabalho em uma chamada só vale
  se o resultado for **filtrado** (6.2) e **limpo** com o Release (6.1).
  Uma chamada que traz tudo e mantém tudo no ProcessData gera um
  ProcessData grande, persistido no banco, o que se quer evitar.
- **Limite:** se juntar tudo deixar o BP muito longo ou o ProcessData
  grande, volte a dividir (lotes, sub-BPs) como nos tópicos 5 e 12.

</div>

### Caso real: atualizar usuários do B2B a partir do LDAP

Um processo atualiza a base do B2B com os usuários que vêm do LDAP. A
situação atual é:

1.  Um **SQL** traz a lista de grupos (no exemplo, 20 grupos).
2.  Para **cada grupo**, o BP chama um **XSLT** e depois consulta o
    **LDAP** para buscar os usuários do grupo (no exemplo, 10 usuários
    por grupo).
3.  Para **cada usuário**, o BP chama um **XSLT** que transforma o
    usuário no XML esperado pelo **serviço de import**, e então chama o
    import.

### Sugestão de otimização A: transformar a lista de uma vez

1.  Um SQL traz os grupos.
2.  Um **único XSLT** recebe a lista inteira e gera de uma vez as
    requisições no padrão que o serviço LDAP espera.
3.  O BP percorre as requisições em loop, uma chamada de LDAP por grupo.
4.  Para os usuários de cada grupo, um **único XSLT** formata todos os
    usuários (os 10) no padrão do serviço de import.
5.  O BP chama o import em loop, **um usuário por vez**, porque o
    serviço de import deve receber um de cada.

| Chamada                               | Situação atual       | Sugestão A                                                |
|---------------------------------------|----------------------|-----------------------------------------------------------|
| SQL de grupos                         | 1                    | 1                                                         |
| XSLT para montar a requisição do LDAP | 20 (um por grupo)    | 1 (lista inteira)                                         |
| Chamadas ao LDAP                      | 20                   | 20                                                        |
| XSLT para formatar usuários do import | 200 (um por usuário) | 20 (um por grupo, com os 10 usuários)                     |
| Chamadas ao serviço de import         | 200                  | 200                                                       |
| **Total de chamadas XSLT**            | **220**              | **21**                                                    |
| **Ganho**                             | \-                   | **199 chamadas XSLT a menos (redução de cerca de 90,5%)** |

<figure>
<p>[Gráfico ilustrativo: execuções por projeto nos últimos 30 dias. Os
dados são fictícios.]</p>
<figcaption>Ilustração com os números do exemplo (20 grupos e 10
usuários por grupo). Os valores reais dependem do seu
volume.</figcaption>
</figure>

### Sugestão de otimização B: um sub-BP assíncrono por grupo

Em vez de um BP único percorrer todos os grupos, o BP principal faz o
SQL e, **para cada grupo, dispara um sub-BP assíncrono**. Cada sub-BP
trata um grupo: consulta o LDAP, formata os usuários com um único XSLT e
chama o import em loop, um usuário por vez.

<div class="box dica">

<span class="t">Mais paralelismo com a distribuição no cluster</span>

Como são **sub-BPs assíncronos**, o **Workflow distribui** a execução
deles entre os **nodes do cluster** e os executa **em paralelo**. Isso
permite **mais paralelismo** do que um único BP percorrendo os grupos em
sequência: vários grupos são tratados ao mesmo tempo, aproveitando as
threads de mais de um node.

</div>

| Vantagem                                                                                                                       | Ponto de atenção                                                                                                            |
|--------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| Os grupos são processados em paralelo, e cada execução fica curta (veja os tópicos [5](#loops) e [12](#longos)).               | O paralelismo consome threads: planeje a [queue](#filas) usada pelos sub-BPs e o limite de quantos disparar ao mesmo tempo. |
| O ProcessData de cada sub-BP é pequeno, pois só recebe os dados do seu grupo (veja `message_to_child` no tópico [8](#invoke)). | O import grava na base do B2B: várias importações simultâneas podem disputar os mesmos recursos. Teste com volume real.     |
| Uma falha afeta só o grupo que falhou, e dá para reprocessar só ele.                                                           | É preciso um jeito de acompanhar o resultado de cada sub-BP, já que o BP principal não espera por eles.                     |

<div class="box dica">

<span class="t">Qual escolher</span>

A **sugestão A** é a mudança mais simples e já corta a maior parte das
chamadas, mantendo um único BP. A **sugestão B** pesa mais em desenho e
acompanhamento, mas escala melhor com muitos grupos. Elas também podem
ser combinadas: o BP principal dispara um sub-BP por grupo, e cada
sub-BP aplica a sugestão A.

</div>

<div class="box perigo">

<span class="t">Importante: paralelizar exige combinar a capacidade dos
serviços externos</span>

Ao paralelizar o processamento, o Sterling B2B passa a fazer **muitas
chamadas ao mesmo tempo** aos serviços externos. Esses serviços precisam
ser **combinados com o administrador** deles, para saber **quantas
chamadas simultâneas são suportadas** antes de aumentar o paralelismo.

Isso vale para serviços externos como **HTTP, FTP, SFTP e outros
protocolos** que têm **controle de sessões simultâneas** (limite de
conexões ou sessões ao mesmo tempo).

Sem esse alinhamento, o paralelismo pode esgotar o limite do destino e
gerar erros de conexão, recusas, timeouts ou bloqueios. O resultado é o
oposto do ganho esperado. Combine o número de sub-BPs simultâneos e a
taxa de chamadas com o administrador de cada serviço, e teste com volume
real.

</div>

<div class="box atencao">

<span class="t">Sobre este exemplo</span>

O caso do LDAP, os números (20 grupos e 10 usuários) e as duas sugestões
são uma proposta de desenho a partir do cenário descrito, não uma regra
da documentação IBM. Valide no seu ambiente se o serviço de import e o
LDAP aceitam o formato e o volume que você pretende enviar.

</div>

</div>

<div id="checklist" class="section card">

<span id="checklist"></span>

## Checklist rápido antes de publicar um BP

- O nome do BP segue o padrão da equipe (sigla, projeto, tipo,
  funcionalidade, descrição)?
- A persistência é a menor possível que ainda funciona? Testei os
  serviços?
- Existe *Client End Session* no fluxo normal **e** no `onFault`,
  chamado só depois de verificar que a sessão existe?
- Cada *Sequence Start* do BP tem um nome único (principal e `onFault`
  não repetem o nome)?
- Os loops têm limite de contador?
- Uso *Release* para limpar o ProcessData a cada iteração?
- Gravo só os campos necessários e dentro de um nó próprio (não na
  raiz)?
- Nenhum xPath começa com `//`?
- Nas chamadas Invoke, passo só o necessário com `message_to_child`?
- Estou usando os adapters e serviços atuais, e não os da *Retiring
  stencil*?
- Evitei adapters pesados (ex.: Command Line) quando um mais simples
  (File System ou Assign com xPath) resolve? Para tarefas realmente
  custosas (ex.: split, parse/Base64), avaliei delegar ao SO via Command
  Line?
- No File System Adapter, avaliei `attachFile` quando sei o nome exato
  do arquivo? Testei a performance?
- O diretório de coleta só tem arquivos que ainda precisam ser
  processados?
- Não há chamada HTTP escrita direto no Java Task (uso o HTTP
  Client/Server Adapter)?
- Se uso Java Task, a lógica está em JARs pré-compilados e o código
  inline é mínimo?
- Se a chamada for HTTPS, o CA foi importado no Dashboard e nos arquivos
  de segurança do Java, com reciclagem do ambiente?
- O BP evita Sleep (usa eventos, agendamento ou mensageria no lugar de
  espera ativa)?
- A duração do BP é curta? Dividi processos longos em partes menores?
- Cada chamada faz o trabalho completo (ex.: um XSLT para a lista
  inteira em vez de um por item), sem perder modularidade e
  legibilidade?
- Ao paralelizar (ex.: sub-BPs assíncronos), combinei com o
  administrador de cada serviço externo (HTTP, FTP, SFTP, LDAP etc.)
  quantas chamadas e sessões simultâneas ele suporta?
- O BP está na queue adequada à sua prioridade, ao tempo esperado de
  execução e à criticidade (e não só na queue 4 por padrão)? Essa queue
  já está alocada?
- Os arquivos já entregues saem da mailbox (arquivamento +
  MailboxDeleteService agendado), em vez de ficarem em `/TRANSMITIDOS`
  ou `/JA_ENTREGUE`?
- Escolhi o modo certo de routing rule (automática para mensagens novas,
  manual para reprocessar)? Evitei várias regras automáticas
  simultâneas?

Para se aprofundar, veja o webcast completo:
<a href="http://www-01.ibm.com/support/docview.wss?uid=swg27043594"
target="_blank">Melhores práticas de desenvolvimento de BPML</a>.

</div>

<div id="glossario" class="section card">

<span id="glossario"></span>

## Glossário rápido

| Termo                             | Significado                                                                                                                               |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| **ISBI**                          | IBM Sterling B2B Integrator, a plataforma onde os BPs rodam.                                                                              |
| **Nomenclatura padronizada**      | Padrão de nome único para todos os BPs, por exemplo `SIGLA_PROJETO_TIPO_FUNCIONALIDADE_DESCRICAO`.                                        |
| **BP / BPML**                     | Business Process, e a linguagem XML (Business Process Modeling Language) usada para descrevê-lo.                                          |
| **ProcessData**                   | Dados em XML mantidos durante a execução de um BP (também chamado de Instance Data).                                                      |
| **Persistência**                  | Quanto da execução é gravado no banco de dados.                                                                                           |
| **onFault**                       | Bloco do BPML executado quando ocorre um erro.                                                                                            |
| **xPath**                         | Linguagem para localizar nós em um XML.                                                                                                   |
| **wf.log**                        | Log do workflow engine em `install_dir/logs`; mostra o tempo de execução e os erros de cada BP.                                           |
| **Performance Statistics Report** | Relatório em *Operations \> System \> Performance \> Statistics* com invocações e tempos (mín., máx., média) por BP e atividade.          |
| **Retiring stencil**              | Grupo do Graphical Process Modeler com adapters e serviços em desuso.                                                                     |
| **Step**                          | Cada Service ou Adapter executado em um BP; segue o paradigma Input, Transformation e Output.                                             |
| **Java Task**                     | Serviço que executa código Java dentro do BP; o código é compilado a cada execução.                                                       |
| **attachFile**                    | Parâmetro do File System Adapter que aponta direto para um arquivo específico.                                                            |
| **Queue (1 a 9)**                 | Nível de execução em que o BP roda; define quantas threads atendem o BP. Definida no parâmetro *Set Queue* do Process Editor (padrão: 4). |
| **queueDepth**                    | Quantidade de BPs aguardando na queue para serem executados.                                                                              |
| **IBM Control Center Monitor**    | Ferramenta de monitoramento do ambiente. Chamadas HTTP feitas direto no Java Task não aparecem nela.                                      |
| **MailboxDeleteService**          | BP que remove mensagens da mailbox conforme a retenção configurada; deve ser agendado com regularidade.                                   |
| **Routing rule automática**       | Avaliada uma única vez por mensagem, pelo serviço Mailbox Evaluate All Automatic Rules.                                                   |
| **Routing rule manual**           | Avaliada sob demanda pelo serviço Mailbox Evaluate Routing Rule; a mensagem pode ser roteada várias vezes.                                |

</div>

<div id="referencias" class="section card">

<span id="referencias"></span>

## Referências

- <a href="https://www.ibm.com/support/pages/node/1121517"
  target="_blank">Business Process development and implementation Hints
  &amp; Tips for better performance (IBM Support, node 1121517)</a>
- <a
  href="https://www.ibm.com/docs/en/b2b-integrator/6.2.0?topic=troubleshooting-symptoms-causes-poor-business-process-execution-time"
  target="_blank">Symptoms and Causes of Poor Business Process Execution
  Time (IBM Docs 6.2.0)</a>
- <a
  href="https://www.ibm.com/docs/en/b2b-integrator/6.2.0?topic=adapters-example-creating-service"
  target="_blank">Example: Creating a Service (IBM Docs 6.2.0)</a>

</div>

</div>

</div>

Tradução e adaptação didática de fontes IBM:
<a href="https://www.ibm.com/support/pages/node/1121517"
target="_blank">Hints &amp; Tips for better performance (blog
técnico)</a> · <a
href="https://www.ibm.com/docs/en/b2b-integrator/6.2.0?topic=troubleshooting-symptoms-causes-poor-business-process-execution-time"
target="_blank">Symptoms and Causes of Poor Business Process Execution
Time</a> (IBM Sterling B2B Integrator 6.2.0) · © IBM Corp. · IBM Bob
