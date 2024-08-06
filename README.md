# Proxy-web

Repo importé de SVN pour gestion du proxy-web 'historique' sur une seule patte.

Voir https://gitlab.si.u75.fr/infra/ansible/projects/haproxy-conf/ pour la config du nouveau proxy-web sur deux pattes.

# HAproxy

## 1. stocker les certificats

Les stocker dans `/usr/local/etc/ssl/certs/haproxy` ; les fichiers doivent avoir les permissions suivantes :
- droits 0640
- propriétaire `root`
- groupe `ssl-cert`

Les fichiers doivent se nommer : `cert+chain+key_<name>_<numero>.crt` où `<name>` est le FQDN, les points remplacés par des soulignés et où `<numero>` est le numéro du certificat dans l'AC TERENA (_ie_. DigiCert en 2019).

```bash
n=1234 ; echo -n "cert+chain+key_" ; hostname | sed -e "s/\./_/g; s/$/_$n.crt/"
```

Les fichiers doivent contenir, concaténés dans cet ordre :
* le certificat
* la « chaîne » _ie_. le certificat : `Subject: C = NL, ST = Noord-Holland, L = Amsterdam, O = TERENA, CN = TERENA SSL CA 3`
* la clef (privée en clair)

## 2. générer les certificats

```bash
$ hostname=plop.u-paris.fr
$ openssl req -new -newkey rsa:2048 -nodes -out ${hostname}.csr -keyout ${hostname}.key \
?    -subj "/C=FR/ST=/L=Paris/O=UNIVERSITE de PARIS/OU=DSIN/CN=${hostname}" \
?    -config openssl_csr.cnf
```

avec dans le fichier `openssl_csr.cnf` :

```
[ req ]
req_extensions     = req_ext
distinguished_name = req_distinguished_name
[ req_distinguished_name ]
countryName                = FR
stateOrProvinceName        =
localityName               = Paris
organizationName           = UNIVERSITE de PARIS
organizationalUnitName     = DSIN
commonName                 = plop.u-paris.fr
[ req_ext ]
subjectAltName             = @alt_names
[ alt_names ]
DNS.1 = plop1.u-paris.fr
DNS.2 = plop2.u-paris.fr
DNS.3 = plop3.u-paris.fr
```

## 3. déployer les certificats

FIXME
