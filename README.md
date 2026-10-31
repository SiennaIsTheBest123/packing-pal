# Packing Pal

Packing Pal is a virtual holiday planner. You say when and where you are travelling and Packing Pal creates a reliable and easy itinerary for your trip. In addition, Packing Pal also tells you the weather of your destination and gives you activity suggestions to help you plan your journey.

It's a single `index.html` file with no build step. It's hosted on Vercel and uses Supabase for logins and saved trips.

## Features

**Log in and My trips**
- The first screen lets people log in or sign up with an email and password. New users confirm their email before logging in.
- After logging in, **My trips** shows every saved trip with its destination, dates, who it's for and a countdown. Open a trip, start a new one, or delete one.
- The **My trips** and **Log out** buttons sit at the top of the app.
- Each person only ever sees their own trips.

**Saves automatically**
- Every change is saved about a second after you make it: ticks, quantities, outfits, plans, saved ideas and the app colour.
- A note at the top shows "Saving…" and then "All changes saved". If you're offline, it says so and saves when you're back online.
- Refreshing the page reopens the trip you were on. Changes made on another device load automatically.

**Trip set-up in four steps**
1. **Where are you going?** Pick from a dropdown of 70 destinations (cities, beaches, countryside, winter breaks and ski resorts) or type anywhere. Then choose the type of trip from 14 options: Beach & sun, City break, Sightseeing tour, Countryside & walking, Road trip, Cruise, Theme parks & family fun, Spa & wellness, Camping, Festival or event, Adventure & safari, Business trip, Winter sightseeing, and Ski / snowboard.
2. **Pick your dates.** A monthly calendar: tap the day you leave, then the day you come home. It shows how many days are left until you go.
3. **Who's travelling?** Name and age. The age sets the app's theme and adjusts the list:
   - **Sunny** (0–12): bigger, rounder, with kids' extras such as snacks and a comfort toy
   - **Night Flight** (13–19): a dark theme with teen extras such as headphones and a second power bank
   - **Classic** (20–59)
   - **Easy Read** (60+): larger text, stronger contrast and comfort items
4. **Weather.** Typical daytime highs and night-time lows for that destination and month, with a weather icon (sunny, sunny spells, cloudy, rain, sleet, snow or storms).

**Countdown.** The trip summary shows how many days are left until the trip, "Tomorrow!", "It's today!", which day of the trip you're on, or "Trip finished".

**The trip folder.** Results open in a file-style folder with four tabs:

