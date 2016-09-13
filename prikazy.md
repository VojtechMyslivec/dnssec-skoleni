# Skoleni DNSSEC

## Generovani klicu

Pro testovaci ucely pridat parametr:

    -r /dev/urandom

**KSK**

    dnssec-keygen -a RSASHA256 -b 2048 -n ZONE -f KSK z110.skoleni.

**ZSK**

    dnssec-keygen -a RSASHA256 -b 1024 -n ZONE z110.skoleni.

## Podepsani zony

    dnssec-signzone -o z110.skoleni. -N UNIXTIME -k keys/Kz110.skoleni.+008+23470.key zones/z110.skoleni.conf keys/Kz110.skoleni.+008+28854.key

## NSEC3

"Bezpecne" generovani soli

    head -c 16 /dev/random | xxd -p

Podepsani s NSEC3 zaznamy

    dnssec-signzone -o z110.skoleni. -N UNIXTIME -3 8c9a745f9bcf5a5e19ed98dc1a3bd885 -H 10 -k keys/Kz110.skoleni.+008+23470.key zones/z110.skoleni.conf keys/Kz110.skoleni.+008+28854.key

