# Procedimento Operacional: Configuração de IP Estático e Resiliência de DNS (Netplan)

Este documento detalha a fixação de endereçamento IP no Ubuntu Server utilizando o Netplan como gerenciador declarativo.

## Contexto e Risco Arquitetural

Serviços críticos de rede, como servidores DNS (Sinkholes como o Pi-hole), não podem operar sob escopo DHCP dinâmico. A alteração não planejada do IP do servidor invalida o apontamento de DNS distribuído pelo roteador principal, resultando em queda generalizada de resolução de nomes na infraestrutura local.

## 1. Mapeamento da Interface de Rede

Antes de aplicar a configuração declarativa, é mandatório identificar a nomenclatura lógica da interface de rede atribuída pelo kernel (ex: `eth0`, `enp3s0`).

```bash
# Lista as interfaces de rede ativas e seus respectivos estados (Link UP/DOWN)
ip a
```

## 2. Declaração da Configuração (YAML)

O Netplan gerencia o *backend* de rede (`systemd-networkd` ou `NetworkManager`) através de arquivos estruturados em YAML.

```bash
# Edição do arquivo de provisionamento do Netplan
sudo nano /etc/netplan/00-installer-config.yaml
```

**Topologia Lógica Aplicada:**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.6/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

*(Nota Operacional: A validação de indentação com espaços é crítica. O Netplan falhará se detectar o uso da tecla `TAB`).*

## 3. Aplicação e Validação de Estado

Após salvar o arquivo, o estado declarado deve ser compilado e injetado no sistema operacional.

```bash
# Valida a sintaxe estrutural e aplica a nova configuração de rede instantaneamente
sudo netplan apply

# Atesta o atrelamento do IP estático à interface
ip a

# Valida o roteamento do gateway e a resolução de DNS externo
ping -c 4 1.1.1.1
ping -c 4 github.com
```
