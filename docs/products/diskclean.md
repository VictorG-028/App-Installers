# diskclean

Limpeza de disco para Windows. Varre caches de desenvolvimento, programas instalados, restos de desinstalação e outros candidatos; mostra quanto cada um ocupa, quando foi usado e o que o próprio app sabe sobre o risco; **só apaga o que o usuário marca** e nunca enquanto o programa dono está em execução.

> Esta página descreve o **produto instalado** (o que ele faz no seu computador). Não descreve o catálogo interno, as regras de proteção nem o código — ver [0004](../decisions/0004-builds-publicos-nao-sao-opacos.md).

## Instalar / atualizar / desinstalar
- **Instalar:** `diskclean-Setup-<versão>.exe` da release **`diskclean-stable`** em [Releases](../../../../releases). Instalação por máquina (`C:\Program Files\diskclean`), assistente em português, atalhos na Área de Trabalho e no Menu Iniciar, caixa final "Executar o diskclean" marcada.
- **Abrir:** o app pede permissão de administrador a cada abertura (precisa apagar pastas de sistema e ler o índice de uso do Windows).
- **Atualizar:** em **Configurações**, o aviso de nova versão aparece no topo; ou use o atalho **"Atualizar diskclean"** do Menu Iniciar. O app **não** procura atualizações sozinho.
- **Desinstalar:** em "Aplicativos instalados". O desinstalador pergunta se deve **apagar também seus dados** (caixa desmarcada: por padrão eles ficam).

## Onde ele grava
`%LOCALAPPDATA%\diskclean`: banco `diskclean.db` (decisões, notas, histórico de apagamentos), `settings.json`, `diskclean.log`, `backups\` (cópias anteriores a migrações), cache do Chromium em `_electron\`. Nada fora disso, exceto o que **você** manda apagar.

## O que ele conversa com a rede
Somente o atualizador, e somente quando você abre Configurações: baixa `latest.yml` e, se você pedir, o instalador, de `github.com/VictorG-028/App-Installers/releases/download/diskclean-stable`. A interface não tem acesso à rede.

## Versões
| Versão | Resumo |
|---|---|
| 0.2.0 | Primeira versão com instalador, atualizador e desinstalador: interface em Electron + motor Python; visual novo, regras e textos do app anterior preservados; dados do app anterior reaproveitados (banco migrado com cópia de segurança). |

## Suporte
Uso pessoal, sem garantia. Veja o [aviso do README](../../README.md#aviso).
