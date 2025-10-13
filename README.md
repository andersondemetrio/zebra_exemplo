# Documentação Técnica - Impressão de Etiquetas Intelipost

## Visão Geral

Esta documentação descreve como integrar com a API da Intelipost para impressão de etiquetas de envio, incluindo suporte para impressoras Zebra (formato ZPL).

## Autenticação

Todas as requisições devem incluir um token de autenticação no header:

```
Authorization: Bearer {seu_token}
```

## Endpoint Principal

### Obter Etiqueta de Envio

**Método:** `GET`  
**URL:** `https://api.intelipost.com.br/api/v1/shipment/{shipment_id}/label`

## Parâmetros

### Path Parameters

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `shipment_id` | string | Sim | ID único do envio/pedido na Intelipost |

### Query Parameters

| Parâmetro | Tipo | Obrigatório | Valores Aceitos | Descrição |
|-----------|------|-------------|-----------------|-----------|
| `format` | string | Não | `pdf`, `zpl`, `epl` | Formato da etiqueta. **Padrão:** `pdf` |
| `type` | string | Não | `A6`, `normal` | Tipo/tamanho da etiqueta |
| `resolution` | integer | Não | `203`, `300`, `600` | Resolução em DPI (para ZPL) |

## Formatos Disponíveis

### PDF (Padrão)
- Formato universal para visualização e impressão
- Retorna documento binário
- Ideal para impressoras convencionais

### ZPL (Zebra Programming Language)
- Formato específico para impressoras Zebra
- Retorna comandos em texto puro
- Otimizado para impressão térmica de alta velocidade
- **Use este formato para impressoras Zebra**

### EPL (Eltron Programming Language)
- Formato para impressoras Eltron/Zebra antigas
- Similar ao ZPL mas com sintaxe diferente

## Exemplos de Requisição

### 1. Etiqueta em PDF (formato padrão)

```bash
curl -X GET \
  'https://api.intelipost.com.br/api/v1/shipment/12345/label' \
  -H 'Authorization: Bearer seu_token_aqui' \
  -H 'Content-Type: application/json'
```

### 2. Etiqueta em ZPL para Zebra

```bash
curl -X GET \
  'https://api.intelipost.com.br/api/v1/shipment/12345/label?format=zpl' \
  -H 'Authorization: Bearer seu_token_aqui' \
  -H 'Content-Type: application/json'
```

### 3. Etiqueta ZPL com resolução específica

```bash
curl -X GET \
  'https://api.intelipost.com.br/api/v1/shipment/12345/label?format=zpl&resolution=300' \
  -H 'Authorization: Bearer seu_token_aqui' \
  -H 'Content-Type: application/json'
```

## Exemplos de Código

### JavaScript/Node.js

```javascript
const axios = require('axios');

async function obterEtiquetaZebra(shipmentId, token) {
  try {
    const response = await axios.get(
      `https://api.intelipost.com.br/api/v1/shipment/${shipmentId}/label`,
      {
        params: {
          format: 'zpl',
          resolution: 300
        },
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    return response.data;
  } catch (error) {
    console.error('Erro ao obter etiqueta:', error.response?.data || error.message);
    throw error;
  }
}

// Uso
const etiquetaZPL = await obterEtiquetaZebra('12345', 'seu_token');
console.log(etiquetaZPL); // Comandos ZPL em texto
```

### Python

```python
import requests

def obter_etiqueta_zebra(shipment_id, token):
    url = f"https://api.intelipost.com.br/api/v1/shipment/{shipment_id}/label"
    
    headers = {
        'Authorization': f'Bearer {token}',
        'Content-Type': 'application/json'
    }
    
    params = {
        'format': 'zpl',
        'resolution': 300
    }
    
    response = requests.get(url, headers=headers, params=params)
    
    if response.status_code == 200:
        return response.text  # Retorna comandos ZPL
    else:
        raise Exception(f"Erro: {response.status_code} - {response.text}")

# Uso
etiqueta_zpl = obter_etiqueta_zebra('12345', 'seu_token')
print(etiqueta_zpl)
```

### PHP

```php
<?php

function obterEtiquetaZebra($shipmentId, $token) {
    $url = "https://api.intelipost.com.br/api/v1/shipment/{$shipmentId}/label";
    
    $params = http_build_query([
        'format' => 'zpl',
        'resolution' => 300
    ]);
    
    $options = [
        'http' => [
            'header' => [
                "Authorization: Bearer {$token}",
                "Content-Type: application/json"
            ],
            'method' => 'GET'
        ]
    ];
    
    $context = stream_context_create($options);
    $result = file_get_contents("{$url}?{$params}", false, $context);
    
    return $result;
}

