# 0002 — App elevado, instalado por máquina em Program Files

- **Data:** 2026-11-21 · **Estado:** vigente · **Produto de origem:** diskclean

## Contexto
Um limpador de disco precisa apagar pastas de sistema, desinstalar programas e ler o índice de uso do Windows (Prefetch), o que exige administrador. O app antigo já rodava inteiro elevado (UAC a cada abertura).

## Decisão
- O executável do app declara `requireAdministrator`: **um UAC a cada abertura**, como antes.
- O instalador é **por máquina** (Program Files), não por usuário.
- A conversa interface↔motor é por mensagens JSON, o que **permite elevar só o motor no futuro** sem reescrever a interface.

## Consequências
- Como o app já está elevado, o motor **não** precisa do caminho "script PowerShell elevado em lote" que existia (e que gravava um `.ps1` em pasta gravável pelo usuário para executar como administrador — um vetor de escalada). Esse caminho deixa de ser usado.
- Um app elevado com Chromium exige endurecimento (obrigatório, não opcional):
  - `contextIsolation`, `sandbox`, sem `nodeIntegration`, CSP restrita, **nenhum conteúdo remoto**, navegação e `window.open` bloqueados, permissões negadas;
  - IPC com checagem do remetente e **lista branca** de métodos;
  - *fuses* do Electron: sem `RunAsNode`, sem `NODE_OPTIONS`, sem `--inspect`, só carrega do `asar` com integridade validada;
  - **o build de release recusa qualquer opção de linha de comando que não conheça** (`--remote-debugging-port`, `--inspect`, `--js-flags`…): uma porta de depuração em app elevado é escalada de privilégio local.
- O atualizador herda a elevação: o instalador roda silencioso, sem novo UAC.

## Risco residual conhecido: dados do usuário consumidos por um processo elevado
Um app elevado que **lê dados de uma pasta que o usuário comum pode escrever** herda o risco desses dados. No diskclean, o motor executa, ao apagar, comandos guardados nas linhas do banco
local (`%LOCALAPPDATA%\<produto>`): um processo comum que altere uma linha poderia fazer um comando rodar com privilégio quando o usuário clicar em "Apagar". Não foi corrigido na 0.2.0 porque as saídas
mexem em permissões ou comportamento: restringir a escrita da pasta de dados a administradores (o app é sempre elevado e o único escritor) ou, no momento de apagar, refazer o comando a partir do
catálogo em vez de confiar no banco. **Lição para os próximos produtos:** se o processo é elevado, trate todo arquivo de dados em pasta de usuário como entrada não confiável.

## Como revisar
Reabra se o UAC a cada abertura incomodar: o caminho é elevar só o motor sob demanda (a interface já fala JSON com ele) e deixar a interface sem privilégio.
