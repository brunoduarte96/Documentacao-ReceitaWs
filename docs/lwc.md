# Componente LWC

## Descrição

O componente `CnpjInput` permite a entrada e validação de um número de CNPJ com 14 dígitos numéricos. Ele realiza as seguintes funcionalidades:

1. Valida o CNPJ conforme o formato exigido (14 dígitos numéricos).
2. Exibe uma mensagem de erro caso o formato do CNPJ esteja incorreto.
3. Ativa um botão "Buscar Dados" que, ao ser clicado, inicia um Flow (`ReceitaWs`) passando o CNPJ como variável de entrada.
4. Exibe um Flow Lightning para consultar os dados relacionados ao CNPJ.

## Estrutura do Componente

### HTML (Template)

```html
<template>
    <lightning-card title="Validação de CNPJ">
        <div class="slds-p-around_medium">
            <!-- Campo de entrada de CNPJ -->
            <lightning-input 
                label="CNPJ" 
                value={cnpj} 
                onchange={handleCNPJChange} 
                maxlength="14" 
                pattern="\d{14}" 
                required>
            </lightning-input>            
           
            <!-- Exibição da mensagem de erro -->
            <template if:true={errorMessage}>
                <div class="slds-text-color_error slds-m-top_small">
                    {errorMessage}
                </div>
            </template>            
          
            <!-- Botão para acionar a busca de dados -->
            <lightning-button 
                label="Buscar Dados" 
                onclick={handleSearch} 
                variant="brand" 
                class="slds-m-top_medium"
                disabled={isButtonDisabled}>
            </lightning-button>
        </div>
    </lightning-card>
   
    <!-- Exibição do Flow quando ativado -->
    <template if:true={flowAvailable}>
        <lightning-flow 
            flow-name="ReceitaWs" 
            onstatuschange={handleStatusChange}>
        </lightning-flow>
    </template>
</template>
````

### JavaScript

## Descrição do Código JavaScript:


`LightningElement`, `track`, e `wire`: Usados para criar o componente LWC e gerenciar reatividade e comunicação com Salesforce.


`ShowToastEvent`: Usado para exibir mensagens de notificação (toast).

`getRecord`: Usado para recuperar registros do Salesforce (no caso, o campo `CNPJ__c` de um `Account`).
Propriedades:

`cnpj`: Armazena o valor inserido pelo usuário no campo CNPJ.

`errorMessage`: Armazena a mensagem de erro que será exibida quando o CNPJ não for válido.

`isButtonDisabled`: Controla o estado do botão "Buscar Dados" (habilitado ou desabilitado).

`flowAvailable`: Controla a visibilidade do Flow.

#### Métodos:

`handleCNPJChange`: Este método é chamado toda vez que o valor do CNPJ é alterado. Ele valida se o CNPJ tem 14 dígitos e atualiza a mensagem de erro, além de habilitar ou desabilitar o botão.

`handleSearch`: Este método é chamado quando o botão "Buscar Dados" é clicado. Ele inicia o Flow, passando o CNPJ como variável de entrada, se o CNPJ for válido.

`showToast`: Exibe uma mensagem de toast (notificação) no Salesforce.

`handleStatusChange`: Este método é chamado quando o status do Flow muda. Exibe uma notificação de sucesso quando o Flow é concluído.

```javascript
import { LightningElement, track, wire } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import { getRecord } from 'lightning/uiRecordApi';

const FIELDS = ['Account.CNPJ__c'];

export default class CnpjInput extends LightningElement {
    @track cnpj = ''; // Armazena o valor do CNPJ inserido
    @track errorMessage = ''; // Armazena a mensagem de erro de validação
    @track isButtonDisabled = true; // Controla se o botão de busca está desabilitado
    @track flowAvailable = false; // Controla a visibilidade do Flow

    @wire(getRecord, { recordId: '$recordId', fields: FIELDS })
    account;

    get cnpjFromAccount() {
        return this.account.data ? this.account.data.fields.CNPJ__c.value : '';
    }

    handleCNPJChange(event) {
        this.cnpj = event.target.value;

        // Validação de CNPJ
        const isValidCNPJ = /^[0-9]{14}$/.test(this.cnpj);
        if (!isValidCNPJ) {
            this.errorMessage = 'CNPJ deve conter exatamente 14 dígitos numéricos.';
            this.isButtonDisabled = true;
        } else {
            this.errorMessage = '';
            this.isButtonDisabled = false;
        }
    }

    handleSearch() {
        if (this.isButtonDisabled) {
            this.showToast('Erro', 'CNPJ inválido. Verifique o formato.', 'error');
            return;
        }

        // Torna o Flow visível e começa o Flow
        this.flowAvailable = true;

        // Defina os parâmetros para o Flow, mapeando o valor do CNPJ para a variável de entrada "CNPJInput"
        const flowInputVariables = [
            {
                name: 'CNPJInput',  // Nome da variável de entrada no Flow
                type: 'String',
                value: this.cnpj
            }
        ];

        // Inicia o Flow apenas depois de o flowAvailable ser true
        setTimeout(() => {
            const flow = this.template.querySelector('lightning-flow');
            if (flow) {
                flow.startFlow('ReceitaWs', flowInputVariables);  // 'ReceitaWs' é o nome do Flow
                console.log('Flow iniciado com sucesso');
            } else {
                console.error('Componente lightning-flow não encontrado');
            }
        }, 0); // Utiliza setTimeout para garantir que o Flow seja carregado primeiro
    }

    showToast(title, message, variant) {
        const evt = new ShowToastEvent({
            title: title,
            message: message,
            variant: variant,
        });
        this.dispatchEvent(evt);
    }

    handleStatusChange(event) {
        const status = event.detail.status;
        if (status === 'FINISHED') {
            this.showToast('Sucesso', 'Flow concluído com sucesso!', 'success');
        }
    }
}
````
