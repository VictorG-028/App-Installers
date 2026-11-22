# App-Installers

Instaladores prontos para Windows — **somente os binários** (na aba [Releases](../../releases)) — mais a **documentação das decisões técnicas** por trás deles: que tecnologias escolher para interface, motor e banco de dados, e como projetar instalador, atualizador e desinstalador. **Não há código-fonte aqui.**

O repositório serve a vários produtos; cada um tem as suas releases. A documentação é viva: cada decisão tem data e um campo "como revisar", e é atualizada quando se descobre algo novo.

## Produtos

| Produto | O que é | Instalar |
|---|---|---|
| **diskclean** | Limpeza de disco para Windows: varre caches, programas e restos, mostra o que cada coisa ocupa e só apaga o que você marcar. | Baixe `diskclean-Setup-<versão>.exe` da release `diskclean-stable` (a mais recente) em [Releases](../../releases). Detalhes em [docs/products/diskclean.md](docs/products/diskclean.md). |

## Como instalar

1. Baixe o `.exe` de instalação da release mais recente do produto.
2. Execute. O Windows pede permissão de administrador (o instalador é por máquina e o app também roda elevado).
3. Siga o assistente (português). No fim, a caixa **"Executar o …"** vem marcada.
4. Para atualizar depois: abra o app → **Configurações** → o aviso de nova versão aparece lá (ou use o atalho "Atualizar …" do Menu Iniciar).

## Confiança e segurança — leia antes de instalar

- Os instaladores são **assinados com um certificado autoassinado** (`CN=VictorG-028 App Installers`). Em um computador que não o conhece, o Windows mostra **"editor desconhecido"** (e o SmartScreen pode avisar). Isso é esperado: não há autoridade certificadora comercial por trás. Se você não conhece o autor, não instale.
- O que o app faz no seu computador é descrito na página do produto. O atualizador só consulta o GitHub quando **você** abre as Configurações, e só instala uma versão cuja assinatura confere com a do editor.
- Como funciona a assinatura, o risco aceito e como confiar no certificado (só no computador do autor): [docs/installers/signing.md](docs/installers/signing.md).

## Documentação

Comece pelo [índice](docs/README.md). Os pontos de entrada:

- [Decisões (ADRs)](docs/decisions/) — o que foi decidido, por quê, e como revisar.
- [Stacks: interface, motor e banco](docs/stacks/) — o que usar e quando, com medições.
- [Instalador, atualizador, desinstalador, assinatura e feed](docs/installers/).
- [Medições](docs/measurements/shells-and-engines.md) e [armadilhas](docs/pitfalls.md) encontradas.

## Aviso

Uso pessoal, **sem garantia de qualquer tipo**. Todos os direitos reservados: este repositório **não** concede licença de reutilização, redistribuição ou engenharia reversa dos binários. Os textos de documentação podem ser lidos e citados com atribuição.
