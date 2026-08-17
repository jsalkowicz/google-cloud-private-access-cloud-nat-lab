# Lab Notes — Private Google Access and Cloud NAT

## Goal

Create a VM with no external IP address and test how IAP, Private Google Access, and Cloud NAT provide different kinds of access.

## Environment

### Network

- VPC: `privatenet`
- Subnet: `privatenet-us`
- Subnet CIDR: `10.130.0.0/20`
- SSH firewall rule: `privatenet-allow-ssh`
- SSH source range: Google IAP
- SSH port: TCP 22

### VM

- VM: `vm-internal`
- Machine family: E2
- OS: Debian GNU/Linux 12
- External IP: None

## IAP Test

I connected to the private VM using:

```bash
gcloud compute ssh vm-internal --zone <LAB_ZONE> --tunnel-through-iap
```

The shell prompt changed to `vm-internal`, confirming that the IAP tunnel was working.

I then tested external connectivity:

```bash
ping -c 2 www.google.com
```

The test returned 100% packet loss before Private Google Access and Cloud NAT were configured.

## Private Google Access

I created a Cloud Storage bucket and copied the lab test file into it.

From `vm-internal`, I attempted to copy the object while Private Google Access was still disabled. The request did not complete.

I then enabled **Private Google Access** on the `privatenet-us` subnet and repeated the Cloud Storage copy.

The copy completed successfully without adding an external IP address to the VM.

## Cloud NAT

I created:

- Cloud NAT gateway: `nat-config`
- VPC: `privatenet`
- Cloud Router: `nat-router`

After the NAT gateway reached `Running`, I tested outbound package access from `vm-internal`:

```bash
sudo apt-get update
```

The update reached Debian and Google package repositories and completed successfully.

## Cloud NAT Logging

I enabled Cloud NAT logging for translations and errors.

After generating outbound traffic from the VM, I filtered Logs Explorer using:

```text
resource.type="nat_gateway"
```

The results included NAT flow entries with `allocation_status: "OK"` and fields for the connection, destination, endpoint, gateway identifiers, and VPC.

## Takeaway

IAP, Private Google Access, and Cloud NAT solve different problems.

IAP provides administrative access to the VM. Private Google Access allows a private-only VM to reach supported Google APIs and services. Cloud NAT provides outbound internet access without assigning a public IP directly to the VM.

The logging step gave me evidence that the NAT path was actually being used.
