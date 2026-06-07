# Lab — ReceiverDemo : BroadcastReceiver Android (Java)

**Auteur :** DOSSAH Yao Landry  
**Filière :** Génie CyberDéfense et Systèmes de Télécommunications Embarquées (GCDSTE)  
**Établissement :** ENSA Marrakech

---

## Contexte pédagogique

Ce laboratoire porte sur les `BroadcastReceiver` Android — composants qui réagissent aux événements diffusés par le système ou par l'application elle-même. L'application **ReceiverDemo** implémente trois receivers : un receiver dynamique pour le mode avion, un receiver statique pour le démarrage du téléphone (`BOOT_COMPLETED`), et un receiver custom pour les broadcasts internes.

---

## Environnement

| Composant | Détail |
|-----------|--------|
| IDE | Android Studio |
| Langage | Java |
| Min SDK | API 24 |
| Permissions | `RECEIVE_BOOT_COMPLETED` |

---

## Architecture du projet

```
ReceiverDemo/
├── app/src/main/
│   ├── java/com/example/receiverdemo/
│   │   ├── MainActivity.java          ← enregistrement dynamique + envoi custom broadcast
│   │   ├── AirplaneModeReceiver.java  ← receiver dynamique (mode avion)
│   │   ├── BootReceiver.java          ← receiver statique (BOOT_COMPLETED)
│   │   └── CustomEventReceiver.java   ← receiver custom broadcast intra-app
│   ├── res/layout/
│   │   └── activity_main.xml
│   └── AndroidManifest.xml
```

---

## Les trois types de Receiver implémentés

### 1. Receiver dynamique — Mode Avion (`AirplaneModeReceiver`)

Enregistré et désenregistré manuellement dans `MainActivity` via `registerReceiver()` / `unregisterReceiver()`. Il n'est actif que lorsque l'Activity est en vie — ce qui est la bonne pratique pour préserver la batterie.

```java
IntentFilter filter = new IntentFilter();
filter.addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED);
registerReceiver(airplaneReceiver, filter);
```

Dans `onReceive()`, l'état du mode avion est lu via `intent.getBooleanExtra("state", false)` et un Toast informe l'utilisateur immédiatement.

**Important :** `onReceive()` s'exécute sur le thread principal — aucune opération bloquante (réseau, I/O) ne doit y être faite.

### 2. Receiver statique — Boot (`BootReceiver`)

Déclaré dans le Manifest avec un `<intent-filter>` sur `BOOT_COMPLETED`. Il se déclenche même si l'application n'est pas lancée — c'est l'une des rares exceptions autorisées par Android pour les receivers statiques.

```xml
<receiver
    android:name=".BootReceiver"
    android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

### 3. Receiver custom — Broadcast intra-app (`CustomEventReceiver`)

`MainActivity` envoie un broadcast avec une action personnalisée et un extra `message`. `CustomEventReceiver` l'intercepte et affiche le contenu via Toast.

```java
Intent intent = new Intent("com.example.receiverdemo.CUSTOM_EVENT");
intent.putExtra("message", "Bonjour depuis le custom broadcast !");
sendBroadcast(intent);
```

---

## Cycle de vie et gestion mémoire

`MainActivity` suit rigoureusement le cycle register/unregister pour éviter les fuites mémoire :

```
onCreate()    → instanciation du receiver
btnToggle     → registerReceiver() / unregisterReceiver()
onDestroy()   → unregisterReceiver() si encore enregistré
```

---

## Résultats observés

**Receiver mode avion :** après activation du receiver via le bouton, activer/désactiver le mode avion dans les paramètres système déclenche immédiatement le Toast correspondant — sans aucun délai perceptible.

**Custom broadcast :** le bouton "Envoyer Custom Broadcast" déclenche instantanément le Toast "Custom reçu : Bonjour depuis le custom broadcast !" — le broadcast reste strictement intra-app grâce à `exported="false"`.

**Boot receiver :** après redémarrage de l'émulateur (`adb reboot`), le Toast "Téléphone démarré - Receiver statique activé !" apparaît sans que l'application soit ouverte manuellement.

---

## Points clés retenus

- **Dynamique vs Statique** — le receiver dynamique est recommandé pour tout ce qui est non critique ; le statique est réservé aux événements système rares comme `BOOT_COMPLETED`.
- **`exported="false"`** — obligatoire depuis Android 12 pour tout receiver déclaré dans le Manifest, afin d'empêcher des applications tierces d'envoyer des broadcasts non sollicités.
- **Thread principal** — `onReceive()` s'exécute sur le thread UI ; toute opération longue doit être déléguée à un `Service` ou un `WorkManager`.
- **Nettoyage obligatoire** — un receiver dynamique non désenregistré provoque une fuite mémoire et une `IllegalArgumentException` au redémarrage de l'Activity.
- **Broadcasts implicites restreints** — depuis Android 8+, la plupart des broadcasts implicites système ne peuvent plus être reçus par des receivers statiques ; `BOOT_COMPLETED` reste une exception explicitement autorisée.

---

*Lab réalisé dans le cadre du cours Développement Mobile — ENSA Marrakech, Filière GCDSTE*
