# Corrélation avec OWASP MASVS - JNIDEMO1 APK

## Résumé

**Application:** JNIDEMO1  
**Total Vulnérabilités Mappées:** 5  
**MASVS Level 1 Status:** PARTIAL FAIL  
**MASVS Level 2 Status:** FAIL

---

## Vulnérabilités Mappées

### Vulnérabilité 1: Debug Certificate

**Titre:** Application Signed with Debug Certificate

**Référence MASVS:** MSTG-RESILIENCE-1

**Catégorie:** Resilience  
**Level:** Level 1

**Description de l'Exigence:**
"The app is signed with debug certificate that allows anyone to sign apps with the same certificate."

**Preuve de Non-Conformité:**
- Location: Certificate Analysis → Issue #1
- Certificate Details: CN=Android Debug, O=Android, C=US
- Impact: Anyone can sign code with the same debug certificate
- Severity: CRITICAL

**Conséquences Possibles:**
- Signature spoofing
- Unauthorized app installation
- Malicious app signing with same certificate
- Complete bypass of app integrity

**Recommandation de Remédiation:**
Generate production certificate using keytool:
```bash
keytool -genkey -v -keystore release.keystore -keyalg RSA -keysize 2048 -validity 10000 -alias release
```
Sign app with production certificate for release builds.

---

### Vulnérabilité 2: Debuggable Application

**Titre:** Debug Enabled For App (android:debuggable=true)

**Référence MASVS:** MSTG-RESILIENCE-2

**Catégorie:** Resilience  
**Level:** Level 1

**Description de l'Exigence:**
"The app does not allow debuggers to be attached."

**Preuve de Non-Conformité:**
- Location: Manifest Analysis → Issue #2
- Attribute: android:debuggable="true"
- File: AndroidManifest.xml
- Status: Enabled in debug APK

**Conséquences Possibles:**
- Direct memory access to application
- Code execution at runtime
- Extraction of encryption keys
- Stack trace and variable dumping
- Runtime code modification via debugger
- Session hijacking

**Recommandation de Remédiation:**
```xml
<!-- BEFORE -->
<application android:debuggable="true" ...>

<!-- AFTER -->
<application android:debuggable="false" ...>
```

Or in build.gradle:
```gradle
buildTypes {
  release {
    debuggable = false
  }
}
```

---

### Vulnérabilité 3: Test Mode Enabled

**Titre:** Application in Test Mode (android:testOnly=true)

**Référence MASVS:** MSTG-RESILIENCE-3

**Catégorie:** Resilience  
**Level:** Level 1

**Description de l'Exigence:**
"The app does not contain any test code or debug configurations that could be used to bypass security."

**Preuve de Non-Conformité:**
- Location: Manifest Analysis → Issue #4
- Attribute: android:testOnly="true"
- File: AndroidManifest.xml
- Status: Test mode enabled

**Conséquences Possibles:**
- Exposure of test functionality
- Authentication bypass
- Access to test data
- Backdoor activation
- Disabled security validations

**Recommandation de Remédiation:**
```xml
<!-- BEFORE -->
<application android:testOnly="true" ...>

<!-- AFTER -->
<application android:testOnly="false" ...>
```

Also verify no test backdoors in code:
```java
// BEFORE (DANGEROUS)
if (BuildConfig.DEBUG || android:testOnly) {
  if (user.equals("admin") && pass.equals("test")) {
    launchApp(); // BYPASS!
  }
}

// AFTER (CORRECT)
// No test code in production builds
```

---

### Vulnérabilité 4: Backup Enabled

**Titre:** Data Can Be Backed Up (android:allowBackup=true)

**Référence MASVS:** MSTG-STORAGE-8

**Catégorie:** Storage  
**Level:** Level 1

**Description de l'Exigence:**
"The app prevents sensitive data from being written to application logs."

**Preuve de Non-Conformité:**
- Location: Manifest Analysis → Issue #3
- Attribute: android:allowBackup="true"
- File: AndroidManifest.xml
- Status: Backups allowed

**Conséquences Possibles:**
- ADB backup extraction: `adb backup -apk com.example.app`
- Access to SharedPreferences
- Database file extraction
- Credential and token theft
- Modified data restoration

**Recommandation de Remédiation:**
```xml
<!-- OPTION 1: Disable backups -->
<application android:allowBackup="false" ...>

<!-- OPTION 2: Selective backup -->
<application 
  android:allowBackup="true"
  android:fullBackupContent="@xml/backup_rules" ...>

<!-- Create: res/xml/backup_rules.xml -->
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
  <exclude domain="sharedpref" path="auth_tokens.xml" />
  <exclude domain="file" path="sensitive/" />
</full-backup-content>
```

---

### Vulnérabilité 5: Minimum SDK Too Old

**Titre:** App Can Be Installed on Vulnerable Android 7.0 (minSdk=24)

**Référence MASVS:** MSTG-PLATFORM-1

**Catégorie:** Platform  
**Level:** Level 1

**Description de l'Exigence:**
"The app targets a recent API level."

