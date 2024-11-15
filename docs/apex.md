# Documentação da Classe Apex - **ReceitaWSRequest** , **ReceitaWSResponse** , **ReceitaWSService** **ReceitaWSHttpCalloutMock** e **ReceitaWSServiceTest**

Este documento descreve a funcionalidade da classe Apex `ReceitaWSRequest`, `ReceitaWSResponse`, e `ReceitaWSService` responsáveis por consultar a API ReceitaWS e retornar informações de CNPJ para o Salesforce.

## **Classe `ReceitaWSRequest`**

A classe `ReceitaWSRequest` é usada para representar a solicitação que será enviada à API ReceitaWS, contendo um único parâmetro: o CNPJ.

### Atributos:
- **cnpj** (String) - O CNPJ que será consultado na API. Esse campo é obrigatório e deve ser passado para o método invocado.

```apex
public with sharing class ReceitaWSRequest {
    @InvocableVariable(label='CNPJ' required=true)
    public String cnpj;
}

```

## **Classe `ReceitaWSResponse`**

A classe `ReceitaWSResponse` é usada para armazenar a resposta da consulta à API ReceitaWS. Ela contém os dados retornados pela API e o status da consulta.

### Atributos:

- **status** (`String`)  
  O status da consulta (Ex: 'Erro', 'Sucesso').

- **mensagemErro** (`String`)  
  Mensagem de erro, caso haja algum problema.

- **nome** (`String`)  
  Razão social da empresa.

- **cnpj** (`String`)  
  CNPJ da empresa.

- **logradouro** (`String`)  
  Logradouro do endereço.

- **numero** (`String`)  
  Número do endereço.

- **complemento** (`String`)  
  Complemento do endereço.

- **bairro** (`String`)  
  Bairro do endereço.

- **municipio** (`String`)  
  Município do endereço.

- **uf** (`String`)  
  Estado do endereço.

- **cep** (`String`)  
  CEP do endereço.

- **telefone** (`String`)  
  Telefone da empresa.

- **situacao** (`String`)  
  Situação cadastral do CNPJ.

### Exemplo de Código:

