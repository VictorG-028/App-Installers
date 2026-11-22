# 0003 — O que acontece com os dados ao desinstalar e atualizar

- **Data:** 2026-11-21 · **Estado:** vigente

## Decisão
- Os dados do usuário (banco, decisões, notas, configurações) ficam em `%LOCALAPPDATA%\<produto>`, **fora** da pasta de instalação.
- **Atualizar nunca toca em dados** nem em atalhos.
- **Desinstalar mantém os dados por padrão.** O desinstalador tem uma página "Seus dados" com a caixa **"Apagar também meus dados" desmarcada**; só marcada ela apaga a pasta.
- Desinstalação silenciosa (`/S`) mantém os dados, salvo com `--delete-app-data` (para automação).
- O cache do atualizador (`%LOCALAPPDATA%\<produto>-updater`) sempre sai numa desinstalação real.
- O banco tem **migrações** (`PRAGMA user_version`): antes de migrar um banco existente o app guarda uma cópia consistente (`backups\`, mantém as 3 mais recentes); se a migração falhar, a cópia é restaurada.

## Consequências
- Reinstalar depois de desinstalar reaproveita decisões e histórico do usuário.
- Perder o banco é **aceitável** (o valor do produto está nas regras, não nos dados), mas as decisões do usuário são tratadas como insubstituíveis: por isso o backup antes da migração e a gravação atômica dos arquivos de configuração.
- O desinstalador precisa usar o **contexto do usuário** (não o de todos os usuários) para achar a pasta — ver [armadilhas](../pitfalls.md).

## Como revisar
Se um produto tiver dados que não valem manter, o padrão da caixa pode ser outro; o mecanismo (página + `--delete-app-data`) é reaproveitável. Detalhes de implementação: [uninstaller-patterns](../installers/uninstaller-patterns.md).
