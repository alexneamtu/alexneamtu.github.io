# Astro Portfolio Redesign Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace the Jekyll resume site with a dark minimal Astro portfolio, deployed on GitHub Pages.

**Architecture:** Single-page Astro site with static HTML output. Sections: Hero, About, Skills, Experience, Education, Footer. Sticky nav with anchor links. `@media print` CSS for PDF download via `window.print()`. GitHub Actions deploys to Pages.

**Tech Stack:** Astro 5, TypeScript, vanilla CSS (no framework), GitHub Actions for deployment.

---

### Task 1: Scaffold Astro project

**Files:**
- Create: `package.json`, `astro.config.mjs`, `tsconfig.json`, `src/pages/index.astro`, `src/layouts/Layout.astro`
- Delete: `_config.yml`

**Step 1: Initialize Astro project in-place**

Remove the Jekyll config and scaffold Astro. Since the repo already has files, init manually:

```bash
cd /Users/aneamtu/Development/personal/alexneamtu.github.io
rm _config.yml
npm init astro -- --template minimal --no-install .
```

If the template command doesn't work in an existing dir, create these files manually:

`package.json`:
```json
{
  "name": "alexneamtu-portfolio",
  "type": "module",
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview"
  }
}
```

`astro.config.mjs`:
```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://alexneamtu.github.io',
});
```

`tsconfig.json`:
```json
{
  "extends": "astro/tsconfigs/strict"
}
```

`src/pages/index.astro`:
```astro
---
import Layout from '../layouts/Layout.astro';
---
<Layout title="Alex Neamtu — Senior Software Engineer">
  <h1>Alex Neamtu</h1>
</Layout>
```

`src/layouts/Layout.astro`:
```astro
---
interface Props {
  title: string;
}
const { title } = Astro.props;
---
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content="Alex Neamtu — Senior Software Engineer, Bucharest. 20+ years of experience in backend and full-stack development." />
    <title>{title}</title>
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
  </head>
  <body>
    <slot />
  </body>
</html>
```

**Step 2: Install dependencies**

```bash
npm install astro
```

**Step 3: Verify dev server starts**

```bash
npm run dev
```

Expected: Astro dev server running, page shows "Alex Neamtu" at localhost:4321.

**Step 4: Commit**

```bash
git add -A
git commit -m "feat: scaffold Astro project, remove Jekyll config"
```

---

### Task 2: Global styles and CSS variables

**Files:**
- Create: `src/styles/global.css`
- Modify: `src/layouts/Layout.astro`

**Step 1: Create global CSS with dark theme**

`src/styles/global.css`:
```css
:root {
  --color-bg: #0a0a0a;
  --color-bg-secondary: #111111;
  --color-text: #e4e4e7;
  --color-text-muted: #a1a1aa;
  --color-accent: #22d3ee;
  --color-accent-muted: rgba(34, 211, 238, 0.15);
  --color-border: #27272a;
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
  --max-width: 720px;
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;
  --space-xl: 3rem;
  --space-2xl: 5rem;
}

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  scroll-behavior: smooth;
  scroll-padding-top: 5rem;
}

body {
  font-family: var(--font-sans);
  background-color: var(--color-bg);
  color: var(--color-text);
  line-height: 1.7;
  font-size: 16px;
  -webkit-font-smoothing: antialiased;
}

a {
  color: var(--color-accent);
  text-decoration: none;
  transition: opacity 0.2s;
}

a:hover {
  opacity: 0.8;
}

::selection {
  background-color: var(--color-accent-muted);
  color: var(--color-accent);
}
```

**Step 2: Import global CSS and fonts in Layout**

Update `src/layouts/Layout.astro` `<head>`:
```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
<style is:global>
  @import '../styles/global.css';
</style>
```

**Step 3: Verify dark background renders**

```bash
npm run dev
```

Expected: Dark background, light text, Inter font.

**Step 4: Commit**

```bash
git add src/styles/global.css src/layouts/Layout.astro
git commit -m "feat: add dark theme global styles and typography"
```

---

### Task 3: Sticky navigation

**Files:**
- Create: `src/components/Nav.astro`
- Modify: `src/layouts/Layout.astro`

**Step 1: Create Nav component**

