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
