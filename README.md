# OpenNovaLauncher

Workspace di reverse engineering e rebranding di un launcher Android, derivato dalla decompilazione
di un APK di terze parti. Uso locale, prima parte, interoperabilità.

## Struttura

```
re/
  opennovalauncher.apk     APK ricostruito, rebrandizzato e firmato (debug key)
  jadx-basic/              decompilazione jadx (sorgenti + risorse)
  jadx-full/               decompilazione jadx estesa
  single/                  file Java isolati per analisi mirata
  jadx.log                 log di decompilazione
apk/                       APK decodificato con apktool (risorse modificabili, smali assente)
```

## Cosa è stato rinominato

- Nome applicazione e tutte le stringhe visibili: `OpenNovaLauncher` / `OpenNova`
- Nomi di stile e risorse che contenevano il brand
- MIME type dei backup: `application/vnd.opennovalauncher.backup`
- Icona launcher: nuova, monogramma `ON`, con adaptive icon (sfondo gradiente vettoriale
  `res/drawable/opennova_icon_bg.xml`, foreground `res/mipmap-*/adaptive_foreground_home.png`,
  legacy `res/mipmap-*/ic_launcher_home.png`)

## Cosa NON è stato rinominato (per compatibilità)

- Package `com.teslacoilsw.launcher`
- Nomi delle classi Java (`NovaLauncher`, `NovaApplication`, ...) e relativi riferimenti nel manifest
- Nomi interni delle risorse (`nova_action`, ...), solo i valori testuali sono cambiati

Motivo: l'APK è stato ricostruito a partire da risorse decodificate con apktool senza smali.
Rinominare package o classi richiederebbe la decompilazione e ricompilazione dei bytecode, non
disponibile in questo workspace.

## Rimozione tracker

`re/opennovalauncher.apk` è la build con tracker neutralizzati (`tools/strip-trackers.py`):

- ContentProvider/Service rimossi dal manifest: Branch (3 provider + 8 service), Meta Audience
  Network, Vungle, Instabridge
- Permessi rimossi: `AD_ID`, `ACCESS_ADSERVICES_ATTRIBUTION`, `ACCESS_ADSERVICES_AD_ID`,
  `BIND_GET_INSTALL_REFERRER_SERVICE`
- `zy/c` (facade Firebase): metodi di invio eventi e `recordException` resi no-op
- `yx/b` (wrapper Amplitude): init e tracking resi no-op
- `NovaApplication`: Amplitude, Crashlytics e Firebase Analytics non inizializzati;
  Instabridge `isEligibleForMobile` forzato a false

Le classi degli SDK restano nel dex: R8 ha fuso classi eterogenee negli stessi file, eliminarle
romperebbe l'app. Sono disattivate, non cancellate.

Riproduzione:

```bash
apktool d re/opennovalauncher.apk -o apk-full
python3 tools/strip-trackers.py apk-full
apktool b apk-full -o unsigned.apk
```

## Build

```bash
java -jar apktool.jar b apk -o unsigned.apk
zipalign -f -p 4 unsigned.apk aligned.apk
apksigner sign --ks ~/.android/debug.keystore --ks-key-alias androiddebugkey \
  --ks-pass pass:android --key-pass pass:android --out opennovalauncher.apk aligned.apk
```

Nota: due attributi privati del framework (`colorAccentPrimary`, `colorProgressBackgroundNormal`)
sono stati sostituiti con `colorAccent` e `colorBackground` per permettere il rebuild con aapt2.

## Avvertenza

Codice decompilato da software di terzi. Non pubblicare, non distribuire, non caricare su store.
