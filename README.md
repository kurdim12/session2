# IEEE UOP Student Branch Website

Official website for the **IEEE University of Petra Student Branch** — Amman, Jordan.

## Overview

A fully responsive single-page website built with vanilla HTML, CSS, and JavaScript. No frameworks or build tools required.

## Sections

| Section | Description |
|---|---|
| Hero | Branch intro, key stats, animated circuit background |
| About | Mission, vision, and values |
| Chapters | CS, Power & Energy, Robotics & Automation, WIE |
| Events | Upcoming workshops, seminars, and competitions |
| Board | Executive board for 2025–2026 |
| Contact | Contact info and inquiry form |
| Footer | Quick links and external resources |

## Project Structure

```
session2/
├── index.html   # Main HTML structure
├── style.css    # All styles (dark theme, responsive)
├── script.js    # Navbar scroll, mobile menu, form, animations
└── README.md    # This file
```

## Getting Started

No build step needed — just open `index.html` in a browser:

```bash
# Option 1: open directly
open index.html

# Option 2: serve locally
npx serve .
# or
python3 -m http.server 8080
```

## Customization

- **Colors** — edit CSS variables at the top of `style.css` (`:root { ... }`)
- **Board members** — update names/roles in the `#board` section of `index.html`
- **Events** — add or edit event cards inside `#events` in `index.html`
- **Contact email** — replace `ieee@uop.edu.jo` in the contact section

## Tech Stack

- HTML5 / CSS3 / Vanilla JS
- [Inter](https://fonts.google.com/specimen/Inter) font via Google Fonts
- Intersection Observer API for scroll animations

## Links

- [IEEE.org](https://www.ieee.org)
- [IEEE Xplore](https://ieeexplore.ieee.org)
- [University of Petra](https://www.uop.edu.jo)
- [IEEE Region 8](https://www.ieeer8.org)