`src/components/Nav.astro`:
```astro
---
const links = [
  { label: 'About', href: '#about' },
  { label: 'Skills', href: '#skills' },
  { label: 'Experience', href: '#experience' },
  { label: 'Education', href: '#education' },
];
---
<nav class="nav">
  <div class="nav-inner">
    <a href="#" class="nav-logo">AN</a>
    <div class="nav-links" id="nav-links">
      {links.map(link => (
        <a href={link.href} class="nav-link">{link.label}</a>
      ))}
      <button class="nav-download" onclick="window.print()">
        Download PDF
      </button>
    </div>
    <button class="nav-toggle" id="nav-toggle" aria-label="Toggle menu">
      <span></span>
      <span></span>
      <span></span>
    </button>
  </div>
</nav>

<style>
  .nav {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 100;
    background-color: rgba(10, 10, 10, 0.85);
    backdrop-filter: blur(10px);
    border-bottom: 1px solid var(--color-border);
  }

  .nav-inner {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: 0.75rem var(--space-md);
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .nav-logo {
    font-family: var(--font-mono);
    font-weight: 700;
    font-size: 1.1rem;
    color: var(--color-accent);
  }

  .nav-links {
    display: flex;
    align-items: center;
    gap: var(--space-lg);
  }

  .nav-link {
    color: var(--color-text-muted);
    font-size: 0.875rem;
    font-weight: 500;
    transition: color 0.2s;
  }

  .nav-link:hover {
    color: var(--color-text);
    opacity: 1;
  }

  .nav-download {
    background: var(--color-accent);
    color: var(--color-bg);
    border: none;
    padding: 0.4rem 1rem;
    border-radius: 6px;
    font-size: 0.8rem;
    font-weight: 600;
    cursor: pointer;
    font-family: var(--font-sans);
    transition: opacity 0.2s;
  }

  .nav-download:hover {
    opacity: 0.85;
  }

  .nav-toggle {
    display: none;
    flex-direction: column;
    gap: 4px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 4px;
  }

  .nav-toggle span {
    display: block;
    width: 20px;
    height: 2px;
    background: var(--color-text);
    transition: transform 0.2s;
  }

  @media (max-width: 640px) {
    .nav-links {
      display: none;
      position: absolute;
      top: 100%;
      left: 0;
      right: 0;
      flex-direction: column;
      background-color: var(--color-bg-secondary);
      border-bottom: 1px solid var(--color-border);
      padding: var(--space-md);
      gap: var(--space-md);
    }

    .nav-links.open {
      display: flex;
    }

    .nav-toggle {
      display: flex;
    }
  }
</style>

<script>
  const toggle = document.getElementById('nav-toggle');
  const links = document.getElementById('nav-links');
  toggle?.addEventListener('click', () => {
    links?.classList.toggle('open');
  });
  // Close menu when a link is clicked
  links?.querySelectorAll('.nav-link').forEach(link => {
    link.addEventListener('click', () => {
      links?.classList.remove('open');
    });
  });
</script>
```

**Step 2: Add Nav to Layout**

In `src/layouts/Layout.astro`, add import and render `<Nav />` before `<slot />`:
```astro
---
import Nav from '../components/Nav.astro';
---
```
```html
<body>
  <Nav />
  <main style="padding-top: 4rem;">
    <slot />
  </main>
</body>
```

**Step 3: Verify nav renders and links scroll**

Expected: Sticky dark nav with "AN" logo, section links, Download PDF button, hamburger on mobile.

**Step 4: Commit**

```bash
git add src/components/Nav.astro src/layouts/Layout.astro
git commit -m "feat: add sticky navigation with mobile hamburger"
```

---

### Task 4: Hero section

**Files:**
- Create: `src/components/Hero.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create Hero component**

`src/components/Hero.astro`:
```astro
---
const socialLinks = [
  { label: 'GitHub', href: 'https://github.com/alexneamtu', icon: 'github' },
  { label: 'LinkedIn', href: 'https://www.linkedin.com/in/alexneamtu/', icon: 'linkedin' },
  { label: 'Twitter', href: 'https://twitter.com/neamtualexandru', icon: 'twitter' },
  { label: 'Email', href: 'mailto:alexneamtu@gmail.com', icon: 'email' },
];
---
<section class="hero" id="hero">
  <p class="hero-tagline">.code .coffee .cocktails</p>
  <h1 class="hero-name">Alex Neamtu</h1>
  <p class="hero-title">Senior Software Engineer</p>
  <p class="hero-location">Bucharest, Romania</p>
  <div class="hero-links">
    {socialLinks.map(link => (
      <a href={link.href} class="hero-social" target="_blank" rel="noopener noreferrer" aria-label={link.label}>
        {link.label}
      </a>
    ))}
  </div>
  <button class="hero-pdf" onclick="window.print()">
    ↓ Download Resume
  </button>
