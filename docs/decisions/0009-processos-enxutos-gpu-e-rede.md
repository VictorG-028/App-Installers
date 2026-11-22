# 0009 — Menos processos: renderização por software e rede no processo principal

- **Data:** 2026-11-22 · **Estado:** vigente · medido em uma máquina Windows 11 de desenvolvimento

## Contexto
Um Electron "oi mundo" mede ~320 MB de working set (4 processos). A interface do diskclean é 2D simples (listas, texto, caixas): não precisa de aceleração por GPU. Medição do app já com a tela principal aberta:

| Configuração | Processos | Working set privado (≈ Gerenciador de Tarefas) |
|---|---|---|
| padrão | 7 | ~290 MB |
| `disable-gpu` | 7 | ~135 MB (o processo de GPU cai de ~175 para ~20 MB) |
| `NetworkServiceInProcess2` | 6 | ~278 MB |
| os dois | 6 | **~120 MB** |

## Decisão
`app.disableHardwareAcceleration()` e `app.commandLine.appendSwitch('enable-features', 'NetworkServiceInProcess2')`, em todos os sabores.

## Consequências
- Desempenho **igual ou melhor** com software: abrir grupo de 300 linhas ~33 ms, rolagem ~220 fps, tela principal ~49 ms (ver [medições](../measurements/shells-and-engines.md)).
- Troca consciente de isolamento: o serviço de rede passa a rodar no processo principal. A página não tem rede (CSP `connect-src 'none'`, sem conteúdo remoto); só o atualizador, no processo principal, usa a rede.
- Para apps com gráficos pesados (canvas/WebGL/vídeo) esta decisão **não** vale: medir antes.

## Como revisar
Reveja a cada troca de versão major do Electron (nomes de *features* do Chromium mudam: `NetworkServiceInProcess2` pode virar outro).
