# Customisation guide

## White-label without code

**Settings › Branding** (admins) stores everything in the database (`tenants.branding`), so it survives upgrades:

- app name and tagline (sidebar, browser title, sign-in page);
- logo (PNG, JPG or WebP) and favicon (PNG or ICO);
- four theme colours: brand, highlight, background, ornament (lighter and darker shades are derived);
- sign-in page headline and text;
- footer text (sidebar, sign-in page, public RSVP page).

## Themes (code)

All design tokens live in **`resources/css/theme.css`**: colours, fonts, radius and shadows. `resources/css/app.css` maps them to Tailwind utilities (`bg-brand`, `text-highlight-deep`, `font-display`…) and shadcn-vue variables. Change the defaults there, then rebuild:

```bash
npm install
npm run build
```

Branding colours set in Settings override these defaults per account. Components under `resources/js/components/ui/` are shadcn-vue components you own and can restyle.

## Adding a language

Two steps, no rebuild:

1. Copy `lang/en.json` to, for example, `lang/mr.json` and translate the values. Keep the keys and the placeholders (`{name}` in the app, `:name` in emails and server messages) exactly as they are. Plural forms are separated by ` | `.
2. Add one line to `config/locales.php`:

   ```php
   'mr' => ['native' => 'मराठी', 'intl' => 'mr-IN', 'font' => 'Noto Sans Devanagari', 'dir' => 'ltr'],
   ```

Then run `php artisan test --filter=Localization`: it fails if a key is missing or a placeholder doesn't match English. The new language appears in Settings › Preferences. For invitation images, add a font that covers the script to `public/fonts/invitation/` and `app/Invitations/Fonts.php` (or upload a `.ttf` in the Invitations page).

## Adding a module

Follow the pattern of an existing module (Vendors is a good, small example):

1. **Migration** in `database/migrations/`: every business table gets
   `$table->foreignId('tenant_id')->constrained()->cascadeOnDelete();` and an index starting with `tenant_id`.
2. **Model** in `app/Models/` using `BelongsToTenant` (filters every query to the current account and fills `tenant_id`) and, if changes should be logged, `LogsActivity`. Store money as integer paise (`App\Support\Money`).
3. **Ability**: add it to `App\Enums\Role::ABILITIES` and to the roles that should have it. It becomes a Gate and appears in the frontend as `auth.can`; add it to the `Ability` type in `resources/js/types/index.d.ts`.
4. **Routes** in `routes/web.php` inside the `can:workspace.access` group, with `->middleware('can:your.ability')`. `tests/Feature/RouteAuthorizationTest.php` fails if a route has no ability.
5. **Controller** returning `Inertia::render('yourmodule/Index', [...])`; scope lookups to the current project (`App\Support\CurrentProject`) and paginate lists.
6. **Page** in `resources/js/pages/yourmodule/Index.vue` using `AppLayout`, `PageHeader`, `FormDialog`, `NativeSelect`, `Pagination`… from `resources/js/components/common/`.
7. **Navigation**: add an item to `resources/js/lib/navigation.ts` with its `ability`.
8. **Texts**: add every string to `lang/en.json`, `hi.json` and `gu.json` (no hard-coded text).
9. **Tests** in `tests/Feature/`.

Need help with custom work? Email support@rsflowhub.com.

## Sample invitation designs

`app/Invitations/SampleTemplates.php` defines the three sample designs (layers, fonts, colours, the QR layer), and `app/Invitations/SampleArtwork.php` draws their backgrounds in code (`php artisan toran:sample-art`).
