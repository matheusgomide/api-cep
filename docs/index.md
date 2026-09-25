# API de Consulta de CEP

## Visão Geral

API REST desenvolvida em FastAPI para consulta de informações de endereços brasileiros através de CEP (Código de Endereçamento Postal). A API utiliza o serviço ViaCEP como fonte de dados.

## Características

- Consulta de CEP em formato simples (12345678) ou formatado (12345-678)
- Validação automática de formato de CEP
- Tratamento de erros completo
- Documentação interativa via Swagger UI
- Respostas padronizadas em JSON

## Requisitos

- Python 3.7+
- FastAPI
- Uvicorn
- HTTPX

## Instalação

```bash
pip install -r requirements.txt
```

## Execução

### Desenvolvimento

```bash
uvicorn main:app --reload
```

### Produção

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

### Docker

```bash
docker build -t api-cep .
docker run -p 8000:8000 api-cep
```

## Endpoints

### Consultar CEP

Retorna informações detalhadas de um endereço a partir do CEP fornecido.

**Endpoint:** `GET /cep/{cep}`

**Parâmetros:**

| Nome | Tipo   | Localização | Obrigatório | Descrição                                    |
|------|--------|-------------|-------------|----------------------------------------------|
| cep  | string | path        | Sim         | CEP no formato 12345678 ou 12345-678         |

**Respostas:**

| Código | Descrição                               |
|--------|-----------------------------------------|
| 200    | CEP encontrado com sucesso              |
| 400    | CEP inválido (formato incorreto)        |
| 404    | CEP não encontrado na base de dados     |
| 502    | Falha ao consultar o serviço externo    |

## Exemplos de Uso

### cURL

#### Consulta básica

```bash
curl http://localhost:8000/cep/01310100
```

#### Consulta com CEP formatado

```bash
curl http://localhost:8000/cep/01310-100
```

#### Resposta de sucesso

```json
{
  "cep": "01310-100",
  "logradouro": "Avenida Paulista",
  "complemento": "",
  "bairro": "Bela Vista",
  "localidade": "São Paulo",
  "uf": "SP",
  "ibge": "3550308",
  "gia": "1004",
  "ddd": "11",
  "siafi": "7107"
}
```

### Python

#### Usando requests

```python
import requests

response = requests.get("http://localhost:8000/cep/01310100")
if response.status_code == 200:
    data = response.json()
    print(f"Endereço: {data['logradouro']}, {data['bairro']}")
    print(f"Cidade: {data['localidade']}/{data['uf']}")
else:
    print(f"Erro: {response.status_code}")
```

#### Usando httpx (async)

```python
import httpx
import asyncio

async def consultar_cep(cep: str):
    async with httpx.AsyncClient() as client:
        response = await client.get(f"http://localhost:8000/cep/{cep}")
        return response.json()

data = asyncio.run(consultar_cep("01310100"))
print(data)
```

### JavaScript

#### Fetch API

```javascript
fetch('http://localhost:8000/cep/01310100')
  .then(response => response.json())
  .then(data => {
    console.log(`Endereço: ${data.logradouro}, ${data.bairro}`);
    console.log(`Cidade: ${data.localidade}/${data.uf}`);
  })
  .catch(error => console.error('Erro:', error));
```

#### Axios

```javascript
const axios = require('axios');

axios.get('http://localhost:8000/cep/01310100')
  .then(response => {
    const data = response.data;
    console.log(`Endereço: ${data.logradouro}, ${data.bairro}`);
    console.log(`Cidade: ${data.localidade}/${data.uf}`);
  })
  .catch(error => {
    console.error('Erro:', error.response.data.detail);
  });
```

#### Async/Await

```javascript
async function consultarCEP(cep) {
  try {
    const response = await fetch(`http://localhost:8000/cep/${cep}`);
    if (!response.ok) {
      throw new Error(`HTTP error ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Erro ao consultar CEP:', error);
    return null;
  }
}

// Uso
consultarCEP('01310100').then(data => console.log(data));
```

### PHP

```php
<?php
$cep = '01310100';
$url = "http://localhost:8000/cep/{$cep}";

$response = file_get_contents($url);
$data = json_decode($response, true);

if ($data) {
    echo "Endereço: {$data['logradouro']}, {$data['bairro']}\n";
    echo "Cidade: {$data['localidade']}/{$data['uf']}\n";
} else {
    echo "Erro ao consultar CEP\n";
}
?>
```

### Java

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import com.google.gson.Gson;

public class ConsultaCEP {
    public static void main(String[] args) throws Exception {
        String cep = "01310100";
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("http://localhost:8000/cep/" + cep))
            .build();
        
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
        
        if (response.statusCode() == 200) {
            System.out.println(response.body());
        } else {
            System.out.println("Erro: " + response.statusCode());
        }
    }
}
```

## Tratamento de Erros

### CEP Inválido (400)

```bash
curl http://localhost:8000/cep/123
```

Resposta:

```json
{
  "detail": "CEP inválido"
}
```

### CEP Não Encontrado (404)

```bash
curl http://localhost:8000/cep/99999999
```

Resposta:

```json
{
  "detail": "CEP não encontrado"
}
```

### Erro do Serviço Externo (502)

```json
{
  "detail": "Falha ao consultar ViaCEP"
}
```

## Estrutura da Resposta

| Campo       | Tipo   | Descrição                                      |
|-------------|--------|------------------------------------------------|
| cep         | string | CEP no formato 12345-678                       |
| logradouro  | string | Nome da rua/avenida                            |
| complemento | string | Complemento do endereço                        |
| bairro      | string | Nome do bairro                                 |
| localidade  | string | Nome da cidade                                 |
| uf          | string | Sigla do estado (2 caracteres)                 |
| ibge        | string | Código IBGE do município                       |
| gia         | string | Código GIA (Guia de Informação e Apuração)     |
| ddd         | string | Código DDD da região                           |
| siafi       | string | Código SIAFI do município                      |

## Documentação Interativa

Acesse a documentação interativa Swagger UI para testar os endpoints diretamente no navegador:

```
http://localhost:8000/docs
```

Documentação alternativa ReDoc:

```
http://localhost:8000/redoc
```

Especificação OpenAPI em JSON:

```
http://localhost:8000/openapi.json
```

## Limitações

- A API depende do serviço ViaCEP para funcionar
- Limitações de rate limit são aplicadas pelo ViaCEP
- Apenas CEPs brasileiros são suportados
- Não há cache de respostas (cada requisição consulta o ViaCEP)

## Boas Práticas de Uso

1. **Validação de entrada**: Sempre valide o formato do CEP antes de fazer a requisição
2. **Tratamento de erros**: Implemente tratamento adequado para todos os códigos de resposta
3. **Timeout**: Configure timeouts adequados nas requisições
4. **Rate limiting**: Implemente rate limiting no lado do cliente para evitar sobrecarga
5. **Cache**: Considere implementar cache local para CEPs consultados frequentemente
6. **Retry logic**: Implemente lógica de retry com backoff exponencial para erros 502

## Exemplos de CEPs para Teste

| CEP       | Localização                            |
|-----------|----------------------------------------|
| 01310-100 | Avenida Paulista, São Paulo/SP         |
| 20040-020 | Praça Marechal Âncora, Rio de Janeiro/RJ |
| 30130-010 | Praça Sete de Setembro, Belo Horizonte/MG |
| 40020-000 | Praça Municipal, Salvador/BA           |
| 80010-000 | Praça Tiradentes, Curitiba/PR          |

## Licença

Este projeto é fornecido como exemplo educacional.