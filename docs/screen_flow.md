# Screen Flow: Cadastro de Clientes com Validação de CNPJ

Este Screen Flow guia o usuário pelo processo de cadastro de clientes no Salesforce, garantindo validação de CNPJ, integração com a API ReceitaWS e salvamento automatizado de dados no objeto Account.

---

## **Visão Geral**
O fluxo foi projetado para simplificar o processo de cadastro de clientes. Ele valida o CNPJ, busca dados automaticamente na API ReceitaWS e permite ao usuário revisar e ajustar informações antes de salvar.

---

## **Estrutura do Fluxo**

### **1. Tela de Entrada do CNPJ**
- **Objetivo:** Capturar o CNPJ e validar o formato.
- **Componentes:**
  - Campo de texto para entrada do CNPJ (obrigatório).
  - Mensagem de erro caso o campo esteja vazio ou o CNPJ esteja no formato incorreto.
  - Validação do formato (14 dígitos numéricos).

![Tela de Entrada do CNPJ](https://github.com/user-attachments/assets/7f6796f6-2f31-4b3d-9232-1706da101837)


---

### **2. Chamada à Classe Apex**
- **Objetivo:** Integrar com a API ReceitaWS para buscar dados do CNPJ.
- **Componentes:**
  - Elemento de Ação do Flow: Chamada à classe Apex `ReceitaWSService`.
  - Parâmetro enviado: `CNPJ`.
  - Dados retornados: Nome, Endereço, Telefone, Status Cadastral, etc.

#### **Configuração Técnica:**
- Classe Apex: `ReceitaWSService`.
- Método: `public static ReceitaResponse getDadosCNPJ(String cnpj)`.




---

### **3. Tela de Revisão dos Dados**
- **Objetivo:** Permitir que o usuário revise e edite os dados retornados pela API.
- **Componentes:**
  - Campos editáveis para:
    - Razão Social.
    - Endereço (Logradouro, Número, Complemento, Bairro).
    - Cidade, Estado, CEP.
    - Telefone.
    - Status Cadastral.

![Tela de Revisão dos Dados](https://github.com/user-attachments/assets/0e83fb6d-ad5e-4106-b86c-5cc7c06d9e82)

---

### **4. Criação do Registro**
- **Objetivo:** Salvar os dados revisados no objeto Account.
- **Componentes:**
  - Elemento de Criação de Registros.
  - Objeto: Account.
  - Campos:
    - Name (Razão Social).
    - CNPJ__c.
    - BillingStreet, BillingCity, BillingState, BillingPostalCode.
    - Phone.
    - Status_Cadastral__c.



---

### **5. Confirmação**
- **Objetivo:** Exibir uma mensagem de sucesso ao usuário após o cadastro.
- **Componentes:**
  - Tela com mensagem: *"Cadastro realizado com sucesso!"*
  - Botão para finalizar ou iniciar um novo cadastro.

![Tela de Confirmação](https://github.com/user-attachments/assets/ec4cb983-050c-40c0-a5e5-0803e490da9d)


---

## **Fluxograma do Screen Flow**

Abaixo está o fluxograma que representa o processo do Screen Flow:

![Fluxograma do Screen Flow](https://github.com/user-attachments/assets/e8afb128-ded9-4dd8-8c9e-2db64a305c31)![image](https://github.com/user-attachments/assets/71353f03-3be8-4365-9539-c4ae6fe0ee77)



---

## **Considerações Adicionais**
- **Validações Importantes:**
  - Garantir que o CNPJ seja único antes de salvar.
  - Implementar mensagens claras para erros.

- **Testes:**
  - Validação de entrada do CNPJ.
  - Fluxo com dados completos e incompletos da API.
  - Simulações para erros de conexão ou limites da API.

- **Segurança:**
  - Certifique-se de que o fluxo esteja disponível apenas para usuários autorizados (ex.: Representantes de Vendas).
