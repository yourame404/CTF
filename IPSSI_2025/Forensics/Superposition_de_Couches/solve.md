# **Superposition de couches** (Forensics)

## **Enoncé**

**"Un fichier super important a été placé à l'intérieur d'une image Docker par un développeur.
Il pense que le packaging de son application et le fait que le fichier soit chiffré sont des éléments suffisants.
Prouvez lui le contraire en récupérant le fichier non chiffré."**

<br>

## **Solution**
Au niveau des blobs se trouvaient des archives POSIX. Les extraire nous donnait deux fichiers clés :
 ***flag*** qui contenait des données chiffrées et ***script.sh***:

```shell
#!/bin/bash

wget http://192.168.10.115/flag1

for A in {1..10}; do B=$(( $A + 1 )); openssl aes-256-cbc -a -pbkdf2 -in flag$A -out flag$B -k a; rm flag$A; done;

mv flag* flag
```

Ce script bash sert à chiffrer itérativement le fichier initial (*flag1*) en appliquant, à chaque itération, une transformation OpenSSL (AES-256-CBC en Base64 avec PBKDF2) pour produire des couches de chiffrement jusqu'à obtenir le fichier final **"flag"**.

<br>

Pour déchiffrer le contenu de flag:
```shell
#!/bin/bash

# Copie le fichier chiffré dans un fichier temporaire
cp fs_reconstruit/flag flag_enc

# Effectue 10 itérations de déchiffrement
for i in {1..10}; do
    openssl aes-256-cbc -a -pbkdf2 -d -in flag_enc -out flag_tmp -k a

    mv flag_tmp flag_enc
done

echo "Le flag déchiffré est :"
cat flag_enc
```

**Résultat :** `Le flag déchiffré est : Flag : **L35L4Y3R5D3D0CK3RC357QU37QU3CH053**`
