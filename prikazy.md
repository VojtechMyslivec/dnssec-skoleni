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

Podepsani s `NSEC3` zaznamy

    dnssec-signzone -o z110.skoleni. -N UNIXTIME -3 8c9a745f9bcf5a5e19ed98dc1a3bd885 -H 10 -k keys/Kz110.skoleni.+008+23470.key zones/z110.skoleni.conf keys/Kz110.skoleni.+008+28854.key

## Vymena ZSK

### Faze 0

  * Vypnuti automatickeho podepisovni demonem *bind*

### Faze 1

  * Vygenerovani noveho *ZSK* klice
  * Pridani noveho klice jako `$INCLUDE` do zonoveho souboru
  * Podepsani zony starym klicem
  * Reload demona

Kontrola

  * Vystup dnssec-signzone

```
Algorithm: RSASHA256: KSKs: 1 active, 0 stand-by, 0 revoked
                      ZSKs: 1 active, 1 stand-by, 0 revoked
```

  * `dig z110.skoleni. DNSKEY +dnssec`
    * Vrati 3 `DNSKEY` a 2 `RRSIG` zaznamy
    * `NOERROR`, `ad` flag

### Faze 2

  * Podepsani zony novym klicem
  * Reload demona

Kontrola

  * Viz Faze 1 - zmeni se jen *ID* u `RRSIG`

### Faze 3

  * Pockat na vyprseni TTL `RRSIG` zaznamu
  * Odstranit stary *ZSK* klic
  * Podepsat (novym) *ZSK* klicem
  * Reload demona

Kontrola

  * Vystup dnssec-signzone

```
Algorithm: RSASHA256: KSKs: 1 active, 0 stand-by, 0 revoked
                      ZSKs: 1 active, 0 stand-by, 0 revoked

```

  * `dig z110.skoleni. DNSKEY +dnssec`
    * Vrati 2 `DNSKEY` a 2 `RRSIG` zaznamy
    * `NOERROR`, `ad` flag

## Vymena KSK

### Faze 0

  * dtto

### Faze 1

  * Vygenerovani noveho *KSK* klice
  * Pridani noveho klice jako `$INCLUDE` do zonoveho souboru
  * Podepsani zony obema klici (parametr `-k KSK.key` dvakrat)
  * Reload demona
  * Poslani noveho *KSK* nadrazene zone

Kontrola

  * Vystup dnssec-signzone

```
Algorithm: RSASHA256: KSKs: 2 active, 0 stand-by, 0 revoked
                      ZSKs: 1 active, 0 stand-by, 0 revoked
```

  * `dig z110.skoleni. DNSKEY +dnssec`
    * Vrati 3 `DNSKEY` a 3 `RRSIG` zaznamy
    * `NOERROR`, `ad` flag

### Faze 2

  * Vyckani na nasazeni noveho *KSK* v nadrazene zone

Kontrola

  * `dig z110.skoleni. DS +dnssec @pc113`
    * Vraci ID noveho *KSK* klice

