# Motor (back)

**Escolha atual:** motor Python (3.12+), empacotado com PyInstaller `--onedir --console`, iniciado oculto pelo app e falando **NDJSON por stdin/stdout**. Decisão: [0001](../decisions/0001-shell-electron-com-motor-python.md). Última revisão: 2026-11-22.

## Protocolo (resumo do que funcionou)
- **Uma mensagem JSON por linha.** Saída do motor **só ASCII** (`ensure_ascii`), entrada em UTF-8 — nenhuma página de código do console estraga acento. `stdout` leva só o protocolo (qualquer `print` é redirecionado para `stderr`); `stderr` é o log.
- Requisição `{id, method, params}` → resposta `{id, result}` ou `{id, error:{code,message}}`; **eventos** sem `id` (progresso, resultado por item, fim de tarefa). As respostas **podem chegar fora de ordem** (o motor atende em paralelo): o cliente espera cada `id`.
- **Handshake `ready`** com versão, protocolo, pasta de dados, se está elevado e o esquema do banco. Se a versão diferir da do app, o app recusa.
- **Tarefas longas** devolvem um `jobId` na hora e emitem `progress`/`done`/`error`. Há **matriz de conflitos** (o que pode rodar junto), não "uma tarefa por vez": a interface antiga deixava apagar durante a varredura em segundo plano.
- **Cancelamento** por exceção `BaseException` no *callback* de progresso (para atravessar os `except Exception` do código legado); a deleção só cancela **entre itens**.
- **EOF no stdin = o app morreu**: responder o que já leu, cancelar tarefas e sair.
- Um evento de tarefa pode chegar **antes** da resposta que dá o `jobId`: o cliente guarda e reproduz.

## Processos filhos do motor
`stdin=DEVNULL` em **todo** subprocesso (com o protocolo em stdin, um filho que herde o pipe o consome) e diretório de trabalho neutro. Passar o **ambiente inteiro** ao motor: vários módulos leem `LOCALAPPDATA`, `ProgramFiles*`, `PATH`… e "somem" sem erro se o ambiente for filtrado.

## Teste do motor
`pytest` contra o **processo real** (e depois contra o `.exe` congelado: a mesma suíte), com pasta de dados temporária; migração testada contra uma **cópia** do banco real; deleção só em pastas-cobaia; guarda de dados (o modo teste **recusa** a pasta real).

## Varrer arquivos: linguagem importa? (132 mil arquivos, cache quente)
| Implementação | Tempo |
|---|---|
| Python (`os.scandir`) | 720 ms |
| Node (síncrono / assíncrono 32) | 1 436 ms / 888 ms |
| Rust (1 / 4 / 8 threads) | 550 / 194 / 132 ms |

Node **não enxerga** atributos NTFS (esparso, comprimido, *placeholder* do OneDrive), então não calcula "tamanho alocado" sem código nativo. Rust ganha de 3,7× a 5,4× com vários threads — só vale a reescrita se a varredura virar o gargalo **medido** do produto. No diskclean a varredura completa leva ~23 s dominada por consultas a programas externos, não por varredura de pastas.

## Não esquecer
PyInstaller **não é opaco** ([0004](../decisions/0004-builds-publicos-nao-sao-opacos.md)). O exe do motor é assinado como o resto. O `.venv\Scripts\pythonw.exe` do `uv` é um *trampolim* que cria um processo filho: medições de janela/memória precisam olhar a árvore de processos.
