# Chapter 31: Cloud Integration (AWS, Azure, GCP)

## Introduction

Cloud CLI automation with Bash enables infrastructure management across AWS, Azure, and GCP. This chapter covers practical cloud automation scripts.

---

## AWS CLI Automation

### EC2 Instance Management

```bash
#!/bin/bash
# aws-ec2-manager.sh

INSTANCE_TYPE="t2.micro"
AMI_ID="ami-12345678"
KEY_NAME="my-key"
SECURITY_GROUP="sg-12345678"

# Launch instance
launch_instance() {
    instance_id=$(aws ec2 run-instances \
        --image-id "$AMI_ID" \
        --instance-type "$INSTANCE_TYPE" \
        --key-name "$KEY_NAME" \
        --security-group-ids "$SECURITY_GROUP" \
        --query 'Instances[0].InstanceId' \
        --output text)
    
    echo "Launched instance: $instance_id"
    
    # Wait for running
    aws ec2 wait instance-running --instance-ids "$instance_id"
    
    # Get public IP
    public_ip=$(aws ec2 describe-instances \
        --instance-ids "$instance_id" \
        --query 'Reservations[0].Instances[0].PublicIpAddress' \
        --output text)
    
    echo "Instance IP: $public_ip"
}

# Stop instance
stop_instance() {
    aws ec2 stop-instances --instance-ids "$1"
}

# Terminate instance
terminate_instance() {
    aws ec2 terminate-instances --instance-ids "$1"
}
```

### S3 Operations

```bash
#!/bin/bash
# aws-s3-backup.sh

S3_BUCKET="my-backup-bucket"
LOCAL_DIR="/data"
DATE=$(date +%Y%m%d)

# Sync to S3
aws s3 sync "$LOCAL_DIR" "s3://$S3_BUCKET/$DATE/" \
    --exclude "*.tmp" \
    --delete

# Set lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
    --bucket "$S3_BUCKET" \
    --lifecycle-configuration file://lifecycle.json

echo "S3 backup completed"
```

---

## Azure CLI Automation

### VM Management

```bash
#!/bin/bash
# azure-vm-manager.sh

RESOURCE_GROUP="my-rg"
VM_NAME="my-vm"
LOCATION="eastus"
IMAGE="UbuntuLTS"

# Create VM
create_vm() {
    az vm create \
        --resource-group "$RESOURCE_GROUP" \
        --name "$VM_NAME" \
        --image "$IMAGE" \
        --admin-username azureuser \
        --generate-ssh-keys \
        --location "$LOCATION"
}

# Start VM
start_vm() {
    az vm start --resource-group "$RESOURCE_GROUP" --name "$VM_NAME"
}

# Stop VM
stop_vm() {
    az vm deallocate --resource-group "$RESOURCE_GROUP" --name "$VM_NAME"
}

# Delete VM
delete_vm() {
    az vm delete --resource-group "$RESOURCE_GROUP" --name "$VM_NAME" --yes
}
```

---

## GCP CLI Automation

### Compute Engine Operations

```bash
#!/bin/bash
# gcp-instance-manager.sh

PROJECT_ID="my-project"
ZONE="us-central1-a"
INSTANCE_NAME="my-instance"
MACHINE_TYPE="n1-standard-1"

# Create instance
create_instance() {
    gcloud compute instances create "$INSTANCE_NAME" \
        --project="$PROJECT_ID" \
        --zone="$ZONE" \
        --machine-type="$MACHINE_TYPE" \
        --image-family=debian-10 \
        --image-project=debian-cloud
}

# Stop instance
stop_instance() {
    gcloud compute instances stop "$INSTANCE_NAME" \
        --project="$PROJECT_ID" \
        --zone="$ZONE"
}

# Delete instance
delete_instance() {
    gcloud compute instances delete "$INSTANCE_NAME" \
        --project="$PROJECT_ID" \
        --zone="$ZONE" \
        --quiet
}
```

---

## Multi-Cloud Cost Monitoring

### Cloud Cost Reporter

```bash
#!/bin/bash
# cloud-cost-report.sh

echo "=== Cloud Cost Report ==="
echo "Date: $(date)"
echo

# AWS costs
echo "AWS Costs (Current Month):"
aws ce get-cost-and-usage \
    --time-period Start=$(date -d "$(date +%Y-%m-01)" +%Y-%m-%d),End=$(date +%Y-%m-%d) \
    --granularity MONTHLY \
    --metrics "UnblendedCost" \
    --query 'ResultsByTime[0].Total.UnblendedCost' \
    --output text

# Azure costs
echo -e "\nAzure Costs (Current Month):"
az consumption usage list \
    --start-date $(date -d "$(date +%Y-%m-01)" +%Y-%m-%d) \
    --end-date $(date +%Y-%m-%d) \
    --query "[].{cost:pretaxCost}" \
    --output table

# GCP costs
echo -e "\nGCP Costs (Current Month):"
gcloud billing accounts get-billing-info \
    --billing-account="$BILLING_ACCOUNT_ID"
```

---

## Best Practices

1. **Use IAM roles** instead of access keys
2. **Implement cost controls**
3. **Tag resources** properly
4. **Monitor usage** regularly
5. **Automate cleanup**
6. **Use infrastructure as code**

---

## Summary

- Automate AWS EC2, S3 operations
- Manage Azure VMs and resources
- Control GCP Compute Engine
- Monitor multi-cloud costs
- Implement cloud best practices

---

**Next Chapter:** [32 - CI/CD Environments](32-cicd-environments.md)
