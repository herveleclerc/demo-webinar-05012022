# modop

1. Créer un cluster `minikube start`

2. bootstrap flux

```
source .github/token

flux bootstrap github \
  --owner=herveleclerc \
  --repository=demo-webinar-05012022 \
  --path=clusters/local \
  --personal
```

3. Création du secret sops

```

# Utilisation d'une clé gpg au préalablement créée

gpg --batch --full-generate-key << EOF
Key-Type: 1
Key-Length: 4096
Subkey-Type:1
Subkey-Length: 4096
Name-Real: demo-webinar-05012022
Name-Email: herve.leclerc@alterway.fr
Expire-Date: 0
%no-protection
EOF

gpg --list-secret-keys ${your_email_address}

gpg --export-secret-keys --armor D8ABD101519825A1F15D40A61BEBE158DC3B255A | kubectl create secret generic sops-gpg  --namespace=flux-system --from-file=sops.asc=/dev/stdin


# Création du secrets  sops pour les crédentials Azure
sops --encrypt \
    --pgp=D8ABD101519825A1F15D40A61BEBE158DC3B255A \
    --encrypted-regex '^(data|stringData)$' \
    01-credentials.yaml > 01-encrypted-credentials.yaml

# 1-encrypted-credentials.yaml sera utilisé plus tard
```

4. Editer le fichier clusters/local/gotk-sync.yaml -  Ajouter les déclarations pour utiliser sops

```yaml
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: flux-system
  namespace: flux-system
spec:
  # Ajouter ca 
  decryption:
    provider: sops
    secretRef:
      name: sops-gpg
  # fin ajouter ca ;)
  interval: 10m0s
  path: ./clusters/local
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
  validation: client

```


5. Créer une kuztomization FLUX - infra

   - pour gérer l'infrastructure des clusters 
     - sources des repo helm (crossplane)
   
6. Créer une kuztomization FLUX - apps

    - pour gérer les "applications" a déployer 
      - socle de base
      - et par cluster
   
7. Créer les spécificités de chaque cluster 