# 0010 — Sabor "e2e" para ensaiar instalar / atualizar / desinstalar sem risco

- **Data:** 2026-11-22 · **Estado:** vigente

## Problema
Testar o desinstalador "apagando dados", ou uma atualização, no app **real** instalado põe em risco os dados reais do usuário.

## Decisão
Cada produto tem dois sabores de build a partir do mesmo código:

| | `prod` (release) | `e2e` (ensaio) |
|---|---|---|
| Nome / pasta | `<produto>` | `<produto>-e2e` |
| AppId e GUID de desinstalação | os do produto | próprios (convivem) |
| Dados | `%LOCALAPPDATA%\<produto>` (fixo; variável de ambiente é **ignorada**) | `%LOCALAPPDATA%\<produto>-e2e` (ou `DISKCLEAN_DATA_DIR`) |
| Cache do atualizador | `<produto>-updater` | `<produto>-e2e-updater` |
| Porta de depuração / DevTools | **recusadas** | permitidas |
| Motor | — | modo teste: **recusa** abrir a pasta de dados real |

O `NSIS` usa `${APP_FILENAME}` para o nome da pasta de dados; no sabor e2e isso é `<produto>-e2e` — **o ensaio nunca consegue apagar os dados reais**.

## O que o ensaio cobre (roteiro automatizado)
Instalação silenciosa (arquivos, registro, atalhos, assinatura válida) · abertura elevada testada por CDP · desinstalar mantendo dados · reinstalar sobre os dados mantidos · desinstalar com `--delete-app-data` · **atualização real** de uma versão para a seguinte por um feed local · assistentes reais (instalar e desinstalar) dirigidos por mensagens Win32 (a página "Seus dados" e a caixa final).

## Como revisar
Mantenha o roteiro executável junto com o instalador do produto; a cada mudança no `.nsh` ou na configuração do builder, rode-o inteiro.
