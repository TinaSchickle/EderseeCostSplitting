# Edersee Kasse 🌞🏖️

Eine kleine, sommerliche Splitwise-Alternative für unseren Edersee-Trip.
Trägt Ausgaben tabellarisch ein und rechnet automatisch aus, **wer wem was schuldet**
(meistens: alle → Tina, weil Tina die meisten Kosten auslegt).

## Features
- Schöne, freundliche Sommer-Optik 🌊
- Alles direkt in der Tabelle editierbar (wie eine Mini-Kalkulationstabelle):
  Datum, Beschreibung, Betrag, Gezahlt von, Aufteilungstyp, und eine Spalte pro Person
- Datum lässt sich pro Zeile mit **‹ ›** tageweise ändern
- Zwei Aufteilungstypen:
  - **Alle gleich** – Betrag wird automatisch auf alle 10 Personen verteilt (Personen-Spalten gesperrt)
  - **Individuell** – Betrag pro Person selbst eintragen; die Summe muss dem Betrag
    entsprechen, sonst bleibt die Zeile rot markiert (⚠️)
- Neue Zeile über den **➕ Neue Ausgabe**-Button unter der Tabelle
- Unterkunft (856 €) ist vorbefüllt: Paare teilen sich ein Zimmer, Person 1 zahlt voll,
  Person 2 nur 50 % (halber Anteil: **Lisa** mit Denis, **Natalia +1** mit Natalia) –
  als „Individuell"-Zeile mit passend vorausgefüllten Beträgen
- Abrechnung am Ende der Tabelle: **wer schuldet Tina was** (bzw. umgekehrt)
- **Live-Sync über Supabase** (Realtime): Änderungen erscheinen sofort auf allen
  Handys. Zusätzlich „🔄 Aktualisieren"-Button und Auto-Refresh beim Öffnen der App.
- **Als App installierbar** (Home-Bildschirm / „Zum Startbildschirm hinzufügen").
- **Zweiter Tab „🛒 Shoppingliste"**: Blöcke anlegen (z. B. „Ragout") und darunter
  die dafür benötigten Zutaten eintragen. Block „Sonstiges" ist fest und lässt
  sich nicht löschen, eigene Blöcke schon (inkl. aller Einträge).

## Personen
Jacob, Andy, Ralf, Denis, Lisa, Natalia, Natalia +1, Thorsten, Tina, Abel

## Supabase
Die App ist bereits mit dem (gemeinsamen) Supabase-Projekt verbunden – dieselbe
Instanz wie kitchenMagic. URL + öffentlicher Key stehen oben im `<script>` in
`index.html`.

**Einmalig** muss die Tabelle angelegt werden: in [supabase.com](https://supabase.com)
das Projekt öffnen → **SQL Editor** → dieses SQL ausführen:

```sql
create table if not exists public.expenses (
  id          uuid primary key default gen_random_uuid(),
  created_at  timestamptz not null default now(),
  date        date not null,
  description text not null default '',
  amount      numeric not null default 0,
  paid_by     text not null default 'Tina',
  split_type  text not null default 'equal',   -- 'equal' oder 'individual'
  shares      jsonb not null default '{}'::jsonb  -- {"Jacob": 95.11, ...} Betrag pro Person
);

-- Row Level Security an, aber für diese kleine Trip-Kasse alles erlauben:
alter table public.expenses enable row level security;

create policy "edersee_all_read"   on public.expenses for select using (true);
create policy "edersee_all_insert" on public.expenses for insert with check (true);
create policy "edersee_all_update" on public.expenses for update using (true) with check (true);
create policy "edersee_all_delete" on public.expenses for delete using (true);

-- Live-Sync einschalten (Realtime):
alter publication supabase_realtime add table public.expenses;
```

Danach zeigt der Status oben in der App „live synchronisiert 🟢“ und alle Geräte
sehen sofort denselben Stand.

**Falls die Tabelle schon (mit altem Stand) existiert** und die App „Spalte 'shares'
fehlt“ meldet, reicht diese Migration statt `create table`:

```sql
alter table public.expenses add column if not exists shares jsonb not null default '{}'::jsonb;
alter table public.expenses alter column description set default '';
alter table public.expenses alter column amount set default 0;
alter publication supabase_realtime add table public.expenses;
```

**Für die Shoppingliste** zusätzlich einmalig dieses SQL ausführen:

```sql
create table if not exists public.shopping_blocks (
  id          uuid primary key default gen_random_uuid(),
  created_at  timestamptz not null default now(),
  name        text not null,
  is_fixed    boolean not null default false
);

create table if not exists public.shopping_items (
  id          uuid primary key default gen_random_uuid(),
  created_at  timestamptz not null default now(),
  block_id    uuid not null references public.shopping_blocks(id) on delete cascade,
  label       text not null
);

alter table public.shopping_blocks enable row level security;
alter table public.shopping_items enable row level security;

create policy "shopping_blocks_all" on public.shopping_blocks for all using (true) with check (true);
create policy "shopping_items_all" on public.shopping_items for all using (true) with check (true);

alter publication supabase_realtime add table public.shopping_blocks;
alter publication supabase_realtime add table public.shopping_items;
```

> Hinweis: Der öffentliche Key steht im Browser sichtbar. Für diese private
> Trip-Kasse ist das ok – die Daten sind für jeden mit dem Link les- und
> schreibbar, also nichts Sensibles eintragen.

## Online stellen (GitHub Pages)
1. Repo-Einstellungen → **Settings → Pages**
2. **Source: Deploy from a branch**, Branch: `main`, Ordner: `/ (root)` → Save
3. Nach ~1 Min. erreichbar unter `https://tinaschickle.github.io/EderseeCostSplitting/`
