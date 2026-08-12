# Homelab Infrastructure & IaC

Repositório dedicado à documentação de arquitetura, provisionamento e troubleshooting do meu ambiente de laboratório (Homelab).

## Topologia Atual
- **Hypervisor:** Proxmox VE
- **Rede & DNS:** Pi-hole (DNS Sinkhole com roteamento estático)
- **Orquestração:** Docker + Portainer (LXC/VM no Ubuntu Server)
- **Aplicações (Workloads):** Immich (Self-hosted Media Server)

## Destaques de Engenharia e Boas Práticas
- **Storage Management:** Expansão de partições LVM online (*hot resize*) e manipulação avançada de blocos (Physical/Logical Volumes).
- **Isolamento de Recursos (Hard Quotas):** Provisionamento de discos virtuais secundários, mapeados persistentemente via UUID no `/etc/fstab`, blindando o SO contra esgotamento de I/O.
- **Resiliência L3:** Configuração de IP estático e nameservers diretamente no *kernel* (Netplan/YAML) para garantir alta disponibilidade de rede.
- **Troubleshooting de Contêineres:** Análise de logs, refatoração de dependências em *crash loop* (transição de extensões `pgvecto-rs` para `pgvector`) e correção de mapeamento de portas.
