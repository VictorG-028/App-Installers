# 0006 — Assinatura de código autoassinada

- **Data:** 2026-11-21 · **Estado:** vigente (revisar se mais alguém for instalar)

## Contexto
Sem assinatura, o UAC mostra "editor desconhecido" a cada abertura do app elevado, e o atualizador não consegue verificar de quem é o instalador que baixou. Um certificado comercial custa (~US$ 10/mês ou mais) e só se justifica se terceiros instalarem.

## Decisão
Certificado **autoassinado** (`CN=VictorG-028 App Installers`, RSA 3072, SHA-256, 5 anos, uso: assinatura de código), **confiado só no computador do autor** (`LocalMachine\Root` e `TrustedPublisher`, um UAC, por script).

## Consequências
- No computador do autor: o UAC e "Aplicativos instalados" mostram o editor, e o `electron-updater` valida (`Get-AuthenticodeSignature` = *Valid* **e** nome do editor igual ao `publisherName` gravado no `app-update.yml`).
- Em outro computador: "editor desconhecido" (esperado e documentado no README).
- **Risco aceito:** quem tiver a chave privada assina código em que *este* computador confia. Mitigação: a chave fica **só no perfil do usuário**, com a senha protegida por DPAPI — nunca no repositório, nunca em unidade de backup, nunca no GitHub.
- Perder a chave: criar outra com o **mesmo CN** e confiar de novo; o atualizador compara o **nome**, não a impressão digital.
- Sem carimbo de tempo (decisão simples, sem dependência de rede no build): a assinatura vale enquanto o certificado vale.

Procedimento completo: [signing](../installers/signing.md).

## Como revisar
Se outras pessoas passarem a instalar: certificado de uma AC comercial (ou assinatura gerenciada) e remoção do passo de confiança local.
