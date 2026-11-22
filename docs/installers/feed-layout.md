# Layout do feed no GitHub (um repositório, vários produtos)

Última revisão: 2026-11-22. Decisão: [0005](../decisions/0005-atualizador-feed-por-produto-no-github.md).

## Tags e releases
| Tag | Para quê | Conteúdo |
|---|---|---|
| `<produto>-vX.Y.Z` | arquivo histórico imutável da versão | `<produto>-Setup-X.Y.Z.exe`, `latest.yml` |
| `<produto>-stable` | **ponteiro móvel**: é o feed do atualizador | cópia do `latest.yml` e do instalador **mais novos** |

URL do feed (provedor `generic`): `https://github.com/<dono>/App-Installers/releases/download/<produto>-stable`. O `latest.yml` cita o instalador por nome relativo, resolvido contra essa base.

## Ordem de publicação de uma versão
1. Criar `<produto>-vX.Y.Z` com o instalador e o `latest.yml` (e as notas, em linguagem humana, **sem trechos de código**).
2. Atualizar `<produto>-stable`: subir o **instalador primeiro**, **`latest.yml` por último** (`--clobber`) e remover os assets antigos do ponteiro.
3. Testar o caminho real: um app da versão anterior, em Configurações, deve ver a nova e atualizar.

## Regras
- Nomes de asset **sem espaço** (o GitHub os troca por ponto e o `latest.yml` deixaria de casar).
- Sem blockmap/diferencial: o ponteiro só guarda a versão mais nova.
- Binários **só em Releases**; o repositório git guarda apenas documentação.
- Um produto novo = duas tags novas + o mesmo roteiro. Nada no código do atualizador de um produto depende dos outros.

## Plano B
Se o redirecionamento de download do GitHub falhar com o `generic`: publicar `latest.yml` e o instalador por GitHub Pages do repositório (`/feeds/<produto>/`) e gravar essa URL no `app-update.yml`.
