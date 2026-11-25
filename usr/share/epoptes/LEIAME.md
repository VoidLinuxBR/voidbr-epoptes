# Epoptes no VoidBR e no Chili Linux

O Epoptes é um programa para administrar e acompanhar um laboratório de informática: o professor vê as telas dos alunos em miniatura, observa ou assume uma tela, transmite a própria tela, manda mensagens, bloqueia telas, executa comandos e liga, reinicia ou desliga as máquinas.

Esta versão é a do Epoptes oficial (epoptes.org) com as mudanças do VoidBR: suporte a Wayland nos clientes, funcionamento com runit (Void) e systemd (Arch/Chili), diagnóstico, selos de estado e outras melhorias descritas abaixo.

## Pacotes

| | Void (VoidBR) | Arch (Chili Linux) |
|---|---|---|
| Servidor (máquina do professor) | voidbr-epoptes | chili-epoptes |
| Cliente (máquinas dos alunos) | voidbr-epoptes-client | chili-epoptes-client |

Repositórios: github.com/voidlinuxbr/voidbr-epoptes e github.com/voidlinuxbr/voidbr-epoptes-client.

## Instalação do servidor

**Void:**

    sudo xbps-install -S voidbr-epoptes
    sudo ln -s /etc/sv/epoptes /var/service/

**Arch/Chili:**

    sudo pacman -S chili-epoptes

No Arch, a instalação já habilita e inicia o serviço e cria o certificado.

**Grupo epoptes:** só quem está no grupo epoptes pode abrir a interface. A instalação, no Void e no Arch, cria o grupo e põe nele o usuário que instalou. Para outro usuário:

    sudo usermod -aG epoptes nome-do-usuario

O grupo só vale depois de sair da sessão e entrar de novo; no GNOME, reinicie a máquina (o systemd --user continua rodando com os grupos antigos). Se abrir o epoptes antes disso, ele mostra um aviso explicando.

**Certificado:** o servidor usa o certificado /etc/epoptes/server.crt, criado na instalação. Se ele for recriado (por exemplo, reinstalando o servidor do zero), cada cliente precisa buscar o novo (veja "Instalação do cliente").

**Porta:** o servidor escuta na porta TCP 789 (PORT= em /etc/default/epoptes). Libere essa porta no firewall do servidor para a rede dos clientes.

## Instalação do cliente

**Void:**

    sudo xbps-install -S voidbr-epoptes-client

**Arch/Chili:**

    sudo pacman -S chili-epoptes-client

Depois, em cada cliente:

1. Indique o servidor em /etc/default/epoptes-client, na linha SERVER= (nome ou IP; um IP do Tailscale também serve). Sem isso, o cliente procura uma máquina chamada "server".
2. Busque o certificado do servidor:

        sudo epoptes-client -c

3. Habilite o serviço do cliente. No Void:

        sudo ln -s /etc/sv/epoptes-client /var/service/

   No Arch a instalação já habilita; se precisar:

        sudo systemctl enable --now epoptes-client

4. Entre na sessão gráfica. A conexão da sessão sobe sozinha no login (/etc/xdg/autostart/epoptes-client.desktop).

## Como o cliente se conecta

Cada cliente abre duas conexões com o servidor:

- **Conexão do root**, pelo serviço (runit ou systemd). Existe desde o boot, mesmo sem ninguém logado. É por ela que funcionam desligar, reiniciar e o terminal de root.
- **Conexão da sessão**, aberta no login do usuário. É por ela que funcionam a miniatura, observar, assistir, mensagens e bloquear a tela.

Se só existe a conexão do root, a máquina aparece com o ícone "std" e sem miniatura.

## Usando a janela principal

### Clientes e grupos

A coluna "Grupos" tem "Clientes detectados" (todos os conectados) e os grupos que você criar com o botão +. Para pôr uma máquina num grupo, arraste-a para o grupo ou use o item de adicionar ao grupo, no menu Clientes. O menu Etiquetas muda o texto embaixo de cada máquina (nome da máquina, do usuário ou os dois). A barra embaixo, com as lupas, muda o tamanho das miniaturas.

Sem nenhuma máquina selecionada, os comandos valem para todas as do grupo; com uma ou mais selecionadas, só para elas.

### Selos de estado

Embaixo do nome de cada máquina aparece um selo:

| Selo | O que quer dizer |
|---|---|
| Wayland · Hyprland, X11 · XFCE... | tudo certo; tipo de sessão e ambiente |
| Sem usuário logado | só a conexão do root; ninguém entrou na sessão gráfica |
| ⚠ Sessão não conectou (usuário) | alguém entrou, mas a sessão não conectou |
| ⚠ Sessão duplicada ×N | o mesmo usuário com mais de uma conexão de sessão |
| ⚠ Sem conexão do root | o serviço epoptes-client da máquina não está conectado |
| ⚠ Miniatura falhou | a captura da tela falhou no cliente; o motivo aparece na dica |

O aviso "Sessão não conectou" costuma ser o grupo epoptes, o autostart ou o certificado, e pode levar até uns 20 segundos para aparecer depois do login.

### Dica

Parando o mouse sobre uma máquina aparece uma dica com IP, MAC, versão do cliente, estado das duas conexões, usuários logados, se o Observar funciona naquela máquina e, quando há problema, o que fazer.

### Menu Clientes

- **Diagnosticar cliente...** roda o epoptes-client --check na máquina selecionada e mostra o resultado colorido, com os botões Rodar de novo e Copiar. Precisa do cliente atualizado (com o --check).
- **Reconectar sessão** refaz a conexão da sessão da máquina selecionada: ela some da lista por alguns segundos e volta. Numa sessão duplicada, só encerra a conexão escolhida; a dica diz qual é a mais antiga (a que deve ficar).
- **Informações** e **Desempenho da rede** são as do Epoptes original.