| Tab | What it does |
|---|---|
| **Packing List** | Clothes as a checklist, with an extra section for the trip type (e.g. deck shoes and dress-up outfits for a cruise). Tick items off, change any quantity with − / + (e.g. trousers 1 → 5), remove items or add your own. |
| **Essentials** | Toiletries, documents, tech and the extras for your age group, with the same checklist controls. |
| **Daily Outfits** | One page per trip day. Type what you're doing and get outfit ideas that match the activity, the weather and the trip type. Add any idea to the Packing List in one tap, hide ideas, or add your own pieces. |
| **Things to Do** | A few quick questions (who's going, interests, pace, budget), then picture tiles of real places near the destination. Tap the heart to save one, or tap the tile for a full page with a description, quick facts, what to wear and links to Tripadvisor and Google Maps. Add any idea to a day, or type your own plans. |

**Other features**
- **Copy list:** copies the Packing List and Essentials as plain text to paste into Notes or a message.
- **App colour:** the palette button in the top-right corner recolours the whole app (background, cards, folder tabs and buttons). There are 12 preset colours or you can choose your own. Light text switches on automatically for dark colours. Each trip remembers its own colour.
- **Works on phones:** the layout fits phone screens as well as desktops.

## Setting it up

### 1. Supabase
1. Create a Supabase project.
2. In the **SQL Editor**, create the `trips` table. The app uses the columns `id`, `user_id`, `name`, `data` and `updated_at`, and the rules below make sure each person can only see and change their own trips:
   ```sql
   create table public.trips (
     id uuid primary key default gen_random_uuid(),
     user_id uuid not null default auth.uid() references auth.users(id) on delete cascade,
     name text not null default 'New trip',
     data jsonb not null default '{}'::jsonb,
     created_at timestamptz not null default now(),
     updated_at timestamptz not null default now()
   );
   alter table public.trips enable row level security;
   create policy "Read own trips" on public.trips for select using (auth.uid() = user_id);
   create policy "Add own trips" on public.trips for insert with check (auth.uid() = user_id);
   create policy "Change own trips" on public.trips for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
   create policy "Delete own trips" on public.trips for delete using (auth.uid() = user_id);
   ```
3. Go to **Authentication → URL Configuration** and set the **Site URL** to your Vercel address (for example `https://packing-pal-sooty.vercel.app`). The confirmation email links back to this address.

### 2. Connect the app
Near the top of the `<script>` block in `index.html`, set your project's details:
```js
const SUPABASE_URL="https://your-project.supabase.co";
const SUPABASE_KEY="your publishable (public) key";
```
The publishable key is designed to be public, so it's safe in the file. Never put the secret (service role) key here.

### 3. Publish on Vercel
1. Put `index.html` and this `README.md` in a GitHub repository.
2. In Vercel, choose **Add New → Project**, import the repository and click **Deploy**. No build settings are needed.
3. Each time you upload a new `index.html`, Vercel publishes it automatically.

## How it works

- **Saved trips:** each trip is one row in the `trips` table. The `data` column holds the whole trip (destination, dates, traveller, weather, lists, ticks, quantities, outfits, plans, saved ideas and colour) as JSON. The `name` column holds a short label such as "Lisbon · Fri 20 Nov – Wed 25 Nov".
- **Offline use:** the open trip is also kept in the browser, so it still works if the connection drops. Changes are sent to Supabase when the connection comes back. Logging out removes trips from that browser.
- **Weather data:** each built-in destination has typical monthly high and low temperatures (climate averages) stored in the file. These are not live forecasts. The app reminds travellers to check a real forecast about 10 days before they fly. For places not on the list, the traveller picks the usual climate (Freezing, Cold, Mild, Warm or Hot).
- **Outfit ideas:** the Daily Outfits page looks for activity words in your plans, such as *hike*, *beach*, *dinner*, *museum*, *temple*, *spa*, *meeting* or *boat*. It combines them with the day's weather and the trip type to suggest clothes. With no plans entered, it suggests clothes from the weather and trip type, plus comfy travel clothes on the first and last days.
- **Things to Do ideas and photos:** the app finds real places near the destination using Wikipedia's free API, ranks them by the traveller's interests, and shows their photos from Wikimedia Commons with a credit link on each details page. Places without a suitable photo are left out. Opening times and prices aren't included, so the app tells travellers to check before they go.

## Project structure

```
index.html   The whole app: markup, styles, scripts and the logo (embedded as an image)
README.md    This file
```

External resources:
- **Supabase JavaScript library** (`@supabase/supabase-js` version 2) from cdn.jsdelivr.net, for logins and saving.
- **Google Fonts** (Fredoka, Nunito and Atkinson Hyperlegible). Without them the app falls back to standard system fonts.
- **Wikipedia and Wikimedia Commons**, for Things to Do ideas and photos.

## Customising

All the content lives in the `<script>` block inside `index.html`:

- **Add or edit destinations:** the `PLACES` object. Each entry has a trip type, altitude in metres, and 12 monthly highs and lows in °C (January to December):
  ```js
  "Lisbon, Portugal":{t:"city",alt:0,hi:[15,16,19,20,23,26,28,29,27,23,18,15],lo:[8,9,11,12,14,17,18,19,18,15,11,9]},
  ```
  Destination types are `city`, `beach`, `country`, `snow` and `ski`.
- **Add or rename trip types:** the `TYPES` list. Each type's extra packing items are in the `X` list inside `buildList()`, and its daily outfit ideas are in `TYPE_DAY`.
- **Change the starting packing list:** the `buildList()` function.
- **Change outfit ideas for activities:** the `RULES` list, which pairs activity words with suggested clothes.
- **Change the colour choices:** the `TILES` list (palette) and the `:root` colour settings at the top of the `<style>` block (age themes).

## Browser support

Works in current versions of Chrome, Edge, Safari and Firefox on desktop and mobile. Your own custom colours and the dark Night colour use the CSS `color-mix()` function, which needs a browser released in 2023 or later.

## Credits

Created by Vanita Rattan. The Packing Pal logo is part of the project.

## Licence

No licence has been chosen yet. Add one before sharing the code publicly, for example via **Add file → Create new file → `LICENSE`** on GitHub, which offers templates.
