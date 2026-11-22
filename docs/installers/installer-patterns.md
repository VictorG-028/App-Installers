# Padrões do instalador (electron-builder + NSIS)

Última revisão: 2026-11-22 · electron-builder 26.

## Configuração que funcionou (resumo)
- `nsis.oneClick: false`, `perMachine: true`, `allowElevation: true`, `allowToChangeInstallationDirectory: false` — assistido, por máquina, sem escolha de pasta.
- `createDesktopShortcut: 'always'`, `createStartMenuShortcut: true`, `shortcutName` = nome do produto.
- `runAfterFinish: true` → caixa "Executar o …" **marcada** na página final.
- `deleteAppDataOnUninstall: false` (a pasta de dados fica; a caixa do desinstalador decide — [uninstaller-patterns](uninstaller-patterns.md)).
- `language: '1046'`, `multiLanguageInstaller: true`, `installerLanguages: ['pt_BR']`, `displayLanguageSelector: false` → assistente em português.
- `win.requestedExecutionLevel: 'requireAdministrator'`; `win.signtoolOptions` com o certificado e `publisherName`; `publish: [{ provider: 'generic', url, publisherName }]` (o `publisherName` vai para o `app-update.yml` e faz o atualizador **verificar a assinatura**).
- `artifactName: '<produto>-Setup-${version}.exe'` (sem espaço).
- `electronFuses`: `runAsNode:false`, `enableNodeOptionsEnvironmentVariable:false`, `enableNodeCliInspectArguments:false`, `enableEmbeddedAsarIntegrityValidation:true`, `onlyLoadAppFromAsar:true`. (Deixar `grantFileProtocolExtraPrivileges` no padrão: a página é carregada por `file://`.)
- `files` em **lista branca** (`out/**`, `package.json`, sem `*.map`/`*.ts`); o motor entra como `extraResources` (`resources\engine`).
- `extraMetadata: { name: '<produto>-e2e' }` no sabor de ensaio, para o cache do atualizador não colidir ([0010](../decisions/0010-sabor-e2e-para-ensaiar-instalador.md)).

## Personalização por `build/installer.nsh`
Macros usadas: `customInstall` (atalho extra "Atualizar …"), `customUnWelcomePage` (página "Seus dados"), `customUnInstall` (apagar dados/cache). Lições:
- **Declare `Var` só no passe do desinstalador** (`!ifdef BUILD_UNINSTALLER`): no passe do instalador a variável "não referenciada" vira **aviso → erro** (o builder trata avisos como erros).
- O arquivo `.nsh` com acentos precisa de **UTF-8 com BOM**.
- Dentro do contexto "todos os usuários", `$LOCALAPPDATA` aponta para `C:\ProgramData`; use `SetShellVarContext current` para achar a pasta **do usuário** e volte a `all` para atalhos do Menu Iniciar.
- Atualizar roda o desinstalador antigo em silêncio com `--updated`: proteja tudo que não deve acontecer numa atualização com `${ifNot} ${isUpdated}`.

## Build e verificação (automatizados)
Motor → assinatura do exe do motor → bundle → instalador → **verificação**: instalador e exes assinados pelo editor certo, `latest.yml` com a versão certa, lista do `app.asar` **sem** nada proibido (fontes, mapas, docs, testes, metadados pessoais, chaves), `app-update.yml` com feed e `publisherName`, **nenhuma chave privada ou token em texto** nos arquivos empacotados. A saída do build fica **fora** do repositório.

## Ensaio no próprio computador
Veja [0010](../decisions/0010-sabor-e2e-para-ensaiar-instalador.md). Automação dos assistentes reais por **mensagens Win32** (`EnumChildWindows`, `BM_GETCHECK`, `BM_CLICK`) — o UI Automation não enxerga as páginas aninhadas do NSIS, e janelas elevadas só podem ser dirigidas por um processo elevado.