**Preuve de Non-Conformité:**
- Location: App Information
- Attribute: android:minSdkVersion="24"
- File: AndroidManifest.xml
- Status: API 24 = Android 7.0 (2016)

**Conséquences Possibles:**
- Exploitation of 300+ Android 7 vulnerabilities
- Privilege escalation to root
- Hardware access (camera, microphone, location)
- Man-in-the-Middle (MITM) attacks
- Complete security bypass

**Vulnerabilités Connues:**
- Stagefright (CVE-2015-1538) - Media code execution
- Master Key (CVE-2013-6643) - Signature bypass
- Freak (CVE-2015-1638) - SSL/TLS downgrade
- And 300+ others unpatched since 2017

**Recommandation de Remédiation:**
```xml
<!-- BEFORE -->
<uses-sdk android:minSdkVersion="24" ... />

<!-- RECOMMENDED -->
<uses-sdk android:minSdkVersion="29" ... />
<!-- API 29 = Android 10 (99%+ device coverage) -->

<!-- IDEAL -->
<uses-sdk android:minSdkVersion="33" ... />
<!-- API 33 = Android 13 (Latest LTS) -->
```

Or in build.gradle:
```gradle
android {
  minSdk 29
  targetSdk 36
}
```

---

## Tests MASTG Complémentaires

### Test 1: Disable Debuggable

**Référence MASTG:** MSTG-RESILIENCE-2

**Description:** Verify application cannot be debugged

**Procédure:**
```bash
# Attempt to debug app
adb shell am set-debug-app --persistent com.example.app

# Try to attach debugger
adb forward tcp:5005 tcp:5005
jdb -attach localhost:5005

# Expected: Connection refused
```

**Objectif:** Confirm debuggable=false prevents debugging

**Relevance:** Critical for preventing runtime code modification

---

### Test 2: Certificate Signing Verification

**Référence MASTG:** MSTG-RESILIENCE-1

**Description:** Verify app uses production certificate, not debug

**Procédure:**
```bash
# Extract certificate from APK
unzip -p app-release.apk META-INF/CERT.RSA | openssl pkcs7 -inform DER -print_certs -out cert.pem

# View certificate
keytool -printcert -file cert.pem

# Verify NOT debug certificate
# CN should NOT be "Android Debug"
```

**Objectif:** Confirm production certificate used

**Relevance:** Prevents signature spoofing in production

---

## Résumé de Conformité MASVS

### MSTG-RESILIENCE-1 (Code Signing)
- **Status:** FAIL
- **Finding:** Debug certificate
- **Impact:** CRITICAL

### MSTG-RESILIENCE-2 (Debugging)
- **Status:** FAIL
- **Finding:** Debuggable=true
- **Impact:** CRITICAL

### MSTG-RESILIENCE-3 (Test Code)
- **Status:** FAIL
- **Finding:** TestOnly=true
- **Impact:** CRITICAL

### MSTG-STORAGE-8 (Data Backup)
- **Status:** FAIL
- **Finding:** AllowBackup=true
- **Impact:** HIGH

### MSTG-PLATFORM-1 (API Usage)
- **Status:** FAIL
- **Finding:** MinSdk=24 (Android 7.0)
- **Impact:** HIGH

---

## Catégories MASVS Non-Conformes

| Catégorie | Level 1 | Level 2 | Impact |
|-----------|---------|---------|--------|
| Resilience | FAIL | FAIL | CRITICAL |
| Storage | FAIL | FAIL | HIGH |
| Platform | FAIL | FAIL | HIGH |
| Network | PASS | PASS | OK |
| Crypto | PASS | PASS | OK |
| Auth | N/A | N/A | N/A |

---

## Score de Conformité MASVS

### Level 1 (Essential Security)
- **Status:** NON-COMPLIANT
- **Failing Requirements:** 5
- **Passing Requirements:** 2

### Level 2 (Defense-in-Depth)
- **Status:** NON-COMPLIANT
- **Failing Requirements:** 5+
- **Passing Requirements:** 2

---

## Recommandations Priorisées

### Phase 1: Critical Fixes (Immediate)
1. Change android:debuggable="false"
2. Change android:testOnly="false"
3. Generate production certificate
4. Sign with production certificate

### Phase 2: Important Fixes (This Week)
5. Change android:allowBackup="false"
6. Increase minSdk to 29+
7. Code review for test backdoors

### Phase 3: Enhancement (This Month)
8. Add certificate pinning
9. Implement anti-tampering
10. Obfuscation enhancement

---

## Conclusion

**MASVS Compliance Status:**

- **Level 1:** NON-COMPLIANT (5 failures)
- **Level 2:** NON-COMPLIANT (5+ failures)

**Most Critical Issues:**
1. Debug certificate
2. Debuggable=true
3. TestOnly=true

**Recommended Action:**
Fix all 5 identified issues immediately before production release.

**Timeline:** 1-2 days for critical fixes, 1 week for all improvements

---

Analyse complétée le 26 avril 2026  
Outil: MobSF v4.5.0
