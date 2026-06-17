# Schéma de base de données — Chrono Hotel

## Vue d'ensemble

9 tables au total : 7 tables métier + 1 table de jointure (`room_amenities`) + 1 table standalone (`faqs`).

```
users ──┬──< hotels ────< rooms ──┬──< bookings
        │                  │      │
        │                  │      └──< photos
        │                  │
        │                  └──< room_amenities >── amenities
        │
        └──< bookings
        │
        └──< favorites >── hotels

faqs  (standalone — page d'aide)
```

## Tables

### `users` — Authentification (Devise)

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | Identifiant |
| `email` | string | Email unique |
| `encrypted_password` | string | Mot de passe hashé (Devise) |
| `role` | integer | Enum : `client` (0) / `hotel_manager` (1) |
| `first_name` | string | Prénom |
| `last_name` | string | Nom |
| `reset_password_token` | string | Devise |
| `reset_password_sent_at` | datetime | Devise |
| `remember_created_at` | datetime | Devise |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

### `hotels` — Établissements

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `user_id` | bigint (FK) | → `users.id` (manager propriétaire) |
| `name` | string | Nom de l'hôtel |
| `description` | text | Description |
| `address` | string | Adresse |
| `city` | string | Ville |
| `country` | string | Pays |
| `star_rating` | integer | Nombre d'étoiles (1-5) |
| `phone` | string | Téléphone |
| `website` | string | Site web |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

### `rooms` — Chambres

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `hotel_id` | bigint (FK) | → `hotels.id` |
| `title` | string | Ex: "Suite Deluxe" |
| `description` | text | Description |
| `room_type` | string | Type (simple, double, suite...) |
| `capacity` | integer | Nombre de personnes |
| `original_price` | decimal(10,2) | Prix de base (barré) |
| `discounted_price` | decimal(10,2) | Prix après remise |
| `discount_percent` | integer | % appliqué (0-100) |
| `available_from` | date | Début de disponibilité |
| `available_until` | date | Fin de disponibilité |
| `is_available` | boolean | Disponible immédiatement |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

### `bookings` — Réservations

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `user_id` | bigint (FK) | → `users.id` (client) |
| `room_id` | bigint (FK) | → `rooms.id` |
| `check_in` | date | Date d'arrivée |
| `check_out` | date | Date de départ |
| `total_price` | decimal(10,2) | Prix total calculé |
| `status` | integer | Enum : `pending` (0) / `confirmed` (1) / `cancelled` (2) |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

### `photos` — Images des chambres

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `room_id` | bigint (FK) | → `rooms.id` |
| `image_url` | string | URL de l'image (Active Storage ou URL externe) |
| `position` | integer | Ordre d'affichage (carrousel) |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

### `favorites` — Favoris des clients

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `user_id` | bigint (FK) | → `users.id` |
| `hotel_id` | bigint (FK) | → `hotels.id` |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

**Index unique** : `(user_id, hotel_id)` — un user ne peut avoir qu'un seul favori par hôtel.

### `amenities` — Équipements disponibles

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `name` | string | Ex: "WiFi", "Petit-déjeuner", "Parking", "Climatisation" |
| `icon` | string | Nom de l'icône (Bootstrap Icons) |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

### `room_amenities` — Table de jointure (Room ↔ Amenity)

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `room_id` | bigint (FK) | → `rooms.id` |
| `amenity_id` | bigint (FK) | → `amenities.id` |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

**Index unique** : `(room_id, amenity_id)` — évite les doublons.

### `faqs` — Foire aux questions (page d'aide)

| Colonne | Type | Rôle |
|---|---|---|
| `id` | bigint (PK) | — |
| `question` | text | Question |
| `answer` | text | Réponse |
| `category` | string | Catégorie (voyageurs, hôtels, etc.) |
| `position` | integer | Ordre d'affichage |
| `created_at` | datetime | — |
| `updated_at` | datetime | — |

## Relations ActiveRecord (à implémenter)

```ruby
class User < ApplicationRecord
  has_many :hotels, dependent: :destroy
  has_many :bookings, dependent: :destroy
  has_many :favorites, dependent: :destroy
  has_many :favorite_hotels, through: :favorites, source: :hotel
end

class Hotel < ApplicationRecord
  belongs_to :user
  has_many :rooms, dependent: :destroy
  has_many :favorites, dependent: :destroy
  has_many :favorited_by, through: :favorites, source: :user
end

class Room < ApplicationRecord
  belongs_to :hotel
  has_many :bookings, dependent: :destroy
  has_many :photos, dependent: :destroy
  has_many :room_amenities, dependent: :destroy
  has_many :amenities, through: :room_amenities
end

class Booking < ApplicationRecord
  belongs_to :user
  belongs_to :room
end

class Photo < ApplicationRecord
  belongs_to :room
end

class Favorite < ApplicationRecord
  belongs_to :user
  belongs_to :hotel
  validates :user_id, uniqueness: { scope: :hotel_id }
end

class Amenity < ApplicationRecord
  has_many :room_amenities, dependent: :destroy
  has_many :rooms, through: :room_amenities
end

class RoomAmenity < ApplicationRecord
  belongs_to :room
  belongs_to :amenity
  validates :room_id, uniqueness: { scope: :amenity_id }
end

class Faq < ApplicationRecord
  # standalone — pas de relation
end
```

## Notes

- **Devise** génère automatiquement plusieurs colonnes (`reset_password_token`, etc.) lors de l'installation. Le schéma ci-dessus liste les principales.
- **`role`** est stocké en `integer` avec enum Rails (plus performant et sûr qu'un string).
- **`status`** des bookings utilise aussi un enum integer.
- **Active Storage** pourrait remplacer `photos.image_url` si on préfère gérer l'upload directement dans Rails (recommandé pour la certif).
- Typos corrigées par rapport au schéma initial : `adress` → `address`, `categorie` → `category`.
