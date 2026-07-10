# 🧠 ARSENAL — Stabilisation de l’intégration Fujitsu Airstage

## Version
v1.0 — Intégration durcie et maîtrisée

---

## 🎯 Objectif

Assurer la stabilité opérationnelle de l’intégration communautaire
**Fujitsu Airstage** dans l’écosystème Arsenal, malgré :

- les limites de l’API Fujitsu,
- les faiblesses de la librairie `pyairstage`,
- l’absence de maintenance amont régulière.

L’objectif est de garantir :

- un comportement prévisible,
- l’absence d’exceptions non gérées,
- une intégration durable.

---

## 🧩 Constat initial

L’intégration reposait sur des hypothèses invalides :

- disponibilité permanente des données,
- absence de valeurs `None`,
- cohérence immédiate après démarrage.

En pratique :

- données absentes fréquentes,
- valeurs non initialisées,
- latence réseau.

Conséquences :

- `TypeError`
- `KeyError`
- `Task exception was never retrieved`
- instabilité du `DataUpdateCoordinator`

➡️ Non-conformité au contrat Home Assistant.

---

## 🎯 Principe directeur

> Toute propriété exposée à Home Assistant
> doit retourner une valeur valide ou `None`,
> et ne doit jamais lever d’exception.

---

## 🧱 Stratégie de stabilisation

### 1. Identification des zones critiques

Fichiers concernés :

| Domaine | Fichier |
|---------|----------|
| Climate | `climate.py` |
| Sensor  | `sensor.py` |
| Switch  | `switch.py` |

Méthodes sensibles :

- `current_temperature`
- `target_temperature`
- `min_temp`
- `max_temp`
- `hvac_mode`
- `fan_mode`
- `native_value`
- `is_on`

---

### 2. Analyse des patterns d’erreur

Schémas récurrents :

- `int(None)`
- accès direct à `_lastGoodValue[...]`

Signatures :

- `TypeError: int() argument must be ... NoneType`
- `KeyError: <ACParameter.*>`

➡️ Accès non défensifs à des données absentes.

---

### 3. Application d’un modèle défensif

Modèle générique :

```python
try:
    value = appel_fujitsu()
except (TypeError, ValueError, KeyError):
    return None
```

Puis :

- validation explicite,
- fallback cohérent.

---

## 🔧 Implémentation

---

### ❄️ climate.py

Sécurisation complète des propriétés :

- `current_temperature`
- `target_temperature`
- `min_temp`
- `max_temp`
- `hvac_mode`
- `fan_mode`

Mesures :

- encapsulation systématique des accès,
- fallback métier conservé,
- mapping sécurisé.

Résultat :

- aucune exception levée.

---

### 🌡️ sensor.py

Méthode `native_value` sécurisée pour :

- `INDOOR_TEMPERATURE`
- `OUTDOOR_TEMPERATURE`

Traitement :

- gestion des `KeyError`,
- prévention de `Decimal(None)`.

Retour systématique de `None` si donnée absente.

---

### 🔘 switch.py

Sécurisation de tous les `is_on` :

- état principal,
- energy save fan,
- quiet fan,
- autres options.

Modèle uniforme :

- encapsulation,
- validation,
- retour `True / False / None`.

---

## 🌐 Gestion des indisponibilités réseau

### Erreurs typiques

```
Timeout while connecting to device
Config entry setup failed
```

Origine :

- veille matérielle,
- latence réseau,
- réveil lent du module Wi-Fi.

Nature :

- non logicielle,
- non maîtrisable.

Décision :

> Acceptation comme bruit opérationnel.

Aucune tentative de contournement logiciel.

---

## 🔄 Politique de maintenance

### Intégration figée

- Pas de mise à jour automatique
- Version locale validée
- Correctifs internes conservés

---

### Procédure en cas d’évolution amont

1. Lecture du changelog
2. Comparaison des fichiers clés
3. Vérification des correctifs
4. Réintégration si nécessaire

---

## 📋 Documentation locale

Fichier recommandé :

```
custom_components/fujitsu_airstage/PATCHES_ARSENAL.md
```

Contenu minimal :

```
- climate.py : durcissement accesseurs
- sensor.py  : sécurisation températures
- switch.py  : sécurisation états
```

---

## 📊 Critères de conformité

L’intégration est considérée comme stabilisée si :

| Critère           | État              |
| ----------------- | ----------------- |
| Exceptions Python | Absentes          |
| Crash coordinator | Absent            |
| Erreurs critiques | Absentes          |
| Erreurs restantes | Réseau uniquement |
| Comportement      | Prévisible        |

---

## 🏁 Résultat

Après stabilisation :

- disparition des erreurs bloquantes,
- stabilité sur la durée,
- maintenance minimale,
- confiance opérationnelle restaurée.

L’intégration devient une composante fiable du système Arsenal.

---

## 🧭 Positionnement méthodologique

Principe appliqué :

> Corriger les défaillances logicielles déterministes.
> Accepter les aléas matériels non maîtrisables.

---

## 📌 Conclusion

L’intégration Fujitsu Airstage est désormais :

- conforme au modèle Home Assistant,
- défensive par conception,
- indépendante des fragilités amont,
- durablement exploitable.

Elle est intégrée dans l’architecture Arsenal comme une brique maîtrisée.

---

Fin du document.
