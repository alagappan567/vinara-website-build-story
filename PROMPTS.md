# Prompts

These are the prompts used to build the Vinara Infra LLP website, cleaned up so you can paste them straight into an AI coding agent. They follow the order the site was built in. Each prompt comes with pseudo code showing how the feature works.

**How to use them**

- Run them in order. Later prompts build on earlier ones.
- Send one prompt at a time, check the result in the browser, then ask for tweaks.
- Swap in your own company name, numbers and images.
- Put API keys in environment variables. Never paste them into a prompt.

## Contents

1. [Project setup and design system](#1-project-setup-and-design-system)
2. [Site-wide motion layer](#2-site-wide-motion-layer)
3. [Logo preloader](#3-logo-preloader)
4. [Floating navbar](#4-floating-navbar)
5. [Scroll-scrubbed video hero](#5-scroll-scrubbed-video-hero)
6. [Sticky bottom ticker](#6-sticky-bottom-ticker)
7. [Curved card arc](#7-curved-card-arc)
8. [Jelly ribbon and jelly frames](#8-jelly-ribbon-and-jelly-frames)
9. [Live 3D site plan](#9-live-3d-site-plan)
10. [Projects page](#10-projects-page)
11. [Gallery](#11-gallery)
12. [Leadership](#12-leadership)
13. [Clients and partners marquee](#13-clients-and-partners-marquee)
14. [Services page](#14-services-page)
15. [Controls and QHSE page](#15-controls-and-qhse-page)
16. [Careers, Get a Quote and contact emails](#16-careers-get-a-quote-and-contact-emails)
17. [Admin panel](#17-admin-panel)
18. [Mobile rules](#18-mobile-rules)
19. [Speed and polish pass](#19-speed-and-polish-pass)

---

## 1. Project setup and design system

```text
Create a Next.js (App Router) + TypeScript + Tailwind CSS website for a
construction company that builds Grade-A warehouses, industrial buildings and
infrastructure. Pages: Home, About, Services, Projects, Controls & QHSE,
Leadership, Gallery, Careers, Contact / Get a Quote, Privacy, Terms.

Design system:
- Royal-blue theme taken from the logo. Deep navy for text (#06204f), royal blue
  for actions (#0048b0), soft blue tints for surfaces, and an amber accent used
  sparingly.
- Bold display font for headings, clean sans-serif for body text.
- White page backgrounds. Only the hero sections are dark or photographic.
- No empty white gaps between sections anywhere on the site.
- Every page title gets the same treatment: the last words in a blue-to-amber
  gradient.
- Keep all page copy in /data files so it can be edited without touching
  components.

Install GSAP (with ScrollTrigger and SplitText), Lenis, Motion, three.js,
React Three Fiber and Drei.
```

```text
PSEUDO CODE

theme tokens:
  ink     = #06204f      // text
  primary = #0048b0      // buttons, links
  tint    = #eef4ff      // card surfaces
  accent  = amber        // underline, gradient tail

layout(page):
  <Preloader/>           // shows only on a hard refresh
  <Nav/>
  <SmoothScroll>         // Lenis drives the scroll; GSAP listens to it
    <main>{page}</main>
    <Footer/>
  </SmoothScroll>
  <StickyMarquee/>       // fixed ticker along the bottom
  <TextReveal/> <SectionFx/> <ButtonFx/>   // site-wide motion, mounted once

content lives in /data/content/*.ts  ->  components only read from it
```

---

## 2. Site-wide motion layer

```text
Add animation to the whole site without editing every component. Mount three
helpers once in the root layout:

1. Text reveal: every heading and paragraph reveals as it scrolls into view.
   Headings rise word by word out of a line mask; paragraphs fade up softly.
2. Section effects: each section opens from a rounded, inset panel to full
   width as it scrolls in. Rows of cards tip up into place one after another.
   Photos wipe open from a slight zoom and then drift gently against the scroll.
3. Button effects: every pill-shaped button leans towards the cursor, fills
   with colour from the point the cursor came in, sweeps a shine across itself,
   and bursts into small sparks when clicked.

Let any element opt out with a data attribute. Respect prefers-reduced-motion.
Use Lenis for smooth scrolling and keep ScrollTrigger in sync with it.
```

```text
PSEUDO CODE

TextReveal:
  on route change and on DOM mutation:
    for el in querySelectorAll("main h1, h2, h3, p") not inside [data-no-reveal]:
      if heading: split into words inside line masks
                  ScrollTrigger(el, start "top 85%") -> words.from(yPercent 110, stagger)
      else:       ScrollTrigger(el) -> el.from(opacity 0, y 24)

SectionFx:
  for section in main > section not [data-no-sx]:
    curtain: clipPath inset(4% round 32px) -> inset(0) scrubbed to scroll
    cards:   rows of >= 3 siblings -> batch.from(y 70, rotateX -14, opacity 0)
    media:   img.from(scale 1.15, clip wipe) then parallax y while in view

ButtonFx (event delegation on document):
  pointerover pill  -> mark [data-fx-btn], set fill origin = cursor position
  pointermove       -> translate button slightly toward cursor
  pointerleave      -> drain fill toward exit point
  click             -> spawn 8 sparks, fly out, fall with gravity, remove

if prefers-reduced-motion: skip all of the above
```

---

## 3. Logo preloader

```text
Build a preloader that shows on every hard refresh (not on in-site page
changes). Style it like an engineering drawing sheet: corner marks, small
labels ("Site Elevation", "Grade-A Warehousing", "Bengaluru · India") and a
big percentage counter in the corner.

In the middle, draw a site elevation line by line: a finished warehouse with a
truck on the left, a steel frame under a tower crane on the right, and the logo
mark standing on the ground line between them, with soft sun rays behind it.
Below it, show the company name, a 12-block loading bar and the tagline
"GROW TOGETHER".

While loading, the logo "breathes" like a jellyfish: a slow, curvy
squash-and-stretch. Show it for at least 3 seconds and at most 7. When it
finishes, the curtain lifts and the logo flies along a curve into its exact
spot in the navbar, landing on top of the navbar logo. Block scrolling while it
is visible. Never show it on /admin.
```

```text
PSEUDO CODE

phase = "breathing"
after max(3s, page load) or at 7s at the latest: phase = "leaving"

while breathing:
  draw SVG strokes one by one (stroke-dashoffset 1 -> 0, staggered delays)
  logo CSS animation: scaleY 1 -> 1.06 -> 0.95, scaleX opposite (jelly breath)
  counter ticks up to 100%, loading blocks fill
  block wheel/touchmove, lenis.stop()

on leaving:
  freeze logo at its current breath pose
  target = navbar logo rect, offset to where the mark sits inside it
  hide the navbar's own mark (clip-path)
  timeline:
    squash (1.08, 0.94)
    move x with power3.inOut and y with power4.inOut   // different eases make a curve
    scale down to the target size
    reveal navbar mark, fade flying logo
  curtain slides up (CSS)
  phase = "gone", lenis.start()
```

---

## 4. Floating navbar

```text
Make the navbar a floating, rounded glass pill near the top of the screen:
logo on the left, links in the middle, a "Get Quote" button on the right.

- The active page has a solid royal-blue pill behind its link. The pill slides
  smoothly to the new link when you change page.
- A soft spotlight follows the cursor inside the bar.
- A thin blue-to-amber progress line along the bottom of the bar.
- The bar gets more compact once you scroll, hides when scrolling down, and
  comes back when scrolling up.
- The Get Quote button has an arrow in a circle, a shine sweep on hover and a
  ripple on click.
- On mobile: logo + menu button. The menu opens full screen and its close
  button must always be visible above the overlay.
```

```text
PSEUDO CODE

on scroll:
  compact = scrollY > 40
  hidden  = scrollingDown && scrollY > 200
  progress line scaleX = scrollY / (docHeight - viewport)

on pointermove inside bar:
  set CSS vars --x, --y  ->  radial-gradient spotlight at (--x, --y)

active pill:
  shared layout element (Motion layoutId="nav-pill") under the current link
  -> animates position/width automatically when the route changes
```

---

## 5. Scroll-scrubbed video hero

```text
Replace the home hero with a scroll-driven drone video. I have a 4K video of a
warehouse: the camera starts in a drone view, flies through an open shutter,
moves through the interior and comes back out to a straight-on front view.

Cut it into frames (about 30 fps, WebP, in 1920 and 2560 widths) and play the
frames on a full-screen canvas as the user scrolls, like Apple product pages.

- Pin the hero while scrolling through about 7 screens of scroll.
- Show the intro (badge, "Grow Together." headline, short text, Explore
  Projects and Contact Us buttons) at the start. It fades out as soon as the
  user scrolls.
- Then tell a story in 4 captions that alternate bottom-right and top-left:
  The Land (acres), The Build (buildings and sq.ft), The Standard (QHSE),
  The Legacy (years of experience).
- After the last frame, hold the picture while the next section slides up over
  the hero.
- Load frames progressively so a fast scroll never shows a blank canvas, and
  use the smaller frames on phones.
- On mobile, the hero must fill the full height with no white gap when the
  browser address bar hides.
```

```text
PSEUDO CODE

FRAMES = 361
width  = phone ? 1920 : (screen needs > 2000px ? 2560 : 1920)

load order = every 32nd frame, then 16th, 8th, 4th, 2nd, 1st   // coarse-to-fine
queue loads with at most 6 in flight
when the user scrolls to a frame that isn't loaded:
  move the 12 frames around it to the front of the queue

render(frame):
  pick the nearest decoded frame (search both directions)
  draw it to cover the canvas (scale = max(cw/iw, ch/ih), centred)

timeline = ScrollTrigger(pin hero, end "+=700%", scrub 0.6):
  0.00 -> 1.00  state.frame: 0 -> FRAMES-1, onUpdate render
  0.00 -> 0.03  intro copy: opacity 0, y -40
  for i in 0..3:
    t = 0.05 + i * 0.23
    caption[i] in at t (from bottom-right or top-left), out at t + 0.13
  1.00 -> 1.17  hold last frame

canvas section height = 100lvh   // largest viewport, so no gap on mobile
```

---

## 6. Sticky bottom ticker

```text
Add a ticker fixed to the bottom of every page. It scrolls continuously along a
gently waving line, like a ribbon. Items:

- OUR WORK: 213.06 acres under development across three parks
- PROJECTS HANDLED: 12 buildings · 31,33,500 sq.ft built-up
- CUSTOMERS: Trusted by Amazon · DHL · CBRE · Turner & Townsend
- BUILD QUALITY: QHSE-driven delivery with in-house RMC and site controls
- LEGACY: 37+ years of engineering experience

Labels in spaced-out blue capitals, values in white, a small diamond between
items, on a deep navy band. It pauses only while the mouse is over it.
```

```text
PSEUDO CODE

build SVG <path> for a sine wave across the screen
<textPath href="#wave" startOffset={offset}> items repeated twice </textPath>

every frame:
  offset -= SPEED * dt * pace
  if offset < -oneCopyWidth: offset += oneCopyWidth   // seamless loop
  wave phase += DRIFT * dt                            // the wave itself slides
pace eases to 0 on mouse enter, back to 1 on leave
pause entirely when off screen (IntersectionObserver)
```

---

## 7. Curved card arc

```text
Build a reusable "card arc" section: cards sit on the inside of a curved wall,
as if you are standing inside a cylinder. The centre card is the smallest and
faces you straight on. Cards towards the edges get taller and tilt inwards, so
their bottom edges sweep into an arc.

- The arc scrolls by itself in a loop and only stops while the mouse is on a
  card.
- It can be dragged left and right with the mouse or a finger.
- On phones, use a tighter ring with about 3 cards in view, so it still looks
  curved.
- Under the arc, a curved text ribbon carries a line of text, e.g. "Eight
  disciplines. One accountable team. We plan, build and hand it over."

Use it for: Services on Home, the Our Work gallery, Landmark projects, and the
Controls & QHSE steps.
```

```text
PSEUDO CODE

all sizes in --u = clamp(px, 1vw, px)   // scales with viewport

for card i:
  angle = (i - position) * stepAngle
  transform = perspective(CAMERA)
              rotateY(angle)
              translateZ(-RADIUS)          // sit on the inside of the cylinder
  cards past the far side wrap to the other end (loop)

every frame:
  position += dt / SECONDS_PER_CARD   unless hovered or dragging
on drag:
  if moved > 6px: it's a drag, not a click
  position -= dx / cardWidth
  on release: keep the momentum, then ease back to auto-scroll

media (max-width: 767px): use COMPACT_RING (fewer cards per turn, closer camera)
```

---

## 8. Jelly ribbon and jelly frames

```text
Add a playful "jelly" feel that still looks professional for an industrial
company (no water bubbles):

1. Jelly ribbon: a thick curved band that hangs across a section like a banner.
   Text runs along it. It keeps a slow wave going, and wobbles where the
   cursor touches it. Use it for "Contact us" and for key numbers under the
   arcs. On a phone it must keep the same curved shape, not a flat slice.
2. Jelly frame: a rounded-square photo frame whose edge ripples gently and
   wobbles where the cursor crosses it. Use it on leader photos and feature
   images.
```

```text
PSEUDO CODE

// both use the same spring chain
points[N] along the ribbon centre line (or around a squircle for the frame)

every frame (dt):
  for p in points:
    target  = wave(p.x, time)                       // travelling wave
    force   = PULL * (target - p.y)
            + TENSION * (neighbour avg - p.y)
            - DAMPING * p.velocity
    if pointer within POKE_REACH: force += push away from pointer
    p.velocity += force * dt
    p.y        += p.velocity * dt

draw a smooth closed/open path through the points (midpoint quadratic curves)
ribbon: text on a <textPath> along that path
frame:  use the path as a clip-path for the image

on phones: squeeze the chain into the visible part, so the whole curve fits
```

---

## 9. Live 3D site plan

```text
Build a live 3D model of each warehouse park with React Three Fiber, as
realistic as possible. Show it on Home in a "Walk our sites before you visit"
section, and on the Projects page.

Scene:
- Long Grade-A warehouses with pitched metal roofs, roof-light strips, ridge
  vents and solar panels, blue cladding, an office block at one end, loading
  docks with shutters along the long side, and drive-in doors at the back.
- Concrete yards, internal service roads, a compound wall, trees, street
  lights, a staff car park, and a public road outside the gate.
- Movement: trucks reversing onto docks while shutters open, trucks and cars
  on the road, forklifts, workers carrying cartons, tower cranes on buildings
  still under construction. No clouds.
- Use real photo-textured 3D models (trucks, forklifts, cranes, cars) and
  photo-scanned ground textures, compressed so they load fast.

Interaction:
- Drag to orbit. Click first, then scroll to zoom (so the page scroll still
  works). Pinch to zoom on mobile.
- Tap a building to walk inside it in first person: pallet racks, aisles, a
  reach truck moving, dock doors open to daylight. Drag to look around, WASD or
  arrow keys to move, or tap the floor to walk there.
- Switch between parks with pills (NELA 1, NELA 2, Bikaner House).
- A fullscreen button.

Performance: only load the 3D code when the section is about to scroll into
view, and start downloading it while the browser is idle.
```

```text
PSEUDO CODE

<Live3DShowcase>
  requestIdleCallback(() => import("./MasterPlanScene"))     // prefetch
  IntersectionObserver(rootMargin 400px) -> mount <Canvas>

<MasterPlanScene site>
  for b in site.buildings:
    <Warehouse size, docks, status>
      roof sheets + roof-lights + ridge vents + PV arrays
      docks.map(d => <Shutter animated={d.inUse}/> + <Truck reversed/>)
      status == "in progress" ? steel frame + <TowerCrane/> : full cladding
    onClick -> enterBuilding(b)
  <Ground textures: asphalt, grass, concrete (tiled)/>
  <CompoundWall/> <Trees instanced/> <StreetLights/> <PublicRoad/>
  <SiteLife>
    trucks loop the service circuit, cars on the public road
    workers walk dock fronts, carry a carton out, walk back empty
  </SiteLife>
  <OrbitControls enableZoom={focused}/>

models: useGLTF(meshopt-compressed .glb)
        one InstancedMesh per mesh, however many copies are placed

<WarehouseInterior building>
  racks (back-to-back pairs + aisles, 5 levels), high-bay lights
  reach truck shuttles along the cross aisle
  first-person camera at eye height:
    drag = look, WASD/arrows = move, tap floor = walk to point
```

---

## 10. Projects page

```text
Build the Projects page:

1. Hero "Portfolio at a glance." with the live 3D site plan on the right and
   key stats: acres, sq.ft built-up, buildings completed, buildings in
   progress.
2. "Every site. One standard." Three site cards (NELA 1, NELA 2, Bikaner
   House) with acres, buildings, sq.ft and a progress bar.
3. For each site, the 3D plan next to a summary card (total sq.ft, number of
   buildings, a bar split by building) and one card per building: name,
   status chip (Completed / Handed over / Work in progress), sq.ft and sq.m,
   share of site, rank by area, and a "View photos & specs" link.
4. A pinned full-screen building showcase: as you scroll, each building's photo
   opens up from the bottom to fill the screen, with its name, client, area and
   progress. The previous photo drifts back as the next one covers it.
5. "Built alongside the best": clients, PMC partners and tenants in three
   columns.
6. Closing CTA "Have a project to build like these?" with a Contact Us button.
```

```text
PSEUDO CODE

SQM_PER_SQFT = 0.092903
for site in sites:
  total = sum(b.sqft for b in site.buildings)
  for b in site.buildings:
    b.share = b.sqft / total
    b.rank  = position when sorted by sqft desc
    b.sqm   = b.sqft * SQM_PER_SQFT

BuildingShowcase:
  pin frame; scroll length = buildings * (1 + HOLD) screens
  for i, photo in photos:
    if i > 0: photo.clipPath inset(100% 0 0 0) -> inset(0)   // wipe up
              photos[i-1] -> scale 0.9, y -10%                // recede
    caption lines rise in as the photo settles
  reduced motion -> plain stack of full-height panels
```

---

## 11. Gallery

```text
Build the Gallery:

- Home "Our Work" section: an auto-looping row of photo cards on a royal-blue
  panel. Hovering a card opens it up to show the category and a short
  description.
- Gallery page: full-width grid with no empty space at the top. A sticky
  filter bar (All, NELA 1, NELA 2, Bikaner House, and activities). Mix 16:9
  landscape cards with shorter 4:5 portrait cards in a stable random pattern.
  Finished warehouses come first as a collage with the first photo large.
- Clicking a photo opens a lightbox that steps through photos in the same
  order.
- Use real warehouse and construction photos. All photos are managed from the
  admin panel.
```

```text
PSEUDO CODE

shape(i) = hash(i) % 3 == 0 ? "4:5" : "16:9"   // stable on server and client

sections = [finished warehouses] + group(photos by activity) + [untagged]
lightboxOrder = flatten(sections)

filter chip -> photos.filter(p => p.site == chip || chip == "All")
```

---

## 12. Leadership

```text
Build the leadership section and page:

- Home: "The people behind the build." with portraits on a curved wall (the
  card arc), auto-scrolling, with name and role under each. Order by role:
  Managing Director, Directors, CEO, then heads of department, then project
  managers.
- Leadership page: a hero with team stats (today's manpower, staff and
  operators, daily safe manhours), director cards with a jelly-frame photo,
  title and a short bio, then the wider team.
- If a leader has no photo yet, show a distinct stand-in portrait for each
  person and "Name to be added" where needed, until the client uploads real
  ones from the admin panel.
- Put the manpower stats at the end of the page.
```

```text
PSEUDO CODE

ROLE_ORDER = ["Managing Director", "Director", "CEO", "Head", "Senior PM", "PM", "DPM", "Engineer"]
leaders.sort(by ROLE_ORDER.indexOf(rank(role)))

photo(leader) = leader.photo ?? standIns[hash(leader.name) % standIns.length]
```

---

## 13. Clients and partners marquee

```text
Add a "The world's most demanding operators." section: client and partner
logos (Amazon, DHL, CBRE, Turner & Townsend, FABS, and others) in two rows
that loop in opposite directions along gentle waves. Show the logos in their
own colours, no boxes around them. Hovering pauses the row. On mobile, at least
three logos must be visible at once. Logos and names are uploaded and edited
from the admin panel, and either the logo or the text can be hidden.
```

```text
PSEUDO CODE

rows = split(partners, 2)
row[0] moves left, row[1] moves right
each row follows its own sine wave (different crest count), staying in its lane
partner.showLogo ? <img logo/> : null ;  partner.showName ? <span name/> : null
```

---

## 14. Services page

```text
Build the Services page:

1. Hero "One partner. Every discipline." on a navy background, with tags
   (Design, Build, Precast, Supply) and a row of 8 tall image panels:
   Warehousing & Logistics Parks, Industrial Construction, Precast
   Construction, Infrastructure Development, MEP Works, PEB & Fabrication,
   Interior Works, RMC Plant. The selected panel opens wide to show its title
   and summary; the others stay narrow with a number and an arrow.
2. Service chapters: on desktop, pin the block and let the page scroll carry
   the chapters sideways, one screen each. A numbered list on the left tracks
   the current chapter ("01 / 07") with a "Next:" link. Each chapter has a
   jelly-framed photo, a description, an "On site" example card and a "Scope
   of work" list. Each scroll settles on the next chapter.
3. RMC plant section: product range, quality assurance, fleet & logistics, and
   key strengths.
4. On mobile, chapters stack vertically.
```

```text
PSEUDO CODE

ServicesHero:
  panels.map((p, i) => <Panel flex={i == active ? 6 : 1}/>)   // CSS flex transition
  click / hover -> active = i

ServiceChapters (desktop, tall enough screens):
  pin block at top = 104px (under the nav)
  track.x = -progress * (COUNT - 1) * 100vw
  rail bar height = progress
  on scrollEnd: snap to nearest chapter in the direction of travel
  #slug in URL -> scroll to that chapter's position along the pin
mobile -> chapters stacked
```

---

## 15. Controls and QHSE page

```text
Merge the old Planning and Quality pages into one "Controls & QHSE" page on a
white background. Lay it out as clear steps:

Preconstruction → Planning → Quality → Safety → Progress monitoring → Handover

- "The control cycle": Plan, Execute, Measure cards in the card arc.
- Quality responsibilities as a two-column checklist.
- Emergency response steps (Detect, Alert, Evacuate, Assemble, Account, All
  clear) and near-miss reporting steps (Immediate action, Report, Identify
  root cause, Corrective action, Preventive action, Learn & share) as numbered
  flows.
- A progress marquee at the bottom. Remove any organisation hierarchy chart.
```

```text
PSEUDO CODE

steps = [Preconstruction, Planning, Quality, Safety, Progress, Handover]
steps.map((s, i) => <Step n={i+1} title={s.title} points={s.points}/>)
flows.map(flow => <NumberedFlow items={flow.items} note={flow.note}/>)
```

---

## 16. Careers, Get a Quote and contact emails

```text
1. Careers page: hero "Build your career where things get built." with a
   "Latest openings" card. List open roles from the database. Each role has an
   Apply form (name, email, phone, role, experience, message).
2. Get a Quote page: a step-by-step "quote studio" form: project type, site
   location, area, timeline, then contact details. Departments: Managing
   Director, BD, Accounts, Planning, Quality, HSE, HR.
3. Contact page: form plus an accurate, light-coloured map of the head office.
4. Send every form by email with Resend:
   - job applications -> HR inbox
   - quote requests   -> BD inbox
   - contact form     -> info@ inbox
   Set reply-to to the person's email. Validate on the server and escape all
   input. Keep the API key and the inbox addresses in environment variables.
```

```text
PSEUDO CODE

POST /api/enquiry { type, fields }
  if type not in ["application", "quote", "contact"]: 400
  fields = trim and cap each value at 4000 chars
  if !fields.name or !validEmail(fields.email): 400

  to = { application: env.HR_EMAIL,
         quote:       env.BD_EMAIL,
         contact:     env.INFO_EMAIL }[type] ?? env.FALLBACK_EMAIL

  subject = application -> "Job application — {role} — {name}"
            quote       -> "Quote request — {company}"
            contact     -> "Website enquiry — {name}"

  html = table of escape(key) / escape(value)
  send via the Resend email API
       { from, to, reply_to: fields.email, subject, html }
  ok ? 200 : 502
```

---

## 17. Admin panel

```text
Add a password-protected admin panel at /admin so the client can update the
site without a developer:

- Edit, add and delete: projects, gallery photos, building photos and
  captions, home page sections, leadership photos, job openings, and partners
  (name + logo upload, with options to hide the text or the logo).
- Store data in Neon Postgres. Upload images to Cloudinary and delete the old
  image when one is replaced.
- Security:
  - one admin account, password hashed with scrypt
  - signed, httpOnly, same-site session cookie that lasts 8 hours
  - lock an IP address out for 15 minutes after 3 wrong passwords
  - "Forgot password" emails a one-time reset link that expires in 30 minutes
    (max 3 per hour)
  - "Change password" signs out every other session
  - password rules: at least 10 characters, letters and numbers, not
    containing the username
- No smooth-scroll or preloader on admin pages.
```

```text
PSEUDO CODE

tables: admin_account, admin_login_attempts, admin_reset_tokens,
        projects, gallery_items, building_photos, home_items, jobs, partners

login(username, password, ip):
  if lockedFor(ip) > 0: 429 "try again in N minutes"
  account = checkCredentials(username, password)   // check both, constant time
  if !account:
    left = recordFailure(ip)        // at 3 failures: locked_until = now + 15 min
    return 401 "{left} attempts left"
  clearFailures(ip)
  set cookie = "{expiry}.{sessionVersion}.{hmac(secret, expiry.version)}"

isAdmin():
  verify hmac, expiry > now, version == account.sessionVersion

changePassword(old, new):
  require isAdmin and old is correct and passwordProblem(new) == null
  store scrypt(new); sessionVersion += 1   // logs out other sessions

forgot():
  if resets in last hour < 3:
    token = random; store sha256(token), expires now + 30 min
    email link /admin/reset?token=...

upload(image):
  { url, publicId } = cloudinary.upload(image)
  on replace: cloudinary.destroy(oldPublicId)
```

---

## 18. Mobile rules

```text
Check the whole site on mobile (including iOS Safari) and apply these rules
everywhere:

- Every row of cards becomes a horizontal swipe row on mobile only.
- Hero text must never overlap; use shorter text on mobile.
- Curved arcs and ribbons keep the same curved look as on the laptop.
- No flicker while scrolling pinned sections (use native scroll and fixed
  pins on mobile).
- The full hero is visible at 100% zoom, with no white gap when the address
  bar hides.
- The mobile menu's close button is always visible.
```

```text
PSEUDO CODE

.cards { display: grid }
@media (max-width: 767px) {
  .cards { display: flex; overflow-x: auto; scroll-snap-type: x mandatory }
  .cards > * { flex: 0 0 80%; scroll-snap-align: start }
}

hero height = 100lvh (picture), copy inside 100svh
ScrollTrigger.config({ ignoreMobileResize: true })
pins on mobile: pinType "fixed" (no transform pin)
```

---

## 19. Speed and polish pass

```text
Test the full load time and speed of the site and fix anything slow:
- 3D scenes must not delay the rest of the page. Load them only near the
  viewport.
- Hero frames: coarse-to-fine loading, a limited number of requests at a time,
  smaller frames on phones, and keep them sharp (no quality drop).
- Serve images through the Next.js image optimiser with the right sizes.
- Replace the default favicon with the company logo mark.
- Add Privacy Policy and Terms pages.
- Run a production build and fix every error before deploying to Vercel.
```

```text
PSEUDO CODE

for each heavy thing (3D scene, frame sequence, large gallery):
  defer until idle or near viewport
  cap concurrent requests
  pick asset size from the device (DPR, width, pointer type)

npm run build  ->  fix errors  ->  deploy
```
