# Camp Experts Italia -- Assembly & Deployment Guide

Complete guide for assembling, customizing, and deploying the Camp Experts Italia website.

---

## 1. File Structure

```
site/
├── _base.html                    # HTML template (reference only, not served)
├── tailwind.config.js            # Tailwind CSS configuration (design tokens)
├── index.html                    # Homepage
├── chi-sono.html              # About Laura page
├── servizi.html                  # Services page
├── programmi.html                # Programs directory page
├── prenota.html                  # Booking / consultation page
├── faq.html                      # FAQ page with accordion
├── blog.html                     # Blog / articles listing page
├── privacy.html                  # Privacy policy (to be created)
├── cookie.html                   # Cookie policy (to be created)
├── sitemap.xml                   # XML sitemap for search engines
├── robots.txt                    # Crawler directives
├── ASSEMBLY.md                   # This file
│
├── css/
│   └── styles.css                # Custom styles (buttons, nav, cards, footer, etc.)
│
├── js/
│   └── programs.json             # Programs data (consumed by programmi.html)
│
├── assets/
│   ├── images/                   # All site images
│   │   └── og-image.jpg          # Open Graph sharing image (1200x630)
│   ├── icons/                    # Favicons and app icons
│   │   ├── favicon.svg           # SVG favicon (preferred)
│   │   ├── favicon-32x32.png     # PNG favicon 32x32
│   │   └── apple-touch-icon.png  # Apple touch icon 180x180
│   └── fonts/                    # Local font files (if needed)
│
└── components/
    ├── header.html               # Top bar + logo + nav + mobile menu
    ├── footer.html               # Footer with contacts, territories, links
    ├── cta-banner.html           # Call-to-action banner section
    ├── whatsapp-fab.html         # WhatsApp floating action button
    └── cookie-banner.html        # GDPR cookie consent banner
```

---

## 2. How to Combine Header/Footer Components into Each Page

The site uses a **static component pattern**. Each page is a self-contained HTML file that includes the header, footer, WhatsApp FAB, and cookie banner inline. The `components/` folder contains reference copies of each component.

### Manual Assembly (Current Approach)

Each page already contains the full header and footer markup inline. When making changes to the header or footer:

1. Edit the component file in `components/` (e.g., `components/header.html`)
2. Copy the updated markup into **every page** that includes it
3. Pages that need updating: `index.html`, `chi-sono.html`, `servizi.html`, `programmi.html`, `prenota.html`, `faq.html`, `blog.html`

### Automated Assembly (Recommended for Production)

To avoid manually duplicating components, consider one of these approaches:

**Option A: Build-time includes with a static site generator**

