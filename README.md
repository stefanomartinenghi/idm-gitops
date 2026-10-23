# idm-gitops

Repository dichiarativo per il deploy di `idm-app` tramite Argo CD e Kustomize.

## Ambienti

| Overlay | Namespace | Replica iniziali | Promozione prevista |
|---|---|---:|---|
| `local-test` | `idm-test` | 1 | automatica dopo un push su `main` di `idm-app` |
| `local-qual` | `idm-qual` | 2 | automatica dopo un tag `vX.Y.Z` di `idm-app` |
| `local-prod` | `idm-prod` | 3 | manuale e protetta, da definire |

Argo CD osserva sempre il branch `main`. Le pipeline applicative promuovono una versione
modificando esclusivamente `newTag` nell'overlay dell'ambiente destinazione.

## Validazione locale

```bash
kubectl kustomize apps/idm/overlays/local-test
kubectl kustomize apps/idm/overlays/local-qual
kubectl kustomize apps/idm/overlays/local-prod
```

## Database esterno Keycloak

Keycloak usa il database MySQL esterno `idmccno_keycloak26` raggiungibile su
`10.166.18.126:3306`. Il repository contiene solamente la referenza al database; le
credenziali non sono versionate.

La VPN/rete aziendale deve essere attiva e la porta `10.166.18.126:3306` deve essere
raggiungibile sia da WSL sia dai pod del cluster prima che Keycloak possa diventare Ready.

Prima di attivare un ambiente creare nel relativo namespace il Secret atteso dal
Keycloak Operator:

```bash
kubectl create secret generic keycloak-mysql-database \
  --namespace keycloak-test \
  --from-literal=username='<mysql-username>' \
  --from-literal=password='<mysql-password>'
```

Usare lo stesso nome di Secret nei namespace `keycloak-qual` e `keycloak-prod` quando
verranno attivati. Non collegare contemporaneamente ambienti indipendenti allo stesso
schema senza aver definito esplicitamente la strategia di condivisione e aggiornamento.

Il tag iniziale `bootstrap` è un segnaposto. Prima del primo deploy verrà sostituito dalla
pipeline con il tag immutabile derivato dal commit dell'applicazione.
