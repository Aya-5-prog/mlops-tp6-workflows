# Orchestrateur ou integration continue ? (TP06)

| Critere                           | Prefect (orchestrateur)                                                        | GitHub Actions (CI)                                                           |
|-----------------------------------|--------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| Ce qui declenche l'execution      | Une planification (cron, lundi 6 h), un Quick run depuis l'interface, ou un appel API | Un evenement Git : chaque git push, ou le bouton "Run workflow" (workflow_dispatch) |
| Ou s'execute le traitement        | Sur notre machine, par un processus qui reste ouvert : serve (terminal 3) ou worker d'un work pool | Sur une machine Linux jetable (ubuntu-latest) fournie par GitHub              |
| Qui surveille, et ou              | Prefect surveille chaque run et chaque tache, dans le tableau de bord (127.0.0.1:4200 : etats, durees, journaux) | GitHub surveille chaque run, dans l'onglet Actions du depot (pastille verte ou rouge sur le commit) |
| Ce qui se passe en cas d'echec    | La tache valider leve une erreur, le flow passe en Failed, notifier ne s'execute pas ; reessais possibles (2 sur dvc repro) | L'etape "Verifier la regle de qualite" sort avec le code 1, le job passe au rouge, le commit est marque en rouge ; pas de reessai automatique |
| Type de tache adapte              | Traitements recurrents ou a la demande : reentrainement, validation, notification | Verification du code a chaque modification : tests, controle de la regle de qualite |
| Ce qui est versionne dans Git     | flow_mlops.py, planifier.py, prefect.yaml (le deploiement); mais pas les executions ni l'etat du serveur | .github/workflows/ci.yml, verifier_qualite.py, requirements-ci.txt (toute la chaine est dans Git) |

## Conclusion (5 lignes)

La chaine d'integration continue valide le code a chaque modification : elle repond a la question "ce changement casse-t-il le pipeline ?".
L'orchestrateur execute et surveille des traitements recurrents : il repond a la question "le reentrainement de lundi s'est-il bien passe ?".
Dans les deux cas, la meme regle de qualite (F1 >= seuil_f1 de params.yaml) arrete la chaine quand le modele est degrade.
Avec max_depth: 2, Prefect a affiche un run Failed sur la tache valider et GitHub Actions une croix rouge sur l'etape de verification.
Les deux sont complementaires : aucun ne remplace l'autre.