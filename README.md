[README.md](https://github.com/user-attachments/files/25110894/README.md)
# Système de Placement d'Équipements Industriels - Unreal Engine

## Description

Système complet permettant aux joueurs de placer des équipements industriels (pompes, équipements sous pression) sur des plateformes et de les connecter via des tuyauteries et câbles électriques.

## Fonctionnalités

### ✅ Placement d'Équipements
- Placement interactif avec feedback visuel (vert = valide, rouge = invalide)
- Validation automatique de la position (détection de plateforme, anti-collision)
- Rotation libre ou par incréments
- Snap to grid optionnel
- Système de catalogue pour choisir les équipements

### ✅ Système de Connexions
- **Tuyauteries hydrauliques** : Diamètres DN25 à DN200
- **Câbles électriques** : Connexions avec calibres variés
- **Tuyauteries pneumatiques** : Pour systèmes à air comprimé
- Placement multi-points avec chemins personnalisables
- Validation automatique de compatibilité des connexions

### ✅ Gestion des Équipements
- Catalogue extensible via Data Asset
- Propriétés: hauteur, diamètre, poids, consommation électrique
- Points de connexion multiples par équipement
- Suivi des connexions actives

## Architecture du Code

### Classes Principales

#### `AIndustrialEquipmentBase`
Classe de base pour tous les équipements industriels.

**Propriétés clés:**
- `FEquipmentData`: Données de l'équipement (taille, poids, mesh, etc.)
- `TArray<FConnectionPoint>`: Points de connexion disponibles
- `bIsPlaced`: État de placement
- `bIsPowered`: État d'alimentation électrique

**Méthodes principales:**
```cpp
void PlaceEquipment(const FVector& Location, const FRotator& Rotation);
bool CanBePlacedAt(const FVector& Location);
TArray<FConnectionPoint> GetAvailableConnectionPoints(EConnectionType Type);
FConnectionPoint* GetConnectionPointByName(FName SocketName);
```

#### `APipelineSystem`
Gère la création et visualisation des tuyauteries/câbles.

**Propriétés clés:**
- `EPipeDiameter`: Diamètre du tuyau
- `EConnectionType`: Type (Hydraulique, Électrique, Pneumatique)
- `TArray<FVector> PathPoints`: Chemin du pipeline
- `TArray<FPipeSegment>`: Segments visuels

**Méthodes principales:**
```cpp
void StartPipeline(AIndustrialEquipmentBase* Equipment, FName ConnectionPointName);
void AddPathPoint(const FVector& Point);
bool CompletePipeline(AIndustrialEquipmentBase* Equipment, FName ConnectionPointName);
void UpdatePreviewToPoint(const FVector& Point);
```

#### `UEquipmentCatalog` (Data Asset)
Catalogue d'équipements disponibles.

**Méthodes:**
```cpp
TArray<FEquipmentData> GetEquipmentByType(EEquipmentType Type);
FEquipmentData GetEquipmentByName(const FString& Name);
```

#### `UEquipmentManagerSubsystem`
Subsystem gérant tous les équipements et pipelines placés.

**Méthodes principales:**
```cpp
AIndustrialEquipmentBase* SpawnEquipmentFromCatalog(const FString& EquipmentName, const FVector& Location);
void RemoveEquipment(AIndustrialEquipmentBase* Equipment);
APipelineSystem* CreatePipeline(EPipeDiameter Diameter, EConnectionType Type);
float GetTotalPowerConsumption() const;
```

#### `AIndustrialPlayerController`
Contrôleur joueur gérant toutes les interactions.

**Modes de placement:**
- `PlacingEquipment`: Placement d'équipement
- `PlacingPipeline`: Création de tuyauterie
- `SelectingEquipment`: Sélection/édition

## Configuration dans Unreal Engine

### 1. Intégration des fichiers C++

1. Copiez tous les fichiers .h et .cpp dans votre projet Unreal : `YourProject/Source/YourProject/`
2. Ajoutez les dépendances dans votre `.Build.cs` :

```csharp
PublicDependencyModuleNames.AddRange(new string[] { 
    "Core", 
    "CoreUObject", 
    "Engine", 
    "InputCore",
    "UMG",
    "Slate",
    "SlateCore"
});
```

3. Compilez le projet

### 2. Créer le Data Asset du Catalogue

1. Dans l'éditeur : **Clic droit** → **Miscellaneous** → **Data Asset**
2. Choisir `UEquipmentCatalog`
3. Nommer : `DA_EquipmentCatalog`

### 3. Configurer le Catalogue

Dans le Data Asset, ajoutez vos équipements :

**Exemple de Pompe:**
```
Equipment Name: "Pompe Centrifuge 50m³/h"
Equipment Type: Pump
Height: 3.0
Diameter: 1.5
Weight: 850.0
Power Requirement: 15.0
Mesh: [Votre Static Mesh]
Connection Points:
  - Socket Name: "Inlet"
    Connection Type: Hydraulic
    Diameter: 0.1 (DN100)
    Relative Location: (0, -75, 150)
  - Socket Name: "Outlet"
    Connection Type: Hydraulic
    Diameter: 0.1
    Relative Location: (0, 75, 150)
  - Socket Name: "Power"
    Connection Type: Electric
    Diameter: 0.032 (32A)
    Relative Location: (50, 0, 200)
```

**Exemple d'Équipement sous Pression:**
```
Equipment Name: "Réservoir Pression 10 bars"
Equipment Type: Pressure Vessel
Height: 3.0
Diameter: 1.5
Weight: 1200.0
Power Requirement: 0.0
Mesh: [Votre Static Mesh]
Connection Points:
  - Socket Name: "TopInlet"
    Connection Type: Hydraulic
    Diameter: 0.05 (DN50)
    Relative Location: (0, 0, 300)
  - Socket Name: "BottomOutlet"
    Connection Type: Hydraulic
    Diameter: 0.08 (DN80)
    Relative Location: (0, 0, 0)
```

### 4. Créer les Blueprints

#### Blueprint du PlayerController

1. Créer un Blueprint basé sur `AIndustrialPlayerController`
2. Nommer : `BP_IndustrialPlayerController`
3. Configurer :
   - **Default Catalog** : Référencer votre `DA_EquipmentCatalog`
   - **Placement Grid Size** : 50.0
   - **Snap To Grid** : True
   - **Valid Placement Material** : Matériau vert semi-transparent
   - **Invalid Placement Material** : Matériau rouge semi-transparent

#### Configurer les Input Mappings

Dans **Project Settings** → **Input** :

**Action Mappings:**
```
LeftClick → Left Mouse Button
RightClick → Right Mouse Button
RotateClockwise → E
RotateCounterClockwise → Q
Cancel → Escape
Delete → Delete
```

### 5. Créer les Meshes de Tuyauterie

Créez un Static Mesh cylindrique pour les tuyaux :
- Longueur : 100 unités
- Rayon : 1 unité (sera mis à l'échelle)
- Pivots aux extrémités pour le SplineMesh

**Matériaux recommandés:**
- Tuyauterie hydraulique : Métal gris/acier
- Câble électrique : Isolant noir/jaune/vert
- Tuyauterie pneumatique : Bleu/rouge

### 6. Configuration du GameMode

Dans votre GameMode Blueprint :
- **Player Controller Class** : `BP_IndustrialPlayerController`

## Utilisation en Jeu

### Placement d'un Équipement

1. **Ouvrir le catalogue** : Appeler `ToggleCatalog()` depuis votre UI
2. **Sélectionner un équipement** : Cliquer sur un équipement dans le catalogue
3. **Positionner** : Déplacer la souris pour positionner
4. **Tourner** : Touches E/Q pour rotation
5. **Valider** : Clic gauche si vert (valide)
6. **Annuler** : Échap

### Création d'une Tuyauterie

**Méthode 1 : Connection Directe**
1. Sélectionner un équipement placé
2. Choisir le type de connexion (Hydraulique/Électrique)
3. Choisir le diamètre
4. Clic sur le point de départ
5. Clic sur le point d'arrivée

**Méthode 2 : Avec Points Intermédiaires**
1. Clic gauche : Point de départ (sur équipement)
2. Clic droit : Ajouter des points intermédiaires
3. Clic gauche : Point final (sur équipement compatible)

### Suppression

1. Sélectionner un équipement : Clic gauche
2. Supprimer : Touche Delete
   - Supprime l'équipement ET toutes ses connexions

## Extension du Système

### Ajouter un Nouveau Type d'Équipement

1. Ajouter dans l'enum `EEquipmentType` :
```cpp
UENUM(BlueprintType)
enum class EEquipmentType : uint8
{
    Pump,
    PressureVessel,
    Tank,
    Compressor,
    HeatExchanger // NOUVEAU
};
```

2. Créer une classe dérivée si besoin :
```cpp
UCLASS()
class AHeatExchanger : public AIndustrialEquipmentBase
{
    GENERATED_BODY()
    
    // Logique spécifique...
};
```

3. Ajouter au catalogue

### Ajouter un Nouveau Type de Connexion

```cpp
UENUM(BlueprintType)
enum class EConnectionType : uint8
{
    Hydraulic,
    Electric,
    Pneumatic,
    Steam // NOUVEAU
};
```

### Personnaliser la Validation de Placement

Overrider dans votre classe dérivée :
```cpp
bool AMyCustomEquipment::CanBePlacedAt(const FVector& Location)
{
    // Votre validation personnalisée
    if (!Super::CanBePlacedAt(Location))
        return false;
    
    // Exemple : Vérifier la proximité d'une source d'eau
    if (!IsNearWaterSource(Location))
        return false;
    
    return true;
}
```

## Optimisations Possibles

### Performance
- Utiliser des Instance Static Meshes pour les tuyaux
- LOD sur les équipements complexes
- Occlusion culling pour les grandes installations

### Gameplay
- Système de coût par équipement
- Limites de consommation électrique totale
- Calcul de pression dans les pipelines
- Simulation de flux

### UI
- Minimap avec équipements
- Statistiques en temps réel
- Alertes de maintenance
- Mode "schéma" 2D

## Troubleshooting

**L'équipement ne se place pas (toujours rouge)**
- Vérifier qu'il y a bien une surface en dessous
- Vérifier les collisions dans le Static Mesh
- Augmenter la distance de trace

**Les tuyauteries ne se connectent pas**
- Vérifier que les types de connexion correspondent
- Vérifier que les points ne sont pas déjà connectés
- Vérifier les noms des sockets

**Crash au spawn**
- Vérifier que le catalogue est bien assigné
- Vérifier que les meshes sont valides
- Vérifier les includes et dépendances

## Ressources

- [Unreal Engine Documentation - Spline Mesh Component](https://docs.unrealengine.com/en-US/API/Runtime/Engine/Components/USplineMeshComponent/)
- [Unreal Engine Documentation - Data Assets](https://docs.unrealengine.com/en-US/WorkingWithContent/DataAssets/)
- [Unreal Engine Documentation - World Subsystems](https://docs.unrealengine.com/en-US/ProgrammingAndScripting/Subsystems/)

## License

Code fourni comme exemple éducatif. Libre d'utilisation et de modification.

---

**Bon développement ! 🏭**
