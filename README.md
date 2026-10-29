# Suivi des contrats – appareils MAX (FR)

## Pourquoi

Des ascenseurs perdent leur contrat mais gardent leur boîtier MAX provisionné : la SIM reste payée et le matériel n'est pas récupéré. Jusqu'ici, ces cas se repéraient à la main dans View (suffixe `.Lost`), ce qui n'est pas fiable.

Le notebook `MAX_Suivi_contrats.ipynb` sort automatiquement la liste des appareils FR équipés d'un MAX qui n'ont plus de contrat actif, à traiter avec le process de déprovisionnement (MAX + SIM).

## Comment

Notebook Databricks en lecture seule (aucune table écrite) :

1. **Appareils MAX** : `md.v_elevator`, FR, `ProvisioningStatus` actif.
2. **Contrats** : `md.v_contract`, dernière version de chaque contrat. Un appareil est sous contrat si au moins une ligne est `ACTIVE` ou `NOT STARTED`.
3. **Classement** :
   - **À déprovisionner** : aucun contrat actif ;
   - **MAX à retirer** : contrat actif avec résiliation déjà saisie ;
   - **OK** : rien à faire.
4. **Listes** par agence, avec MAC et ICCID pour le déprovisionnement.

Les choix (statut plutôt que dates, une ligne par option de contrat, agence = `BranchNumber`…) sont expliqués dans la section "Constats sur les données" du notebook.

Planification possible chaque matin ou une fois par semaine via un job Databricks. L'onglet *Runs* du job sert d'historique.

## Résultat (28/10/2026)

| | Appareils |
|---|---|
| Appareils avec MAX | 8 558 |
| À déprovisionner | 771 (dont 642 sans aucun contrat) |
| MAX à retirer | 0 (les résiliations ne sont pas saisies à l'avance) |
| OK | 7 787 |

## À vérifier avant d'agir

Le notebook propose une liste, il ne décide pas. **Vérifier chaque appareil dans View avant de le déprovisionner**, surtout ceux marqués "Aucun contrat trouvé" (installation récente, contrat sous un autre UnitId..). Déprovisionner à tort coupe le service d'un client sous contrat.

**Risque connu** : les "Aucun contrat trouvé" font 642 des 771 appareils à déprovisionner. Une partie peut être des installations récentes (MAX provisionné avant la saisie du contrat). À contrôler avec `v_elevator.ProvisioningStatusChangeDate` ; s'il y en a beaucoup, les signaler dans la liste.

Après toute modification du notebook, relancer et contrôler : 1 ligne par appareil (section 6) et des chiffres cohérents avec le tableau ci-dessus.
