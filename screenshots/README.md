# Screenshots

Drop seven PNG / JPG / WebP files here, named exactly as listed in the
project README's screenshot table. Suggested capture viewport
**1440 × 900** (desktop) or **390 × 844** (mobile, iPhone 14 Pro).

| Filename | What to capture | URL |
|---|---|---|
| `01-hero.png`                  | Home hero, current locale, hover paused so the hero variant is stable | https://www.robodor.com/tr |
| `02-products-listing.png`      | Products list with one filter active (e.g. `?application=high-speed`) | https://www.robodor.com/en/urunler?application=high-speed |
| `03-product-detail.png`        | One product, scrolled to the Technical Specs table (six-locale table value visible) | https://www.robodor.com/de/urunler/mr220 |
| `04-admin-shell.png`           | Admin dashboard, full sidebar with all nav items visible — sign in first | https://www.robodor.com/admin |
| `05-admin-spec-editor.png`     | A product's edit page, Technical Specs section, with one card expanded showing locale tabs | https://www.robodor.com/admin/products/{id}/edit |
| `06-admin-legal-editor.png`    | Legal document editor with locale tabs visible and at least one section expanded | https://www.robodor.com/admin/legal/privacy/edit |
| `07-mobile-menu.png`           | Mobile hamburger menu open, with the six-locale switcher row visible | https://www.robodor.com/en (mobile) |

## Capture tips

- **Crop customer logos.** The Referanslar / References page has real
  client logos. If you screenshot anything that includes them, blur or
  crop them out — license risk for a public repo.
- **Don't include admin emails or internal IDs.** When capturing admin
  shell screenshots, scroll user-name labels off-screen or blur them.
- **Demo content for legal editor.** If the live legal text contains the
  full registered company name and you don't want it public, edit a
  scratch entry first and capture that — the editor accepts unsaved
  changes.
- Use the dev tools' device emulation (Cmd-Opt-I → toggle device
  toolbar) for the mobile shot at exactly 390 × 844.

## Optional GIF

A short GIF showing the locale switcher in action — the same product
detail rendered across TR / EN / FR / DE / IT / ES — is a strong
addition. Drop it as `08-locale-switcher.gif`.
