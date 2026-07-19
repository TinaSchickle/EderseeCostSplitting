# Edersee Kasse 🌞🏖️

Eine kleine, sommerliche Splitwise-Alternative für unseren Edersee-Trip.
Trägt Ausgaben tabellarisch ein und rechnet automatisch aus, **wer wem was schuldet**
(meistens: alle → Tina, weil Tina die meisten Kosten auslegt).

## Features
- Schöne, freundliche Sommer-Optik 🌊
- Ausgaben-Tabelle mit Datum, Beschreibung, Betrag, Zahler und Aufteilung
- Datum ist vorausgefüllt (heute) und lässt sich mit **‹ ›** tageweise ändern
- Zwei Aufteilungen:
  - **Gleich** – auf alle 10 Personen zu gleichen Teilen
  - **Unterkunft** – Paare teilen sich ein Zimmer: Person 1 zahlt voll, Person 2 nur 50 %
    (halber Anteil: **Lisa** mit Denis, **Natalia +1** mit Natalia)
- Automatische Abrechnung mit minimaler Anzahl Zahlungen
- Speicherung in **Supabase** (alle sehen dieselben Daten). Ohne Supabase-Konfiguration
  speichert die App lokal im Browser (Fallback zum Ausprobieren).

## Personen
Jacob, Andy, Ralf, Denis, Lisa, Natalia, Natalia +1, Thorsten, Tina, Abel

## Supabase anbinden
1. In [supabase.com](https://supabase.com) ein Projekt öffnen → **SQL Editor** → dieses SQL ausführen:

```sql
create table if not exists public.expenses (
  id          uuid primary key default gen_random_uuid(),
  created_at  timestamptz not null default now(),
  date        date not null,
  description text not null,
  amount      numeric not null,
  paid_by     text not null default 'Tina',
  split_type  text not null default 'equal'   -- 'equal' oder 'accommodation'
);

-- Row Level Security an, aber für diese kleine Trip-Kasse alles erlauben:
alter table public.expenses enable row level security;

create policy "edersee_all_read"   on public.expenses for select using (true);
create policy "edersee_all_insert" on public.expenses for insert with check (true);
create policy "edersee_all_update" on public.expenses for update using (true) with check (true);
create policy "edersee_all_delete" on public.expenses for delete using (true);
```

2. **Project Settings → API** → `Project URL` und `anon public` key kopieren.
3. In `index.html` oben im `<script>` die beiden Zeilen ersetzen:

```js
const SUPABASE_URL = "https://xxxx.supabase.co";
const SUPABASE_ANON_KEY = "eyJhbGciOi...";
```

4. Speichern, committen, pushen. Fertig – der Status oben in der App zeigt dann
   „mit Supabase verbunden“ 🟢.

> Hinweis: Der `anon`-Key ist öffentlich sichtbar (steht im Browser). Für diese
> private Trip-Kasse ist das ok. Die Daten sind für jeden mit dem Link les- und
> schreibbar – bitte nicht für Sensibles nutzen.

## Online stellen (GitHub Pages)
1. Repo-Einstellungen → **Settings → Pages**
2. **Source: Deploy from a branch**, Branch: `main`, Ordner: `/ (root)` → Save
3. Nach ~1 Min. erreichbar unter `https://tinaschickle.github.io/EderseeCostSplitting/`
