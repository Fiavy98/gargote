## 0.2 Lire le schéma relationnel
1. Écrivez le schéma relationnel des 4 tables, en soulignant les clés primaires et en marquant les clés étrangères d’un `#` :

categorie(**id**,libelle)
plat(**id**,nom,prix,#id_categorie)
commande(**id**,date_commande,numero_table)
commande_ligne(**id**,#id_commande,#id_plat,quantite,prix_unitaire)

2. Pourquoi `commande_ligne` contient-elle une colonne `prix_unitaire`, alors que le prix est déjà dans `plat` ? Donnez un exemple concret où les deux valeurs seraient différentes.
- le commande_ligne contient un colonne prix_unitaire , car le plat.prix peuvent changer et 
le commande_ligne recupere le prix du plat actuelle , donc le prix pour l'ancien 
commande et deja stocker et il n'est pas ecraser 
ex :
- Plat 
    - Plat 1 
    - prix : 10000

- commande 20 oct 2026 
        - ligne commande 
            - plat 1 = 10000 (geter le plat actuelle dans le table)
    
- changement du plat 
    - 21 oct 2026 
    - plat 1 = 15000 


3. Que se passe-t-il pour les lignes d’une commande quand on supprime la commande ? Quelle instruction en est responsable ?
- le ligne de commande est automatiquement supprimer si le commande est null 
- instruction : ON DELETE CASCADE

4. Repérez **trois données absurdes** que le schéma actuel accepte sans broncher (indice : essayez d’insérer à la main un prix négatif). Notez-les : vous les corrigerez au niveau 1 et 2.
- ligne_commande.prix accepte le valeur negatif
- le plat.prix peuvent etre negatif
- commande_ligne.quantite peuvent etre negatif

## 0.3 Interroger l’API à la main
 **la réponse HTTP** de chaque requête :

```bash
# a) liste des soupes
curl -i http://localhost:8080/api/plats?categorie=2
- 200

# b) création d'un plat valide
curl -i -X POST http://localhost:8080/api/plats \
     -H "Content-Type: application/json" \
     -d '{"nom":"Mofo gasy","prix":300,"id_categorie":1}'
- 201

# c) création d'un plat invalide
curl -i -X POST http://localhost:8080/api/plats \
     -H "Content-Type: application/json" \
     -d '{"nom":"","prix":"cher"}'
- 422

# d) suppression d'un plat présent dans une commande
curl -i -X DELETE http://localhost:8080/api/plats/2

# e) une commande qui n'existe pas
curl http://localhost:8080/api/commandes/999
```

5. Pour la requête **b**, quel code obtenez-vous ? Pourquoi pas 200 ?
- on obtient 201 car le requete fait porte un insertion via le meth POSTE 
- le reponse du methode POSTE reusit c est 201

6. Pour la requête **d**, qui bloque la suppression : PHP ou PostgreSQL ? Montrez la ligne de code qui transforme ce blocage en réponse 409.
- c est postgreSQL qui bloque la suppression via le cle etrangere du id_Plat dans commande
- ligne qui transforme le blocage en code : 
```php
    return $this->erreur('Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.'409);

```
7. Créez une commande avec `curl` (table 6, deux *Romazava* et un *Café*). Écrivez la commande `curl` complète.

```bash
    curl -i -X POST http://localhost:8080/api/commandes/ \
    -H "Content-Type: application/json" \
     -d '{"numero_table":6,"lignes":[{"id_plat":4,"quantite":2},{"id_plat":10,"quantite":1}]}'
```

## 0.4 Suivre une requête de bout en bout
Quand on clique sur **Ajouter le plat**, voici le chemin parcouru. Complétez le tableau en indiquant, pour chaque étape, **le fichier et le nom de la fonction** :

| # | Étape | Fichier | Fonction / ligne |
|---|---|---|---|
| 1 | Le formulaire est soumis | `components/PlatForm.vue` | enregistrer()/await api.plats.creer(form.value) |
| 2 | La requête HTTP est construite | /src/api.js  | `requete()` |
| 3 | Vite transmet `/api/...` au port 8080 | frontend/vite.config.js|  proxy: {'/api': 'http://localhost:8080',}|
| 4 | CodeIgniter choisit le contrôleur |backend/app/Config/Routes.php| $routes->post('plats', 'Plats::create');|
| 5 | Le contrôleur lit le JSON |Controllers/Api/Plats.php|function create()/ $donnees = $this->donneesJson();|
| 6 | Les règles de validation sont vérifiées |Models/PlatModel.php |$validationRules, $validationMessages |
| 7 | La ligne est insérée | /Controllers/Api/Plats.php| $id = $this->plats->insert($donnees); |
| 8 | Le formulaire prévient l’écran parent |components/PlatForm.vue | `emit(…)` |
| 9 | L’écran recharge la liste | `views/CarteView.vue` |charger()/  api.plats.lister(),|

    
8. Où est le code métier de la création d’une commande ? Pourquoi n’est-il pas dans le contrôleur `Commandes.php` ?
Le Code metier de le creation d une commande est dans `CommandeService.php`
on ne fait pas dans le controleur `Commandes.php`  car il fait un creation et gerer plusieurs table 
comme : `commande_ligne` et `plat`

## 0.5 Repérer les notions de Vue
| Notion | À quoi ça sert | Votre exemple |
|---|---|---|
| `ref()` | une variable dont le changement met l’écran à jour | const message = ref('') |
| `computed()` | une valeur calculée à partir d’autres, recalculée automatiquement |  
const modeEdition = computed(() => props.plat !== null)|
| `v-for` | répéter un élément pour chaque élément d’une liste |
<section v-for="(liste, categorie) in groupes" :key="categorie" class="ardoise-groupe">|
| `v-if` / `v-else` | afficher un élément seulement si une condition est vraie | 
  <p v-if="message" class="alerte" :class="`alerte-${message.type}`">{{ message.texte }}</p>|
| `v-model` | lier un champ de formulaire à une variable, dans les deux sens |
        <input v-model.number="numeroTable" type="number" min="1" />
 |
| `defineProps` | recevoir des données du composant parent | defineProps({
  commande: { type: Object, required: true },
})|
| `defineEmits` / `emit` | envoyer un événement au composant parent |const emit = defineEmits(['plus', 'moins']) |
| `onMounted` | exécuter du code quand le composant apparaît |

onMounted(async () => {
  commandes.value = await api.commandes.lister()
})|

## 0.6 Premières modifications
a) gargote tsara
b) 
```sql
\c gargote;
INSERT INTO categorie (libelle) VALUES ('Goûter');
```

11. La catégorie apparaît dans la liste déroulante du formulaire sans que vous ayez écrit de code. Expliquez pourquoi, en citant la route et le composant concernés.
- la categorie apparait dans la liste sans toucher au code , car le donnees afficher 
au liste deroulant du categorie a envoyer un api qui recupere le donnees du base actuellement
- route concernes : $routes->get('categories', 'Categories::index');
- composants : PlatForm.vue