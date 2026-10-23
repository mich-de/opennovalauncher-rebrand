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