</section>

<style>
  .hero {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: var(--space-2xl) var(--space-md);
    min-height: 60vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .hero-tagline {
    font-family: var(--font-mono);
    color: var(--color-accent);
    font-size: 0.9rem;
    margin-bottom: var(--space-md);
  }

  .hero-name {
    font-size: clamp(2.5rem, 6vw, 4rem);
    font-weight: 700;
    line-height: 1.1;
    margin-bottom: var(--space-sm);
    color: var(--color-text);
  }

  .hero-title {
    font-size: 1.25rem;
    color: var(--color-text-muted);
    margin-bottom: var(--space-xs);
  }

  .hero-location {
    font-size: 0.9rem;
    color: var(--color-text-muted);
    margin-bottom: var(--space-lg);
  }

  .hero-links {
    display: flex;
    gap: var(--space-md);
    flex-wrap: wrap;
    margin-bottom: var(--space-lg);
  }

  .hero-social {
    color: var(--color-text-muted);
    font-size: 0.875rem;
    font-weight: 500;
    padding: 0.3rem 0.6rem;
    border: 1px solid var(--color-border);
    border-radius: 6px;
    transition: color 0.2s, border-color 0.2s;
  }

  .hero-social:hover {
    color: var(--color-accent);
    border-color: var(--color-accent);
    opacity: 1;
  }

  .hero-pdf {
    align-self: flex-start;
    background: none;
    color: var(--color-accent);
    border: 1px solid var(--color-accent);
    padding: 0.5rem 1.25rem;
    border-radius: 6px;
    font-size: 0.875rem;
    font-weight: 600;
    cursor: pointer;
    font-family: var(--font-sans);
    transition: background-color 0.2s, color 0.2s;
  }

  .hero-pdf:hover {
    background-color: var(--color-accent);
    color: var(--color-bg);
  }
</style>
```

**Step 2: Update index.astro**

```astro
---
import Layout from '../layouts/Layout.astro';
import Hero from '../components/Hero.astro';
---
<Layout title="Alex Neamtu — Senior Software Engineer">
  <Hero />
</Layout>
```

**Step 3: Verify hero renders**

Expected: Large name, tagline in mono/cyan, social links as bordered pills, download button.

**Step 4: Commit**

```bash
git add src/components/Hero.astro src/pages/index.astro
git commit -m "feat: add hero section with social links and PDF button"
```

---

### Task 5: About section

**Files:**
- Create: `src/components/About.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create About component**

`src/components/About.astro`:
```astro
<section class="section" id="about">
  <h2 class="section-heading">About</h2>
  <p class="about-text">
    Senior Software Engineer based in Bucharest with 20+ years of experience. Full-stack and backend specialist — from media delivery solutions and direct mail automation to video analysis systems and cloud-native serverless architectures on AWS. TypeScript and Node.js at the core, with experience across the stack from React frontends to Go microservices and Docker-based infrastructure.
  </p>
</section>

<style>
  .section {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: var(--space-2xl) var(--space-md);
  }

  .section-heading {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: var(--space-lg);
    color: var(--color-text);
    display: flex;
    align-items: center;
    gap: var(--space-md);
  }

  .section-heading::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--color-border);
  }

  .about-text {
    color: var(--color-text-muted);
    font-size: 1rem;
    line-height: 1.8;
  }
</style>
```

**Step 2: Add to index.astro**

Import and render `<About />` after `<Hero />`.

**Step 3: Commit**

```bash
git add src/components/About.astro src/pages/index.astro
git commit -m "feat: add about section"
```

---

### Task 6: Skills section

**Files:**
- Create: `src/components/Skills.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create Skills component**

`src/components/Skills.astro`:
```astro
---
const skillGroups = [
  {
    title: 'Languages',
    skills: [
      { name: 'TypeScript', level: 'primary' },
      { name: 'JavaScript', level: 'primary' },
      { name: 'PHP', level: 'strong' },
      { name: 'Go', level: 'working' },
      { name: 'Python', level: 'familiar' },
      { name: 'Shell/Bash', level: 'working' },
    ],
  },
  {
    title: 'Frontend',
    skills: ['React', 'AngularJS', 'Vite', 'shadcn/ui', 'HTML', 'CSS'],
  },
  {
    title: 'Backend',
    skills: ['Node.js', 'Koa', 'Fastify', 'GraphQL', 'REST APIs', 'Magento 2'],
  },
  {
    title: 'Databases',
    skills: ['MongoDB', 'PostgreSQL', 'MySQL', 'DynamoDB', 'Elasticsearch', 'Redis'],
  },
  {
    title: 'Cloud & DevOps',
    skills: ['AWS', 'GCP', 'Docker', 'Kubernetes', 'Cloudflare Workers', 'Turborepo'],
  },
  {
    title: 'Testing',
    skills: ['Jest'],
  },
];

function levelOpacity(level: string): string {
  switch (level) {
    case 'primary': return '1';
    case 'strong': return '0.85';
    case 'working': return '0.65';
    case 'familiar': return '0.5';
    default: return '1';
  }
}
---
<section class="section" id="skills">
  <h2 class="section-heading">Skills</h2>
  <div class="skill-groups">
    {skillGroups.map(group => (
      <div class="skill-group">
        <h3 class="skill-group-title">{group.title}</h3>
        <div class="skill-tags">
          {Array.isArray(group.skills) && group.skills.map(skill => {
            const name = typeof skill === 'string' ? skill : skill.name;
            const level = typeof skill === 'string' ? null : skill.level;
            return (
              <span
                class="skill-tag"
                style={level ? `opacity: ${levelOpacity(level)}` : ''}
                title={level ? `Proficiency: ${level}` : ''}
              >
                {name}
              </span>
            );
          })}
        </div>
      </div>
    ))}
  </div>
</section>

<style>
  .section {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: var(--space-2xl) var(--space-md);
  }

  .section-heading {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: var(--space-lg);
    color: var(--color-text);
    display: flex;
    align-items: center;
    gap: var(--space-md);
  }

  .section-heading::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--color-border);
  }

  .skill-groups {
    display: grid;
    gap: var(--space-lg);
  }

  .skill-group-title {
    font-size: 0.8rem;
    font-weight: 600;
    color: var(--color-text-muted);
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: var(--space-sm);
  }

  .skill-tags {
    display: flex;
    flex-wrap: wrap;
    gap: var(--space-sm);
  }

  .skill-tag {
    font-family: var(--font-mono);
    font-size: 0.8rem;
    color: var(--color-accent);
    background: var(--color-accent-muted);
    padding: 0.25rem 0.65rem;
    border-radius: 4px;
  }
