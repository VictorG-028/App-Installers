# Padrões do desinstalador

Última revisão: 2026-11-22. Decisão: [0003](../decisions/0003-dados-do-usuario-ao-desinstalar-e-atualizar.md).

## Página "Seus dados" (assistente)
Página própria (`nsDialogs`) entre as páginas de boas-vindas e de progresso do desinstalador: texto explicando **onde** ficam os dados (o caminho **real do usuário**) e a caixa **"Apagar também meus dados"**, **desmarcada**. Quem marca, apaga `%LOCALAPPDATA%\<produto>`.
- Não aparece em atualização (`${isUpdated}`) nem em modo silencioso (`${Silent}`).
- Verificada com o assistente real: padrão desmarcado; marcada apaga; texto mostra a pasta certa.

## Linha de comando (automação, winget, testes)
| Comando | Efeito |
|---|---|
| `Uninstall <produto>.exe /S` | silencioso, **mantém** os dados |
| `Uninstall <produto>.exe /S --delete-app-data` | silencioso, **apaga** os dados |
| (executado pelo instalador novo numa atualização) `/S --updated` | nunca toca em dados nem atalhos |

O `--delete-app-data` **nativo** do electron-builder só apaga o `%APPDATA%` (roaming); a pasta de dados do produto, em `%LOCALAPPDATA%`, é tratada pela macro própria.

## O que sai sempre numa desinstalação real
Arquivos do app, entrada em "Aplicativos instalados", atalhos (Área de Trabalho, Menu Iniciar, "Atualizar …"), cache do atualizador. **Fica**: a pasta de dados (salvo a caixa/flag).

## Apagar com segurança (o desinstalador é elevado)
- **Nunca `RMDir /r` em pasta que o usuário comum possa escrever:** o NSIS 3.0.4 segue junctions e apagaria o alvo ([0011](../decisions/0011-desinstalador-nao-segue-links.md)). Use uma macro com `nsExec::Exec '"$SYSDIR\cmd.exe" /c rd /s /q "<pasta>"'`, que remove o link sem seguir (topo ou aninhado), e recuse agir se a variável de pasta vier vazia.
- Em Program Files o `RMDir /r $INSTDIR` do template é aceitável: um usuário comum não consegue plantar links ali.
- Teste isso no instalador real: o passo "junction" do ensaio planta uma junction no caminho do cache e outra dentro da pasta de dados, desinstala com `--delete-app-data` e confere que os alvos continuam lá.

## Detalhes
- O desinstalador não "espera": o `Uninstall.exe` copia-se para `%TEMP%` e sai; testes precisam **esperar a pasta de instalação sumir**.
- Antes de desinstalar, o processo do app (e o do motor) deve estar fechado; o instalador já pede.
- Contexto de pastas: veja o aviso sobre `$LOCALAPPDATA` em [installer-patterns](installer-patterns.md).
