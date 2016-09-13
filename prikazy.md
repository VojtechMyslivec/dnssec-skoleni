# Skoleni DNSSEC

## Generovani klicu

Pro testovaci ucely pridat parametr:

    -r /dev/urandom

**KSK**

    dnssec-keygen -a RSASHA256 -b 2048 -n ZONE -f KSK z110.skoleni.

**ZSK**

    dnssec-keygen -a RSASHA256 -b 1024 -n ZONE z110.skoleni.

