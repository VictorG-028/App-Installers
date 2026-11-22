# 0004 — Builds públicos não são opacos: o que isso muda

- **Data:** 2026-11-21 · **Estado:** vigente

## Fato
Nenhum build desktop esconde a lógica de quem tem o arquivo:
- PyInstaller guarda bytecode `.pyc` extraível; o `app.asar` do Electron é um arquivo comum, sem criptografia.
- Ofuscar ou compilar para bytecode V8 só aumenta o trabalho de quem quiser ler.

## Decisão
- Publicar **somente binários**, e **não** prometer sigilo da lógica nem de qualquer dado que esteja dentro do app (catálogos, listas embutidas, textos).
- O que o autor **aceita** que vaze deve ser decidido **antes**, produto a produto. No primeiro produto foi aceito: o catálogo de itens específico da máquina de desenvolvimento e um endereço de e-mail padrão nas configurações (indesejável, aceito).
- O que **nunca** pode ir no build: segredos (tokens, chaves, senhas), a chave de assinatura, fontes, mapas, documentos internos, arquivos de metadado pessoal. Uma verificação automática roda a cada build (ver [release-process](../installers/release-process.md)): lista do `app.asar`, padrões de chave/token em todos os arquivos empacotados.
- O código-fonte fica em repositório **privado**, fora do GitHub.

## Consequências
Regras de negócio que não possam vazar não devem ir no app; devem ficar em serviço próprio (fora do escopo destes instaladores).

## Como revisar
Se um produto precisar proteger algo de verdade, a resposta é mover isso para um servidor — não ofuscar.