</style>
```

**Step 2: Add to index.astro**

Import and render `<Skills />` after `<About />`.

**Step 3: Commit**

```bash
git add src/components/Skills.astro src/pages/index.astro
git commit -m "feat: add skills section with proficiency indicators"
```

---

### Task 7: Experience section

**Files:**
- Create: `src/components/Experience.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create Experience component**

Use condensed descriptions from README.md merged with structure from github-profile.md.

`src/components/Experience.astro`:
```astro
---
const experiences = [
  {
    role: 'Senior Software Engineer',
    company: 'Archbee',
    url: 'https://www.archbee.com/',
    period: '2025 – Present',
    description: null,
  },
  {
    role: 'Senior Software Engineer',
    company: 'Sand Technologies',
    url: 'https://www.sandtech.com/',
    period: '2021 – Present',
    description: 'Designed and implemented cloud-native serverless APIs and workflows on AWS. Built systems for AI content generation (Diginym), video analysis (CLIPr), trading card marketplace (CardSeer), and talent matching (The Room).',
  },
  {
    role: 'Freelance Software Developer',
    company: 'Self-Employed',
    url: null,
    period: '2012 – Present',
    description: 'Built Risc Seismic Bucuresti (seismic risk checker), contributed to Liquid Investigations (secure journalism collaboration server) and Hoover (document search toolset).',
  },
  {
    role: 'Senior Full Stack Developer',
    company: 'Scenset',
    url: 'https://scenset.com/',
    period: '2021 – 2025',
    description: 'Frontend and backend development for personalized travel platform. React, Node.js, Cloud Firestore, Algolia on GCP with Pulumi.',
  },
  {
    role: 'Senior Backend Developer',
    company: 'optilyz',
    url: 'https://www.optilyz.com/',
    period: '2020 – 2021',
    description: 'Designed internal and external APIs for Europe\'s leading direct mail automation software.',
  },
  {
    role: 'Senior Software Engineer',
    company: 'Eau de Web',
    url: 'https://www.eaudeweb.ro/',
    period: '2019 – 2020',
    description: 'Built School Meals (WFP food delivery management), Ogor (satellite imagery farm analytics), and a UN/TED procurement scraper. Taught Django at University of Bucharest.',
  },
  {
    role: 'Senior Backend Developer',
    company: 'OWNZONES Entertainment',
    url: 'https://ownzones.com/',
    period: '2017 – 2019',
    description: 'Designed and built the Discover media delivery platform with Node.js, TypeScript, GraphQL, and Kubernetes on AWS.',
  },
  {
    role: 'Earlier Roles',
    company: 'Endava, Coinzone, Pionix, Axway, Stefanini, Ascensys',
    url: null,
    period: '2004 – 2017',
    description: 'Lead Designer and Tech Lead at Endava (paywall systems for US publications). Bitcoin payment gateway at Coinzone. Automation engineering at Axway. E-commerce development at Ascensys.',
  },
];
---
<section class="section" id="experience">
  <h2 class="section-heading">Experience</h2>
  <div class="timeline">
    {experiences.map(exp => (
      <div class="timeline-item">
        <div class="timeline-header">
          <div>
            <h3 class="timeline-role">{exp.role}</h3>
            <p class="timeline-company">
              {exp.url ? (
                <a href={exp.url} target="_blank" rel="noopener noreferrer">{exp.company}</a>
              ) : (
                exp.company
              )}
            </p>
          </div>
          <span class="timeline-period">{exp.period}</span>
        </div>
        {exp.description && (
          <p class="timeline-desc">{exp.description}</p>
        )}
      </div>
    ))}
  </div>
</section>

<style>
  .section {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: var(--space-2xl) var(--space-md);
  }

  .section-heading {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: var(--space-lg);
    color: var(--color-text);
    display: flex;
    align-items: center;
    gap: var(--space-md);
  }

  .section-heading::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--color-border);
  }

  .timeline {
    display: grid;
    gap: var(--space-lg);
  }

  .timeline-item {
    padding-left: var(--space-lg);
    border-left: 1px solid var(--color-border);
    position: relative;
  }

  .timeline-item::before {
    content: '';
    position: absolute;
    left: -4px;
    top: 6px;
    width: 7px;
    height: 7px;
    border-radius: 50%;
    background: var(--color-accent);
  }

  .timeline-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: var(--space-md);
    margin-bottom: var(--space-xs);
  }

  .timeline-role {
    font-size: 1rem;
    font-weight: 600;
    color: var(--color-text);
  }

  .timeline-company {
    font-size: 0.875rem;
    color: var(--color-text-muted);
  }

  .timeline-period {
    font-family: var(--font-mono);
    font-size: 0.8rem;
    color: var(--color-text-muted);
    white-space: nowrap;
    flex-shrink: 0;
  }

  .timeline-desc {
    font-size: 0.875rem;
    color: var(--color-text-muted);
    line-height: 1.7;
    margin-top: var(--space-sm);
  }

  @media (max-width: 640px) {
    .timeline-header {
      flex-direction: column;
      gap: 0;
    }

    .timeline-period {
      margin-top: 2px;
    }
  }
</style>
```

