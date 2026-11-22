# 0007 — Instalador assistido (não "um clique") por causa de "abrir ao terminar"

- **Data:** 2026-11-21 · **Estado:** vigente

## Contexto
Queria-se um instalador simples, mas com a caixa **"Abrir o app ao terminar"** — o usuário escolhe.

## Decisão
NSIS **assistido** (`oneClick: false`), por máquina, em português: o instalador "um clique" do electron-builder **não** tem página final, só abre o app sozinho. O assistido tem a página final com a caixa **"Executar o …"** (vem marcada), sem página de escolha de pasta (`allowToChangeInstallationDirectory: false`) e sem licença.

Cria atalho na Área de Trabalho e no Menu Iniciar, mais um atalho extra **"Atualizar …"** (abre o app direto em Configurações — ver [0008](0008-atualizacao-so-dentro-de-configuracoes.md)).

## Consequências
- Um passo a mais (Avançar/Instalar/Concluir) — aceito.
- O idioma do assistente só sai em português com `multiLanguageInstaller: true` + `installerLanguages: ['pt_BR']` (com `false` o assistente sai em inglês; ver [armadilhas](../pitfalls.md)).
- Verificado com o assistente real (automação Win32): caixa final marcada por padrão e o app abre ao concluir.

## Como revisar
Se "um clique" voltar a ser desejável, a alternativa é o instalador one-click com `runAfterFinish` implícito (sem escolha).
