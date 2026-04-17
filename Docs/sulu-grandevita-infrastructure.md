# Tehnična Dokumentacija: Sulu CMS & Sylius Integracija (Grandevita)

Ta dokument vsebuje podrobno analizo infrastrukture Sulu CMS projekta in nastavitve zaheadless integracijo s Sylius trgovino.

## 1. Sistemske Informacije

*   **Sulu CMS Verzija:** `~3.0.5`
*   **Symfony Ogrodje:** `^7.4`
*   **PHP Verzija:** `^8.2` (Docker uporablja `8.3-fixuid-xdebug-alpine`)
*   **Ključne Knjižnice:**
    *   `sulu/headless-bundle`: `^3.0@dev` (Glavni bundle za API dostop)
    *   `sulu/sulu`: `~3.0.5`
    *   `doctrine/doctrine-bundle`: `^2.13`
    *   `friendsofsymfony/http-cache-bundle`: `^3.0`
    *   `cmsig/seal-loupe-adapter`: `^0.12.2` (Iskalnik)

## 2. Docker & Lokalni Setup

### Servisi
Projekt teče na naslednjih Docker servisih:
*   **sulu-php:** PHP 8.3 FPM (Image: `ghcr.io/sylius/sylius-php:8.3-fixuid-xdebug-alpine`)
*   **sulu-nginx:** Spletni strežnik (Image: `nginx:alpine`)
*   **sulu-mysql:** Baza podatkov (Image: `mysql:8.4`)

### Omrežje in Porti
*   **Interno omrežje:** `grandevita_sulu_network`
*   **Dostopnost (Host):** Sulu je dosegljiv na `http://localhost:81`
*   **PHP-FPM Port:** `9000` (interno znotraj `sulu-php`)
*   **Nginx Port:** `80` (interno), mapirano na `81` na hostu.
*   **MySQL:** `3306` (interno)

### Ključne Nastavitve
*   **DATABASE_URL:** `mysql://root@sulu-mysql/sulu_dev?serverVersion=8.4`
*   **APP_ENV:** `dev`
*   **APP_URL:** Ni eksplicitno nastavljen v `docker-compose.yml`, vendar Nginx konfigurira `SERVER_PORT` in `HTTP_X_FORWARDED_PORT` na `81` za pravilno generiranje URL-jev.

## 3. Headless & API Konfiguracija

### Mehanizem serviranja podatkov
Sulu servira podatke primarno preko **SuluHeadlessBundle**.
*   **Konfiguracija poti:** Registrirana v `config/routes/sulu_headless_website.yaml` preko `@SuluHeadlessBundle/Resources/config/routing_website.yml`.
*   **GraphQL:** Ni zaslediti namenske GraphQL knjižnice (npr. `tizianogonzalez/sulu-graphql`), zato se uporablja REST API, ki ga nudi HeadlessBundle.
*   **Kontrolerji:** `articleholder` stran uporablja `Sulu\Bundle\HeadlessBundle\Controller\HeadlessWebsiteController::indexAction`.

## 4. Vsebina Struktura (Content Architecture)

### Webspaces
*   **Datoteka:** `config/webspaces/website.xml`
*   **Ključ (Key):** `website`
*   **Jeziki (Locales):** `en` (privzeto), `sl`, `hr`, `bs`.

### Tipi strani (Templates)
Nahajajo se v `config/templates/pages/`:
*   **homepage:** Vstopna stran.
*   **default:** Osnovna vsebinska stran (naslov, rlp, članek).
*   **articleholder:** Namenska stran za prikaz seznamov (Smart Content), uporablja Headless kontroler.

### Snippets
Nahajajo se v `config/templates/snippets/`:
*   **default:** Osnovni snippet z naslovom in opisom.

### Sylius Povezava
Trenutno v XML konfiguracijah **ni** definiranih namenskih polj za neposredno povezavo s Sylius produkti (npr. `product_code`). Priporočljivo je dodati polje tipa `text_line` ali namenski `selection` v prihodnje.

## 5. Media & Storage

*   **Shranjevanje:** Slike in datoteke se shranjujejo na **lokalnem volumnu**.
*   **Adapter:** Flysystem uporablja `local` adapter.
*   **Pot:** `%kernel.project_dir%/var/storage/default` (znotraj Dockerja `/srv/sulu/var/storage/default`).
*   **Formatiranje:** Sulu Media format manager je nastavljen na 90% kvaliteto za JPEG, WebP in AVIF.

## 6. Admin & Varnost

### API Dostop
*   **Admin dostop:** Varovan s standardno Symfony/Sulu formo (`json_login`).
*   **Headless API:** Glede na `security.yaml` in trenutne nastavitve `SuluHeadlessBundle`, je API dostopen javno (brez JWT ali API ključev), kar je pogosto za headless Sulu, kjer se avtentikacija ureja na ravni infrastrukture ali proxy-ja, če je potrebno.
*   **2FA:** Omogočena je dvofaktorska avtentikacija (SchebTwoFactorBundle) za admin uporabnike.
