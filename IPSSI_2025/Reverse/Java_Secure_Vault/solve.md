# **Java SecureVault** (Reverse Engineering)  
<br>

## **1. Extraction du JAR**

La première étape consiste à extraire le contenu du fichier JAR:
```shell
└─$ unzip SecureVault.jar 
Archive:  SecureVault.jar
   creating: META-INF/
  inflating: META-INF/MANIFEST.MF    
  inflating: Main.class              
  inflating: PasswordValidator.class
```
Plusieurs fichiers sont extraits, notamment Main.class et PasswordValidator.class, qui contiennent la logique principale de l'application.

<br>

## **2. Analyse des fichiers .class**

L'outil javap nous permet de désassembler les classes en bytecode, pour avoir une vue d’ensemble de leur fonctionnement :
```shell
─$ javap -c -p Main.class 

─$ javap -c -p PasswordValidator.class
```
- Main.class : Cette classe est chargée de l'interaction avec l'utilisateur. Elle demande un mot de passe et délègue la vérification à la classe PasswordValidator.

- PasswordValidator.class : Cette classe reçoit le mot de passe saisi et le compare à une valeur attendue. Le processus de vérification consiste à transformer une chaîne de caractères par une opération XOR avec la chaîne statique "5up3rS3cr3t!".

<br>

## **3. Récupération du mot de passe avec un script Python**
   
Pour retrouver le bon mot de passe, nous pouvons reproduire l'opération XOR réalisée par la classe PasswordValidator, grâce à ce script python:
```python
# La chaîne à vérifier
expected_string = "x42FA!A!"

# Clé XOR
xor_key = "5up3rS3cr3t!"

# Convertion des chaînes en bytes
expected_bytes = bytes(expected_string, 'utf-8')
xor_bytes = bytes(xor_key, 'utf-8')

# Opération XOR pour récupérer le mot de passe
password_bytes = bytearray()
for i in range(len(expected_bytes)):
    password_bytes.append(expected_bytes[i] ^ xor_bytes[i % len(xor_bytes)])

# Conversion
password = password_bytes.decode('utf-8')
print("Mot de passe :", password)
```

**Résultat :** `Mot de passe : MABu3rrB`
