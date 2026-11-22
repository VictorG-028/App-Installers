# Padrões do atualizador (electron-updater)

Última revisão: 2026-11-22 · electron-updater 6.8. Decisões: [0005](../decisions/0005-atualizador-feed-por-produto-no-github.md), [0008](../decisions/0008-atualizacao-so-dentro-de-configuracoes.md).

## Configuração
`autoDownload=false`, `autoInstallOnAppQuit=false`, `allowDowngrade=false`, `allowPrerelease=false`, `disableDifferentialDownload=true`. Provedor `generic` com a URL da release-ponteiro do produto; `publisherName` no `app-update.yml`.

## Máquina de estados (o que a interface mostra)
`idle → checking → (none | available | error)` · `available → downloading → downloaded → instalando` · `blocked` (há tarefa em andamento) · `refused` (assinatura inválida) · `disabled` (app não empacotado).
Só a tela de Configurações mostra o estado; ela dispara a checagem ao abrir e ao apertar "Verificar atualização".

## Verificação de assinatura (como o electron-updater faz no Windows)
Depois de baixar, roda `Get-AuthenticodeSignature` no instalador e exige **`Status = Valid`** *e* o **CN** do certificado igual a um dos `publisherName` do `app-update.yml`. Consequências:
- O certificado precisa ser **confiável na máquina** (raiz) — por isso o passo de confiança do [signing](signing.md).
- **Sem `publisherName` no `app-update.yml` a verificação é pulada** — o build confere que ele existe.
- Falha → estado `refused`, nada é instalado.

## Instalação
`quitAndInstall(true, true)` (silencioso, reabre o app). Antes, o app para o motor com calma e confere que não há tarefa em andamento. Como o app já está elevado, o instalador roda **sem novo UAC**. O instalador novo executa o desinstalador antigo em silêncio com `--updated` (dados e atalhos intocados).

## Testar sem GitHub
- **Não empacotado:** o electron-updater lê `dev-app-update.yml`; o app de teste escreve um apontando para um servidor local (ou `updateConfigPath`).
- **Empacotado (sabor e2e):** gravar a URL local no `app-update.yml` **na hora do build** (`-FeedUrl`), porque um app elevado iniciado por UAC **não herda variáveis de ambiente** do processo que o chamou.
- Servir uma pasta com `latest.yml` + instalador por HTTP (script simples) e clicar no botão real. Roteiro: [0010](../decisions/0010-sabor-e2e-para-ensaiar-instalador.md).
- Casos testados: sem checagem fora de Configurações; banner e notas; download com progresso e **checksum sha512**; bloqueio durante tarefa; 404 do feed; checksum divergente; "já está na versão mais recente"; atalho `--abrir-atualizacoes`; atualização completa com dados e atalhos preservados.
