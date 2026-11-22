# Assinatura de código (autoassinada)

Última revisão: 2026-11-22. Decisão e riscos: [0006](../decisions/0006-assinatura-autoassinada.md).

## Criar o certificado (uma vez, sem UAC)
`New-SelfSignedCertificate -Type CodeSigningCert -Subject "CN=VictorG-028 App Installers" -KeyAlgorithm RSA -KeyLength 3072 -HashAlgorithm SHA256 -KeyExportPolicy Exportable -NotAfter (Get-Date).AddYears(5)` (Windows PowerShell 5.1: o módulo PKI).
Exporta o PFX com **senha aleatória**, grava a senha **protegida por DPAPI** (`ConvertFrom-SecureString`, legível só pelo usuário atual no computador atual), exporta o `.cer` público e **remove o certificado do repositório de certificados** — fica só o arquivo exportado. Tudo na pasta do perfil do usuário, com permissão restrita ao usuário.

**A chave privada nunca vai para o repositório, para uma unidade de backup nem para o GitHub.** O `.gitignore` bloqueia `*.pfx`, `*.p12`, `*.cer` e a varredura anti-vazamento bloqueia esses nomes.

## Confiar neste computador (um UAC)
Importa o `.cer` em `LocalMachine\Root` **e** `LocalMachine\TrustedPublisher`. Efeito: UAC mostra o editor, o instalador mostra assinatura válida, e o atualizador valida. Reversível (`-Remove`).

## Assinar
- O electron-builder assina o exe do app, o instalador e o desinstalador (`signtoolOptions.certificateFile/Password`).
- O exe do **motor** (extra resource) é assinado **antes** do empacotamento com `Set-AuthenticodeSignature -HashAlgorithm SHA256` (não depende de `signtool` instalado).
- A senha do PFX é lida do arquivo DPAPI **só durante o build** e removida do ambiente em seguida.
- **Sem carimbo de tempo, de propósito:** o build define `ELECTRON_BUILDER_OFFLINE=true`; sem isso o electron-builder pede o carimbo a `timestamp.digicert.com` e o build cai quando esse servidor está fora do ar ([armadilhas](../pitfalls.md)).

## Conferir
`Get-AuthenticodeSignature <arquivo>`: antes de confiar → `UnknownError` (assinado, raiz não confiável); depois → `Valid`. A verificação do build exige o CN `VictorG-028 App Installers` em instalador, app e motor.

## Perdi a chave / expirou
Criar outra com o **mesmo CN** (`-Force`) e confiar de novo. O atualizador compara o **nome** do editor, não a impressão digital, então apps já instalados continuam atualizando depois que o novo certificado for confiável.

## Se outras pessoas forem instalar
Trocar por certificado de uma AC (ou assinatura gerenciada) e remover o passo de confiança local; reveja o README (o aviso de "editor desconhecido" deixa de valer).
