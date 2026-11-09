# Testes Caixa Cinza — API de Autenticação com Supabase

**Projeto:** Projeto Calculadora  
**Contexto:** FinançApp  
**Aluno:** Enzo Müller Santana  
**RA:** 252314  
**Curso:** Análise e Desenvolvimento de Sistemas  
**Turma:** AS406TSN4  
**Disciplina:** UX/UI e Testes de Software  
**Professor:** Daniel Ohata  
**Semestre:** 4º Semestre — 2026.2

# Introdução

Este repositório documenta a configuração e a execução de testes caixa cinza na API de autenticação do Supabase, utilizando o Postman Desktop. A atividade considera o conhecimento parcial da API: endpoint, método HTTP, headers, corpo JSON e respostas HTTP. Não há interface gráfica; todo o trabalho foi feito por requisições HTTP.

# Objetivo da Atividade

Configurar a autenticação por e-mail no Supabase, preparar requisições HTTP no Postman, executar cenários positivos e negativos, analisar as respostas e registrar evidências e resultados.

# Configuração do Supabase

## Como foi feito

1. Criação de uma conta em [supabase.com](https://supabase.com/).
2. Criação de um novo projeto e espera pela inicialização.
3. Em **Authentication → Providers → Email**, confirmação de que o provedor de e-mail e senha está habilitado.
4. Criação do usuário de teste `aluno1@email.com` em **Authentication → Users**. O e-mail é fictício e o usuário foi confirmado automaticamente (`email_confirmed_at` presente na resposta do login).
5. Em **Project Settings → API**, cópia da Project URL e da chave pública (anon/publishable).

**Tipo de autenticação:** e-mail e senha.  
**Objetivo da configuração:** ter um usuário válido e um endpoint de autenticação real para executar os testes.

| Dado | Valor |
|---|---|
| Nome do projeto | `MLLRZO's Org` |
| Project URL | `https://iscdrwfrausflfozzgpd.supabase.co` (ocultada) |
| Provedor | E-mail e senha |
| Usuário de teste | `aluno1@email.com` |
| Data da configuração | 08/11/2026 |

> **Segurança:** a senha, o access token, a chave `service_role` e a chave anon não estão publicados neste repositório.

## Evidências

> **Pendente:** os prints abaixo ainda precisam ser capturados e salvos em `prints/` (mostrando o seu usuário/identificação no painel do Supabase e ocultando valores sensíveis).

- `prints/01-projeto-supabase.png` — projeto criado e painel;
- `prints/02-auth-email.png` — provedor de e-mail habilitado;
- `prints/03-usuario-teste.png` — usuário criado;
- `prints/04-api-config.png` — credenciais da API (valores ocultos).

# Configuração do Postman

## Como foi feito

1. Instalação do Postman Desktop.
2. Criação do Workspace **Enzo's Workspace**.
3. Importação da coleção `postman/supabase-auth-gray-box.postman_collection.json` (**Supabase - Testes Caixa Cinza de Autenticação**).
4. Importação do Environment `postman/ambiente-modelo.postman_environment.json` (**Supabase - Modelo**) e preenchimento dos valores localmente no Postman.
5. Seleção do Environment **Supabase - Modelo** antes de executar as requisições.

O arquivo versionado contém apenas valores vazios ou de exemplo. O Environment preenchido não é exportado nem versionado.

| Variável | Onde fica | Finalidade |
|---|---|---|
| `base_url` | Environment | URL do projeto Supabase |
| `api_key` | Environment (secret) | Chave pública/anon do projeto |
| `email` | Environment | E-mail do usuário de teste |
| `password` | Environment (secret) | Senha do usuário de teste |
| `invalid_password` | Coleção | Senha incorreta (`SenhaErrada-123`) para o CT-02 |
| `unknown_email` | Coleção | E-mail não cadastrado (`usuario-inexistente@example.com`) para o CT-03 |

O Postman monta cada requisição substituindo as variáveis `{{...}}`, envia ao Supabase e executa os scripts de teste (aba *Scripts*) que validam o status HTTP e o JSON.

## Evidências

> **Pendente:** adicionar prints reais em `prints/`.

- `prints/05-workspace-postman.png` — Workspace criado;
- `prints/06-environment-postman.png` — Environment criado;
- `prints/07-variaveis-postman.png` — variáveis configuradas (ocultando valores sensíveis).

# Configuração das Requisições

**Método:** `POST`  
**Endpoint:** `{{base_url}}/auth/v1/token?grant_type=password`  
**Finalidade:** solicitar ao Supabase um token de acesso usando o fluxo de e-mail e senha. Em caso de sucesso, a API responde HTTP 200 com um JSON contendo `access_token`, `token_type`, `expires_in` e dados do usuário.

**Exemplo de Header**

| Header | Valor |
|---|---|
| `apikey` | `XXXXX` |
| `Authorization` | `Bearer XXXXX` |
| `Content-Type` | `application/json` |

**Exemplo do Body JSON** (raw → JSON)

```json
{
  "email": "usuario@email.com",
  "password": "123456"
}
```

No Postman o body usa variáveis: `"email": "{{email}}"` e `"password": "{{password}}"`.

## Evidências

O print `prints/08-login-valido.png` mostra o endpoint, o método POST, o body JSON, o Workspace e a coleção com as requisições criadas. Os headers (11 no total, contando os automáticos do Postman) podem ser complementados com um print da aba *Headers*.

# Execução dos Testes

Os cinco cenários obrigatórios foram executados em 08/11/2026 no Postman Desktop. Os scripts de cada requisição passaram (*Test Results 2/2* em todas).

| ID | Cenário | Entrada utilizada | Resultado esperado | Resultado obtido | Status |
|---|---|---|---|---|---|
| CT-01 | Login válido | `aluno1@email.com` + senha correta | HTTP 200 + `access_token` | HTTP 200 OK (884 ms); JSON com `access_token`, `token_type: bearer`, `expires_in: 3600`, `refresh_token` e `user` | OK |
| CT-02 | Senha incorreta | `aluno1@email.com` + `SenhaErrada-123` | Erro de autenticação (4xx) | HTTP 400 Bad Request (1,05 s); `invalid_credentials` — "Invalid login credentials" | OK |
| CT-03 | Usuário inexistente | `usuario-inexistente@example.com` + senha válida | Acesso negado (4xx) | HTTP 400 Bad Request (403 ms); `invalid_credentials` — "Invalid login credentials" | OK |
| CT-04 | Campos vazios | `email: ""` e `password: ""` | Erro de validação (4xx) | HTTP 400 Bad Request (227 ms); `validation_failed` — "missing email or phone" | OK |
| CT-05 | Credenciais inválidas | `email-invalido` + `qualquer-senha` | Mensagem de erro (4xx) | HTTP 400 Bad Request (423 ms); `invalid_credentials` — "Invalid login credentials" | OK |

## Evidências

| Cenário | Print |
|---|---|
| Login válido (access token oculto) | ![CT-01](prints/08-login-valido.png) |
| Senha incorreta | ![CT-02](prints/09-senha-incorreta.png) |
| Usuário inexistente | ![CT-03](prints/10-usuario-inexistente.png) |
| Campos vazios | ![CT-04](prints/11-campos-vazios.png) |
| Credenciais inválidas | ![CT-05](prints/12-credenciais-invalidas.png) |

# Registro dos Testes

Os testes foram registrados em [`planilha/registro_de_testes.xlsx`](planilha/registro_de_testes.xlsx), com os campos: ID do teste, Cenário, Entrada, Resultado esperado, Resultado obtido, Status e Observações. A planilha tem lista suspensa para o Status (Sucesso, Falha, Pendente) e uma aba **Resumo** com os totais calculados por fórmula.

A documentação é importante porque permite reproduzir os testes, comparar o comportamento esperado com o real, detectar regressões e comunicar falhas de forma objetiva.

**Falhas identificadas:** nenhuma falha de execução. Duas observações de comportamento da API:

- No CT-05, a API não validou o formato do e-mail; tratou como `invalid_credentials` em vez de um erro de validação.
- Credenciais incorretas retornam HTTP 400, não 401.

## Evidências

> **Pendente:** adicionar um print da planilha preenchida em `prints/13-planilha-preenchida.png`.

# Resultados Obtidos

- Total de cenários executados: 5
- Cenários aprovados: 5
- Cenários reprovados: 0
- Falhas identificadas: nenhuma (apenas as observações acima)
- Status HTTP: 200 no login válido; 400 nos quatro cenários negativos.
- JSON: o login válido retorna `access_token`, `token_type`, `expires_in`, `expires_at`, `refresh_token` e `user`. Os erros retornam `code`, `error_code` e `msg`.
- Comportamento observado: as respostas de senha incorreta e usuário inexistente são idênticas, o que evita revelar quais e-mails estão cadastrados.

# Conclusão

Os cinco cenários obrigatórios foram executados corretamente e a autenticação do Supabase funcionou como esperado: o login válido devolveu HTTP 200 com `access_token` e os quatro cenários negativos foram rejeitados com HTTP 400 e mensagem de erro em JSON.

**Dificuldades:** configurar as variáveis de Environment e de coleção corretamente e cuidar para não expor tokens e chaves nos prints.

**Falhas e melhorias:** não houve falhas, mas o CT-05 mostrou que o formato do e-mail não é validado neste endpoint. Melhorias possíveis: validar o formato no cliente antes de chamar a API, ampliar os scripts de teste do Postman para checar também `error_code` e `msg`, e acrescentar cenários como senha muito longa ou tentativas repetidas (limite de requisições).

**Importância dos testes caixa cinza:** combinam testes externos de requisições e respostas com conhecimento parcial da API. Assim é possível verificar contratos HTTP, autenticação, validações e tratamento de erros sem acesso ao código-fonte.

# Estrutura do Repositório

```text
.
├── README.md
├── .gitignore
├── postman/
│   ├── Supabase - Modelo.postman_environment
│   └── Supabase - Testes Caixa Cinza de Autenticação.postman_collection
├── prints/
│   ├── login_valido.png
│   ├── senha_incorreta.png
│   ├── usuario_inexistente.png
│   ├── campos_vazios.png
│   └── credenciais_invalidas.png
└── planilha/
    └── registro_de_testes.xlsx
```