```apex
public with sharing class ReceitaWSResponse {
    @InvocableVariable(label='Status')
    public String status;
    
    @InvocableVariable(label='Mensagem de Erro')
    public String mensagemErro;
    
    @InvocableVariable(label='Nome')
    public String nome;
    
    @InvocableVariable(label='CNPJ')
    public String cnpj;
    
    @InvocableVariable(label='Logradouro')
    public String logradouro;
    
    @InvocableVariable(label='Número')
    public String numero;
    
    @InvocableVariable(label='Complemento')
    public String complemento;
    
    @InvocableVariable(label='Bairro')
    public String bairro;
    
    @InvocableVariable(label='Município')
    public String municipio;
    
    @InvocableVariable(label='UF')
    public String uf;
    
    @InvocableVariable(label='CEP')
    public String cep;
    
    @InvocableVariable(label='Telefone')
    public String telefone;
    
    @InvocableVariable(label='Situação Cadastral')
    public String situacao;
}
````
## **Classe `ReceitaWSService`**

A classe `ReceitaWSService` é responsável por realizar a consulta à API ReceitaWS utilizando o CNPJ fornecido, processar a resposta e retornar os dados relevantes. Ela inclui métodos para realizar chamadas HTTP para a API e mapear a resposta para objetos da classe `ReceitaWSResponse`.

### Método: `consultarReceita`

O método **consultarReceita** é responsável por fazer a consulta à API ReceitaWS, processar a resposta e retornar um objeto `ReceitaWSResponse` com os dados ou mensagens de erro.

#### Parâmetros:

- **requests** (`List<ReceitaWSRequest>`)  
  Lista de objetos `ReceitaWSRequest` contendo os CNPJs a serem consultados.

#### Retorno:

- **resultsList** (`List<ReceitaWSResponse>`)  
  Retorna uma lista de objetos `ReceitaWSResponse`, cada um representando o resultado da consulta para um CNPJ.

#### Exemplo de Código:

```apex
public with sharing class ReceitaWSService {

    @InvocableMethod(label='Consultar ReceitaWS' description='Consulta a API ReceitaWS para obter informações de um CNPJ' callout=true)
    public static List<ReceitaWSResponse> consultarReceita(List<ReceitaWSRequest> requests) {
        List<ReceitaWSResponse> resultsList = new List<ReceitaWSResponse>();

        for (ReceitaWSRequest request : requests) {
            ReceitaWSResponse result = new ReceitaWSResponse();

            // Remover pontuação do CNPJ diretamente no Apex
            String cnpjLimpo = request.cnpj.replaceAll('[^0-9]', '').trim(); // Remove tudo que não for número

            // Validação do CNPJ
            if (String.isBlank(cnpjLimpo) || cnpjLimpo.length() != 14 || !cnpjLimpo.isNumeric()) {
                result.status = 'Erro';
                result.mensagemErro = 'CNPJ inválido';
                resultsList.add(result);
                continue;
            }
            
            // Realiza a consulta à API com o CNPJ limpo
            Http http = new Http();
            HttpRequest httpRequest = new HttpRequest();
            httpRequest.setEndpoint('https://www.receitaws.com.br/v1/cnpj/' + cnpjLimpo);
            httpRequest.setMethod('GET');
            
            HttpResponse response = http.send(httpRequest);
            
            // Processa a resposta da API
            if (response.getStatusCode() == 200) {
                // Deserializa a resposta JSON
                Map<String, Object> apiResults = (Map<String, Object>) JSON.deserializeUntyped(response.getBody());
                
                // Verifica se o status da API indica um erro
                if (apiResults.containsKey('status') && (String) apiResults.get('status') == 'ERROR') {
                    result.status = 'Erro';
                    result.mensagemErro = apiResults.containsKey('message') ? (String) apiResults.get('message') : 'CNPJ não encontrado ou inválido.';
                } else {
                    // Mapeia os dados quando a API retorna um resultado válido
                    result = mapearDados(apiResults);
                    result.status = 'Sucesso';
                    result.mensagemErro = 'CNPJ consultado com sucesso!';
                }
            } else if (response.getStatusCode() == 400) {
                result.status = 'Erro';
                result.mensagemErro = 'CNPJ não encontrado ou inválido.';
            } else if (response.getStatusCode() == 429) {
                result.status = 'Erro';
                result.mensagemErro = 'Limite de chamadas à API excedido. Tente novamente mais tarde.';
            } else {
                result.status = 'Erro';
                result.mensagemErro = 'Erro na chamada da API. Código de Status: ' + response.getStatusCode();
            }
            
            // Adiciona o resultado à lista de respostas
            resultsList.add(result);
        }

        return resultsList;
    }
    
    // Método privado para mapear os dados da resposta da API para o objeto ReceitaWSResponse
    private static ReceitaWSResponse mapearDados(Map<String, Object> apiResults) {
        ReceitaWSResponse result = new ReceitaWSResponse();
        result.nome = apiResults.containsKey('nome') ? (String) apiResults.get('nome') : 'Não informado';
        result.cnpj = apiResults.containsKey('cnpj') ? (String) apiResults.get('cnpj') : 'Não informado';
        result.logradouro = apiResults.containsKey('logradouro') ? (String) apiResults.get('logradouro') : 'Não informado';
        result.numero = apiResults.containsKey('numero') ? (String) apiResults.get('numero') : 'Não informado';
        result.complemento = apiResults.containsKey('complemento') ? (String) apiResults.get('complemento') : 'Não informado';
        result.bairro = apiResults.containsKey('bairro') ? (String) apiResults.get('bairro') : 'Não informado';
        result.municipio = apiResults.containsKey('municipio') ? (String) apiResults.get('municipio') : 'Não informado';
        result.uf = apiResults.containsKey('uf') ? (String) apiResults.get('uf') : 'Não informado';
        result.cep = apiResults.containsKey('cep') ? (String) apiResults.get('cep') : 'Não informado';
        result.telefone = apiResults.containsKey('telefone') ? (String) apiResults.get('telefone') : 'Não informado';
        result.situacao = apiResults.containsKey('situacao') ? (String) apiResults.get('situacao') : 'Não informado';

        return result; 
    }
}
````

