# 0008 — Atualização só dentro de Configurações

- **Data:** 2026-11-21 · **Estado:** vigente

## Decisão
O app **nunca** procura, avisa nem instala atualização por conta própria. Ele só consulta o feed quando o usuário **entra em Configurações** (ou aperta **"Verificar atualização"**, ou usa o atalho "Atualizar …" do Menu Iniciar, que abre direto em Configurações).

- Aviso de nova versão: um **banner no topo de Configurações** (qualquer aba) e um bloco "Atualizações" no topo da aba General; **nada** em nenhuma outra tela.
- Um clique em **"Atualizar e reiniciar"** baixa (com progresso), verifica checksum e assinatura, para o motor com calma e instala em silêncio; o instalador reabre o app.
- **Nunca no meio de uma varredura/deleção**: nesse caso o estado é "bloqueado" com mensagem, e o botão volta a funcionar quando a tarefa termina.
- A atualização **não toca** em dados nem em atalhos (ver [0003](0003-dados-do-usuario-ao-desinstalar-e-atualizar.md)).

## Consequências
- Sem verificação em segundo plano: o usuário só sabe de uma versão nova quando vai procurar. É a intenção.
- Falhas têm mensagens próprias: sem conexão, feed ausente, checksum divergente, assinatura recusada.

## Como revisar
Se algum produto precisar de correção urgente de segurança, considerar um aviso opcional (nunca modal) — decisão separada, por produto.
