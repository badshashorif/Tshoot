graph TD

    subgraph CORE & NAT
        A[CORE / NAT Routers<br>Loopback: 172.30.0.0/24]
    end

    subgraph ACCESS
        B[ACCESS Routers<br>Loopback: 172.30.100.0/22]
    end

    subgraph POP
        C[POP Switch Block-1<br>172.30.30.0/22]
        D[POP Switch Block-2<br>172.30.50.0/22]
    end

    A --- B
    A --- C
    A --- D

    subgraph VLAN_P2P [Service VLAN & Peering P2P IPs]
        V1[INT<br>VLAN 3001–3100<br>172.25.0.0/16]
        V2[CDN<br>VLAN 3201–3300<br>172.26.0.0/16]
        V3[FNA<br>VLAN 3401–3500<br>172.27.0.0/16]
        V4[GGC<br>VLAN 3601–3700<br>172.28.0.0/16]
        V5[BDIX<br>VLAN 3801–3900<br>172.29.0.0/16]
    end

    B --- V1
    B --- V2
    B --- V3
    B --- V4
    B --- V5

    subgraph PPPoE [PPPoE Pools & Packages]
        E[EXPIRED_POOL<br>10.250.0.0/16]

        P1[500TK_REGULAR<br>10.11.0.0/16]
        P2[600TK_FAMILY<br>10.12.0.0/16]
        P3[700TK_PRIME<br>10.13.0.0/16]
        P4[800TK_FREEDOM<br>10.14.0.0/16]
        P5[1000TK_SILVER<br>10.15.0.0/16]
        P6[1200TK_GOLD<br>10.16.0.0/16]
        P7[1500TK_PLATINUM<br>10.17.0.0/16]
        P8[2000TK_DIAMOND<br>10.18.0.0/16]
    end

    B --- E
    B --- P1
    B --- P2
    B --- P3
    B --- P4
    B --- P5
    B --- P6
    B --- P7
    B --- P8
