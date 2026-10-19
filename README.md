<div align="center">

# Laboratório de Redes e Segurança

**Redes · Linux · Fundamentos de Segurança**

Da configuração do ambiente à investigação de serviços: um registro prático de aprendizagem.

[![Status: em andamento](https://img.shields.io/badge/status-em_andamento-2563eb?style=flat-square)](#progresso)
[![Ambiente: VirtualBox](https://img.shields.io/badge/ambiente-VirtualBox-183A61?style=flat-square)](#ambiente)
[![Sistema: Kali Linux](https://img.shields.io/badge/sistema-Kali_Linux-557C94?style=flat-square)](#ambiente)
[![Tipo: laboratório pessoal](https://img.shields.io/badge/tipo-laboratório_pessoal-475569?style=flat-square)](#escopo)

[Visão geral](#visão-geral) · [Ambiente](#ambiente) · [Diário técnico](#diário-técnico) · [Progresso](#progresso) · [Aprendizados](#aprendizados)

</div>

---

## Visão geral

Este projeto documenta a construção de um laboratório pessoal para estudar redes, Linux e fundamentos de segurança.

O ponto de partida é uma máquina Kali Linux no VirtualBox. A evolução do projeto será registrada com comandos, resultados observados e explicações sobre as decisões tomadas.

### Objetivos

- Interpretar interfaces de rede, endereços IP e tabelas de rotas.
- Compreender a comunicação entre máquinas virtuais.
- Preparar uma rede separada para exercícios de segurança.
- Identificar serviços e analisar os resultados com contexto.
- Praticar documentação técnica e controle de versões com Git e GitHub.

> **Etapa atual:** identificação da configuração de rede do Kali. A máquina de laboratório para os próximos exercícios ainda não foi adicionada.

## Ambiente

| Componente | Configuração observada |
| :--- | :--- |
| Sistema hospedeiro | Windows |
| Memória RAM do computador | 16 GB |
| Plataforma de virtualização | VirtualBox |
| Máquina virtual | Kali Linux |
| Modo do adaptador de rede | NAT |
| Interface de rede no Kali | `eth0` |
| Endereço IPv4 do Kali | `10.0.2.15/24` |
| Rede diretamente conectada | `10.0.2.0/24` |
| Gateway padrão | `10.0.2.2` |

Os endereços representam a configuração observada durante esta etapa e podem mudar.

### Caminho da saída de rede

```text
Kali Linux
eth0 · 10.0.2.15/24
        │
        ▼
Gateway virtual
10.0.2.2 · NAT do VirtualBox
        │
        ▼
Conexão de rede do Windows
```

O esquema representa o caminho configurado para saída. A conectividade com a internet ainda não foi validada neste registro.

## Diário técnico

### 01 — Preparação da máquina virtual

A máquina Kali Linux já existia de um curso anterior. Para iniciar este projeto:

1. O adaptador de rede foi alterado de **bridge** para **NAT**.
2. A máquina virtual foi iniciada.
3. O acesso à área de trabalho foi confirmado.

**Motivo da escolha:** utilizar o NAT nesta etapa inicial para permitir conexões de saída pela rede do computador hospedeiro. Uma rede separada será preparada para os exercícios entre máquinas virtuais.

### 02 — Identificação das interfaces

**Objetivo:** verificar quais interfaces existem, seus estados e endereços.

```bash
ip -br addr
```

**Resumo dos dados IPv4 observados:**

| Interface | Estado exibido | Endereço IPv4 | Interpretação |
| :--- | :--- | :--- | :--- |
| `lo` | `UNKNOWN` | `127.0.0.1/8` | Interface de loopback, usada na comunicação do sistema consigo mesmo. |
| `eth0` | `UP` | `10.0.2.15/24` | Interface ativa que conecta o Kali à rede virtual. |

A saída também apresentou endereços IPv6, que não foram analisados nesta etapa.

**Conclusão:** a interface `eth0` está ativa e possui um endereço IPv4 privado. Isso, isoladamente, não comprova acesso à internet.

### 03 — Identificação das rotas

**Objetivo:** descobrir a rede diretamente conectada e o caminho padrão para outras redes.

```bash
ip route
```

<details>
<summary><strong>Ver saída registrada</strong></summary>

```text
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
```

</details>

#### Interpretação

| Trecho da saída | Significado |
| :--- | :--- |
| `default via 10.0.2.2` | Define o gateway usado quando não existe uma rota mais específica para o destino. |
| `dev eth0` | Indica a interface utilizada pela rota. |
| `src 10.0.2.15` | Indica o endereço de origem preferido nessa rota. |
| `10.0.2.0/24` | Identifica a rede diretamente conectada. |
| `proto dhcp` | Indica que a rota padrão foi configurada por DHCP. |

**Conclusão:** o Kali possui uma rota direta para `10.0.2.0/24` e uma rota padrão pelo gateway `10.0.2.2`.

### 04 — Configuração da rede interna

**Objetivo:** preparar uma interface dedicada à comunicação entre as máquinas do laboratório.

Com o Kali desligado, o Adaptador 2 do VirtualBox foi habilitado em modo **Rede Interna**, com o nome `lab-cyber`. O Adaptador 1 permaneceu em NAT.

No Kali, a segunda interface foi identificada como `eth1`.

#### Problema encontrado

O endereço configurado com `ip addr add` apareceu inicialmente, mas não permaneceu aplicado. Na verificação seguinte:

- A interface `eth1` estava sem IPv4.
- O NetworkManager indicava estado `disconnected`, sem conexão ativa.
- A rota da rede do laboratório não estava presente.

Esses resultados confirmaram a ausência da configuração, mas não determinaram a causa exata de sua remoção.

#### Configuração permanente

Foi criado um perfil no NetworkManager com endereço IPv4 manual e ativação automática:

```bash
sudo nmcli connection add type ethernet ifname eth1 con-name lab-cyber ipv4.method manual ipv4.addresses 192.168.77.10/24 ipv4.never-default yes connection.autoconnect yes
```

Em seguida, o perfil foi ativado:

```bash
sudo nmcli connection up lab-cyber
```

A opção `ipv4.never-default yes` impede que esse perfil seja utilizado como rota padrão IPv4.

#### Validação após reiniciar

Comando:

```bash
ip -4 -br addr
```

Resultado:

```text
lo               UNKNOWN        127.0.0.1/8
eth0             UP             10.0.2.15/24
eth1             UP             192.168.77.10/24
```

Comando:

```bash
ip -4 route
```

Resultado:

```text
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
192.168.77.0/24 dev eth1 proto kernel scope link src 192.168.77.10 metric 101
```

**Conclusão:** o endereço IPv4 e a rota da rede interna permaneceram após reiniciar. A rota padrão continuou pelo adaptador NAT. A comunicação com outra máquina do laboratório ainda será testada.

### 05 — Adição do servidor Ubuntu

Foi criada a máquina virtual `ubuntu-lab` com Ubuntu Server e dois adaptadores de rede:

| Interface | Modo no VirtualBox | IPv4 |
| :--- | :--- | :--- |
| `enp0s3` | NAT | `10.0.2.15/24` |
| `enp0s8` | Rede Interna — `lab-cyber` | `192.168.77.20/24` |

O serviço OpenSSH foi selecionado durante a instalação.

O endereço NAT coincide com o do Kali porque cada máquina utiliza uma instância separada do NAT padrão do VirtualBox. A comunicação entre as duas máquinas neste exercício utiliza a rede interna `192.168.77.0/24`.

### 06 — Teste de comunicação

No Kali:

```bash
ping -c 4 192.168.77.20
```

Foram observadas quatro respostas do Ubuntu, com tempos entre `0.251 ms` e `0.372 ms`.

**Conclusão:** houve comunicação entre Kali e Ubuntu pela rede interna. Esse teste não determina quais serviços estão disponíveis.

### 07 — Identificação do SSH

No Kali:

```bash
nmap -sV -p 22 192.168.77.20
```

Trecho do resultado:

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.8 (Ubuntu Linux; protocol 2.0)
```

Interpretação:

- `-p 22` limita a verificação à porta 22.
- `-sV` tenta identificar o serviço e sua versão.
- `open` indica que a porta está aceitando conexões.
- O serviço identificado foi SSH, compatível com a instalação do OpenSSH no Ubuntu.

**Limitação:** apenas a porta TCP 22 foi examinada. A identificação de versão não comprova uma vulnerabilidade.

### 08 — Acesso remoto e confirmação do ambiente

A conexão foi iniciada no Kali:

```bash
ssh netrunner@192.168.77.20
```

Após entrar, foram executados os seguintes comandos na sessão remota:

```text
netrunner@ubuntu-lab:~$ whoami
netrunner
netrunner@ubuntu-lab:~$ hostname
ubuntu-lab
```

**Conclusão:** a sessão utilizou o usuário `netrunner` no servidor `ubuntu-lab`. O Kali atuou como cliente SSH; os comandos da sessão foram executados no Ubuntu.

### 09 — Comparação entre porta aberta e fechada

No Kali, foram verificadas duas portas TCP do Ubuntu:

```bash
nmap -sV -p 22,80 192.168.77.20
```

Resultado principal:

```text
PORT   STATE  SERVICE VERSION
22/tcp open   ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.8 (Ubuntu Linux; protocol 2.0)
80/tcp closed http
```

#### Interpretação

- **22/TCP aberta:** o serviço SSH estava acessível. O acesso com o usuário do laboratório já havia sido validado na etapa anterior.
- **80/TCP fechada:** o alvo respondeu, mas não havia serviço aceitando conexões nessa porta.
- O nome `http` associa a porta 80 ao seu uso habitual; não comprova a presença de um servidor web.

### Conclusão da primeira versão

Foi montado um laboratório com Kali Linux e Ubuntu Server conectados pela rede interna `lab-cyber`.

Foram validados:

- Endereçamento e rotas do Kali, incluindo persistência após reiniciar.
- Comunicação do Kali com o Ubuntu.
- Identificação do serviço SSH.
- Acesso remoto autenticado e execução de comandos no Ubuntu.
- Comparação entre uma porta TCP aberta e uma fechada.

**Limitações:** a verificação de serviços ficou restrita às portas TCP 22 e 80. Não foram avaliadas vulnerabilidades, resistência das senhas ou configurações de segurança do SSH. Os resultados não permitem concluir que o servidor é seguro ou vulnerável.

## Progresso

### Preparação e reconhecimento do ambiente

- [x] Iniciar o Kali Linux no VirtualBox.
- [x] Alterar o adaptador de bridge para NAT.
- [x] Consultar interfaces e endereços.
- [x] Identificar a rede diretamente conectada.
- [x] Identificar o gateway padrão.
- [x] Registrar a configuração inicial no README.

### Próximas etapas

- [ ] Preparar uma rede separada para os exercícios.
- [ ] Adicionar uma máquina de laboratório.
- [ ] Validar a comunicação entre as máquinas.
- [ ] Identificar portas e serviços do alvo de laboratório.
- [ ] Documentar resultados, limitações e conclusões.

## Aprendizados

| Conceito | Entendimento trabalhado |
| :--- | :--- |
| Interface de rede | É o ponto de conexão do sistema com uma rede física ou virtual. |
| Loopback | Permite que o sistema se comunique consigo mesmo. |
| Endereço IP | Identifica uma interface dentro de uma rede. |
| Prefixo `/24` | Indica que os primeiros 24 bits do endereço IPv4 representam a parte da rede. |
| Gateway padrão | É o próximo salto utilizado para destinos sem uma rota mais específica. |
| Porta aberta | Indica um serviço aceitando comunicação; não confirma, sozinha, uma vulnerabilidade. |

## Escopo

Este repositório registra **atividades de aprendizagem em laboratório** e não representa experiência profissional em pentest.

Os exercícios serão realizados em máquinas próprias ou ambientes expressamente autorizados, utilizando dados fictícios ou públicos. A documentação publicada não deve conter credenciais ou informações confidenciais.

---

<div align="center">

Criado e mantido por [netrunner-k](https://github.com/netrunner-k)

**Observar → entender → praticar → documentar**

</div>
