# High Availability Switch Configuration with Trunking and DHCP



## VLAN Design

| VLAN ID | VLAN Name | Purpose | Port Assignment |
|---------|-----------|---------|-----------------|
| 5 | proxmox_cluster_and_vms | Proxmox VM network | fa0/1 - fa0/12 |
| 6 | proxmox_cororsync_network | Corosync network | fa0/13 - fa0/16 |
| 10 | idrac | iDRAC management | fa0/17 - fa0/20 |
| 80 | proxmox_cluster | Proxmox cluster | fa0/21 - fa0/23 |

## Switch 1 Configuration

### Initial Setup and Hostname
```cisco
enable
configure terminal
hostname Switch1
```

### VLAN Creation
```cisco
vlan 5
name proxmox_cluster_and_vms
exit
vlan 6
name proxmox_cororsync_network
exit
vlan 10
name idrac
exit
vlan 80
name proxmox_cluster
exit
```

### Port to VLAN Assignment
```cisco
interface range fa0/1 - 12
switchport mode access
switchport access vlan 5
exit

interface range fa0/13 - 16
switchport mode access
switchport access vlan 6
exit

interface range fa0/17 - 20
switchport mode access
switchport access vlan 10
exit

interface range fa0/21 - 23
switchport mode access
switchport access vlan 80
exit
```

### Trunk Configuration
```cisco
interface fa0/24
switchport mode trunk
switchport trunk allowed vlan 5,6,10,80
exit
```

### VLAN Interface Configuration
```cisco
interface vlan 5
ip address 10.10.5.2 255.255.255.0
no shutdown
exit

interface vlan 6
ip address 10.10.6.1 255.255.255.0
no shutdown
exit

interface vlan 10
ip address 192.168.0.1 255.255.255.0
no shutdown
exit

interface vlan 80
ip address 10.10.80.11 255.255.255.0
no shutdown
exit
```

### Default Gateway Configuration
```cisco
ip default-gateway 10.10.5.222
```

### DHCP Relay Configuration
```cisco
interface vlan 5
ip helper-address 10.10.5.254
exit

interface vlan 6
ip helper-address 10.10.5.254
exit

interface vlan 10
ip helper-address 10.10.5.254
exit

interface vlan 80
ip helper-address 10.10.5.254
exit
```

### Save Configuration
```cisco
end
write memory
```

## Switch 2 Configuration

### Initial Setup and Hostname
```cisco
enable
configure terminal
hostname Switch2
```

### VLAN Creation
```cisco
vlan 5
name proxmox_cluster_and_vms
exit
vlan 6
name proxmox_cororsync_network
exit
vlan 10
name idrac
exit
vlan 80
name proxmox_cluster
exit
```

### Port to VLAN Assignment
```cisco
interface range fa0/1 - 12
switchport mode access
switchport access vlan 5
exit

interface range fa0/13 - 16
switchport mode access
switchport access vlan 6
exit

interface range fa0/17 - 20
switchport mode access
switchport access vlan 10
exit

interface range fa0/21 - 23
switchport mode access
switchport access vlan 80
exit
```

### Trunk Configuration
```cisco
interface fa0/24
switchport mode trunk
switchport trunk allowed vlan 5,6,10,80
exit
```

### VLAN Interface Configuration
```cisco
interface vlan 5
ip address 10.10.5.3 255.255.255.0
no shutdown
exit

interface vlan 6
ip address 10.10.6.2 255.255.255.0
no shutdown
exit

interface vlan 10
ip address 192.168.0.2 255.255.255.0
no shutdown
exit

interface vlan 80
ip address 10.10.80.12 255.255.255.0
no shutdown
exit
```

### Default Gateway Configuration
```cisco
ip default-gateway 10.10.5.222
```

### DHCP Relay Configuration
```cisco
interface vlan 5
ip helper-address 10.10.5.254
exit

interface vlan 6
ip helper-address 10.10.5.254
exit

interface vlan 10
ip helper-address 10.10.5.254
exit

interface vlan 80
ip helper-address 10.10.5.254
exit
```

### Save Configuration
```cisco
end
write memory
```

## Verification Commands

After completing the configuration, use these commands to verify proper setup:

```cisco
show vlan brief                 # Verifies VLAN creation and port assignments
show interfaces trunk           # Verifies trunk configurations
show ip interface brief         # Checks the status and IP addresses of interfaces
show running-config             # Displays the entire current configuration
show ip route                   # Verifies routing information
```

## Troubleshooting

### Common Issues and Solutions

1. **VLAN interface not coming up**:
   - Ensure that there's at least one active port in the VLAN
   - Verify the "no shutdown" command was applied
   
2. **Trunk not passing traffic for certain VLANs**:
   - Check "show interfaces trunk" to verify allowed VLANs
   - Ensure VLANs are created on both switches
   
3. **DHCP not working across VLANs**:
   - Verify ip helper-address is configured correctly
   - Check connectivity to the DHCP server
   - Ensure the DHCP server has scopes configured for each VLAN subnet

4. **Inter-VLAN routing issues**:
   - Verify default gateway is set correctly
   - Check that the router has routes to all VLANs
   - Ensure SVI interfaces are up
