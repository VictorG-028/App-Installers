# Interface (front)

**Escolha atual:** Electron + React + TypeScript + Tailwind, empacotado com electron-builder (NSIS). Decisão e alternativas medidas: [0001](../decisions/0001-shell-electron-com-motor-python.md). Última revisão: 2026-11-22.

## Versões e ferramentas (as que passaram no teste de fumaça)
Electron 45 · electron-vite 6 + Vite 8 · React 19 · Tailwind 4 (`@tailwindcss/vite`) · **TypeScript 7** · Vitest 5 · electron-builder 26 · electron-updater 6.
- TypeScript 7 **removeu `baseUrl`** (use `paths` relativos com `./`) e exige tipos explícitos para importações só de efeito colateral (`import './x.css'` precisa de `vite/client`).
- Antes de fixar versões novas: criar o esqueleto, `typecheck` + `build` + abrir a janela por CDP. Se falhar, recuar para a combinação anterior e registrar aqui.

## Padrões que valeram a pena
- **Estado da página como dados puros e imutáveis** (`Page`, `Row`), com todas as regras (seleção, tri-state, filtros, dependências) em funções testáveis sem DOM. A UI só desenha.
- **Lista virtualizada de passo fixo**: só as linhas visíveis (+ margem) existem no DOM; posição = `índice × passo`. Abrir 300 linhas em ~33 ms, rolagem ~220 fps.
- **Tooltip único** para o app inteiro (um host), com atraso de fechamento e texto selecionável.
- **Diálogos como promessas** (`await ask({...})`): o fluxo de "confirmar → digitar o nome → apagar" fica linear.
- **Tema por variáveis CSS** (um conjunto escuro e um claro) e `prefers-color-scheme`; os componentes só nomeiam *tons* (`ok`, `fail`, `warn`…), nunca cores.
- Texto **selecionável e copiável** por padrão (`user-select`), com `none` só em ícones e controles.
- **Pré-carregar a primeira tela** enquanto o usuário está na anterior, e **aquecer** glifos de emoji/símbolos uma vez na partida (a primeira pintura com emoji é lenta no Windows).
- Erros do motor chegam à página como `E_CODE|mensagem` (a ponte do Electron só leva `message` e `stack`); um wrapper os converte em um `RpcError` com `.code`.

## Testes de interface
- **Unitários (Vitest):** regras puras; formatação comparada com **vetores gerados pelo código original** (o arredondamento por empate do Python difere do `toFixed` do JS); paridade de textos contra a especificação da interface antiga.
- **E2E por CDP** no Electron real, com pasta de dados temporária e dados-fixture; capturas de tela revisadas por olho; orçamento de desempenho como teste.
- Em app elevado, **porta de depuração só nos sabores de teste** — o release a recusa ([0002](../decisions/0002-app-elevado-e-por-maquina.md)).

## Não usar / cuidado
- `pywebview`: o mais lento e o mais pesado em RAM nas medições ([medições](../measurements/shells-and-engines.md)).
- Captura de ponteiro (`setPointerCapture`) **não é concedida** a ponteiros sintéticos do CDP: para "pintar arrastando", ouça `pointermove`/`pointerup` na `window` durante o arraste.
- Aceleração por GPU: desnecessária para UI 2D simples ([0009](../decisions/0009-processos-enxutos-gpu-e-rede.md)).
