# Descrição

Este repositório contém código de amostra que ajuda você a criar uma codificação padrão para o modelo de programação de aplicativo ABAP RESTful (RAP) no ambiente SAP BTP, ABAP.

## Novo: Branch On-Premise-2020

Agora existe uma versão do Gerador RAP disponível que funciona em sistemas SAP S / 4HANA 2020 no local.

## Pré-requisitos

O Gerador RAP pode ser usado em
- Sistemas de ambiente SAP BTP ABAP
- SAP S / 4HANA 2020 FSP1 e superior

## O que há de novo com 2102

- todos os objetos de um serviço V4 habilitado para rascunho agora podem ser gerados e ativados. O RAP Generator gera as tabelas de rascunho para você.
- os serviços V4 habilitados para rascunho podem ser registrados automaticamente para o aplicativo de Gerenciar Configuração de Negócios
- chamar o RAP Generator como uma API ficou mais simples. Ele simplesmente se resume a três linhas de código
  <pre>
     DATA(json_string) = '<your json string>'.
     DATA(rap_generator) = NEW /dmo/cl_rap_generator( json_string ).
     DATA(todos) = rap_generator->generate_bo(  ).
  </pre>
- Agora você pode especificar uma solicitação de transporte que deve ser usada
- Você pode registrar seu OData V4, serviço habilitado para rascunho para uso com o aplicativo Manter configuração de negócios
- Você pode adicionar suporte para a API de transporte de gravação

## Motivação

A ideia básica por trás do * Gerador RAP * é tornar mais fácil para o desenvolvedor criar a pilha completa de objetos que são necessários para implementar um objeto de negócios RAP. O objetivo é gerar a maior parte da codificação padrão para que o desenvolvedor possa iniciar mais rapidamente a implementação da lógica de negócios.

A primeira fonte de dados suportada são as tabelas. Ao criar novas tabelas para cenários de campo verde, o uso de tabelas com chaves baseadas em uuid é recomendado, de modo que um cenário gerenciado onde nenhum código precise ser implementado para as operações CRUD e numeração inicial possa ser usado.
Resta ao desenvolvedor implementar determinações, validações e ações.

Para cenários brownfield onde a lógica de negócios existente existe para criar, atualizar e excluir dados de negócios, um cenário não gerenciado pode ser gerado.

Como uma segunda fonte de dados, o gerador RAP agora também suporta visualizações CDS. Desta forma, será possível criar objetos de negócios RAP baseados em visualizações CDS existentes.

Para tornar o uso da ferramenta o mais fácil possível, a entrada necessária para o gerador pode ser fornecida como um arquivo JSON.
Uma amostra simples de um arquivo JSON que geraria um objeto de negócios gerenciado com base nas duas tabelas / dmo / a_travel_d e / dmo / a_booking_d teria a seguinte aparência.
<pre>
{
  "implementationType": "managed_uuid",
  "namespace": "Z",
  "suffix": "_####",
  "prefix": "RAP_",
  "package": "ZRAP_####",
  "datasourcetype": "table",
  "bindingtype": "odata_v4_ui",
  "draftenabled": true,  
  "hierarchy": {
    "entityName": "SalesOrder",
    "dataSource": "/dmo/a_travel_d",
    "drafttable": "zdtravel_d_####",
    "objectId": "travel_id",
    "uuid": "travel_uuid",
    "lastChangedAt": "last_changed_at",
    "lastChangedBy": "local_last_changed_by",
    "createdAt": "local_created_at",
    "createdBy": "local_created_by",
    "localInstanceLastChangedAt": "local_last_changed_at",
    "children": [
      {
        "entityName": "Booking",
        "dataSource": "/dmo/a_booking_d",
        "drafttable": "zdbook_d_####",
        "objectId": "booking_id",
        "uuid": "booking_uuid",
        "parentUuid": "parent_uuid",
        "localInstanceLastChangedAt": "local_last_changed_at"
      }
    ]
  }
}
</pre>

