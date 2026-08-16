# Dell Latitude E6520 — Wi-Fi hard blocked sous Xubuntu

## Contexte

PC : Dell Latitude E6520
OS : Xubuntu
Carte Wi-Fi : Intel Centrino Advanced-N 6205 AGN

Objectif :
Réactiver le Wi-Fi afin de pouvoir utiliser le PC comme serveur local
pour les projets backend/web et permettre à d'autres appareils du réseau
d'accéder aux endpoints du serveur.

---

# 1. Symptôme

Dans Xubuntu, le Wi-Fi semblait absent ou impossible à activer.

Le système possédait pourtant une interface réseau Wi-Fi.

---

# 2. Vérifier les interfaces réseau

Commande :

    ip link

Résultat important :

    lo
    eno1
    wlp3s0

Signification :

    lo      → interface loopback
    eno1    → Ethernet
    wlp3s0  → interface Wi-Fi

Conclusion :

    Linux possède bien une interface Wi-Fi.
    Le problème n'est donc pas simplement "Linux ne voit aucun Wi-Fi".

---

# 3. Vérifier le matériel PCI

Commande :

    lspci | grep -i -E 'network|wireless|wifi'

Résultat important :

    Network controller: Intel Corporation Centrino Advanced-N 6205 ...

Conclusion :

    La carte Wi-Fi physique est détectée par Linux.

Remarque :
Le modèle réellement détecté par le pilote est la
Intel Centrino Advanced-N 6205 AGN.

---

# 4. Vérifier le blocage RF-KILL

Commande :

    rfkill list

Résultat :

    Dell WiFi
        Soft blocked: no
        Hard blocked: yes

    Dell Bluetooth
        Soft blocked: no
        Hard blocked: yes

    PHY0
        Soft blocked: no
        Hard blocked: yes

Conclusion :

    Le Wi-Fi est HARD BLOCKED.

Un "soft block" peut normalement être retiré logiciellement.
Un "hard block" indique que le système reçoit une indication
de désactivation provenant du matériel/firmware.

---

# 5. Tester le déblocage logiciel

Commande :

    rfkill unblock all

Puis :

    rfkill list

Résultat :

    Hard blocked: yes

Le blocage reste présent.

Conclusion :

    Le problème n'est pas un simple blocage logiciel rfkill.

---

# 6. Examiner les contrôleurs rfkill

Commande :

    ls /sys/class/rfkill/

Résultat :

    rfkill0
    rfkill1
    rfkill2

