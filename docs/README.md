# Documentação

Última revisão geral: **2026-11-22** (criação, a partir da construção do primeiro instalador, o do diskclean).

Regras desta documentação:

- Cada decisão é um arquivo em `decisions/` com **data, contexto, decisão, consequências e como revisar**. Decisão mudou → novo arquivo que a substitui (e o antigo aponta para o novo); nada é reescrito em silêncio.
- Números de medição dizem **onde e como** foram medidos. Eles envelhecem: reveja ao trocar de versão de Electron, Windows ou do motor.
- **Nada pessoal, nada do catálogo/regra de negócio de um produto, nenhum segredo.** Antes de cada publicação roda-se uma varredura anti-vazamento (ver [release-process](installers/release-process.md)).

## Índice

### Decisões
| # | Decisão |
|---|---|
| [0001](decisions/0001-shell-electron-com-motor-python.md) | Interface em Electron + motor Python como processo filho |
| [0002](decisions/0002-app-elevado-e-por-maquina.md) | App elevado, instalado por máquina em Program Files |
| [0003](decisions/0003-dados-do-usuario-ao-desinstalar-e-atualizar.md) | O que acontece com os dados ao desinstalar e atualizar |
| [0004](decisions/0004-builds-publicos-nao-sao-opacos.md) | Builds públicos não são opacos: o que isso muda |
| [0005](decisions/0005-atualizador-feed-por-produto-no-github.md) | Atualizador: feed por produto em um repositório público do GitHub |
| [0006](decisions/0006-assinatura-autoassinada.md) | Assinatura de código autoassinada |
| [0007](decisions/0007-instalador-assistido-com-abrir-ao-terminar.md) | Instalador assistido (não "um clique") por causa de "abrir ao terminar" |
| [0008](decisions/0008-atualizacao-so-dentro-de-configuracoes.md) | Atualização só dentro de Configurações |
| [0009](decisions/0009-processos-enxutos-gpu-e-rede.md) | Menos processos: renderização por software e rede no processo principal |
| [0010](decisions/0010-sabor-e2e-para-ensaiar-instalador.md) | Sabor "e2e" para ensaiar instalar/atualizar/desinstalar sem risco |
| [0011](decisions/0011-desinstalador-nao-segue-links.md) | Desinstalador elevado nunca apaga com `RMDir /r` pasta gravável pelo usuário (junctions) |

### Stacks
[Interface](stacks/frontend.md) · [Motor](stacks/backend.md) · [Banco de dados](stacks/database.md)

### Instalador, atualizador, desinstalador
[Padrões do instalador](installers/installer-patterns.md) · [Atualizador](installers/updater-patterns.md) · [Desinstalador](installers/uninstaller-patterns.md) · [Assinatura](installers/signing.md) · [Layout do feed](installers/feed-layout.md) · [Processo de release](installers/release-process.md)

### Medições e armadilhas
[Interface e motor: medições](measurements/shells-and-engines.md) · [Armadilhas](pitfalls.md)

### Produtos
[diskclean](products/diskclean.md)
