
# API de Solicitações de Notas Fiscais

API HTTP em Node.js/TypeScript (Express) para criar, listar, buscar, emitir e cancelar solicitações de Nota Fiscal. Persistência em SQLite (arquivo), sem precisar de servidor de banco externo.

## Requisitos
- Node.js >= 20
- npm

## Instalação
```bash
npm install
```

## Execução
- Desenvolvimento (hot reload): `npm run dev`
- Produção: `npm run build && npm start`
- Porta padrão: `3000` (ajuste `PORT` se quiser)
- Banco: arquivo SQLite em `data/invoices.db` (padrão). Para personalizar, defina a variável de ambiente `INVOICE_DB_PATH`.

## Testes
```bash
npm test
```
- Suíte `tests/invoice.spec.ts`: cria, lista, busca, emite (sucesso e erro 400 simulado), cancela pendente e bloqueia cancelamento de emitidas.
- Para rodar um teste específico: `npm test -- --testNamePattern "texto do teste"` ou `npx jest tests/invoice.spec.ts -t "texto do teste"`.

## Endpoints
Base URL: `http://localhost:3000`

- `POST /invoices`  
  Cria solicitação com status inicial `PENDENTE_EMISSAO`.
  Body (JSON):
  ```json
  {
    "cnpj": "12345678901234",
    "municipio": "São Paulo",
    "estado": "SP",
    "valorServico": 1000,
    "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
    "descricaoServico": "Serviço de consultoria"
  }
  ```

- `GET /invoices`  
  Lista todas as solicitações persistidas.

- `GET /invoices/:id`  
  Busca uma solicitação.

- `POST /invoices/:id/emit`  
  Emite NF chamando a API externa `https://api.drfinancas.com/testes/notas-fiscais` com header `Authorization: ################################`.
  Sucesso (200): salva `numeroNF`, `dataEmissao`, status vira `EMITIDA`.
  Erros 400/401/500 retornam mensagens adequadas.

- `POST /invoices/:id/cancel`  
  Cancela solicitações pendentes. Não permite cancelar emitidas.

### Exemplos com curl (copiáveis)
- Criar:
```bash
curl -X POST http://localhost:3000/invoices -H "Content-Type: application/json" -d "{\"cnpj\":\"12345678901234\",\"municipio\":\"Sao Paulo\",\"estado\":\"SP\",\"valorServico\":1000,\"dataDesejadaEmissao\":\"2025-12-31T10:00:00.000Z\",\"descricaoServico\":\"Servico de consultoria\"}"
```
- Listar:
```bash
curl http://localhost:3000/invoices
```
- Buscar:
```bash
curl http://localhost:3000/invoices/{id}
```
- Emitir:
```bash
curl -X POST http://localhost:3000/invoices/{id}/emit
```
- Cancelar:
```bash
curl -X POST http://localhost:3000/invoices/{id}/cancel
```

## Status suportados
- `PENDENTE_EMISSAO` (inicial)
- `EMITIDA`
- `CANCELADA`

## Notas
- A API externa é usada apenas no fluxo de emissão; demais operações são locais.
- Para resetar dados, pare o servidor e apague o arquivo `data/invoices.db` (ou o caminho definido em `INVOICE_DB_PATH`); ele será recriado na próxima execução.
- Logs e erros aparecem no console do servidor.
- Comportamento da emissão: a API externa responde aleatoriamente 200/400/401/500; a chave de autorização já está embutida no código e é usada apenas na rota `/emit`.
- Retornos esperados:
  - Criar: 201
  - Listar/Buscar: 200 (404 se id não existir)
  - Emitir: 200 com `numeroNF`/`dataEmissao`/status `EMITIDA`, ou 400/401/500 conforme resposta externa; 400 se tentar emitir cancelada/já emitida
  - Cancelar: 200 se pendente, 400 se emitida ou já cancelada, 404 se id não existir

## Variáveis de ambiente
- `PORT`: porta do servidor (padrão 3000).
- `INVOICE_DB_PATH`: caminho do arquivo SQLite (padrão `./data/invoices.db` em relação ao cwd).

## Passo a passo rápido (Thunder/Postman)
- GET `http://localhost:3000/invoices`  
  Headers: `Content-Type: application/json` (opcional em GET).
- POST `http://localhost:3000/invoices`  
  Headers: `Content-Type: application/json`  
  Body:
  ```json
  {
    "cnpj": "12345678901234",
    "municipio": "Sao Paulo",
    "estado": "SP",
    "valorServico": 1000,
    "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
    "descricaoServico": "Servico de consultoria"
  }
  ```
- GET `http://localhost:3000/invoices/{id}`
- POST `http://localhost:3000/invoices/{id}/emit` (sem body)
- POST `http://localhost:3000/invoices/{id}/cancel` (sem body)

