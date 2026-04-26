# Analyse des Permissions - JNIDEMO1 APK

## Résumé

**Total Permissions Déclarées:** 1  
**Permissions Dangereuses:** 0  
**Permissions Normales:** 0  
**Permissions Custom:** 1  
**Permissions Malware:** 0/25

---

## Permissions Dangereuses

**STATUS: AUCUNE DETECTEE**

Les permissions "dangereuses" (Dangerous Permissions Level) de Google Play incluent:
- SEND_SMS
- RECEIVE_SMS
- READ_SMS
- ACCESS_FINE_LOCATION
- ACCESS_COARSE_LOCATION
- CAMERA
- RECORD_AUDIO
- READ_CONTACTS
- WRITE_CONTACTS
- READ_CALL_LOG
- WRITE_CALL_LOG
- READ_EXTERNAL_STORAGE
- WRITE_EXTERNAL_STORAGE

**Verdict:** JNIDEMO1 n'en déclare AUCUNE

---

## Permissions Normales

**STATUS: AUCUNE DETECTEE**

Les permissions "normales" (Normal Permissions Level) incluent:
- INTERNET
- ACCESS_NETWORK_STATE
- CHANGE_NETWORK_STATE
- BLUETOOTH
- MODIFY_AUDIO_SETTINGS

**Verdict:** JNIDEMO1 n'en déclare AUCUNE

(Application utilise probablement des permissions implicites via les bibliothèques AndroidX)

---

## Permissions Custom Déclarées

### 1. com.example.jnidemo1.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION

| Propriété | Valeur |
|-----------|--------|
| Type | CUSTOM (Définie par l'application) |
| Status | UNKNOWN |
| Level | Custom |
| Usage | Probablement utilisée pour protéger un Receiver |

**Analyse:**
- Permission personnalisée déclarée dans le manifeste
- Non reconnue par Android ou Google Play
- Utilisée pour contrôler l'accès à un composant spécifique
- Risque: MINIMAL (Permission personnalisée, non dangereuse)

---

## Permissions Implicites via Librairies

L'application utilise les bibliothèques AndroidX suivantes:
- androidx.activity:activity@1.8.0
- androidx.appcompat:appcompat@1.7.1
- androidx.core:core@1.13.0
- androidx.fragment:fragment@1.5.4
- androidx.lifecycle:lifecycle-viewmodel@dynamic
- com.google.android.material:material@1.13.0
- androidx.profileinstaller:profileinstaller@1.4.0

Ces librairies peuvent ajouter des permissions comme INTERNET et ACCESS_NETWORK_STATE (nécessaires et standard).

---

## Matrice de Risque

| Permission Type | Count | Risk |
|-----------------|-------|------|
| Dangerous | 0 | NONE |
| Normal | 0 | NONE |
| Custom | 1 | LOW |
| Malware-related | 0/25 | NONE |

**Risque Global de Permissions:** TRES FAIBLE

---

## Conformité aux Standards

### OWASP MASVS MSTG-PLATFORM-1
Requirement: "Apps should request a minimal set of permissions"  
**Status:** PASS

### Google Play Policy
Requirement: "Apps must not request more permissions than needed"  
**Status:** COMPLIANT

### Android Security Best Practices
Reference: https://developer.android.com/privacy-and-security/permissions  
**Status:** FOLLOWS BEST PRACTICES

---

## Recommandations

### Pour Développeur
- Continuer cette approche minimale en permissions
- Déclarer uniquement les permissions strictement nécessaires
- Si ajouter des features nécessitant permissions: justifier chacune
- Utiliser Runtime Permissions pour permissions dangereuses

### Pour Sécurité
- Aucune action requise pour permissions
- Configuration actuelle est sécurisée
- Continuer à limiter les demandes de permissions

---

## Conclusion

L'application JNIDEMO1 suit les meilleures pratiques en matière de permissions:
- AUCUNE permission dangereuse
- AUCUNE permission malveillante
- Approche minimale appropriée
- Utilisation correcte des permissions custom

**STATUS:** COMPLIANT avec OWASP MASVS et Google Play Policy

**Risque Global:** TRES FAIBLE

---

Analyse complétée le 26 avril 2026  
Outil: MobSF v4.5.0
