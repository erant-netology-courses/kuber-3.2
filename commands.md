sudo tee /etc/haproxy/haproxy.cfg > /dev/null <<'EOF'
global
    log /dev/log local0
    maxconn 4096

defaults
    mode tcp
    timeout connect 5s
    timeout client 1m
    timeout server 1m

frontend k8s-api
    bind *:6443
    default_backend k8s-api-backend

backend k8s-api-backend
    balance roundrobin
    server master1 10.129.0.29:6443 check
    server master2 10.129.0.37:6443 check
    server master3 10.129.0.21:6443 check
EOF



sudo tee /etc/keepalived/keepalived.conf > /dev/null <<'EOF'
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1
    use_vmac false
    authentication { auth_type PASS
                     auth_pass 1234 }
    virtual_ipaddress { 10.129.0.100/24 }
    unicast_src_ip 10.129.0.29
    unicast_peer {
        10.129.0.37
        10.129.0.21
    }
}
EOF

sudo tee /etc/keepalived/keepalived.conf > /dev/null <<'EOF'
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    use_vmac false
    authentication { auth_type PASS
                     auth_pass 1234 }
    virtual_ipaddress { 10.129.0.100/24 }
    unicast_src_ip 10.129.0.37
    unicast_peer {
        10.129.0.29
        10.129.0.21
    }
}
EOF

sudo tee /etc/keepalived/keepalived.conf > /dev/null <<'EOF'
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 99
    advert_int 1
    use_vmac false
    authentication { auth_type PASS
                     auth_pass 1234 }
    virtual_ipaddress { 10.129.0.100/24 }
    unicast_src_ip 10.129.0.21
    unicast_peer {
        10.129.0.29
        10.129.0.37
    }
}
EOF
