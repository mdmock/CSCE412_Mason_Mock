FORWARD CURVE
Energy Advisory — Containerization Project Documentation
=========================================================

## What this project is

Forward Curve is a small energy-advisory brand built for this assignment. It
consists of five static websites, all sharing one visual identity:

| Service      | Purpose                                              | Folder         |
|--------------|-------------------------------------------------------|----------------|
| Hub          | Firm homepage, links out to the four tools below       | `hub/`         |
| Curve        | Commodities spot/forward pricing dashboard              | `curve/`       |
| Ledger       | Emissions and ESG reporting                             | `ledger/`      |
| Portal       | Client sign-in page                                      | `portal/`      |
| Field Notes  | Team insights and open roles                             | `fieldnotes/`  |

Each site is plain HTML/CSS (no build step, no backend) so it can be served
directly by nginx inside a container.

## Components and tools used

**Containers vs. virtual machines.** A container packages an application with
just the dependencies it needs and shares the host machine's kernel, so it
starts in about a second and uses a fraction of the resources a full VM would.
A VM virtualizes an entire machine, including its own OS kernel. This project
uses containers because each site only needs one thing — nginx — to serve
static files.

**Docker.** Docker builds and runs the containers. Two pieces matter here:
- A **Dockerfile** is a recipe for an image: it starts `FROM` a base image
  (`nginx:stable-alpine`, a minimal Linux + nginx image), `COPY`s this site's
  files into nginx's web root, `EXPOSE`s port 80, and sets the `CMD` that
  runs when the container starts.
- An **image** is the built, frozen result of that recipe. A **container** is
  a running instance of an image — you can run the same image many times at
  once, which is exactly what Part E below does.

**Docker Compose.** `docker-compose` lets you define and run several
containers together from one YAML file instead of typing `docker run`
repeatedly. Each `service:` entry maps to one container, with its own build
context and port mapping.

**nginx.** A lightweight, widely used web server. Inside each container it
just serves the static files it was given on port 80.

**AWS (EC2 / Lightsail).** The Ubuntu instance is where Docker actually runs.
Getting this working also means practicing cloud fundamentals that are
separate from Docker itself: provisioning a Linux server, connecting over
SSH, and opening inbound firewall rules (HTTP/8080-range ports, ICMP, SSH) so
the containers are reachable from a browser outside the instance.

## Part E — five identical containers

`docker-compose.identical.yml` builds the `hub/` image once and runs it five
times, each bound to a different host port (8081–8085). This demonstrates
that a container is a repeatable, disposable unit — the same image scales
horizontally just by starting more copies of it.

```
docker-compose -f docker-compose.identical.yml up -d --build
```

Visit `http://<instance-ip>:8081` through `:8085` — all five show the same
Forward Curve homepage.

## Part F — five unique containers from a shared base

`docker-compose.unique.yml` builds all five folders (`hub`, `curve`,
`ledger`, `portal`, `fieldnotes`) as five separate images and runs one
container per service on ports 9081–9085. Each uses the identical
Dockerfile pattern from Part C/D, but a different `COPY` context, so the
content differs while the underlying recipe doesn't.

```
docker-compose -f docker-compose.unique.yml up -d --build
```

Visit `http://<instance-ip>:9081` (Hub) through `:9085` (Field Notes). The
navigation bar on every page links to the others using these same ports —
if you deploy to a real instance, update the `http://localhost:908X/` links
in each `index.html` to use the instance's public IP before rebuilding.

## Design notes

All five sites share one stylesheet (`styles.css`, duplicated into each
folder so every container is self-contained): a petrol-green and amber
palette on a warm paper background, Source Serif 4 for headlines, IBM Plex
Sans for body text, and IBM Plex Mono for tabular/pricing data — a nod to
the numbers-heavy nature of the content. The same masthead, wordmark, and
nav bar appear on every page so the five containers read as one product
suite rather than five disconnected demo pages.

## Repo layout

```
forward-curve/
├── README.md
├── docker-compose.identical.yml
├── docker-compose.unique.yml
├── hub/            (Home)
├── curve/          (Pricing dashboard)
├── ledger/         (Emissions reporting)
├── portal/         (Client sign-in)
└── fieldnotes/     (Insights & careers)
    each folder: Dockerfile, index.html, styles.css
```