### Observar e Assistir

Nos menus, Observar é **Monitorar usuário** e Assistir é **Auxiliar usuário**; o duplo clique numa máquina é o Auxiliar usuário. Observar só mostra a tela do aluno; Assistir deixa usar o mouse e o teclado dele. Funciona em X11 (x11vnc), em Hyprland, Sway e outros compositores wlroots (wayvnc) e no GNOME com Wayland (servidor VNC do próprio GNOME; veja abaixo). Antes de abrir, o epoptes avisa as máquinas que não têm como atender e por quê, e as deixa de fora.

### Zoom

Os botões A−, 100% e A+, à direita da barra de ferramentas, e os atalhos Ctrl +, Ctrl − e Ctrl 0 aumentam e diminuem a fonte e os ícones de todas as janelas do epoptes, de 70% a 200%. O tamanho escolhido fica salvo (font_scale em ~/.config/epoptes/settings).

### Ajuda

Ajuda > Manual do VoidBR (ou F1) mostra este texto. Ajuda > Sobre mostra os créditos.

## Suporte a Wayland nos clientes

| Ambiente do aluno | Miniatura | Observar/Assistir |
|---|---|---|
| X11 (qualquer ambiente) | sim | sim (x11vnc) |
| Hyprland, Sway, river, labwc (wlroots) | sim (grim) | sim (wayvnc) |
| GNOME com Wayland | não (bloqueada pelo GNOME 41+) | sim, pelo VNC do GNOME |
| KDE Plasma com Wayland | spectacle (não testado) | não |

No GNOME 41 ou mais novo, a captura de tela fica restrita aos componentes do próprio GNOME, por isso a miniatura não funciona lá. As outras funções (mensagem, bloquear tela, transmitir tela, terminal) seguem o Epoptes original e não foram testadas no Wayland.

### Observar no GNOME com Wayland

O GNOME não deixa programas de fora capturarem a tela; só o servidor VNC dele mesmo, o gnome-remote-desktop. O cliente o inicia apenas durante o Observar/Assistir e o encerra depois, devolvendo a configuração que o usuário tinha.

- Instale no cliente o pacote gnome-remote-desktop e o secret-tool (pacote libsecret). Eles não são dependência do pacote do cliente; sem eles, o epoptes avisa.
- A senha de cada Observar é gerada pelo servidor e guardada no chaveiro do GNOME. Se o chaveiro estiver trancado, aparece na tela do aluno um pedido para destrancá-lo, e o Observar espera até 60 segundos.
- O GNOME mostra ao aluno um botão laranja no canto superior direito enquanto a tela é vista; clicando no quadrado, o aluno encerra o Observar.
- Se o aluno já estiver com o compartilhamento de tela do GNOME ligado, o Observar não abre.
- O visualizador do servidor recebe a senha sozinho se for o TigerVNC; outros visualizadores pedem a senha.

## Diagnóstico no cliente

Na máquina do aluno, como usuário, dentro da sessão gráfica:

    epoptes-client --check

Ele confere: arquivos do pacote, serviço habilitado e rodando (runit ou systemd), se o nome do servidor resolve, o certificado local e se ele confere com o do servidor, as conexões de cada usuário (aponta duplicadas e diz qual comando encerra as sobrando) e a miniatura. Termina com "Tudo certo." ou com o número de problemas e, em cada um, o que fazer. Rodando como root, confere o lado do serviço.

## Problemas comuns

**O servidor não mostra nenhum cliente.** No cliente, rode epoptes-client --check. As causas mais comuns são o certificado diferente do servidor (rode sudo epoptes-client -c e reinicie o serviço do cliente e a sessão), o nome do servidor que não resolve (ajuste SERVER= em /etc/default/epoptes-client ou o /etc/hosts) e a porta 789 bloqueada no firewall do servidor.

**A máquina aparece como "std" e sem miniatura.** Só a conexão do root chegou. Veja se alguém está logado; se está, use Clientes > Diagnosticar cliente ou o --check na sessão do aluno.

**Sessão duplicada.** Use Clientes > Reconectar sessão na conexão mais nova (a dica diz qual). No cliente, o --check mostra o comando para encerrar as sobrando (pkill -KILL -P com o PID).

**O epoptes avisa que o usuário não está no grupo epoptes.** Saia da sessão e entre de novo; no GNOME, reinicie a máquina.

**Miniatura falhou.** O motivo aparece na dica. Em Hyprland e Sway é o grim; no GNOME com Wayland a miniatura não funciona (veja a tabela acima).

**Cliente com a rede lenta para subir (Tailscale, VPN).** O cliente espera o nome do servidor resolver e tenta de novo sozinho; não precisa reiniciar nada.

## Arquivos

| Arquivo | Para que serve |
|---|---|
| /etc/default/epoptes | porta e grupo do servidor |
| /etc/default/epoptes-client | servidor (SERVER=), porta e Wake On LAN do cliente |
| /etc/epoptes/server.crt | certificado do servidor (no servidor e copiado em cada cliente) |
| ~/.config/epoptes/ | grupos e configurações da interface, por usuário |
| /usr/share/epoptes/client-functions | funções que o servidor envia a cada cliente na conexão |

## Créditos

Epoptes: Alkis Georgopoulos, Fotis Tsamis e a equipe do Epoptes (epoptes.org), licença GPL-3.0 ou posterior.

Mudanças do VoidBR e do Chili Linux: Vilmar Catafesta <vcatafesta@gmail.com>.
