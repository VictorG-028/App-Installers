# project-archive-room

Catálogo de projetos de software em uma tela: fichas ou lista compacta, filtro por status e tag, busca com tolerância a erro de digitação, pastas virtuais, pessoas, edição de status/tags/descrição. Lê os arquivos de metadado `.project-meta.yaml` dos seus projetos e os guarda num banco local, para continuar funcionando com o disco dos projetos desconectado.

## Instalação
Baixe `project-archive-room-Setup-<versão>.exe` na [aba Releases](https://github.com/VictorG-028/App-Installers/releases) (release `project-archive-room-v<versão>`). **Por usuário, sem UAC**, em `%LOCALAPPDATA%\Programs\Project Archive Room`. O instalador é assinado com o certificado autoassinado do [signing](../installers/signing.md): para o Windows mostrar o editor como confiável, o certificado precisa estar instalado como confiável na máquina.

## Onde ficam os dados
`%APPDATA%\project-archive-room\` — `archive-db.json` (banco, com `schemaVersion`), `backups\` (as 10 cópias mais recentes, no máximo uma por minuto), `config.json` (as pastas-raiz onde procurar `.project-meta.yaml`), `logs\`. O desinstalador **mantém** essa pasta, a menos que você marque "Apagar também meus dados". Nenhum arquivo `.project-meta.yaml` é tocado pelo instalador ou pelo desinstalador.

## O que ele conversa com a rede
Nada, exceto quando **você** abre o atalho "Atualizar Project Archive Room" do Menu Iniciar: baixa `latest.yml` e, se você confirmar, o instalador, de `github.com/VictorG-028/App-Installers/releases/download/project-archive-room-stable`.

## Como o banco funciona
- Na primeira abertura importa (somente leitura) os `.project-meta.yaml` das raízes, o estado de pastas virtuais/pessoas/status criados e o glossário de tags. Depois da importação o **banco manda**: editar um yaml à mão não muda um projeto já importado; projetos novos aparecem sozinhos. Status, tags e pessoas editados no app são espelhados no yaml quando a pasta está acessível (nunca descrição nem nome).
- Cada projeto tem **um** caminho. Se a pasta não existe, "Abrir pasta" avisa e oferece escolher a nova (a pasta precisa conter o `.project-meta.yaml`; o caminho anterior é esquecido; uma pasta que já é outro projeto pode ser "adotada").

## Versões
| Versão | Resumo |
|---|---|
| 0.4.0 | Primeira versão com instalador, desinstalador e atualizador (por usuário, sem UAC); banco local com importação; aviso de caminho não encontrado com atualização e adoção; descrição editável no app; fundo animado para o status "Estável". |

## Suporte
Uso pessoal, sem garantia. Veja o [aviso do README](../../README.md#aviso).