Use a tool like [Eleventy (11ty)](https://www.11ty.dev/) or a simple Node.js script:

```bash
npm install -g @11ty/eleventy
```

Convert `_base.html` into a layout template and use `{% include %}` tags.

**Option B: Server-side includes (SSI)**

If hosted on Apache or Nginx, enable SSI and use:

```html
<!--#include file="components/header.html" -->
```

**Option C: Simple build script (Node.js)**

Create a `build.js` script that reads each page template and replaces `<!-- #include file="..." -->` comments with actual file contents:

```js
const fs = require('fs');
const path = require('path');

const pages = ['index.html', 'chi-sono.html', 'servizi.html',
               'programmi.html', 'prenota.html', 'faq.html', 'blog.html'];

pages.forEach(page => {
  let html = fs.readFileSync(path.join('src', page), 'utf8');
  html = html.replace(/<!-- #include file="(.+?)" -->/g, (match, file) => {
    return fs.readFileSync(path.join('src', file), 'utf8');
  });
  fs.writeFileSync(path.join('dist', page), html);
});
```

---

## 3. CSS/JS Dependency Order

Load resources in this exact order in the `<head>` of every page:

```html
<!-- 1. Preconnect (performance) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- 2. Google Fonts -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;1,400;1,500;1,600;1,700&subset=latin,latin-ext&display=swap" rel="stylesheet">

<!-- 3. Tailwind CSS CDN -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- 4. Tailwind Config (must come after Tailwind CDN) -->
<script src="/tailwind.config.js"></script>

<!-- 5. Custom Styles (must come after Tailwind to override when needed) -->
<link rel="stylesheet" href="/css/styles.css">
```

Scripts at the bottom of `<body>` (or page-specific inline scripts):

```html
<!-- 6. Mobile nav toggle (inline in header component) -->
<!-- 7. Page-specific JS (e.g., FAQ accordion, program filters) -->
<!-- 8. Cookie banner (last, just before </body>) -->
```

### Production Note: Replacing Tailwind CDN

For production, replace the Tailwind CDN with a compiled CSS file:

```bash
npx tailwindcss -i ./css/input.css -o ./css/tailwind-output.css --minify
```

Then replace the CDN `<script>` tag and config script with:

```html
<link rel="stylesheet" href="/css/tailwind-output.css">
```

---

## 4. How to Update programs.json

The file `js/programs.json` contains the data for all programs displayed on `programmi.html`. Each program entry follows this schema:

```json
{
  "id": "unique-slug",
  "name": "Program Name",
  "location": "City, State, Country",
  "country": "USA",
  "type": "sport",
  "ageMin": 8,
  "ageMax": 16,
  "duration": "2-8 settimane",
  "description": "Brief Italian description of the program...",
  "highlights": ["Feature 1", "Feature 2", "Feature 3"],
  "image": "/assets/images/programs/program-slug.jpg",
  "url": "https://external-program-website.com",
  "featured": true
}
```

### Adding a New Program

1. Add a new JSON object to the array in `programs.json`
2. Ensure the `id` is unique (use kebab-case slugs)
3. Use Italian for `description` and `highlights`
4. Place the program image in `assets/images/programs/`
5. Set `"featured": true` if it should appear in highlighted sections

### Program Types

Use these values for the `type` field (for filtering):

- `sport` -- Athletic/sports camps
- `lingua` -- Language courses
- `educativo` -- Educational/academic programs
- `artistico` -- Arts and creative programs
- `avventura` -- Adventure/outdoor programs
- `tecnologia` -- Technology/STEM programs

---

## 5. How to Swap in Real Images

### Naming Conventions

Use lowercase, kebab-case filenames:

```
assets/images/
├── hero-home.jpg                 # Homepage hero
├── hero-chi-e-laura.jpg          # About page hero
├── laura-portrait.jpg            # Laura's professional photo
├── laura-casual.jpg              # Laura casual/candid photo
├── og-image.jpg                  # Open Graph image (social sharing)
├── testimonials/
│   ├── famiglia-rossi.jpg        # Testimonial family photos
│   └── famiglia-bianchi.jpg
└── programs/
    ├── camp-name-slug.jpg        # Program card images
    └── camp-name-slug-hero.jpg   # Program detail hero images
```

### Recommended Image Sizes

| Use Case               | Dimensions    | Format  | Max File Size |
|------------------------|---------------|---------|---------------|
| Hero backgrounds       | 1920 x 1080  | WebP/JPG | 200 KB       |
| Program card images    | 800 x 600    | WebP/JPG | 80 KB        |
| Testimonial photos     | 400 x 400    | WebP/JPG | 40 KB        |
| Laura portrait         | 600 x 800    | WebP/JPG | 80 KB        |
| OG image (social)      | 1200 x 630   | JPG      | 100 KB       |
| Favicon SVG            | any           | SVG      | 5 KB         |
| Apple touch icon       | 180 x 180    | PNG      | 20 KB        |

### Image Optimization

Use [Squoosh](https://squoosh.app/) or the CLI tool:

```bash
# Convert to WebP with quality 80
npx @aspect-ratio/cli --width 1920 --quality 80 --format webp input.jpg output.webp
```

Or use `<picture>` elements for format fallback:

```html
<picture>
  <source srcset="/assets/images/hero-home.webp" type="image/webp">
  <img src="/assets/images/hero-home.jpg" alt="Description" loading="lazy">
</picture>
```

---

## 6. Deployment Instructions

### Option A: Netlify (Recommended)

1. Push the `site/` folder to a Git repository (GitHub, GitLab, or Bitbucket)
2. Sign in to [Netlify](https://www.netlify.com/)
3. Click "Add new site" > "Import an existing project"
4. Connect your repository
5. Set build settings:
   - **Base directory**: `site`
   - **Build command**: (leave empty for static sites, or use your build script)
   - **Publish directory**: `site` (or `dist` if using a build script)
6. Click "Deploy site"
7. Configure custom domain: `www.campexpertsitalia.com`
8. Enable HTTPS (automatic with Netlify)

**Netlify `_redirects` file** (create in `site/`):

```
# Redirect index to clean URL
/index.html    /    301
```

### Option B: Vercel

1. Install Vercel CLI: `npm i -g vercel`
2. Navigate to the `site/` directory
3. Run `vercel` and follow prompts
4. Configure custom domain in the Vercel dashboard

**Vercel `vercel.json`** (create in `site/`):

```json
{
  "cleanUrls": true,
  "trailingSlash": false
}
```

### Option C: Simple Static Hosting (Apache/Nginx)

Upload the contents of `site/` to your web root.

**Apache `.htaccess`**:

```apache
# Enable gzip compression
<IfModule mod_deflate.c>
  AddOutputFilterByType DEFLATE text/html text/css application/javascript application/json image/svg+xml
</IfModule>

# Cache static assets
<IfModule mod_expires.c>
  ExpiresActive On
  ExpiresByType text/css "access plus 1 year"
  ExpiresByType application/javascript "access plus 1 year"
  ExpiresByType image/jpeg "access plus 1 year"
  ExpiresByType image/png "access plus 1 year"
  ExpiresByType image/webp "access plus 1 year"
  ExpiresByType image/svg+xml "access plus 1 year"
  ExpiresByType font/woff2 "access plus 1 year"
</IfModule>

# Force HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

**Nginx config**:

```nginx
server {
    listen 443 ssl http2;
    server_name www.campexpertsitalia.com;
    root /var/www/campexpertsitalia/site;
    index index.html;

    # Gzip
    gzip on;
    gzip_types text/css application/javascript application/json image/svg+xml;

    # Cache static assets
    location ~* \.(css|js|jpg|jpeg|png|webp|svg|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Clean URLs
    location / {
        try_files $uri $uri.html $uri/ =404;
    }
}
```

---

## 7. How to Connect Booking Form to HubSpot

The booking page (`prenota.html`) contains a consultation request form. To connect it to HubSpot:

### Option A: HubSpot Embedded Form (Easiest)

1. In HubSpot, go to **Marketing > Forms > Create Form**
2. Build a form with fields matching your booking form (name, email, phone, child age, message, etc.)
3. Get the embed code from HubSpot
4. Replace the existing `<form>` in `prenota.html` with the HubSpot embed:

```html
<div id="hubspot-form-container">
  <script charset="utf-8" type="text/javascript" src="//js.hsforms.net/forms/embed/v2.js"></script>
  <script>
    hbspt.forms.create({
      region: "eu1",           // or "na1" depending on your HubSpot account
      portalId: "YOUR_PORTAL_ID",
      formId: "YOUR_FORM_ID",
      target: "#hubspot-form-container",
      locale: "it",
      translations: {
        it: {
          submitText: "Prenota la Consulenza",
          fieldLabels: {
            email: "Email",
            firstname: "Nome",
            lastname: "Cognome",
            phone: "Telefono"
          }
        }
      }
    });
  </script>
</div>
```

### Option B: Custom Form with HubSpot API

Keep the existing styled form and submit data via the HubSpot Forms API:

```javascript
document.getElementById('booking-form').addEventListener('submit', function(e) {
  e.preventDefault();

  var formData = {
    fields: [
      { name: 'firstname', value: document.getElementById('nome').value },
      { name: 'lastname', value: document.getElementById('cognome').value },
      { name: 'email', value: document.getElementById('email').value },
      { name: 'phone', value: document.getElementById('telefono').value },
      { name: 'message', value: document.getElementById('messaggio').value }
    ],
    context: {
      pageUri: window.location.href,
      pageName: document.title
    }
  };

  fetch('https://api.hsforms.com/submissions/v3/integration/submit/YOUR_PORTAL_ID/YOUR_FORM_ID', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(formData)
  })
  .then(function(response) {
    if (response.ok) {
      // Show success message
      document.getElementById('form-success').style.display = 'block';
      document.getElementById('booking-form').style.display = 'none';
    } else {
      throw new Error('Submission failed');
    }
  })
  .catch(function(error) {
    // Show error message
    document.getElementById('form-error').style.display = 'block';
  });
});
```

### HubSpot Setup Checklist

- [ ] Create HubSpot account (free CRM tier is sufficient)
- [ ] Create a contact form with required fields
- [ ] Set up a workflow to send email notifications to Laura on new submissions
- [ ] Configure a thank-you email autoresponder in Italian
- [ ] Test form submission end-to-end
- [ ] Verify data appears in HubSpot CRM contacts

---

## 8. Launch Checklist

### Pre-Launch

- [ ] **Broken links**: Test all internal and external links with [W3C Link Checker](https://validator.w3.org/checklink) or `npx broken-link-checker https://www.campexpertsitalia.com`
- [ ] **Mobile testing**: Test on real iOS and Android devices at minimum, plus Chrome DevTools responsive mode
- [ ] **Cross-browser testing**: Chrome, Firefox, Safari, Edge (latest versions)
- [ ] **Favicon**: Ensure `favicon.svg`, `favicon-32x32.png`, and `apple-touch-icon.png` are in `assets/icons/`
- [ ] **OG image**: Verify `og-image.jpg` (1200x630) exists and displays correctly when sharing on social media (use [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/))
- [ ] **HTML validation**: Run all pages through the [W3C Validator](https://validator.w3.org/)
- [ ] **Accessibility**: Test with [axe DevTools](https://www.deque.com/axe/) or [WAVE](https://wave.webaim.org/) -- aim for zero critical/serious issues
- [ ] **Performance**: Run [Lighthouse](https://pagespeed.web.dev/) and aim for 90+ scores on Performance, Accessibility, Best Practices, SEO
- [ ] **Forms**: Test booking form submission, email notification, and autoresponder
- [ ] **WhatsApp link**: Verify `wa.me/33618455997` opens correctly on mobile
- [ ] **Phone link**: Verify `tel:+33618455997` initiates a call on mobile
- [ ] **404 page**: Create a custom 404 page in Italian

### Analytics (GA4)

1. Create a Google Analytics 4 property at [analytics.google.com](https://analytics.google.com/)
2. Get the Measurement ID (format: `G-XXXXXXXXXX`)
3. Add the GA4 script to every page, in the `<head>`, **after** the cookie consent default:

```html
<!-- Google Analytics 4 -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}

  // Default consent state (denied until user accepts cookies)
  gtag('consent', 'default', {
    'analytics_storage': 'denied',
    'ad_storage': 'denied',
    'ad_user_data': 'denied',
    'ad_personalization': 'denied',
    'wait_for_update': 500
  });

  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

4. The cookie banner component (`components/cookie-banner.html`) already includes commented-out code for updating consent when the user accepts/rejects cookies. Uncomment the `gtag('consent', 'update', ...)` block.

### GDPR / Cookie Consent

- [ ] Include the cookie banner component on every page (see `components/cookie-banner.html`)
- [ ] Create a Cookie Policy page (`cookie.html`) listing all cookies used
- [ ] Create a Privacy Policy page (`privacy.html`) compliant with Italian/EU regulations
- [ ] Ensure GA4 runs in consent mode (no tracking before user accepts)
- [ ] Test that rejecting cookies disables analytics tracking
- [ ] Verify localStorage key `ce_cookie_consent` is set correctly after user interaction

### SEO

- [ ] Verify `sitemap.xml` is accessible at `/sitemap.xml`
- [ ] Verify `robots.txt` is accessible and references the sitemap
- [ ] Submit sitemap to Google Search Console
- [ ] Verify all pages have unique `<title>` and `<meta description>` tags
- [ ] Verify canonical URLs are correct on all pages
- [ ] Verify FAQ schema markup on `faq.html` with [Google Rich Results Test](https://search.google.com/test/rich-results)
- [ ] Verify Open Graph tags render correctly on social platforms

### DNS & SSL

- [ ] Point `campexpertsitalia.com` DNS to your hosting provider
- [ ] Configure `www` subdomain redirect
- [ ] Verify SSL certificate is active and auto-renewing
- [ ] Test HTTPS redirect (HTTP should 301 to HTTPS)

---

## 9. Post-Launch: Ongoing Maintenance

### Adding Blog Posts

1. Create a new HTML file for the blog post (e.g., `blog/come-scegliere-camp.html`)
2. Use the same page template structure (header, main, footer, WhatsApp FAB)
3. Add the article card to `blog.html` -- copy an existing `<article>` card and update:
   - Title, date, category tag, excerpt, and link
   - Add the corresponding image to `assets/images/blog/`
4. Update `sitemap.xml` with the new blog post URL and `lastmod` date
5. Share on social media with the correct OG tags

### Adding New Programs

1. Add the new program entry to `js/programs.json` (see Section 4 above)
2. Add the program image to `assets/images/programs/`
3. The programs page will automatically display the new entry if it uses dynamic rendering from the JSON

### Adding Testimonials

1. Prepare the testimonial text in Italian
2. Get a family photo (400x400 recommended) and add to `assets/images/testimonials/`
3. Add the testimonial card to the relevant page section
4. Include: family name, child's age/program, quote, and optional photo

### Updating Content

- **Contact info changes**: Update in `components/header.html`, `components/footer.html`, and all assembled pages
- **New territory**: Add to the footer territories list in all pages
- **Price changes**: Update in `faq.html` (FAQ about pricing) and `servizi.html`
- **Seasonal updates**: Refresh hero messaging and featured programs before each summer season

### Monitoring

- Check Google Analytics weekly for traffic trends
- Monitor Google Search Console for crawl errors and search performance
- Run Lighthouse quarterly to maintain performance scores
- Review and respond to any form submissions promptly
- Keep SSL certificate current (auto-renewal recommended)
