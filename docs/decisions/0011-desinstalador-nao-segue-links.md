# 0011 — Desinstalador elevado nunca apaga com `RMDir /r` uma pasta que o usuário comum pode escrever

- **Data:** 2026-11-22 · **Estado:** vigente · **Produto de origem:** diskclean

## Contexto
O desinstalador roda como administrador e apaga pastas em `%LOCALAPPDATA%` (o cache do atualizador sempre; os dados do usuário se ele pedir). Essa é uma área onde **qualquer processo
comum do usuário** pode criar arquivos, pastas e **junctions** (links de diretório, sem privilégio nenhum).

## Medição
Teste isolado, em pasta descartável, com o NSIS 3.0.4.1 do electron-builder: uma junction apontando para uma pasta com dois arquivos, e um instalador de teste que só faz
`RMDir /r "<junction>"`. Resultado: a junction **e os arquivos da pasta-alvo** foram apagados. O `RMDir /r` do NSIS **segue** links.

Com `cmd /c rd /s /q`: a junction é removida e o alvo fica intacto, também quando a junction está **aninhada** dentro de uma pasta real e quando é um symlink de diretório.

## Cenário de ataque
Um processo comum troca `%LOCALAPPDATA%\<produto>-updater` por uma junction para qualquer diretório. Na próxima desinstalação (que o administrador autorizou com um UAC), o
conteúdo do diretório escolhido pelo atacante é apagado com privilégio de administrador. É local e depende de alguém desinstalar, mas é uma primitiva clássica de
"apagar diretório arbitrário" e custa pouco evitar.

## Decisão
No script do instalador, todo apagamento recursivo **fora de Program Files** usa uma macro que chama `cmd /c rd /s /q` (e nunca `RMDir /r`). A macro também não age se
`$LOCALAPPDATA` vier vazio. O ensaio do instalador ganhou um passo que **planta** as duas formas de junction (no topo e aninhada), desinstala e confere que os alvos sobreviveram.

## Resíduo conhecido
O template do próprio electron-builder faz `RMDir /r "$APPDATA\<nome>"` quando o desinstalador recebe `--delete-app-data` explícito. É opt-in e só importa se houver algo (e uma junction)
em `%APPDATA%\<nome>`; o diskclean não usa essa pasta. Se um produto usar, apague-a antes pela macro segura.

## Como revisar
Reveja ao trocar a versão do NSIS ou do electron-builder (o comportamento do `RMDir /r` pode mudar; repita a medição) e a cada novo apagamento no script.
