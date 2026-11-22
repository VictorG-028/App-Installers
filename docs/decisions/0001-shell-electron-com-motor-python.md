# 0001 — Interface em Electron + motor Python como processo filho

- **Data:** 2026-11-21 · **Estado:** vigente · **Produto de origem:** diskclean

## Contexto
O diskclean era um app Python (customtkinter) com a lógica de varredura/deleção e a interface no mesmo processo. Queria-se: interface bonita no estilo "HTML", instalador/atualizador/desinstalador de verdade, e **não piorar o tempo de resposta**. O motor (varredura de disco, regras de segurança, deleção) é o ativo valioso e já funciona.

## Alternativas medidas (ver [medições](../measurements/shells-and-engines.md))
1. **Retocar o customtkinter** — bonito é difícil no paradigma de canvas/widgets; descartado como caminho principal.
2. **pywebview + WebView2** — mais leve em disco, mas medido **o mais lento e o mais pesado em RAM** (1,2–1,9 s, 439 MB), com 7–9 processos.
3. **Electron + motor Python congelado (PyInstaller) como processo filho** — escolhido.
4. **Reescrever o motor em TypeScript/Rust** — não agora: o motor é 6,5 mil linhas de regras testadas na prática; reescrever só se uma medição pedir. Fica registrado como roteiro, sem compromisso.

## Decisão
Electron (React + TypeScript + Tailwind) na interface; o motor Python empacotado com PyInstaller `--onedir --console` dentro do instalador, iniciado **oculto** pelo app. A conversa é **JSON-RPC em uma linha por mensagem (NDJSON) por stdin/stdout**, com eventos de progresso do motor para a interface.

## Consequências
- Custo estrutural do Chromium aceito: abrir em ~0,2–0,6 s e ~120–170 MB de memória privada (ver [0009](0009-processos-enxutos-gpu-e-rede.md)). Em troca, listas/cliques ficaram **mais rápidos** que no app antigo.
- O app recusa um motor de **outra versão** (handshake `ready` com versão e protocolo), para que uma atualização interrompida nunca misture versões.
- A lógica que só existia na interface antiga (seleção, filtros, dependências) foi portada para TypeScript com testes; o motor ficou com dados e regras de deleção.
- O protocolo é estável o bastante para, no futuro, elevar **só o motor** (ver [0002](0002-app-elevado-e-por-maquina.md)).

## Como revisar
Reabra se: o tamanho do instalador (~114 MB) virar problema; surgir necessidade de rodar sem o interpretador Python; ou uma medição mostrar que a varredura em outra linguagem ganha o suficiente para justificar a reescrita (ver [backend](../stacks/backend.md)).
