# Lab topology

Лабораторията съдържа две виртуални машини:

- `Gateway-NIS` – gateway
- `Server-NIS` – server

Мрежата е разделена на три сегмента:

- `WAN` – NAT Network с достъп до интернет
- `LAN1` – VirtualBox Host-Only Ethernet Adapter
- `LAN2` – VirtualBox Host-Only Ethernet Adapter #2

`Gateway-NIS` е свързан към WAN, LAN1 и LAN2.

`Server-NIS` е свързан към LAN1.

## Topology

Internet
   |
  WAN
   |
Gateway-NIS
   /     \
 LAN1    LAN2
   |
Server-NIS

IP forwarding на `Gateway-NIS` е включен.

`Server-NIS` има свързаност до `Gateway-NIS`, а `Gateway-NIS` има достъп до интернет.