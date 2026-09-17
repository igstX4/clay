# Clay — pottery studio landing page

![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=white)
![GSAP](https://img.shields.io/badge/GSAP-3-88CE02?logo=greensock&logoColor=white)
![Ant Design](https://img.shields.io/badge/Ant_Design-5-0170FE?logo=antdesign&logoColor=white)
![Sass](https://img.shields.io/badge/Sass-CF649A?logo=sass&logoColor=white)

A one-page marketing site for a pottery studio: it introduces the workshop,
shows the portfolio and the reviews, lists the workshop tariffs and takes
bookings through a modal form. Built as a single React page with smooth
section-to-section scrolling.

## Sections

```
Header (logo + nav + burger menu on mobile)
Hero image
StagesOfWork   — how a session goes, three illustrated steps
MasterClass    — photo mosaic of the studio
Services       — four tariffs, each with a "Забронировать" button
Works          — portfolio gallery
Reviews        — customer photos and quotes
Footer
Booking modal  — opened from the header or from any tariff card
```

| Tariff | Details |
|--------|---------|
| Мастер-класс на одного | 1,5 часа / 1499 ₽ |
| Мастер-класс для двоих | 2 часа / 2990 ₽ |
| Подарочный сертификат | 4 часа / 4990 ₽ |
| Абонемент на 1 месяц | 1 месяц / 6990 ₽ |

Picking a tariff does two things at once: it stores the chosen product in the
page state and opens the modal, so the form already knows what is being booked.

## Interesting pieces

**Smooth anchor navigation.** `Main.jsx` holds a ref per scrollable block and
hands them to the header. Clicking a nav item calls

```js
gsap.registerPlugin(ScrollToPlugin);
const scrollTo = (target) => gsap.to(window, { duration: 1, scrollTo: target });
```

which animates the viewport to the block — `scroll-behavior: smooth` would not
give the same control over duration and easing.

**A hand-written date picker.** `Components/DatePicker` renders the current week
(Пн–Вс) with prev/next arrows, closes on an outside click (a `mousedown` listener
registered in `useEffect`) and greys out days that are already in the past, so an
invalid booking date cannot be picked in the first place. Keeping it in-house
avoids shipping a calendar library for a single week view.

**Form validation.** The booking modal uses React Hook Form with required rules
on name, phone and e-mail, so errors are shown per field without wiring a
controlled input for each one.

## Project structure

```
src/
├── Components/
│   ├── BurgerMenu/       mobile navigation
│   ├── DatePicker/       weekly picker + its own SCSS
│   ├── Footer/  Header/  layout chrome, inline SVG icons in Svgs.jsx
│   ├── MasterClass/      studio photo mosaic
│   ├── Modal/            booking form
│   ├── Reviews/  Works/  social proof and portfolio galleries
│   ├── Services/         tariffs and their booking buttons
│   └── StagesOfWork/     "how it works" steps
├── Pages/Main/Main.jsx   the page: sections, refs, scroll helper, modal state
├── assets/               photos and decorative SVGs
├── App.js                renders <Main />
└── index.js  index.css   CRA entry point
```

Each component keeps its own SCSS module (plus two plain `.scss` files for the
header and the picker) and exports inline SVG icons from a neighbouring
`Svgs.jsx` instead of pulling an icon font.

## Tech stack

React 18 (Create React App) · GSAP 3 + ScrollToPlugin · Ant Design 5 (the time
slot `Select`) · React Hook Form · axios · SCSS modules · Sass.

## Running it

```bash
npm install
npm start      # http://localhost:3000
npm run build  # production bundle in ./build
```

The page is fully static apart from the booking request, so the build is a plain
folder of assets and can be hosted anywhere.

## Notes / where to take it next

- The submit handler still posts to a placeholder:

  ```js
  axios.post('API_IP', { fullName, phoneNumber, product });
  ```

  Point it at a real endpoint (and read the response) to make the booking flow
  end-to-end; today the form validates and then silently does nothing.
- The modal heading is a fixed string while the payload carries the selected
  tariff — the heading should use the same `activeProduct` value.
- `react-datepicker` and `react-select` are declared in `package.json` but no
  longer imported: the picker is custom and `antd` covers the selects. Removing
  them would slim the dependency list.
- The chosen date and time live inside their components, so they are not part of
  the submitted payload yet — lifting them into the modal state is the first step
  to a complete request.