// Uso
$etiquetaZPL = obterEtiquetaZebra('12345', 'seu_token');
echo $etiquetaZPL;
```

### C# (.NET)

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class IntelipostService
{
    private readonly HttpClient _httpClient;
    private readonly string _token;
    
    public IntelipostService(string token)
    {
        _token = token;
        _httpClient = new HttpClient();
        _httpClient.DefaultRequestHeaders.Add("Authorization", $"Bearer {token}");
    }
    
    public async Task<string> ObterEtiquetaZebraAsync(string shipmentId)
    {
        var url = $"https://api.intelipost.com.br/api/v1/shipment/{shipmentId}/label?format=zpl&resolution=300";
        
        var response = await _httpClient.GetAsync(url);
        response.EnsureSuccessStatusCode();
        
        return await response.Content.ReadAsStringAsync();
    }
}

// Uso
var service = new IntelipostService("seu_token");
var etiquetaZPL = await service.ObterEtiquetaZebraAsync("12345");
Console.WriteLine(etiquetaZPL);
```

## Resposta da API

### Formato ZPL (Exemplo)

```zpl
^XA
^FO50,50^A0N,50,50^FDIntelipost^FS
^FO50,100^BY3^BCN,100,Y,N,N
^FD123456789^FS
^FO50,250^A0N,30,30^FDDestino: São Paulo, SP^FS
^XZ
```

### Formato PDF

Retorna um documento binário PDF que pode ser salvo diretamente:

```javascript
// Exemplo de salvamento em Node.js
const fs = require('fs');
const response = await axios.get(url, { responseType: 'arraybuffer' });
fs.writeFileSync('etiqueta.pdf', response.data);
```

## Como Imprimir ZPL

### 1. Envio Direto para Impressora (Windows)

```javascript
const { exec } = require('child_process');

function imprimirZPL(comandosZPL, nomeImpressora) {
  // Salva temporariamente
  fs.writeFileSync('temp.zpl', comandosZPL);
  
  // Envia para impressora
  exec(`copy /b temp.zpl \\\\localhost\\${nomeImpressora}`, (error) => {
    if (error) {
      console.error('Erro ao imprimir:', error);
    }
    fs.unlinkSync('temp.zpl');
  });
}
```

### 2. Envio via Socket (Rede)

```javascript
const net = require('net');

function imprimirZPLRede(comandosZPL, ipImpressora, porta = 9100) {
  const client = new net.Socket();
  
  client.connect(porta, ipImpressora, () => {
    client.write(comandosZPL);
    client.end();
  });
  
  client.on('error', (error) => {
    console.error('Erro ao imprimir:', error);
  });
}
```

## Códigos de Resposta HTTP

| Código | Descrição |
|--------|-----------|
| 200 | Sucesso - Etiqueta gerada |
| 400 | Requisição inválida - Verifique os parâmetros |
| 401 | Não autorizado - Token inválido ou expirado |
| 404 | Envio não encontrado - Verifique o shipment_id |
| 500 | Erro interno do servidor |

## Tratamento de Erros

```javascript
async function obterEtiquetaComRetry(shipmentId, token, maxTentativas = 3) {
  for (let tentativa = 1; tentativa <= maxTentativas; tentativa++) {
    try {
      return await obterEtiquetaZebra(shipmentId, token);
    } catch (error) {
      if (error.response?.status === 404) {
        throw new Error('Envio não encontrado');
      }
      
      if (error.response?.status === 401) {
        throw new Error('Token inválido ou expirado');
      }
      
      if (tentativa === maxTentativas) {
        throw error;
      }
      
      // Aguarda antes de tentar novamente
      await new Promise(resolve => setTimeout(resolve, 1000 * tentativa));
    }
  }
}
```

## Boas Práticas

1. **Cache de Etiquetas**: Armazene etiquetas já geradas para evitar requisições desnecessárias
2. **Tratamento de Erros**: Sempre implemente retry logic para falhas temporárias
3. **Validação**: Valide o shipment_id antes de fazer a requisição
4. **Logs**: Registre todas as requisições para auditoria
5. **Token**: Nunca exponha o token no frontend, sempre faça as requisições no backend

## Referências

- [Documentação Oficial Intelipost](https://intelipost.stoplight.io/docs/tms-embarcador)
- [ZPL Programming Guide](https://www.zebra.com/content/dam/zebra/manuals/printers/common/programming/zpl-zbi2-pm-en.pdf)

## Suporte

Para dúvidas ou problemas com a integração, entre em contato com o suporte da Intelipost através do portal de ajuda oficial.
