# PhotoAlbum

Django fotoalbum alkalmazás. Fotók feltöltése, szervezése, megtekintése. Render-en fut IaC alapon.

## Projekt Struktúra

```
photoalbum/          # Django projekt config
  ├── settings.py    # env változókat olvassa
  ├── urls.py
  ├── wsgi.py
  
photos/              # App
  ├── models.py      # Photo modell
  ├── views.py       # fotó lista, feltöltés, törlés
  ├── forms.py
  ├── urls.py
  └── migrations/    # adatbázis séma

templates/           # HTML-ek
  ├── base.html      # navbar, footer
  ├── photos/        # fotó lapok
  └── registration/  # login, register

static/css/          # CSS stílusok
.github/workflows/   # CI pipeline (GitHub Actions)
render.yaml          # IaC Blueprint (Render)
requirements.txt     # Python csomagok
manage.py            # Django parancs eszköz
```

## Funkciók

- Fotó feltöltés, törlés
- Fotó galériás nézet
- Keresés, szortírozás
- Felhasználó regisztráció, bejelentkezés
- Hozzáférés kontroll

---

## GitHub Actions CI

A `.github/workflows/ci.yml` futtatja az ellenőrzéseket push után a main branchre.

Lépések:
1. Kód letöltése
2. Python 3.12 setup
3. Python csomagok telepítése
4. Django system check
5. Tesztek futtatása

Ha bármelyik lépés bukik, a build piros. Sikeres build után Render deployol.

---

## Render Blueprint (IaC)

A `render.yaml` definiálja az infrastruktúrát: PostgreSQL adatbázis, Python web service, build és start parancsok.

**autoDeployTrigger: checksPass** - sikeres GitHub Actions CI után deployol az alkalmazás.

Build lépések:
- Csomagok telepítése
- Adatbázis séma frissítés (migrate)
- Statikus fájlok gyűjtése

Start: Gunicorn alkalmazásszerver, `/healthz/` health check végpont.

Env változók: SECRET_KEY (auto), DEBUG=False, DATABASE_URL, Cloudinary config, stb.

---

## Deploy Folyamata

1. Push a main branchre
2. GitHub Actions futtatja az ellenőrzéseket
3. Sikeres check után Render automatikus deploy
4. Build: install, migrate, collectstatic
5. Start: Gunicorn elindul
6. Health check: app él

Adatbázis nem törlődik deploy közben, csak a séma frissül. Fotók megmaradnak.

---

## Lokális Futtatás

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# .env-ben: SECRET_KEY, DEBUG=True
python manage.py migrate
python manage.py runserver
```

http://127.0.0.1:8000

---

## Új Migráció

Ha modellt módosítasz:

```bash
python manage.py makemigrations
python manage.py migrate
git add photos/migrations/
git commit -m "..."
git push origin main
```

Render automatikusan futtatja a migrációt a deploynál.

---

## Hasznos

```bash
python manage.py check      # Django ellenőrzés
python manage.py test       # Tesztek
python manage.py shell      # Django shell
python manage.py createsuperuser  # Admin felület
```

---

## Cloudinary

Fotók a Cloudinary CDN-ben tárolódnak (nem lokálisan).

Lokálisan: `USE_CLOUDINARY=False`, fotók a `media/` mappában.
Producton: `USE_CLOUDINARY=True`, fotók Cloudinary-ben.
