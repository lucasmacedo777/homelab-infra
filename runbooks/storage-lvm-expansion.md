# Procedimento Operacional: Gerenciamento de Storage e LVM

Este documento detalha as intervenções realizadas para contornar problemas de alocação de blocos físicos no Proxmox e o redimensionamento de volumes no Ubuntu Server.

## 1. Expansão Online de Volume Lógico (Hot Resize)

Quando o vDisk do Proxmox possui espaço não alocado (unallocated space) na partição base do LVM, o redimensionamento deve ser feito em três camadas contíguas: Bloco, Volume Físico e Volume Lógico.

```bash
# 1. Mapeia a topologia e identifica partições menores que o disco físico (vDisk)
lsblk

# 2. Expande a partição física L3 (Ex: sda3) para preencher o limite estrutural do disco
sudo growpart /dev/sda 3

# 3. Notifica o kernel (gerenciador LVM) sobre a nova geometria do Physical Volume (PV)
sudo pvresize /dev/sda3

# 4. Injeta o espaço no Logical Volume (LV) e expande o Filesystem (ext4) simultaneamente sem downtime (-r)
sudo lvextend -l +100%FREE -r /dev/mapper/ubuntu--vg-ubuntu--lv
```

## 2. Hard Quotas e Isolamento de IO (Montagem Persistente)

Para isolar microsserviços com alto consumo de I/O (como bancos de dados e indexadores de mídia) do disco primário do SO, evitando falhas em cascata por falta de espaço (Kernel Panic):

```bash
# 1. Formatação de um novo vDisk atrelado a quente (hotplug no hypervisor) com File System ext4
sudo mkfs.ext4 /dev/sdb

# 2. Coleta do Identificador Universal Único (UUID) para resiliência contra reordenação de portas SATA/SCSI
sudo blkid /dev/sdb

# 3. Injeção da regra no fstab para montagem automática no boot
echo 'UUID=<insira_o_uuid_aqui> /mnt/app_data ext4 defaults 0 0' | sudo tee -a /etc/fstab

# 4. Validação de sintaxe do fstab sem necessidade de reboot (Mitigação de risco crítico)
sudo mount -a
```
