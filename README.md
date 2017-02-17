# Mettre à l'abri les signatures d'applications Android

Afin de ne jamais laisser de traces du keypassword ou storepassword sur un remote repo git...

## Comment faire
1. Dans notre projet, créer `gradle/signing.gradle` et y mettre:
```groovy
if (hasProperty(SIGNING_PROPERTY_NAME)) {
    def String signingScriptPath = getProperty(SIGNING_PROPERTY_NAME) + ".gradle";
    if (new File(signingScriptPath).exists()) {
        apply from: signingScriptPath;
    }
}
```
 
2. On définit donc une nouvelle propriété dans notre `gradle.properties` du projet:
```
SIGNING_PROPERTY_NAME=soutenance.signing
```
Cette propriété contiendra le chemin vers notre `.gradle` et notre `.jks` qui seront bien l'abri sur notre machine.

3. Nous allons tout de suite la définir dans `~/.gradle/gradle.properties` (à créer si inexistant):
```
soutenance.signing=/Users/USERNAME/.signing/soutenance
```

4. Créez `/Users/USERNAME/.signing/soutenance.gradle` et mettez-y:
```groovy
android {
  signingConfigs {
    release {
      storeFile file(getProperty("soutenance.signing").toString() + ".jks")
      storePassword "mon-keypassword-ultra-top-secret"
      keyAlias "soutenance"
      keyPassword "mon-keypassword-ultra-top-secret"
    }
  }

  buildTypes {
    release {
      signingConfig signingConfigs.release
    }
  }
}
```


5. La génération du keystore, notez bien les mots de passe que vous renseignez afin de les réutiliser dans la configuration de gradle:
```bash
keytool -keystore ~/.signing/soutenance.jks -genkey -alias soutenance
```

6. Ajoutez ce qui suit à votre `build.gradle` au niveau du module :
 
 ```groovy
 
 android { 
     signingConfigs {
         debug {
             storeFile file("../debug.keystore")
             storePassword "android"
             keyAlias "androiddebugkey"
             keyPassword "android"
         }
         release {
         }
     }
 
     buildTypes {
         debug {
             minifyEnabled false
         }
         release {
             minifyEnabled true
             proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
             signingConfig signingConfigs.release // IMPORTANT
         }
     }
 }
 
 dependencies {
    // ...
 }
 
 
 //----------------------------------------------------------------------------------------------------------------------
 // Signing configuration
 //-- IMPORTANT: Appel du script qui ira chercher le gradle config du signing
 apply from: "$rootDir/gradle/signing.gradle";
 ```
 
### Signer l'application
Pour finir, une simple commande `sh gradlew aR` (ou assembleRelease) pour générer un APK signé