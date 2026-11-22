# Medições: interface (shell) e motor

Medido em **uma** máquina Windows 11 de desenvolvimento, cache quente, 5 execuções (mediana). Os números envelhecem: reveja ao trocar de versão de Electron/Windows/motor. Última revisão: 2026-11-22.

## Interfaces candidatas (app "oi mundo" equivalente, janela visível)
| Interface | Abrir | RAM (working set / privado) | Processos | Tamanho |
|---|---|---|---|---|
| customtkinter (Python) | ~510 ms | 57 MB / 34 MB | 2 | 14 MB (onefile) |
| Electron 33 | 0,8–1,1 s | 323 MB / 207 MB | 4 | 268 MB descompactado |
| pywebview + WebView2 | 1,2–1,9 s | 439 MB / 230 MB | 7–9 | 28 MB (onedir) |

Primeira execução **a frio** do Electron (antivírus examinando os arquivos novos): 18,6 s no teste de "oi mundo". Renderizar 1 000 linhas em HTML: ~3 ms (no Tk: 47 ms para 300).

## O diskclean 0.2.0 (Electron 45 + motor Python), mesma máquina
Contra os números do app antigo; tela de 980×640; grupo de 300 linhas.
| Métrica | App antigo | Novo | Teto acordado |
|---|---|---|---|
| Abrir grupo de 300 linhas (sem cache) | 47 ms | **~33 ms** | 50 ms |
| Trocar para grupo em cache | 8 ms | **~5 ms** | 16 ms |
| Clique em caixa de seleção | < 1 ms | **0,4 ms** | 5 ms |
| Rolagem contínua | ~120 fps | **~220 fps** | ≥ 60 fps |
| Redimensionar a janela | 55 ms | **~3 ms** | 55 ms |
| Abrir Configurações | 25 ms | **~10 ms** | 40 ms |
| Tela principal | 65 ms | **~49 ms** | — |
| Janela visível (Electron sem empacotar) | ~510 ms | **~215–250 ms** | 1 200 ms |
| Janela visível (instalado, elevado) | — | 520–594 ms | — |
| Motor pronto | — | ~140–220 ms (isolado); ~0,9–1,0 s (instalado, junto da janela) | 1 500 ms |
| Memória privada (Gerenciador de Tarefas) | ~34 MB | **~120 MB ocioso / ~170 MB após teste pesado**, 6 processos | 350 MB |
| Instalador / instalado | — | ~114 MB / ~385 MB | — |

Como se chegou lá: lista virtualizada, tela principal pré-carregada, glifos aquecidos, GPU por software e rede no processo principal ([0009](../decisions/0009-processos-enxutos-gpu-e-rede.md)). Varredura completa: ~22,6–23,1 s na CLI antiga, 22,5–24,0 s pelo motor via JSON-RPC, 23,5–23,7 s pelo motor congelado (±5%).
O motor congelado (PyInstaller `--onedir`, console): 25,7 MB, pronto em 140–220 ms.

## Varredura de pastas por linguagem
Ver [stacks/backend](../stacks/backend.md) (Python 720 ms · Node 888–1 436 ms · Rust 550/194/132 ms com 1/4/8 threads, 132 mil arquivos).

## Como reproduzir
Teste de desempenho automatizado no Electron real por CDP, com dados-fixture e orçamento por métrica (falha se piorar); memória pela árvore de processos usando o contador *Working Set - Private*.
