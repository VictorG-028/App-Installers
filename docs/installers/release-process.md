# Processo de release

Última revisão: 2026-11-22. O código e os scripts de build ficam no repositório **privado** do produto; aqui está o roteiro, para qualquer produto seguir.

## Roteiro de uma versão
1. **Versão única:** a mesma versão em `package.json` do app e no arquivo de versão do motor (o app recusa um motor de outra versão). Um script confere antes de construir.
2. **Árvore limpa** e testes verdes: unitários, do motor, e2e da interface (inclui orçamento de desempenho).
3. **Build assinado** + **verificação** automática ([installer-patterns](installer-patterns.md)).
4. **Varredura anti-vazamento** (abaixo) — do clone do repositório público **e** dos arquivos da release.
5. **Ensaio do instalador** no sabor e2e (instalar, abrir elevado, atualizar, desinstalar) se o instalador, o `.nsh` ou o atualizador mudaram.
6. **Publicar** na ordem do [feed-layout](feed-layout.md).
7. **Provar a atualização real**: um app da versão anterior atualiza pelo botão, dados e atalhos intactos.
8. Atualizar a página do produto e o log de decisões.

## Varredura anti-vazamento (obrigatória antes de qualquer push ou release)
Nada de dados pessoais nem segredos no repositório público: e-mail, nome completo, nome de usuário do Windows e caminhos de pastas pessoais, nomes de máquina, tokens (`ghp_`, `github_pat_`…), chaves privadas, `.env`/`.pfx`/`.pwd`, o catálogo ou regras de negócio do produto. A varredura olha o **working tree**, **todo o histórico do git** (um segredo removido continua no histórico) e os **assets** da release (chaves, tokens e nomes de arquivos secretos). Os padrões pessoais são montados **em tempo de execução** a partir da configuração local — nunca ficam escritos em nada que se publique. Qualquer achado **bloqueia** a publicação.

## Identidade de commit
Os commits deste repositório usam o e-mail `noreply` do GitHub, não um e-mail pessoal.

## Licença
Nenhum arquivo `LICENSE`: todos os direitos reservados, uso pessoal, sem garantia (README). Decisão provisória; reveja se alguém mais for usar.
