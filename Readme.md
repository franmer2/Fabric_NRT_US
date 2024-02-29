Microsoft Fabric propose une suite d'outils très pratiques pour accélérer la mise en place de project "quasi temps réel"

Dans cet article, nous allons voir un exemple d'implémentation de ces outils pour mettre en place un rapport Power BI "quasi temps réel" à partir des données de ce [site web](https://bctransit.com/open-data). Plus précismenent nous allons utiliser les données se trouvant [ici](https://bct.tmix.se/gtfs-realtime/vehicleupdates.js?operatorIds=12).

Ci-dessous une vue globale la solution

![Overview](/Pictures/001.jpg)



# Prérequis

- Une [capacité Fabric](https://learn.microsoft.com/fr-fr/fabric/enterprise/buy-subscription#buy-an-azure-sku)
- Une [souscription Azure](https://azure.microsoft.com/en-ca/free/)
- Un service [Azure Logic Apps déployé](https://learn.microsoft.com/fr-fr/azure/logic-apps/logic-apps-overview#get-started)



# Microsoft Fabric
## Base de données KQL


Afin de stocker les données que nous allons récupérer et créer des rapports "quasi temps réel", la base de données KQL est le candidat idéal ici.

Pour commencer, connectez-vous sur le portail [Microsoft Fabric](https://fabric.microsoft.com).

[Créez un nouvel espace de travail](https://learn.microsoft.com/fr-fr/fabric/get-started/create-workspaces) afin d'y déployer les différents services.

Une fois dans votre espace de travail nouvellement créer, choisissez le persona "Real-Time Analytics"

![Persona](/Pictures/002.jpg)

Cliquez sur le bouton "+ New" puis sur "KQL Database"

![KQL](/Pictures/003.png)

Donnez un nom à votre base KQL et cliquez sur le bouton "Create"

![Create](/Pictures/004.png)

Une fois votre base KQL créée, vous devriez obtenir une fenêtre similaire à celle-ci dessous :

![Create](/Pictures/005.png)

## Eventstream

Depuis votre espace de travail, cliquez sur le bouton "+ New" puis sur "Eventstream".

![Eventstream](/Pictures/006.png)

Donnez un nom à votre "Eventstream" puis cliquez sur "Create".

![Eventstream](/Pictures/007.png)

### Creation de l'entrée des évènements

Une fenêtre similaire à celle-ci dessous devrait s'ouvrir. Cliquez sur "New source" puis sur "Custom App"

![Input](/Pictures/008.png)

Sur la droîte, un panneau devrait s'ouvrir. Donnez un nom à la source. Cliquez sur le bouton "Add"

![Eventstream](/Pictures/009.png)

Une fois votre entrée créée, cliquez dessus. Dans la partie inférieure centrale de l'écran cliquez sur "Keys".

Notez les valeurs pour les éleéments suivants, nous allons en avoir besoin ultérieurement :
- Event hub name
- Primary Key

![Eventstream](/Pictures/010.png)

Avant d'aller plus loin avec Microsoft Fabric Eventstream, nous devons envoyer des données dans le moteur d'ingestion afin d'otenir le schéma de données.

Dans note exemple, nous alons déployer un Azure Logic Apps pour récupérer les données et les envoyer à Microsoft Fabric Eventstream. (il est bien entendu possible d'utiliser d'autres solutions comme les "Azure Functions").

# Azure
# Azure logic apps

Depuis le [portail Azure](https://portal.azure.com), allez dans votre service Azure logic app.
Dans le panneau de gauche, cliquez sur "Workflows" puis sur le bouton "Add", donnez un nom à votre Workflow et sélectionnez "Statefeul..." puis cliquez sur "Create".

![LogicApps](/Pictures/015.png)

Une fois votre "workflow" créé, cliquez dessus.

![LogicApps](/Pictures/016.png)

Cliquez sur "Designer" puis sur "Add a trigger"

![LogicApps](/Pictures/017.png)

Comme on désire récupérer les données toutes les 30 secondes, on va choisir un déclencheur de type récurrence. 
Dans la zone de recherche, entrez "recurrence", puis sélectionnez le déclecheur "Schedule / Recurrence".

![LogicApps](/Pictures/019.png)

Définissez les paramètres de votre déclencheur. Ici je demande un déclenchement toutes les 30 secondes.

![LogicApps](/Pictures/020.png)

Sous le déclencheur "Recurrence", cliquez sur le signe "+", cherchez avec le mot clef "https" puis choissisez l'action "HTTP"

![LogicApps](/Pictures/021.png)

Dans l'onglet "Parameters", entrez les valeurs suivantes pour les champs :
- **URI** : https://bct.tmix.se/gtfs-realtime/vehicleupdates.js?operatorIds=12
- **Method** : GET

![LogicApps](/Pictures/022.png)


En dessous de l'action "HTTP", cliquez sur le signe "+", recherchez "Event hub" et sélectionnez "Event Hubs / Sent **E**vent"

![LogicApps](/Pictures/023.png)

Si aucune connexion existe, vous devriez avoir la fenêtre suivante :

![LogicApps](/Pictures/024.png)

L'information concernant la "Connection string" se retrouve dans l'entrée de votre Microsoft Fabric Eventstream (l'information a été notée plus tôt dans cet article).
Cliquez sur le bouton "Create New".

La fenêtre suivante apparaît alors. Renseignez le nom de votre Eventstream dans le champ "Event Hub Name". Dans la liste déroulante "Advanced parameters" sélectionnez "Content".

![LogicApps](/Pictures/025.png)

Cliquez dans le champ "Content" afin de faire apparaîte la petite bulle bleue. Cliquez sur l'éclair. 

![LogicApps](/Pictures/026.png)

Puis sous l'action "HTTP" sélectionnez "Body".

![LogicApps](/Pictures/027.png)

Vous devriez avoir une fenêtre similaire à celle ci-dessous :

![LogicApps](/Pictures/028.png)

Cliquez sur le bouton "Save"

![LogicApps](/Pictures/029.png)

Afin de vérifier que tout se passe bien, après avoir sauvegarder votre "workflow", cliquez sur "Overview" et vérifier que votre "workflow"se déclenche bien toutes les 30 secondes et qu'une erreur n ést repportée dans la colonne "Status.

![LogicApps](/Pictures/030.png)


# Microsoft Fabric (suite)
 
Après avoir créer notre "workflow" Azure Logic Apps, retournez dans Microsoft Fabric, et plus précisement dans votre Eventstream. 
Cliquez sur sur votre Evenstream afin de vérifier que les évènements arrivent correctement.

![LogicApps](/Pictures/031.png)


### Création de la destination des évènements

Cliquez sur "New destination", puis sur "KQL Database".

![Output](/Pictures/011.png)

Choisissez "Direct ingestion" (Dans le cas où vous souhaiteriez faire des transformations avant l'envoie vers la destination, vous avez la possibilité de choisir l'option "Event processing before ingestion"). 
Puis renseignez les informations de votre base KQL. Cliquez sur "Add and configure".

![Output](/Pictures/012.png)

Maintenant, nous allons définir vers quelle table envoyer les données. Cliquez sur "New table".

![Picture](/Pictures/013.png)

Donnez un nom à votre table puis cliquez sur "Next".

![Picture](/Pictures/014.png)


Dans l'étape "Inspect the data", l'assistant devrait pouvoir retrouver un échantillon des données, Cliquez sur "Finish".

![Picture](/Pictures/032.png)

Puis durant l étape "Summary", cliquez sur "Close".

![Picture](/Pictures/033.png)

Votre process d'ingestion est maintenant prêt !

![Picture](/Pictures/034.png)

## KQL Database

Si tout va bien, votre table à du être créée dans votre base KQL. Ci dessous on peut voir ma table "FranmerBronze". Cliquez sur les 3 petits points à droite de la table afin d'aller chercher la commande "Show any 100 records"

![Picture](/Pictures/035.png)

La fenêtre "Explore your data" s'ouvre et vous donne un apreçu des données qui arrivent dans votre table.

![Picture](/Pictures/036.png)

En l'état, les données mne sont pas facilement exploitable pour la création d'un rapport "quasi temps réel". C'est ici que le langage Kusto entre en scène.

### Requête Kusto (KQL Queryset)

Grâce au langage Kusto, il est relativement assez simpe de créer un script qui permet de rendre les données plus facilement exploitable. 
Nous allons utiliser la fonctionnalité "KQL Queryset" afin d'écrire notre requête.

Depuis votre espace de travail, cliquez sur le bouton "New"puis sur le bouton "KQL Queryset".

![Picture](/Pictures/037.png)

Donnez un nom à votre "Queryset" et cliquez sur "Create".

![Picture](/Pictures/038.png)

Sélectionnez votre base KQL et cliquez sur le bouton "Connect".

![Picture](/Pictures/039.png)

Vous devriez avoir une fenêtre similaire à celle ci-dessous :

![Picture](/Pictures/040.png)


Effacez les exemples de script dans la fenêtre principale puis copiez le code ci-dessous :

```java

.create-or-alter function with (docstring="A function to flatten the BCT JSON", folder="Transformation functions") FlattenBctJson(){
FranmerBronze
| extend MyIngestionTime = ingestion_time()
| extend gtfs_realtime_version    = toreal(header.gtfs_realtime_version)
        ,incrementality           = tostring(header.incrementality)
        ,incrementalitySpecified  = tobool(header.incrementalitySpecified)
        // Had to rename the next two fields due to duplicate names further down in the JSON
        // Note how we are converting the unix timestamp on the fly
        ,HeaderTimestamp          = unixtime_seconds_todatetime(tolong(header.timestamp)) 
        ,HeadertimestampSpecified = tobool(header.timestampSpecified) 
        // 
| mv-expand entity 
| extend id	                    = toint(entity.id)
        ,is_deletedSpecified	= tobool(entity.is_deletedSpecified)
        ,vehicle		        = entity.vehicle
        ,Alert                  = entity.alert
| project-away entity, header // cleaning up to free memory
| extend congestion_level               = tostring(vehicle.congestion_level)
        ,congestion_levelSpecified      = tobool(vehicle.congestion_levelSpecified)
        ,consist                        = vehicle.consist // This is an array but is empty now. We should ask if this need to be expanded eventually
        ,current_status                 = tostring(vehicle.current_status)
        ,current_statusSpecified        = tobool(vehicle.current_statusSpecified)
        ,current_stop_sequence          = toint(vehicle.current_stop_sequence)
        ,current_stop_sequenceSpecified = tobool(vehicle.current_stop_sequenceSpecified)
        ,occupancy_statusSpecified      = tobool(vehicle.occupancy_statusSpecified)
        ,position                       = vehicle.position
        ,stop_id                        = tolong(vehicle.stop_id)
        ,stop_idSpecified               = tobool(vehicle.stop_idSpecified)
        ,timestamp                      = unixtime_seconds_todatetime(tolong(vehicle.timestamp))
        ,timestampSpecified             = tobool(vehicle.timestampSpecified)
        ,trip                           = vehicle.trip
| project-away vehicle // cleaning up to free memory
// Unpacking the "position bag"
| extend Latitude           = toreal(position.latitude)
        ,Longitude          = toreal(position.longitude)
        ,Bearing            = toreal(position.bearing)
        ,BearingSpecified   = tobool(position.bearingSpecified)
        ,Odometer           = toreal(position.odometer)
        ,odometerSpecified  = tobool(position.odometerSpecified)
        ,speed              = toreal(position.speed)
        ,speedSpecified     = tobool(position.speedSpecified)
| project-away position // cleaning up to free memory
// Unpacking the Trip bag
| extend  direction_id                   = toint(trip.direction_id)
         ,direction_idSpecified          = tobool(trip.direction_idSpecified)
         ,route_id                       = tostring(trip.route_id)
         ,route_idSpecified              = tobool(trip.route_idSpecified)
         ,schedule_relationship          = tostring(trip.schedule_relationship)
         ,schedule_relationshipSpecified = tobool(trip.schedule_relationshipSpecified)
         ,start_date                     = tostring(trip.start_date)
         ,start_dateSpecified            = tobool(trip.start_dateSpecified)
         ,start_time                     = tostring(trip.start_time) // Not sure why they separate the two... will create new column to recombine
         ,start_timeSpecified            = tobool(trip.start_timeSpecified)
         ,trip_id                        = tostring(trip.trip_id)
         ,trip_idSpecified               = tobool(trip.trip_idSpecified)
| project-away trip // cleaning up to free memory        
// Now we reshuffle the ugly date and recombine the parts to build a proper ISO-8601 date
// Note how Kusto converts it to UTC as God intended!         
| extend TripStartDate = todatetime(
                               strcat(
                                        substring(start_date,0,4)
                                       ,'-'
                                       ,substring(start_date,4,2)
                                       ,'-'
                                       ,substring(start_date,6,2)
                                       , ' '
                                       ,start_time 
                                      )
                                   )
}

```

Vous devriez obtenir quelque chose de similaire à l écran ci-dessous :
![Picture](/Pictures/041.png)

Ce script va créer une fonction dont le nom sera "FlattenBCTJson".
Cliquez sur le bouton "Run".

![Picture](/Pictures/042.png)

Si tout va bien vous devriez obtenir un résultat similaire à celui ci-dessous :

![Picture](/Pictures/043.png)

### Update policy

Nous allons maintenant utiliser la fonctionnalité "Update Policy" afin d'éxécuter la fonction lorsque un évènement arrivera dans la table "FranmerBronze".
Etant donnée que le but de cette fonction est de rendre les données facilement exploitable pour l'analyse ou la création de rapports, nous allons conserver le résultat de la fonction dans une seconde table que j'appellerai "FranmerGold".

Nous allons donc créer une nouvelle table avec le schéma correspondant à la sortie de fla fonction précédement créée. 
Cliquez sur le bouton "+" afin de créer une nouvelle page.

![Picture](/Pictures/044.png)

copier le code suivant et exécutez le en cliquant sur le bouton "Run" (Dans le cas ou le bouton Run est grisé, changez de fenêtre en restant au sein de Microsoft Fabric en allant dans un autre artefact, puis revenez dans votre Queryset):

```java
//Create target table with schema only
.set-or-append FranmerGold <| FlattenBctJson | limit 0

```

![Picture](/Pictures/045.png)

Vous deviez obtenir une seconde table comme illustré ci-dessous :

![Picture](/Pictures/046.png)

Maintenant nous sommes prêt pour créer notre "Update Policy".
Copiez le code ci-dessous dans une nouvelle fenêtre et exécutez le code : 

```java
//Create a table policy
.alter table FranmerGold policy update "[{\"IsEnabled\":true,\"Source\":\"FranmerBronze\",\"Query\":\"FlattenBctJson\",\"IsTransactional\":false,\"PropagateIngestionProperties\":true,\"ManagedIdentity\":null}]"

```

![Picture](/Pictures/047.png)

Si tout va bien vous devriez obtenir un résultat similaire :


![Picture](/Pictures/048.png)

Revenez dans votre base de donnez KQL (Vous pouvez utiliser les "switchs" situés sur la gauche pour changer rapidement d'artefact). Vous devriez voir votre nouvelle table (cliquez sur "Refresh" si ce n ést pas le cas).
Faites une requêtre sur votre nouvelle table afin de voir les nouvelles données arriver dedans.

![Picture](/Pictures/049.png)

Maintenant que les données arrivent en quasi temps réel formattée à notre convenance, nous allons créer un rapport Power BI directement à aprtir de cette table.
Cliquez sur les 3 petits points à droite du nom de votre table et sélectionnez "Build Power BI Report".

![Picture](/Pictures/050.png)

L'éditeur de rapport Power BI s'ouvre dans une autre fenêtre. Vous pouvez maintenant créer un rapport directement à partir des colonnes de la table. Sur la droite, vous y trouverez toutes les colonnes utiles pour contruire votre nouveau rapport.

![Picture](/Pictures/051.png)

Pour que le rapport réagisse en quasi temps réel, **N'OUBLIEZ PAS** de paramétrer le rapport afin qu'il se mette à jour automatiquement suivant la fréquence désirée.
Cliquez sur un espace **vide** du rapport. Cliquez ensuite sur l'icône "Format" puis activez "Page Refresh". Définissez la période de mise à jour automatique. Ici je choisi 30 secondes.

![Picture](/Pictures/052.png)

Il ne vous reste plus qu'a vous construire un beau rapport quasi temps réel !

# Astuce Kusto

Mon collègue [Gilles L'herault](https://www.linkedin.com/in/gilleslherault/), beau jeune homme devant l'éternel, m'a suggéré l'astuce suivante permettant de générer un script permetant de reconstruire complètement la base KQL dans un autre environnement. Utilisez simplement la commande suivante:

```java
.show database YourKqlDbName schema as csl script
```

![Picture](/Pictures/053.png)