**Step 2: Add to index.astro**

Import and render `<Experience />` after `<Skills />`.

**Step 3: Commit**

```bash
git add src/components/Experience.astro src/pages/index.astro
git commit -m "feat: add experience timeline section"
```

---

### Task 8: Education section

**Files:**
- Create: `src/components/Education.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create Education component**

`src/components/Education.astro`:
```astro
---
const education = [
  {
    degree: 'BSc Computer Science',
    school: 'Politehnica University of Bucharest',
    url: 'https://upb.ro',
    period: '2002 – 2007',
    detail: 'Computer Systems Architecture specialization',
  },
  {
    degree: 'Cisco CCNA 1',
    school: 'Certification',
    url: null,
    period: '',
    detail: null,
  },
];
---
<section class="section" id="education">
  <h2 class="section-heading">Education</h2>
  <div class="edu-list">
    {education.map(edu => (
      <div class="edu-item">
        <div class="edu-header">
          <div>
            <h3 class="edu-degree">{edu.degree}</h3>
            <p class="edu-school">
              {edu.url ? (
                <a href={edu.url} target="_blank" rel="noopener noreferrer">{edu.school}</a>
              ) : (
                edu.school
              )}
            </p>
          </div>
          {edu.period && <span class="edu-period">{edu.period}</span>}
        </div>
        {edu.detail && <p class="edu-detail">{edu.detail}</p>}
      </div>
    ))}
  </div>