## **Classe `ReceitaWSHttpCalloutMock`**

A classe `ReceitaWSHttpCalloutMock` implementa a interface `HttpCalloutMock` para simular as respostas de uma chamada HTTP para a API ReceitaWS durante os testes. Ela é usada para mockar as respostas da API, garantindo que os testes sejam isolados e não dependam de chamadas reais à API externa.

### Método: `respond`

O método **respond** simula a resposta de um serviço HTTP com base no CNPJ enviado na requisição.

#### Parâmetros:

- **request** (`HTTPRequest`)  
  Objeto que representa a requisição HTTP feita à API ReceitaWS.

#### Retorno:

- **response** (`HttpResponse`)  
  Retorna um objeto `HttpResponse` simulado, que pode representar um CNPJ válido, inválido ou não encontrado.

### Lógica de Resposta:

- **CNPJ Válido:**  
  Para o CNPJ `28395380000120`, a resposta será simulada como um CNPJ válido, retornando informações como nome, data de abertura, endereço, telefone, etc.

- **CNPJ Inválido:**  
  Para o CNPJ `123`, a resposta será simulada com o status "ERROR" e a mensagem "CNPJ inválido".

- **CNPJ Não Encontrado:**  
  Para o CNPJ `22222222222222`, a resposta será simulada com o status "ERROR" e a mensagem "CNPJ não encontrado".
  
```apex
@isTest
public class ReceitaWSHttpCalloutMock implements HttpCalloutMock {
    public HTTPResponse respond(HTTPRequest request) {
        HttpResponse response = new HttpResponse();
        response.setHeader('Content-Type', 'application/json');
        
        // Verifica o CNPJ que foi enviado na requisição
        if (request.getEndpoint().contains('28395380000120')) {
            // Resposta para um CNPJ válido
            response.setStatusCode(200);
            response.setBody('{' +
            '"nome": "SYSMAP SOLUTIONS SOFTWARE E CONSULTORIA LTDA SCP", ' +
            '"cnpj": "28395380000120", ' + 
            '"data_abertura": "02/01/2012", ' +
            '"capital_social": "R$ 10.000,00", ' +
            '"situacao": "ATIVA", ' +
            '"logradouro": "RUA DESEM ELISEU GUILHERME", ' +
            '"numero": "200", ' +
            '"complemento": "ANDAR 11", ' +
            '"bairro": "PARAISO", ' +
            '"municipio": "SAO PAULO", ' +
            '"uf": "SP", ' +
            '"cep": "04.004-030", ' +
            '"telefone": "(11) 3884-7479" ' +
            '}');  
        } 
        else if (request.getEndpoint().contains('123')) {
            // Resposta para um CNPJ inválido
            response.setStatusCode(200); // Status OK 200, mas com um erro na resposta
            response.setBody('{' +
                '"status": "ERROR", ' +
                '"message": "CNPJ inválido" ' +
            '}');
        } 
        else if (request.getEndpoint().contains('22222222222222')) {
            // Resposta para um CNPJ não encontrado
            response.setStatusCode(200); // Status OK 200, mas com um erro na resposta
            response.setBody('{' +
                '"status": "ERROR", ' +
                '"message": "CNPJ não encontrado" ' +
            '}');
        }

        return response;
    }
}

````



---

## **Classe `ReceitaWSServiceTest`**

A classe `ReceitaWSServiceTest` é responsável pelos testes unitários do serviço `ReceitaWSService`, utilizando a classe `ReceitaWSHttpCalloutMock` para simular as respostas da API.

### Método: `testeConsultarReceita`

Este método realiza os testes unitários para garantir que a consulta à API ReceitaWS funcione corretamente, verificando as respostas para diferentes cenários de CNPJ.

#### Passos do Teste:

1. **Teste para CNPJ Válido:**  
   - O método envia um CNPJ válido (`28395380000120`) para a API e verifica se o nome da empresa retornada é "SYSMAP SOLUTIONS SOFTWARE E CONSULTORIA LTDA SCP".

2. **Teste para CNPJ Inválido:**  
   - O método envia um CNPJ inválido (`123`) para a API e verifica se a resposta contém a mensagem de erro "CNPJ inválido".

3. **Teste para CNPJ Não Encontrado:**  
   - O método envia um CNPJ não encontrado (`22222222222222`) para a API e verifica se a resposta contém a mensagem de erro "CNPJ não encontrado".

#### Exemplo de Código:

```apex


