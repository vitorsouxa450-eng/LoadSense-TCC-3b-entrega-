# LoadSense

Sistema de pesagem e gestão de cargas para transportadoras. Ele reúne uma balança embarcada (ESP32), uma API, um banco de dados MySQL, um aplicativo mobile-first e um site de gestão, todos ligados aos mesmos dados.

Projeto desenvolvido como Trabalho de Conclusão de Curso da ETEC de Taboão da Serra (Centro Paula Souza), 2026.

## Sumário

- [Problema e objetivo](#problema-e-objetivo)
- [Público-alvo](#público-alvo)
- [Principais funcionalidades](#principais-funcionalidades)
- [Arquitetura](#arquitetura)
- [Tecnologias](#tecnologias)
- [Métodos de criação: HTML, CSS e JavaScript](#métodos-de-criação-html-css-e-javascript)
- [Modelo de negócio e contas de acesso](#modelo-de-negócio-e-contas-de-acesso)
- [Como abrir o projeto (passo a passo)](#como-abrir-o-projeto-passo-a-passo)
  - [Parte 1: Preparar a pasta e o terminal](#parte-1-preparar-a-pasta-e-o-terminal)
  - [Parte 2: Conferir os pré-requisitos](#parte-2-conferir-os-pré-requisitos)
  - [Parte 3: Instalar as dependências](#parte-3-instalar-as-dependências)
  - [Parte 4: Criar o banco de dados](#parte-4-criar-o-banco-de-dados)
  - [Parte 5: Ligar o sistema](#parte-5-ligar-o-sistema)
  - [Parte 6: Entrar e testar](#parte-6-entrar-e-testar)
  - [Parte 7: Desligar e ligar de novo](#parte-7-desligar-e-ligar-de-novo)
  - [Parte 8: Problemas comuns](#parte-8-problemas-comuns)
  - [Parte 9: Lista de conferência](#parte-9-lista-de-conferência)
  - [Apêndice: Linux e macOS](#apêndice-linux-e-macos)
- [Testes automatizados](#testes-automatizados)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Integrantes](#integrantes)

## Problema e objetivo

Em muitas transportadoras, a pesagem de caminhões é registrada em papel ou em planilhas. Assim, é difícil conferir se o peso da carga bate com o esperado, saber quem fez cada pesagem e acompanhar o transporte até o destino.

O LoadSense trata esse problema em três frentes:

1. **Pesagem na balança**: a balança mede o caminhão vazio, depois com a carga e depois cheio. Cada etapa é enviada ao sistema somente quando a leitura está estável.
2. **Conferência**: o sistema compara o peso total com a soma das etapas e acusa divergência acima da tolerância.
3. **Acompanhamento**: operações, alertas, rastreio do transporte e relatórios ficam disponíveis no site e no aplicativo, com os mesmos dados.

## Público-alvo

- **Empresas de transporte e logística** que pesam cargas e precisam registrar cada pesagem.
- **Gestores de frota e de operação**, que acompanham operações, alertas e relatórios.
- **Operadores de balança**, que registram as pesagens no dia a dia.
- **Professores e avaliadores** do projeto, que podem testar todas as funções com os bancos e as contas descritos neste documento.

## Principais funcionalidades

**Balança (hardware embarcado)**
- Sequência de pesagem: vazio, carga e total, com envio somente após leitura estável.
- Verificação do peso total em relação à soma das etapas, com alerta de divergência.
- Sinal de vida enviado ao sistema a cada 60 segundos, para mostrar se a balança está online.
- Repetição automática do envio em caso de falha de rede.

**Painel e operações**
- Painel com indicadores do período e situação das operações.
- Cadastro de operações: escolha de caminhão, motorista, rota e produto.
- Pesagem, finalização, cancelamento e acompanhamento de cada operação.
- Planilha operacional para consulta e edição.

**Monitoramento e rastreio**
- Acompanhamento do transporte em andamento, com etapas e status.
- Mapa com a rota e a posição do veículo.
- Histórico de operações concluídas.

**Alertas**
- Alertas gerados por divergência de peso e por eventos de rastreio.
- Fluxo de reconhecer, resolver e reabrir cada alerta, com registro de quem fez a ação.

**Relatórios**
- Relatórios por período, com resumo e detalhamento.
- Exportação em CSV e XLSX (planos Profissional e superiores).

**Cadastros e gestão de equipe**
- Caminhões, motoristas, produtos, rotas, postos e balanças.
- Usuários, cargos e permissões. Cada cargo define o que o usuário pode ver e alterar.

**Segurança e conformidade**
- Login com sessão por token (JWT).
- Segundo fator de autenticação (TOTP, compatível com aplicativos autenticadores).
- Recuperação de senha por código enviado por e-mail.
- CPF, CNH e telefone são armazenados cifrados, em atendimento à LGPD.
- Telas de termos de uso, política de privacidade e LGPD no site.

**Planos**
- Essencial, Profissional, Avançado e Enterprise, com limites de caminhões e de usuários por plano.

## Métodos de criação: HTML, CSS e JavaScript

O aplicativo e o site foram criados com as três linguagens da web, cada uma com uma função definida:

**HTML**
- Cada projeto tem um arquivo `index.html` (em `app/` e em `site/`). Ele é a página de entrada, com um elemento `<div id="root">` que recebe a interface e um script que carrega o `main.tsx`.
- As telas são escritas em arquivos `.tsx`, que misturam a marcação HTML com a lógica do componente (JSX). Quando o projeto é executado, o React gera o HTML de cada tela no navegador.
- Os formulários, botões, tabelas e cartões usam as tags HTML padrão (`form`, `input`, `button`, `table`, `section`, entre outras).

**CSS**
- A estilização usa o **Tailwind CSS**, um conjunto de classes prontas (por exemplo, `flex`, `p-4` e `text-sm`), com as configurações em `tailwind.config.js`.
- O arquivo `src/index.css` define as cores do sistema como variáveis, o tema claro e o escuro, as animações de troca de tela e de entrada dos elementos, e o foco visível para navegação pelo teclado.
- O layout do aplicativo é fixo em coluna de 420 pixels, para simular a tela de um celular. O site é pensado para uso em computador e adapta o conteúdo à largura da janela.

**JavaScript e TypeScript**
- A lógica das telas e o cliente da API são escritos em **TypeScript**, uma extensão do JavaScript que acrescenta tipos. Isso ajuda a encontrar erros antes de executar o sistema. Na execução, o Vite converte o código para JavaScript, que o navegador entende.
- A API (`api/`) e os scripts de teste e de criação do banco são escritos em **JavaScript** puro, executados pelo Node.js. Os arquivos têm extensão `.js` ou `.mjs`.
- O firmware da balança é escrito em **C++**, no formato de sketch do Arduino (arquivo `.ino`), e não em JavaScript.

## Arquitetura

```
Balança (ESP32 + HX711 + célula de carga + LCD 16x2)
          │  HTTP (chave do dispositivo)
          ▼
        API (Node.js + Express)  ───────►  Banco MySQL 8.0 (loadsense)
          ▲                                      ▲
          │  HTTP (token JWT)                    │
   ┌──────┴──────┐                               │
   │             │                               │
Aplicativo     Site de gestão ──────────────────┘
(porta 5173)   (porta 5175)
```

O aplicativo e o site não têm banco próprio. Os dois falam com a mesma API, que lê e grava no mesmo banco. Uma pesagem registrada na balança aparece no site e no aplicativo. Um alerta resolvido no aplicativo deixa de aparecer no painel do site.

## Tecnologias

| Área | Tecnologias |
| --- | --- |
| Hardware | ESP32 DevKit C V4, célula de carga de 10 kg, amplificador HX711, LCD 16x2 com interface I2C, buzzer e LEDs |
| Firmware | C++ (framework Arduino para ESP32), bibliotecas HX711 e LiquidCrystal_I2C |
| API | Node.js 20, Express 5, mysql2, jsonwebtoken, bcryptjs, exceljs, nodemailer |
| Banco de dados | MySQL 8.0 (tabelas, views, procedures e triggers) |
| Aplicativo | React 18, TypeScript, Vite, Tailwind CSS, React Router, Leaflet (mapa) |
| Site | React 18, TypeScript, Vite, Tailwind CSS, React Router, Leaflet (mapa) |
| Qualidade | Prettier, verificação de tipos com TypeScript, scripts de teste da API e do banco |

## Modelo de negócio e contas de acesso

No modelo de negócio do LoadSense, a empresa contratante recebe do LoadSense as **contas de administrador** do seu plano. Com essa conta, a empresa cria as demais contas (supervisor, operador e outros usuários) e configura cadastros, balanças e rotas.

Para a avaliação, os dois bancos (demonstração e entrega) já vêm com as contas abaixo. **A senha é a mesma para todas as contas: `LoadSense@2026`.**

| E-mail | Senha | Papel | Para que serve |
| --- | --- | --- | --- |
| `admin@loadsense.local` | `LoadSense@2026` | Administrador | Conta principal, com acesso a todas as telas e cadastros |
| `supervisor@loadsense.local` | `LoadSense@2026` | Supervisor | Acompanha operações e alertas |
| `operador@loadsense.local` | `LoadSense@2026` | Operador de pesagem | Registra pesagens e operações |
| `admin.essencial@loadsense.local` | `LoadSense@2026` | Administrador do plano Essencial | Testa os limites do plano Essencial (3 caminhões e 3 usuários) |
| `admin.profissional@loadsense.local` | `LoadSense@2026` | Administrador do plano Profissional | Testa a exportação de planilhas e os limites do plano Profissional |
| `admin.avancado@loadsense.local` | `LoadSense@2026` | Administrador do plano Avançado | Testa os limites do plano Avançado (25 caminhões) |

Para trocar a senha de uma conta depois de entrar, use a opção **Alterar senha** no aplicativo. Se a senha for esquecida, use **Esqueci minha senha** na tela de login.

Também é possível criar cinco contas extras de aluno (`teste1@loadsense.local` a `teste5@loadsense.local`, com a mesma senha) pelo script `banco/ferramentas/contas_alunos.sql`. Esse passo é opcional e está na Parte 4.

## Como abrir o projeto (passo a passo)

Este passo a passo foi escrito para um computador com Windows 10 ou 11. No apêndice, no fim deste documento, estão os comandos para Linux e macOS.

Siga as partes na ordem. Em cada parte há uma conferência, para saber se a etapa deu certo antes de seguir.

Os comandos `set` e `node` são digitados no **Prompt de Comando**, aberto dentro da pasta do projeto.

> **Sobre as senhas:** neste guia, `SUA_SENHA` é a senha do MySQL que você escolheu na instalação. Onde aparecer `SUA_SENHA`, troque pela sua senha, sem as aspas e sem os colchetes. Não é preciso usar uma senha específica.

### Parte 1: Preparar a pasta e o terminal

1. Copie a pasta do projeto, `LoadSense`, para um caminho curto, como `C:\LoadSense`.
2. Abra essa pasta no Explorador de Arquivos.
3. Clique na barra de endereço, apague o texto, digite `cmd` e aperte **Enter**.
4. Uma janela preta se abre já dentro da pasta. Essa janela é o **Prompt de Comando**.

### Parte 2: Conferir os pré-requisitos

Confirme que estão instalados:

- **Node.js 20 LTS**
- **MySQL 8.0**, com um usuário `root` e uma senha de sua escolha, criada na instalação

No Prompt de Comando, digite:

```
node -v
```

Deve aparecer uma versão começando com `v20`.

Confira também o servidor MySQL. Digite o comando abaixo, troque `SUA_SENHA` pela senha que você escolheu e aperte **Enter**:

```
"C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -h 127.0.0.1 -P 3306 -u root -pSUA_SENHA -e "SELECT VERSION();"
```

Deve aparecer uma tabela com a versão `8.0.x`.

> Se aparecer `Can't connect to MySQL server`, o serviço do MySQL está parado. Abra o **Gerenciador de Serviços** (tecle **Windows + R**, digite `services.msc` e clique em **OK**), procure **MySQL80**, clique com o botão direito e escolha **Iniciar**.

### Parte 3: Instalar as dependências

As dependências são as bibliotecas que o projeto usa. Esta etapa é feita uma vez.

1. No Prompt de Comando, dentro da pasta `LoadSense`, digite:

```
npm run instalar
```

2. Aperte **Enter** e espere, de 3 a 10 minutos. Aparecem muitas linhas de texto. Aguarde até o cursor voltar, sem erros em vermelho com a palavra `ERR!`.

**Conferência:** as pastas `api\node_modules`, `app\node_modules` e `site\node_modules` existem.

Se aparecer `ERR!`, feche a janela, reinicie o computador e repita esta parte.

### Parte 4: Criar o banco de dados

Há duas opções. As duas criam o banco `loadsense` com as mesmas contas. **Cada opção apaga o banco `loadsense` que já existir**, por isso os comandos só rodam com a palavra `--sim` no final.

Escolha **uma** das opções. Você pode trocar de opção depois, rodando o comando da outra.

**4.1. Informar a conexão com o banco**

No Prompt de Comando, dentro da pasta `LoadSense`, digite as linhas abaixo, uma de cada vez, apertando **Enter** no fim de cada uma. Troque `SUA_SENHA` pela sua senha.

```
set LS_DB_HOST=127.0.0.1
set LS_DB_PORT=3306
set LS_DB_USER=root
set LS_DB_PASSWORD=SUA_SENHA
```

> As linhas `set` valem só para esta janela. Se você fechar a janela, digite-as de novo.

**Opção A: banco de demonstração (com dados)**

Use esta opção para ver o sistema com dados: operações, pesagens, alertas, rastreio e relatórios fictícios.

1. Digite:

```
node banco\ferramentas\criar-banco-demonstracao.mjs --sim
```

2. Aperte **Enter** e espere. O comando mostra quatro etapas:

```
1/4 banco anterior removido
2/4 schema carregado
3/4 dados-base carregados
4/4 histórico de demonstração gerado
```

3. Ao final aparece a frase `Pronto. Entre com admin@loadsense.local / LoadSense@2026`. Essa etapa pode levar de 1 a 3 minutos.

**Conferência:** digite a linha abaixo, trocando `SUA_SENHA`. O resultado deve ser um número maior que zero.

```
"C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -h 127.0.0.1 -P 3306 -u root -pSUA_SENHA loadsense -e "SELECT COUNT(*) AS operacoes FROM weighing_operations;"
```

**Opção B: banco vazio (versão de entrega)**

Use esta opção para testar o cadastro do zero, como um cliente faria. O banco tem só a estrutura, as contas de acesso e uma balança por empresa (`BAL-01`). Não há caminhões, motoristas, rotas, operações nem alertas: tudo isso é cadastrado pelo próprio sistema.

1. Digite:

```
node banco\ferramentas\criar-banco-entrega.mjs --sim
```

2. Aperte **Enter** e espere. O comando mostra três etapas:

```
1/3 banco anterior removido
2/3 schema carregado
3/3 contas de acesso criadas
```

**Conferência:** use o mesmo comando de conferência da Opção A. O resultado deve ser `0`.

**Passo opcional: contas de aluno (teste1 a teste5)**

Para criar as cinco contas extras, rode na mesma janela, trocando `SUA_SENHA`:

```
"C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -h 127.0.0.1 -P 3306 -u root -pSUA_SENHA loadsense < banco\ferramentas\contas_alunos.sql
```

Depois, `teste1@loadsense.local` até `teste5@loadsense.local` entram com a senha `LoadSense@2026`.

### Parte 5: Ligar o sistema

O sistema tem três partes que precisam estar ligadas ao mesmo tempo. Cada uma fica em uma janela própria do Prompt de Comando. **Não feche essas janelas enquanto estiver usando o sistema.**

**5.1. Janela 1: a API (servidor)**

1. Abra uma nova janela do Prompt de Comando dentro da pasta `LoadSense`.
2. Digite as linhas abaixo, uma de cada vez, trocando `SUA_SENHA`:

```
cd api
set LS_DB_USER=root
set LS_DB_PASSWORD=SUA_SENHA
set LS_DB_NAME=loadsense
node server/index.js
```

3. Aguarde aparecer `LoadSense API ouvindo em http://0.0.0.0:5174` e `Banco: root@127.0.0.1:3306/loadsense`.

**5.2. Janela 2: o aplicativo**

1. Abra outra janela do Prompt de Comando dentro da pasta `LoadSense`.
2. Digite:

```
cd app
npx vite --host 0.0.0.0 --port 5173
```

3. Aguarde aparecer a linha com `Local: http://localhost:5173/`.

**5.3. Janela 3: o site**

1. Abra outra janela do Prompt de Comando dentro da pasta `LoadSense`.
2. Digite:

```
cd site
npx vite --host 0.0.0.0 --port 5175
```

3. Aguarde aparecer a linha com `Local: http://localhost:5175/`.

**Conferência da API:** abra o navegador e acesse **http://localhost:5174/api/saude**. A página deve mostrar uma resposta, sem erro.

### Parte 6: Entrar e testar

**6.1. Abrir o site (computador)**

1. Abra o navegador (Chrome, Edge ou Firefox).
2. Acesse **http://localhost:5175**.
3. Clique em **Entrar**.

**6.2. Abrir o aplicativo**

1. Acesse **http://localhost:5173** no navegador.
2. O aplicativo aparece em uma coluna de 420 pixels. No celular, ele ocupa a tela inteira.

**6.3. Fazer o login**

1. Em **E-mail**, digite `admin@loadsense.local`.
2. Em **Senha**, digite `LoadSense@2026`.
3. Clique em **Entrar**.

Se o login for aceito, o painel aparece. No banco de demonstração, o painel mostra operações e gráficos. No banco de entrega, o painel aparece vazio, porque ainda não há dados cadastrados.

**6.4. Testar outros perfis**

1. Saia da conta.
2. Entre com um dos e-mails da tabela de [Modelo de negócio e contas de acesso](#modelo-de-negócio-e-contas-de-acesso). A senha é sempre `LoadSense@2026`.

**6.5. Roteiro sugerido para o banco vazio**

1. Entre com `admin@loadsense.local`.
2. Cadastre um caminhão, um motorista, um produto e uma rota.
3. Crie uma operação em **Nova operação** e escolha caminhão, motorista e rota.
4. Na página da operação, use **Gerar chave** para criar a chave da balança, se quiser testar o envio de leituras. A chave aparece uma única vez.
5. Entre com `supervisor@loadsense.local` e confira se a operação aparece na lista.

**6.6. Testar a balança sem o hardware (opcional)**

Com as janelas da Parte 5 abertas, abra outra janela do Prompt de Comando dentro da pasta `LoadSense` e digite, trocando `SUA-CHAVE` pela chave gerada na página da operação:

```
node embarcado\balanca\teste_sem_balanca.mjs http://localhost:5174 SUA-CHAVE
```

O comando envia três pesagens de teste. Antes, a operação precisa estar escolhida para a balança, pelo botão **Receber leituras desta operação**. Se a operação não estiver escolhida, a resposta é `409`.

### Parte 7: Desligar e ligar de novo

**7.1. Desligar**

1. Em cada janela da Parte 5, aperte **Ctrl + C**. Se aparecer `Terminar o arquivo em lotes (S/N)?`, digite `S` e aperte **Enter**.
2. Feche as janelas.

**7.2. Ligar no dia seguinte**

1. Não é preciso repetir a instalação nem recriar o banco.
2. Confira se o MySQL está em execução (Parte 2).
3. Repita a Parte 5 com as três janelas. Se as linhas `set` da Parte 4.1 não estiverem na janela, digite-as de novo.

> **Atenção:** rodar de novo um dos comandos `criar-banco-...` da Parte 4 apaga todos os dados do banco. Faça isso só se quiser começar do zero.

### Parte 8: Problemas comuns

| O que aparece | O que fazer |
| --- | --- |
| `'node' não é reconhecido como um comando` | Reinicie o computador e confira a Parte 2. |
| `Can't connect to MySQL server` | O serviço do MySQL está parado. Veja a Parte 2. |
| `Access denied for user 'root'` | A senha digitada não é a sua senha do MySQL. Confira a linha `set LS_DB_PASSWORD` da Parte 4.1 e o comando de conferência. |
| O comando de criação pede `--sim` | Falta a palavra `--sim` no fim do comando. Digite o comando de novo com `--sim` no final. |
| `ERR!` ao rodar `npm run instalar` | Feche a janela, reinicie o computador e repita a Parte 3. |
| `EADDRINUSE` ou porta em uso | Outro programa usa a porta 5173, 5174 ou 5175. Feche a janela antiga ou reinicie o computador. |
| O login não funciona | A janela da API (Parte 5.1) foi fechada ou não mostra `ouvindo`. Repita a Parte 5.1. |
| O painel aparece vazio | É o esperado no banco de entrega (Opção B). Use a Opção A para ver dados. |
| O mapa não aparece | Confira a conexão com a internet. O restante do sistema continua funcionando. |
| A balança ou o teste recebe `409` | A operação não está escolhida para a balança. Abra a operação e clique em **Receber leituras desta operação**. |
| O login é recusado com a senha correta | A senha foi trocada. Use **Esqueci minha senha** na tela de login. |

### Parte 9: Lista de conferência

Marque cada item ao concluir:

- [ ] Node.js 20 LTS e MySQL 8.0 instalados, e a conferência da Parte 2 deu certo.
- [ ] Dependências instaladas (Parte 3), sem erros `ERR!`.
- [ ] Banco criado com a Opção A ou B, e a mensagem `Pronto` apareceu.
- [ ] As três janelas (API, aplicativo e site) estão abertas e sem erro.
- [ ] `http://localhost:5174/api/saude` abre uma resposta.
- [ ] Entrei com `admin@loadsense.local` e a senha `LoadSense@2026`, e o painel apareceu.
- [ ] Entrei com `supervisor@loadsense.local` e com `operador@loadsense.local`, e os dois funcionaram.

### Apêndice: Linux e macOS

Os mesmos passos valem, com estas diferenças:

- Use o Terminal, dentro da pasta do projeto.
- Troque `set NOME=valor` por `export NOME=valor`.
- Use `/` no lugar de `\` nos caminhos. Por exemplo: `node banco/ferramentas/criar-banco-demonstracao.mjs --sim`.
- Para conferir o MySQL, use `mysql -h 127.0.0.1 -P 3306 -u root -p`, e digite a sua senha quando o terminal pedir.
- Para subir a API, o aplicativo e o site de uma vez, use `./iniciar.sh`, depois de exportar as variáveis `LS_DB_*`.

## Testes automatizados

| O que testar | Comando | Observação |
| --- | --- | --- |
| Tipos do aplicativo e do site | `npm --prefix app run typecheck` e `npm --prefix site run typecheck` | Não depende do banco |
| API: rotas, regras e permissões | Dentro de `api`, rode `npm run testes` | Requer o banco de demonstração. Altera dados. |
| API: LGPD | Dentro de `api`, rode `npm run testes:lgpd` | Requer o banco carregado. Altera dados. |
| Diagnóstico do banco | `mysql -u root -p loadsense < banco\db\04_diagnostico.sql` | Esperado: as verificações retornam `0`, exceto a `11b`, que é informativa |

Os testes da API alteram dados. Para voltar ao estado original, recrie o banco com um dos comandos da Parte 4.

## Estrutura do repositório

| Pasta ou arquivo | Conteúdo |
| --- | --- |
| `api/` | API Node.js/Express: rotas, regras de negócio, autenticação e scripts de teste |
| `app/` | Aplicativo mobile-first (React, TypeScript, Vite) |
| `site/` | Site de gestão (React, TypeScript, Vite) |
| `shared/` | Código compartilhado entre o aplicativo e o site |
| `banco/` | Schema completo do MySQL, scripts de criação, migrações e diagnóstico |
| `embarcado/` | Firmware da balança (ESP32) e instruções de gravação |
| `wokwi/` | Projeto de simulação da balança no Wokwi, sem hardware |
| `iniciar.sh` | Sobe API, aplicativo e site de uma vez (Linux, macOS ou Bash no Windows) |
| `docker-compose.yml` | Configuração para subir tudo com Docker (não testada no ambiente de desenvolvimento) |

## Integrantes

- Levi Nathanael Mendes Sousa
- Pedro Henrique Coneglian de Arruda
- Vitor Silva Souza
- Vítor Oliveira Rangel

Escola: ETEC de Taboão da Serra (Centro Paula Souza), 2026.