</section>

<style>
  .section {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: var(--space-2xl) var(--space-md);
  }

  .section-heading {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: var(--space-lg);
    color: var(--color-text);
    display: flex;
    align-items: center;
    gap: var(--space-md);
  }

  .section-heading::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--color-border);
  }

  .edu-list {
    display: grid;
    gap: var(--space-lg);
  }

  .edu-item {
    padding-left: var(--space-lg);
    border-left: 1px solid var(--color-border);
    position: relative;
  }

  .edu-item::before {
    content: '';
    position: absolute;
    left: -4px;
    top: 6px;
    width: 7px;
    height: 7px;
    border-radius: 50%;
    background: var(--color-accent);
  }

  .edu-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: var(--space-md);
  }

  .edu-degree {
    font-size: 1rem;
    font-weight: 600;
    color: var(--color-text);
  }

  .edu-school {
    font-size: 0.875rem;
    color: var(--color-text-muted);
  }

  .edu-period {
    font-family: var(--font-mono);
    font-size: 0.8rem;
    color: var(--color-text-muted);
    white-space: nowrap;
  }

  .edu-detail {
    font-size: 0.875rem;
    color: var(--color-text-muted);
    margin-top: var(--space-xs);
  }
</style>
```

**Step 2: Add to index.astro**

Import and render `<Education />` after `<Experience />`.

**Step 3: Commit**

```bash
git add src/components/Education.astro src/pages/index.astro
git commit -m "feat: add education section"
```

---

### Task 9: Footer

**Files:**
- Create: `src/components/Footer.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create Footer component**

`src/components/Footer.astro`:
```astro
---
const links = [
  { label: 'GitHub', href: 'https://github.com/alexneamtu' },
  { label: 'LinkedIn', href: 'https://www.linkedin.com/in/alexneamtu/' },
  { label: 'Twitter', href: 'https://twitter.com/neamtualexandru' },
  { label: 'Email', href: 'mailto:alexneamtu@gmail.com' },
];
const year = new Date().getFullYear();
---
<footer class="footer">
  <div class="footer-inner">
    <div class="footer-links">
      {links.map(link => (
        <a href={link.href} target="_blank" rel="noopener noreferrer">{link.label}</a>
      ))}
    </div>
    <p class="footer-copy">&copy; {year} Alex Neamtu</p>
  </div>
</footer>

<style>
  .footer {
    border-top: 1px solid var(--color-border);
    margin-top: var(--space-2xl);
  }

  .footer-inner {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: var(--space-xl) var(--space-md);
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .footer-links {
    display: flex;
    gap: var(--space-md);
  }

  .footer-links a {
    color: var(--color-text-muted);
    font-size: 0.875rem;
  }

  .footer-links a:hover {
    color: var(--color-accent);
    opacity: 1;
  }

  .footer-copy {
    font-size: 0.8rem;
    color: var(--color-text-muted);
  }

  @media (max-width: 640px) {
    .footer-inner {
      flex-direction: column;
      gap: var(--space-md);
      text-align: center;
    }
  }
</style>
```

**Step 2: Add to index.astro**

Import and render `<Footer />` after `<Education />`.