@isTest
public class ReceitaWSServiceTest {
    @isTest
    static void testeConsultarReceita() {
        Test.setMock(HttpCalloutMock.class, new ReceitaWSHttpCalloutMock());

        // Teste com um CNPJ válido
        String cnpjValido = '28395380000120'; 
        ReceitaWSRequest requestValido = new ReceitaWSRequest();
        requestValido.cnpj = cnpjValido;
        
        List<ReceitaWSRequest> requests = new List<ReceitaWSRequest>{requestValido};
        List<ReceitaWSResponse> resultCnpj = ReceitaWSService.consultarReceita(requests);
        
        // Verificações para CNPJ válido
        System.assertNotEquals(null, resultCnpj, 'O resultado não deve ser nulo.');
        System.assertEquals('SYSMAP SOLUTIONS SOFTWARE E CONSULTORIA LTDA SCP', 
            resultCnpj[0].nome, 'O nome da empresa deve ser "SYSMAP SOLUTIONS SOFTWARE E CONSULTORIA LTDA SCP".');
        
        // Teste com um CNPJ inválido
        String cnpjInvalido = '123'; 
        ReceitaWSRequest requestInvalido = new ReceitaWSRequest();
        requestInvalido.cnpj = cnpjInvalido;
        
        List<ReceitaWSRequest> requestsInvalido = new List<ReceitaWSRequest>{requestInvalido};
        List<ReceitaWSResponse> resultInvalido = ReceitaWSService.consultarReceita(requestsInvalido);
        
        // Verificações para CNPJ inválido
        System.assertNotEquals(null, resultInvalido, 'O resultado não deve ser nulo para CNPJ inválido.');
        System.assertEquals('Erro', resultInvalido[0].status, 'O resultado deve indicar erro para um CNPJ inválido.');
        System.assertEquals('CNPJ inválido', resultInvalido[0].mensagemErro, 'A mensagem de erro deve ser "CNPJ inválido".');

        // Teste com um CNPJ não encontrado
        String cnpjNaoEncontrado = '22222222222222'; 
        ReceitaWSRequest requestNaoEncontrado = new ReceitaWSRequest();
        requestNaoEncontrado.cnpj = cnpjNaoEncontrado;
        
        List<ReceitaWSRequest> requestsNaoEncontrado = new List<ReceitaWSRequest>{requestNaoEncontrado};
        List<ReceitaWSResponse> resultNaoEncontrado = ReceitaWSService.consultarReceita(requestsNaoEncontrado);
        
        // Verificações para CNPJ não encontrado
        System.assertNotEquals(null, resultNaoEncontrado, 'O resultado não deve ser nulo para CNPJ não encontrado.');
        System.assertEquals('Erro', resultNaoEncontrado[0].status, 'O resultado deve indicar erro para um CNPJ não encontrado.');
        System.assertEquals('CNPJ não encontrado', resultNaoEncontrado[0].mensagemErro, 'A mensagem de erro deve ser "CNPJ não encontrado".');
    }
}

