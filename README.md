# Notion Cloud Water Tracker Widget

A minimal, real-time sync water tracking widget designed to be embedded directly into Notion workspaces via GitHub Pages. I created this as my bottle is 621ml and every other trackers I saw would either only track 500ml or 1L. 


## The Problem
Notion embeds run inside isolated client webviews. If you use standard browser `localStorage` to save state, the data will not sync between your desktop app and mobile app. Furthermore, Notion's mobile app frequently clears webview caches, resetting your progress to zero unexpectedly.

## The Solution
This widget establishes cross-device persistence by migrating state from local client memory to a centralized cloud database. Every log action writes to a remote database, ensuring your desktop layout and mobile interface are perfectly aligned in real time.

## Tech Stack
* **Frontend:** Vanilla HTML5, CSS3 Custom Properties, and ES6 JavaScript.
* **Backend Database:** Supabase (PostgreSQL).
* **Animations:** Hardware-accelerated CSS 3D transforms (`translate3d`) running dual layered SVG sine-wave paths at mismatched keyframe rates for a fluid look.

## Key Technical Details

### 1. Height Normalization Logic
Because the wavy SVG paths physically extend upwards past the container box boundary, standard percentage heights make the water look too high at partial capacities (e.g., crossing the exact visual center line at 50%). 

The UI script applies a normalization dampener to intermediate increments so that the visual volume matches the mathematical state:
```javascript
let fillHeight = percentage;
if (percentage > 0 && percentage < 100) {
    fillHeight = percentage - 4; // Visual compensation for SVG wave heights
}
```

### 2. Database Schema & Security Policies
The backend utilizes a single row relational table with Row-Level Security (RLS) policies configured to allow public reads and updates via an anonymized client key:

```sql
create table water_tracking (
  id int8 primary key generated always as identity,
  current_ml int4 default 0,
  updated_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- Seed row
insert into water_tracking (current_ml) values (0);

-- Security Configuration
alter table water_tracking enable row level security;

create policy "Allow public read access" on water_tracking for select to anon using (true);
create policy "Allow public updates" on water_tracking for update to anon using (true) with check (true);
```

## Setup & Local Deployment

1. Clone the repository:
   ```bash
   git clone https://github.com
   cd notion-water-tracker
   ```
2. Open `index.html` and replace the placeholder API strings with your own Supabase project credentials:
   ```javascript
   const SUPABASE_URL = "https://supabase.co";
   const SUPABASE_ANON_KEY = "your-anon-key";
   ```
3. Open the file in any browser or host it via GitHub Pages.

## Attribution
UI layout concept inspired by the Blocs Water Tracker widget. Recreated independently from scratch to demonstrate full-stack state management, mathematical UI alignment, and cloud database optimization.
