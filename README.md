# 🌐 Réseau NGN : backbone IP/MPLS, LAN redondant et supervision

> Maquette GNS3 d'un réseau NGN : cœur IP/MPLS avec VPN (VRF, MP-BGP),
> LAN étendu segmenté en VLAN, et services d'administration (AAA, NTP, SNMP,
> Syslog, Zabbix, IP SLA).

## Contexte
Projet de Sécurité des Réseaux, TEK-UP, année 2025-2026, réalisé en binôme avec
Rached Jaafar. Encadrant académique : Nour Barrani.
Rapport complet : [rapport_projet_securite_reseau.pdf](rapport_projet_securite_reseau.pdf)

## Architecture

![Topologie du backbone MPLS](topologie-mpls.png)

- 2 routeurs de bord **PE** (PE1, PE2) et 2 routeurs de cœur **P** (P1, P2)
- 4 routeurs clients **CE** (CE11, CE12, CE21, CE22), répartis en deux VPN
- Environnement : GNS3, images Cisco IOS

![Maquette finale](topologie-finale.png)

## Partie 1 : cœur IP/MPLS et VPN
| Brique | Rôle |
|---|---|
| OSPF | Connectivité interne du backbone (loopbacks des PE et P) |
| LDP | Distribution des labels, construction des LSP |
| VRF (RD, RT) | Isolation des clients, adresses IP réutilisables sans conflit |
| MP-BGP (VPNv4) | Échange des routes clients entre PE1 et PE2 |

**Vérifications montrées dans le rapport :**

| Test | Commande | Capture |
|---|---|---|
| Voisin LDP établi | `show mpls ldp neighbor` | ![LDP](ldp.png) |
| Routes VPNv4 (RD 100:3) | `show bgp vpnv4 unicast all` | ![BGP](bgp-vpnv4.png) |
| Routes de la VRF sur PE1 et PE2 | `show ip route vrf VPN-SSIR` | ![VRF](vrf-pe1.png) |

Sur PE1, les routes du site distant apparaissent bien en `B` (BGP, via 2.2.2.2),
et les routes locales en `O` (OSPF avec le CE).

## Partie 2 : LAN étendu
- VLAN de management (101, 103) séparés du VLAN de données (102)
- Trunk 802.1Q, routage inter-VLAN en router-on-a-stick sur les CE
- DHCP sur le routeur CE pour le VLAN DATA (adresse reçue par PC1 vérifiée)
- Sécurisation des accès : mots de passe console, VTY et enable secret
- Redondance : OSPF avec coûts différenciés sur les liens

## Partie 3 : administration et supervision
| Service | Usage |
|---|---|
| AAA (RADIUS) | Authentification centralisée des administrateurs |
| NTP | Synchronisation horaire de l'infrastructure |
| SNMP + Zabbix | Supervision des équipements (tableau de bord Zabbix opérationnel) |
| Syslog | Journalisation centralisée |
| IP SLA (udp-jitter) | Mesure de gigue, latence et perte entre agences et siège |

## Limites
- Les tests de redondance (coupure de lien, bascule) sont décrits mais les résultats
  ne sont pas documentés
- Les mesures IP SLA (gigue, RTT) ne sont pas encore capturées
- Pas de test de ping de bout en bout entre CE11 et CE12 montré dans le rapport
- Le choix HSRP/VRRP et STP/EtherChannel n'est pas précisé
- Pas de chiffrement des flux : le VPN-MPLS isole le trafic mais ne le chiffre pas
- SNMP en v2c : v3 serait préférable pour l'authentification et le chiffrement

## Perspectives
Tests de bascule documentés, SNMPv3, filtrage des routes par route-target,
ajout d'IPsec par-dessus le VPN-MPLS, qualité de service (QoS) pour la voix.

## Technologies
GNS3, Cisco IOS, OSPF, LDP, MPLS, MP-BGP, VRF, 802.1Q, DHCP, RADIUS, NTP,
SNMP, Syslog, Zabbix, IP SLA.

## Auteurs
- Hamza Jbeli
- Rached Jaafar