## Para testar com dados existentes
Se já houver registros no banco, use `GET http://localhost:3000/invoices` para listar e pegar os `id`s. Em seguida, teste:
- `GET /invoices/{id}` para detalhar
- `POST /invoices/{id}/emit` para emitir
- `POST /invoices/{id}/cancel` para cancelar se ainda estiver pendente

# Testes atuais feitos e confirmados:
##### Post/Create:

http://localhost:3000/invoices

Headers: Content-Type: application/json

Body:
```json
{
  "id": "2657a828-df62-4a9b-8b81-f76aa2ce1b5f",
  "cnpj": "12345678901234",
  "municipio": "São Paulo",
  "estado": "SP",
  "valorServico": 1000,
  "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
  "descricaoServico": "Serviço de consultoria",
  "status": "PENDENTE_EMISSAO",
  "createdAt": "2025-12-15T17:23:42.957Z",
  "updatedAt": "2025-12-15T17:23:42.957Z"
}
```

##### /Listar:
GET: http://localhost:3000/invoices
Headers: Content-Type: application/json
```json
[
  {
    "id": "abeb5749-b84c-4265-9107-b8a396afb1bb",
    "cnpj": "12345678901234",
    "municipio": "São Paulo",
    "estado": "SP",
    "valorServico": 1000,
    "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
    "descricaoServico": "Serviço de consultoria",
    "status": "EMITIDA",
    "createdAt": "2025-12-15T17:13:36.280Z",
    "updatedAt": "2025-12-15T17:16:40.971Z",
    "numeroNF": "00847",
    "dataEmissao": "2025-12-15T14:16:40-03:00"
  },
  {
    "id": "fc7f48b2-b2be-4c24-8ac0-7a7efea99814",
    "cnpj": "12345678901234",
    "municipio": "São Paulo",
    "estado": "SP",
    "valorServico": 1000,
    "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
    "descricaoServico": "Serviço de consultoria",
    "status": "PENDENTE_EMISSAO",
    "createdAt": "2025-12-15T17:23:05.863Z",
    "updatedAt": "2025-12-15T17:23:05.863Z",
    "numeroNF": null,
    "dataEmissao": null
  },
  {
    "id": "2657a828-df62-4a9b-8b81-f76aa2ce1b5f",
    "cnpj": "12345678901234",
    "municipio": "São Paulo",
    "estado": "SP",
    "valorServico": 1000,
    "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
    "descricaoServico": "Serviço de consultoria",
    "status": "CANCELADA",
    "createdAt": "2025-12-15T17:23:42.957Z",
    "updatedAt": "2025-12-15T17:24:35.517Z",
    "numeroNF": null,
    "dataEmissao": null
  }
]
```
##### List/ID:
GET: http://localhost:3000/invoices/abeb5749-b84c-4265-9107-b8a396afb1bb

```json
{
  "id": "abeb5749-b84c-4265-9107-b8a396afb1bb",
  "cnpj": "12345678901234",
  "municipio": "São Paulo",
  "estado": "SP",
  "valorServico": 1000,
  "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
  "descricaoServico": "Serviço de consultoria",
  "status": "EMITIDA",
  "createdAt": "2025-12-15T17:13:36.280Z",
  "updatedAt": "2025-12-15T17:16:40.971Z",
  "numeroNF": "00847",
  "dataEmissao": "2025-12-15T14:16:40-03:00"
}
```

##### Cancelar:
POST: http://localhost:3000/invoices/2657a828-df62-4a9b-8b81-f76aa2ce1b5f/cancel

```json
{
  "id": "2657a828-df62-4a9b-8b81-f76aa2ce1b5f",
  "cnpj": "12345678901234",
  "municipio": "São Paulo",
  "estado": "SP",
  "valorServico": 1000,
  "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
  "descricaoServico": "Serviço de consultoria",
  "status": "CANCELADA",
  "createdAt": "2025-12-15T17:23:42.957Z",
  "updatedAt": "2025-12-15T17:24:35.517Z",
  "numeroNF": null,
  "dataEmissao": null
}
```

##### Emitir nota:

POST: http://localhost:3000/invoices/abeb5749-b84c-4265-9107-b8a396afb1bb/emit

```json
{
  "id": "abeb5749-b84c-4265-9107-b8a396afb1bb",
  "cnpj": "12345678901234",
  "municipio": "São Paulo",
  "estado": "SP",
  "valorServico": 1000,
  "dataDesejadaEmissao": "2025-12-31T10:00:00.000Z",
  "descricaoServico": "Serviço de consultoria",
  "status": "EMITIDA",
  "createdAt": "2025-12-15T17:13:36.280Z",
  "updatedAt": "2025-12-15T17:16:40.971Z",
  "numeroNF": "00847",
  "dataEmissao": "2025-12-15T14:16:40-03:00"
}
```
