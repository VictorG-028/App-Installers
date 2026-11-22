# Banco de dados

**Escolha atual:** SQLite local, um arquivo, na pasta de dados do usuário (`%LOCALAPPDATA%\<produto>`), aberto **só pelo motor**. Última revisão: 2026-11-22.

## Por que SQLite
Um app de desktop de um usuário só, sem servidor, que pode perder o banco sem perder o produto (o valor está nas regras). SQLite vem no Python, não precisa de instalação nem de serviço, e é um arquivo copiável. Nenhum SQL de servidor foi considerado: não há concorrência entre máquinas nem rede.

## Padrões adotados
- **Migrações com `PRAGMA user_version`.** Um módulo leva o banco à versão atual; recusa abrir um banco de versão **mais nova** (atualize o app em vez de abrir com um antigo).
- **Backup antes de migrar** um banco existente, pela API de *backup* do SQLite (cópia consistente em qualquer modo de journal), guardando as 3 mais recentes em `backups\`; se a migração falhar, a cópia é restaurada.
- **WAL + `busy_timeout`** (10 s): a interface lê enquanto uma varredura escreve. Uma conexão curta por operação.
- **Decisões do usuário em tabelas próprias** (selos, notas) que **sobrevivem** à limpeza de linhas de varredura — o usuário não deve perder julgamentos porque o scanner reescreveu uma linha.
- **Arquivos de configuração** (JSON) gravados **atomicamente** (arquivo temporário + `os.replace`), e um arquivo ilegível é **copiado para `.corrupt-<hora>`** em vez de ser sobrescrito pelos valores padrão.
- Datas em UTC **ingênuo** (`isoformat`): a interface trata como UTC ao exibir.

## Testes
Migração testada com uma **cópia** do banco real do desenvolvedor (contagens por tabela e conteúdo dos selos idênticos antes/depois), mais um banco "legado" sintético; todos os testes usam pasta de dados temporária.

## Armadilhas
- O modo WAL cria `-wal`/`-shm`: ao copiar o banco manualmente, copie os três (ou use a API de *backup*).
- `json_extract` (JSON1) precisa existir no SQLite empacotado — o do Python 3.12+ tem.
