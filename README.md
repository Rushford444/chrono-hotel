# Chrono Hotel

Plateforme de réservation de chambres d'hôtel à prix réduit, dernière minute.
Les hôtels publient leurs chambres disponibles à 24-48h avec une remise,
les clients peuvent réserver à prix cassé.

Projet de certification — Le Wagon (Concepteur Développeur d'Application Web, Niveau 6).

## Stack

- Ruby on Rails 8.x
- PostgreSQL
- Bootstrap 5
- SCSS
- JavaScript (esbuild)
- Devise (authentification)

## Pré-requis

- Ruby 3.3+
- PostgreSQL 14+
- Node.js 18+
- Yarn

## Installation

```bash
bundle install
rails db:create db:migrate
rails s
```

L'application est disponible sur http://localhost:3000.

## Fonctionnalités (MVP)

- [x] Authentification (client / hôtelier)
- [ ] Recherche de chambres avec filtres
- [ ] Page détail d'une chambre
- [ ] Réservation
- [ ] Dashboard hôtelier (gestion des chambres)

## Auteur

Luis — [@Rushford444](https://github.com/Rushford444)

## Licence

Distribué sous licence MIT. Voir `LICENSE`.
