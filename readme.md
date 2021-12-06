# modop

1. Créer un cluster `minikube start`

2. bootstrap flux

3. Création du secret sops

```
gpg --list-secret-keys ${your_email_address}
gpg --export-secret-keys --armor xxxxx | kubectl create secret generic sops-gpg  --namespace=flux-system --from-file=sops.asc=/dev/stdin
