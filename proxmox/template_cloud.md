# How to install template cloud proxmox

### Link reference
- https://pve.proxmox.com/wiki/Cloud-Init_Support


## Using scripts

- Edit variables

| Name | Type | Description |
| --- | --- | --- |
| `RESIZE` | +int | Disk size for template (ex: `+30G`) |
| `MEMORY` | init | Memory size for tempalte (ex: `2048`) |
| `BRIDGE` | string | Network bridge name |
| `SSHKEY` | string | SSH key default for template |
| `STORAGE` | string | define your storage here local/local-lvm or any other storage you created in Proxmox using its name |
| `FORMAT` | string | the disk format of the disk, in case of local=qcow2 in case of local-lvm=raw |

## Run on node proxmox

```bash 
bash create-cloud-tempalte.sh 

```