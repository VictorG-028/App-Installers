# Armadilhas encontradas

Cada item é algo que **custou tempo** e não é óbvio. Última revisão: 2026-11-22.

## Instalador / NSIS / electron-builder
- **Aviso vira erro:** `Var` declarada mas não usada no passe do instalador → `warning 6001` → build quebra. Declare variáveis do desinstalador dentro de `!ifdef BUILD_UNINSTALLER`.
- **`.nsh` com acentos** precisa de UTF-8 **com BOM**, senão os textos saem quebrados.
- **`$LOCALAPPDATA` no contexto "todos os usuários" é `C:\ProgramData`.** O desinstalador mostrava (e poderia apagar) a pasta errada; use `SetShellVarContext current` para a pasta do usuário.
- **`multiLanguageInstaller: false` ignora `language`** e o assistente sai em inglês; para português use `multiLanguageInstaller: true` + `installerLanguages: ['pt_BR']`.
- O **instalador "um clique"** não tem página final: para "abrir ao terminar" com escolha, use o assistido.
- **O build assinado depende de um servidor de terceiros por padrão:** o electron-builder pede carimbo de tempo (RFC 3161) a `timestamp.digicert.com` para cada arquivo assinado. Quando esse servidor não responde, o `signtool` falha ("timestamp server could not be reached") e o build inteiro cai, mesmo com o GitHub no ar. Com certificado autoassinado confiado só na própria máquina o carimbo não traz nada; defina `ELECTRON_BUILDER_OFFLINE=true` no build (a variável só desliga o carimbo no código de assinatura, não os downloads).
- Nome de asset com **espaço** vira ponto no GitHub e o `latest.yml` deixa de casar.
- O `Uninstall.exe` se copia para `%TEMP%` e retorna: automação deve esperar a pasta sumir.
- `Set-Content -Encoding UTF8` do **Windows PowerShell 5.1 grava BOM**: um `package.json` com BOM não é JSON válido para o builder. Use `[IO.File]::WriteAllText` com `UTF8Encoding($false)`.
- Editar scripts com escapes (`\\`, `\x27`) por `node -e` no shell **corrompe barras invertidas** em silêncio — edite arquivos com a ferramenta de edição, não por substituição em linha de comando.

## Scripts de release em Windows PowerShell 5.1
Cada item abaixo derrubou uma tentativa de release (por sorte, sempre **antes** de publicar: ordene o script para que as verificações venham primeiro e a publicação por último).
- **stderr de comando nativo lança exceção** com `$ErrorActionPreference = 'Stop'`, mesmo com `2>$null` ou `*> $null` (ex.: `gh release view` de uma tag que ainda não existe, "release not found"). Rode o comando com `Continue`, junte o stderr ao texto e confira o código de saída à mão.
- **Nome de função não pode ser igual ao de um executável:** nomes de comando não diferenciam maiúsculas e a função vence o executável. Uma função `Gh` que chamava `& gh` chamava a si mesma (`CallDepthOverflow`); `GP` é alias de `Get-ItemProperty`. Chame o executável com a extensão (`gh.exe`) e dê nomes próprios às funções.
- **`powershell -File` não recebe array:** `-Assets a, b` entrega `b` como argumento posicional solto e falha. Passe uma string única separada por vírgula e divida dentro do script.
- **`Invoke-WebRequest().Content` é `byte[]`** quando o servidor responde `application/octet-stream`, que é o caso de qualquer asset de release do GitHub (inclusive `latest.yml`): decodifique antes de procurar texto.
- **Arquivo `.ps1` com acento precisa de BOM UTF-8**, senão o 5.1 o lê como ANSI e o texto publicado sai com mojibake.
- A verificação do feed deve fazer o que o cliente faz: ler o `latest.yml`, **baixar o instalador**, conferir o sha512 e a assinatura. Conferir só os cabeçalhos HTTP não pega um asset errado.

## Atualizador
- **Sem `publisherName` no `app-update.yml` o electron-updater pula a verificação de assinatura.** Confira no build.
- O provedor `github` pega "a última release do repositório": **quebra com mais de um produto** → `generic` + release-ponteiro.
- Sem empacotar, o electron-updater exige `dev-app-update.yml` (ou `updateConfigPath`).
- App elevado iniciado por UAC **não herda variáveis de ambiente** do processo que o iniciou → para o ensaio, grave a URL do feed no `app-update.yml` do build.

## Segurança do app elevado
- **`--remote-debugging-port`** em app elevado é escalada de privilégio local: o release recusa switches desconhecidos (o Chromium já abriu a porta antes do JS rodar, então recuse e saia cedo).
- O `contextBridge` entrega à página **só `message` e `stack`** de um `Error` (o `name` volta a "Error" e propriedades extras somem): carregue o código de erro no texto (`E_CODE|mensagem`).
- Fuses do Electron fecham `RunAsNode`/`NODE_OPTIONS`/`--inspect`; sem isso um app elevado vira interpretador para qualquer processo local.

## Automação e teste
- **UIPI:** janelas elevadas só podem ser dirigidas por processo elevado (um UAC a mais para o driver).
- O **UI Automation não enxerga** as páginas aninhadas do NSIS; use `EnumChildWindows` + `BM_GETCHECK`/`BM_CLICK`.
- **Captura de ponteiro** não é concedida a eventos sintéticos do CDP: ouça `pointermove` na `window` durante o arraste.
- `Get-ChildItem -LiteralPath … -Include …` não filtra como se espera: liste com `-Force -File` e filtre com `Where-Object`.
- Seletores CSS com barra invertida (`[placeholder="D:\repos"]`) não casam: use `id`.
- Ao rodar testes e2e depois de um build `prod`, lembre que ele **recusa** a porta de depuração — reconstrua o sabor `e2e` antes.
- Um shell parado dentro de `out\...` trava o `rmdir` da próxima compilação (EBUSY).

## Ferramentas
- **TypeScript 7** removeu `baseUrl`; `paths` passa a ser relativo (`./`).
- `npx <bin>` pode falhar ("could not determine executable") dentro de scripts PowerShell: chame o binário de `node_modules\.bin\*.cmd`.
- `.venv\Scripts\pythonw.exe` do `uv` é um trampolim com processo filho: a janela pertence ao filho.
- O postinstall do Electron pode não baixar o binário: rode `node node_modules/electron/install.js`.
- Windows Defender examinando um app novo faz a **primeira** execução a frio ficar lenta (segundos a dezenas de segundos).