**Step 3: Commit**

```bash
git add src/components/Footer.astro src/pages/index.astro
git commit -m "feat: add footer with social links"
```

---

### Task 10: Print CSS for PDF download

**Files:**
- Create: `src/styles/print.css`
- Modify: `src/layouts/Layout.astro`

**Step 1: Create print stylesheet**

`src/styles/print.css`:
```css
@media print {
  :root {
    --color-bg: #ffffff;
    --color-text: #111111;
    --color-text-muted: #444444;
    --color-accent: #0369a1;
    --color-border: #d4d4d8;
  }

  body {
    font-size: 11pt;
    line-height: 1.4;
    background: white;
    color: black;
  }

  .nav,
  .hero-pdf,
  .nav-download,
  .footer {
    display: none !important;
  }

  .hero {
    min-height: auto !important;
    padding: 0 !important;
    margin-bottom: 1rem;
  }

  .hero-name {
    font-size: 22pt !important;
  }

  .hero-tagline {
    display: none;
  }

  .hero-links {
    margin-bottom: 0.5rem;
  }

  .hero-social {
    border: none !important;
    padding: 0 !important;
    font-size: 9pt;
  }

  .section {
    padding: 0.75rem 0 !important;
  }

  .section-heading {
    font-size: 13pt;
    margin-bottom: 0.5rem;
  }

  .timeline-item,
  .edu-item {
    break-inside: avoid;
  }

  .skill-tag {
    font-size: 8pt;
    padding: 1px 4px;
    border: 1px solid var(--color-border);
    background: none;
    color: var(--color-text);
  }

  a {
    color: var(--color-text) !important;
    text-decoration: none;
  }

  a[href^="http"]::after {
    content: none;
  }
}
```

**Step 2: Import print CSS in Layout**

Add to the `<style is:global>` block in Layout.astro:
```css
@import '../styles/print.css';
```

**Step 3: Test print preview**

Open the site and press Cmd+P. Expected: clean single-column resume, white background, no nav/footer, readable typography.

**Step 4: Commit**

```bash
git add src/styles/print.css src/layouts/Layout.astro
git commit -m "feat: add print stylesheet for PDF download"
```

---

### Task 11: GitHub Actions workflow for Astro

**Files:**
- Modify: `.github/workflows/pages.yml`

**Step 1: Replace Jekyll workflow with Astro**

`.github/workflows/pages.yml`:
```yaml
name: Deploy Astro site to GitHub Pages

on:
  push:
    branches: ["master"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

This removes the Jekyll build and PDF generation jobs entirely. PDF is now handled client-side.

**Step 2: Commit**

```bash
git add .github/workflows/pages.yml
git commit -m "feat: replace Jekyll workflow with Astro build and deploy"
```

---

### Task 12: Update README.md and cleanup

**Files:**
- Modify: `README.md`
- Modify: `CLAUDE.md`
- Delete: none (Jekyll config already removed in task 1)

**Step 1: Update README.md**

Replace the resume content with a project readme:

```markdown
# alexneamtu.github.io

Personal portfolio and resume site built with [Astro](https://astro.build/).

## Development

```bash
npm install
npm run dev      # Start dev server at localhost:4321
npm run build    # Build for production
npm run preview  # Preview production build
```

## Deployment

Pushes to `master` trigger GitHub Actions to build and deploy to GitHub Pages.

## Resume PDF

Click "Download Resume" on the site or use your browser's print function (Cmd/Ctrl+P).
```

**Step 2: Update CLAUDE.md**

Update to reflect Astro instead of Jekyll.

**Step 3: Add .gitignore**

```
node_modules/
dist/
.astro/
```

**Step 4: Commit**

```bash
git add README.md CLAUDE.md .gitignore
git commit -m "docs: update README and CLAUDE.md for Astro, add .gitignore"
```

---

### Task 13: Final build verification

**Step 1: Full build**

```bash
npm run build
```

Expected: Clean build, output in `dist/`.

**Step 2: Preview**

```bash
npm run preview
```

Expected: Site renders at localhost:4321 with all sections, nav works, print CSS produces clean PDF.

**Step 3: Verify responsive**

Check mobile layout in browser dev tools. Nav collapses to hamburger, timeline stacks vertically.
