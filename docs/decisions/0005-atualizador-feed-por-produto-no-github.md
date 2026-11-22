# 0005 — Atualizador: feed por produto em um repositório público do GitHub

- **Data:** 2026-11-21 · **Estado:** vigente

## Contexto
Os binários são públicos e ficam em **um repositório para vários produtos**. O `electron-updater` tem o provedor `github`, que pega "a última release do repositório" — **quebra** assim que existe mais de um produto.

## Decisão
- Provedor **`generic`** apontando para uma release-ponteiro **por produto**: `…/releases/download/<produto>-stable`.
- Releases arquivadas por versão: `<produto>-vX.Y.Z`. A release `<produto>-stable` guarda **cópias** do `latest.yml` e do instalador mais novos.
- Ordem de publicação: instalador primeiro, **`latest.yml` por último** (o cliente nunca vê um `latest.yml` apontando para arquivo ausente).
- Sem atualização diferencial (`disableDifferentialDownload`): um instalador por release, sem depender do blockmap da versão anterior.
- O nome do arquivo não pode ter espaço (o GitHub troca por ponto): `<produto>-Setup-<versão>.exe`.
- **Plano B**, se o redirecionamento de download do GitHub falhar com o `generic`: servir o feed por GitHub Pages do próprio repositório.

Detalhes e comandos: [feed-layout](../installers/feed-layout.md).

## Consequências
Cada produto novo só precisa de duas tags e do mesmo roteiro. O ponteiro é mutável (por isso `--clobber`), as versões arquivadas não.

## Como revisar
Se o GitHub passar a limitar downloads anônimos de assets ou o repositório precisar ficar privado, migrar o feed (Pages, bucket) mantém o cliente igual — muda só a URL gravada no `app-update.yml`.
