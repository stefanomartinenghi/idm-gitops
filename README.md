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

Il tag iniziale `bootstrap` è un segnaposto. Prima del primo deploy verrà sostituito dalla
pipeline con il tag immutabile derivato dal commit dell'applicazione.
