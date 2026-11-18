Projeto 2 — Gestão de Pacotes e Patches (apt no Linux, winget no Windows)
Ficheiro README.md pronto para publicar no GitHub.
Objetivo
Demonstrar como instalar, atualizar e remover programas pela linha de comandos em dois sistemas, e perceber porque manter os pacotes atualizados é uma medida de segurança.
O que é um package manager
Um package manager é um programa que instala, atualiza e remove aplicações por nós, indo buscá-las a repositórios conhecidos e verificando que o ficheiro não foi alterado (assinatura / hash). Isto evita sites clonados e downloads falsos.
• Pacote: um ficheiro com tudo o que um programa precisa (o programa, as configurações, as instruções).
• Repositório: o "armazém" online de onde vêm os pacotes.
• Dependências: outros pacotes de que um programa precisa; o package manager instala-os automaticamente.
Package managers por família de Linux
Família
Distribuições
Formato
Ferramenta
Debian
Ubuntu, Kali
.deb
apt
Red Hat
Red Hat, CentOS
.rpm
yum / dnf
apt — comandos principais (Linux)
sudo apt update              # atualiza o catalogo (nao instala nada)
sudo apt upgrade -y          # instala as atualizacoes do que ja existe
sudo apt install nload       # instala um programa
sudo apt remove nload        # desinstala um programa
apt list --installed         # lista o que esta instalado
• sudo — executa como administrador (superutilizador).
• -y — responde "sim" automaticamente às perguntas.
winget — o package manager do Windows
O Windows também tem um package manager de linha de comandos, o winget, parecido com o apt. Corre-se no PowerShell:
winget upgrade               # mostra os programas com versao nova disponivel
winget upgrade --all         # atualiza todos de uma vez
winget search <nome>         # procura um programa (e confirma o autor)
Nota de segurança: ao instalar algo novo, escrever bem o nome. Existem pacotes falsos com nomes parecidos (técnica de typosquatting). O winget search ajuda a confirmar o autor antes de instalar.
Porque atualizar é segurança
1.  Todos os programas têm vulnerabilidades (falhas no código).
2.  Cada falha recebe um código público, um CVE, que os atacantes também consultam.
3.  Quem faz o programa lança uma correção (um patch) numa versão nova do pacote.
4.  Quem não atualiza fica com a falha aberta mesmo já existindo correção. Muitos ataques reais acontecem em sistemas por atualizar.
O mesmo princípio aplica-se em todo o lado: Windows Update, Microsoft Store, Play Store, App Store — todos corrigem vulnerabilidades antes que sejam exploradas.
Conceitos aprendidos
• Diferença entre update (catálogo) e upgrade (instalar).
• Famílias de pacotes: .deb vs .rpm.
• Dependências e repositórios.
• Vulnerabilidades, CVE e patches.
• Vantagem de segurança de instalar pela fonte oficial em vez de procurar no Google.
Resultado
Capacidade de gerir software com segurança em Linux e Windows pela linha de comandos, percebendo o lado de segurança por trás das atualizações.
