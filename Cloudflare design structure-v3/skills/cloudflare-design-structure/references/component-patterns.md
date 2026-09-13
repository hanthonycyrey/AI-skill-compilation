# Component Patterns

Concrete starting points for the structural patterns described in SKILL.md. These are original implementations of the *pattern*, not copies of Cloudflare's actual code — adapt naming, spacing, and framework to whatever the project already uses (plain CSS, Tailwind, React, etc.).

## Table of contents
- [Asymmetric bento grid](#asymmetric-bento-grid)
- [Count-up stat card](#count-up-stat-card)
- [Live workflow/agent status cards](#live-workflowagent-status-cards)
- [Ticker / threshold-crossing strip](#ticker--threshold-crossing-strip)
- [Ambient hero background](#ambient-hero-background)

---

## Asymmetric bento grid

The unevenness is the whole point — avoid `repeat(3, 1fr)` with identical cards.

```css
.bento-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-rows: 220px;
  gap: 1.5rem;
  max-width: 1280px;
  margin-inline: auto;
}

.bento-card--wide   { grid-column: span 2; }
.bento-card--tall   { grid-row: span 2; }
.bento-card--hero   { grid-column: span 2; grid-row: span 2; }

@media (max-width: 768px) {
  .bento-grid { grid-template-columns: 1fr; }
  .bento-card--wide,
  .bento-card--hero { grid-column: span 1; }
}
```

Assign `--hero` to one or two cards that deserve the most visual weight; leave the rest as plain 1×1 cells.

## Count-up stat card

Animate the number on scroll-into-view rather than showing a static figure.

```jsx
function useCountUp(target, durationMs = 1200, start = false) {
  const [value, setValue] = useState(0);

  useEffect(() => {
    if (!start) return;
    let raf;
    const t0 = performance.now();
    const tick = (now) => {
      const progress = Math.min((now - t0) / durationMs, 1);
      // ease-out cubic
      const eased = 1 - Math.pow(1 - progress, 3);
      setValue(Math.round(target * eased));
      if (progress < 1) raf = requestAnimationFrame(tick);
    };
    raf = requestAnimationFrame(tick);
    return () => cancelAnimationFrame(raf);
  }, [start, target, durationMs]);

  return value;
}

function StatCard({ value, suffix, label }) {
  const ref = useRef(null);
  const [inView, setInView] = useState(false);
  const displayValue = useCountUp(value, 1200, inView);

  useEffect(() => {
    const obs = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) setInView(true);
    }, { threshold: 0.4 });
    if (ref.current) obs.observe(ref.current);
    return () => obs.disconnect();
  }, []);

  return (
    <div ref={ref} className="stat-card">
      <div className="stat-card__number">{displayValue}{suffix}</div>
      <div className="stat-card__label">{label}</div>
    </div>
  );
}
```

## Live workflow/agent status cards

The three ingredients: a kicker line, an incrementing counter, and staggered per-unit cards whose action text can change.

```jsx
const PIPELINE_UNITS = [
  { id: "parse-agent", action: "extracting skills", metric: "resumes", delayMs: 0 },
  { id: "match-agent", action: "ranking roles", metric: "matches", delayMs: 600 },
  { id: "verify-agent", action: "checking credentials", metric: "checks", delayMs: 1200 },
];

function WorkflowStatusCluster() {
  const [visibleUnits, setVisibleUnits] = useState([]);
  const [counts, setCounts] = useState({});

  useEffect(() => {
    const timers = PIPELINE_UNITS.map((unit) =>
      setTimeout(() => {
        setVisibleUnits((prev) => [...prev, unit]);
        // simulate a live-incrementing metric per unit
        const interval = setInterval(() => {
          setCounts((prev) => ({
            ...prev,
            [unit.id]: (prev[unit.id] ?? 0) + Math.ceil(Math.random() * 3),
          }));
        }, 1400);
        return () => clearInterval(interval);
      }, unit.delayMs)
    );
    return () => timers.forEach(clearTimeout);
  }, []);

  return (
    <div className="workflow-cluster">
      <p className="workflow-cluster__kicker">
        Processing new applicants… {visibleUnits.length} agents active
      </p>
      <div className="workflow-cluster__cards">
        {visibleUnits.map((unit) => (
          <div key={unit.id} className="workflow-card workflow-card--enter">
            <span className="workflow-card__id">{unit.id}</span>
            <span className="workflow-card__action">{unit.action}</span>
            <span className="workflow-card__metric">
              {counts[unit.id] ?? 0} {unit.metric}
            </span>
          </div>
        ))}
      </div>
    </div>
  );
}
```

```css
.workflow-card--enter {
  animation: workflow-fade-in 400ms ease-out both;
}
@keyframes workflow-fade-in {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

If the pipeline is real (e.g. Tarbajo's actual parsing pipeline), wire `counts` and `visibleUnits` to real events/websocket messages instead of the `setInterval` simulation above — the pattern works identically either way, but real data is always more convincing than simulated data if it's available.

## Ticker / threshold-crossing strip

Represents time or usage as a strip of small units that flip state at a threshold — used for the "pay only when code runs" style demo.

```jsx
function ThresholdTicker({ totalUnits = 60, thresholdUnit = 45, unitMs = 60 }) {
  const [activeUnit, setActiveUnit] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setActiveUnit((u) => (u + 1) % totalUnits);
    }, unitMs);
    return () => clearInterval(id);
  }, [totalUnits, unitMs]);

  return (
    <div className="ticker">
      {Array.from({ length: totalUnits }).map((_, i) => (
        <span
          key={i}
          className={[
            "ticker__unit",
            i <= activeUnit ? "ticker__unit--played" : "",
            i === thresholdUnit ? "ticker__unit--threshold" : "",
            i > thresholdUnit && i <= activeUnit ? "ticker__unit--paid" : "",
          ].join(" ")}
        >
          {i > thresholdUnit && i <= activeUnit ? "paid" : "free"}
        </span>
      ))}
    </div>
  );
}
```

## Ambient hero background

Keep it cheap and reduced-motion-safe. A lightweight canvas particle/line treatment is usually enough — reach for a full three.js globe only if the project specifically wants an interactive, draggable visualization elsewhere on the site.

```jsx
function AmbientHeroCanvas() {
  const canvasRef = useRef(null);
  const prefersReducedMotion = useRef(
    typeof window !== "undefined" &&
      window.matchMedia("(prefers-reduced-motion: reduce)").matches
  );

  useEffect(() => {
    if (prefersReducedMotion.current) return; // fall back to poster image in JSX below
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");
    let raf;
    let t = 0;

    function resize() {
      canvas.width = canvas.offsetWidth * devicePixelRatio;
      canvas.height = canvas.offsetHeight * devicePixelRatio;
    }
    resize();
    window.addEventListener("resize", resize);

    function draw() {
      t += 0.004;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      // draw slow-drifting connecting lines/nodes here — kept deliberately simple;
      // swap in three.js / react-globe.gl if a rotating globe is specifically wanted
      raf = requestAnimationFrame(draw);
    }
    draw();
    return () => {
      cancelAnimationFrame(raf);
      window.removeEventListener("resize", resize);
    };
  }, []);

  if (prefersReducedMotion.current) {
    return <img src="/hero-poster.avif" alt="" className="hero__poster" />;
  }
  return <canvas ref={canvasRef} className="hero__canvas" aria-hidden="true" />;
}
```