Pour examiner les informations de chaque contrôleur :

    for x in /sys/class/rfkill/*; do
        echo "=== $x ==="
        cat "$x/type"
        cat "$x/hard"
        cat "$x/soft"
    done

Résultat observé :

    rfkill0
    wlan
    1
    0

    rfkill1
    bluetooth
    1
    0

    rfkill2
    wlan
    1
    0

Interprétation :

    type
        wlan / bluetooth

    hard = 1
        blocage matériel/firmware

    soft = 0
        pas de blocage logiciel

Conclusion :

    Wi-Fi et Bluetooth sont tous deux déclarés hard-blocked.

---

# 7. Vérifier les messages du noyau

Commande :

    dmesg | grep -i -E 'rfkill|wireless|wifi|bluetooth|wlan'

Résultat important :

    iwlwifi ... Detected Intel(R) Centrino(R) Advanced-N 6205 AGN

    iwlwifi ... loaded firmware version 18.168.6.1 ...

    iwlwifi ... reporting RF_KILL (radio disabled)

    iwlwifi ... RF_KILL bit toggled to disable radio.

    iwlwifi ... wlp3s0: renamed from wlan0

Interprétation :

    Carte Wi-Fi détectée                 → OK
    Pilote iwlwifi chargé                → OK
    Firmware chargé                      → OK
    Interface wlp3s0 créée               → OK
    Radio désactivée par RF_KILL         → PROBLÈME

Conclusion :

    Le problème ne vient probablement pas d'un pilote manquant.

    Le pilote fonctionne mais reçoit l'information que la radio
    Wi-Fi doit être désactivée.

---

# 8. Vérification du modèle

Modèle :

    Dell Latitude E6520

La documentation du modèle indique la présence de paramètres
BIOS liés au contrôle des périphériques sans fil.

---

# 9. Vérification du BIOS

Entrer dans le BIOS avec :

    F12

Deux sections importantes ont été trouvées :

    Wireless Device Enable
    Wireless Switch

## Wireless Device Enable

Cette section permet d'autoriser les périphériques sans fil.

Configuration fonctionnelle :

    WLAN       Enabled
    Bluetooth  Enabled

Lorsque ces options ont été désactivées, les périphériques Wi-Fi
et Bluetooth ont disparu de Xubuntu.

Donc ces options doivent rester activées.

---

# 10. Wireless Switch

Cette section contient une indication du type :

    Device that can be controlled by wireless switch

Initialement, WLAN et Bluetooth étaient sélectionnés.

Configuration finale :

    WLAN       Disabled / décoché
    Bluetooth  Disabled / décoché

Cela signifie que le Wireless Switch ne contrôle plus ces
périphériques.

Après sauvegarde du BIOS et redémarrage :

    Wi-Fi       → fonctionnel
    Bluetooth   → fonctionnel
    Réseaux     → visibles
    Interface   → utilisable

Conclusion :

    Le mécanisme Wireless Switch du BIOS était à l'origine du
    RF_KILL / hard block observé sous Linux.

---

# 11. Configuration finale

BIOS :

    Wireless Device Enable
        WLAN       Enabled
        Bluetooth  Enabled

    Wireless Switch
        WLAN       Disabled
        Bluetooth  Disabled

Xubuntu :

    Wi-Fi fonctionnel
    Bluetooth fonctionnel
    wlp3s0 disponible

---

# 12. Chaîne de diagnostic

Le problème a finalement été localisé en suivant cette chaîne :

    Carte Wi-Fi
        ↓
    lspci
        ↓
    Carte détectée
        ↓
    iwlwifi
        ↓
    Firmware chargé
        ↓
    RF_KILL
        ↓
    rfkill
        ↓
    Hard blocked
        ↓
    BIOS Wireless Switch
        ↓
    Modification de la configuration
        ↓
    Wi-Fi fonctionnel

---

# 13. Commandes utiles pour un futur diagnostic

Vérifier les interfaces :

    ip link

Vérifier le matériel :

    lspci | grep -i -E 'network|wireless|wifi'

Vérifier le blocage :

    rfkill list

Tenter un déblocage logiciel :

    rfkill unblock all

Voir les contrôleurs rfkill :

    ls /sys/class/rfkill/

Voir les informations détaillées :

    for x in /sys/class/rfkill/*; do
        echo "=== $x ==="
        cat "$x/type"
        cat "$x/hard"
        cat "$x/soft"
    done

Voir les messages du noyau :

    dmesg | grep -i -E 'rfkill|wireless|wifi|bluetooth|wlan'

---

# Résumé

Problème :
    Wi-Fi impossible à activer sous Xubuntu.

Cause :
    RF_KILL indiquait une radio désactivée et rfkill affichait
    "Hard blocked: yes".

Le pilote et le firmware étaient pourtant correctement chargés.

Solution :
    BIOS → Wireless Device Enable :
        WLAN + Bluetooth activés

    BIOS → Wireless Switch :
        WLAN + Bluetooth retirés des périphériques contrôlables.

Résultat :
    Wi-Fi et Bluetooth fonctionnels.

À retenir :
    Si une carte Wi-Fi est détectée par lspci, que le pilote
    iwlwifi est chargé, mais que rfkill indique "Hard blocked: yes",
    vérifier en priorité les mécanismes matériels/firmware et les
    paramètres BIOS avant de réinstaller le pilote.