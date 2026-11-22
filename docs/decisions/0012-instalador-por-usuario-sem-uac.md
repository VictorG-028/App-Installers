# 0012 — App que não precisa de administrador: instalador por usuário, sem UAC

- **Data:** 2026-11-22 · **Estado:** vigente · **Produto de origem:** project-archive-room

## Contexto
O [0002](0002-app-elevado-e-por-maquina.md) fixou "elevado + por máquina" para o diskclean porque limpar disco exige administrador. Um app que só lê e grava arquivos do próprio usuário (catálogo de projetos, notas, bancos locais) não precisa disso, e um UAC a cada abertura ou a cada atualização seria custo sem benefício.

## Decisão
Para apps sem necessidade de administrador:
- Instalador NSIS **por usuário**: `perMachine: false`, `allowElevation: false`, instala em `%LOCALAPPDATA%\Programs\<produto>`, entrada de desinstalação em **HKCU**, atalhos da Área de Trabalho e do Menu Iniciar do usuário. Continua **assistido** (caixa "abrir ao terminar", [0007](0007-instalador-assistido-com-abrir-ao-terminar.md)).
- Executável `requestedExecutionLevel: asInvoker`. **Nenhum UAC** na instalação, na abertura, na atualização nem na desinstalação (medido: o token do instalador nunca estava elevado).
- Tudo o mais é igual ao diskclean: assinatura autoassinada ([0006](0006-assinatura-autoassinada.md)), feed por produto com release-ponteiro ([0005](0005-atualizador-feed-por-produto-no-github.md)), desinstalador que **mantém os dados** por padrão, apagamento seguro sem `RMDir /r` ([0011](0011-desinstalador-nao-segue-links.md)), sabor `e2e` para ensaiar ([0010](0010-sabor-e2e-para-ensaiar-instalador.md)), fuses do Electron e asar íntegro.
- **Atualização só sob pedido** e sem tocar na interface do app: atalho "Atualizar <produto>" no Menu Iniciar abre o app com `--atualizar`; o resultado é mostrado em **caixas nativas** (`dialog.showMessageBox`). O app nunca procura atualização sozinho. O download só é instalado se o sha512 e a assinatura (`publisherName`) conferem; instalação silenciosa por usuário, sem UAC, e o app reabre.

## Consequências
- Um instalador por usuário continua precisando do certificado confiável para que a assinatura apareça como válida; o efeito de "editor conhecido" no UAC deixa de existir porque não há UAC.
- App instalado e app de desenvolvimento compartilham a pasta de dados (`%APPDATA%\<name>`) e a trava de instância única; "apagar meus dados" no desinstalador apaga os dois. Quem testa o instalador usa o sabor `e2e`, de nome (e pasta de dados) diferente.
- O processo de release é o mesmo (instalador primeiro, `latest.yml` por último).

## Como revisar
Reabra se o app passar a precisar de administrador (aí vale o 0002) ou se algum produto quiser verificação automática de atualização (a decisão 0008 do diskclean restringia a Configurações; este produto não tem tela de configurações e usa o atalho).