## Como usar o Gerador RAP (nos sistemas de teste)

O pacote / DMO / RAP_Generator foi importado para todos os sistemas de teste para sua conveniência.

Esta é uma breve descrição de como o Gerador RAP pode ser usado.
1. Certifique-se de ter definido a seguinte opção "Quebra e escape de texto ao colar em literal de string" para seu editor de código-fonte ABAP em suas preferências de ADT, conforme descrito em meu blog [Como encapsular strings longas automaticamente em ADT] (https: / /blogs.sap.com/2020/07/29/how-to-wrap-long-strings-automatically-in-adt/)
2. Crie uma classe ** zcl_rap_generator_console _ #### ** em seu pacote usando o seguinte código como modelo:
<pre>
CLASS zcl_rap_generator_console_#### DEFINITION
  PUBLIC
  INHERITING FROM cl_xco_cp_adt_simple_classrun
  FINAL
  CREATE PUBLIC .
  PUBLIC SECTION.
  PROTECTED SECTION.
    METHODS main REDEFINITION.
  PRIVATE SECTION.
ENDCLASS.

CLASS ZCL_RAP_GENERATOR_CONSOLE_#### IMPLEMENTATION.
  METHOD main.
    DATA(json_string) = '{ "AddYourJsonFileHere" : true }'.
    DATA(rap_generator) = NEW /dmo/cl_rap_generator( json_string ).
    DATA(todos) = rap_generator->generate_bo(  ).
    DATA(rap_bo_name) = rap_generator->root_node->rap_root_node_objects-service_binding.
    out->write( |RAP BO { rap_bo_name }  generated successfully| ).
    out->write( |Todo's:| ).
    LOOP AT todos INTO DATA(todo).
      out->write( todo ).
    ENDLOOP.
  ENDMETHOD.
ENDCLASS.
</pre>
4. Copie a string json mostrada acima ou uma das strings json para os diferentes cenários, que você pode encontrar na pasta [json_templates] (../../ tree / master / json_templates) entre as duas aspas simples.
 <pre>DATA(json_string) = <b>''</b>.</pre>
 5. Substitua as hastags <b> #### </b> que são usadas como espaço reservado por strings apropriadas para que se ajustem ao nome do seu pacote e ao sufixo que você deseja usar.
6. Execute a classe usando F9



A classe herda da classe ** cl_xco_cp_adt_simple_classrun ** que é fornecida pela estrutura XCO. Esta classe capturará todas as exceções lançadas pelo Gerador RAP e mostrará a pilha de chamadas conforme você está acostumado com ela pelo ADT.

Uma descrição muito mais detalhada (incluindo capturas de tela) pode ser encontrada em meu blog a seguir [The RAP Generator] (https://blogs.sap.com/2020/05/17/the-rap-generator/).
   
Em vez disso, movi a descrição dos detalhes técnicos desta ferramenta para este arquivo readme.md.

## Cenários suportados

O gerador suporta vários cenários listados nesta tabela:

<table style="width:100%">
  <tr>
    <th>implementation type</th>
    <th>key type</th>
    <th>datasource types</th>
    <th>Comment</th>
  </tr>
  <tr>
    <td>managed</td>
    <td>uuid</td>
    <td>table</td>
    <td>Green field scenario with simple data structure</td>
  </tr>
  <tr>
    <td>managed</td>
    <td>semantic key</td>
    <td>table</td>
    <td>Requires external numbering</td>
  </tr>
  <tr>
    <td>unmanaged</td>
    <td>semantic key</td>
    <td>table</td>
    <td>Requires external numbering</td>
  </tr>
   <tr>
    <td>managed</td>
    <td>uuid</td>
    <td>cds view</td>
    <td>no mapping can be generated</td>
  </tr>
  <tr>
    <td>managed</td>
    <td>semantic key</td>
    <td>cds view</td>
    <td>no mapping can be generated</td>
  </tr>
  <tr>
    <td>unmanaged</td>
    <td>semantic key</td>
    <td>cds view</td>
    <td>no mapping can be generated</td>
  </tr>
</table>

## Parâmetros do arquivo JSON

O arquivo JSON contém algumas propriedades obrigatórias que são necessárias para a geração dos objetos de repositório.
O nó tem um esquema que contém uma matriz chamada filhos, cada um dos quais também são instâncias de nó.
Dessa forma, podemos modelar um nó raiz incluindo seus nós filho e neto de uma forma que seja legível e reutilizável pelo desenvolvedor.
Vamos começar com a explicação das propriedades (obrigatórias) do próprio objeto de negócios.

### Arquivos de esquema JSON

Neste repositório, agora há uma nova pasta que contém esquemas JSON que descrevem quais parâmetros são obrigatórios para determinados cenários.

[json_Schemas](../../tree/master/json_schemas) 

### Parâmetros obrigatórios do root node

#### "implementationType" 

O gerador atualmente suporta três tipos de implementação
-	managed_uuid
-	managed_semantic_key
-	unmanaged_semantic_key

Se o tipo de implementação ** managed_uuid ** for usado, o gerador irá gerar um objeto de negócios gerenciado que usa numeração interna. Portanto, é necessário que os campos-chave dos nós e, portanto, também os campos-chave das tabelas subjacentes sejam do tipo raw (16) (UUID).

<pre>
key client      : abap.clnt not null;
key uuid        : sysuuid_x16 not null;
</pre>

Se um dos cenários ** managed_semantic_key ** ou ** unmanaged_semantic_key ** for usado, o gerador espera que haja uma hierarquia de tabelas onde a tabela de cabeçalho sempre contém todos os campos-chave da tabela de itens.

- Travel
<pre>
key client                : abap.clnt not null;
key travel_id             : /dmo/travel_id not null;
</pre>
- Booking
<pre>
key client                : abap.clnt not null;
key travel_id             : /dmo/travel_id not null;
key booking_id            : /dmo/booking_id not null;
</pre>
- BookingSupplements
<pre>
key client                : abap.clnt not null;
key travel_id             : /dmo/travel_id not null;
key booking_id            : /dmo/booking_id not null;
key booking_supplement_id : /dmo/booking_supplement_id not null;
</pre>

Quando o tipo de implementação ** managed_semantic_key ** é escolhido, o gerador irá gerar um objeto de negócios que usa uma implementação gerenciada que requer numeração externa enquanto ** unmanaged_semantic_key ** irá gerar um objeto de negócios que usa uma implementação não gerenciada.

#### “namespace”

Aqui você deve especificar o namespace dos objetos de repositório. Normalmente, seria o valor “Z” ou seu próprio namespace, caso você tenha registrado um.

#### "package"

Com o parâmetro “pacote” você deve fornecer o nome de um pacote onde todos os objetos de repositório de seu objeto de negócios RAP serão gerados.

#### "datasourceType"

O gerador oferece suporte a tabelas e visualizações CDS como fonte de dados.
Observe que ao iniciar a partir de tabelas, o gerador também será capaz de gerar um mapeamento, enquanto um mapeamento deve ser criado manualmente pelo desenvolvedor ao iniciar com visualizações CDS como fontes de dados. Você deve fornecer um dos seguintes valores aqui:
- tabela
- cds_view

### Parâmetros opcionais do root node

#### "draftenabled"

<pre> "draftenabled" : true </pre>

Usando o parâmetro booleano ** draftenabled **, você pode especificar que o objeto RAP gerado oferece suporte a rascunho.

Observe que, para um cenário habilitado para rascunho, você deve especificar os nomes das tabelas de rascunho para cada nó "node" da árvore de composição.

#### "transportrequest"

<pre> "transportrequest" : "SIDK900075" </pre>

Agora você pode fornecer o nome de uma solicitação de transporte que deve ser usada para todos os objetos que estão sendo gerados. Se nenhuma solicitação de transporte for especificada, o Gerador RAP primeiro procurará por qualquer transporte modificável que se ajuste à camada de transporte do pacote que pertence ao desenvolvedor.

Caso contrário, esse transporte é encontrado, uma nova solicitação de transporte está sendo criada.

#### “sufixo” e “prefixo”
Esses são parâmetros opcionais que podem ser usados para ajustar os nomes dos objetos de repositório.

A convenção de nomenclatura usada pelo gerador segue as convenções de nomenclatura propostas pelo Virtual Data Model (VDM) usado no SAP S / 4 HANA.
Por exemplo, o nome de uma visualização de interface do CDS seria gerado a partir das propriedades mencionadas acima da seguinte maneira:
`DATA(lv_name) = |{ namespace }I_{ prefix }{ entityname }{ suffix }|.`

O nome da entidade que faz parte do nome do objeto do repositório é definido pela propriedade ** “entityName” ** no nível do nó (veja abaixo).

#### addToManageBusinessConfiguration

<pre>"addToManageBusinessConfiguration" : true</pre>

Ao definir o parâmetro booleano addToManageBusinessConfiguration como true, o RAP Generator tentará registrar sua vinculação de serviço habilitada para rascunho OData V4 para que possa ser usada usando o aplicativo Fiori * Maintain Business Configurations *.

se não for informado de outra forma, o Gerador RAP tentará usar alguns valores padrão para * Nome *, * Identificador * e * Descrição * para registrar sua ligação de serviço no Aplicativo Gerenciar Configuração de Negócios. O * Nome * e a * Descrição * podem ser alterados posteriormente usando uma API, enquanto o * Identificador * como um campo-chave não pode ser alterado posteriormente.

SE quiser anular as configurações escolhidas pelo Gerador RAP, você pode definir os seguintes valores:

<pre>
       "manageBusinessConfigurationName" : "My MBC name",
       manageBusinessConfigurationIdentifier : "ZMY_MBC_IDENTIFIER",
       manageBusinessConfigurationDescription : "My MBC Description",
</pre>       
       
### Propriedades obrigatórias de objetos de nó
Para cada objeto de nó deve especificar as seguintes propriedades obrigatórias

#### “entityName”
Here you have to specify the name of your entity (e.g. “Travel” or “Booking”). The name of the entity becomes part of the names of the repository objects that will be generated and it is used as the name of associations (e.g. "_Travel").
Please note, that the value of “entityName” must be unique within a business object.

#### “dataSource” e “dataSourceType”
O gerador suporta os tipos de fonte de dados “table” e "cds_view".
O nome da fonte de dados é o nome da tabela subjacente ou o nome da exibição cds subjacente.

#### “objectId”
Com ** objectId ** denotamos um campo de chave semântica que faz parte da fonte de dados (tabela ou visualização cds).
Em nosso cenário de viagem / reserva, este seria o campo ** travel_id ** para a entidade Viagem e ** booking_id ** para a entidade Reserva se a fonte de dados fosse tabelas e seria ** travelid ** e ** bookingid * * se as visualizações do CDS do cenário de demonstração de vôo forem usadas.
Para cenários gerenciados, o gerador irá gerar uma determinação para cada objectid.

Você também deve especificar um ** objectid ** para cenários semânticos.


#### “uuid”, “parent_uuid”, “root_uuid”
Em um cenário gerenciado que usa chaves do tipo ** uuid **, as tabelas de um nó filho devem conter um campo onde a chave da entidade pai é armazenada.
Os nós netos e seus filhos devem, além disso, armazenar os valores dos campos-chave do pai e da entidade raiz.
Isso é necessário, entre outros, para o mecanismo de travamento.
O gerador por padrão espera as seguintes convenções de nomenclatura para esses campos

- uuid
- parent_uuid
- root_uuid
<br>
If you don’t want to use the same field names in all tables and prefer more descriptive names, such as
<pre>
key travel_uuid       : <b>sysuuid_x16</b> not null;
</pre>
and
<pre>
key booking_uuid      : <b>sysuuid_x16</b> not null;
    travel_uuid       : <b>sysuuid_x16</b> not null;
</pre>
você deve especificar esses nomes de campo na definição do nó, fornecendo valores para `uuid` e` parentUuid` na definição da entidade filha e para `uuid` na definição da entidade root.

<pre>
...
  "node": {
    "entityName": "Travel",
    "dataSource": "zrap_atrav_0002",
    "dataSourceType" : "table",
    "objectId": "TRAVEL_ID",
    <b>"uuid": "travel_uuid",</b>
    "children": [
      {
        "entityName": "Booking",
        "dataSource": "zrap_abook_0002",
        "dataSourceType" : "table",
        "objectId": "BOOKING_ID",
        <b>"uuid": "booking_uuid",
        "parentUuid": "travel_uuid"</b>
      }
    ]
  }
...
</pre>

#### "lastChangedAt",  "lastChangedBy",  "createdAt", "createdBy", "localInstanceLastChangedAt" and "localInstanceLastChangedBy"

Em um cenário gerenciado, é necessário que a entidade raiz forneça campos para armazenar dados administrativos quando uma entidade foi criada e alterada e por quem essas ações foram executadas.
Novamente, o gerador assume alguns valores padrão para esses nomes de campo, a saber:

- “last_changed_at",
- "last_changed_by,
- "created_at",
- "created_by" and
- "local_last_changed_at"
- "local_last_changed_by"
<br>

Se as tabelas que você está usando não seguem essa convenção de nomenclatura, é possível informar ao gerador sobre os nomes de campo reais definindo essas propriedades opcionais.

Um bom exemplo é a tabela que é usada no cenário de rascunho de referência de vôo, onde precisamos do seguinte mapeamento

<pre>
    "lastChangedAt": "last_changed_at",
    "lastChangedBy": "local_last_changed_by",
    "createdAt": "local_created_at",
    "createdBy": "local_created_by",
    "localInstanceLastChangedAt": "local_last_changed_at",
</pre>

Observe:
Tecnicamente obrigatórios são apenas os campos
last_changed_at (usado como etag total)
local_last_changed_at (usado como etag para o respectivo node)
No entanto, é uma boa prática adicionar todos os campos no nível do nó raiz e pelo menos os carimbos de data / hora no nível do sub node.

### Parâmetros opcionais para o node

#### drafttable
Quando você especifica que um objeto de negócios RAP deve suportar rascunho usando o parâmetro ** "draftenabled": true ** você deve especificar o nome da tabela de rascunho que está sendo gerada para cada nó usando a seguinte sintaxe

<pre>
"drafttable": "zd_book_0000",
</pre>

## Parâmetros opcionais para cenários de workshop

Os parâmetros a seguir foram implementados para que seja possível criar objetos de negócios RAP incluindo um mapeamento (se as visualizações CDS forem usadas como uma fonte de dados) e incluindo associações e ajudas de valor.

Isso é útil se objetos idênticos devem ser criados para treinamentos ou workshops.

Ao usar esses parâmetros, os arquivos json se tornarão mais complicados. Como resultado, o uso desses parâmetros não é recomendado se você deseja desenvolver um único objeto RAP. Eles serão usados em workshops, como sessões de TechEd, CodeJams ou cursos OpenSAP, onde há a necessidade de fornecer aos participantes objetos de negócios RAP completos como ponto de partida.

#### mapping
Usando este parâmetro, você pode fornecer o mapeamento entre os nomes de campo da exibição CDS e os nomes de campo usados pela lógica de negócios legada se as exibições CDS forem usadas como uma fonte de dados.
Ao usar tabelas como fontes de dados, esse mapeamento é gerado pelo gerador.
Quando as visualizações CDS são usadas como uma fonte de dados, esse mapeamento é criado manualmente pelo desenvolvedor, se não tiver sido definido.

<pre>
{
  "implementationType": "unmanaged_semantic",
  "namespace": "Z",
  "suffix": "_####",
  "prefix": "RAP_",
  "package": "ZRAP_####",
  "datasourcetype": "cds_view",
  "hierarchy": {
    "entityName": "Travel",
    "dataSource": "/DMO/I_Travel_U",
    "objectId": "TravelID",
    "persistenttable": "/dmo/travel",    
    "lastchangedat": "lastchangedat", 
    <b>"mapping": [
      {
        "dbtable_field": "TRAVEL_ID",
        "cds_view_field": "TravelID"
      },
      {
        "dbtable_field": "AGENCY_ID",
        "cds_view_field": "AgencyID"
      },
      {
        "dbtable_field": "CUSTOMER_ID",
        "cds_view_field": "CustomerID"
      },
      {
        "dbtable_field": "BEGIN_DATE",
        "cds_view_field": "BeginDate"
      },
      {
        "dbtable_field": "BOOKING_FEE",
        "cds_view_field": "BookingFee"
      },
      {
        "dbtable_field": "TOTAL_PRICE",
        "cds_view_field": "TotalPrice"
      },
      {
        "dbtable_field": "CURRENCY_CODE",
        "cds_view_field": "CurrencyCode"
      },
      {
        "dbtable_field": "DESCRIPTION",
        "cds_view_field": "Description"
      },
      {
        "dbtable_field": "STATUS",
        "cds_view_field": "Status"
      },
      {
        "dbtable_field": "LASTCHANGEDAT",
        "cds_view_field": "Lastchangedat"
      }
    ],</b>   
    "children": [
      {
        "entityName": "Booking",
        "dataSource": "/DMO/I_Booking_U",
        "objectId": "BookingID",
        "persistenttable": "/dmo/booking"      
      }
    ]
  }
}
</pre>

#### “associations” e “valuehelps”
No nível do nó, também é possível definir informações para gerar ajudas de valor e associações.
Essas propriedades foram introduzidas principalmente para configurações em cursos em que se gostaria que os participantes pudessem gerar um objeto de negócios que contivesse exatamente aquelas associações e definições de ajuda de valor necessárias para o curso.
Embora também possa ser útil para outros cenários, você vê que a complexidade do seu arquivo JSON aumentará e pode ser mais simples codificar isso manualmente nas visualizações CDS que são geradas pelo gerador RAP.

##### “associations”
Associações é uma matriz de objetos em que cada objeto representa uma associação.
Uma associação precisa de várias propriedades
- "name" is the name of the association and its name must start with an underscore '_'.
- "target" is the name of the CDS view that is the target of your associaton
- "cardinality" here you can enter one of the following values
  - "zero_to_one" [0..1]
  - "one" [1]
  - "zero_to_n" [0..n]
  - "one_to_n" [1..n]
  - "one_to_one" [1..1]
- "conditions" is again an array of objects with two properties
  - "projectionField" this is the field ´"$projection.*AgencyID* = _Agency.AgencyID"´
  - "associationField" this is the field ´"$projection.AgencyID = _Agency.*AgencyID*"´
 

```
    "associations": [
      {
        "name": "_Agency",
        "target": "/DMO/I_Agency",
        "cardinality": "zero_to_n",
        "conditions": [
          {
            "projectionField": "AgencyID",
            "associationField": "AgencyID"
          }
        ]
      },
      {
        "name": "_Customer",
        "target": "/DMO/I_Customer",
        "cardinality": "zero_to_n",
        "conditions": [
          {
            "projectionField": "CustomerID",
            "associationField": "CustomerID"
          }
        ]
      }
    ]

```

##### valueHelps

Valuehelps também é uma matriz de objetos em que cada objeto representa uma ajuda de valor. Cada objeto pode conter uma matriz adicional que contém as informações para a ligação adicional.
- "alias" é o alias da ajuda de valor usada na definição do serviço.
- "nome" é o nome da visão CDS usada como uma ajuda de valor
- "localelement" indica o nome do campo em sua visualização de projeção. Uma vez que este já é o nome do campo CDS, você deve estar ciente das conversões de nomenclatura que são usadas pelo gerador quando ele cria campos cds a partir dos nomes de campo ABAP subjacentes, ou seja, usando a conversão em notação camelCase.

<pre>
      @Consumption.valueHelpDefinition: [{ entity : {name: '/DMO/I_Agency', element: 'AgencyID'  } }]
      <b>AgencyID</b>,
</pre>

- "element" this is the field name in the CDS view that is used as a value help. 

<pre>
      @Consumption.valueHelpDefinition: [{ entity : {name: '/DMO/I_Agency', element: '<b>AgencyID</b>'  } }]
      AgencyID,
</pre>

- "additionalBinding" is again an array of objects with two properties
- "localElement" these are the following fields 
 
 <pre>
      @Consumption.valueHelpDefinition: [ {entity: {name: '/DMO/I_Flight', element: 'ConnectionID'},
                                                 additionalBinding: [ { localElement: '<b>FlightDate</b>',   element: 'FlightDate'},
                                                                      { localElement: '<b>AirlineID</b>',    element: 'AirlineID'},
                                                                      { localElement: '<b>FlightPrice</b>',  element: 'Price', usage: #RESULT},
                                                                      { localElement: '<b>CurrencyCode</b>', element: 'CurrencyCode' } ] } ]
</pre>                                                                                                                                        
                                                                   
 - "element" these are the following fields
 
<pre>
      @Consumption.valueHelpDefinition: [ {entity: {name: '/DMO/I_Flight', element: 'ConnectionID'},
                                                 additionalBinding: [ { localElement: 'FlightDate',   element: '<b>FlightDate</b>'},
                                                                      { localElement: 'AirlineID',    element: '<b>AirlineID</b>'},
                                                                      { localElement: 'FlightPrice',  element: '<b>Price</b>', usage: #RESULT},
                                                                      { localElement: 'CurrencyCode', element: '<b>CurrencyCode</b>' } ] } ]
                                                                      
</pre>                                                                  

 - "usage" these are the following (optinonal) fields in a valuehelp
 
<pre>
      @Consumption.valueHelpDefinition: [ {entity: {name: '/DMO/I_Flight', element: 'ConnectionID'},
                                                 additionalBinding: [ { localElement: 'FlightDate',   element: 'FlightDate'},
                                                                      { localElement: 'AirlineID',    element: 'AirlineID'},
                                                                      { localElement: 'FlightPrice',  element: 'Price', usage: <b>#RESULT</b>},
                                                                      { localElement: 'CurrencyCode', element: 'CurrencyCode' } ] } ]
                                                                      
</pre> 

Para gerar uma ajuda de valor conforme mencionado acima, a seguinte entrada teria que ser adicionada a um objeto de nó, aqui a entidade de reserva.

<pre>      
    "valueHelps": [
          {
            "alias": "Flight",
            "name": "/DMO/I_Flight",
            "localElement": "ConnectionID",
            "element": "ConnectionID",
            "additionalBinding": [
              {
                "localElement": "ConnectionID",
                "element": "ConnectionID"
              },
              {
                "localElement": "CarrierID",
                "element": "AirlineID"
              },
              {
                "localElement": "ConnectionID",
                "element": "ConnectionID"
              }
            ]
          },
          {
            "alias": "Currency",
            "name": "I_Currency",
            "localElement": "CurrencyCode",
            "element": "Currency"
          }
        ]
      }
    ]

</pre>

#### objectswithadditionalfields
Ao usar o Gerador RAP em workshops, descobriu-se que as visualizações CDS também devem conter campos que não fazem parte da fonte de dados subjacente, como campos que são recuperados por meio de uma associação.

<pre>
_Customer.LastName as CustomerName,
</pre>

Se campos adicionais forem adicionados, isso deve ser feito em vários locais, nomeadamente

- cds_interface_view
- cds_projection_view
- draft_table

É necessário especificar o nome do campo, o alias e, opcionalmente, a palavra-chave * localizado *.

Portanto, o arquivo JSON deve conter uma matriz ** objectswithadditionalfields **.

Para cada tipo de objeto, uma matriz adicional chamada ** additionalfields ** deve ser preenchida, contendo os campos adicionais para cada objeto.

Observe que os nomes de campo para campos adicionais na tabela de rascunho devem corresponder ao nome do campo (ou o nome do alias) do campo na visualização da interface.

<pre>

 "objectswithadditionalfields": [
      {
        "object": "cds_projection_view",
        "additionalfields": [
          {
            "fieldname": "_HolidayTxt.HolidayDescription",
            "alias": "HolidayDescription",
            "localized": true
          },
          {
            "fieldname": "_DeprecationText.ConfignDeprecationCodeName",
            "alias": "DeprecationDescription",
            "localized": true
          },
          {
            "fieldname": "Criticality",
            "alias": "Criticality"
          }
        ]
      },
      {
        "object": "cds_interface_view",
        "additionalfields": [
          {
            "fieldname": "case overall_status when ' ' then 2  when 'N' then 2  when 'I' then 0   when 'P' then 0   when 'D' then 3   when 'X' then 1   else 0    end",
            "alias": "Criticality"
          }
        ]
      },
      {
        "object": "draft_table",
        "additionalfields": [
          {
            "fieldname": "Criticality",
            "builtintype": "int1"
          }
        ]
      }
    ],

</pre>

# Requirements

Este código de amostra atualmente só funciona
  a) em SAP BTP, ambiente ABAP onde a estrutura XCO foi habilitada e
  b) em sistemas SAP S / 4HANA no local a partir da versão 2020 FPS1 (porque OData V4 só é compatível com FPS1)

Certifique-se de ter definido a seguinte opção "Quebrar e escapar texto ao colar em literal de string" para seu editor de código-fonte ABAP em suas preferências de ADT, conforme descrito em meu blog [Como quebrar strings longas automaticamente em ADT] (https: // blogs .sap.com / 2020/07/29 / how-to-wrap-long-strings-automatically-in-adt /)

Para obter informações mais detalhadas, verifique também a seguinte postagem do blog:
https://blogs.sap.com/2020/05/17/the-rap-generator

# Download e Instalação

O código de amostra pode simplesmente ser baixado usando o plug-in abapGIT nas Ferramentas de Desenvolvimento ABAP no Eclipse ao trabalhar com SAP BTP, Ambiente ABAP.
Para isso, você deve criar um pacote no namespace Z (por exemplo ZRAP_GENERATOR) e vinculá-lo como um repositório abapGit.

# Problemas conhecidos

O código de amostra é fornecido "no estado em que se encontra".

A versão local da estrutura XCO ("Componentes de extensão") em 2020 não oferece todos os recursos disponíveis na versão em nuvem.

# Como obter suporte
Se tiver problemas ou perguntas, você pode [publicá-los na comunidade SAP] (https://answers.sap.com/questions/ask.html) usando a tag primária "[SAP BTP, ambiente ABAP] (https: / /answers.sap.com/tags/73555000100800001164) "ou" [ABAP RESTful Application Programming Model] (https://answers.sap.com/tags/7e44126e-7b27-471d-a379-df205a12b1ff) ".

# Contribuindo
Este projeto é atualizado apenas por funcionários SAP.

# License
Copyright (c) 2020 SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE](LICENSE) file.
